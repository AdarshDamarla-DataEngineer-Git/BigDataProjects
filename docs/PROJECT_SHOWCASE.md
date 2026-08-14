# Project showcase

## One-line summary

Built an Azure e-commerce data platform that ingests more than 1.5 million Olist records from HTTP, SQL, and MongoDB sources, transforms them with PySpark, and serves curated Parquet through Synapse Serverless SQL.

## Portfolio description

This project implements a multi-source Azure analytics pipeline for the Brazilian Olist marketplace dataset. Azure Data Factory uses metadata-driven ingestion to land GitHub CSV and SQL data in ADLS Gen2. Azure Databricks cleans eight source datasets, engineers delivery-performance metrics, and joins orders with customers, payments, items, products, sellers, and MongoDB-hosted category translations. Synapse Serverless SQL exposes the enriched Parquet through lake views and materializes delivered-order data into a Gold external table with CETAS.

## Resume bullets

- Engineered an Azure data platform processing approximately 1.56 million Olist e-commerce source rows across nine datasets.
- Implemented metadata-driven Azure Data Factory ingestion from GitHub HTTP and database sources into ADLS Gen2.
- Built PySpark transformations for null/duplicate handling, delivery-delay feature engineering, and multi-table marketplace enrichment.
- Published query-ready Parquet through Synapse Serverless SQL using `OPENROWSET`, external data sources, and CETAS for BI consumption.

## Interview talking points

### Why use ADF metadata-driven ingestion?

One parameterized pattern can ingest many source files. Adding another dataset becomes primarily a metadata change instead of a duplicated activity chain.

### Why use MongoDB for category enrichment?

It demonstrates combining relational/file-based operational data with a small NoSQL reference collection during Spark processing.

### Why use Synapse Serverless?

It queries Parquet directly in ADLS without provisioning a dedicated warehouse and supports views and CETAS-based serving patterns.

### What would you improve next?

Use explicit schemas and dataset-specific expectations, model separate fact grains, store curated data in Delta, parameterize infrastructure, add automated tests, and deploy through CI/CD.

## Suggested GitHub topics

```text
azure-data-factory
azure-databricks
adls-gen2
azure-synapse
pyspark
mongodb
mysql
parquet
olist
ecommerce-data
data-engineering
big-data
cetas
serverless-sql
```

## Suggested repository description

End-to-end Olist e-commerce pipeline using Azure Data Factory, ADLS Gen2, Azure Databricks/PySpark, MongoDB enrichment, and Synapse Serverless SQL.
