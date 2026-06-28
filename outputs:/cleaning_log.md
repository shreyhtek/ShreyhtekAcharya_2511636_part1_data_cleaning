# Data Cleaning Log — Part 1: Data Cleaning
**Date:** 2025  
**Analyst:** Data Analyst  
**Source File:** `data/raw_orders.xlsx`  
**Output File:** `data/cleaned_orders.xlsx`

---

## 1. Issues Found

| # | Issue | Field(s) | Count |
|---|-------|----------|-------|
| 1 | Inconsistent text casing (ALL CAPS, mixed case) | segment, region, ship_mode, order_status, payment_status | ~200 rows |
| 2 | Leading/trailing/extra spaces | All text fields | ~150 rows |
| 3 | Missing region values | region | 26 rows |
| 4 | Missing ship_mode values | ship_mode | 22 rows |
| 5 | Missing discount values | discount | 18 rows |
| 6 | Negative discount values | discount | ~12 rows |
| 7 | Discount stored as percentage string (e.g. "70%") | discount | ~5 rows |
| 8 | Discount above 100% (> 1.0 decimal) | discount | ~3 rows |
| 9 | Inconsistent date formats (7 different formats) | order_date, ship_date | All rows |
| 10 | Ship date earlier than order date | order_date, ship_date | ~15 rows |
| 11 | Exact duplicate rows | All columns | 32 rows |
| 12 | Duplicate order_id with conflicting data | order_id | ~31 rows |

---

## 2. Cleaning Actions Performed

### Task 2 – Text Fields
- Applied `.strip()` and `.title()` to all text columns: `customer_name`, `segment`, `region`, `state`, `city`, `category`, `sub_category`, `ship_mode`, `payment_status`, `order_status`
- Collapsed multiple internal spaces using regex `\s{2,}` → single space
- Standardized known variants (e.g. `"Small  Business"` → `"Small Business"`)
- Tools used: `str.strip()`, `str.title()`, `str.replace()` with regex

### Task 3 – Dates
- Parsed 7 date formats: `%d %b %Y`, `%m/%d/%Y`, `%d-%m-%Y`, `%Y-%m-%d`, `%d/%m/%Y` and pandas auto-detect
- Standardized all dates to `YYYY-MM-DD` format
- Created `date_flag` column for: missing dates, ship before order date
- Created `shipping_delay_days` = `ship_date` − `order_date`

### Task 4 – Duplicates
- Identified 32 exact duplicate rows → removed all duplicates, kept first occurrence
- Identified ~31 `order_id` values appearing with different data → flagged as `'Review: duplicate order_id'`; kept first occurrence
- Summary documented in `outputs/data_quality_report.xlsx` → sheet "Duplicate Summary"

### Task 5 – Business Rules
- **Missing `region`**: Filled with `"Unknown"`, flagged in quality report
- **Missing `ship_mode`**: Filled with `"Unknown"`, flagged in quality report
- **Missing `discount`**: Treated as `0` (only where all sales fields valid); flagged
- **Negative discount**: Flagged as `"Invalid: negative discount"`; set `cleaned_discount = 0`
- **Discount > 100%**: Flagged as `"Invalid: discount above 100%"`; set `cleaned_discount = 0`
- **Discount as %string** (e.g. `"70%"`): Converted to decimal (0.70)
- **Cancelled/Failed/Refunded orders**: Excluded from completed sales pivot summaries
- **Refunded orders**: Summarized separately in pivot report
- **Ship date before order date**: Flagged as `"Invalid shipping record"` in `date_flag`

---

## 3. Calculated Columns Added

| Column | Formula |
|--------|---------|
| `cleaned_discount` | Clamped discount between 0 and 1; invalid discounts → 0 |
| `calculated_sales` | `quantity × unit_price × (1 − cleaned_discount)` |
| `calculated_profit` | `calculated_sales − cost` |
| `profit_margin` | `calculated_profit / calculated_sales` |
| `shipping_delay_days` | `ship_date − order_date` (days) |
| `order_month` | Month extracted from `order_date` |
| `order_year` | Year extracted from `order_date` |
| `data_quality_flag` | `Clean` / `Warning` / `Invalid` based on combined flags |

---

## 4. Assumptions Made

1. Missing discount with valid sales fields → treated as 0 (not invalid)
2. For conflicting duplicate `order_id`, the **first** occurrence is retained
3. "Percentage" discounts stored as strings (e.g. `"70%"`) are converted to decimals
4. `calculated_sales` is considered the correct value; the raw `sales` column may contain reporting mismatches
5. `Cancelled`, `Returned`, `Refunded`, and `Failed` orders are **excluded** from completed sales summaries
6. Title Case standardization applied uniformly; no manual override except for documented variants

---

## 5. Records Removed

- **32 exact duplicate rows** removed
- **~31 conflicting duplicate order_id records** kept but flagged (not deleted)

---

## 6. Records Flagged

| Flag Type | Count |
|-----------|-------|
| Clean | 795 |
| Warning | 70 |
| Invalid | 35 |

---

## 7. Limitations

- Date ambiguity: formats like `01/07/2024` could be Jan 7 or Jul 1 — assumed `MM/DD/YYYY` for slash-separated dates unless year appears first
- Sales mismatches between `sales` and `calculated_sales` may be due to rounding in source system
- Duplicate order_id resolution is conservative (keep first); a business review is recommended
- No external reference data available to validate customer or product IDs
