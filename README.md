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

### 3.2 PostgreSQL Server

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

### Nginx Server

**Step 1: Install nginx on the physical server used for proxy**

*Because we need PageSpeed module, we need to install nginx from source:*

```
sudo apt install build-essential libssl-dev libpcre3 libpcre3-dev libxml2-dev libxslt1-dev libgd-dev libgeoip-dev libperl-dev curl
```

*Install nginx:*

```
bash <(curl -f -L -sS https://ngxpagespeed.com/install) --nginx-version latest
```

*When asked, input the following params:*

```
--prefix=/usr/local/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-log-path=/var/log/nginx/access.log --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --user=nginx --group=nginx --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_addition_module --with-http_dav_module --with-http_geoip_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_image_filter_module --with-http_v2_module --with-http_sub_module --with-http_xslt_module --with-stream --with-stream_ssl_module --with-file-aio --with-mail --with-mail_ssl_module --with-threads
```

**Warning**
*If you encounter error when building nginx on Ubuntu 20.04 or above, try to install Nginx and Pagespeed manually by following these instructions:*

> *First, download the Nginx source code and the PSOL pre-built library:*
>
> ```
> wget https://nginx.org/download/nginx-1.22.0.tar.gz
> ```
>
> (or you can replace 1.22.0 by the newest version)
>
> ```
> wget http://www.tiredofit.nl/psol-focal.tar.xz
> ```
>
> (or you can replace focal with the code of your ubuntu version, eg: jammy for Ubuntu 22.04...)
>
> *Extract the downloaded files:*
>
> ```
> tar xvf psol-focal.tar.xz
>
> tar zxvf nginx-1.22.0.tar.gz
> ```
>
> *Clone the source code of Pagespeed module:*
>
> ```
> git clone --depth=1 https://github.com/apache/incubator-pagespeed-ngx.git
> ```
>
> *Move the extracted psol into Pagespeed directory:*
>
> ```
> mv psol incubator-pagespeed-ngx
> ```
>
> *Finally, configure and install Nginx:*
>
>```
> cd nginx-1.22.0
>
> ./configure --prefix=/usr/local/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=var/lib/nginx/fastcgi --http-log-path=/var/log/nginx/access.log --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --lock path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --user=nginx --group=nginx --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_addition_module --with-http_dav_module --with-http_geoip_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_image_filter_module --with-http_v2_module --with-http_sub_module --with-http_xslt_module --with-stream --with-stream_ssl_module --with-file-aio --with-mail --with-mail_ssl_module --with-threads --add-module=../incubator-pagespeed-ngx
>
> make -j4
>
> make install
> ```

*Create a new user to run nginx service:*

```
sudo adduser --system --no-create-home --group nginx
```

*Create the following two directories if they don't exist:*

```
sudo mkdir -p /var/log/nginx && sudo chown nginx:nginx /var/log/nginx

sudo mkdir -p /var/lib/nginx && sudo chown nginx:nginx /var/lib/nginx
```

*Create a new nginx service to manage via `systemctl` commands:*

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

