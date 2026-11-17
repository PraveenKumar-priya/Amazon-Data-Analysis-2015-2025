# Amazon Data Analysis (2015–2025)
## Detailed Project Documentation

## 1. Executive summary

This project analyses annual Amazon India sales datasets covering 2015 through 2025. The work follows an end-to-end analytics workflow: understand the source data, profile data quality, clean and standardise fields, consolidate the yearly files, perform exploratory analysis, load the master data into SQLite, and build a Power BI report for business-facing analysis.

The consolidated master dataset recorded in the project notebook contains **1,127,609 rows and 34 columns**. The project dashboard is organised into four report pages: Executive Overview; Sales and Payment Analysis; Customer Analysis; and Product & Operations Analysis.

## 2. Business objectives

The analysis is intended to help answer questions such as:

- How does revenue change across years and months?
- Which categories, subcategories, brands and products contribute most to revenue?
- Which states and customer segments contribute to sales?
- How do payment methods compare by transaction volume and revenue?
- How do festival sales compare with non-festival sales?
- What does the return-status distribution look like?
- How do delivery types, delivery days and product/customer ratings relate to sales activity?

## 3. Data overview

### 3.1 Source files

The project folder contains annual raw CSV files under `data/raw/` and cleaned annual CSV files under `data/cleaned/`. The cleaning notebook discovers annual files using the pattern `amazon_india_*.csv`, applies cleaning functions, writes cleaned outputs, and combines cleaned files into `amazon_master_2015_2025.csv`.

### 3.2 Master dataset

- Period covered: 2015–2025.
- Recorded shape: 1,127,609 rows × 34 columns.
- Master file: `data/cleaned/amazon_master_2015_2025.csv`.

The fields cover transaction/order identifiers, dates, customers, products, category and brand attributes, price and discount amounts, quantities, delivery charges, customer geography and segments, payment method, delivery information, festival and Prime indicators, return status, ratings, and derived calendar fields.

Examples of columns used in analysis include `transaction_id`, `order_date`, `customer_id`, `product_id`, `product_name`, `category`, `subcategory`, `brand`, `original_price_inr`, `discount_percent`, `discounted_price_inr`, `quantity`, `subtotal_inr`, `delivery_charges`, `final_amount_inr`, `customer_city`, `customer_state`, `customer_tier`, `customer_spending_tier`, `customer_age_group`, `payment_method`, `delivery_days`, `delivery_type`, `is_prime_member`, `is_festival_sale`, `festival_name`, `customer_rating`, `return_status`, `order_month`, `order_year`, `order_quarter`, `product_weight_kg`, `is_prime_eligible`, and `product_rating`.

## 4. Data understanding and profiling

The initial analysis notebook starts with annual CSV loading and dataset inspection. Profiling includes:

- Dataset shape, column names and data types.
- Null counts and null percentages.
- Duplicate row and duplicate transaction-ID checks.
- Frequency distributions for categorical fields.
- Distinct-value checks for customer and product attributes.
- Customer-to-city/state, customer-to-tier and product-ID-to-product-name consistency checks.
- Inspection of mixed date formats and mixed numeric/text representations.

This stage is used to identify data-quality issues and understand which missing values or variations require cleaning rules.

## 5. Data cleaning methodology

The cleaning notebook defines reusable functions and applies them across the annual raw files. The visible notebook logic includes the following workstreams:

### 5.1 Missing-value treatment

- Customer age-group gaps are filled with `Unknown` in the cleaning function.
- Missing customer ratings are preserved rather than inventing a rating.
- Festival-name gaps are treated in relation to the festival flag; non-festival records can be labelled `Not Applicable`.
- Delivery-charge gaps are handled by the notebook's cleaning logic; inspect the exact function and business rule before changing it.

### 5.2 Rating standardisation

Customer-rating text is cleaned by removing a `stars` suffix and extracting the value before a slash where present. The result is converted to numeric using coercion for values that cannot be parsed.

### 5.3 Price standardisation

Price strings are cleaned by removing currency symbols/prefixes and thousands separators, trimming whitespace, and converting the result to numeric. Numeric validation is performed on price fields.

### 5.4 Category and city standardisation

Known category variants such as `ELECTRONICS`, `Electronic`, and `Electronicss` are mapped to `Electronics`. City spelling and naming variants are mapped to standard names, including common Bangalore/Bengaluru and Chennai variants.

### 5.5 Date and dataset validation

The notebooks inspect and parse order dates, check missing or invalid dates, compare raw and cleaned shapes, review remaining missing values, and check duplicate records and transaction identifiers. The cleaning notebook then processes the annual files, saves cleaned CSVs, concatenates them, and writes the consolidated master dataset.

**Data-quality caution:** A cleaning rule should preserve business meaning. A missing rating should not be replaced with an invented score, and festival fields should distinguish a true non-festival order from an unknown festival value. Review the notebook's final functions and validation output before describing any rule as universally applied to every year.

## 6. Exploratory Data Analysis

The EDA notebook loads the consolidated master CSV and performs grouped aggregations. The analysis areas include:

| Analysis area | Example measures / grouping |
|---|---|
| Year-wise revenue | Sum of `final_amount_inr` by `order_year` |
| Category performance | Revenue by `category` |
| Customer performance | Revenue by `customer_id`; top customers |
| Geographic performance | Revenue by `customer_state` |
| Payment analysis | Distinct transaction count and revenue by `payment_method` |
| Return analysis | Transaction count and revenue by `return_status` |
| Festival analysis | Sales comparison using festival indicators |
| Subcategory performance | Revenue by `subcategory` |
| Product performance | Revenue by `product_name` |
| Brand performance | Revenue by `brand` |
| Delivery analysis | Transaction count, average delivery days and revenue by `delivery_type` |
| Customer segments | Revenue, transactions and unique customers by customer tier and age group |
| Ratings | Transaction counts by customer and product rating |

The dashboard turns selected aggregations into visual comparisons and trends. Use the underlying notebook outputs when writing specific findings; do not infer causation from descriptive aggregations.

## 7. SQL database integration

The SQL integration notebook reads `data/cleaned/amazon_master_2015_2025.csv`, creates the `database` folder if needed, and connects to `database/amazon_database.db` with Python's `sqlite3` module.

The master DataFrame is loaded into a SQLite table named `amazon_sales` using `DataFrame.to_sql`, with `if_exists="replace"` and a chunk size of 10,000 rows. The notebook also creates indexes on:

- `customer_id` — `idx_customer_id`
- `order_date` — `idx_order_date`
- `brand` — `idx_brand`

The database supports SQL-based exploration and querying of the consolidated sales data. The database can be recreated from the master CSV by running the SQL integration notebook.

## 8. Power BI dashboard

The Power BI report is saved as `Amazon_Dashboard.pbix` and contains four pages.

### 8.1 Executive Overview

- KPI cards: Total Revenue, Total Orders, Total Customers, Total Products and Average Order Value.
- Year-wise revenue trend.
- Top brands by revenue.
- Revenue contribution by subcategory.
- State-wise revenue.

### 8.2 Sales and Payment Analysis

- Revenue by payment method.
- Monthly revenue trend.
- Festival versus non-festival sales.
- Order/return status analysis.

### 8.3 Customer Analysis

- Customer-level KPI cards.
- Revenue by customer tier.
- Revenue by customer age group.
- Unique customers by tier.
- Top customers by revenue.

### 8.4 Product & Operations Analysis

- Top products by revenue.
- Product-rating distribution.
- Orders by delivery type.
- Order-status distribution.
- Average delivery time by delivery type.

The report's measures and visuals should be validated against equivalent Python and SQL calculations after all slicers and filters are finalised. A Power BI Service link can be added after publishing with an access method appropriate to the dataset's permissions.

## 9. Recorded project metrics

The project work previously recorded the following overall metrics for the master dataset:

| Metric | Recorded value |
|---|---:|
| Rows / transactions | 1,127,609 |
| Revenue | ₹76,888,729,563.25 |
| Revenue in crore | Approximately ₹7,688.87 crore |
| Unique customers | 354,969 |
| Unique products | 2,004 |
| Average order value | Approximately ₹68,187.40 |

These are project-recorded values, not newly recalculated in this documentation pass. Before final submission, run a reconciliation that checks the same definitions and filter context in Pandas, SQLite and Power BI. In particular, verify whether the transaction count is a row count or a distinct count of `transaction_id`, and calculate AOV using the agreed denominator.

## 10. Project files and folder structure

```text
Amazon_Data_Analysis/
├── .venv/                         # Local virtual environment; do not commit
├── data/
│   ├── raw/                        # Annual source CSV files
│   └── cleaned/                    # Annual cleaned CSVs and master CSV
├── database/
│   └── amazon_database.db          # SQLite database
├── notebook/
│   ├── amazon_analysis.ipynb
│   ├── amazon_data_cleaning.ipynb
│   ├── amazon_2017_analysis.ipynb
│   ├── exploratory_data_analysis.ipynb
│   └── 03_sql_database_integration.ipynb
├── Amazon_Dashboard.pbix
├── README.md
├── PROJECT_DOCUMENTATION.md
├── requirements.txt
└── .gitignore
```

## 11. Environment setup and execution

The notebooks were developed in a Python 3.13 environment. The imports observed in the notebooks include Pandas, NumPy, Matplotlib, and Python standard-library modules such as `pathlib`, `os`, and `sqlite3`.

Install the third-party dependencies:

```bash
python -m pip install -r requirements.txt
```

Run notebooks from the `notebook/` directory or preserve the relative paths used in the notebooks. Expected data files must be present in `data/raw/` and the cleaning notebook generates files under `data/cleaned/`.

Recommended execution sequence:

1. `amazon_analysis.ipynb`
2. `amazon_data_cleaning.ipynb`
3. `amazon_2017_analysis.ipynb` (year-specific work; run when required by the cleaning workflow)
4. `exploratory_data_analysis.ipynb`
5. `03_sql_database_integration.ipynb`
6. Open `Amazon_Dashboard.pbix` in Power BI Desktop and refresh if required.

## 12. Limitations and considerations

- The source dataset's provenance, representativeness and permission to redistribute must be confirmed before public release.
- Some delivery-day values are text categories such as `Same Day`, `Express` or ranges rather than numeric values. Any numeric conversion or average requires an explicitly documented rule.
- Missing values may have different meanings by field; do not treat every null as zero.
- Power BI totals may change under slicers, page filters, report filters and visual interactions. Reconciliation must use the same filter context.
- The dashboard's visual formatting and slicer/interactivity review remain planned work at the time this document was drafted.

## 13. Remaining work before final submission

1. Complete professional styling consistently across all four Power BI pages.
2. Add and test slicers and cross-filter interactions.
3. Reconcile core measures across Pandas, SQLite and Power BI, documenting metric definitions.
4. Add final dashboard screenshots and, if authorised, the Power BI Service report link.
5. Confirm repository contents and data-sharing permissions; exclude `.venv/`, secrets and restricted datasets.
6. Prepare a concise mentor/portfolio presentation describing the business questions, methods, results and limitations.

## 14. Conclusion

This project demonstrates a complete analytics workflow across Python-based data profiling and preparation, exploratory analysis, SQLite integration and Power BI reporting. The final portfolio submission should pair the reproducible notebooks and SQL work with clear documentation, validated metrics, and a polished interactive dashboard.
