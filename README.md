# The Architecture of Data Integrity — Project 1: Data Cleaning & Preparation

**DecodeLabs Internship — Data Analytics Track (Batch 2026)**

## Overview

This project transforms a raw e-commerce orders dataset into a production-ready, "gold standard" dataset by identifying and fixing data quality issues — missing values, duplicate records, and inconsistent formatting. The work follows the three-phase framework covered in training: **Strategic Imputation**, **The Integrity Audit**, and **Speak One Language**.

## Dataset

- **Source:** `dataset.csv` (tab-separated), 1,200 rows × 14 columns
- **Columns:** OrderID, Date, CustomerID, Product, Quantity, UnitPrice, ShippingAddress, PaymentMethod, OrderStatus, TrackingNumber, ItemsInCart, CouponCode, ReferralSource, TotalPrice

## What Was Done

### 1. Missing Values (Strategic Imputation)
- `CouponCode` had 309 missing values (25.75% of rows).
- Rather than deleting these rows (which would reduce statistical power) or filling with a vague placeholder, missing values were filled with `"No Coupon"` — a meaningful category representing orders where no coupon was applied.

### 2. Duplicate Records (The Integrity Audit)
- Checked for full-row duplicates and duplicate `OrderID` values (the unique identifier).
- **Result:** 0 duplicates found on either check. The dataset's identifiers were already unique.

### 3. Format Standardization (Speak One Language)
- **Dates:** Converted to ISO 8601 format (`YYYY-MM-DD`) using `pd.to_datetime()` with error coercion to catch any hidden unparseable dates.
- **Text:** Trimmed whitespace and applied consistent title casing to display-text columns (`Product`, `PaymentMethod`, `OrderStatus`, `ReferralSource`, `ShippingAddress`). `CouponCode` was trimmed only (not title-cased) since it's a code, not display text — preserving formats like `SAVE10`.
- **Numbers:** Rounded `UnitPrice` and `TotalPrice` to 2 decimal places for consistent precision.

## Verification Gate Results

| Check | Result |
|---|---|
| Missing values remaining | 0 |
| Duplicate OrderIDs | 0 (0.00%) |
| Invalid/unparseable dates | 0 (0.00%) |
| Final row count | 1,200 |

Both required gates — **0% duplicate identifiers** and **0% incorrectly formatted dates** — are satisfied.

## Files

- `project.ipynb` — full cleaning pipeline with markdown documentation and inline comments
- `dataset.csv` — original raw data
- `cleaned_dataset.csv` — final cleaned output
- `change_log.pdf` — change log documenting what was changed and why (Change ID, Description, Impact, Status)

## Tools Used

Python, pandas — chosen for scalability and reproducibility over manual spreadsheet editing.

## Key Takeaway

The dataset arrived largely clean on structural checks (no duplicates, valid dates already in ISO format), with its main real issue being systematic missing data in one column. This distinction mattered: the correct fix was to treat the missing `CouponCode` values as a genuine category ("no coupon used") rather than as an error to be imputed with mean/median/mode, which would have misrepresented the underlying business reality of the data.

---

# Project 2: Exploratory Data Analysis (EDA)

## Overview

Building on the Project 1 output, this phase analyzes `cleaned_dataset.csv` (1,200 rows, 0 missing values, 0 duplicates) to uncover patterns, trends, and outliers — calculating descriptive statistics, checking distribution shape, detecting outliers, running correlation analysis, and translating findings into business-relevant insight.

## What Was Done

### 1. Basic Statistics
Ran `.describe()` on all four numeric columns (`Quantity`, `UnitPrice`, `ItemsInCart`, `TotalPrice`) to get count, mean, median, and quartiles in one pass.

### 2. Distribution Shape
Plotted histograms for `UnitPrice`, `TotalPrice`, and `Quantity`. `TotalPrice` came back right-skewed (mean \$1,053.97 vs. median \$823.62), while `UnitPrice` and `Quantity` were close to uniform.

### 3. Outlier Detection (IQR Method)
Applied the standard IQR rule (outside Q1 − 1.5×IQR / Q3 + 1.5×IQR) via boxplots across all four numeric columns.
- **Result:** 8 outlier orders (0.67%) on `TotalPrice` only, all combining high `Quantity` with high `UnitPrice` — judged as genuine high-value orders (signal), not data errors (noise).

### 4. Correlation Analysis
Computed a Pearson correlation matrix and heatmap across the numeric columns.
- `UnitPrice` ↔ `TotalPrice`: r = 0.72, `Quantity` ↔ `TotalPrice`: r = 0.62, `Quantity` ↔ `ItemsInCart`: r = 0.65, `UnitPrice` ↔ `Quantity`: r ≈ 0.01.
- Explicitly noted that the TotalPrice correlations are a mathematical artifact of `TotalPrice = Quantity × UnitPrice`, not an independent discovery — correlation ≠ causation.

### 5. Trend Analysis
- Monthly revenue trend across the 2023–2025 date range (highest: June 2024 at \$68,068.54; lowest: April 2023 at \$27,751.71).
- Order breakdowns by `OrderStatus`, `PaymentMethod`, and `ReferralSource`.

## Key Findings

| Finding | Business Meaning |
|---|---|
| TotalPrice mean is well above its median | Report median, not mean, for "typical order" — mean overstates it |
| 8 statistical outliers on TotalPrice | Likely genuine high-value customers, worth a VIP-segment follow-up rather than exclusion |
| Cancelled + Returned = 41% of all orders | The single most significant finding — points to a real fulfillment/checkout issue worth investigating |
| Instagram leads referral revenue (\$275,285) | Candidate for increased acquisition spend, pending CAC data |
| UnitPrice and Quantity are uncorrelated | No bulk-buying-discount pattern present in this data |

## Recommendations

1. Investigate the 41% cancelled/returned rate as the top priority.
2. Use median (not mean) order value when reporting "typical" spend.
3. Follow up on the 8 high-value outlier orders as a potential repeatable customer segment.
4. Evaluate shifting acquisition spend toward Instagram if CAC data becomes available.

## Files

- `Project2_EDA.ipynb` — full EDA notebook with statistics, visualizations, and written findings, executed against `cleaned_dataset.csv`
- `distributions.png`, `boxplots.png`, `correlation_heatmap.png`, `monthly_revenue.png`, `categorical_trends.png` — exported charts from the notebook

## Tools Used

Python, pandas, matplotlib, seaborn.
