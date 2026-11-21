# Amazon Reviews Data Pipeline

A three-notebook PySpark workflow for cleaning Amazon Electronics reviews, enriching them with product metadata, and writing a curated feature dataset to Azure Data Lake Storage.

**PySpark · Azure Databricks · ADLS Gen2 · Parquet**

## Pipeline

```text
Review Parquet → validation and text cleaning → clean_reviews/
Metadata JSON ──────────────────────────────→ product join → enriched_reviews/
                                                           → features_v1/
```

| Notebook | Responsibility |
| --- | --- |
| [`01_load_and_clean_reviews.ipynb`](01_load_and_clean_reviews.ipynb) | Require product/reviewer IDs, valid 1–5 ratings, and meaningful review text |
| [`02_enrich_with_metadata.ipynb`](02_enrich_with_metadata.ipynb) | Left-join product title, brand, and price by `asin`; persist enriched output |
| [`03_write_gold_features_v1.ipynb`](03_write_gold_features_v1.ipynb) | Derive review year and write selected review/product fields as Parquet |

## Run in Databricks

1. Create an Azure Databricks workspace with access to your ADLS Gen2 account.
2. Make the reviews available as Parquet under `processed/reviews/` and the repaired metadata under `raw/meta_Electronics_fixed.json`.
3. Import the notebooks and run them in the numbered order, in the same Python session.
4. Set `storage_account_name` to your account name in the first notebook. Configure storage access through your workspace; do not paste account keys into notebook source.
5. Inspect row counts, schemas, and samples after each stage.

The [ingestion runbook](https://github.com/MohZak-AI/amazon-electronics-ingestion) describes source metadata repair. The review JSON-to-Parquet ingestion step must be completed before this pipeline begins.

Outputs use overwrite mode. Use a dedicated development prefix before pointing the notebooks at shared storage.

## Design choices and limitations

The metadata join retains reviews without matching product records. Missing product fields remain null. Metadata is expected to contain at most one row per `asin`; duplicate metadata keys can multiply review rows and should be checked before running.

The stored outputs are **Parquet**. This implementation does not claim Delta Lake transactions or time travel. Storage resources and source datasets are not bundled.
