## Setting Up Your First Databricks Free Edition Environment

This guide walks you through creating a free Databricks account, understanding key limitations, and building your first notebook in minutes. Screenshot placeholders are included where you can add images later.

### Prerequisites
- **Email address**: Required for signup and verification
- **Modern browser**: Chrome, Edge, Firefox, or Safari

---

### 1) Create your Databricks Free Edition account (step-by-step)
1. **Open the signup page**: Go to the Databricks Community Edition signup.
   - Link: [Databricks Community Edition signup](https://community.cloud.databricks.com/signup)
   - [screenshot: Databricks signup page]
2. **Create your account**: Provide your name, work/personal email, and set a password.
   - Accept the terms to continue.
   - [screenshot: account details form]
3. **Verify your email**: Check your inbox for a verification message and click the link.
   - If you don’t see it, check spam/junk.
   - [screenshot: email verification message]
4. **Complete initial setup**: After verification, sign in at the Community Edition login page.
   - Link: [Databricks Community Edition login](https://community.cloud.databricks.com/)
   - You’ll land on your workspace home page.
   - [screenshot: workspace home]

Notes:
- Community Edition is Databricks’ free, always-on offering intended for learning and small experiments.
- If you instead start a free trial of a paid workspace, steps and limits will differ (trial expires; CE does not).

---

### 2) Understand Free Edition limitations
Community Edition (Free Edition) is excellent for learning, but it has constraints compared to paid tiers:

- **Compute capacity**: Limited to a small single-node cluster; constrained CPU/RAM.
- **Runtime options**: Fewer Databricks Runtime versions available than paid workspaces.
- **Feature availability**: Many enterprise features are unavailable (e.g., Unity Catalog, Delta Live Tables, advanced Jobs, private networking, SCIM/SSO, secret scopes for external key vaults).
- **Libraries and integrations**: Ability to install custom libraries is limited; external data source connectivity options are restricted.
- **Storage and workspace limits**: Lower quotas for notebooks, files, and cluster usage; idle timeouts may be stricter.
- **Support and SLAs**: No enterprise support or SLAs.

These constraints are fine for tutorials, classroom work, and proofs-of-concept. For production or heavier experiments, consider a paid workspace.

---

### 3) Create your first notebook

#### A. Start (or create) a cluster
1. In the left sidebar, click **Compute**.
   - [screenshot: compute sidebar]
2. Click **Create compute** (or **Create cluster**) if one doesn’t already exist.
   - Use the default settings (name, single-node) and the default Databricks Runtime.
   - [screenshot: create cluster dialog]
3. Click **Create** and wait for the cluster state to become **Running**.
   - [screenshot: cluster running]

Tips:
- Free Edition typically allows only one small single-node cluster.
- If your cluster auto-terminates, re-start it before running notebooks.

#### B. Create a new notebook
1. Click **New** → **Notebook** (or use the New button in the top bar).
   - Name it, for example, “Hello Databricks”.
   - Choose a default language: **Python** or **SQL**.
   - Attach to your running cluster.
   - [screenshot: new notebook dialog]
2. Run a simple cell to verify everything works.
   - [screenshot: notebook first run]

Example cells you can try:

Python (in a Python notebook):
```python
print("Spark version:", spark.version)

data = [
    ("Alice", 85),
    ("Bob", 92),
    ("Carol", 78),
]

df = spark.createDataFrame(data, ["name", "score"])
display(df)

from pyspark.sql import functions as F
df.groupBy().agg(F.avg("score").alias("avg_score")).display()
```

SQL (in a SQL notebook, or in a Python/Scala notebook with a `%%sql` cell):
```sql
CREATE OR REPLACE TEMP VIEW demo AS
SELECT 1 AS id, 'Alice' AS name, 85 AS score
UNION ALL SELECT 2, 'Bob', 92
UNION ALL SELECT 3, 'Carol', 78;

SELECT * FROM demo;

SELECT AVG(score) AS avg_score FROM demo;
```

#### C. Save and share
- Use the notebook menu to **Rename**, **Export** (HTML, IPYNB), or **Clone** your work.
- In Community Edition, collaboration features are limited compared to paid tiers.

---

### 4) Helpful resources
- **Databricks Free Edition (Community Edition)**: [Sign up or log in](https://community.cloud.databricks.com/)
- **Databricks docs**:
  - [Databricks notebooks](https://docs.databricks.com/en/notebooks/index.html)
  - [Compute (clusters)](https://docs.databricks.com/en/compute/index.html)
- **Learn Python (basics)**:
  - [Official Python tutorial](https://docs.python.org/3/tutorial/)
  - [W3Schools Python](https://www.w3schools.com/python/)
- **Learn SQL (basics)**:
  - [Mode SQL Tutorial](https://mode.com/sql-tutorial/)
  - [W3Schools SQL](https://www.w3schools.com/sql/)

---

### Screenshot placeholders (for later)
- [screenshot: Databricks signup page]
- [screenshot: account details form]
- [screenshot: email verification message]
- [screenshot: workspace home]
- [screenshot: compute sidebar]
- [screenshot: create cluster dialog]
- [screenshot: cluster running]
- [screenshot: new notebook dialog]
- [screenshot: notebook first run]

You can replace these placeholders with actual images using Markdown image syntax, for example:

```markdown
![Signup page](./screenshots/signup-page.png)
```


