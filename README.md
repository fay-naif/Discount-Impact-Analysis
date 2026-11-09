<h1 align="center">Discount Impact Analysis</h1>

<p align="center">
  A one-page Power BI data story that asks whether Superstore discounts bring growth or burn profit (2015–2018).
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Power%20BI-0b1730?style=for-the-badge&color=0b1730&labelColor=0b1730&logoColor=c9ced8&logo=powerbi" alt="Power BI">
  <img src="https://img.shields.io/badge/DAX-0b1730?style=for-the-badge&color=0b1730&labelColor=0b1730&logoColor=c9ced8" alt="DAX">
  <img src="https://img.shields.io/badge/Power%20Query-0b1730?style=for-the-badge&color=0b1730&labelColor=0b1730&logoColor=c9ced8" alt="Power Query">
  <img src="https://img.shields.io/badge/Excel-0b1730?style=for-the-badge&color=0b1730&labelColor=0b1730&logoColor=c9ced8&logo=microsoftexcel" alt="Excel">
  <img src="https://img.shields.io/badge/Python-0b1730?style=for-the-badge&color=0b1730&labelColor=0b1730&logoColor=c9ced8&logo=python" alt="Python">
  <img src="https://img.shields.io/badge/SQL-0b1730?style=for-the-badge&color=0b1730&labelColor=0b1730&logoColor=c9ced8&logo=sqlite" alt="SQL">
</p>

<p align="center">
  <img src="screenshots/discount-cliff.jpg" alt="Discount Cliff dashboard" width="100%">
</p>

A short walkthrough video is available in [`assets/Discount_Profit_Analysis.mp4`](assets/Discount_Profit_Analysis.mp4).

## The question

Sales grew 51% between 2015 and 2018 — but is part of that growth paid for with discounts that lose money? The report follows profit margin across discount levels to find the point where discounts stop paying off.

## Dataset

`data/Superstore_Clean.xlsx` — the Superstore sample dataset after cleaning: 9,994 order lines, 26 columns, with a data quality sheet (0 blank cells, 0 duplicates, 0 invalid values). Added columns include `discount_band`, `days_to_ship`, `is_returned` and `region_manager`. A `Date` table supports the year-over-year measures.


## Data cleaning & preparation

Data cleaning, validation and preparation by **Fay Naif**. The workbook starts from the public Superstore sample (the source is credited in the `Data Quality` sheet); every record and column was kept, with no deletions or imputed values.

**Quality checks** (all results are recorded in the `Data Quality` sheet of `data/Superstore_Clean.xlsx`):

| Check | Result |
| --- | ---: |
| Records / columns | 9,994 / 26 |
| Blank cells | 0 |
| Duplicate records / duplicate `row_id` | 0 / 0 |
| Invalid numbers / invalid dates | 0 / 0 |
| Ship date before order date | 0 |
| `order_year`, `order_month`, `days_to_ship` mismatches | 0 |
| Discount outside 0–1 / invalid quantity / invalid return flag | 0 |
| Cells with leading or trailing spaces | 0 |
| Rows with negative profit (kept as valid data) | 1,871 |

**Columns added:** `order_year`, `order_month`, `days_to_ship`, `discount_band`, `is_returned`, `region_manager`.

**Power Query steps** (full code in [`power-query/queries.pq`](power-query/queries.pq)): load the workbook through the `FilePath` parameter → select the 26 columns → set data types (dates, numbers, text IDs and postal codes) → add `discount_band_order` so discount bands sort correctly. A DAX `Date` table (Year, Quarter, Month) is built from the order dates.

```powerquery
Typed = Table.TransformColumnTypes(Selected, {{"order_date", type date}, {"ship_date", type date}, {"sales", type number}, {"discount", type number}, {"profit", type number}, ...}, "en-US"),
BandOrder = Table.AddColumn(Typed, "discount_band_order", each if [discount_band] = "0%" then 1 else if [discount_band] = "1-10%" then 2 else ... else 6, Int64.Type)
```

## Report

One 1920×1080 page, **Discount Cliff**:

- Five KPI cards for 2018 compared with 2017: sales, profit, margin, orders and the loss from discounts above 20%
- Profit margin by discount band (losses in terracotta) with the share of order lines losing money
- Yearly profit vs. the loss from deep discounts
- Profit by sub-category, margin by region and the five states with the biggest losses
- A "what should we do?" box with the recommendation

## Key findings

| Discount band | Profit margin |
| --- | ---: |
| 0% | 29.5% |
| 1–10% | 16.6% |
| 11–20% | 11.6% |
| 21–30% | −10.0% |
| 31–50% | −24.8% |
| Over 50% | −119.2% |

- Margin turns negative as soon as the discount goes above 20% — this is the "cliff".
- Discounts above 20% cover 14% of order lines, and 97% of those lines lose money; together they lose $135,376.
- In 2018 the loss from deep discounts was $47.6K, up 51.7% from 2017, while total profit grew 14.2% to $93.4K and margin fell 0.7 points to 12.7%.

| 2018 KPI | Value | vs. 2017 |
| --- | ---: | ---: |
| Sales | $733.2K | ▲ 20.4% |
| Profit | $93.4K | ▲ 14.2% |
| Margin | 12.7% | ▼ 0.7 pts |
| Orders | 1,687 | ▲ 28.3% |
| Loss from discounts over 20% | −$47.6K | ▲ 51.7% |

## Key measures

The semantic model contains 59 DAX measures in display folders (Story, KPI cards, KPIs, Time Intelligence, Top 5 loss states, Returns &amp; Shipping); the full list is in [`dax/DAX-measures.dax`](dax/DAX-measures.dax).

```dax
Deep Discount Profit =
    CALCULATE ( [Total Profit], Orders[discount] > 0.2 )

Margin YoY pts =
    IF ( NOT ISBLANK ( [Margin LY] ), ( [Profit Margin %] - [Margin LY] ) * 100 )
```

## Python &amp; SQL

The data checks and the main analysis are also written in code, so they can be re-run outside Power BI:

| File | What it does |
| --- | --- |
| [`python/data_cleaning.py`](python/data_cleaning.py) | Rebuilds the derived columns (year, month, days to ship, discount band, region manager), checks they match, runs the 16 quality checks and prints the KPIs and margin by discount band |
| [`python/load_to_sqlite.py`](python/load_to_sqlite.py) | Loads the data into a SQLite database and runs every query in `sql/analysis.sql` |
| [`sql/analysis.sql`](sql/analysis.sql) | 10 queries: KPIs, return rate, yearly sales with YoY growth (window function), margin by discount band, deep-discount lines, loss-making sub-categories, region × segment, top 10 products, shipping days, returns by region manager |

```bash
pip install -r python/requirements.txt
python python/data_cleaning.py
python python/load_to_sqlite.py
```

The results match the figures in the dashboard.

## Tools

- **Power BI Desktop** — report, semantic model and custom theme (`Aubergine Terracotta`)
- **DAX** — measures, time intelligence, conditional colours and rankings
- **Power Query (M)** — loading the clean workbook through a `FilePath` parameter
- **Excel** — data cleaning and quality checks
- **Python (pandas)** — data checks and preparation
- **SQL (SQLite)** — analysis queries

## Project structure

```
Discount-Impact-Analysis/
├── dashboard/
│   ├── Superstore_Discount_Story.pbip              # open this in Power BI Desktop
│   ├── Superstore_Discount_Story.Report/           # report page and visuals
│   ├── Superstore_Discount_Story.SemanticModel/    # data model, Power Query, DAX
│   └── Aubergine_Terracotta_Theme.json             # Power BI theme
├── dax/
│   └── DAX-measures.dax
├── power-query/
│   └── queries.pq
├── python/
│   ├── data_cleaning.py
│   ├── load_to_sqlite.py
│   └── requirements.txt
├── sql/
│   └── analysis.sql
├── data/
│   └── Superstore_Clean.xlsx
├── screenshots/
├── assets/
│   └── Discount_Profit_Analysis.mp4
├── README.md
└── README_AR.md
```

## How to open

1. Clone the repository or download it as a ZIP and extract it (do not open files from inside the ZIP).
2. Open `dashboard/Superstore_Discount_Story.pbip` in a recent version of Power BI Desktop.
3. Click **Home → Refresh**. The data is embedded in the model, so the report works without moving any file.
4. To read from the workbook directly, set **Transform data → Edit parameters → FilePath** to the full path of `data/Superstore_Clean.xlsx`, then refresh.

If Power BI shows a privacy-levels prompt, set **File → Options and settings → Options → Privacy → Ignore the Privacy Levels** and refresh again. To keep a single file, use **File → Save as → Power BI file (.pbix)**.

## نبذة بالعربي

قصة بيانات من صفحة وحدة بسؤال: «الخصومات تجيب لنا نمو… ولا تحرق أرباحنا؟» — تتبّع هامش الربح حسب نسبة الخصم في بيانات Superstore (2015–2018) وتبيّن إن أي خصم فوق 20% يقلب الربح خسارة. التفاصيل الكاملة في [README_AR.md](README_AR.md).

---

<p align="center"><b>Author:</b> Fay Naif · <a href="https://github.com/fay-naif">GitHub</a> · <a href="https://www.linkedin.com/in/fay-naif-4295663b7">LinkedIn</a></p>
