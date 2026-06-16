# First SLURM CPU Job Guide

This guide shows the basic SLURM workflow for running a simple CPU job on ENUCC.

This example covers:

1. Creating a simple `run.sh`
2. Running CPU only, no GPU
3. Using the Conda `base` environment
4. Submitting the job with `sbatch`
5. Checking job status
6. Getting the compute node name
7. Entering the node while the job is running
8. Checking output and error files
9. Cancelling or killing the job

---

## 1. Login to ENUCC

First, login to the ENUCC login node.

After login, your terminal may look like this:

```bash
[40000000@login1[enucc] ~]$
```

The login node is used for:

```text
editing files
creating scripts
submitting jobs
checking job status
reading output/error logs
```

Do not run heavy computation directly on the login node.

Use SLURM jobs for computation.

---

## 2. Load Conda

Load Anaconda:

```bash
module load apps/anaconda3/2024.10/bin
```

Enable Conda:

```bash
source "$(conda info --base)/etc/profile.d/conda.sh"
```

Check available Conda environments:

```bash
conda env list
```

If you only see:

```text
base  /opt/gridware/depots/9d06fe54/el8/pkg/apps/anaconda3/2024.10/bin
```

it means only the `base` environment is currently available.

For this first simple SLURM test, using `base` is fine.

Activate base:

```bash
conda activate base
```

Check Python:

```bash
which python
python --version
```

---

## 3. Create Log Folder

Before submitting the job, create the log folder.

This is important because SLURM needs the output folder to already exist.

```bash
mkdir -p /users/40000000/slurmlogs
```

---

## 4. Create `run.sh`

Create a file called `run.sh`:

```bash
nano run.sh
```

Paste this script:

```bash
#!/bin/bash
#SBATCH --job-name=first_cpu_job
#SBATCH --partition=nodes
#SBATCH --cpus-per-task=1
#SBATCH --mem=2G
#SBATCH --time=00:10:00
#SBATCH --output=/users/40000000/slurmlogs/first_cpu_job_%j.out
#SBATCH --error=/users/40000000/slurmlogs/first_cpu_job_%j.err

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

echo "Working directory after cd: $(pwd)"

echo ""
echo "============================================================"
echo "LOAD CONDA ENVIRONMENT"
echo "============================================================"

module load apps/anaconda3/2024.10/bin
source "$(conda info --base)/etc/profile.d/conda.sh"

# For first simple test, use base environment
conda activate base

echo "Conda environment: $CONDA_DEFAULT_ENV"
echo "Python path: $(which python)"
echo "Python version:"
python --version

echo ""
echo "============================================================"
echo "RUN SIMPLE CPU TEST"
echo "============================================================"

python - << 'EOF'
import os
import time
import platform

print("Hello from SLURM CPU job")
print("Hostname:", platform.node())
print("Current folder:", os.getcwd())

for i in range(1, 6):
    print(f"Running step {i}/5")
    time.sleep(10)

print("CPU job finished successfully")
EOF

echo ""
echo "============================================================"
echo "END JOB"
echo "============================================================"
echo "End time: $(date)"
```

Save and exit:

```text
Ctrl + O
Enter
Ctrl + X
```

---

## 5. Make `run.sh` Executable

```bash
chmod +x run.sh
```

---

## 6. Submit the Job

Submit the job using:

```bash
sbatch run.sh
```

Example output:

```text
Submitted batch job 123456
```

Here, `123456` is the SLURM job ID.

---

## 7. Check Job Status

Check your jobs:

```bash
squeue -u $USER
```

Example output:

```text
JOBID   PARTITION   NAME           USER      ST   TIME   NODES   NODELIST(REASON)
123456  nodes       first_cpu_job  40000000  R    0:20   1       node01
```

Important status codes:

```text
R   = Running
PD  = Pending / waiting
CG  = Completing
```

If the job is running, you will see a node name in the `NODELIST` column.

---

## 8. Check One Specific Job

Replace `123456` with your real job ID:

```bash
scontrol show job 123456
```

This shows detailed information about the job, including:

```text
JobId
JobName
UserId
Partition
JobState
RunTime
NodeList
WorkDir
StdOut
StdErr
```

---

## 9. Get the Compute Node Name

Use:

```bash
squeue -u $USER
```

Look at the `NODELIST` column.

Example:

```text
node01
```

You can also use:

```bash
scontrol show job 123456 | grep NodeList
```

Example output:

```text
NodeList=node01
```

---

## 10. Get to the Node While Job Is Running

If the job is still running and the HPC allows direct SSH to compute nodes, use:

```bash
ssh node01
```

Replace `node01` with the real node name from `squeue`.

Check that you are on the compute node:

```bash
hostname
```

To leave the compute node:

```bash
exit
```

Note: some HPC systems do not allow direct SSH to compute nodes. If SSH does not work, it is likely restricted by the cluster policy.

---

## 11. Alternative: Enter Running Job Allocation

Some SLURM systems allow entering the running job allocation with:

```bash
srun --jobid=123456 --pty bash
```

Replace `123456` with your real job ID.

Then check:

```bash
hostname
```

To exit:

```bash
exit
```

If this does not work, use the normal log files to monitor the job instead.

---

## 12. Check Output File

The output file is created from this line in `run.sh`:

```bash
#SBATCH --output=/users/40000000/slurmlogs/first_cpu_job_%j.out
```

`%j` becomes the job ID.

Example output file:

```text
/users/40000000/slurmlogs/first_cpu_job_123456.out
```

View the output:

```bash
cat /users/40000000/slurmlogs/first_cpu_job_123456.out
```

Follow output live while the job is running:

```bash
tail -f /users/40000000/slurmlogs/first_cpu_job_123456.out
```

---

## 13. Check Error File

The error file is created from this line in `run.sh`:

```bash
#SBATCH --error=/users/40000000/slurmlogs/first_cpu_job_%j.err
```

Example error file:

```text
/users/40000000/slurmlogs/first_cpu_job_123456.err
```

View the error file:

```bash
cat /users/40000000/slurmlogs/first_cpu_job_123456.err
```

Follow error live:

```bash
tail -f /users/40000000/slurmlogs/first_cpu_job_123456.err
```

If the error file is empty, that usually means there was no error.

---

## 14. List Log Files

```bash
ls -lh /users/40000000/slurmlogs
```

Sort by newest first:

```bash
ls -lht /users/40000000/slurmlogs
```

---

## 15. Cancel or Kill a Job

Cancel one job:

```bash
scancel 123456
```

Cancel all your jobs:

```bash
scancel -u $USER
```

Check again:

```bash
squeue -u $USER
```

If the job disappears from `squeue`, it has stopped.

---

## 16. Common SLURM Commands

Check your jobs:

```bash
squeue -u $USER
```

Submit a job:

```bash
sbatch run.sh
```

Show detailed job information:

```bash
scontrol show job 123456
```

Cancel a job:

```bash
scancel 123456
```

Cancel all your jobs:

```bash
scancel -u $USER
```

View output:

```bash
cat /users/40000000/slurmlogs/first_cpu_job_123456.out
```

View error:

```bash
cat /users/40000000/slurmlogs/first_cpu_job_123456.err
```

Follow output live:

```bash
tail -f /users/40000000/slurmlogs/first_cpu_job_123456.out
```

Follow error live:

```bash
tail -f /users/40000000/slurmlogs/first_cpu_job_123456.err
```

---

## 17. Full Simple Workflow

```bash
module load apps/anaconda3/2024.10/bin
source "$(conda info --base)/etc/profile.d/conda.sh"
conda activate base

mkdir -p /users/40000000/slurmlogs

nano run.sh
chmod +x run.sh

sbatch run.sh
squeue -u $USER

tail -f /users/40000000/slurmlogs/first_cpu_job_<JOBID>.out
```

Replace:

```text
<JOBID>
```

with your real SLURM job ID.

To cancel:

```bash
scancel <JOBID>
```

---

## 18. CPU-Only Settings Used Here

This script uses CPU only:

```bash
#SBATCH --partition=nodes
#SBATCH --cpus-per-task=1
#SBATCH --mem=2G
#SBATCH --time=00:10:00
```

No GPU is requested.

There is no line like:

```bash
#SBATCH --gpus=1
```

So this job will not use GPU.

---

## 19. When to Create Your Own Conda Environment

For this first SLURM test, `base` is okay.

For real research work, it is better to create your own environment, for example:

```bash
mkdir -p /users/40000000/conda/envs

conda create -p /users/40000000/conda/envs/python310 python=3.10 -y

conda activate /users/40000000/conda/envs/python310

pip install numpy pandas matplotlib scikit-learn tqdm jupyter ipykernel
```

Then in future SLURM scripts, replace:

```bash
conda activate base
```

with:

```bash
conda activate /users/40000000/conda/envs/python310
```

For now, if only `base` exists, use:

```bash
conda activate base
```

---

## 20. Important Notes

Submit jobs from the login node:

```bash
sbatch run.sh
```

Check jobs from the login node:

```bash
squeue -u $USER
```

Cancel jobs from the login node:

```bash
scancel <JOBID>
```

Use compute nodes for actual computation.

Do not run heavy Python scripts directly on the login node.
