---
id: your-first-data-import-dbfs
title: Your First Data Import — Kaggle to Databricks (DBFS, Free Edition)
description: Download a Kaggle dataset programmatically, upload it to DBFS, and explore it in Databricks with Spark. Works on Databricks Free Edition.
sidebar_position: 3
---

This guide will walk you through the steps to download a Kaggle dataset, save it to DBFS, and run basic data analysis. It works in Databricks Free Edition (Unity Catalog is not available there).

:::::tip What you’ll achieve
- Download a dataset from Kaggle using the Kaggle API
- Create a DBFS folder for your data
- Upload the data to DBFS
- Explore the data with Spark (schema, quality checks, basic stats, quick visuals)
:::::

## Prerequisites
- Kaggle account
- A Databricks workspace (Free Edition works)
- A Databricks Personal Access Token (PAT) if you plan to use the local Databricks CLI

:::::note Free Edition friendly
This guide uses DBFS paths (not Unity Catalog) so it works in Databricks Free Edition. If you have Unity Catalog and prefer Volumes, see the Unity Catalog version of this guide.
:::::

::::warning Using Free Edition?
Unity Catalog is not available in Databricks Free Edition. If you are on Free Edition, use the DBFS-based workflow instead (see `getting-started/setting-up-your-first-databricks-free-edition-environment`).
::::

## Step 0 — Choose a small, friendly dataset
We’ll use the classic Iris dataset from Kaggle because it’s tiny and clean. You can swap it later for any dataset you like.

- Dataset slug we’ll use: `uciml/iris`
- Local working folder we’ll use: `data/iris`
- Target DBFS path we’ll use: `dbfs:/FileStore/iris`

## Step 1 — Install and set up the Kaggle API

### 1.1 Install the Kaggle package (CLI included)

Mac/Linux/Windows (PowerShell):
```bash
pip install kaggle
```

### 1.2 Create your Kaggle API token
1) Go to Kaggle → Account → API → Create New API Token. This downloads a file named `kaggle.json`.

2) Place `kaggle.json` as follows:

- macOS/Linux: `~/.kaggle/kaggle.json`
- Windows: `%USERPROFILE%\.kaggle\kaggle.json`

3) Set permissions (macOS/Linux):
```bash
chmod 600 ~/.kaggle/kaggle.json
```

::::warning Keep your secrets safe
Never commit `kaggle.json` to Git. Treat it like a password.
::::

### 1.3 Verify the CLI works
```bash
kaggle datasets list -s iris | head -n 5
```
You should see a few Iris datasets listed. If not, re-check `kaggle.json` placement and permissions.

## Step 2 — Download the dataset locally

We’ll place files under `data/iris`.

macOS/Linux:
```bash
mkdir -p data/iris
kaggle datasets download -d uciml/iris -p data/iris --force
unzip -o data/iris/iris.zip -d data/iris
rm data/iris/iris.zip
ls -lah data/iris
```

Windows (PowerShell):
```powershell
mkdir data\iris
kaggle datasets download -d uciml/iris -p data\iris --force
Expand-Archive -Force data\iris\iris.zip -DestinationPath data\iris
Remove-Item data\iris\iris.zip
Get-ChildItem data\iris | Format-Table
```

::::tip Try another dataset later
Swap `uciml/iris` with your favorite dataset’s slug, e.g., `zynicide/wine-reviews`.
::::

## Step 3 — Install the Databricks CLI (modern v0)

The unified `databricks` CLI (v0) is the current recommended tool.

Install with pip (or pipx):
```bash
pip install databricks
# or, recommended
pipx install databricks
```

Verify install:
```bash
databricks -v
```

Log in (creates a local profile):
```bash
databricks auth login --host https://<your-workspace-host> --profile kevo-academia
```

When prompted, choose Personal Access Token and paste the PAT you generated in Databricks.

:::::note Docs
DBFS: https://docs.databricks.com/en/dbfs/index.html
:::::

## Step 4 — Create a DBFS folder for your data

Create the target folder in DBFS (either from the CLI or a notebook). This step ensures the destination exists and is easy to browse later.

CLI option:
```bash
databricks fs mkdirs dbfs:/FileStore/iris --profile kevo-academia
```

Notebook option (Python cell):
```python
dbutils.fs.mkdirs("dbfs:/FileStore/iris")
```

## Step 5 — Upload local files to DBFS

Upload a single file with the CLI (use the `dbfs:/...` prefix):
```bash
databricks fs cp --overwrite ./data/iris/iris.csv dbfs:/FileStore/iris/iris.csv --profile kevo-academia
```

Upload a whole folder recursively:
```bash
databricks fs cp --overwrite --recursive ./data/iris dbfs:/FileStore/iris --profile kevo-academia
```

:::::tip Quick visual check
Use a notebook cell to list the folder contents: `display(dbutils.fs.ls("dbfs:/FileStore/iris"))`.
:::::

## Step 6 — Explore the data in a Databricks notebook (PySpark)

Create a new Notebook in Databricks with the Python language and attach it to a running compute (serverless or cluster, depending on your workspace).

### 6.1 Load the CSV from DBFS
```python
from pyspark.sql import functions as F

df = (
    spark.read
        .option("header", "true")
        .option("inferSchema", "true")
        .csv("dbfs:/FileStore/iris/iris.csv")
)

df.printSchema()
df.show(5, truncate=False)
```

### 6.2 Basic quality checks
```python
# Count rows
print("Rows:", df.count())

# Null counts by column
null_counts = df.select([F.count(F.when(F.col(c).isNull(), c)).alias(c) for c in df.columns])
null_counts.show(vertical=True, truncate=False)

# Distinct values per column (for small columns)
for c in df.columns:
    distinct_cnt = df.select(c).distinct().count()
    print(f"{c}: {distinct_cnt} distinct values")
```

### 6.3 Descriptive statistics and quick aggregations
```python
df.describe().show(truncate=False)

(
    df.groupBy("species")
      .agg(
          F.count("*").alias("count"),
          F.avg("sepal_length").alias("avg_sepal_length"),
          F.avg("sepal_width").alias("avg_sepal_width"),
          F.avg("petal_length").alias("avg_petal_length"),
          F.avg("petal_width").alias("avg_petal_width"),
      )
      .orderBy(F.desc("count"))
      .show(truncate=False)
)
```

### 6.4 Visualize
Use the built-in charting in Databricks: click the chart icon above a table output, choose Bar/Histogram, and configure X/Y.

Or do a quick pandas plot (fine for tiny datasets like Iris):
```python
pdf = df.toPandas()
axes = pdf.hist(figsize=(10, 6))
```

### 6.5 SQL alternative (in a `%sql` cell)
```sql
CREATE OR REPLACE TEMP VIEW iris AS
SELECT * FROM csv.`dbfs:/FileStore/iris/iris.csv`
OPTIONS('header'='true', 'inferSchema'='true');

SELECT species, COUNT(*) AS cnt
FROM iris
GROUP BY species
ORDER BY cnt DESC;
```

## (Alternative) Download directly inside a Databricks notebook

If your workspace allows outbound internet and you prefer not to use the local CLI, you can download from Kaggle directly in a notebook.

::::warning Handle secrets carefully
Do NOT hardcode credentials. Prefer [Databricks Secrets](https://docs.databricks.com/en/security/secrets/index.html). For demo only:
::::

```python
import os

# Demo-only: set Kaggle credentials from environment variables or a secret scope
os.environ["KAGGLE_USERNAME"] = "<your-kaggle-username>"
os.environ["KAGGLE_KEY"] = "<your-kaggle-key>"

# Install kaggle package in the cluster environment
%pip install kaggle

# Create a temp download folder on the driver
local_dir = "/databricks/driver/data/iris"
import os
os.makedirs(local_dir, exist_ok=True)

# Use shell commands from the notebook
dataset = "uciml/iris"
_ = os.system(f"kaggle datasets download -d {dataset} -p {local_dir} --force")
_ = os.system(f"unzip -o {local_dir}/iris.zip -d {local_dir}")

# Copy into DBFS for durable storage/access
dbutils.fs.mkdirs("dbfs:/FileStore/iris")
dbutils.fs.cp(
    f"file:{local_dir}/iris.csv",
    "dbfs:/FileStore/iris/iris.csv",
    recurse=False,
)
```

## Troubleshooting
- Kaggle 403/401: Re-check `~/.kaggle/kaggle.json` placement and permissions or set `KAGGLE_USERNAME` and `KAGGLE_KEY` as env vars.
- CLI not found: Ensure `pipx install databricks` (or `pip install databricks`) completed and your shell PATH includes the scripts directory.
- Upload fails: Confirm your profile is logged in (`databricks auth profiles`) and the `--host` matches your workspace URL (for Free Edition it's often `https://community.cloud.databricks.com`). Verify the `dbfs:/FileStore/iris` path exists or create it with `mkdirs`.
- Big files: Use `--recursive` and avoid large unzipped archives in your repo; keep raw data out of Git.

## What’s next?
- Swap in a bigger Kaggle dataset
- Save DataFrame as Delta for faster analytics (`df.write.format("delta").save("dbfs:/FileStore/iris/iris_delta")`)
- Build simple ML with Spark MLlib or run SQL dashboards

## Clean up (optional)
Remove data from DBFS if you no longer need it:
```bash
databricks fs rm -r dbfs:/FileStore/iris --profile kevo-academia
```