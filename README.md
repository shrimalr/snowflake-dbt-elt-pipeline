# End-to-End Cloud ELT & Analytics Engineering Pipeline

A production-style modern data engineering pipeline demonstrating automated cloud ingestion from **Python (`boto3`)** into **AWS S3**, bulk loading into **Snowflake**, and modular 3-tier transformations with **dbt Core**.

---

## 🏗️ Architecture Overview

```text
+-------------------+       +-------------------+       +-----------------------+
|  Python (`boto3`) | ----> |    AWS S3 Bucket  | ----> |  Snowflake RAW Layer  |
|  Data Extraction  |       |  Partitioned CSV  |       |  External Stage/COPY  |
+-------------------+       +-------------------+       +-----------------------+
                                                                    |
                                                                    v
+-------------------+       +-------------------+       +-----------------------+
|   BI / Reporting  | <---- |  dbt Marts Layer  | <---- |   dbt Staging Layer   |
|  (Fact & Dim)     |       | Incremental/Table |       |  Views + YAML Tests   |
+-------------------+       +-------------------+       +-----------------------+
```

---

## 📂 Repository Structure

```text
snowflake-dbt-elt-pipeline/
├── ingestion/
│   ├── extract_and_upload.py      # Extracts transactional records & uploads partitioned CSVs to AWS S3
│   └── load_to_snowflake.sql      # Configures Snowflake External Stage & executes COPY INTO bulk load
├── dbt_transform/
│   ├── models/
│   │   ├── staging/
│   │   │   ├── stg_orders.sql     # Standardized view over raw Snowflake source
│   │   │   └── schema.yml         # Source freshness SLAs & automated data quality tests
│   │   ├── intermediate/
│   │   │   └── int_customer_orders.sql  # Ephemeral customer-level aggregations
│   │   └── marts/
│   │       ├── fct_orders.sql     # Incremental fact table with unique_key merge protection
│   │       └── dim_customers.sql  # Dimensional customer segmentation table
│   └── dbt_project.yml            # Layer-wise materialization & schema configuration
├── .env.example                   # Template for environment credentials
├── .gitignore                     # Excludes secrets, virtual environments, and dbt artifacts
├── requirements.txt               # Python dependencies
└── README.md
```

---

## ⚙️ Key Engineering Highlights

* **Partitioned Cloud Staging (AWS S3):** Raw files are organized by ingestion date (`raw/orders/YYYY/MM/DD/`) using `boto3` to support scalable, idempotent batch ingestion.
* **Optimized Warehouse Loading (Snowflake):** Uses Snowflake External Stages and `COPY INTO` pattern matching for high-throughput ingestion.
* **3-Tier dbt Architecture:**
  * **Staging (`view`):** Cleans data types, standardizes casing, and enforces `source()` freshness checks.
  * **Intermediate (`ephemeral`):** Encapsulates reusable customer aggregation logic without cluttering the warehouse.
  * **Marts (`table` & `incremental`):** Builds analytics-ready Fact (`fct_orders`) and Dimension (`dim_customers`) models using incremental updates to minimize compute overhead.
* **Automated Data Quality Guardrails:** Enforces `unique` and `not_null` constraints via `dbt test` to prevent duplicate keys and schema drift.

---

## 🚀 Quickstart Guide

### 1. Clone & Set Up Virtual Environment
```bash
git clone https://github.com/<YOUR_GITHUB_USERNAME>/snowflake-dbt-elt-pipeline.git
cd snowflake-dbt-elt-pipeline
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 2. Configure Credentials
Copy `.env.example` to `.env` and populate your AWS and Snowflake credentials:
```bash
cp .env.example .env
```
Ensure your local `~/.dbt/profiles.yml` includes a `snowflake_dbt_elt` profile pointing to `ENTERPRISE_ELT_DB`.

### 3. Run Ingestion & Transformations
```bash
# Step 1: Extract data and upload to AWS S3
python ingestion/extract_and_upload.py

# Step 2: Execute ingestion/load_to_snowflake.sql in Snowflake Worksheet

# Step 3: Run dbt models and data quality tests
cd dbt_transform
dbt debug
dbt source freshness
dbt run
dbt test
```
