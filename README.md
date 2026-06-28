Part 1: Data Cleaning — Retail Orders Dataset

1. Problem Summary

A retail company exports order-level sales data from multiple internal systems. The raw dataset contains issues such as inconsistent text formatting, date format problems, duplicate records, missing values, invalid discounts, and order status inconsistencies. The goal of this project is to clean the dataset, validate business rules, document all issues found, and create summary reports using Excel.


2. Dataset Description

AttributeDetailSource filedata/raw_orders.xlsxRaw row count932 rowsCleaned row count900 rowsColumns (raw)21Columns (cleaned)33 (21 original + 12 calculated/flag columns)Date range2024 – 2025Fieldsorder_id, order_date, ship_date, customer_id, customer_name, segment, region, state, city, category, sub_category, product_name, ship_mode, quantity, unit_price, discount, sales, cost, profit, payment_status, order_status


3. Tools Used


Python 3 — data cleaning, transformation, and file generation
pandas — data manipulation and analysis
openpyxl — Excel file creation and formatting
Excel — for reviewing output pivot and quality report files



4. Cleaning Steps Performed

Step 1 – Preserve Raw Data

The original data/raw_orders.xlsx was not modified. All cleaning was performed on a separate copy saved as data/cleaned_orders.xlsx.

Step 2 – Clean Text Fields

Applied to: customer_name, segment, region, state, city, category, sub_category, ship_mode, payment_status, order_status


Stripped leading/trailing whitespace
Collapsed multiple internal spaces to single space
Standardized to Title Case
Fixed known variants (e.g. "Small  Business" → "Small Business", "STANDARD CLASS" → "Standard Class")


Step 3 – Clean and Validate Dates


Parsed 7 different date formats across order_date and ship_date
Standardized all dates to YYYY-MM-DD
Flagged missing dates and ship dates earlier than order dates
Created shipping_delay_days column = ship_date − order_date


Step 4 – Handle Duplicates


Identified and removed 32 exact duplicate rows
Identified ~31 duplicate order_id values with conflicting data — kept first occurrence and flagged for review
Full summary documented in outputs/data_quality_report.xlsx


Step 5 – Apply Business Rules


Missing region → filled as "Unknown", flagged
Missing ship_mode → filled as "Unknown", flagged
Missing discount → treated as 0 where all sales fields are valid
Negative discounts → flagged as invalid
Discount > 100% → flagged as invalid
Percentage string discounts (e.g. "70%") → converted to decimal (0.70)
Cancelled, Refunded, and Failed orders → excluded from completed sales summaries
Refunded orders → summarized separately


Step 6 – Add Calculated Columns

See table in Section 6 below.


5. Business Rules Applied

Rule AreaAction TakenMissing regionFilled as Unknown; flagged in quality reportMissing ship_modeFilled as Unknown; flagged in quality reportMissing discountTreated as 0 only if all other sales fields are validNegative discountFlagged as invalid; cleaned_discount set to 0Discount above allowed range (>1.0)Flagged as invalid; cleaned_discount set to 0Cancelled ordersExcluded from completed sales pivot summaryFailed paymentsExcluded from completed sales pivot summaryRefunded ordersSummarized separately in pivot reportShip date before order dateFlagged as invalid shipping record


6. Summary of Data Quality Issues Found

IssueCountMissing region values26Missing ship_mode values22Missing discount values18Negative discount values~12Discount as % string (e.g. "70%")~5Discount above 100%~3Inconsistent date formatsAll rows (7 formats)Ship date earlier than order date~15Exact duplicate rows32Duplicate order_id (conflicting data)~31Text case/spacing inconsistencies~200+

Final data quality breakdown (900 cleaned rows):

FlagCount%Clean79588.3%Warning707.8%Invalid353.9%


7. Summary of Final Pivot Reports

All pivot summaries are in outputs/pivot_summary.xlsx. Only Completed + Paid orders are included unless stated otherwise.

SheetKey FindingBy RegionSouth region leads in total sales; all 4 regions representedBy CategoryTechnology is the top-selling category overallBy Ship ModeStandard Class is the most frequently used shipping methodBy SegmentProfit margins are relatively consistent across segmentsBy Region IssuesDistribution of cancelled/returned/failed orders by regionMonthly TrendSales distributed across 2024–2025; visible seasonal patterns

Completed + Paid orders summary:


Total orders: 601
Total calculated sales: ₹58,87,466
Total calculated profit: ₹16,57,009
Top region by sales: South
Top category by sales: Technology



8. Key Business Insights


88% of records are clean — the dataset is largely reliable but has systematic text formatting issues across all text fields.
Discount field is the most problematic — negative values, string percentages, and missing values all appear; this likely indicates a data entry or system export issue.
Date inconsistency is universal — 7 different date formats were found, suggesting data comes from multiple systems with no standardization.
South region leads in sales — consistently high order volume and revenue.
Technology category generates the most revenue — followed by Furniture and Office Supplies.
~10% of orders are Cancelled/Returned/Refunded — this is a meaningful loss rate worth investigating.



9. Assumptions and Limitations

Assumptions


Slash-separated dates (e.g. 01/07/2024) assumed to be MM/DD/YYYY unless the year appears first.
Missing discounts treated as 0 (not missing or erroneous) where all related sales fields are valid.
For conflicting duplicate order_id records, the first occurrence is retained; a business review is recommended.
calculated_sales is treated as the correct value; the raw sales column contains source system rounding mismatches.
Title Case applied uniformly to all text fields.


Limitations


No external reference data available to validate customer_id or product_name consistency.
Date ambiguity cannot be fully resolved without knowledge of the source system locale.
Conflicting duplicate order_id records were conservatively handled — a domain expert should review flagged rows.
Sales mismatches between sales and calculated_sales may reflect discounts or promotions not captured in the dataset.



10. Screenshots

Screenshots of the raw data preview, cleaned data preview, and pivot summaries are included in the screenshots/ folder:


screenshots/raw_data_preview.png — first view of the raw dataset
screenshots/cleaned_data_preview.png — cleaned dataset with new calculated columns
screenshots/pivot_summary_1.png — sales & profit by region and category
screenshots/pivot_summary_2.png — monthly sales trend and ship mode summary



