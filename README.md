# Data Warehouse & Analytics
A SQL Server data warehouse that consolidates ERP and CRM sales data into a single analytical model, built using the medallion architecture (Bronze → Silver → Gold). I built this to get hands-on practice with end-to-end data engineering: raw ingestion, cleansing, dimensional modelling, and SQL-based reporting.
# 📖 Why I built this
I wanted a portfolio piece that goes beyond writing isolated queries and shows the full pipeline — taking messy source files and turning them into something an analyst or stakeholder could actually use. The project mirrors a realistic scenario: two separate source systems (an ERP and a CRM) that need to be cleaned, reconciled, and joined into a coherent star schema.
# 🏗️ Architecture
The warehouse follows a three-layer medallion design, where each layer has a clear responsibility:

- **Bronze** — Raw data loaded as-is from CSV exports into SQL Server. No transformations; this layer preserves the source of truth and makes the pipeline re-runnable.
- **Silver** — Cleansed and standardised data. This is where I handle the bulk of the work: deduplication, fixing inconsistent formatting, normalising values across the two systems, and resolving data quality issues so the two sources can be trusted together.
- **Gold** — Business-ready tables modelled into a star schema (fact and dimension tables) optimised for analytical queries and reporting.
# 🎯 What's Involved
- **Data architecture** — Designing the layered warehouse and documenting how data moves through it.
- **ETL pipelines** — Extracting from CSVs, transforming across the Silver layer, and loading into each stage.
- **Data modelling** — Building fact and dimension tables aimed at fast, readable analytical queries.
- **Analytics** — SQL reports covering customer behaviour, product performance, and sales trends.
# 🚀 Scope
This focuses on the latest snapshot of the data only — no historisation or slowly-changing-dimension handling, which keeps the scope tight for a portfolio build. The end goal is a clean, documented model that supports reporting rather than a production-grade pipeline.
# 🛠️ Tools
- **Database** - SQL Server Express
- **Querying / management** - SQL Server Management Studio (SSMS)
- **Version control** - Git / GitHub

# BI: Analytics & Reporting (Data Analysis)

## Objective
Develop SQL-based analytics to deliver detailed insights into:
- **Customer Behavior**
- **Product Performance**
- **Sales Trends**

These insights empower stakeholders with key business metrics, enabling strategic decision-making. 
# 📂 Repository Structure
```
data-warehouse-project/
│
├── datasets/                           # Raw datasets used for the project (ERP and CRM data)
│
├── docs/                               # Project documentation and architecture details
│   ├── etl.drawio                      # Draw.io file shows all different techniquies and methods of ETL
│   ├── data_architecture.drawio        # Draw.io file shows the project's architecture
│   ├── data_catalog.md                 # Catalog of datasets, including field descriptions and metadata
│   ├── data_flow.drawio                # Draw.io file for the data flow diagram
│   ├── data_models.drawio              # Draw.io file for data models (star schema)
│   ├── naming-conventions.md           # Consistent naming guidelines for tables, columns, and files
│
├── scripts/                            # SQL scripts for ETL and transformations
│   ├── bronze/                         # Scripts for extracting and loading raw data
│   ├── silver/                         # Scripts for cleaning and transforming data
│   ├── gold/                           # Scripts for creating analytical models
│
├── tests/                              # Test scripts and quality files
│
├── README.md                           # Project overview and instructions
├── LICENSE                             # License information for the repository
├── .gitignore                          # Files and directories to be ignored by Git
└── requirements.txt                    # Dependencies and requirements for the project
```
# Analytics goals
The reporting layer answers questions a business would actually ask:


How do customers behave, and which segments matter most?
Which products perform best, and where?
How are sales trending over time?

# Acknowledgements
This project was built while following Baraa's SQL Data Warehouse tutorial as a learning exercise. The structure and approach follow the tutorial; the implementation and documentation here are my own working version.
