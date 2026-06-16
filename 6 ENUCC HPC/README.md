# Example Workflow in JupyterLab on HPC

This guide shows a simple example of working in JupyterLab on HPC.

It includes:

```text
1. Upload data from your local PC to HPC
2. Open JupyterLab
3. Create a notebook
4. Load CSV data
5. Load public example data
6. Do simple visual analysis
```

This guide assumes your Conda environment already exists:

```text
/users/40020957/conda/envs/python310
```

and JupyterLab already works through SLURM.

---

## 1. Prepare a Project Folder on HPC

Login to ENUCC:

```bash
ssh ENUCC
```

Create a project folder:

```bash
mkdir -p /users/40020957/TTHER/jupyter_example
mkdir -p /users/40020957/TTHER/jupyter_example/data
```

Go to the project folder:

```bash
cd /users/40020957/TTHER/jupyter_example
```

---

## 2. Upload CSV Data from Your PC to HPC

Assume you have a CSV file on your Mac or PC:

```text
my_data.csv
```

On your local machine, open Terminal and run:

```bash
scp my_data.csv ENUCC:/users/40020957/TTHER/jupyter_example/data/
```

If your file is in Downloads:

```bash
scp ~/Downloads/my_data.csv ENUCC:/users/40020957/TTHER/jupyter_example/data/
```

If you do not use the `ENUCC` shortcut, use:

```bash
scp -o ProxyJump=40020957@gateway.napier.ac.uk \
    ~/Downloads/my_data.csv \
    40020957@login.enucc.napier.ac.uk:/users/40020957/TTHER/jupyter_example/data/
```

After upload, check the file on HPC:

```bash
ls -lh /users/40020957/TTHER/jupyter_example/data/
```

You should see:

```text
my_data.csv
```

---

## 3. Start JupyterLab Using SLURM

Submit your Jupyter SLURM script.

For CPU JupyterLab:

```bash
sbatch run_jupyter_cpu.sh
```

For GPU JupyterLab:

```bash
sbatch run_jupyter_gpu.sh
```

Check job status:

```bash
squeue -u 40020957
```

When the job is running, read the output file:

```bash
ls -lt /users/40020957/slurmlogs/
```

Example:

```bash
cat /users/40020957/slurmlogs/jupyter_gpu_JOBID.out
```

Copy the SSH tunnel command from the `.out` file.

Example:

```bash
ssh -N -L 8765:gpu01.enucc.enu.alces.network:8765 ENUCC
```

Run that command on your local Mac terminal.

Then open:

```text
http://localhost:8765
```

Copy the token from the `.out` file and paste it into the browser.

---

## 4. Create a New Notebook

Inside JupyterLab:

```text
File -> New -> Notebook
```

Choose the Python kernel:

```text
Python 3.10 (python310)
```

You can rename the notebook:

```text
example_visual_analysis.ipynb
```

---

# Part A: Load CSV Data Uploaded from Your PC

## 5. Import Python Libraries

In the first notebook cell:

```python
import os
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
```

---

## 6. Load Your Uploaded CSV File

Set the file path:

```python
CSV_PATH = "/users/40020957/TTHER/jupyter_example/data/my_data.csv"

df = pd.read_csv(CSV_PATH)

print("Shape:", df.shape)
df.head()
```

Check column names:

```python
df.columns
```

Check basic information:

```python
df.info()
```

Check missing values:

```python
df.isna().sum()
```

---

## 7. Basic Dataset Summary

Show simple statistics:

```python
df.describe()
```

Show number of rows and columns:

```python
print("Number of rows:", df.shape[0])
print("Number of columns:", df.shape[1])
```

Show data types:

```python
df.dtypes
```

---

## 8. Example Visual Analysis

Choose one numeric column from your dataset.

Example:

```python
numeric_cols = df.select_dtypes(include=["number"]).columns

print("Numeric columns:")
print(numeric_cols)
```

Plot histogram for the first numeric column:

```python
col = numeric_cols[0]

plt.figure(figsize=(8, 5))
plt.hist(df[col].dropna(), bins=30)
plt.title(f"Distribution of {col}")
plt.xlabel(col)
plt.ylabel("Frequency")
plt.grid(True)
plt.show()
```

Plot boxplot:

```python
plt.figure(figsize=(6, 5))
plt.boxplot(df[col].dropna())
plt.title(f"Boxplot of {col}")
plt.ylabel(col)
plt.grid(True)
plt.show()
```

Plot line graph:

```python
plt.figure(figsize=(10, 5))
plt.plot(df[col].dropna().values)
plt.title(f"Line Plot of {col}")
plt.xlabel("Index")
plt.ylabel(col)
plt.grid(True)
plt.show()
```

---

## 9. Correlation Heatmap Without Seaborn

Select only numeric columns:

```python
numeric_df = df.select_dtypes(include=["number"])

corr = numeric_df.corr()

corr
```

Plot correlation matrix:

```python
plt.figure(figsize=(8, 6))
plt.imshow(corr, aspect="auto")
plt.colorbar(label="Correlation")
plt.xticks(range(len(corr.columns)), corr.columns, rotation=90)
plt.yticks(range(len(corr.columns)), corr.columns)
plt.title("Correlation Matrix")
plt.tight_layout()
plt.show()
```

---

# Part B: Load Public Example Data

This part uses the public Iris dataset from `scikit-learn`.

It does not need your own CSV file.

---

## 10. Load Public Iris Dataset

In a new notebook cell:

```python
from sklearn.datasets import load_iris

iris = load_iris(as_frame=True)

df_iris = iris.frame

df_iris.head()
```

Check the shape:

```python
print("Shape:", df_iris.shape)
```

Check columns:

```python
df_iris.columns
```

The target class is stored in:

```python
target
```

Add readable class names:

```python
df_iris["species"] = df_iris["target"].map(
    dict(enumerate(iris.target_names))
)

df_iris.head()
```

---

## 11. Summary of Public Dataset

Check number of samples per class:

```python
df_iris["species"].value_counts()
```

Summary statistics:

```python
df_iris.describe()
```

---

## 12. Visual Analysis of Public Dataset

Plot class count:

```python
class_counts = df_iris["species"].value_counts()

plt.figure(figsize=(7, 5))
plt.bar(class_counts.index, class_counts.values)
plt.title("Number of Samples per Iris Species")
plt.xlabel("Species")
plt.ylabel("Count")
plt.grid(axis="y")
plt.show()
```

Plot sepal length distribution:

```python
plt.figure(figsize=(8, 5))
plt.hist(df_iris["sepal length (cm)"], bins=20)
plt.title("Distribution of Sepal Length")
plt.xlabel("Sepal Length (cm)")
plt.ylabel("Frequency")
plt.grid(True)
plt.show()
```

Plot scatter graph:

```python
plt.figure(figsize=(8, 6))

for species in df_iris["species"].unique():
    subset = df_iris[df_iris["species"] == species]
    plt.scatter(
        subset["sepal length (cm)"],
        subset["sepal width (cm)"],
        label=species
    )

plt.title("Sepal Length vs Sepal Width")
plt.xlabel("Sepal Length (cm)")
plt.ylabel("Sepal Width (cm)")
plt.legend()
plt.grid(True)
plt.show()
```

Plot petal length vs petal width:

```python
plt.figure(figsize=(8, 6))

for species in df_iris["species"].unique():
    subset = df_iris[df_iris["species"] == species]
    plt.scatter(
        subset["petal length (cm)"],
        subset["petal width (cm)"],
        label=species
    )

plt.title("Petal Length vs Petal Width")
plt.xlabel("Petal Length (cm)")
plt.ylabel("Petal Width (cm)")
plt.legend()
plt.grid(True)
plt.show()
```

---

# Part C: Save Results from JupyterLab

## 13. Save a Processed CSV File

Create output folder:

```python
OUTPUT_DIR = "/users/40020957/TTHER/jupyter_example/results"

os.makedirs(OUTPUT_DIR, exist_ok=True)
```

Save processed Iris data:

```python
output_csv = os.path.join(OUTPUT_DIR, "iris_processed.csv")

df_iris.to_csv(output_csv, index=False)

print("Saved:", output_csv)
```

---

## 14. Save a Figure

Create and save a figure:

```python
output_fig = os.path.join(OUTPUT_DIR, "iris_petal_scatter.png")

plt.figure(figsize=(8, 6))

for species in df_iris["species"].unique():
    subset = df_iris[df_iris["species"] == species]
    plt.scatter(
        subset["petal length (cm)"],
        subset["petal width (cm)"],
        label=species
    )

plt.title("Petal Length vs Petal Width")
plt.xlabel("Petal Length (cm)")
plt.ylabel("Petal Width (cm)")
plt.legend()
plt.grid(True)
plt.savefig(output_fig, dpi=300, bbox_inches="tight")
plt.show()

print("Saved:", output_fig)
```

Check files from terminal:

```bash
ls -lh /users/40020957/TTHER/jupyter_example/results/
```

---

# Part D: Download Results Back to Your PC

On your local machine, download the results:

```bash
scp -r ENUCC:/users/40020957/TTHER/jupyter_example/results ~/Downloads/
```

Or download one file:

```bash
scp ENUCC:/users/40020957/TTHER/jupyter_example/results/iris_processed.csv ~/Downloads/
```

---

# Full Notebook Example

You can copy this into one Jupyter notebook.

```python
import os
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import load_iris

# =========================
# PROJECT PATHS
# =========================

PROJECT_DIR = "/users/40020957/TTHER/jupyter_example"
DATA_DIR = os.path.join(PROJECT_DIR, "data")
OUTPUT_DIR = os.path.join(PROJECT_DIR, "results")

os.makedirs(DATA_DIR, exist_ok=True)
os.makedirs(OUTPUT_DIR, exist_ok=True)

# =========================
# OPTION 1: LOAD YOUR OWN CSV
# =========================

CSV_PATH = os.path.join(DATA_DIR, "my_data.csv")

if os.path.exists(CSV_PATH):
    df = pd.read_csv(CSV_PATH)

    print("Loaded local CSV:")
    print("Shape:", df.shape)
    display(df.head())

    print("Columns:")
    print(df.columns)

    print("Missing values:")
    display(df.isna().sum())

    numeric_cols = df.select_dtypes(include=["number"]).columns

    if len(numeric_cols) > 0:
        col = numeric_cols[0]

        plt.figure(figsize=(8, 5))
        plt.hist(df[col].dropna(), bins=30)
        plt.title(f"Distribution of {col}")
        plt.xlabel(col)
        plt.ylabel("Frequency")
        plt.grid(True)
        plt.show()
    else:
        print("No numeric columns found in your CSV.")
else:
    print("No local CSV found at:")
    print(CSV_PATH)
    print("Skipping local CSV example.")

# =========================
# OPTION 2: LOAD PUBLIC DATASET
# =========================

iris = load_iris(as_frame=True)
df_iris = iris.frame

df_iris["species"] = df_iris["target"].map(
    dict(enumerate(iris.target_names))
)

print("Loaded public Iris dataset:")
print("Shape:", df_iris.shape)
display(df_iris.head())

# =========================
# SIMPLE SUMMARY
# =========================

print("Class counts:")
display(df_iris["species"].value_counts())

print("Statistics:")
display(df_iris.describe())

# =========================
# VISUAL ANALYSIS
# =========================

class_counts = df_iris["species"].value_counts()

plt.figure(figsize=(7, 5))
plt.bar(class_counts.index, class_counts.values)
plt.title("Number of Samples per Iris Species")
plt.xlabel("Species")
plt.ylabel("Count")
plt.grid(axis="y")
plt.show()

plt.figure(figsize=(8, 5))
plt.hist(df_iris["sepal length (cm)"], bins=20)
plt.title("Distribution of Sepal Length")
plt.xlabel("Sepal Length (cm)")
plt.ylabel("Frequency")
plt.grid(True)
plt.show()

plt.figure(figsize=(8, 6))

for species in df_iris["species"].unique():
    subset = df_iris[df_iris["species"] == species]
    plt.scatter(
        subset["petal length (cm)"],
        subset["petal width (cm)"],
        label=species
    )

plt.title("Petal Length vs Petal Width")
plt.xlabel("Petal Length (cm)")
plt.ylabel("Petal Width (cm)")
plt.legend()
plt.grid(True)

output_fig = os.path.join(OUTPUT_DIR, "iris_petal_scatter.png")
plt.savefig(output_fig, dpi=300, bbox_inches="tight")
plt.show()

# =========================
# SAVE OUTPUT CSV
# =========================

output_csv = os.path.join(OUTPUT_DIR, "iris_processed.csv")

df_iris.to_csv(output_csv, index=False)

print("Saved CSV:", output_csv)
print("Saved figure:", output_fig)
```

---

# Summary

Typical JupyterLab workflow on HPC:

```text
1. Upload data from PC to HPC using scp
2. Start JupyterLab using sbatch
3. Open JupyterLab in browser through SSH tunnel
4. Create a notebook
5. Load CSV using pandas
6. Explore data with df.head(), df.info(), df.describe()
7. Plot graphs with matplotlib
8. Save results to HPC folder
9. Download results back to PC using scp
```

Useful commands:

```bash
scp ~/Downloads/my_data.csv ENUCC:/users/40020957/TTHER/jupyter_example/data/

sbatch run_jupyter_gpu.sh

squeue -u 40020957

cat /users/40020957/slurmlogs/jupyter_gpu_JOBID.out

scp -r ENUCC:/users/40020957/TTHER/jupyter_example/results ~/Downloads/
```
