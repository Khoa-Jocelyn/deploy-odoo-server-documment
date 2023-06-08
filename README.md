# Steps To Deploy Odoo Server With Ubuntu OS

Here are the steps to deploy the most basic Odoo Server system

## 1 - Setups SSH For Server

SSH is a standard protocol to control a remote machine over network. We use SSH to execute commands on servers also to transfer file to/from servers.

**OpenSSH is an open source software that implements SSH protocol on Linux. OpenSSH provides:**

- A SSHD (Secure Shell Daemon) service run on server, listens continuously for connections from client tools.
- A set of client tools (ssh, scp,...) to connect to server for executing commands, transfer files,...

*OpenSSH can use many authentication methods, including plain password or public key.*

### 1.1 - SSH Server

*Install SSHD service on server if not installed yet:*

```
sudo apt install openssh-server
```

*To start SSHD:*

```
sudo systemctl start sshd
```

*To stop SSHD:*

```
sudo systemctl stop sshd
```

*To restart SSHD:*

```
sudo systemctl restart sshd
```

*To view status of SSHD:*

```
sudo systemctl status sshd
```

**A - Configuration SSHD**

- To configure SSHD: before, edit file /etc/ssh/sshd_config after restart SSHD

*Some common configs:*

- Port: default to 22, change to listen client connections on another port.

### 1.2 - SSH Client

**B - Install ssh client tools if not installed**

```
sudo apt install openssh-client
```

*Some main tools:*

- ssh: login and execute commands on server
- scp: transfer file to/from server
- ssh-keygen: generate SSH keys

**C - Connect from client to server**

SSH allow authentication between two hosts without the need of a password. SSH key authentication uses a private key and a public key.

**Step 1: [Generate the ssh keys](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)**

*From a terminal prompt enter:*

```
ssh-keygen -t rsa
```
*Or for old version:*

```
ssh-keygen -t rsa -m PEM
```

*During the process you will be prompted for a passphrase. You can hit Enter when prompted to create keys without passphrase.*

*By default, the public key is saved in the file `~/.ssh/id_rsa.pub`, while `~/.ssh/id_rsa` is the private key. The public key needs to be deployed to the server where client want to connect to.*

**Step 2: Deploy the public key to server**

To deploy the public key to server, do one of two methods:

- Method 1: Manually append content of the `id_rsa.pub` file to the file `~/.ssh/authorized_keys` on server.
- Method 2: Execute command: `ssh-copy-id {username}@{server-ip} -i {path-to-public-key}`. This method requires a working ssh connection before (e.g. using password authentication).

## 2 - Setup SSH Key To Clone/Pull Code From Git Repositories.

**Step 1: Generate a new ssh key**
- Follow section [1.2/Connect from client to server/Step 1](generate the ssh keys)

**Step 2: Add ssh key to git**
- [Add ssh key to your git account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)

## 3 - Setup Servers

### 3.1 - Odoo Server

**Step 1: Install requirements for the backup/restore/kill process feature**

```
sudo apt install zip unzip pv psmisc
```

**Step 2: Create system user and group odoo**

```
sudo adduser --system --home=/opt/odoo --disabled-login --disabled-password --group odoo
```

*Replace /opt/odoo with the path where you intend to put Odoo and custom addons repositories.*

**Step 3: Install multiple Python versions to support multiple Odoo versions**

*Add the repository for Python packages:*

```
sudo add-apt-repository ppa:deadsnakes/ppa

sudo apt-get update
```

*Install the Python version which corresponding to the Odoo version:*

| Odoo Version | Python Version |
| :----------: | :------------: |
| <= 10.0 | 2.7 |
| 11.0, 12.0 | 3.6, 3.7 |
| 13.0 | 3.7 |
| 14.0, 15.0 | 3.8 |
| 15.0, 16.0 | 3.9 |

**Step 4: Setup environment for each Odoo version, example for Odoo 14.0**

*[Install system package requirements](https://www.odoo.com/documentation/14.0/administration/install/install.html#id14):*

```
sudo apt install python3-dev libxml2-dev libxslt1-dev libldap2-dev libsasl2-dev libtiff5-dev libjpeg8-dev libopenjp2-7-dev zlib1g-dev libfreetype6-dev liblcms2-dev libwebp-dev libharfbuzz-dev libfribidi-dev libxcb1-dev libpq-dev
```

*Create Python virtual environment:*

```
python3.8 -m venv /opt/python3.8-venv/odoo14
```

*Enter the above environment to install python package requirements:*

```
source /opt/python3.8-venv/odoo14/bin/activate

pip install -r https://raw.githubusercontent.com/Viindoo/odoo/14.0/requirements.txt
```

*Exit the virtual environment:*

```
deactivate
```

**Step 5: Config System Service**

*Create odoo14.service file:*

```
sudo nano /etc/systemd/system/odoo14.service
```

*Copy content to nano windown:*

```
[Unit]
Description=Odoo14
After=network.target

[Service]
Type=simple
SyslogIdentifier=odoo14
PermissionsStartOnly=true
User=odoo
Group=odoo
ExecStart=/opt/python3.8-venv/odoo14/bin/python /opt/odoo/odoo14/odoo-bin -c /opt/odoo/odoo14.conf
StandardOutput=journal+console

[Install]
WantedBy=multi-user.target
```

*Enter command `Ctrl + S` to save -> command `Ctrl + X` to close nano.*

*Create odoo14.conf file:*

```
sudo nano /opt/odoo/odoo14.conf
```

*Copy content to nano windown:*

```
[options]
addons_path = /opt/odoo/odoo14/addons
admin_passwd = mysupersecretpassword
db_host = localhost
db_port = 5432
db_user = odoo
db_password = pwd
# dbfilter = ^mycompany.*$
limit_memory_hard = 1677721600
limit_memory_soft = 629145600
limit_request = 8192
limit_time_cpu = 600
limit_time_real = 1200
max_cron_threads = 1
workers = 8
proxy_mode = True
logfile = /var/log/odoo/odoo.log
```
*Enter command `Ctrl + S` to save -> command `Ctrl + X` to close nano.*

*Reload systemd config:*

```
systemctl daemon-reload
```

*Enable and start running Odoo service:*

```
systemctl enable --now odoo14
```

### 3.2 - PostgreSQL Server

**Step 1: Install PostgreSQL on the physical server used for database**

*Ubuntu includes PostgreSQL by default. You can install PostgreSQL by the following command:*

```
sudo apt install postgresql postgresql-contrib
```

*Edit postgresql.conf to allow remote connection from other servers:*

```
sudo nano /etc/postgresql/your_postgresql_version/main/postgresql.conf
```
*Note:* Replace `your_postgresql_version` with your postgresql version. In the config, set `listen_addresses = '*'`. On production environment, you may need to configure other things. Refer to (https://pgtune.leopard.in.ua) to configure postgresql for better performance.


*Restart postgresql to apply the new configuration:*

```
sudo systemctl restart postgresql
```

*Enter postgresql interactive shell as postgres user:*

```
sudo -u postgres psql
```

*Create a new database user with password (e.g. master) that uses to administer the database:*

```
CREATE USER master WITH LOGIN SUPERUSER CREATEDB CREATEROLE INHERIT REPLICATION CONNECTION LIMIT -1;
ALTER USER postgres PASSWORD 'pwd';
```

*Install `unaccent` extension to handle Vietnamese characters when do searching on database:*

```
CREATE EXTENSION IF NOT EXISTS unaccent;
```

*Exit the shell:*

```
quit
```

*Edit `pg_hba.conf` to configure client authentication:*

```
sudo nano /etc/postgresql/your_postgresql_version/main/pg_hba.conf
```

*Add the following rules or see [here](https://www.postgresql.org/docs/12/auth-pg-hba-conf.html) for details:*

```
# Allow master user on localhost to connect to any database.
host    all    master    127.0.0.1/32    md5

# Allow any user from Odoo server to connect to any database but require password.
# Replace x.x.x.x with the IP address of the Odoo server.
host    all    all    x.x.x.x/32    md5
```

*Reload postgresql to apply new pg_hba.conf configuration:*

```
sudo systemctl reload postgresql
```

*Install some system packages required to zip file:*

```
sudo apt install zip unzip gzip pv
```

### 3.3 - Nginx Server

**Step 1: Install nginx on the physical server used for proxy**

*Install Nginx:*
```
sudo apt install nginx -y
```

**Step 2: Create a new user to run nginx service**

```
sudo adduser --system --no-create-home --group nginx
```

**Step 3: Create the following two directories if they don't exist**

```
sudo mkdir -p /var/log/nginx && sudo chown nginx:nginx /var/log/nginx

sudo mkdir -p /var/lib/nginx && sudo chown nginx:nginx /var/lib/nginx
```

**Step 4: Create a new nginx service to manage via `systemctl` commands**

```
sudo nano /lib/systemd/system/nginx.service
```

*Copy content to nano windown:*

```
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t -q -g 'daemon on; master_process on;'
ExecStart=/usr/local/nginx/sbin/nginx -g 'daemon on; master_process on;'
ExecReload=/usr/local/nginx/sbin/nginx -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target
```
*Enter command `Ctrl + S` to save -> command `Ctrl + X` to close nano.*

*Start the nginx service:*

```
sudo systemctl start nginx
```

*Make the nginx service run automatically on startup:*

```
sudo systemctl enable nginx
```

**Step 5: Create nginx config file**


```
sudo rm -rf /etc/nginx/sites-available/default

sudo nano /etc/nginx/sites-enabled/nginx-odoo.conf
```

*Copy content to nano windown:*

```
# Odoo server
upstream odoo {
  server 127.0.0.1:8069;
}
upstream odoochat {
  server 127.0.0.1:8072;
}
map $http_upgrade $connection_upgrade {
  default upgrade;
  ''      close;
}

# http -> https
server {
  listen 80;
  server_name odoo.mycompany.com; # Replace odoo.mycompany.com with your server name
  rewrite ^(.*) https://$host$1 permanent;
}

server {
  listen 443 ssl;
  server_name odoo.mycompany.com;
  proxy_read_timeout 720s;
  proxy_connect_timeout 720s;
  proxy_send_timeout 720s;

  # SSL parameters
  ssl_certificate /etc/ssl/nginx/server.crt;
  ssl_certificate_key /etc/ssl/nginx/server.key;
  ssl_session_timeout 30m;
  ssl_protocols TLSv1.2;
  ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
  ssl_prefer_server_ciphers off;

  # Log
  access_log /var/log/nginx/odoo.access.log;
  error_log /var/log/nginx/odoo.error.log;

  # Redirect websocket requests to odoo gevent port
  location /websocket {
    proxy_pass http://odoochat;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;
  }

  # Redirect requests to odoo backend server
  location / {
    # Add Headers for odoo proxy mode
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_redirect off;
    proxy_pass http://odoo;
  }

  # common gzip
  gzip_types text/css text/scss text/plain text/xml application/xml application/json application/javascript;
  gzip on;
}
```

*Copy nginx.conf from `/etc/nginx/sites-available/nginx.conf` to `/etc/nginx/sites-enabled/nginx.conf`:*

```
sudo ln -s /etc/nginx/sites-available/odoo.conf /etc/nginx/sites-enabled/odoo.conf
```

*Test nginx config:*

```
sudo nginx -t
```

*Note: If the terminal displays the following message, then it is successful*

```
nginx: the configuration file /etc/nginx/nginx-odoo.conf syntax is ok
nginx: configuration file /etc/nginx/nginx-odoo.conf test is successful
```

*Restart nginx sevice:*
 
```
sudo systemctl start nginx
```

### 3.4 - DNS Server

**Step 1: Install bind9 on the physical server used for DNS**

```
sudo apt install bind9
```

On local environment, to access Odoo instances via its domain name, you need to setup DNS nameservers in your host OS (not guest OS in virtual machines):

*Install the resolvconf package if it's not already installed:*

```
sudo apt install resolvconf
```

*Open the file `/etc/resolvconf/resolv.conf.d/head` and insert the following lines:*

```
# Replace x.x.x.x with the IP address of the DNS server.
nameserver x.x.x.x
```

*Restart resolvconf and systemd-resolved services:*

```
sudo systemctl restart resolvconf && sudo systemctl restart systemd-resolved
```

*To verify if the above setup is successful, open the file `/etc/resolv.conf` and check if there is nameserver x.x.x.x in there.*

### 3.5 - SSL Config

*Reference:*

- https://cipherlist.eu/
- https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal
