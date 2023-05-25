### Step 1: Install Nginx
```
sudo apt install nginx -y
```

### Step 2: Create nginx config file
```
cd /etc/nginx/sites-available/

sudo rm -rf default

sudo nano nginx.conf
```
Copy content in [nginx.conf]() file and enter command `Ctrl + S` to save file -> command `Ctrl + X` to close nano.


### Step 3: Copy nginx.conf from /etc/nginx/sites-available/nginx.conf to /etc/nginx/sites-enabled/nginx.conf

```
sudo ln -s /etc/nginx/sites-available/odoo.conf /etc/nginx/sites-enabled/odoo.conf
```

### Step 4: Test nginx config
```
cd /etc/nginx/sites-enabled/

sudo nginx -t
```

*Note*: If the terminal displays the following message, then it is successful
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### Step 5: Restart Nginx sevice
```
sudo service nginx restart
```
