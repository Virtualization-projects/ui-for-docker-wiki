## Run UI for Docker as a container on a Docker host (Boot2Docker or docker-machine on OS X)
To create a new image and start a container with it:
```
grunt run  # or run-dev for auto-reloading when source files change
```

## Run natively on your computer
```
grunt build
cd dist
./ui-for-docker

```
And in another terminal run `grunt watch` to automatically rebuild the Angular app when source files change.
Note: UI for Docker looks for the docker daemon on `/var/run/docker.sock` by default, use the `-e` flag to override this. 
```
./ui-for-docker -e /var/run/docker.sock
or...
./ui-for-docker -e http://192.168.59.103:2376
```
UI for Docker does not support TLS natively, as a result users with Docker 1.3.0 or greater installed will not be able to use tcp ports with the `-e` flag. See [UI-for-Docker-with-TLS-encryption-and-client-authentication](https://github.com/kevana/ui-for-docker/wiki/UI-for-Docker-with-TLS-encryption-and-client-authentication) for other ways to use TLS. 

You can change the default port using `-p`:
```
./ui-for-docker -p 0.0.0.0:9001
```

## Run Docker with source code in a mounted volume

> Warning: UI for Docker uses http.FileServer which does not mix well with VirtualBox's volume sharing. If you use this approach you may have issues with the server not picking up changes to source files. The `grunt run-dev` approach is recommended for OS X users. 

This is the fastest way to develop with the HTML, CSS, and Javascript portion of UI for Docker.

Use the following Dockerfile
```
FROM scratch
COPY dist /app
EXPOSE 9000
WORKDIR /app
ENTRYPOINT ["ui-for-docker"]
```
And run with 
```
grunt if:binaryNotExist build shell:buildImage
docker run -d \
    -p 9000:9000 \
    -v $(pwd)/dist:/app \
    -v /var/run/docker.sock:/docker.sock \
    me/ui-for-docker \
    -e /docker.sock
grunt watch
```

## Run under Apache with mod_proxy

- clone it into `/var/www/ui-for-docker`
- enable `mod_proxy` and the http_proxy handler
- create a `virtualhost`
- enable `proxypass` to pipe `/dockerapi` request to your docker host

### VirtualHost

```
<VirtualHost *:80>
    ServerName docker.yourcompany
    DocumentRoot /var/www/ui-for-docker
    DirectoryIndex index.html
    ProxyPass /dockerapi/ http://127.0.0.01:4243/
    ProxyPassReverse /dockerapi/ http://127.0.0.1:4243/
</VirtualHost>
```

### mod_proxy

Make sure that you have `mod_proxy` enabled

```
$ ls -l /etc/apache2/mods-enabled/ |grep proxy
lrwxrwxrwx 1 root root 28 juil. 18 11:09 proxy.conf -> ../mods-available/proxy.conf
lrwxrwxrwx 1 root root 33 juil. 18 11:17 proxy_http.load -> ../mods-available/proxy_http.load
lrwxrwxrwx 1 root root 28 juil. 18 11:09 proxy.load -> ../mods-available/proxy.load
```

If not and if you're on Debian for instance : run `a2enmod proxy && a2enmod proxy_http`

### Docker host config

Make sure that it is listening on port 4243