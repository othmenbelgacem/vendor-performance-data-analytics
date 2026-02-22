# 📦 Vendor Performance Data Analytics

> An end-to-end data engineering pipeline for evaluating vendor performance, profitability, and inventory efficiency across millions of retail records.

---

## 🗂 Table of Contents

- [Project Overview](#project-overview)
- [Tech Stack](#tech-stack)
- [Data Architecture & ETL Pipeline](#data-architecture--etl-pipeline)
- [Repository Structure](#repository-structure)
- [Data Scale](#data-scale)
- [Getting Started](#getting-started)
- [Power BI Integration](#power-bi-integration)

---

## Project Overview

This project is an end-to-end data engineering and analytics solution that integrates large-scale retail datasets — spanning inventory, purchases, sales, and freight costs — and transforms raw operational data into actionable business intelligence.

The pipeline covers automated data ingestion, complex SQL transformations, Python-based feature engineering, and preparation for interactive dashboarding in Power BI. It is designed to answer key business questions around **vendor reliability**, **product profitability**, and **supply chain efficiency**.

---

## Tech Stack

| Category | Tools |
|---|---|
| Language | Python 3.x |
| Libraries | `pandas`, `sqlalchemy`, `sqlite3`, `logging`, `os`, `time` |
| Database | SQLite (`inventory.db`) |
| Visualization | Power BI |
| Environment | Jupyter Notebook (exploration), Python Scripts (production ETL) |

---

## Data Architecture & ETL Pipeline

The system follows a structured **Extract → Transform → Load** architecture capable of processing millions of records efficiently.

```
Raw CSV Files (data/)
        │
        ▼
┌─────────────────────┐
│   ingestion_db.py   │  ← Extract & Load raw data into SQLite
└─────────────────────┘
        │
        ▼
┌──────────────────────────┐
│  get_vendor_summary.py   │  ← SQL aggregations + Python KPI engineering
└──────────────────────────┘
        │
        ▼
  vendor_sales_summary (SQLite)
        │
        ▼
   Power BI Dashboard
```

### 1. Automated Data Ingestion — `ingestion_db.py`

- Scans the `data/` directory and extracts all raw CSV files.
- Dynamically generates SQLite tables based on file names — no hardcoded schemas.
- Uses `SQLAlchemy` and `pandas` to efficiently bulk-load large datasets into `inventory.db`.
- Logs all ingestion activity (record counts, timing, errors) via Python's `logging` module.

### 2. SQL Transformation & Aggregation — `get_vendor_summary.py`

Executes multi-layered SQL queries using **Common Table Expressions (CTEs)** to join and aggregate data across domains:

- **FreightSummary** — Aggregates total freight costs per vendor.
- **PurchaseSummary** — Calculates total purchase quantities and dollar amounts, grouped by vendor and brand.
- **SalesSummary** — Aggregates total sales quantities, revenues, and excise taxes.

### 3. Feature Engineering & KPI Calculation — `get_vendor_summary.py`

After SQL aggregation, Python enriches the dataset with business-critical KPIs:

| KPI | Formula |
|---|---|
| **Gross Profit** | Total Sales $ − Total Purchase $ |
| **Profit Margin (%)** | (Gross Profit / Total Sales $) × 100 |
| **Stock Turnover** | Total Sales Qty / Total Purchase Qty |
| **Sales-to-Purchase Ratio** | Total Sales $ / Total Purchase $ |

Data cleaning steps include stripping whitespace from categorical fields and imputing zeros for missing values.

### 4. Business Intelligence — Power BI

The final enriched `vendor_sales_summary` table is written back to `inventory.db`, which serves as the central data source for Power BI. Interactive dashboards are built on top of these pre-calculated KPIs to surface insights into vendor performance and profitability.

---

## Repository Structure

```
vendor-performance-analytics/
│
├── ingestion_db.py              # Automated CSV extraction and database loading
├── get_vendor_summary.py        # Core ETL engine: SQL aggregations + KPI calculations
│
├── Exploratory Data.ipynb       # Initial ingestion logic testing and exploration
├── Untitled.ipynb               # Data validation: schema checks and row count verification
│
├── data/                        # Raw CSV input files (not tracked in version control)
├── logs/
│   ├── ingestion_db.log         # Ingestion execution logs
│   └── get_vendor_summary.log   # Summary generation logs
│
└── inventory.db                 # SQLite database (generated at runtime)
```

> **Note:** The `data/` directory and `inventory.db` are excluded from version control. See setup instructions below.

---

## Data Scale

This pipeline is built to handle significant data volumes:

| Table | Record Count |
|---|---|
| Sales | ~12.8 million |
| Purchases | ~2.37 million |
| Begin / End Inventory | ~430,000 (combined) |

---

## Getting Started

### Prerequisites

Ensure Python 3.x is installed, then install the required dependencies:

```bash
pip install pandas sqlalchemy
```

### Step 1 — Prepare your data

Create a `data/` folder in the project root and place your raw CSV files inside:

```
vendor-performance-analytics/
└── data/
    ├── sales.csv
    ├── purchases.csv
    ├── inventory_begin.csv
    └── ...
```

### Step 2 — Run the ingestion script

This builds and populates `inventory.db` from the CSV files:

```bash
python ingestion_db.py
```

Logs will be written to `logs/ingestion_db.log`.

### Step 3 — Generate the analytics table

This runs the SQL aggregations, calculates KPIs, and writes the final `vendor_sales_summary` table:

```bash
python get_vendor_summary.py
```

Logs will be written to `logs/get_vendor_summary.log`.

---

## Power BI Integration

Once the pipeline has run successfully:

1. Open **Power BI Desktop**.
2. Select **Get Data** → **ODBC** (or **Python Script**).
3. Connect to the local `inventory.db` file.
4. Load the `vendor_sales_summary` table.
5. Begin building visualizations using the pre-calculated KPIs.

Recommended visuals include vendor-level profitability rankings, stock turnover heatmaps, and freight cost breakdowns by supplier.

---

*Built with Python · SQLite · Power BI*
