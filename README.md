# Steps To Deploy Odoo Server With Ubuntu OS

Here are the steps to deploy the most basic Odoo Server system

## 1 - Setups SSH For Server

SSH is a standard protocol to control a remote machine over network. We use SSH to execute commands on servers also to transfer file to/from servers.

**OpenSSH is an open source software that implements SSH protocol on Linux. OpenSSH provides:**

- A SSHD (Secure Shell Daemon) service run on server, listens continuously for connections from client tools.
- A set of client tools (ssh, scp,...) to connect to server for executing commands, transfer files,...

*OpenSSH can use many authentication methods, including plain password or public key.*

### 1.1 - SSH Server

*Install SSHD service on server if not installed yet*

```
sudo apt install openssh-server
```

*To start SSHD*

```
sudo systemctl start sshd
```

*To stop SSHD*

```
sudo systemctl stop sshd
```

*To restart SSHD*

```
sudo systemctl restart sshd
```

*To view status of SSHD*

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

### 3.1 - Setup Odoo Server

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

*[Install system package requirements](https://www.odoo.com/documentation/14.0/administration/install/install.html#id14)*

```
sudo apt install python3-dev libxml2-dev libxslt1-dev libldap2-dev libsasl2-dev libtiff5-dev libjpeg8-dev libopenjp2-7-dev zlib1g-dev libfreetype6-dev liblcms2-dev libwebp-dev libharfbuzz-dev libfribidi-dev libxcb1-dev libpq-dev
```

*Create Python virtual environment*

```
python3.8 -m venv /opt/python3.8-venv/odoo14
```

*Enter the above environment to install python package requirements*

```
source /opt/python3.8-venv/odoo14/bin/activate

pip install -r https://raw.githubusercontent.com/Viindoo/odoo/14.0/requirements.txt
```

*Exit the virtual environment*

```
deactivate
```

**Step 5: Config System Service**

*Create odoo14.service file*

```
sudo nano /etc/systemd/system/odoo14.service
```

*Copy content to nano windown*

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
*Create odoo14.conf file*

```
sudo nano /opt/odoo/odoo14.conf
```

*Copy content to nano windown*

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
*Reload systemd config*

```
systemctl daemon-reload
```

*Enable and start running Odoo service*

```
systemctl enable --now odoo14
```



