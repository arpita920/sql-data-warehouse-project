SQL Data Warehouse with Medallion Architecture

A production-style data warehouse built using SQL Server and T-SQL, implementing the Medallion Architecture (Bronze → Silver → Gold) to transform raw CRM and ERP data into business-ready analytics.
---
Architecture Overview
```
Source Systems (CRM + ERP CSV files)
            ↓
    ┌───────────────┐
    │  BRONZE LAYER │  Raw ingestion — data loaded as-is via BULK INSERT
    └───────┬───────┘
            ↓
    ┌───────────────┐
    │  SILVER LAYER │  Cleaned, standardized, deduplicated data
    └───────┬───────┘
            ↓
    ┌───────────────┐
    │   GOLD LAYER  │  Star Schema — business-ready dimensions & fact tables
    └───────┬───────┘
            ↓
    EDA / Analytics Queries
```
---
Tech Stack

Database: Microsoft SQL Server

Language: T-SQL

Concepts: ETL, Data Warehousing, Star Schema, Stored Procedures, Window Functions

---
Project Structure
```
sql-data-warehouse-project/
│
├── datasets/                  # Source CSV files (CRM + ERP)
│
├── scripts/
│   ├── bronze/
│   │   ├── ddl_bronze.sql         # Creates Bronze schema tables
│   │   └── proc_load_bronze.sql   # Stored procedure to load raw data
│   │
│   ├── silver/
│   │   ├── ddl_silver.sql         # Creates Silver schema tables
│   │   └── proc_load_silver.sql   # Stored procedure for cleaning & transformation
│   │
│   └── gold/
│       └── ddl_gold.sql           # Creates Gold views (Star Schema)
│
├── eda/                           # 13 analytical SQL scripts
│   ├── 01_database_exploration.sql
│   ├── 02_dimensions_exploration.sql
│   ├── 03_date_range_exploration.sql
│   ├── 04_measures_exploration.sql
│   ├── 05_magnitude_analysis.sql
│   ├── 06_ranking_analysis.sql
│   ├── 07_change_over_time_analysis.sql
│   ├── 08_cumulative_analysis.sql
│   ├── 09_performance_analysis.sql
│   ├── 10_data_segmentation.sql
│   ├── 11_part_to_whole_analysis.sql
│   ├── 12_report_customers.sql
│   └── 13_report_products.sql
│
├── tests/                         # Data quality checks
└── README.md
```
---
Layer Details:


- Bronze Layer: Raw Ingestion
  - Loads CSV data from CRM and ERP source systems directly into SQL Server using `BULK INSERT`
  - No transformations — data is stored exactly as received
  - Truncates and reloads on each run for full refresh


- Silver Layer: Cleaning & Standardization
  
  Key transformations applied:
   - Deduplication — Removes duplicate customer records using `ROW_NUMBER()` window function, keeping the most recent entry per customer
   - Type normalization — Decodes coded fields (e.g., `'M'` → `'Male'`, `'S'` → `'Single'`)
   - Null handling — Replaces missing cost values with 0 using `ISNULL()`
   - Date standardization — Casts inconsistent date formats to proper `DATE` type
   - Key derivation — Extracts category IDs and product keys from compound key fields using `SUBSTRING()`
   - SCD handling — Derives product end dates using `LEAD()` window function
   - Audit columns — Adds `dwh_create_date` timestamp to all tables for data lineage tracking

 - Gold Layer: Business-ready analytics (Star Schema)
   
     Three views make up the final analytics model:
      - `gold.dim_customers`: Customer profile combination CRM + ERP data with surrogate keys
      - `gold.dim_products`: Product catalog with category hierarchy, filters out historical records
      - `gold.fact_sales`: Sales transactions linked to product and customer dimensions

     Data integration logic: CRM is used as the primary source for gender; ERP data serves as fallback via `COALESCE()`, mimicking real- world source system prioritization.
---

How to Run
 - Run `scripts/init_database.sql` to create the `DataWarehouse` database and schemas
 - Run `scripts/bronze/ddl_bronze.sql` to create Bronze tables
 - Execute `EXEC bronze.load_bronze` to load raw data
 - Run `scripts/silver/ddl_silver.sql` to create Silver tables
 - Execute `EXEC silver.load_silver` to clean and transform data
 - Run `scripts/gold/ddl_gold.sql` to create Gold views
 - Run any script from the `/eda` folder to explore and analyze
> **Note:** Update file paths in `proc_load_bronze.sql` to match your local dataset location before running.
---

EDA Highlights:
  
  The `/eda` folder contains 13 structured analytical queries covering:
   - Customer segmentation by demographics and geography
   - Product performance and category contribution
   - Sales trends over time and cumulative revenue analysis
   - Ranking analysis for top customers and products
   - Part-to-whole breakdown for category share
---

Key Concepts Demonstrated:
 - Medallion Architecture (Bronze / Silver / Gold)
 - ETL pipeline using T-SQL Stored Procedures
 - Star Schema design (Dimensions + Fact table)
 - Window functions: `ROW_NUMBER()`, `LEAD()`
 - Data quality: deduplication, null handling, type casting
 - Error handling with `TRY/CATCH` and batch load timing
 - Multi-source data integration with priority-based fallback logic
