First add the upstream conf to your nginx configuration :
```
upstream dockerui {
    server 172.17.42.1:9000;
}
```

```
TCP and UNIX-domain sockets can be mixed.

Example:

upstream something {
    server unix:/tmp/somethinginthesocket;
}
```

Then here the sample vhost configuration : 

```
server {
        listen 80;
        server_name dockerui.example.com;

        access_log /var/log/nginx/dockerui.access.log main;
        error_log /var/log/nginx/dockerui.error.log info;

        location / {
 

                #basic auth
                auth_basic            "Docker UI";
                auth_basic_user_file  /somewhere.htpasswd;

                proxy_pass http://dockerui;
                #http 1.1 support...
                proxy_http_version 1.1;
                proxy_set_header Connection "";

                proxy_redirect     off;
                proxy_set_header   Host $host;
                proxy_set_header   X-Real-IP $remote_addr;
                proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header   X-Forwarded-Host $server_name;

        }
        root /home/johndoe/www/;
}

```

Note the explicit http 1.1 support. By default nginx use http 1.0...
Without this configuration, you get HTTP 500 errors.

This isn't in the documentation.