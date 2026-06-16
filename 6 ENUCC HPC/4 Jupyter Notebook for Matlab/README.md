# Run JupyterLab with MATLAB on ENUCC

This guide assumes you already completed the Python JupyterLab setup and already have this Conda environment:

```text
/users/40000000/conda/envs/python310
```

Now we will add **MATLAB support inside JupyterLab**.

After setup, JupyterLab will show both:

```text
Python 3
MATLAB
```

as available notebook kernels.

---

## 1. Login to ENUCC

From your local machine:

```bash
ssh -J 40000000@gateway.napier.ac.uk 40000000@login.enucc.napier.ac.uk
```

After login, check:

```bash
hostname
pwd
```

You should be on the ENUCC login node.

---

## 2. Activate the Existing Python Environment

On the login node:

```bash
module load apps/anaconda3/2024.10/bin
source "$(conda info --base)/etc/profile.d/conda.sh"
conda activate /users/40000000/conda/envs/python310
```

Check:

```bash
which python
python --version
which jupyter
```

Expected Python path:

```text
/users/40000000/conda/envs/python310/bin/python
```

---

## 3. Load MATLAB Module

Load MATLAB R2023b:

```bash
module load apps/matlab/r2023b
```

Check MATLAB:

```bash
which matlab
matlab -batch "disp(version)"
```

---

## 4. Install MATLAB Kernel for Jupyter

Inside the activated `python310` environment:

```bash
python -m pip install --upgrade pip
python -m pip install matlab_kernel
```

Register the MATLAB kernel:

```bash
python -m matlab_kernel install --user
```

Check available kernels:

```bash
python -m jupyter kernelspec list
```

You should see something similar to:

```text
Available kernels:
  matlab     /users/40000000/conda/envs/python310/share/jupyter/kernels/matlab
  python3    /users/40000000/conda/envs/python310/share/jupyter/kernels/python3
```

If you see `matlab`, then the MATLAB Jupyter kernel is installed.

---

# Run JupyterLab with MATLAB Kernel on GPU Node

Use this option when you want to open JupyterLab on a GPU node and use MATLAB from Jupyter.

---

## 5. Create MATLAB Jupyter SLURM Script

Create the file:

```bash
nano run_jupyter_matlab_gpu.sh
```

Paste this script:

```bash
#!/bin/bash
#SBATCH --job-name=jupyter_matlab_gpu
#SBATCH --partition=gpu
#SBATCH --gpus=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=32G
#SBATCH --time=04:00:00
#SBATCH --output=/users/40000000/slurmlogs/jupyter_matlab_gpu_%j.out
#SBATCH --error=/users/40000000/slurmlogs/jupyter_matlab_gpu_%j.err

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
module load apps/matlab/r2023b

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

# Force the correct Conda environment binaries and libraries
export PATH="$CONDA_PREFIX/bin:$PATH"
export LD_LIBRARY_PATH="$CONDA_PREFIX/lib:${LD_LIBRARY_PATH:-}"

# Avoid Python conflicts from login/module environment
unset PYTHONPATH

# Refresh shell command lookup cache
hash -r

echo "Conda environment: $CONDA_PREFIX"
echo "Python path: $(which python)"
echo "Jupyter path: $(which jupyter)"
echo "MATLAB path: $(which matlab)"
python --version

echo ""
echo "============================================================"
echo "PYTHON / LIBRARY CHECK"
echo "============================================================"

python -c "import sys; print('Python executable:', sys.executable)"
python -c "import pyexpat; print('pyexpat OK inside SLURM job')"

echo ""
echo "libexpat used by Python pyexpat:"
ldd "$CONDA_PREFIX"/lib/python3.10/lib-dynload/pyexpat*.so | grep expat || true

echo ""
echo "============================================================"
echo "MATLAB CHECK"
echo "============================================================"

matlab -batch "disp(version)" || true

echo ""
echo "============================================================"
echo "MATLAB KERNEL CHECK"
echo "============================================================"

python -c "import matlab_kernel; print('matlab_kernel OK')" || echo "matlab_kernel not found or failed"

echo ""
echo "Available Jupyter kernels:"
python -m jupyter kernelspec list || true

echo ""
echo "============================================================"
echo "GPU CHECK"
echo "============================================================"

nvidia-smi || true

echo ""
echo "============================================================"
echo "START JUPYTERLAB WITH MATLAB KERNEL"
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

# Force Conda libexpat to load before MATLAB's libraries
export LD_PRELOAD="$CONDA_PREFIX/lib/libexpat.so.1"

python -m jupyter lab --no-browser --ip=0.0.0.0 --port="$PORT" --port-retries=0

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
chmod +x run_jupyter_matlab_gpu.sh
```

---

## 6. Submit MATLAB Jupyter Job

Submit the job:

```bash
sbatch run_jupyter_matlab_gpu.sh
```

Example:

```text
Submitted batch job 138700
```

Check job status:

```bash
squeue -u 40000000
```

---

## 7. Read the Output File

Check the latest log file:

```bash
ls -lt /users/40000000/slurmlogs/
```

Read the output:

```bash
cat /users/40000000/slurmlogs/jupyter_matlab_gpu_JOBID.out
```

Example:

```bash
cat /users/40000000/slurmlogs/jupyter_matlab_gpu_138700.out
```

Or follow the output live:

```bash
tail -f /users/40000000/slurmlogs/jupyter_matlab_gpu_JOBID.out
```

---

## 8. Copy SSH Tunnel Command

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

## 9. Open JupyterLab in Browser

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

## 10. Create a MATLAB Notebook

Inside JupyterLab:

```text
File > New > Notebook
```

Select:

```text
MATLAB
```

Now you can run MATLAB code inside JupyterLab.

---

## 11. Simple MATLAB Test Code

Try this in a MATLAB notebook:

```matlab
clc
clear

disp("Hello from MATLAB inside JupyterLab")

x = linspace(0, 2*pi, 200);
y = sin(x);

plot(x, y, 'LineWidth', 2)
grid on
xlabel("x")
ylabel("sin(x)")
title("MATLAB Kernel Test")
```

---

## 12. Check GPU from MATLAB

Try:

```matlab
gpuDevice
```

If MATLAB can see the GPU, it should show GPU information.

You can also test a simple GPU calculation:

```matlab
A = gpuArray(rand(1000));
B = A * A;
gather(B(1,1))
```

---

## 13. Stop JupyterLab

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
scancel 138700
```

Check again:

```bash
squeue -u 40000000
```

If no job appears, JupyterLab has stopped.

---

## 14. If Port 8765 Is Already Used

Edit the script:

```bash
nano run_jupyter_matlab_gpu.sh
```

Change:

```bash
PORT=8765
```

to another port, for example:

```bash
PORT=8766
```

Then the tunnel command becomes:

```bash
ssh -N -L 8766:NODE_FULLNAME:8766 ENUCC
```

And the browser address becomes:

```text
http://localhost:8766
```

---

## 15. Troubleshooting

### MATLAB kernel does not appear

Check kernels:

```bash
python -m jupyter kernelspec list
```

If `matlab` is missing, run:

```bash
python -m pip install matlab_kernel
python -m matlab_kernel install --user
```

Then restart the SLURM job.

---

### MATLAB works in terminal but not in Jupyter

Check that MATLAB is loaded inside the SLURM script:

```bash
module load apps/matlab/r2023b
```

Also check:

```bash
which matlab
matlab -batch "disp(version)"
```

---

### Jupyter opens but MATLAB notebook fails

Check the error log:

```bash
cat /users/40000000/slurmlogs/jupyter_matlab_gpu_JOBID.err
```

Also check the output log:

```bash
cat /users/40000000/slurmlogs/jupyter_matlab_gpu_JOBID.out
```

Important checks:

```text
- MATLAB module is loaded
- Conda environment is activated
- matlab_kernel is installed
- MATLAB kernel appears in jupyter kernelspec list
- LD_PRELOAD points to Conda libexpat
```

---

## Main Commands Summary

Activate environment:

```bash
module load apps/anaconda3/2024.10/bin
source "$(conda info --base)/etc/profile.d/conda.sh"
conda activate /users/40000000/conda/envs/python310
```

Load MATLAB:

```bash
module load apps/matlab/r2023b
```

Install MATLAB kernel:

```bash
python -m pip install matlab_kernel
python -m matlab_kernel install --user
```

Submit MATLAB Jupyter GPU job:

```bash
sbatch run_jupyter_matlab_gpu.sh
```

Check jobs:

```bash
squeue -u 40000000
```

Read output:

```bash
cat /users/40000000/slurmlogs/jupyter_matlab_gpu_JOBID.out
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

You already have Python JupyterLab working.

To add MATLAB:

```bash
module load apps/matlab/r2023b
python -m pip install matlab_kernel
python -m matlab_kernel install --user
```

Then run:

```bash
sbatch run_jupyter_matlab_gpu.sh
```

Open:

```text
http://localhost:8765
```

Choose the **MATLAB** kernel when creating a new notebook.
