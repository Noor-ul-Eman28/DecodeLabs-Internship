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
