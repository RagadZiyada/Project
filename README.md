# DSAI3202 – Project Phase 1: Data Pipeline, ETL, and Feature Foundations

## Project Overview

This project builds the data foundation for a personalized fashion recommendation system using the [H&M Personalized Fashion Recommendations dataset](https://www.kaggle.com/competitions/h-m-personalized-fashion-recommendations) from Kaggle. The dataset contains real purchase transactions, customer demographics, and article metadata from H&M. Think of it as teaching a computer to have better fashion taste than your average university student.

The goal of Phase 1 is to design and implement a reproducible data pipeline covering ingestion, ETL, data cataloging, exploratory analysis, and feature engineering — producing a curated dataset ready for recommendation model training in future phases. No models were harmed in the making of this pipeline.

**ML Problem:** Given a customer's purchase history and article metadata, predict which articles a customer is most likely to purchase next. Basically, we built the thing that makes you buy things you didn't know you needed.

---

## Repository Structure

```
├── databricks/
│   ├── 01_load_validate_data.ipynb    # Data ingestion and validation
│   ├── 02_clean_transactions.ipynb    # ETL: cleaning and transformation
│   ├── 03_feature_engineering.ipynb   # Feature extraction and selection
│   └── 04_eda.ipynb                   # Exploratory data analysis
├── catalog/
│   ├── data_catalog.md                # Dataset registry and storage zones
│   ├── schema_dictionary.md           # Field definitions and data types
│   ├── lineage.md                     # Data lineage and transformation mapping
│   ├── data_quality_report.md         # Quality checks and observations
│   └── assumptions.md                 # Pipeline assumptions and design decisions
└── README.md
```

---

## Azure Architecture

The pipeline runs entirely on Azure using the following services. We would have used a local machine but 31 million rows had other plans.

| Service | Purpose |
|---------|---------|
| Azure Data Lake Storage Gen2 | Raw, processed, and curated data storage |
| Azure Data Factory | Ingestion pipeline — CSV to Parquet conversion |
| Azure Databricks | ETL, feature engineering, and EDA notebooks |

### Storage Zones

| Container | Zone | Contents |
|-----------|------|---------|
| `raw-p1` | Raw | Original CSV files — never modified |
| `processed-p1` | Landing + Clean | Parquet landing files + cleaned datasets |
| `curated-p1` | Curated | Feature tables ready for modeling |

---

## II.1 Data Ingestion

**Source:** Kaggle — H&M Personalized Fashion Recommendations
**Ingestion mode:** Batch
##Batch ingestion was chosen because the dataset is historical and static, making real-time streaming unnecessary.
**Format:** CSV → Parquet (via Azure Data Factory pipeline `pl_hm_raw_to_processed`)
**Refresh strategy:** One-time historical load (dataset covers 2018-09-20 to 2020-09-22)

### Raw Datasets

| Dataset | File | Size | Rows |
|---------|------|------|------|
| Articles | articles.csv | 34 MB | 105,542 |
| Customers | customers.csv | 197 MB | 1,371,980 |
| Transactions | transactions_train.csv | 3.25 GB | 31,788,324 |

Raw CSV files are preserved in the `raw-p1` container and never overwritten. The ADF pipeline converts them to Parquet in `processed-p1/[dataset]_landing/` before any transformation is applied. We preserve the raw data the same way you preserve your notes before an exam — untouched and just in case.

**Notebook:** `databricks/01_load_validate_data.ipynb`
This notebook loads from the landing zone and runs initial validation: row counts, schema inspection, null reports per column, duplicate checks, required column checks, and invalid value checks (null dates, null prices, price ≤ 0). It's basically the data equivalent of checking your groceries before leaving the store.

---

## II.2 ETL Process

**Notebook:** `databricks/02_clean_transactions.ipynb`

The ETL pipeline reads from the landing zone, applies all transformations, and writes clean Parquet files back to `processed-p1/[dataset]_clean/`. We removed nearly 3 million bad rows — turns out not all data is created equal.

All transformations are deterministic and reproducible, as the pipeline reads from immutable landing data and produces consistent outputs on each run.

### Transformations Applied

**Articles**
- Deduplicated on `article_id`
- All columns cast to `StringType`
- 13 categorical fields filled with `"Unknown"` where null (only `detail_desc` had 416 nulls — apparently some items are just mysterious like that)

**Customers**
- Deduplicated on `customer_id`
- `age` cast to `IntegerType` — values outside range 12–100 set to null (15,861 affected — we don't know who these people are but they're probably lying about their age)
- `FN` and `Active` flags: null values filled with `0` (assumption: no record = not subscribed/active)
- `fashion_news_frequency` lowercased, trimmed, and normalized (`"regularly."` → `"regularly"`, `"none."` → `"none"`) — because consistency matters more than punctuation

**Transactions**
- Type casting: `price` → `DoubleType`, `t_dat` → `DateType`, `sales_channel_id` → `IntegerType`
- Deduplicated on `[t_dat, customer_id, article_id, price, sales_channel_id]`
- Invalid rows removed: null IDs, null dates, null prices, price ≤ 0
- Temporal features derived: `year`, `month`, `week`, `day_of_week`

### Cleaning Results

| Dataset | Before | After | Removed |
|---------|--------|-------|---------|
| Transactions | 31,788,324 | 28,813,419 | 2,974,905 |
| Articles | 105,542 | 105,542 | 0 |
| Customers | 1,371,980 | 1,371,980 | 0 |

All transformations are reproducible — the pipeline reads from immutable landing files and overwrites only the clean zone. No raw data was hurt in this process.

---

## II.3 Cataloging and Governance

Full schema definitions, data types, lineage, and assumptions are documented in the `catalog/` folder. Because undocumented data is just data with trust issues.

- **`data_catalog.md`** — storage zones, dataset registry, row counts
- **`schema_dictionary.md`** — field-level definitions with data types and examples for all datasets including curated feature tables
- **`lineage.md`** — end-to-end data flow: `CSV → raw-p1 → landing → clean → curated`, with per-dataset transformation mapping
- **`data_quality_report.md`** — quality checks applied, null counts, duplicate counts, and data readiness conclusion
- **`assumptions.md`** — documents all design decisions including batch mode choice, null fill strategy, outlier rules, and preference computation logic

### Data Lineage Summary

```
Kaggle CSV
    ↓ ADF Pipeline (pl_hm_raw_to_processed)
raw-p1  (articles.csv, customers.csv, transactions_train.csv)
    ↓ ADF Pipeline
processed-p1 / [dataset]_landing  (Parquet)
    ↓ 02_clean_transactions.ipynb
processed-p1 / [dataset]_clean  (Parquet)
    ↓ 03_feature_engineering.ipynb
curated-p1 / customer_features
curated-p1 / article_features
curated-p1 / customer_article_interactions
curated-p1 / recommendation_dataset
```

---

## II.4 Exploratory Data Analysis

**Notebook:** `databricks/04_eda.ipynb`

EDA was conducted on the curated feature tables to assess data readiness. We looked at the data from every angle — like trying on clothes before buying them, which is kind of the whole point of this project.

### Key Findings

| Analysis | Finding |
|----------|---------|
| Transactions over time | Peak activity in late 2019 — steady coverage across 2018–2020 |
| Sales channel split | Channel 2 (online) dominates transactions |
| Top department | Jersey Basic, Ladieswear categories lead in volume |
| Customer purchase count | Highly skewed — median is low, max is very high (power-law distribution) |
| Price distribution | Normalized values, right-skewed with a long tail |
| Age distribution | Concentrated between 20–50, ~1.1% missing |

### Correlation Analysis (Customer Features)

| Feature Pair | Correlation |
|--------------|------------|
| purchase_count vs total_spend | High positive |
| purchase_count vs avg_basket_size | Moderate positive |
| total_spend vs avg_price | Moderate positive |
| recency_days vs total_spend | Negative (recent customers spend more) |

### Data Risks

1. Customer spend and purchase activity are skewed by a small number of highly active customers — the data has its own influencers
2. A few articles dominate sales — popularity-based features may be imbalanced
3. ~1.1% of customers have missing age — kept as null rather than imputed to avoid introducing bias

### Data Readiness Conclusion

The dataset is clean, well-structured, and ready for recommendation model training. All critical fields have no nulls. Skewness in customer activity is expected and will be handled at the modeling stage. In short: the data is ready, and so are we.

---

## II.5 Feature Extraction and Selection

**Notebook:** `databricks/03_feature_engineering.ipynb`

Features are engineered from the cleaned datasets and written to `curated-p1`. The feature design is motivated by the recommendation hypothesis: a customer is more likely to purchase an article that matches their historical preferences, price sensitivity, and recency of engagement. We asked ourselves "what would make someone buy a shirt?" and turned the answer into 31 features.

### Customer Features (14 features)

| Feature | Justification |
|---------|--------------|
| `customer_purchase_count` | Overall engagement level |
| `customer_total_spend` | Spending capacity |
| `customer_avg_price` | Price sensitivity baseline |
| `customer_unique_articles` | Breadth of product interest |
| `customer_unique_product_types` | Diversity of preferences |
| `customer_unique_departments` | Cross-department shopping behavior |
| `customer_recency_days` | Days since last purchase — recency signal for RFM |
| `customer_purchase_frequency` | Purchases per day — frequency signal for RFM |
| `customer_last_purchase_date` | Most recent engagement date |
| `customer_first_purchase_date` | Tenure as a customer |
| `customer_preferred_department` | Most purchased department — preference signal |
| `customer_preferred_index_group` | Most purchased index group |
| `customer_avg_basket_size` | Average items per shopping session |

### Article Features (13 features)

| Feature | Justification |
|---------|--------------|
| `article_total_sales` | Absolute popularity |
| `article_unique_customers` | Reach across customer base |
| `article_avg_price` | Price point of the article |
| `article_last_sold_date` | Recency of item activity |
| `article_sales_last_7d` | Short-term trending signal |
| `article_sales_last_30d` | Medium-term trending signal |
| `article_popularity_rank` | Global rank by total sales |
| `article_product_type_name` | Product type metadata |
| `article_department_name` | Department metadata |
| `article_index_group_name` | Index group metadata |
| `article_colour_group_name` | Colour metadata |
| `article_garment_group_name` | Garment type metadata |

### Interaction / Cross Features (4 features)

| Feature | Justification |
|---------|--------------|
| `same_preferred_department` | 1 if article matches customer's preferred department — preference alignment signal |
| `same_preferred_index_group` | 1 if article matches customer's preferred index group |
| `price_vs_customer_avg` | Price difference from customer's average — affordability signal |
| `is_online_channel` | 1 if purchase was online — channel behavior signal |

### Feature Selection

All engineered features are retained for Phase 1. No features were dropped because all have direct relevance to the recommendation hypothesis and no redundant features were identified at this stage. Feature importance ranking and dimensionality reduction will be applied in Phase 2 during model training — we'll Marie Kondo the features later.

Feature selection was limited at this stage to retain all potentially informative variables, as model-based selection techniques will be applied in Phase 2.

### Output Datasets

| Dataset | Rows | Description |
|---------|------|-------------|
| `customer_features` | 1,362,281 | One row per customer with all customer-level features |
| `article_features` | 104,547 | One row per article with all article-level features |
| `customer_article_interactions` | 28,813,419 | Full interaction table joining all three datasets |
| `recommendation_dataset` | 28,813,419 | Final modeling dataset with all features joined |

---

## How to Run

Run the notebooks in this exact order. Skipping steps will cause errors and emotional distress.

```
1. 01_load_validate_data.ipynb   — validate raw data
2. 02_clean_transactions.ipynb   — clean and transform
3. 03_feature_engineering.ipynb  — build feature tables
4. 04_eda.ipynb                  — exploratory analysis
```

All notebooks are also configured as a Databricks job (`Job ID: 501428519265434`) with sequential task dependencies. Total pipeline runtime: ~12 minutes — enough time to make a coffee and pretend you understood everything from the start.
