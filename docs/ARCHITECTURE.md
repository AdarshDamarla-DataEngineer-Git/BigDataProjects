# Architecture

## System context

```mermaid
flowchart LR
    SOURCES["GitHub CSV · MySQL · MongoDB"]
    PLATFORM["Azure e-commerce data platform"]
    USERS["Data engineers · analysts · BI users"]

    SOURCES -->|"marketplace source data"| PLATFORM
    USERS -->|"operate and monitor"| PLATFORM
    PLATFORM -->|"curated delivery and sales data"| USERS
```

## Component architecture

```mermaid
flowchart TB
    subgraph SOURCE["Source systems"]
        HTTP["GitHub raw CSV files"]
        SQL["MySQL order payments"]
        NOSQL["MongoDB category translations"]
    end

    subgraph INGEST["Ingestion"]
        META["ForEachInput.json"]
        ADF["Azure Data Factory"]
        META --> ADF
    end

    subgraph LAKE["ADLS Gen2"]
        RAW["Raw Olist datasets"]
        SILVER["Enriched Silver Parquet"]
        GOLD["Gold CETAS output"]
    end

    subgraph PROCESS["Azure Databricks"]
        READ["Spark CSV readers"]
        CLEAN["Null and duplicate cleanup"]
        FEATURES["Delivery feature engineering"]
        JOINS["Multi-table enrichment"]
        READ --> CLEAN --> FEATURES --> JOINS
    end

    subgraph SERVE["Azure Synapse Serverless"]
        VIEW["OPENROWSET views"]
        FILTER["Delivered-order view"]
        CETAS["External Gold table"]
        VIEW --> FILTER --> CETAS
    end

    BI["Power BI · Tableau · Fabric"]

    HTTP --> ADF
    SQL --> ADF
    ADF --> RAW --> READ
    NOSQL --> JOINS
    JOINS --> SILVER --> VIEW
    CETAS --> GOLD
    VIEW --> BI
    CETAS --> BI
```

## Processing sequence

```mermaid
sequenceDiagram
    autonumber
    participant G as GitHub / databases
    participant A as Azure Data Factory
    participant L as ADLS Gen2
    participant D as Databricks
    participant M as MongoDB
    participant S as Synapse Serverless
    participant B as BI consumer

    A->>G: Read configured source datasets
    G-->>A: CSV/table records
    A->>L: Land raw files
    D->>L: Read eight Olist datasets
    D->>M: Read product-category translations
    D->>D: Clean, derive delivery metrics, and join
    D->>L: Overwrite enriched Silver Parquet
    S->>L: Query Silver with OPENROWSET
    S->>L: Write delivered records with CETAS
    B->>S: Query views or external table
```

## Transformation lineage

```mermaid
flowchart LR
    ORD["Orders"] --> OC["Orders + Customers"]
    CUS["Customers"] --> OC
    OC --> OP["+ Payments"]
    PAY["Payments"] --> OP
    OP --> OI["+ Order Items"]
    ITEMS["Order Items"] --> OI
    OI --> PR["+ Products"]
    PROD["Products"] --> PR
    PR --> SE["+ Sellers"]
    SELL["Sellers"] --> SE
    SE --> CAT["+ Category translation"]
    MONGO["MongoDB categories"] --> CAT
    CAT --> FINAL["Enriched Silver Parquet"]
```

Reviews and geolocation are read and cleaned in the notebook but are not currently joined into `final_df`.

## Security boundaries

- ADF requires access to public HTTP sources and write access to ADLS.
- Databricks requires ADLS and MongoDB access.
- Synapse managed identity requires Silver read and Gold write permissions.
- Database credentials and Synapse master-key values should be supplied through secrets, not notebook variables or committed SQL.

## Failure boundaries

- A missing GitHub file or incorrect relative URL fails the related ingestion iteration.
- Blanket null removal can eliminate valid but incomplete records.
- Left joins preserve base orders but can multiply rows for multi-row payments and items.
- Overwrite mode replaces the complete Silver output on every successful run.
- Schema drift can affect `SELECT *` Synapse views and CETAS output.
