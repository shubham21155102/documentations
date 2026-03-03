# 🌐 Nginx

> Frequently used Nginx commands and configuration patterns.

[← Back to Home](../README.md)

---

## 📋 Table of Contents

- [Installation](#installation)
- [Service Management](#service-management)
- [Configuration Structure](#configuration-structure)
- [Common Directives](#common-directives)
- [Virtual Hosts (Server Blocks)](#virtual-hosts-server-blocks)
- [Reverse Proxy](#reverse-proxy)
- [Load Balancing](#load-balancing)
- [SSL/TLS with Let's Encrypt](#ssltls-with-lets-encrypt)
- [Security Headers](#security-headers)
- [Performance Tuning](#performance-tuning)
- [Logging](#logging)

---

## Installation

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install -y nginx

# CentOS/RHEL
sudo yum install -y nginx

# Start and enable
sudo systemctl start nginx
sudo systemctl enable nginx

# Verify
nginx -v
curl http://localhost
```

---

## Service Management

```bash
# Start / stop / restart
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx

# Reload config (no downtime)
sudo systemctl reload nginx
sudo nginx -s reload

# Test configuration
sudo nginx -t
sudo nginx -T              # test and dump full config

# Status
sudo systemctl status nginx

# View logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

---

## Configuration Structure

```
/etc/nginx/
├── nginx.conf              # Main config
├── conf.d/                 # Additional config files
│   └── default.conf
├── sites-available/        # Virtual host definitions
│   └── mysite.conf
├── sites-enabled/          # Symlinks to enabled sites
│   └── mysite.conf -> ../sites-available/mysite.conf
├── snippets/               # Reusable config snippets
│   ├── fastcgi-php.conf
│   └── snakeoil.conf
└── mime.types
```

**Enable a site:**

```bash
sudo ln -s /etc/nginx/sites-available/mysite.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## Common Directives

```nginx
# nginx.conf — main config
user www-data;
worker_processes auto;          # match number of CPUs
pid /run/nginx.pid;

events {
    worker_connections 1024;    # max connections per worker
    use epoll;
    multi_accept on;
}

http {
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    server_tokens off;          # hide nginx version

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_comp_level 6;
    gzip_types text/plain text/css application/json application/javascript;

    # Logging
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

---

## Virtual Hosts (Server Blocks)

**Static website:**

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;

    root /var/www/example.com;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }
}
```

**PHP website:**

```nginx
server {
    listen 80;
    server_name example.com;

    root /var/www/example.com;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

---

## Reverse Proxy

**Simple reverse proxy:**

```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**WebSocket support:**

```nginx
location /ws {
    proxy_pass http://localhost:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_set_header Host $host;
    proxy_read_timeout 86400;
}
```

---

## Load Balancing

```nginx
upstream backend {
    # Round-robin (default)
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;

    # Least connections
    # least_conn;

    # IP hash (sticky sessions)
    # ip_hash;

    # Weighted
    # server 192.168.1.10:8080 weight=3;
    # server 192.168.1.11:8080 weight=1;

    # Health check
    # server 192.168.1.10:8080 max_fails=3 fail_timeout=30s;
    keepalive 32;
}

server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## SSL/TLS with Let's Encrypt

```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d example.com -d www.example.com

# Certificate only (manual config)
sudo certbot certonly --webroot -w /var/www/example.com -d example.com

# Renew certificates
sudo certbot renew
sudo certbot renew --dry-run     # test renewal

# Auto-renew (crontab)
echo "0 12 * * * root certbot renew --quiet" | sudo tee /etc/cron.d/certbot
```

**SSL server block:**

```nginx
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name example.com www.example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    root /var/www/example.com;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}
```

---

## Security Headers

```nginx
# Add to server or http block
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "no-referrer-when-downgrade" always;
add_header Content-Security-Policy "default-src 'self' https: data: 'unsafe-inline'" always;
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
add_header Permissions-Policy "camera=(), microphone=(), geolocation=()" always;

# Block common attacks
location ~* \.(asp|aspx|jsp|cgi)$ {
    return 444;
}

# Limit request methods
if ($request_method !~ ^(GET|HEAD|POST)$) {
    return 444;
}

# Deny dotfiles
location ~ /\. {
    deny all;
    access_log off;
    log_not_found off;
}

# Rate limiting
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
location /api/ {
    limit_req zone=api burst=20 nodelay;
    proxy_pass http://backend;
}
```

---

## Performance Tuning

```nginx
# http block tuning
http {
    # File caching
    open_file_cache max=1000 inactive=20s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 2;
    open_file_cache_errors on;

    # Buffer sizes
    client_body_buffer_size 10K;
    client_header_buffer_size 1k;
    client_max_body_size 8m;
    large_client_header_buffers 2 1k;

    # Timeouts
    client_body_timeout 12;
    client_header_timeout 12;
    keepalive_timeout 15;
    send_timeout 10;

    # Proxy buffers
    proxy_buffer_size 128k;
    proxy_buffers 4 256k;
    proxy_busy_buffers_size 256k;
}
```

---

## Logging

```nginx
# Custom log format
log_format main '$remote_addr - $remote_user [$time_local] '
                '"$request" $status $body_bytes_sent '
                '"$http_referer" "$http_user_agent"';

log_format json_combined '{'
    '"time":"$time_local",'
    '"remote_addr":"$remote_addr",'
    '"method":"$request_method",'
    '"uri":"$uri",'
    '"status":$status,'
    '"body_bytes_sent":$body_bytes_sent,'
    '"request_time":$request_time,'
    '"upstream_response_time":"$upstream_response_time"'
    '}';

access_log /var/log/nginx/access.log json_combined;
error_log /var/log/nginx/error.log warn;

# Disable logging for specific paths
location = /health {
    access_log off;
    return 200 "OK\n";
}
```

---

[← Back to Home](../README.md)
