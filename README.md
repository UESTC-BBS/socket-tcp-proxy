# socket-tcp-proxy

This repo was origin created for docker(tcp port)<------->nginx(unix socket) content transportation.

It works as :

nginx proxy -> use unix socket as upstream -> socket-tcp-proxy -> mapped to docker container.

## To solve:
Container ip changes after docker restart.
We do not want to spend time solving it.
However, if you can get it done, the socker-tcp-proxy is useless for you.

## Principle

Start with pre-defined json configuration `/etc/socket-proxy/proxy.json`.

The json configuration defined socket path, docker name and docker port,

The socket-tcp-proxy would get the docker ip automatically and generate the transportation.

the proxy.json example :
```
{
    "logfile":"/var/log/socket-proxy/proxy.log",
    "proxy":[
        {
            "socket":"/home/data/socket/gitlab.sock",
            "docker":"gitlab",
            "port":"80"
        }
    ]
}
```

the nginx configuration example :

```
upstream gitlab_upstream{
    server unix:/home/data/socket/gitlab.sock;
}

server {
    listen      80;
    server_name git.starstudio.org;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $http_x_real_ip;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://gitlab_upstream;
    }
    include conf.d/whiteips.conf;

    access_log /var/log/nginx/gitlab_access.log;
    error_log /var/log/nginx/gitlab_error.log;
}
```

