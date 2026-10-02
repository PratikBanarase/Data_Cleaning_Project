# Data Cleaning Summary Report

**Source file:** `data_cleaning_raw_dataset.csv`  |  **Output file:** `cleaned_dataset.csv`

## Overview
| Metric | Before | After |
|---|---|---|
| Rows | 20 | 19 |
| Columns | 9 | 18 (incl. encoded columns) |
| Missing values | 8 (plus hidden bad values) | 0 |
| Duplicate rows | 0 visible (1 hidden) | 0 |

## Missing values found (raw data)
| Column | Missing |
|---|---|
| Age | 2 |
| Email | 2 |
| Purchase_Amount | 3 |
| Purchase_Date | 1 |

## Actions taken
- Invalid/missing emails found: 3
- Age: 2 missing -> filled with median (30)
- Purchase_Amount: 3 missing -> filled with category-wise median
- Email: 3 missing/invalid -> set to 'unknown' + Email_Valid flag column
- Purchase_Date: 1 missing -> forward-filled from previous record
- Duplicates: 1 exact duplicate row(s) removed (Customer_ID C003)
- Columns renamed to snake_case and dtypes corrected (age->int, purchase_amount->float, purchase_date->datetime, text->category)
- Outliers: age -> 1 value(s) outside IQR bounds [16.5, 44.5] replaced with median; purchase_amount -> 0 outliers
- Encoding: gender -> label (Female=0, Male=1); city & product_category -> one-hot (7 new columns)
- Validation: 0 missing values, 0 duplicates, all assertions passed
- Exported cleaned dataset -> cleaned_dataset.csv (19 rows x 18 columns)

## Other fixes
- Names: trimmed extra spaces and applied Title Case.
- Gender: M/F/male/female unified to Male/Female.
- City: `pune` and `Mumbai ` (trailing space) unified.
- Product category: `electronics` and `Clothing ` unified.
- Age: text value `thirty` converted to 30.
- Purchase_Amount: removed commas and the rupee symbol, converted to float.
- Purchase_Date: 6 different formats parsed into one datetime format (day-first assumed for dd/mm/yyyy and dd-mm-yyyy).

## Assumptions & limitations
- Ambiguous dates such as `05/04/2026` were read as 5 April (day-first).
- Imputed values (age median, category-median amount, forward-filled date) are estimates, not real observations.
- Customer ID C016 is absent from the source data; it was not invented.
- Rows with an unknown email are kept; use the `email_valid` flag to filter them.

## Final column list
`customer_id`, `customer_name`, `age`, `gender`, `city`, `email`, `purchase_amount`, `purchase_date`, `product_category`, `email_valid`, `gender_encoded`, `city_mumbai`, `city_nagpur`, `city_nashik`, `city_pune`, `category_clothing`, `category_electronics`, `category_grocery`