# Introduction

The native DockerUI container does not support any form of encryption or security controls; however, if you already have TLS encryption between your Docker CLI and hosts, you can use the same set of certificates to both encrypt DockerUI traffic and provide client verification/authentication. This is done using Apache with mod_proxy and mod_ssl. These instructions should work with boot2docker's TLS configuration, although I haven't tested it yet.

# Setup steps

## Certificate setup

First, be sure your personal Docker client certificate (sometimes located in `~/.docker/cert.pem`) has been imported into your browser's certificate database. For Mac users on Chrome or Safari, you should import the certificate into your Mac's Keychain - this can be done by opening the .pem file with Keychain Access or with the command `security import ~/.docker/cert.pem -k ~/Library/Keychains/login.keychain`. For Firefox users, certificates can be managed in Preferences > Advanced > Certificates > View Certificates. Select the "Your Certificates" tab and click the "Import..." button. **Important:** Be sure that both your certificate and private key get imported!

Next, issue a certificate from your Docker host's CA to be used by DockerUI's Apache server. This certificate will be used both to encrypt traffic between the client and DockerUI, as well as to encrypt and authenticate traffic proxied to the Docker host. Following this, you should have three files: `ca.pem` with the CA's certificate, `cert.pem` with the certificate for DockerUI, and `key.pem` with the private key corresponding to DockerUI's certificate. Create a fourth file named `cert_and_key.pem` that has `cert.pem` concatenated with `key.pem` - this will be used by Apache later on.

## Apache setup

Build an Apache container with mod_proxy, mod_proxy_http, mod_ssl and mod_rewrite. This can be done with a Dockerfile like this:

    FROM ubuntu
    
    RUN apt-get update && apt-get install -y apache2 \
        && apt-get clean && rm -rf /var/lib/apt/lists* /tmp/* /var/tmp/*
    
    RUN a2enmod proxy && a2enmod proxy_http && a2enmod ssl && a2enmod rewrite
    
    EXPOSE 80
    EXPOSE 443

    CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]

Create an Apache configuration file that enables SSL with client verification and client proxying.

    <VirtualHost *:443>
      ServerAdmin webmaster@localhost
      DocumentRoot /var/www/dockerui/dist
      
      SSLEngine on
      SSLProtocol all -SSLv2 -SSLv3
      SSLCipherSuite HIGH:MEDIUM

      # Verify the client certificate. Comment these out if you want SSL encryption 
      # but will do authentication via another mechanism.
      SSLVerifyClient require
      SSLVerifyDepth 2
      
      SSLProxyEngine on
      
      # Certificates for use between client and Apache.
      SSLCACertificateFile  /certs/ca.pem
      SSLCertificateFile    /certs/cert.pem
      SSLCertificateKeyFile /certs/key.pem
      
      # Certificates for use between Apache and Docker host.
      SSLProxyMachineCertificateChainFile /certs/ca.pem
      SSLProxyMachineCertificateFile      /certs/cert_and_key.pem
      SSLProxyCheckPeerName off
      SSLProxyCheckPeerCN off
      
      LogLevel debug
      ErrorLog ${APACHE_LOG_DIR}/error.log
      CustomLog ${APACHE_LOG_DIR}/access.log combined
      
      <Location />
        SSLRequireSSL
      </Location>
      
      ProxyPreserveHost On
      ProxyPass /dockerapi/ https://<docker-host-ip-and-port>/
      ProxyPassReverse /dockerapi/ https://<docker-host-ip-and-port>/
    </VirtualHost>

Don't miss replacing the `<docker-host-ip-and-port>` text in the `ProxyPass` and `ProxyPassReverse` commands with the appropriate Docker host IP address and port (the name and port typically set in `$DOCKER_HOST` on the client).

If you want automatic redirection of http to https, add this to your Apache config file:

    <VirtualHost *:80>
      ServerAdmin webmaster@localhost
      DocumentRoot /var/www/html
      
      RewriteEngine on
      RewriteCond %{SERVER_PORT} ^80$
      RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [L,R]
    </VirtualHost>

Save this to a file (such as `000-default.conf`) and add it to your Dockerfile.

    COPY 000-default.conf /etc/apache2/sites-available/

## DockerUI setup

The only step remaining is to build the DockerUI site and add it to /var/www/dockerui/dist in the Apache container. You can do this locally on your system (if you're already set up with node.js and grunt) and just copy the site into the container in your Dockerfile:

    COPY dist /var/www/dockerui/dist/

Or, you can build the site during the container build process with an extra Dockerfile RUN command:

    RUN apt-get install -y git nodejs npm \
        && ln -s /usr/bin/nodejs /usr/bin/node \
        && git clone https://github.com/crosbymichael/dockerui /var/www/dockerui \
        && cd /var/www/dockerui && make install && grunt build

(You should only do one of the two options above.)

# Running the newly built container

Be sure to mount the directory containing DockerUI's certs as a volume into the container at runtime, as well as exposing both ports 80 and 443. Also, it is recommended that you set the hostname of the container equal to the hostname users will be using to access the DockerUI site. 

For example:

    $ docker run -h docker.mydomain.internal --name dockerui \
          -v /private/certs/dockerui:/certs \
          -p 80:80 -p 443:443 \
          myrepo/dockerui

# Troubleshooting

`docker exec -ti dockerui bash` will let you browse around the filesystem to read logs; particularly those stored in `/var/log/apache2/access.log` and `/var/log/apache2/error.log`.