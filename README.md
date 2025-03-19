# Docker Project Commands:

## "Dockerfile"
build image:
>docker build -t catorce-gigantes:1.0 .
>docker images | grep catorce-gigantes

start/stop container:
>docker run -p 3000:3000 catorce-gigantes:1.0
>docker ps
>docker stop "ID"

## "docker-compose.yaml"
build and start 3 containers instances with docker-compose:
>docker-compose up --build -d
stop all containers instances:
>docker-compose down
delete all unused or stopped images, containers, networks, etc:
>docker system prune -a

# NGINX Basic Commands:

>brew install nginx / apt update && apt install nginx
>nginx -v  /  nginx -V

>whereis nginx  -> --conf-path=/opt/homebew/etc/nginx/nginx.conf
>vim /opt/homebew/etc/nginx/nginx.conf

start nginx:
>nginx
get options:
>nginx -h
restart nginx:
>nginx -s reload
stop nginx:
>nginx -s stop

print logs:
>tail -f /usr/local/var/log/nginx/access.log
nginx processes:
>ps aux | grep nginx

# Generate self-signed SSL certificate:

create folder for nginx certificates:
>mkdir ~/nginx-certs
>cd ~/nginx-certs

create self-signed certificate:
>openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginx-selfsigned.key -out nginx-selfsigned.crt

# NGINX Configuration File for http and https access (nginx.conf):


# ---- > Basic NGINX configuration file content <------

## Main context (this is the global configuration)
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    include mime.types;

    # Upstream block to define the Node.js backend servers
    upstream nodejs_cluster {
        server 127.0.0.1:3001;
        server 127.0.0.1:3002;
        server 127.0.0.1:3003;
    }

    server {
        listen 8080;  # Listen on port 80 for HTTP
        server_name localhost;

        # Proxying requests to Node.js cluster
        location / {
            proxy_pass http://nodejs_cluster;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}


# ---- > Complete NGINX SSL configuration file content <------

## Main context (this is the global configuration)
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    include mime.types;

    # Upstream block to define the Node.js backend servers
    upstream nodejs_cluster {
        server 127.0.0.1:3001;
        server 127.0.0.1:3002;
        server 127.0.0.1:3003;
    }

    server {
        listen 443 ssl;  # Listen on port 443 for HTTPS
        server_name localhost;

        # SSL certificate settings
        ssl_certificate /Users/pc-user/nginx-certs/nginx-selfsigned.crt;
        ssl_certificate_key /Users/pc-user/nginx-certs/nginx-selfsigned.key;

        # Proxying requests to Node.js cluster
        location / {
            proxy_pass http://nodejs_cluster;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }

    # Optional server block for HTTP to HTTPS redirection
    server {
        listen 8080;  # Listen on port 80 for HTTP
        server_name localhost;

        # Redirect all HTTP traffic to HTTPS
        location / {
            return 301 https://$host$request_uri;
        }
    }
}

restart nginx: 
>nginx -s reload

->> https://localhost:443