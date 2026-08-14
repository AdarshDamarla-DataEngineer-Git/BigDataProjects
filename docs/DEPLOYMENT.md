# Deployment and operations

## Azure resource checklist

1. Create an ADLS Gen2 account and container with Raw, Silver, and Gold paths.
2. Create Azure Data Factory and configure HTTP/database source connections.
3. Create Azure Databricks and grant it lake access.
4. Create Azure Synapse and grant its managed identity lake permissions.
5. Optionally create MySQL and MongoDB services for payment and category sources.
6. Store credentials in Key Vault, secret scopes, or managed-identity configurations.

## Suggested lake layout

```text
olistdata/
├── raw/
│   ├── customers/
│   ├── orders/
│   ├── order_items/
│   ├── payments/
│   ├── products/
│   ├── sellers/
│   ├── reviews/
│   └── geolocation/
├── silver/
│   └── enriched_orders/
└── gold/
    └── FinalServing/
```

## Deployment order

1. Configure ADF metadata and source/sink datasets.
2. Run the source ingestion pipeline and validate raw file counts.
3. Prepare MySQL/MongoDB data if those source paths are used.
4. Configure Databricks paths and secrets.
5. Run the PySpark transformation and validate Silver output.
6. Configure Synapse database credentials, file format, and external data source.
7. Create the Silver views and Gold external table.
8. Connect BI tools and publish governed semantic models.

## Operational validation

### Ingestion

- Every configured source file exists and is non-empty.
- Source and landed row counts reconcile.
- Filenames and encodings are correct.
- Failed ForEach iterations are retried or quarantined.

### Databricks

- Before/after cleaning counts are recorded.
- Key columns remain non-null.
- Join match rates stay within expected thresholds.
- Output schema and row count are versioned.
- MongoDB resources are closed after use.

### Synapse

- `OPENROWSET` can read the Silver path.
- Managed identity has required ACL and RBAC permissions.
- CETAS writes successfully to an empty/non-conflicting target location.
- External table counts reconcile with the delivered-order view.

## Security note

The original Synapse SQL script contains a literal sample password in its `CREATE MASTER KEY` statement. Replace it with an environment-specific secret and rotate it if it has been used. Do not reproduce real database passwords, MongoDB credentials, storage keys, or master-key passwords in Git.

## Monitoring recommendations

- ADF pipeline/activity failures and source-to-sink row counts
- ADLS file arrival, sizes, and unexpected schema changes
- Databricks job failures, duration, and Spark stage metrics
- Cleaning rejection counts and join miss rates
- Synapse query failures and external-table freshness
- BI refresh duration and semantic-model errors

## Recovery behavior

- ADF file loads can be rerun when sink naming is idempotent or duplicates are handled.
- The Databricks notebook uses overwrite mode, so retain prior versions or use Delta for recoverability.
- CETAS requires careful location management; existing output may need versioned paths.
- Validate a backfill in a separate target before replacing production Silver or Gold data.
