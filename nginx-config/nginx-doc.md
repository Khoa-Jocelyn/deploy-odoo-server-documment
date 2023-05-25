### Step 1: Install Nginx
```sudo apt install nginx -y```

### Step 2: Create nginx config file
```cd /etc/nginx/sites-available/```

```sudo rm -rf default```

```sudo nano nginx.conf```

```
############################
# Odoo server
upstream odoo {
server 127.0.0.1:8069;
}
upstream odoochat {
server 127.0.0.1:8072;
}

server {
listen 80;
server_name jocelyn.com;

proxy_read_timeout 720s;
proxy_connect_timeout 720s;
proxy_send_timeout 720s;

# Add headers for odoo proxy mode
proxy_set_header X-Forwarded-Host $host;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header X-Real-IP $remote_addr;

# Log
access_log /var/log/nginx/odoo.access.log;
error_log /var/log/nginx/odoo.error.log;

# Redirect request to odoo backend server
location / {
proxy_redirect off;
proxy_pass http://odoo;
}
location /longpolling {
proxy_pass http://odoochat;
}

# Common gzip
gzip_types text/css text/less text/plain text/xml application/xml application/javascript image/svg+xml;
gzip on;
gzip_vary on;
gzip_min_length 1000;
gzip_proxied any;
gzip_comp_level 6;

client_body_in_file_only clean;
client_body_buffer_size 32K;
client_max_body_size 500M;
sendfile on;
send_timeout 600s;
keepalive_timeout 300;
}
############################
```

### Step 3: Copy nginx.conf from /etc/nginx/sites-available/nginx.conf to /etc/nginx/sites-enabled/nginx.conf

```sudo ln -s /etc/nginx/sites-available/odoo.conf /etc/nginx/sites-enabled/odoo.conf``` 

### Step 4: Test nginx config
```cd /etc/nginx/sites-enabled/```
```sudo nginx -t```

*Note*: If the terminal displays the following message, then it is successful
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### Step 5: Restart Nginx sevice
```sudo service nginx stop```
```sudo service nginx start```




