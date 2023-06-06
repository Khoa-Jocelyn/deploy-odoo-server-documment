# Steps To Deploy Odoo Server

Here are the steps to deploy the most basic Odoo Server system

## 1 Setups SSH For Server

SSH is a standard protocol to control a remote machine over network. We use SSH to execute commands on servers also to transfer file to/from servers.

OpenSSH is an open source software that implements SSH protocol on Linux. OpenSSH provides:

- A SSHD (Secure Shell Daemon) service run on server, listens continuously for connections from client tools.
- A set of client tools (ssh, scp,...) to connect to server for executing commands, transfer files,...
OpenSSH can use many authentication methods, including plain password or public key.

### 1.1 SSH Server

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

```
sudo systemctl status sshd
```

**Configuration SSHD**

*To configure SSHD*
- Step 1: Edit file /etc/ssh/sshd_config
- Step 2: Restart SSHD `sudo systemctl restart sshd`

*Some common configs:*
- Port: default to 22, change to listen client connections on another port.

### 1.2 SSH Client

**Install ssh client tools if not installed**

```
sudo apt install openssh-client
```

*Some main tools:*

- ssh: login and execute commands on server
- scp: transfer file to/from server
- ssh-keygen: generate SSH keys

**Connect from client to server**

SSH allow authentication between two hosts without the need of a password. SSH key authentication uses a private key and a public key.

*Step 1: generate the ssh keys*

From a terminal prompt enter:

```
ssh-keygen -t rsa
```
Or for old version:

```
ssh-keygen -t rsa -m PEM
```

During the process you will be prompted for a passphrase. You can hit Enter when prompted to create keys without passphrase.

By default, the public key is saved in the file ~/.ssh/id_rsa.pub, while ~/.ssh/id_rsa is the private key. The public key needs to be deployed to the server where client want to connect to.

*Step 2: deploy the public key to server*

To deploy the public key to server, do one of two methods:

- Method 1: Manually append content of the id_rsa.pub file to the file ~/.ssh/authorized_keys on server.
- Method 2: Execute command: ssh-copy-id {username}@{server-ip} -i {path-to-public-key}. This method requires a working ssh connection before (e.g. using password authentication).

