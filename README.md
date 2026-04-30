# 🏅 World Market Data Analysis — Medallion Architecture on Microsoft Fabric

> **Internship Project** · Krabdata · July – October 2025  
> Realized by **Dorra Ourabi** — Software Engineering, INSAT

---

## 📌 Overview

This project implements a full **end-to-end data pipeline** using a **Medallion Architecture (Bronze → Silver → Gold)** on **Microsoft Fabric**, applied to the **World Bank's World Development Indicators** dataset — one of the largest publicly available datasets on global economic and social metrics.

The goal: transform over **1.2 million rows** of raw, wide-format world indicators data into clean, continent-segmented, analytically ready datasets, visualized through interactive **Power BI dashboards**.

---

## 🏗️ Architecture Overview

```
Raw Data (CSV)
     │
     ▼
┌─────────────┐
│   BRONZE    │  ← Raw ingestion into Microsoft Fabric Lakehouse (OneLake)
│             │    CSV files stored as-is / Parquet format
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   SILVER    │  ← PySpark notebook: cleaning, reshaping, interpolation
│             │    Segmented by continent · Pivoted from wide to long format
└──────┬──────┘
       │
       ▼
┌─────────────┐
│    GOLD     │  ← Delta tables in Lakehouse → Power BI semantic model
│             │    Interactive maps + statistical dashboards
└─────────────┘
```

---

## 📊 Dataset

| Source | [World Bank — World Development Indicators](https://datacatalog.worldbank.org/search/dataset/0037712) |
|--------|------------------------------------------------------------------------------------------------------|
| Indicators | 1,500+ |
| Countries | 220+ |
| Time Range | 1960 – 2024 |
| Format | CSV (189 MB main file) |
| Rows after ingestion | **1,283,580 rows × 69 columns** |

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Cloud Platform | **Microsoft Fabric** (SaaS) |
| Storage | **OneLake** (Lakehouse — Parquet / Delta format) |
| Processing | **Apache Spark** via **PySpark notebooks** |
| Data Cleaning | PySpark SQL · Window Functions · Linear Interpolation |
| Visualization | **Power BI** (ARCGis Maps, Pie Charts, Line Charts, KPI Labels) |
| Data Source | World Bank Open Data (CSV) |

---

## 🔄 Pipeline Details

### 🥉 Bronze — Raw Ingestion
- Raw CSV files uploaded to the Lakehouse under a `/Bronze` folder
- Files accessible via ABFS (Azure Blob File System) links
- Data preserved in original format — no transformation at this stage
- Schema inferred manually: string types for country/indicator fields, float for year values (1960–2024)

### 🥈 Silver — Cleaning & Transformation

**Challenge 1: Dataset size (1.2M+ rows)**  
→ Solution: Joined with a `countries_continents.csv` dataset to add a `continent` column, then split into **6 continent-level DataFrames** (Africa, Asia, Europe, North America, South America, Oceania) + 1 `stats` table for regional aggregates.

**Challenge 2: Wide format (69 columns)**  
→ Solution: **Pivot/unpivot** using PySpark's `stack()` function — transformed year columns into a `(year, value)` long format, enabling groupBy, aggregation, and filtering operations.

**Challenge 3: Missing values**  
→ Strategy A: For large null intervals → substitute with regional aggregate values from the `stats` table  
→ Strategy B: For isolated null gaps (≤ 5 years) → **linear interpolation** using PySpark Window functions

**Interpolation Formula:**

$$\text{interp\_value} = V_{prev} + \frac{V_{next} - V_{prev}}{Y_{next} - Y_{prev}} \times (Y - Y_{prev})$$

Results on Africa subset:
- Total imputed values: **95,885**
- Imputation rate: ~2% (expected given dataset width)
- Gain over non-imputed: **2.03%**

### 🥇 Gold — Analytics & Visualization
- Cleaned DataFrames written to Lakehouse as **Delta tables**
- Connected to **Power BI** via SQL endpoint + semantic model
- Dashboards created per continent using **ARCGis for Power BI**:
  - Interactive choropleth maps filterable by indicator & year
  - Hover tooltips: country, indicator, year, value
- Stats dashboard: pie chart (median by year) + line chart (mean over time) + min/max KPI cards

---

## 📁 Project Structure

```
indicators (Lakehouse)
├── Files/
│   └── Bronze/
│       ├── WDICSV.csv           # Main indicators data (189 MB)
│       ├── WDICountry.csv
│       ├── WDISeries.csv
│       ├── WDIcountry-series.csv
│       ├── WDIfootnote.csv
│       ├── WDIseries-time.csv
│       └── countries/
│           └── countries_continents.csv
└── Tables/
    ├── africa_gold
    ├── asia_gold
    ├── europe_gold
    ├── north_america_gold
    ├── south_america_gold
    ├── oceania_gold
    └── stats_gold
```

---

## 💡 Key Engineering Decisions

| Decision | Rationale |
|----------|-----------|
| **DataLakehouse over DataWarehouse** | Dataset contains semi-structured and unstructured entries; Lakehouse supports schema-on-read, lower storage cost |
| **PySpark over Pandas** | 1.2M+ rows — Spark's distributed in-memory processing is essential for BigData at this scale |
| **Continent segmentation ("Divide & Conquer")** | Prevents timeouts and infinite loops on full-dataset computations |
| **Wide → Long pivot** | Unlocks groupBy/agg/filter operations impossible on year-as-column format |
| **Linear interpolation with gap threshold** | Fills short gaps without distorting long-term trend curves |
| **Delta format for Gold layer** | Enables ACID transactions, versioning, and direct Power BI SQL endpoint connection |

---

## 📈 Sample Visualizations

The Power BI dashboards include:
- 🗺️ **Continent maps** — choropleth visualization per indicator per year (Africa, Asia, Europe, North America, South America, Oceania)
- 📊 **Statistical dashboard** — median distribution pie chart, average trend line, global min/max KPI cards

---

## 🚀 What I Learned

- Designing and implementing a **production-grade data architecture** (Medallion pattern) from scratch
- Working with **Microsoft Fabric** — workspace management, Lakehouse, OneLake, Spark notebooks, Power BI integration
- Handling **real-world data quality issues** at scale: missing values, wide-format data, mixed entity types (countries vs. regional aggregates)
- **PySpark** engineering: schema inference, Window functions, stack/unpivot, Delta write
- Building **interactive geospatial dashboards** in Power BI with ARCGis

---

## 🏢 About Krabdata

[Krabdata](https://krabdata.com) is a data consultancy specializing in data engineering, analytics, and business intelligence solutions.

---

## 👤 Author

**Dorra Ourabi**  
Software Engineering Student — INSAT (Institut National des Sciences Appliquées et de Technologie)  
Internship: July – October 2025

---

*Built with Microsoft Fabric · Apache Spark · Power BI · World Bank Open Data*
