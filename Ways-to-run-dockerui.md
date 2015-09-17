## Run DockerUI as a container on a Docker host (Boot2Docker on OS X)
To create a new image and start a container with it:
```
grunt run  # or run-dev for auto-reloading when source files change
```

## Run natively on your computer
```
grunt build
cd dist
./dockerui &
grunt watch
```
Note: Dockerui looks for the docker daemon on `/var/run/docker.sock` by default, use the `-e` flag to override this. 
```
./dockerui -e /var/run/docker.sock
or...
./dockerui -e http://192.168.59.103:2376
```
Dockerui does not support TLS natively, as a result users with Docker 1.3.0 or greater installed will not be able to use tcp ports with the `-e` flag. See [DockerUI-with-TLS-encryption-and-client-authentication](https://github.com/crosbymichael/dockerui/wiki/DockerUI-with-TLS-encryption-and-client-authentication) for other ways to use TLS. 

You can change the default port using `-p`:
```
./dockerui -p 0.0.0.0:9001
```

## Run docker with source code in a mounted volume

> Warning: DockerUI uses http.FileServer which does not mix well with VirtualBox's volume sharing. If you use this approach you may have issues with the server not picking up changes to source files. The `grunt run-dev` approach is recommended for OS X users. 

This is the fastest way to develop with the HTML, CSS, and Javascript portion of dockerui.

Use the following Dockerfile
```
FROM scratch
COPY dist /app
EXPOSE 9000
WORKDIR /app
ENTRYPOINT ["dockerui"]
```
And run with 
```
grunt if:binaryNotExist build shell:buildImage
docker run -d \
    -p 9000:9000 \
    -v $(pwd)/dist:/app \
    -v /var/run/docker.sock:/docker.sock \
    me/dockerui \
    -e /docker.sock
grunt watch
```

## Run under apache with mod_proxy

- clone it into /var/www/dockerui
- enable mod_proxy and the http_proxy handler
- create a virtualhost
- enable proxypass to pipe /dockerapi request to your docker host

### VirtualHost

```
<VirtualHost *:80>
    ServerName docker.yourcompany
    DocumentRoot /var/www/dockerui
    DirectoryIndex index.html
    ProxyPass /dockerapi/ http://127.0.0.01:4243/
    ProxyPassReverse /dockerapi/ http://127.0.0.1:4243/
</VirtualHost>
```

### mod_proxy

Make sure that you have mod_proxy enabled

```
$ ls -l /etc/apache2/mods-enabled/ |grep proxy
lrwxrwxrwx 1 root root 28 juil. 18 11:09 proxy.conf -> ../mods-available/proxy.conf
lrwxrwxrwx 1 root root 33 juil. 18 11:17 proxy_http.load -> ../mods-available/proxy_http.load
lrwxrwxrwx 1 root root 28 juil. 18 11:09 proxy.load -> ../mods-available/proxy.load
```

If not and if you're on debian for instance : run `a2enmod proxy && a2enmod proxy_http`

### Docker host config

Make sure that it is listening on port 4243