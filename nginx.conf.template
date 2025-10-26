user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" upstream=$upstream_addr';

    access_log /var/log/nginx/access.log main;

    sendfile on;
    keepalive_timeout 65;

    upstream backend {
        server app_blue:${PORT} max_fails=1 fail_timeout=5s;
        server app_green:${PORT} backup;
    }

    server {
        listen 80;
        server_name localhost;

        location / {
            proxy_pass http://backend;
            
            # Timeouts - aggressive for quick failover
            proxy_connect_timeout 2s;
            proxy_send_timeout 3s;
            proxy_read_timeout 3s;
            
            # Retry configuration
            proxy_next_upstream error timeout http_500 http_502 http_503 http_504;
            proxy_next_upstream_tries 2;
            proxy_next_upstream_timeout 10s;
            
            # Forward headers from upstream (preserve app headers)
            proxy_pass_request_headers on;
            
            # Standard proxy headers
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # Disable buffering for real-time responses
            proxy_buffering off;
            
            # HTTP version
            proxy_http_version 1.1;
            proxy_set_header Connection "";
        }
    }
}