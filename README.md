# Steps To Deploy Odoo Server

Here are the steps to deploy the most basic Odoo Server system

## 1, Setups SSH For Server

SSH is a standard protocol to control a remote machine over network. We use SSH to execute commands on servers also to transfer file to/from servers.

OpenSSH is an open source software that implements SSH protocol on Linux. OpenSSH provides:

- A SSHD (Secure Shell Daemon) service run on server, listens continuously for connections from client tools.
- A set of client tools (ssh, scp,...) to connect to server for executing commands, transfer files,...
OpenSSH can use many authentication methods, including plain password or public key.

### 1.1, SSH In Server
**Install SSHD service on server if not installed yet**

```
sudo apt install openssh-server
```

**To start SSHD**

```
sudo systemctl start sshd
```

**To stop SSHD**

```
sudo systemctl stop sshd
```

**To restart SSHD**

```
sudo systemctl restart sshd
```

**To view status of SSHD**

`sudo systemctl status sshd`

**Configuration SSHD**

*To configure SSHD*
- Step 1: Edit file /etc/ssh/sshd_config
- Step 2: Restart SSHD `sudo systemctl restart sshd`

*Some common configs:*
- Port: default to 22, change to listen client connections on another port.


