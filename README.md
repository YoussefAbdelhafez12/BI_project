🏬 Retail Data Warehouse — Medallion Architecture
A fully functional end-to-end data warehouse built for a retail business using the Medallion Architecture (Bronze → Silver → Gold), implemented in Microsoft SQL Server and visualized through a Power BI dashboard.

Course: Business Intelligence & Data Analytics — German International University (GIU)
Instructor: Dr. Shaimaa Masry
Team: Youssef Hassan · Ali Mohamed · Omar Samer · Bahaa Ahmed · Hazem Adel

📐 Architecture Overview
CSV Source Files (CRM + ERP)
        │
        ▼
┌─────────────────┐
│   BRONZE LAYER  │  Raw ingestion — no transformations
│   19 tables     │  TRUNCATE & BULK INSERT via stored procedure
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   SILVER LAYER  │  Cleansed, standardized, enriched
│   19 tables     │  Proper types · Derived columns · NULL handling
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│    GOLD LAYER   │  Galaxy schema — 2 star schemas
│   11 tables     │  Conformed dimensions · Surrogate keys
└────────┬────────┘
         │
         ▼
   Power BI Dashboard
🗂️ Source Dataset
19 CSV files from two operational systems covering 47,000+ rows of retail data:

System	Tables
CRM	customers, employees, online_orders, online_order_items, payments, deliveries, delivery_providers
ERP	products, brands, departments, stores, warehouses, suppliers, product_suppliers, promotions, registers, pos_transactions, transaction_items, inventory
🥉 Bronze Layer
Goal: Ingest raw source data into SQL Server with zero transformations — an immutable audit trail.

All 19 tables created as exact mirrors of their CSV source files
All columns stored as NVARCHAR (no type conversion at this stage)
Loaded via bronze.load_bronze stored procedure using TRUNCATE + BULK INSERT
Full audit logging via bronze.load_log (table name, row count, timestamp, status)
🥈 Silver Layer
Goal: Cleanse, standardize, and enrich data with proper types and derived columns.

Transformation	Detail
Type casting	All columns cast from NVARCHAR to proper types (INT, DATE, DATETIME, DECIMAL) using TRY_CAST()
Whitespace	TRIM() applied to all text fields
NULL handling	Rows with NULL/blank primary keys filtered out; optional fields defaulted
Standardization	Gender codes (M/F → Male/Female), Order_Status casing unified
Derived columns	Full_Name, Total_Line_Amount, Delivery_Days, Duration_Days
Enrichment	Products joined with brands and departments to denormalize lookup values
Loaded via silver.load_silver stored procedure with audit logging via silver.load_log.

🥇 Gold Layer
Goal: Business-ready dimensional model for analytics and reporting.

Galaxy Schema Design
Two star schemas sharing 4 conformed dimensions:

                        ┌──────────────┐
                        │  dim_date    │ (conformed)
                        └──────┬───────┘
                               │
┌─────────────┐    ┌───────────┴──────────┐    ┌──────────────┐
│  dim_store  │────│  fact_store_sales    │────│ dim_employee │
└─────────────┘    │  (Star Schema 1)     │    └──────────────┘
                   └──┬──────────────┬───┘
                      │              │
             ┌────────┘              └────────┐
      ┌──────┴──────┐            ┌────────────┴──┐
      │ dim_customer│            │  dim_product  │  (conformed)
      │  (conformed)│            └───────────────┘
      └──────┬──────┘
             │
   ┌─────────┴──────────────┐
   │  fact_online_sales      │
   │  (Star Schema 2)        │
   └──┬──────────────────┬──┘
      │                  │
┌─────┴──────┐    ┌──────┴──────────────┐
│dim_warehouse│   │ dim_order_status    │
└────────────┘    │ dim_payment_method  │
                  │ dim_promotion       │ (conformed)
                  └─────────────────────┘
Gold Tables
Table	Type	Used By
dim_date	Conformed Dimension	Both schemas
dim_customer	Conformed Dimension	Both schemas
dim_product	Conformed Dimension	Both schemas
dim_promotion	Conformed Dimension	Both schemas
dim_store	Dimension	fact_store_sales
dim_employee	Dimension	fact_store_sales
dim_warehouse	Dimension	fact_online_sales
dim_order_status	Dimension	fact_online_sales
dim_payment_method	Dimension	fact_online_sales
fact_store_sales	Fact Table	Star Schema 1 — ~14,962 rows
fact_online_sales	Fact Table	Star Schema 2 — ~7,151 rows
Loaded via gold.load_gold stored procedure.

📁 Repository Structure
├── bronze/
│   ├── bronze01_create_tables.sql
│   └── bronze02_load_bronze.sql
├── silver/
│   ├── silver01_create_tables.sql
│   └── silver02_load_silver.sql
├── gold/
│   ├── gold01_create_tables.sql
│   └── gold02_load_gold.sql
├── powerbi/
│   └── retail_dashboard.pbix
└── README.md
🚀 How to Run
Create the database

CREATE DATABASE DataWarehouse;
Create schemas

CREATE SCHEMA bronze;
CREATE SCHEMA silver;
CREATE SCHEMA gold;
Run in order

bronze01_create_tables.sql
bronze02_load_bronze.sql
silver01_create_tables.sql
silver02_load_silver.sql
gold01_create_tables.sql
gold02_load_gold.sql
Connect Power BI to the gold schema of DataWarehouse

⚠️ Update the file paths in bronze02_load_bronze.sql to match your local CSV directory before running BULK INSERT.

🛠️ Tech Stack
SQL Server T-SQL Power BI

Database: Microsoft SQL Server
Language: T-SQL (stored procedures, CTEs, window functions)
Modeling: Kimball Dimensional Modeling — Star Schema / Galaxy Schema
Visualization: Power BI (DAX, data modeling, KPIs)
