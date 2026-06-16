# Create Python Conda Environment on ENUCC

This guide explains how to create a Python Conda environment on ENUCC.

Replace:

```text
ENU_ID
```

with your ENUCC username.

Example:

```text
40000000
```

---

## 1. Login to ENUCC

From your local machine, login to ENUCC through the Napier gateway:

```bash
ssh -J ENU_ID@gateway.napier.ac.uk ENU_ID@login.enucc.napier.ac.uk
```

Example:

```bash
ssh -J 40000000@gateway.napier.ac.uk 40000000@login.enucc.napier.ac.uk
```

You may need to enter your ENU password more than once.

After login, check that you are on the login node:

```bash
hostname
pwd
```

Your home directory should look like:

```bash
/users/ENU_ID
```

Example:

```bash
/users/40000000
```

---

## 2. Important Note

The Conda environment should be created on the **login node**.

Use the login node for:

```text
- Loading Anaconda
- Creating the Conda environment
- Activating the environment
- Installing Python packages
- Registering the Jupyter kernel
```

Do **not** use the login node for heavy computation, model training, or long-running Jupyter sessions.

---

## 3. Load Anaconda

Load the Anaconda module:

```bash
module load apps/anaconda3/2024.10/bin
```

Enable Conda in the shell:

```bash
source "$(conda info --base)/etc/profile.d/conda.sh"
```

Check Conda:

```bash
conda --version
```

---

## 4. Create Conda Environment Folder

Create a folder to store your Conda environments:

```bash
mkdir -p /users/ENU_ID/conda/envs
```

Example:

```bash
mkdir -p /users/40000000/conda/envs
```

---

## 5. Create Python Environment

Create a Python 3.10 environment:

```bash
conda create -p /users/ENU_ID/conda/envs/python310 python=3.10 -y
```

Example:

```bash
conda create -p /users/40000000/conda/envs/python310 python=3.10 -y
```

---

## 6. Activate the Environment

Activate the environment:

```bash
conda activate /users/ENU_ID/conda/envs/python310
```

Example:

```bash
conda activate /users/40000000/conda/envs/python310
```

After activation, your terminal should show something like:

```bash
(python310) [ENU_ID@login1 ~]$
```

Check Python:

```bash
which python
python --version
```

Expected Python path:

```bash
/users/ENU_ID/conda/envs/python310/bin/python
```

Example:

```bash
/users/40000000/conda/envs/python310/bin/python
```

---

## 7. Install Common Python Packages

Upgrade `pip` first:

```bash
python -m pip install --upgrade pip
```

Install common packages:

```bash
python -m pip install \
    numpy \
    pandas \
    scipy \
    scikit-learn \
    matplotlib \
    seaborn \
    tqdm \
    jupyter \
    jupyterlab \
    notebook \
    ipykernel \
    ipywidgets
```

Check installed packages:

```bash
python -m pip list
```

---

## 8. Register the Environment as a Jupyter Kernel

Register the Conda environment so it appears in Jupyter:

```bash
python -m ipykernel install --user \
    --name python310 \
    --display-name "Python 3.10 (python310)"
```

Check available Jupyter kernels:

```bash
python -m jupyter kernelspec list
```

You should see something like:

```text
python310    /users/ENU_ID/.local/share/jupyter/kernels/python310
```

---

## 9. Test the Environment

Run Python:

```bash
python
```

Test common packages:

```python
import numpy as np
import pandas as pd
import sklearn
import matplotlib

print("Python environment is working.")
```

Exit Python:

```python
exit()
```

---

## 10. Use the Environment Again Later

Every time you login again, run:

```bash
module load apps/anaconda3/2024.10/bin
source "$(conda info --base)/etc/profile.d/conda.sh"
conda activate /users/ENU_ID/conda/envs/python310
```

Example:

```bash
module load apps/anaconda3/2024.10/bin
source "$(conda info --base)/etc/profile.d/conda.sh"
conda activate /users/40000000/conda/envs/python310
```

---

## 11. Useful Commands

Check Conda environments:

```bash
conda env list
```

Check current Python path:

```bash
which python
```

Check Python version:

```bash
python --version
```

Check installed packages:

```bash
python -m pip list
```

Check Jupyter kernels:

```bash
python -m jupyter kernelspec list
```

---

## 12. Remove the Environment If Needed

Only do this if you want to completely delete the environment.

Deactivate Conda first:

```bash
conda deactivate
```

Remove the environment folder:

```bash
rm -rf /users/ENU_ID/conda/envs/python310
```

Example:

```bash
rm -rf /users/40000000/conda/envs/python310
```

---

## Summary

Create the Python environment on the **login node**.

Use this environment path:

```bash
/users/ENU_ID/conda/envs/python310
```

Example:

```bash
/users/40000000/conda/envs/python310
```

After creating it, activate it with:

```bash
conda activate /users/ENU_ID/conda/envs/python310
```
