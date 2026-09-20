<div align="center">

# 🌌 Mastering the Power BI Nightmare Data Model

### Power BI • Power Query • DAX • Star Schema • Data Modeling

<p>
  <strong>From a Chaotic 23-Table Dataset to a Production-Grade Star Schema</strong>
</p>

<p>
  <a href="#-overview">Overview</a> •
  <a href="#-modeling-standards">Standards</a> •
  <a href="#-building-the-dimension-tables">Dimensions</a> •
  <a href="#-building-the-fact-tables">Facts</a> •
  <a href="#-advanced-refinements-and-security">Security</a> •
  <a href="#-sample-dax-measures">DAX</a>
</p>

<br>

![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![Power Query](https://img.shields.io/badge/Power_Query-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)
![DAX](https://img.shields.io/badge/DAX-005B94?style=for-the-badge&logo=microsoft-excel&logoColor=white)
![Star Schema](https://img.shields.io/badge/Star%20Schema-Data%20Modeling-6A1B9A?style=for-the-badge)

<br><br>

<img src="images/model-preview.png" alt="Power BI Star Schema Model Preview" width="900">

<br>

<em>From 23 chaotic source tables to a clean, production-grade Star Schema</em>

</div>

A complete data modeling project: transforming a chaotic, real-world-style dataset into a clean, professional **Star Schema** in Power BI — from raw tables to row-level security.

---

# 📚 Table of Contents

- [📌 Overview](#-overview)
- [📐 Modeling Standards](#-modeling-standards)
- [📁 Power Query Folder Structure](#-power-query-folder-structure)
- [🧩 Building the Dimension Tables](#-building-the-dimension-tables)
- [📊 Building the Fact Tables](#-building-the-fact-tables)
- [🔒 Advanced Refinements and Security](#-advanced-refinements-and-security)
- [🧠 Sample DAX Measures](#-sample-dax-measures)
- [💡 Key Takeaways](#-key-takeaways)

---

# 📌 Overview

Poor data model design is one of the leading causes of slow Power BI reports and inaccurate metrics. This project takes a deliberately messy **"nightmare" dataset** — one designed to mimic the kind of chaos found in real companies — and rebuilds it step by step into a professional Star Schema optimized for **performance, scalability, and data security**.

The project started with **23 unstructured source tables** containing:

```text
✗ Random many-to-many relationships in every direction
✗ Data split arbitrarily across years and across header/detail tables
✗ Duplicate records and inconsistent field naming and data types
```

By the end of the project, this was compressed into an **advanced star-based model** (closer to a galaxy schema) with shared dimension tables and purpose-built fact tables — reducing model size and enabling **dynamic, role-based data security**.

---

# 📐 Modeling Standards

A strict set of standards was followed throughout to mirror professional environments:

## 1️⃣ Star Schema Design
Fact tables sit at the center, surrounded by dimension tables; fact tables are never joined directly to each other.

## 2️⃣ Grain Definition
Explicitly defining what each row in a table represents, to avoid duplication and "fan-out" of numbers.

## 3️⃣ Removing Non-Analytical Columns
Dropping hash keys and other technical fields that don't serve reporting, cutting model size by roughly **20%** and speeding up refresh times.

## 4️⃣ Protecting Totals
Validating that key metrics (like total sales) stayed consistent after every merge, to catch silent data corruption early.

## 5️⃣ Naming Conventions

| Rule | Convention |
|---|---|
| 🌐 Model language | English |
| 🔤 Naming style | `snake_case` for tables and fields |
| 🏷️ Prefixes | `dim_` for dimensions, `fact_` for facts |
| 🔑 Surrogate keys | End in `_key` to distinguish them from source `_id` fields |
| 📝 Field labels | Technical field names translated into clear, business-friendly labels |

---

# 📁 Power Query Folder Structure

Queries were organized into four numbered folders to keep the project manageable as it grew:

```text
1. Stage        <- Raw tables pulled from source (load disabled to save memory)
2. Dimensions   <- Rebuilt and cleaned dimension tables
3. Facts        <- Fact and transaction tables
4. Support      <- Supporting tables (e.g. the security table)
```

---

# 🧩 Building the Dimension Tables

## 👤 Customer Dimension (`dim_customer`)

- Merged **6 source tables** (`customer_master`, `customer_contacts`, `user_details`, `addresses`, `cities`, `regions`) into a single dimension table, favoring a **Star Schema over a Snowflake Schema**.
- Filtered out an invalid test row (ID `999`) to arrive at an accurate count of **60 unique customers**.
- Handled a grain mismatch between customers and their multiple contacts by filtering to primary contacts only (`is_primary = True`), preventing duplicate customer rows.
- Removed geographic IDs and technical hash-key fields that were slowing down refreshes.
- Standardized terminology to **"Customer"** instead of the inherited **"User"** label from one of the source tables.

## 📦 Product Dimension (`dim_product`)

- Used the `products` table as the base, filtering out test rows to land on **60 valid products**.
- Merged in the `subcategories` table (after promoting headers) and split category/subcategory using a delimiter.
- Generated a surrogate `product_key` via an index column to keep the model independent of source IDs.
- Removed long description fields and unused hash codes.
- Standardized category text casing to avoid null mismatches caused by case-sensitive joins.

## 🌍 Geography Dimension (`dim_geo`)

- Built a clean, standalone geography dimension from the cities table, with a surrogate `geo_key` and a unified region key.

---

# 📊 Building the Fact Tables

## 💰 Main Sales Fact (`fact_sales`)

- Combined 2025 and 2026 sales tables via an **Append** operation to resolve year-based data fragmentation.
- Merged order line items with order headers at the **lowest grain** (transaction line level), since that's where the real analytical numbers live.
- Discovered and fixed a sales total discrepancy caused by two products sharing a name but having different source IDs — resolved by tracing and cleaning `source_id` values to restore the correct total.
- Isolated sales channel, order status, and risk flags into a separate **junk dimension** (`dim_order_flags`) with **14 unique combinations** and a single `flag_key`, keeping the fact table narrow.
- Enriched the data by manually translating numeric sales-channel codes into readable labels (e.g. `10` → Online Store, `20` → Retail Partner).
- Implemented a **role-playing dimension**: `dim_geo` is linked to the sales table twice — an active relationship for ship-to city (`ship_to_city_key`) and an inactive one for bill-to city (`bill_to_city_key`), activated on demand via DAX.

## 📦 Inventory Fact (`fact_inventory`)

- Fixed a wide-table structure where months were represented as columns, using Power Query's **Unpivot Columns** to convert them into rows.
- Linked the table to the surrogate `product_key` to eliminate reliance on the duplicated source product name.

## 📣 Marketing Campaigns and a Factless Fact Table

- Split the campaigns table into a static `dim_campaign` (campaign details and estimated cost) and a daily `fact_campaign_spend` table (impressions, clicks, actual spend).
- Built a **factless fact table** (`fact_promotion_coverage`) to represent which products were covered by which campaigns, by splitting comma-separated product lists into individual rows and trimming whitespace — creating a bridge table with no numeric measures of its own.

## 🚚 Accumulating Snapshot Fact (`fact_order_process`)

- Built for logistics analysis: a single fact table that tracks an order's full lifecycle in one row — order date, ship date, actual delivery date, invoice date, and final payment date — all matched to the order's unique ID.

---

# 🔒 Advanced Refinements and Security

## 📅 Dynamic Date Dimension (`dim_date`)

- Avoided hardcoded date lists by generating the date table dynamically with `CALENDARAUTO()`, which scans the model for its earliest and latest dates so the calendar grows automatically as new transactions are added.
- Added standard supporting columns (Year, Month) and set a compact date format across the model to save space.

## 🧮 Centralized Measures Table

- Created an empty table prefixed `_measures` as a single home for all calculated measures, instead of scattering them across tables — reducing duplicate or inconsistent calculations from report builders.

## 🔐 Row-Level Security (RLS)

- Protected regional data privacy by linking a support `security` table to `dim_customer` with a **one-directional filter**.
- Built a dynamic **"Regional Access"** security role using a DAX filter matched to the logged-in user's email:

```dax
[region] = CALCULATE(MAX(security[region]), security[user_email] = USERPRINCIPALNAME())
```

- Tested the role by simulating a regional user, confirming the report automatically scoped down to that user's region only.

---

# 🧠 Sample DAX Measures

Measures were built to align precisely with each fact table's defined grain:

```dax
-- Total Sales (direct sum from the cleaned fact table)
Total Sales = SUM(fact_sales[line_total])

-- Distinct Orders (DISTINCTCOUNT avoids double-counting across line items)
Total Orders = DISTINCTCOUNT(fact_sales[order_id])

-- Active Customers (customers with actual purchases in the fact table)
Total Active Customers = DISTINCTCOUNT(fact_sales[customer_id])

-- Total Registered Customers (from the dimension table directly)
Total Customers = COUNT(dim_customer[customer_id])

-- Average Order-to-Payment Cycle Time (in days)
Average Order To Pay = AVERAGE(fact_order_process[order_to_pay])
```

### 🧮 Concepts Demonstrated

```text
✓ Star Schema Design
✓ Grain Definition
✓ Role-Playing Dimensions
✓ Junk Dimensions
✓ Factless Fact Tables
✓ Accumulating Snapshot Facts
✓ Dynamic Date Table (CALENDARAUTO)
✓ Centralized Measures Table
✓ Row-Level Security (RLS)
✓ DISTINCTCOUNT / CALCULATE / USERPRINCIPALNAME
```

---

# 💡 Key Takeaways

## 1️⃣ ⚡ Faster Refreshes, Smaller Model

Removing unnecessary columns and hash keys noticeably cut file size and improved refresh speed.

## 2️⃣ 🎯 Single Source of Truth

Keeping customer geography inside its own dimension, instead of duplicated inside fact tables, eliminated conflicting numbers across reports.

## 3️⃣ 🧩 Clean Separation Between Facts

Connecting fact tables only through shared dimensions kept calculations accurate even as report complexity grew.

---

# 📁 Project Structure

```text
powerbi-nightmare-data-model/
│
├── images/
│   └── model-preview.png
│
├── data/
│   └── nightmare_dataset/        # 23 raw source tables
│
├── powerbi/
│   └── nightmare_data_model.pbix
│
└── README.md
```

> 📝 عدّل الأسماء والمسارات دي بحيث تطابق بالظبط أسماء الملفات الموجودة فعليًا في الريبو بتاعك.

---

# 👨‍💻 Author

**Ashraf Nabil Mohamed**
Data Analyst Junior & Machine Learning | Transitioning to Data Engineering

- 🎓 B.Sc. Computer Science (AI & Data Science), Zagazig University
- 💻 GitHub: [github.com/ASHRAF29403](https://github.com/ASHRAF29403)

---

<div align="center">
<em>This project was built as a hands-on exercise in advanced data modeling and business intelligence architecture in Power BI.</em>
</div>

</div>
