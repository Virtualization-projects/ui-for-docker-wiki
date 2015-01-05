## Run dockerui on your host machine

Quickstart:
To create a new image and start a container with it:
```
make build run
```
Or run natively on your computer:
```
go build dockerui.go
./dockerui
```
Note: Dockerui looks for the docker daemon on `/var/run/docker.sock` by default, use `-e` to override this. 
```
./dockerui -e /var/run/docker.sock
or...
./dockerui -e http://192.168.59.103:2376
```
Dockerui does not support TLS, as a result users with Docker 1.3.0 or greater installed will not be able to use tcp ports with the `-e` option.

## Run docker with source code in a mounted volume

This is the fastest way to develop with the HTML, CSS, and Javascript portion of dockerui.

- clone this repo to /path/to/dockerui and `cd` into that directory
- in the `Dockerfile` change `ADD . /app/` to `VOLUME /app`
- build the image : `docker build -t me/dockerui ./`
- run the container with `/path/to/dockerui` binded to `/app`

```
docker run -d \
    -p 9000:9000 \
    -v /path/to/dockerui:/app me/dockerui \
    -v /var/run/docker.sock:/docker.sock \
    me/dockerui \
    -e /docker.sock
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