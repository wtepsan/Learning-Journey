# ENUCC Login Guide

## 1. Overview

This guide explains how to connect to the Edinburgh Napier University Compute Cluster, ENUCC.

There are two common ways to connect to ENUCC:

1. SSH Terminal
2. VS Code Remote SSH

If you are outside the university network, you must connect through the Napier SSH gateway first. The gateway acts as an entry point before connecting to the ENUCC login node.

---

## 2. Before You Start

Before logging in, make sure you have:

1. An ENUCC account
2. Your ENU username, for example `40000000`
3. Your ENU account password
4. A terminal or SSH client

Your ENUCC username is usually the same as your university username.

Your password is the same as your ENU Active Directory / university account password.

Recommended SSH clients:

```text
macOS: Terminal or iTerm2
Windows: PowerShell, Windows Terminal, or MobaXterm
Linux: Terminal
```

Important: ENUCC does not provide backups. You should back up your important files yourself.

---

## 3. Login Using SSH Terminal

### 3.1 Login from Inside the University Network

If you are already inside the university network, you may be able to connect directly to the ENUCC login node:

```bash
ssh <username>@login.enucc.napier.ac.uk
```

Replace:

```text
<username>
```

with your ENUCC username.

Example:

```bash
ssh 40000000@login.enucc.napier.ac.uk
```

---

### 3.2 Login from Outside the University Network

If you are outside the university network, connect through the Napier gateway using `ProxyJump`:

```bash
ssh -J <username>@gateway.napier.ac.uk <username>@login.enucc.napier.ac.uk
```

Replace:

```text
<username>
```

with your ENUCC username.

Example:

```bash
ssh -J 40000000@gateway.napier.ac.uk 40000000@login.enucc.napier.ac.uk
```

You may need to enter your ENU password more than once:

```text
1. Password for gateway.napier.ac.uk
2. Password for login.enucc.napier.ac.uk
```

This is normal because SSH first connects to the gateway and then connects to the ENUCC login node.

When login is successful, your terminal prompt should look similar to:

```text
[40000000@login1[enucc] ~]$
```

This means you are now on the ENUCC login node.

---

## 4. Why the Gateway Is Needed

The ENUCC login node is not directly accessible from outside the university network.

Therefore, when you are off campus, the connection path is:

```text
Your computer
   ↓
gateway.napier.ac.uk
   ↓
login.enucc.napier.ac.uk
```

The gateway is only an access point. You should still do your main ENUCC work after reaching the login node.

Sometimes, after logging in to the gateway, you may see an internal hostname such as:

```text
gway-app-liv-03
```

This is normal. You should still use this hostname in your SSH command and SSH config:

```text
gateway.napier.ac.uk
```

---

## 5. Login Using VS Code Remote SSH

You can also connect to ENUCC using VS Code.

### Step 1: Install VS Code

Download and install VS Code:

```text
https://code.visualstudio.com
```

### Step 2: Install the Remote SSH Extension

In VS Code, install this extension:

```text
Remote - SSH
```

### Step 3: Open the Command Palette

For Mac:

```text
Cmd + Shift + P
```

For Windows:

```text
Ctrl + Shift + P
```

Search for:

```text
Remote-SSH: Connect to Host
```

Then choose:

```text
Add New SSH Host
```

### Step 4: Add the SSH Command

For off-campus access, enter:

```bash
ssh -J 40000000@gateway.napier.ac.uk 40000000@login.enucc.napier.ac.uk
```

Replace `40000000` with your own ENUCC username.

VS Code may ask for your ENU password more than once because it first connects to the gateway and then connects to the ENUCC login node.

---

## 6. Set Up SSH Config

Instead of typing the full SSH command every time, you can save the connection in your SSH config file.

Open this file on your local computer:

```bash
~/.ssh/config
```

If the file does not exist, create it.

Add the following configuration:

```sshconfig
Host ext.login.enucc
  HostName login.enucc.napier.ac.uk
  User <username>
  ProxyJump <username>@gateway.napier.ac.uk
```

Replace:

```text
<username>
```

with your ENUCC username.

Example:

```sshconfig
Host ext.login.enucc
  HostName login.enucc.napier.ac.uk
  User 40000000
  ProxyJump 40000000@gateway.napier.ac.uk
```

After saving the file, connect from Terminal using:

```bash
ssh ext.login.enucc
```

In VS Code, you can also choose:

```text
Remote-SSH: Connect to Host
```

Then select:

```text
ext.login.enucc
```

---

## 7. Make Login Easier Using SSH Keys

SSH keys allow you to log in without typing your password every time.

An SSH key has two files:

```text
Private key: kept only on your local computer
Public key: copied to the server
```

Do not share your private key.

---

### 7.1 Create an SSH Key on Your Local Computer

On your local machine, run:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/enucc_key
```

Press Enter when asked for a passphrase, or set a passphrase if you want extra security.

This creates two files:

```text
~/.ssh/enucc_key
~/.ssh/enucc_key.pub
```

The private key is:

```text
~/.ssh/enucc_key
```

The public key is:

```text
~/.ssh/enucc_key.pub
```

---

### 7.2 Copy the Public Key to the Gateway

Copy your public key to the Napier gateway:

```bash
ssh-copy-id -i ~/.ssh/enucc_key.pub <username>@gateway.napier.ac.uk
```

Example:

```bash
ssh-copy-id -i ~/.ssh/enucc_key.pub 40000000@gateway.napier.ac.uk
```

---

### 7.3 Copy the Public Key to the ENUCC Login Node

Because ENUCC is reached through the gateway from outside the university network, copy your public key to the ENUCC login node using `ProxyJump`:

```bash
ssh-copy-id -i ~/.ssh/enucc_key.pub -o ProxyJump=<username>@gateway.napier.ac.uk <username>@login.enucc.napier.ac.uk
```

Example:

```bash
ssh-copy-id -i ~/.ssh/enucc_key.pub -o ProxyJump=40000000@gateway.napier.ac.uk 40000000@login.enucc.napier.ac.uk
```

If `ssh-copy-id` does not work, you can log in manually and paste the content of your public key into this file on the server:

```bash
~/.ssh/authorized_keys
```

Only copy the public key:

```bash
~/.ssh/enucc_key.pub
```

Never copy your private key to the server.

---

### 7.4 Update Your SSH Config File for SSH Key Login

Edit your local SSH config file:

```bash
~/.ssh/config
```

Use this configuration:

```sshconfig
Host enucc.gateway
  HostName gateway.napier.ac.uk
  User <username>
  IdentityFile ~/.ssh/enucc_key
  IdentitiesOnly yes

Host ext.login.enucc
  HostName login.enucc.napier.ac.uk
  User <username>
  ProxyJump enucc.gateway
  IdentityFile ~/.ssh/enucc_key
  IdentitiesOnly yes
```

Replace `<username>` with your ENUCC username.

Example:

```sshconfig
Host enucc.gateway
  HostName gateway.napier.ac.uk
  User 40000000
  IdentityFile ~/.ssh/enucc_key
  IdentitiesOnly yes

Host ext.login.enucc
  HostName login.enucc.napier.ac.uk
  User 40000000
  ProxyJump enucc.gateway
  IdentityFile ~/.ssh/enucc_key
  IdentitiesOnly yes
```

---

### 7.5 Test the SSH Key Connection

From your local computer, run:

```bash
ssh ext.login.enucc
```

If everything is correct, you should connect to the ENUCC login node.

You should see a prompt similar to:

```text
[40000000@login1[enucc] ~]$
```

---

## 8. Basic Port Forwarding

Port forwarding is useful when you run a web-based application on ENUCC, such as Jupyter Notebook or JupyterLab.

A basic SSH port-forwarding command looks like this:

```bash
ssh -NL 8888:localhost:8888 ext.login.enucc
```

This forwards port `8888` from ENUCC to your local computer.

Then, on your local computer, open a browser and go to:

```text
http://localhost:8888
```

If you are running Jupyter on a compute node, the tunnel command may need to point to the compute node hostname instead of `localhost`.

---

## 9. Useful Checks After Login

After logging in to ENUCC, check where you are:

```bash
hostname
hostname -f
pwd
whoami
```

Example output:

```text
login1.enucc.napier.ac.uk
/users/40000000
40000000
```

---

## 10. Important Notes

If you do not set up SSH keys, you can still log in using your ENU password.

Keep your private key safe:

```text
~/.ssh/enucc_key
```

Never share it with another person or upload it to a public website.

Only the public key should be copied to the server:

```text
~/.ssh/enucc_key.pub
```

Use the login node only for login, file management, editing scripts, and submitting jobs.

Do not run heavy computation directly on the login node.

For computation, use Slurm to request a compute node or submit a job.

---

## 11. Useful Learning Resources

If you are new to Linux command line, study the Unix Shell basics:

```text
https://swcarpentry.github.io/shell-novice/
```

For extra shell skills:

```text
https://carpentries-incubator.github.io/shell-extras/
```

For Slurm basics:

```text
https://slurm.schedmd.com/quickstart.html
```
