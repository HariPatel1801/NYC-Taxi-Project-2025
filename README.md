# NYC Taxi Data Engineering Project using Azure Data Factory, ADLS Gen2, Databricks & Delta Lake
Project Overview
This project demonstrates an end-to-end Azure Data Engineering pipeline using the Medallion Architecture (Bronze, Silver, and Gold layers).
The solution ingests NYC Taxi data from external sources using Azure Data Factory, stores the raw data in Azure Data Lake Storage Gen2, performs transformations using Azure Databricks, and serves curated datasets through Delta Lake for analytics and reporting.
Architecture
Data Flow
Source Data → Azure Data Factory → ADLS Gen2 (Bronze Layer) → Databricks (Silver Layer) → Databricks + Delta Lake (Gold Layer) → Reporting & Analytics
Technologies Used
Azure Data Factory (ADF)
Azure Data Lake Storage Gen2 (ADLS Gen2)
Azure Databricks
Apache Spark (PySpark)
Delta Lake
Azure Key Vault
Azure Active Directory Service Principal
Power BI
Project Layers
1. Ingestion Layer (Bronze)
Objective
Load raw NYC Taxi datasets into Azure Data Lake Storage Gen2 without modification.
Tools Used
Azure Data Factory
ADLS Gen2
Process
Source taxi data is extracted through API/files.
Azure Data Factory pipelines(HTTP Activity) ingest the data.
Raw files are stored in the Bronze container.
Bronze Storage Structure
bronze/
│
├── Taxi-Trip-Data-2025/
├── trip_type/
└── trip_zone/
Benefits
Preserves original source data.
Supports data auditing and reprocessing.
Provides a single source of truth.
2. Security Layer
Azure Key Vault
Sensitive credentials are stored securely in Azure Key Vault:
Client ID
Client Secret
Tenant ID
Databricks Secret Scope
Secrets are accessed inside Databricks notebooks using:
secret_key = dbutils.secrets.get("my_scope","secret-key")
client_id = dbutils.secrets.get("my_scope","client-id")
tenant_id = dbutils.secrets.get("my_scope","tenant-id")
Benefits
No hardcoded credentials.
Centralized secret management.
Secure authentication to Azure Storage.
3. Silver Layer (Transformation)
Objective
Clean and transform raw datasets.
Input
Data is read from the Bronze layer.
Datasets Processed
Trip Type Dataset
Transformations:
Rename description column to trip_description.
Standardize schema.
Example:
df_trip_type = df_trip_type.withColumnRenamed(
    "description",
    "trip_description"
)
Trip Zone Dataset
Transformations:
Split zone field into multiple attributes.
Improve reporting usability.
Example:
df_trip_zone = df_trip_zone \
    .withColumn("zone1", split(col("zone"), "/")[0]) \
    .withColumn("zone2", split(col("zone"), "/")[1])
Taxi Trip Dataset
Transformations:
Applied predefined schema.
Converted data types.
Standardized timestamps.
Validated structure.
Example:
df_trip = spark.read.format("parquet") \
    .schema(myschema) \
    .load(bronze_path)
Output
Transformed datasets are written into the Silver container.
silver/
│
├── trip_type/
├── trip_zone/
└── trip_data/
Benefits
Clean and structured data.
Improved data quality.
Optimized for analytics.
4. Gold Layer (Business Ready Data)
Objective
Create curated datasets for reporting and analytics.
Input
Data from the Silver layer.
Storage Format
Delta Lake
Example
df_zone.write.format("delta") \
    .mode("append") \
    .option("path", gold_path) \
    .saveAsTable("gold.trip_zone")
Delta Lake Features Implemented
ACID Transactions
Reliable updates and deletes.
Example:
UPDATE gold.trip_zone
SET Borough = 'EMR'
WHERE LocationID = 1;
Delete Operations
DELETE FROM gold.trip_zone
WHERE LocationID = 1;
Time Travel
Restore previous table versions.
RESTORE gold.trip_zone
TO VERSION AS OF 2;
History Tracking
DESCRIBE HISTORY gold.trip_zone;
Gold Layer Benefits
Business-ready datasets.
High-performance analytics.
Data versioning.
Historical recovery.
Reliable reporting.
Reporting Layer
Tools
Power BI
SQL Analytics
Purpose
The Gold Layer serves as the trusted source for dashboards and reporting.
Typical business insights include:
Trip volume analysis
Revenue analysis
Pickup and drop-off trends
Borough-wise performance
Zone-wise demand analysis
Folder Structure
project/
│
├── notebooks/
│   ├── silver_notebook.ipynb
│   └── gold_notebook.ipynb
│
├── architecture/
│   └── architecture-diagram.png
│
├── datasets/
│
└── README.md
Key Learning Outcomes
Azure Data Factory orchestration
Azure Data Lake Gen2 implementation
Service Principal authentication
Azure Key Vault integration
Databricks Secret Scopes
PySpark transformations
Medallion Architecture
Delta Lake implementation
Time Travel and Versioning
End-to-End Data Engineering Pipeline
Author
Shail Gajjar
Azure Data Engineering Project – NYC Taxi Analytics Platform
