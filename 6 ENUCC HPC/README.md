# Run MATLAB Kernel in JupyterLab on ENUCC

This guide assumes that Python JupyterLab already works with this Conda environment:

```text
/users/40020957/conda/envs/python310
```

Now we want to add MATLAB support so that JupyterLab can run both:

```text
Python notebooks
MATLAB notebooks
```

---

## 1. What Extra Setup Is Needed?

If Python JupyterLab already works, you only need extra setup for MATLAB:

```text
1. Load MATLAB module
2. Install MATLAB Engine API for Python
3. Install matlab_kernel
4. Register/check MATLAB kernel in Jupyter
5. Run JupyterLab using a SLURM script that loads MATLAB
```

This setup should be done on the **login node**.

Do not run heavy notebooks on the login node. Use SLURM to start JupyterLab on a compute or GPU node.

---

## 2. Login to ENUCC

From your local machine:

```bash
ssh -J 40020957@gateway.napier.ac.uk 40020957@login.enucc.napier.ac.uk
```

Check that you are on the login node:

```bash
hostname
pwd
```

---

## 3. Load Python and MATLAB Modules

On the login node:

```bash
module purge
module load apps/anaconda3/2024.10/bin
module load apps/matlab/r2023b
```

Check MATLAB:

```bash
which matlab
matlab -batch "disp(version)"
```

---

## 4. Activate Python Environment

Enable Conda:

```bash
source "$(conda info --base)/etc/profile.d/conda.sh"
```

Activate your Python environment:

```bash
conda activate /users/40020957/conda/envs/python310
```

Check Python and Jupyter:

```bash
which python
python --version

which jupyter
jupyter --version
```

Expected Python path:

```text
/users/40020957/conda/envs/python310/bin/python
```

---

## 5. Install Basic Jupyter Packages

If Jupyter already works, you can skip this step.

Otherwise, install Jupyter packages:

```bash
python -m pip install --upgrade pip

python -m pip install \
    jupyter \
    jupyterlab \
    notebook \
    ipykernel \
    ipywidgets
```

---

## 6. Install MATLAB Engine API for Python

First, find the MATLAB root folder:

```bash
matlab -batch "disp(matlabroot)"
```

Example output may look like:

```text
/opt/apps/alces/matlab/R2023b
```

Go to the MATLAB Engine folder:

```bash
cd /opt/apps/alces/matlab/R2023b/extern/engines/python
```

Install the MATLAB Engine into your active Conda environment:

```bash
python -m pip install .
```

Test MATLAB Engine:

```bash
python -c "import matlab.engine; print('MATLAB Engine OK')"
```

If this prints:

```text
MATLAB Engine OK
```

then the MATLAB Engine is installed correctly.

---

## 7. Install MATLAB Kernel for Jupyter

Install the MATLAB kernel package:

```bash
python -m pip install matlab_kernel
```

Register the MATLAB kernel:

```bash
python -m matlab_kernel install --user
```

Check that the MATLAB kernel appears:

```bash
python -m jupyter kernelspec list
```

You should see something like:

```text
matlab      /users/40020957/.local/share/jupyter/kernels/matlab
python310   /users/40020957/.local/share/jupyter/kernels/python310
```

Test the MATLAB kernel package:

```bash
python -c "import matlab_kernel; print('matlab_kernel OK')"
```

---

## 8. Optional: Check MATLAB Kernel

Run:

```bash
python -m matlab_kernel.check
```

This command checks whether the MATLAB kernel can find MATLAB correctly.

If the check passes, MATLAB should be available inside JupyterLab.

---

# Option 1: MATLAB JupyterLab on CPU Node

Use this if you do not need GPU.

---

## 9A. Create CPU SLURM Script

Create the file:

```bash
nano run_jupyter_matlab_cpu.sh
```

Paste this script:

```bash
#!/bin/bash
#SBATCH --job-name=jupyter_matlab_cpu
#SBATCH --partition=nodes
#SBATCH --cpus-per-task=4
#SBATCH --mem=32G
#SBATCH --time=04:00:00
#SBATCH --output=/users/40020957/slurmlogs/jupyter_matlab_cpu_%j.out
#SBATCH --error=/users/40020957/slurmlogs/jupyter_matlab_cpu_%j.err

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

cd /users/40020957 || exit 1
mkdir -p /users/40020957/slurmlogs

echo "Working directory after cd: $(pwd)"

echo ""
echo "============================================================"
echo "LOAD MODULES"
echo "============================================================"

module purge
module load apps/anaconda3/2024.10/bin
module load apps/matlab/r2023b

module list

echo ""
echo "============================================================"
echo "ACTIVATE CONDA ENVIRONMENT"
echo "============================================================"

source "$(conda info --base)/etc/profile.d/conda.sh"

export CONDA_PKGS_DIRS=/users/40020957/conda/pkgs
export PIP_CACHE_DIR=/users/40020957/conda/pip_cache
export HF_HOME=/users/40020957/conda/hf_cache

conda activate /users/40020957/conda/envs/python310

export PATH="$CONDA_PREFIX/bin:$PATH"
export LD_LIBRARY_PATH="$CONDA_PREFIX/lib:${LD_LIBRARY_PATH:-}"
unset PYTHONPATH
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
echo "============================================================"
echo "MATLAB CHECK"
echo "============================================================"

matlab -batch "disp(version)" || true

echo ""
echo "============================================================"
echo "MATLAB ENGINE CHECK"
echo "============================================================"

python -c "import matlab.engine; print('MATLAB Engine OK')" || echo "MATLAB Engine not found or failed"

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
echo "START JUPYTERLAB WITH MATLAB KERNEL"
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
chmod +x run_jupyter_matlab_cpu.sh
```

Submit:

```bash
sbatch run_jupyter_matlab_cpu.sh
```

---

# Option 2: MATLAB JupyterLab on GPU Node

Use this if you need GPU or CUDA.

---

## 9B. Create GPU SLURM Script

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
#SBATCH --output=/users/40020957/slurmlogs/jupyter_matlab_gpu_%j.out
#SBATCH --error=/users/40020957/slurmlogs/jupyter_matlab_gpu_%j.err

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

cd /users/40020957 || exit 1
mkdir -p /users/40020957/slurmlogs

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

export CONDA_PKGS_DIRS=/users/40020957/conda/pkgs
export PIP_CACHE_DIR=/users/40020957/conda/pip_cache
export HF_HOME=/users/40020957/conda/hf_cache

conda activate /users/40020957/conda/envs/python310

export PATH="$CONDA_PREFIX/bin:$PATH"
export LD_LIBRARY_PATH="$CONDA_PREFIX/lib:${LD_LIBRARY_PATH:-}"
unset PYTHONPATH
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
echo "MATLAB ENGINE CHECK"
echo "============================================================"

python -c "import matlab.engine; print('MATLAB Engine OK')" || echo "MATLAB Engine not found or failed"

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

Submit:

```bash
sbatch run_jupyter_matlab_gpu.sh
```

---

## 10. Check Job Status

Check your running jobs:

```bash
squeue -u 40020957
```

Example:

```text
JOBID     PARTITION     NAME                 USER       ST     TIME     NODELIST
138700    gpu           jupyter_matlab_gpu   40020957   R      0:01     gpu01
```

Wait until the job state is:

```text
R
```

---

## 11. Read the Output File

List log files:

```bash
ls -lt /users/40020957/slurmlogs/
```

For CPU:

```bash
cat /users/40020957/slurmlogs/jupyter_matlab_cpu_JOBID.out
```

For GPU:

```bash
cat /users/40020957/slurmlogs/jupyter_matlab_gpu_JOBID.out
```

Example:

```bash
cat /users/40020957/slurmlogs/jupyter_matlab_gpu_138700.out
```

Or follow live:

```bash
tail -f /users/40020957/slurmlogs/jupyter_matlab_gpu_JOBID.out
```

---

## 12. Copy SSH Tunnel Command

Inside the `.out` file, look for:

```text
Use SSH tunnel from your Mac:
ssh -N -L 8765:NODE_FULLNAME:8765 ENUCC
```

Example:

```bash
ssh -N -L 8765:gpu01.enucc.enu.alces.network:8765 ENUCC
```

Run this command on your **Mac terminal**.

Keep the Mac terminal open.

---

## 13. Open JupyterLab

Open this in your local browser:

```text
http://localhost:8765
```

Copy the token from the `.out` file.

Look for a line similar to:

```text
http://127.0.0.1:8765/lab?token=xxxxxxxxxxxxxxxx
```

Paste the token into the browser.

---

## 14. Select MATLAB Kernel

Inside JupyterLab:

```text
File -> New -> Notebook
```

Then select:

```text
MATLAB
```

or:

```text
matlab
```

You can also check the launcher page and choose the MATLAB notebook option.

---

## 15. Test MATLAB in Jupyter

Create a new MATLAB notebook and run:

```matlab
disp("MATLAB kernel is working")
version
```

Test a simple plot:

```matlab
x = 0:0.1:10;
y = sin(x);

plot(x, y)
title("Simple MATLAB Plot")
xlabel("x")
ylabel("sin(x)")
grid on
```

Test matrix calculation:

```matlab
A = [1 2; 3 4];
B = inv(A);
disp(B)
```

---

## 16. Test MATLAB GPU

Only use this in the GPU job.

In a MATLAB notebook:

```matlab
gpuDevice
```

Test GPU array:

```matlab
A = gpuArray(rand(1000));
B = A * A;
gather(B(1:5, 1:5))
```

If GPU is available, MATLAB should show GPU device information.

---

## 17. Stop JupyterLab

Check jobs:

```bash
squeue -u 40020957
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
squeue -u 40020957
```

If no job appears, JupyterLab has stopped.

---

## 18. Check Error File

For CPU:

```bash
cat /users/40020957/slurmlogs/jupyter_matlab_cpu_JOBID.err
```

For GPU:

```bash
cat /users/40020957/slurmlogs/jupyter_matlab_gpu_JOBID.err
```

Common problems:

```text
matlab_kernel not found
MATLAB Engine not found
MATLAB module not loaded
Jupyter started from wrong Python environment
Port 8765 already used
MATLAB license unavailable
```

---

## 19. If MATLAB Kernel Does Not Appear

Activate the environment again:

```bash
module load apps/anaconda3/2024.10/bin
module load apps/matlab/r2023b
source "$(conda info --base)/etc/profile.d/conda.sh"
conda activate /users/40020957/conda/envs/python310
```

Reinstall and register the kernel:

```bash
python -m pip install matlab_kernel
python -m matlab_kernel install --user
python -m jupyter kernelspec list
```

Check:

```bash
python -c "import matlab_kernel; print('matlab_kernel OK')"
python -m matlab_kernel.check
```

---

## 20. If MATLAB Engine Does Not Work

Check MATLAB root:

```bash
matlab -batch "disp(matlabroot)"
```

Go to the MATLAB Engine folder:

```bash
cd /opt/apps/alces/matlab/R2023b/extern/engines/python
```

Reinstall:

```bash
python -m pip install .
```

Check again:

```bash
python -c "import matlab.engine; print('MATLAB Engine OK')"
```

---

## 21. If Port 8765 Is Already Used

Edit the script and change:

```bash
PORT=8765
```

to another port:

```bash
PORT=8766
```

Then submit again:

```bash
sbatch run_jupyter_matlab_gpu.sh
```

The browser address becomes:

```text
http://localhost:8766
```

---

## Main Commands Summary

Setup MATLAB kernel:

```bash
module purge
module load apps/anaconda3/2024.10/bin
module load apps/matlab/r2023b

source "$(conda info --base)/etc/profile.d/conda.sh"
conda activate /users/40020957/conda/envs/python310

cd /opt/apps/alces/matlab/R2023b/extern/engines/python
python -m pip install .

python -m pip install matlab_kernel
python -m matlab_kernel install --user

python -c "import matlab.engine; print('MATLAB Engine OK')"
python -c "import matlab_kernel; print('matlab_kernel OK')"
python -m jupyter kernelspec list
```

Run MATLAB Jupyter on CPU:

```bash
sbatch run_jupyter_matlab_cpu.sh
```

Run MATLAB Jupyter on GPU:

```bash
sbatch run_jupyter_matlab_gpu.sh
```

Check job:

```bash
squeue -u 40020957
```

Read output:

```bash
cat /users/40020957/slurmlogs/jupyter_matlab_gpu_JOBID.out
```

Run SSH tunnel on Mac:

```bash
ssh -N -L 8765:NODE_FULLNAME:8765 ENUCC
```

Open browser:

```text
http://localhost:8765
```

Stop job:

```bash
scancel JOBID
```

---

## Summary

If Python JupyterLab already works, MATLAB needs this extra setup:

```text
1. MATLAB module
2. MATLAB Engine API for Python
3. matlab_kernel
4. MATLAB kernelspec
5. SLURM script that loads MATLAB before starting JupyterLab
```

After setup, JupyterLab should show both Python and MATLAB kernels.
