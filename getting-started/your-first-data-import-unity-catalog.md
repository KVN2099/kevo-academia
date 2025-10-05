---
id: your-first-data-import-unity-catalog
title: Your First Data Import — Kaggle to Databricks (Unity Catalog)
description: Download a Kaggle dataset programmatically, upload it to a Unity Catalog Volume, and explore it in Databricks with Spark.
sidebar_position: 2
---

This guide will walk you through the steps to download a Kaggle dataset, save it to a Unity Catalog Volume, and run basic data analysis.

::::tip What you’ll achieve
- Download a dataset from Kaggle using the Kaggle API
- Create Unity Catalog objects (catalog, schema, volume)
- Upload the data to a Unity Catalog Volume
- Explore the data with Spark (schema, quality checks, basic stats, quick visuals)
::::

## Prerequisites
- Kaggle account
- A Databricks workspace with Unity Catalog enabled (you need permission to create catalog, schema, and volume)
- A Databricks Personal Access Token (PAT)

::::warning Using Free Edition?
Unity Catalog is not available in Databricks Free Edition. If you are on Free Edition, use the DBFS-based workflow instead (see `getting-started/setting-up-your-first-databricks-free-edition-environment`).
::::

## Step 0 — Choose a small, friendly dataset
We’ll use the classic Iris dataset from Kaggle because it’s tiny and clean. You can swap it later for any dataset you like.

- Dataset slug we’ll use: `uciml/iris`
- Local working folder we’ll use: `data/iris`
- Target Volume path we’ll use: `/Volumes/demo/raw/iris_data`

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

::::note Docs
Unity Catalog Volumes: https://docs.databricks.com/en/volumes/index.html
::::

## Step 4 — Create Unity Catalog objects (catalog, schema, volume)

Run the following in a Databricks notebook cell with `%sql` to set up a managed Volume.

```sql
%sql
CREATE CATALOG IF NOT EXISTS demo;
CREATE SCHEMA IF NOT EXISTS demo.raw;
-- Managed volume (storage managed by Databricks)
CREATE VOLUME IF NOT EXISTS demo.raw.iris_data;
```

## Step 5 — Upload local files to the Volume

Upload a single file with the CLI (note the `dbfs:/Volumes/...` prefix when using the CLI):
```bash
databricks fs cp --overwrite ./data/iris/iris.csv dbfs:/Volumes/demo/raw/iris_data/iris.csv --profile kevo-academia
```

Upload a whole folder recursively:
```bash
databricks fs cp --overwrite --recursive ./data/iris dbfs:/Volumes/demo/raw/iris_data --profile kevo-academia
```

::::tip Quick visual check
Use a notebook cell to list the Volume contents: `display(dbutils.fs.ls("/Volumes/demo/raw/iris_data"))`.
::::

## Step 6 — Explore the data in a Databricks notebook (PySpark)

Create a new Notebook in Databricks with the Python language and attach it to a running compute (serverless or cluster, depending on your workspace).

### 6.1 Load the CSV from the Volume
```python
from pyspark.sql import functions as F

df = (
    spark.read
        .option("header", "true")
        .option("inferSchema", "true")
        .csv("/Volumes/demo/raw/iris_data/iris.csv")
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
SELECT * FROM csv.`/Volumes/demo/raw/iris_data/iris.csv`
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

# Copy into the Unity Catalog Volume for durable storage/access
dbutils.fs.mkdirs("/Volumes/demo/raw/iris_data")
dbutils.fs.cp(
    f"file:{local_dir}/iris.csv",
    "/Volumes/demo/raw/iris_data/iris.csv",
    recurse=False,
)
```

## Clean up (optional)
Remove data from the Volume if you no longer need it:
```bash
databricks fs rm -r dbfs:/Volumes/demo/raw/iris_data --profile kevo-academia
```

## Troubleshooting
- Kaggle 403/401: Re-check `~/.kaggle/kaggle.json` placement and permissions or set `KAGGLE_USERNAME` and `KAGGLE_KEY` as env vars.
- CLI not found: Ensure `pipx install databricks` (or `pip install databricks`) completed and your shell PATH includes the scripts directory.
- Upload fails: Confirm your profile is logged in (`databricks auth profiles`) and the `--host` matches your workspace URL. Ensure you have permissions on the catalog, schema, and volume.
- Big files: Use `--recursive` and avoid large unzipped archives in your repo; keep raw data out of Git.

## What’s next?
- Swap in a bigger Kaggle dataset
- Save DataFrame as Delta for faster analytics (`df.write.format("delta").save("/Volumes/demo/raw/iris_data/iris_delta")`)
- Build simple ML with Spark MLlib or run SQL dashboards


