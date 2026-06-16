# ENUCC Login Guide

This guide explains how to log in to the ENUCC HPC system from your local computer using:

1. SSH Terminal
2. VS Code Remote SSH

---

## 1. Login Overview

To access ENUCC from outside the university network, you usually need to connect through the Napier gateway first.

The login path is:

```text
Your computer
    ↓
Napier gateway
    ↓
ENUCC login node
```

The gateway is used because the ENUCC login node is not directly accessible from outside the university network.

---

## 2. Login Using SSH Terminal

Open Terminal on your local computer and run:

```bash
ssh -J <ENU_ID>@gateway.napier.ac.uk <ENU_ID>@login.enucc.napier.ac.uk
```

Replace:

```text
<ENU_ID>
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

When login is successful, your terminal prompt should look similar to:

```bash
[40000000@login1[enucc] ~]$
```

This means you are now on the ENUCC login node.

---

## 3. Check Where You Are

After logging in, run:

```bash
hostname
hostname -f
pwd
```

Example output may look like:

```text
login1
login1.enucc.napier.ac.uk
/users/40000000
```

Your home directory should be:

```bash
/users/<ENU_ID>
```

Example:

```bash
/users/40000000
```

---

## 4. Optional: Create SSH Config for Easier Login

Instead of typing the full SSH command every time, you can add an SSH config on your local computer.

Open or create the SSH config file:

```bash
nano ~/.ssh/config
```

Add:

```sshconfig
Host ENUCC
    HostName login.enucc.napier.ac.uk
    User <ENU_ID>
    ProxyJump <ENU_ID>@gateway.napier.ac.uk
```

Example:

```sshconfig
Host ENUCC
    HostName login.enucc.napier.ac.uk
    User 40000000
    ProxyJump 40000000@gateway.napier.ac.uk
```

Save the file.

Then you can log in using:

```bash
ssh ENUCC
```

---

## 5. Install VS Code on Your Local Computer

You can also use VS Code to connect to ENUCC and edit files more easily.

### 5.1 Download VS Code

On your local computer, download and install Visual Studio Code from the official Visual Studio Code website.

Choose the correct version for your operating system:

```text
- Windows
- macOS
- Linux
```

After installation, open VS Code.

---

## 6. Install the Remote SSH Extension in VS Code

In VS Code:

```text
1. Open VS Code
2. Go to Extensions
3. Search for: Remote - SSH
4. Install the extension by Microsoft
```

This extension allows VS Code to connect to ENUCC through SSH.

---

## 7. Connect to ENUCC Using VS Code

Before using VS Code, it is recommended to create the SSH config from Section 4.

After that, in VS Code:

```text
1. Press Ctrl + Shift + P on Windows/Linux
   or Cmd + Shift + P on macOS

2. Search for:
   Remote-SSH: Connect to Host...

3. Select:
   ENUCC

4. Enter your ENU password when asked
```

VS Code will connect to the ENUCC login node.

The first connection may take some time because VS Code needs to install its remote server files in your ENUCC account.

---

## 8. Open Your ENUCC Folder in VS Code

After VS Code connects to ENUCC:

```text
1. Click Open Folder
2. Select your home folder:
   /users/<ENU_ID>
```

Example:

```bash
/users/40000000
```

You can also open a specific working folder, for example:

```bash
/users/40000000/TTHER
```

or:

```bash
/users/40000000/DATASETS
```

---

## 9. Open a Terminal Inside VS Code

After connecting to ENUCC with VS Code:

```text
Terminal > New Terminal
```

The terminal inside VS Code should now be running on the ENUCC login node.

You can check with:

```bash
hostname
pwd
```

Example:

```text
login1
/users/40000000
```

---

## 10. Useful First Commands

After logging in, you can check your files:

```bash
ls -l
```

Check available storage folders:

```bash
pwd
du -sh *
```

Create useful working folders:

```bash
mkdir -p ~/DATASETS
mkdir -p ~/TTHER
mkdir -p ~/slurmlogs
mkdir -p ~/conda
```

Recommended folder structure:

```text
/users/<ENU_ID>/
├── DATASETS/
├── TTHER/
├── conda/
└── slurmlogs/
```

---

## 11. Important Notes

Do not run heavy Python, MATLAB, or machine learning jobs directly on the login node.

The login node should mainly be used for:

```text
- Editing files
- Uploading or downloading data
- Submitting SLURM jobs
- Checking job status
```

Heavy computation should be submitted to a compute node using SLURM.

Check your running jobs:

```bash
squeue -u <ENU_ID>
```

Example:

```bash
squeue -u 40000000
```

Cancel a running job:

```bash
scancel <JOB_ID>
```

Example:

```bash
scancel 123456
```

---

## 12. Using VS Code Safely on ENUCC

VS Code is useful for editing files, writing scripts, and managing folders.

Good use of VS Code on the login node:

```text
- Edit Python files
- Edit MATLAB files
- Edit SLURM scripts
- Edit README files
- Submit jobs using sbatch
- Check job output logs
```

Avoid doing heavy work directly in the VS Code terminal on the login node:

```text
- Do not train machine learning models on the login node
- Do not run long Python scripts on the login node
- Do not run heavy MATLAB jobs on the login node
- Do not use the login node like a compute node
```

Instead, submit jobs using SLURM.

Example:

```bash
sbatch run.sh
```

---

## 13. Logout

To exit ENUCC from Terminal, run:

```bash
exit
```

If you are still inside the gateway session, run `exit` again until you return to your local computer terminal.

To disconnect from VS Code:

```text
1. Click the green Remote SSH button in the bottom-left corner
2. Choose Close Remote Connection
```

---

## 14. Quick Login Command Summary

Login directly using SSH:

```bash
ssh -J <ENU_ID>@gateway.napier.ac.uk <ENU_ID>@login.enucc.napier.ac.uk
```

Example:

```bash
ssh -J 40000000@gateway.napier.ac.uk 40000000@login.enucc.napier.ac.uk
```

Or, after setting up `~/.ssh/config`:

```bash
ssh ENUCC
```

For VS Code:

```text
1. Install VS Code
2. Install Remote - SSH extension
3. Add ENUCC to ~/.ssh/config
4. Use Remote-SSH: Connect to Host...
5. Select ENUCC
```
