---
title: Setting Up Your First Databricks Free Edition Environment
sidebar_position: 1
---

This guide walks you through creating a free Databricks account, understanding key limitations, and building your first notebook in minutes.

### Prerequisites
- **Email address**: Required for signup and verification
- **Modern browser**: Chrome, Edge, Firefox, or Safari

---

1. **Open the signup page**: Go to the Databricks Free Edition signup.
   - Link: [Databricks Free Edition signup](https://www.databricks.com/learn/free-edition)
   ![@databricks-free-edition.png](/img/databricks-free-edition.png)
2. **Create your account**: Provide your name, personal email, and set a password.
   ![@databricks-free-edition-signup.png](/img/databricks-free-edition-signup.png)
3. **Verify your email**: Check your inbox for a verification message and click the link.

:::note Can't find the email?
Check your spam/junk folder if you don't see the verification message in your inbox.
:::
4. **Complete initial setup**: After verification, sign in at the Databricks Free Edition login page.
   - Link: [Databricks Free Edition login](https://www.databricks.com/learn/free-edition)
   - You’ll land on your workspace home page.
   ![@databricks-free-edition-workspace.png](/img/databricks-free-edition-workspace.png)

:::info About Databricks Free Edition
Databricks Free Edition is Databricks' free, always-on offering intended for learning and small experiments. If you instead start a free trial of a paid workspace, steps and limits will differ (trial expires; Free Edition does not).
:::

---

### 2) Free Edition vs Paid Databricks

The table below compares Databricks Free Edition with a regular paid Databricks subscription at a high level.

| Area | Free Edition | Paid Databricks (standard subscription) |
| --- | --- | --- |
| Compute resources | Serverless only; small sizes; no custom configs or GPUs ([Free Edition limitations](https://docs.databricks.com/aws/getting-started/community-edition-limitations)) | Full range of compute incl. custom configs, autoscaling multi-node clusters, GPU support |
| SQL warehouses | One warehouse limited to 2X-Small  | Multiple warehouses; many sizes |
| Jobs/Workflows | Max 5 concurrent job tasks/account  | Higher limits; advanced Workflows/Jobs |
| Lakeflow pipelines | One active pipeline per pipeline type  | Multiple active pipelines |
| Model Serving | Endpoint limits; no GPU endpoints; no provisioned throughput; limited models  | Full serving incl. GPU endpoints, provisioned throughput, custom models |
| Vector Search | One endpoint; one Vector Search unit  | Multiple endpoints; scalable units |
| Apps | One app per account; auto-stops after 24h  | Multiple apps with configurable runtimes |
| Languages | Python only (no R/Scala)  | Python, SQL, R, Scala |
| Workspace storage | No custom workspace storage locations  | Custom storage locations supported |
| Online Tables | Not supported  | Supported |
| Clean Rooms | Not supported  | Supported |
| Admin/security | One workspace + one metastore; no account console/APIs; limited auth; no SSO/SCIM; no private networking; no compliance  | Multiple workspaces/metastores; account console/APIs; SSO/SCIM; private networking; compliance options |
| Support & SLA | No support/SLA  | Enterprise support with SLAs |
| Commercial use | Non‑commercial only  | Commercial use |
| Marketplace provider | Not allowed  | Eligible |
| Cost | Free | Usage‑based billing |

Note: Exact capabilities can change and vary by cloud, region, and subscription tier. Always refer to the official Databricks documentation linked above.

---

### 3) Create your first notebook

Databricks Free Edition uses **serverless compute**, which means you don't need to create or manage clusters manually. Compute resources start automatically when you run notebook cells. ([Learn more about Serverless Compute](https://docs.databricks.com/aws/en/compute/serverless/))

#### A. Create a new notebook
1. In your workspace, click **Workspace** in the left sidebar.
2. Click **Create** → **Notebook** (or use the **Create** button).
![@workspace-notebook.png](/img/workspace-notebook.png)
   - Name it, for example, "Hello Databricks".
   - Choose a default language: **Python** or **SQL**.
3. The notebook opens immediately—no cluster attachment needed.
![@notebook.png](/img/notebook.png)
#### B. Run your first cell
1. In the first cell, enter some code (see examples below).
2. Press **Shift+Enter** or click the **Run** icon to execute the cell.
3. Serverless compute will start automatically (this may take a few seconds the first time).

#### C. Example code to try

**Python** (in a Python notebook):
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
display(df.groupBy().agg(F.avg("score").alias("avg_score")))
```

**SQL** (in a SQL notebook, or add `%sql` at the top of a cell in a Python notebook):
```sql
%sql
CREATE OR REPLACE TEMP VIEW demo AS
SELECT 1 AS id, 'Alice' AS name, 85 AS score
UNION ALL SELECT 2, 'Bob', 92
UNION ALL SELECT 3, 'Carol', 78;

SELECT * FROM demo;

SELECT AVG(score) AS avg_score FROM demo;
```

![@first-output.png](/img/first-output.png)

#### D. Save and share
- Use the notebook menu to **Rename**, **Export** (HTML, IPYNB), or **Clone** your work.
- In Free Edition, you can share notebooks, but advanced collaboration features are limited compared to paid tiers.

---

### 4) Helpful resources
- **Databricks Free Edition**: [Sign up or log in](https://www.databricks.com/learn/free-edition)
- **Databricks docs**:
  - [Databricks notebooks](https://docs.databricks.com/aws/en/notebooks/)
  - [Compute (clusters)](https://docs.databricks.com/aws/en/compute/)
- **Learn Python (basics)**:
  - [Official Python tutorial](https://docs.python.org/3/tutorial/)
  - [W3Schools Python](https://www.w3schools.com/python/)
- **Learn SQL (basics)**:
  - [Mode SQL Tutorial](https://mode.com/sql-tutorial/)
  - [W3Schools SQL](https://www.w3schools.com/sql/)
