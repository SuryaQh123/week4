# 🛒 ShopSphere E-Commerce Sales Analysis

A complete, end-to-end data analysis project built for **Week 4: Data Visualization & Your First Complete Project**.
This project loads, cleans, analyzes, and visualizes two years of e-commerce order data, producing four
charts and a written insights report — fully runnable in **Google Colab** with zero setup.

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![Pandas](https://img.shields.io/badge/pandas-2.x-150458)
![Matplotlib](https://img.shields.io/badge/matplotlib-3.x-11557c)
![Status](https://img.shields.io/badge/status-complete-brightgreen)

---

## 📌 1. Project Overview

**Topic chosen:** E-Commerce Sales Analysis (Project Option 4)

**Business context:** *ShopSphere* is a fictional online retailer. Two years of order-level
transaction data (Jan 2024 – Dec 2025) were analyzed to answer four core business questions:

| # | Question | Chart used |
|---|---|---|
| 1 | Which product categories drive the most revenue? | Bar chart |
| 2 | How does revenue trend month-to-month — is there seasonality? | Line chart |
| 3 | How is revenue distributed across regions? | Pie chart |
| 4 | Which payment methods do customers actually use? | Horizontal bar chart |

**Goals / objectives:**
- Practice a complete, real-world data pipeline: **load → clean → analyze → visualize → report**
- Work with realistically messy data (missing values, duplicates, inconsistent text, invalid entries)
- Produce at least 2 different chart types with matplotlib (this project ships **4**)
- Translate raw numbers into clear, written, actionable business insights
- Validate the cleaning and analysis steps with explicit tests, not just "it ran without crashing"

**Dataset:** 4,615 synthetic but realistic order records covering 6 product categories, 4 regions,
5 payment methods, and 2 customer segments. See [`data/`](data/) for details.

---

## ⚙️ 2. Setup Instructions

### Option A — Google Colab (recommended, what this project was built for)

1. Go to [colab.research.google.com](https://colab.research.google.com)
2. `File → Upload notebook` → select `analysis.ipynb` from this project
3. **Either:**
   - **Quick start (no upload needed):** Just click `Runtime → Run all`. If the notebook
     doesn't find `data/ecommerce_sales.csv` in the Colab session, it **automatically
     regenerates an identical dataset** using a fixed random seed — every cell works
     immediately, no setup required.
   - **Use the real shipped data:** Upload this entire project folder to your Google Drive,
     then in the notebook uncomment the two `drive.mount(...)` lines near the top and set
     `PROJECT_PATH` to where you placed the folder. Then `Runtime → Run all`.
4. Charts will be displayed inline and also saved into `visualizations/`.

### Option B — Run locally (Jupyter / VS Code)

```bash
# 1. Clone or download this repository
git clone <your-repo-url>
cd ecommerce-sales-analysis

# 2. Create a virtual environment (recommended)
python3 -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Launch Jupyter and open analysis.ipynb
jupyter notebook analysis.ipynb
# then: Run All Cells
```

**Requirements:** Python 3.10+, see [`requirements.txt`](requirements.txt) for exact package
versions used during development and testing.

---

## 🗂️ 3. Code Structure

```
ecommerce-sales-analysis/
│
├── README.md                          # This file — full project documentation
├── analysis.ipynb                     # Main notebook: load → clean → analyze → visualize
├── requirements.txt                   # Python dependencies with pinned versions
│
├── data/
│   ├── ecommerce_sales.csv            # Raw dataset (4,615 rows, intentionally messy)
│   └── ecommerce_sales_cleaned.csv    # Cleaned dataset produced by the notebook
│
├── visualizations/
│   ├── 01_revenue_by_category_bar.png       # Bar chart  — revenue by category
│   ├── 02_monthly_revenue_trend_line.png    # Line chart — monthly revenue trend
│   ├── 03_revenue_share_by_region_pie.png   # Pie chart  — revenue share by region
│   └── 04_orders_by_payment_method_bar.png  # Bar chart  — orders by payment method
│
└── report/
    └── project_report.md              # Full written report with embedded charts & insights
```

**Notebook section flow** (`analysis.ipynb`):

1. Setup & imports
2. *(Optional)* Mount Google Drive
3. **Load** data — with automatic fallback generation if the CSV isn't present
4. **Explore** the raw data — dtypes, missing values, duplicates, inconsistent text
5. **Clean** the data — standardize text, fix dates, drop invalid rows, fill missing values
6. **Validate** the cleaning with 5 explicit assertions (see Testing Evidence below)
7. **Analyze** — KPIs, revenue by category/month/region/payment method
8. **Visualize** — 4 charts, saved to `visualizations/`
9. Save cleaned dataset
10. Written insights & conclusion

---

## 🖼️ 4. Visual Documentation

All four charts below are generated directly by `analysis.ipynb` and saved automatically
into `visualizations/`.

| Chart | Type | What it shows |
|---|---|---|
| `01_revenue_by_category_bar.png` | Bar | Electronics generates the majority of revenue despite a moderate order share |
| `02_monthly_revenue_trend_line.png` | Line | Clear Oct–Dec seasonal spike, both years |
| `03_revenue_share_by_region_pie.png` | Pie | Revenue fairly evenly split across 4 regions, North leading |
| `04_orders_by_payment_method_bar.png` | Horizontal bar | Credit Card & UPI dominate; "Unknown" flags a data-quality gap worth fixing upstream |

See [`report/project_report.md`](report/project_report.md) for each chart embedded next to
its full written analysis.

---

## 🔧 5. Technical Details

**Pipeline / architecture:**
```
ecommerce_sales.csv (raw, messy)
        │
        ▼
  ── CLEANING ──
  • str.strip().str.title()  on text columns → fixes "electronics"/"ELECTRONICS"/"Electronics"
  • drop_duplicates()        → removes 15 exact duplicate rows
  • filter qty>0 & price>0   → removes 67 data-entry error rows
  • fillna(0) / fillna("Unknown") on Discount % and Payment Method
  • pd.to_datetime()         → proper datetime dtype for Order Date
        │
        ▼
  ── DERIVED FIELDS ──
  Revenue (INR) = Quantity × Unit Price × (1 − Discount/100)
  Month  = Order Date → 'YYYY-MM' period string
        │
        ▼
  ── ANALYSIS ── (pandas groupby/aggregation)
  • groupby("Product Category")["Revenue"].sum()
  • groupby("Month")["Revenue"].sum()              → time series
  • groupby("Region")["Revenue"].sum()
  • value_counts() on Payment Method
        │
        ▼
  ── VISUALIZATION ── (matplotlib.pyplot)
  • Bar, line, pie, horizontal bar — each with titles/axis labels/legends
        │
        ▼
  visualizations/*.png  +  data/ecommerce_sales_cleaned.csv
```

**Key design decisions:**
- **Revenue only counts "Delivered" orders** for KPIs and chart 1–3 (returned/cancelled
  orders never generated real revenue), while chart 4 (payment method volume) uses *all*
  orders since that reflects checkout behavior regardless of fulfillment outcome.
- **Missing `Discount (%)` → 0**, not dropped, because a missing discount almost certainly
  means "no discount was applied," which is a valid business state, not bad data.
- **Missing `Payment Method` → "Unknown"** rather than dropped, because losing ~115 orders
  worth of revenue data to fix a labeling gap would distort the analysis more than it helps.
- **Random seed fixed at 42** (and 1, 7 for shuffling/sampling steps) so the dataset and
  results are 100% reproducible on any machine.

**Error handling:** the notebook includes a `try`/fallback path for data loading (Section 1)
and 5 explicit `assert`-based validation checks (Section 4) that will halt execution with a
clear error message if the cleaned data doesn't meet expected conditions — see below.

---

## ✅ 6. Testing Evidence

Section 4 of `analysis.ipynb` ("Validate the Cleaning") runs five explicit checks on the
cleaned data before any analysis is allowed to proceed. Output from an actual run:

```
✅ Test 1 passed: no missing values in required columns
✅ Test 2 passed: all quantities and prices are positive
✅ Test 3 passed: no duplicate rows
✅ Test 4 passed: category values are standardized
✅ Test 5 passed: Revenue formula verified on a random sample

🎉 All validation checks passed — the cleaned dataset is ready for analysis.
```

| Test | What it checks | Why it matters |
|---|---|---|
| 1 | No missing values in any column the analysis depends on | Prevents silent `NaN`-propagation into KPIs |
| 2 | All `Quantity` and `Unit Price` values are positive | Confirms data-entry errors were actually removed |
| 3 | No duplicate rows remain | Confirms `drop_duplicates()` worked |
| 4 | `Product Category` values belong to the expected fixed set | Confirms casing standardization fully worked |
| 5 | `Revenue` recomputed independently on a random sample matches the stored value | Confirms the revenue formula is implemented correctly, not just "ran without error" |

The notebook was also executed twice end-to-end during development — once with the shipped
`data/ecommerce_sales.csv` present, and once in a completely empty folder to confirm the
auto-generation fallback produces an equivalent, fully working result. Both runs completed
with zero errors and all five tests passing.

---

## 📊 7. Key Insights (summary — full detail in the report)

1. **Electronics drives ~65% of total revenue** despite being only ~22% of orders.
2. **Revenue spikes 50–60% during Oct–Dec** every year — a repeatable seasonal pattern.
3. **Revenue is fairly evenly spread across 4 regions** (19–30% each), with North leading
   and East trailing — a candidate for regional promotions.
4. **Credit Card and UPI dominate payments**; a small "Unknown" bucket flags a real
   checkout-logging gap worth fixing on the live platform.
5. **New and returning customers have nearly identical order values** — the real lever for
   revenue growth is purchase *frequency*, not basket size.

Full reasoning and supporting numbers: [`report/project_report.md`](report/project_report.md)

---

## 📅 Project Timeline (as followed)

| Day | Task |
|---|---|
| 1–2 | Chose topic (e-commerce sales), designed schema, generated dataset, set up folder structure |
| 3 | Loaded and explored raw data — identified missing values, duplicates, inconsistent text |
| 4 | Cleaned data, derived Revenue/Month fields, computed KPIs and group-by metrics |
| 5 | Built 4 matplotlib charts (bar, line, pie, horizontal bar) with titles/labels |
| 6 | Wrote the full report combining numbers, charts, and insights |
| 7 | Added validation tests, re-ran notebook end-to-end, finalized README, packaged for submission |

---

## 📄 License / Use

This project uses a synthetic dataset created for educational purposes. Feel free to fork,
reuse, or adapt the structure and code for your own data analysis projects.
