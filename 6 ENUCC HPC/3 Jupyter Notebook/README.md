# Run JupyterLab with SLURM on ENUCC

This guide assumes you already have the Python Conda environment:

```text
/users/40000000/conda/envs/python310
```

You can run JupyterLab in two ways:

```text
Option 1: CPU JupyterLab
Option 2: GPU JupyterLab
```

Use **CPU** for normal notebooks, data checking, plotting, and light work.

Use **GPU** for machine learning, deep learning, PyTorch, TensorFlow, CUDA, or GPU experiments.

---

## 1. Login to ENUCC

From your local machine:

```bash
ssh -J 40000000@gateway.napier.ac.uk 40000000@login.enucc.napier.ac.uk
```

After login:

```bash
hostname
pwd
```

You should be on the login node.

---

## 2. Activate Python Environment

On the login node:

```bash
module load apps/anaconda3/2024.10/bin
source "$(conda info --base)/etc/profile.d/conda.sh"
conda activate /users/40000000/conda/envs/python310
```

Check Python:

```bash
which python
python --version
```

Expected path:

```text
/users/40000000/conda/envs/python310/bin/python
```

---

## 3. Install JupyterLab

Install JupyterLab inside the `python310` environment:

```bash
python -m pip install --upgrade pip

python -m pip install \
    jupyter \
    jupyterlab \
    notebook \
    ipykernel \
    ipywidgets
```

Check Jupyter:

```bash
which jupyter
jupyter --version
```

Create log folder:

```bash
mkdir -p /users/40000000/slurmlogs
```

---

# Option 1: Run JupyterLab on CPU Node

Use this option when you do **not** need GPU.

---

## 4A. Create CPU SLURM Script

Create the file:

```bash
nano run_jupyter_cpu.sh
```

Paste this script:

```bash
#!/bin/bash
#SBATCH --job-name=jupyter_cpu
#SBATCH --partition=nodes
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --time=04:00:00
#SBATCH --output=/users/40000000/slurmlogs/jupyter_cpu_%j.out
#SBATCH --error=/users/40000000/slurmlogs/jupyter_cpu_%j.err

set -e

echo "============================================================"
echo "SLURM JOB INFO"
echo "============================================================"
echo "Job ID: $SLURM_JOB_ID"
echo "Job name: $SLURM_JOB_NAME"
echo "Node: $(hostname)"
echo "Node full name: $(hostname -f)"
echo "Start time: $(date)"
echo "Submit directory: $SLURM_SUBMIT_DIR"
echo "Working directory before cd: $(pwd)"

echo ""
echo "============================================================"
echo "MOVE TO HOME DIRECTORY"
echo "============================================================"

cd /users/40000000 || exit 1
mkdir -p /users/40000000/slurmlogs

echo "Working directory after cd: $(pwd)"

echo ""
echo "============================================================"
echo "LOAD MODULES"
echo "============================================================"

module purge
module load apps/anaconda3/2024.10/bin

module list

echo ""
echo "============================================================"
echo "ACTIVATE CONDA ENVIRONMENT"
echo "============================================================"

source "$(conda info --base)/etc/profile.d/conda.sh"

export CONDA_PKGS_DIRS=/users/40000000/conda/pkgs
export PIP_CACHE_DIR=/users/40000000/conda/pip_cache
export HF_HOME=/users/40000000/conda/hf_cache

conda activate /users/40000000/conda/envs/python310

echo "Conda environment: $CONDA_PREFIX"
echo "Python path: $(which python)"
echo "Jupyter path: $(which jupyter)"
python --version

echo ""
echo "============================================================"
echo "START JUPYTERLAB CPU"
echo "============================================================"

PORT=8765
NODE_FULLNAME=$(hostname -f)

echo "JupyterLab is running on CPU node:"
echo "$NODE_FULLNAME"
echo ""
echo "Port:"
echo "$PORT"
echo ""
echo "Use SSH tunnel from your Mac:"
echo "ssh -N -L ${PORT}:${NODE_FULLNAME}:${PORT} ENUCC"
echo ""
echo "Then open:"
echo "http://localhost:${PORT}"
echo ""
echo "Copy the token from the JupyterLab URL below."
echo ""

jupyter lab --no-browser --ip=0.0.0.0 --port=${PORT} --port-retries=0

echo ""
echo "============================================================"
echo "END TIME"
echo "============================================================"
date
```

Save and exit:

```text
Ctrl + O
Enter
Ctrl + X
```

Make it executable:

```bash
chmod +x run_jupyter_cpu.sh
```

---

## 5A. Submit CPU Jupyter Job

Submit the CPU job:

```bash
sbatch run_jupyter_cpu.sh
```

Example:

```text
Submitted batch job 138600
```

Check job status:

```bash
squeue -u 40000000
```

---

# Option 2: Run JupyterLab on GPU Node

Use this option when you need GPU.

---

## 4B. Create GPU SLURM Script

Create the file:

```bash
nano run_jupyter_gpu.sh
```

Paste this script:

```bash
#!/bin/bash
#SBATCH --job-name=jupyter_gpu
#SBATCH --partition=gpu
#SBATCH --gpus=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --time=04:00:00
#SBATCH --output=/users/40000000/slurmlogs/jupyter_gpu_%j.out
#SBATCH --error=/users/40000000/slurmlogs/jupyter_gpu_%j.err

set -e

echo "============================================================"
echo "SLURM JOB INFO"
echo "============================================================"
echo "Job ID: $SLURM_JOB_ID"
echo "Job name: $SLURM_JOB_NAME"
echo "Node: $(hostname)"
echo "Node full name: $(hostname -f)"
echo "Start time: $(date)"
echo "Submit directory: $SLURM_SUBMIT_DIR"
echo "Working directory before cd: $(pwd)"

echo ""
echo "============================================================"
echo "MOVE TO HOME DIRECTORY"
echo "============================================================"

cd /users/40000000 || exit 1
mkdir -p /users/40000000/slurmlogs

echo "Working directory after cd: $(pwd)"

echo ""
echo "============================================================"
echo "LOAD MODULES"
echo "============================================================"

module purge
module load apps/anaconda3/2024.10/bin
module load libs/nvidia-cuda/12.2.2/bin

module list

echo ""
echo "============================================================"
echo "ACTIVATE CONDA ENVIRONMENT"
echo "============================================================"

source "$(conda info --base)/etc/profile.d/conda.sh"

export CONDA_PKGS_DIRS=/users/40000000/conda/pkgs
export PIP_CACHE_DIR=/users/40000000/conda/pip_cache
export HF_HOME=/users/40000000/conda/hf_cache

conda activate /users/40000000/conda/envs/python310

echo "Conda environment: $CONDA_PREFIX"
echo "Python path: $(which python)"
echo "Jupyter path: $(which jupyter)"
python --version

echo ""
echo "============================================================"
echo "GPU CHECK"
echo "============================================================"

nvidia-smi || true

echo ""
echo "============================================================"
echo "START JUPYTERLAB GPU"
echo "============================================================"

PORT=8765
NODE_FULLNAME=$(hostname -f)

echo "JupyterLab is running on GPU node:"
echo "$NODE_FULLNAME"
echo ""
echo "Port:"
echo "$PORT"
echo ""
echo "Use SSH tunnel from your Mac:"
echo "ssh -N -L ${PORT}:${NODE_FULLNAME}:${PORT} ENUCC"
echo ""
echo "Then open:"
echo "http://localhost:${PORT}"
echo ""
echo "Copy the token from the JupyterLab URL below."
echo ""

jupyter lab --no-browser --ip=0.0.0.0 --port=${PORT} --port-retries=0

echo ""
echo "============================================================"
echo "END TIME"
echo "============================================================"
date
```

Save and exit:

```text
Ctrl + O
Enter
Ctrl + X
```

Make it executable:

```bash
chmod +x run_jupyter_gpu.sh
```

---

## 5B. Submit GPU Jupyter Job

Submit the GPU job:

```bash
sbatch run_jupyter_gpu.sh
```

Example:

```text
Submitted batch job 138601
```

Check job status:

```bash
squeue -u 40000000
```

---

# 6. Read the Output File

After submitting the job, check your log files:

```bash
ls -lt /users/40000000/slurmlogs/
```

For CPU job:

```bash
cat /users/40000000/slurmlogs/jupyter_cpu_JOBID.out
```

Example:

```bash
cat /users/40000000/slurmlogs/jupyter_cpu_138600.out
```

For GPU job:

```bash
cat /users/40000000/slurmlogs/jupyter_gpu_JOBID.out
```

Example:

```bash
cat /users/40000000/slurmlogs/jupyter_gpu_138601.out
```

You can also follow the output live:

```bash
tail -f /users/40000000/slurmlogs/jupyter_gpu_JOBID.out
```

or:

```bash
tail -f /users/40000000/slurmlogs/jupyter_cpu_JOBID.out
```

---

# 7. Copy SSH Tunnel Command

Inside the `.out` file, find this section:

```text
Use SSH tunnel from your Mac:
ssh -N -L 8765:NODE_FULLNAME:8765 ENUCC
```

Example:

```bash
ssh -N -L 8765:gpu01.enucc.enu.alces.network:8765 ENUCC
```

Run that command on your **Mac terminal**.

Keep the Mac terminal open.

---

# 8. Open JupyterLab in Browser

Open this in your local browser:

```text
http://localhost:8765
```

The Jupyter token is shown in the `.out` file.

Look for a line like:

```text
http://127.0.0.1:8765/lab?token=xxxxxxxxxxxxxxxx
```

Copy the token and paste it into the browser.

---

# 9. Stop JupyterLab

Check your running jobs:

```bash
squeue -u 40000000
```

Cancel the job:

```bash
scancel JOBID
```

Example:

```bash
scancel 138601
```

Check again:

```bash
squeue -u 40000000
```

If no job appears, JupyterLab has stopped.

---

# 10. Check Error File

If something fails, check the `.err` file.

For CPU:

```bash
cat /users/40000000/slurmlogs/jupyter_cpu_JOBID.err
```

For GPU:

```bash
cat /users/40000000/slurmlogs/jupyter_gpu_JOBID.err
```

Common things to check:

```text
- Conda environment path is correct
- Jupyter is installed in python310
- SLURM partition name is correct
- Port 8765 is not already being used
- SSH tunnel command uses the correct node name
```

---

# 11. If Port 8765 Is Already Used

Change this line inside the script:

```bash
PORT=8765
```

For example:

```bash
PORT=8766
```

Then the browser address becomes:

```text
http://localhost:8766
```

The SSH tunnel command will also use the new port.

---

# 12. Main Commands Summary

Activate environment:

```bash
module load apps/anaconda3/2024.10/bin
source "$(conda info --base)/etc/profile.d/conda.sh"
conda activate /users/40000000/conda/envs/python310
```

Submit CPU Jupyter:

```bash
sbatch run_jupyter_cpu.sh
```

Submit GPU Jupyter:

```bash
sbatch run_jupyter_gpu.sh
```

Check jobs:

```bash
squeue -u 40000000
```

Read output:

```bash
cat /users/40000000/slurmlogs/jupyter_cpu_JOBID.out
cat /users/40000000/slurmlogs/jupyter_gpu_JOBID.out
```

Run SSH tunnel on Mac:

```bash
ssh -N -L 8765:NODE_FULLNAME:8765 ENUCC
```

Open browser:

```text
http://localhost:8765
```

Stop Jupyter:

```bash
scancel JOBID
```

---

# Summary

Use **CPU JupyterLab** when you do not need GPU:

```bash
sbatch run_jupyter_cpu.sh
```

Use **GPU JupyterLab** when you need GPU:

```bash
sbatch run_jupyter_gpu.sh
```

Both methods use the same browser address:

```text
http://localhost:8765
```

The node name and token are shown in the SLURM `.out` file.
