# ENUCC Login Guide

This guide explains how to log in to the ENUCC HPC system from your local computer.

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

## 5. Useful First Commands

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

## 6. Important Notes

Do not run heavy Python, MATLAB, or machine learning jobs directly on the login node.

The login node should mainly be used for:

```text
- Editing files
- Uploading or downloading data
- Submitting SLURM jobs
- Checking job status
```

Heavy computation should be submitted to a compute node using SLURM.

Example:

```bash
squeue -u <ENU_ID>
```

Cancel a running job:

```bash
scancel <JOB_ID>
```

---

## 7. Logout

To exit ENUCC, run:

```bash
exit
```

If you are still inside the gateway session, run `exit` again until you return to your local computer terminal.

---

## 8. Quick Login Command Summary

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
