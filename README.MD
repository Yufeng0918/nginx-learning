# Nginx

## Nginx overview
- reserve proxy
- load balancer
- static/dynamic isolation
#### Architecture
- one master process
- multiple worker processes, independent process
- one worker serve multiple client
#### Optimzation
- set worker_process same as CPU
- worker_connection
    + static resource, 2 connection
    + dynamic call, 4 connection
    + max concurrent connection
        + static: worker_process * worker_connection / 2
        + dynamic: worker_process * worker_connection / 4



## Installation
- ./configure
- make && make install
- /usr/local/nginx
#### Config
- /usr/local/nginx/nginx.conf
- session
    + global session, define global settings
        + worker_process: allow number of processes to handle connection
    + event session
        + worker_connections: concurrent connection
    + http session
        + http global
        + server
            + global server
                + listen: port
                + server_name: binding server name
            + location
#### Command
- go to /usr/local/nginx/sbin
- start:    ./nginx
- stop:     ./nginx -s stop
- reload:   ./nginx -s reload



## Reserve Proxy
- listen to 127.0.0.1
- proxy_pass redirect to 127.0.0.1:9090
- "~" is regular expression and case sensitive
- "~*" is regular expression and case insensitive
```
server {
    listen       80;
    server_name  127.0.0.1;

    location / {
        root   html;
        proxy_pass http://127.0.0.1:9090;
        index  index.html index.htm;
    }
}

server {
    listen       9001;
    server_name  127.0.0.1;
    location ~ /edu/ {
        proxy_pass http://127.0.0.1:9091;
    }
    
    location ~ /vod/ {
        proxy_pass http://127.0.0.1:9090;
    }
}
```


## Load Balancer
- define the cluster in upstream
- proxy_pass point to defined cluster
- load balance policy
    + run-robin: default
    + weight: weight=<n> after server, e.g. server 127.0.0.1:9090 weight=10;
    + ip_hash: add ip_hash
    + fair: base on server response time, add fair in end
```
upstream myserver {
   ip_hash;
   server 127.0.0.1:9090;
   server 127.0.0.1:9091;
}

server {
    listen       80;
    server_name  127.0.0.1;

    #charset koi8-r;

    #access_log  logs/host.access.log  main;

    location / {
        root   html;
        proxy_pass http://myserver;
        index  index.html index.htm;
}
```



## Dynamic Seperation
- location base on suffix
- expire define the expire time, nginx will compare the file modification time
```
location /www/ {
    root /data/; 
    index  index.html index.htm;
}

### redirect to image
location /image/ {
    root /data/;
    autoindex  on;
}
```


## High Avaliable
- virtual ip binding to nginx cluster
- both nginx are keepalived
