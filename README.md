# Data Cleaning Project

A step-by-step data cleaning pipeline in Python that turns a messy customer purchase dataset into an analysis- and model-ready CSV. It covers missing values, duplicates, inconsistent formats, outliers, encoding and validation, and finishes with an auto-generated cleaning report.

**Libraries:** Pandas · NumPy · Seaborn · Matplotlib

---

## Project Structure

```
.
├── Data_Cleaning_Project.ipynb        # Notebook version (run cell by cell)
├── data_cleaning.py                   # Script version of the same pipeline
├── data_cleaning_raw_dataset.csv      # Input: raw, messy data (20 rows x 9 cols)
├── cleaned_dataset.csv                # Output: cleaned + encoded data (19 rows x 18 cols)
├── cleaning_summary_report.md         # Output: auto-generated cleaning report
├── figures/                           # Output: saved plots
│   ├── 01_missing_heatmap_before.png
│   ├── 02_boxplots_before.png
│   ├── 03_boxplots_after.png
│   ├── 04_missing_heatmap_after.png
│   └── 05_cleaned_overview.png
└── README.md
```

## Requirements

- Python 3.9+
- pandas (2.0 or later; tested on 3.0), numpy, seaborn, matplotlib

```bash
pip install pandas numpy seaborn matplotlib
```

## How to Run

Keep the raw CSV in the same folder as the notebook or script.

**Notebook**

```bash
jupyter notebook Data_Cleaning_Project.ipynb
```

Then run all cells from top to bottom.

**Script**

```bash
python data_cleaning.py
```

Both create `cleaned_dataset.csv`, `cleaning_summary_report.md` and the `figures/` folder.

---

## Dataset

Customer purchase records with these columns:

| Column | Description |
|---|---|
| Customer_ID | Unique customer identifier (C001–C020) |
| Customer_Name | Customer's full name |
| Age | Age in years |
| Gender | Male / Female |
| City | Pune, Mumbai, Nashik, Nagpur |
| Email | Contact email |
| Purchase_Amount | Amount spent (INR) |
| Purchase_Date | Date of purchase |
| Product_Category | Electronics, Clothing, Grocery |

### Problems in the raw data

- Missing values in Age (2), Email (2), Purchase_Amount (3) and Purchase_Date (1)
- Age stored as text, including the word `thirty` and an impossible value `120`
- Purchase_Amount stored as text with commas and the `₹` symbol (`"12,500"`, `₹12000`)
- Purchase_Date in 6 different formats (`2026-01-15`, `15/01/2026`, `2026/02/10`, `March 5, 2026`, `12-03-2026`, ...)
- Inconsistent labels: `M` / `male` / `Female`, `pune` / `Pune`, `electronics` / `Electronics`
- Extra leading, trailing and double spaces in names, cities and categories
- An invalid email (`vikas@gmail`)
- A duplicate record (C003) hidden by different formatting

---

## Pipeline Steps

| # | Step | What it does |
|---|---|---|
| 1 | Load & explore | `.head()`, `.info()`, `.describe()`, shape, missing counts |
| 2 | Visualise missing data | Seaborn heatmap of nulls before cleaning |
| 3 | Missing-value strategy | Standardise text and numbers, then impute per column (see below) |
| 4 | Duplicates | `.duplicated()` and `.drop_duplicates()` |
| 5 | Rename & retype | snake_case column names, correct dtypes |
| 6 | Outliers | Box plots and the IQR method (1.5 × IQR) |
| 7 | Encoding | Label encoding for gender, one-hot encoding for city and category |
| 8 | Validation | Summary statistics, value counts, assertions, after-cleaning heatmap |
| 9 | Export | Save `cleaned_dataset.csv` |
| 10 | Report | Generate `cleaning_summary_report.md` |

### Missing-value strategy

| Column | Strategy | Reason |
|---|---|---|
| Age | Median | Robust to the outlier value 120 |
| Purchase_Amount | Median of the same product category | Spend differs by category |
| Email | Set to `unknown` plus an `email_valid` flag | An email cannot be guessed |
| Purchase_Date | Forward-fill from the previous record | Records are in chronological order |

### Outliers

- **Age:** the IQR method flags 120 (customer C017). It is a data-entry error, so it is replaced with the median.
- **Purchase_Amount:** no outliers found.

### Encoding

- `gender` → `gender_encoded` (Female = 0, Male = 1)
- `city` → `city_mumbai`, `city_nagpur`, `city_nashik`, `city_pune`
- `product_category` → `category_clothing`, `category_electronics`, `category_grocery`

The original text columns are kept for readability. Drop them before training a model.

---

## Results

| Metric | Before | After |
|---|---|---|
| Rows | 20 | 19 |
| Columns | 9 | 18 (including encoded columns) |
| Missing values | 8 (plus hidden bad values) | 0 |
| Duplicate rows | 0 visible (1 hidden) | 0 |

The validation step uses assertions to confirm there are no missing values or duplicates, ages are between 0 and 100, amounts are positive, and gender contains only `Male` and `Female`.

### Final columns

`customer_id`, `customer_name`, `age`, `gender`, `city`, `email`, `purchase_amount`, `purchase_date`, `product_category`, `email_valid`, `gender_encoded`, `city_mumbai`, `city_nagpur`, `city_nashik`, `city_pune`, `category_clothing`, `category_electronics`, `category_grocery`

---

## Assumptions & Limitations

- **Date format:** ambiguous dates such as `05/04/2026` are read day-first (5 April). Year-first dates such as `2026-03-12` are parsed separately so they are not misread.
- **Imputed values** (age, amount, date) are estimates, not real observations.
- **Duplicates:** C003 only appears as a duplicate after formatting is standardised, so the duplicate check runs after cleaning the values.
- **Missing ID:** Customer ID C016 is absent from the source data and was not invented.
- **Unknown emails:** rows with an unknown email are kept; filter on `email_valid` if needed.
- **Small dataset:** with only 20 rows, medians and IQR bounds are sensitive to individual values.

## Possible Next Steps

- Add regex-based name validation and phone/email domain checks
- Use `missingno` for extra missing-data visuals
- Turn the notebook into a reusable function or pipeline class
- Run the same pipeline on larger real-world datasets
