# DSAI3202 — Project Phase 1 & 2: Fashion Recommendations (Pain)
## H&M Personalized Fashion Recommendations
#### Members: Ragad Ziyada 60301042 & Rawand Elraba 60304948
#### Combined ID: 60304248 (math is hard)

---

### Introduction

H&M has 31 million purchase records and said:  
*"Hey, can you predict what customers will buy next?"*

Us:

<img width="455" alt="confused" src="https://github.com/user-attachments/assets/8a0b99c6-6b53-486f-8099-ca0b9d76a253" />

This project builds a recommendation system that tells customers what to buy. Think of it as teaching a computer to have better fashion taste than your average university student.

**Dataset:** [H&M Personalized Fashion Recommendations](https://www.kaggle.com/competitions/h-and-m-personalized-fashion-recommendations) from Kaggle

**ML Problem:** Given purchase history, predict what a customer will buy next.  
**Translation:** We built the thing that makes you buy things you didn't know you needed.

---

### Project Structure

```
├── notebooks/
│   ├── notebooks/
│       ├── phase1
│           ├── 01_load_validate_data.ipynb    # Load & validate (Phase 1)
│           ├── 02_clean_transactions.ipynb    # ETL & cleaning (Phase 1)
│           ├── 03_feature_engineering.ipynb   # 31 features (Phase 1)
│           ├── 04_eda.ipynb                   # Exploratory analysis (Phase 1)
│       ├── phase2
│           ├── 05_model_baseline.ipynb        # Popularity baseline (Phase 2)
│           ├── 06_model_personalized.ipynb    # Personalized model (Phase 2)
│           ├── 07_model_registration.ipynb    # MLflow registration (Phase 2)
│           ├── 08_deployment.ipynb            # Batch inference (Phase 2)
│           └── 09_deployment_validation.ipynb # Validation checks (Phase 2)
├── catalog/
│   ├── data_catalog.md                # Dataset registry
│   ├── schema_dictionary.md           # Field definitions
│   ├── lineage.md                     # Data flow mapping
│   ├── data_quality_report.md         # Quality checks
│   └── assumptions.md                 # Design decisions
├── azure-pipelines.yml                # CI/CD pipeline
└── README.md
```

---

### README Organization

1. [Phase 1: Data Pipeline](#phase-1-data-pipeline)
2. [Phase 2: ML Pipeline](#phase-2-ml-pipeline)
3. [Results](#results)
4. [Challenges](#challenges)
5. [How to Run](#how-to-run)

---

# Phase 1: Data Pipeline

*Bronze → Silver → Gold, but make it fashion*

---

### The Data

| Dataset | Rows | Size | Vibe |
|---------|------|------|------|
| Transactions | 31,788,324 | 3.25 GB | Pain |
| Customers | 1,371,980 | 197 MB | People |
| Articles | 105,542 | 34 MB | Clothes |

31 million rows. Our laptops said "absolutely not" so we used Azure.

---

### Architecture

| Service | Purpose |
|---------|---------|
| Azure Data Lake Gen2 | Store the chaos |
| Azure Data Factory | Move the chaos |
| Azure Databricks | Clean the chaos |

**Storage Zones:**
| Container | What's in it |
|-----------|--------------|
| `raw-p1` | Original CSVs (untouched, like our mental health) |
| `processed-p1` | Cleaned Parquet files |
| `curated-p1` | Features + models + predictions |

---

### Data Lineage

```
Kaggle CSV
    ↓ ADF Pipeline
raw-p1 (CSV files)
    ↓ ADF Pipeline  
processed-p1/landing (Parquet)
    ↓ Notebook 02
processed-p1/clean (Parquet)
    ↓ Notebook 03
curated-p1/features (Parquet)
    ↓ Notebook 06-08
curated-p1/predictions (Parquet)
```

---

### Notebook 01: Load & Validate

**File:** `01_load_validate_data.ipynb`

Loads data from landing zone and validates everything:
- Row counts
- Schema inspection  
- Null reports per column
- Duplicate checks
- Invalid values (null dates, null prices, price ≤ 0)

It's like checking your groceries before leaving — except the store has 31 million items.

---

### Notebook 02: ETL & Cleaning

**File:** `02_clean_transactions.ipynb`

We cleaned the data. 3 million rows were duplicates.

| Dataset | Before | After | Removed |
|---------|--------|-------|---------|
| Transactions | 31,788,324 | 28,813,419 | 2,974,905 |
| Customers | 1,371,980 | 1,371,980 | 0 |
| Articles | 105,542 | 105,542 | 0 |

Those 3 million rows will not be missed. They knew what they did.

**Transformations:**
- Removed duplicates (there were... so many)
- Fixed ages outside 12-100 (15,861 people lying about their age)
- Normalized text (`"regularly."` → `"regularly"`)
- Removed null/zero prices (free clothes? in THIS economy?)
- Added temporal features: year, month, week, day_of_week

![chaos](https://github.com/user-attachments/assets/1116d5ef-f8e3-476a-9c38-ef1b83bb174d)

*us vs 31 million rows*

---

### Notebook 03: Feature Engineering

**File:** `03_feature_engineering.ipynb`

We asked: *"What makes someone buy a shirt?"*  
Then turned the answer into 31 features.

**Customer Features (14):**
| Feature | Why |
|---------|-----|
| `purchase_count` | Engagement level |
| `total_spend` | Spending capacity |
| `avg_price` | Price sensitivity |
| `unique_articles` | Product interest breadth |
| `unique_departments` | Cross-department shopping |
| `recency_days` | Days since last purchase |
| `preferred_department` | Most purchased department |
| `avg_basket_size` | Items per session |
| + 6 more | Various signals |

**Article Features (13):**
| Feature | Why |
|---------|-----|
| `total_sales` | Popularity |
| `unique_customers` | Reach |
| `popularity_rank` | Global rank |
| `sales_last_7d` | Short-term trend |
| `sales_last_30d` | Medium-term trend |
| `department_name` | Category |
| + 7 more | Metadata |

**Interaction Features (4):**
| Feature | Why |
|---------|-----|
| `same_preferred_department` | Preference alignment |
| `price_vs_customer_avg` | Affordability signal |
| `is_online_channel` | Channel behavior |
| `same_preferred_index_group` | Category alignment |

**Output Datasets:**
| Dataset | Rows |
|---------|------|
| `customer_features` | 1,362,281 |
| `article_features` | 104,547 |
| `customer_article_interactions` | 28,813,419 |
| `recommendation_dataset` | 28,813,419 |

---

### Notebook 04: Exploratory Data Analysis

**File:** `04_eda.ipynb`

We looked at the data from every angle — like trying on clothes before buying them.

**Key Findings:**
| Analysis | Finding |
|----------|---------|
| Date range | 2018-09-20 to 2020-09-22 |
| Avg transactions/customer | 21.2 |
| Avg articles/customer | 16.8 |
| Single-purchase customers | 157,802 (11.6%) |
| Top department | Jersey Basic (2.6M transactions) |
| Sales channel | Online dominates (people hate stores) |

**Correlation Analysis:**
| Feature Pair | Correlation |
|--------------|-------------|
| purchase_count vs total_spend | High positive |
| purchase_count vs avg_basket_size | Moderate positive |
| recency_days vs total_spend | Negative (recent = more spend) |

**Data Risks Identified:**
1. Heavy spenders skew the data — the data has its own influencers
2. Few articles dominate sales — popularity features may be imbalanced
3. ~1.1% missing ages — kept as null (honest nulls > confident lies)

![understanding](https://github.com/user-attachments/assets/4fb63b4e-2ab0-443f-b074-998429a72aa5)

*"wait… WE actually understand the data now???"*

---

### Data Catalog & Governance

Full documentation in `catalog/` folder:
- `data_catalog.md` — Storage zones, dataset registry
- `schema_dictionary.md` — Field definitions, data types
- `lineage.md` — End-to-end data flow
- `data_quality_report.md` — Quality checks, null counts
- `assumptions.md` — Design decisions

Because undocumented data is just data with trust issues.

---

# Phase 2: ML Pipeline

*"We thought Phase 1 was hard. We were so young. So naive."*

---

### The ALS Incident (A Tragedy)

**The Plan:** Use ALS (Alternating Least Squares), fancy collaborative filtering. Very sophisticated. Very recommended by tutorials.

**The Reality:**
```
MEMORY ERROR: Your cluster has died.
```

We tried:
- Sampling to 10% → Crashed
- Sampling to 5% → Crashed  
- Sampling to 1% → Ran, results were garbage
- Crying → Did not help

**The Pivot:** Fine. We'll build something that actually works.

![fine](https://github.com/user-attachments/assets/e39f4a53-8d46-42d7-afcb-a8c08a83de37)

---

### Notebook 05: Baseline Model

**File:** `05_model_baseline.ipynb`

Strategy: Recommend the 12 most popular items to everyone.  
It's basically peer pressure as a service.

**Experimental Setup:**
| Parameter | Value |
|-----------|-------|
| Split Strategy | Temporal (not random — no cheating) |
| Split Date | 2020-09-15 |
| Train Transactions | 28,571,904 |
| Test Transactions | 241,515 |
| Test Customers | 75,481 |
| K | 12 recommendations |
| Random Seed | 42 (because we're not animals) |

**Results:**
| Metric | Value |
|--------|-------|
| MAP@12 | 0.00299 |
| Recall@12 | 0.00802 |

Small numbers because recommending the same 12 items to 75,481 different people is about as personalized as "Dear Valued Customer."

---

### Notebook 06: Personalized Model

**File:** `06_model_personalized.ipynb`

Strategy: Recommend top 12 popular items **from customer's preferred department**.

You like swimwear? Here's popular swimwear.  
You like blouses? Here's popular blouses.  
Revolutionary? No. Does it work? Yes.

**Results:**
| Model | MAP@12 | Recall@12 | Improvement |
|-------|--------|-----------|-------------|
| Baseline | 0.00299 | 0.00802 | — |
| **Personalized** | **0.00806** | **0.02097** | **+170%** |

**+170% improvement.** Not bad for a model that runs in 30 seconds instead of crashing.

**Error Analysis (Why It Fails Sometimes):**
| Failure Type | Example |
|--------------|---------|
| Cross-department purchase | Customer prefers "Swimwear" but bought "Jacket" |
| Cold start | New article not in training data |
| Long-tail preferences | Customer only buys limited edition |
| Seasonal shift | Summer items recommended in September |

These failures are expected — no system is perfect, especially without collaborative filtering.

**Why This Over ALS:**
| Reason | Explanation |
|--------|-------------|
| Shared cluster | RDD operations blocked |
| Memory | Even 10% sampling crashed |
| Deadline | Approaching like a freight train |
| Interpretability | "You like X, here's popular X" makes sense |

---

### Notebook 07: Model Registration

**File:** `07_model_registration.ipynb`

Model registered in MLflow with full metadata tracking.

| Property | Value |
|----------|-------|
| Model Name | `hm_personalized_recommender_60304248` |
| Experiment | `/Shared/hm_recommendation_60304248` |
| Artifact Path | `curated-p1/models/personalized_recommender_v1/` |
| Model Type | Lookup Table (Department → Top 12 Articles) |
| Departments | 250 |
| Total Recommendations | 2,862 unique articles |

**Tracked Metadata:**
- Training data lineage
- Feature list
- Performance metrics
- Known limitations
- Team ID & timestamp

Because untracked models are like untagged photos — you'll never find them when you need them.

---

### Notebook 08: Batch Deployment

**File:** `08_deployment.ipynb`

The model generates recommendations for ALL 1.36 million customers.

**Deployment Specs:**
| Parameter | Value |
|-----------|-------|
| Deployment Type | Batch inference |
| Customers Served | 1,362,281 |
| Execution Time | ~2 seconds |
| Output Format | Parquet |
| Output Path | `curated-p1/predictions/latest/` |

**Output Schema:**
```
customer_id: string
customer_preferred_department: string  
recommended_articles: array<string>  (12 article IDs)
prediction_date: string
```

**Inference Logic:**
```python
def get_recommendations(customer_id):
    1. Look up customer's preferred department
    2. If found → return department's top 12 articles
    3. If not found → return global top 12 (fallback)
    4. Return with metadata
```

**Sample API Response:**
```json
{
  "customer_id": "abc123...",
  "department": "Swimwear",
  "recommendations": ["0351484002", "0688537004", ...],
  "prediction_date": "2026-04-10",
  "model_version": "v1"
}
```

---

### Notebook 09: Deployment Validation

**File:** `09_deployment_validation.ipynb`

Before declaring victory, we validated everything. Trust but verify.

**Validation Checks:**
| Check | Result |
|-------|--------|
| All customers have predictions | ✅ PASS (1,362,281 / 1,362,281) |
| All predictions have 12 items | ⚠️ PARTIAL (118 have <12 — small departments) |
| No null recommendations | ✅ PASS |
| Predictions match model artifact | ✅ PASS (5/5 samples verified) |
| All article IDs valid | ✅ PASS |

**Performance Benchmarks:**
| Operation | Latency |
|-----------|---------|
| Single customer lookup | ~600ms |
| Batch (100 customers) | ~515ms |
| Full inference (1.36M) | ~2 seconds |

**Status: ✅ DEPLOYMENT VALIDATED**

It works. We can finally sleep.

---

### CI/CD Pipeline (Azure DevOps)

Because manually running notebooks is for Phase 1.

**Pipeline File:** `azure-pipelines.yml`

**Stages:**
```yaml
Stage 1: Validate
  - Setup Python 3.9
  - Install nbformat
  - Validate all notebook JSON syntax
  - List repository contents

Stage 2: Report
  - Print pipeline summary
  - Confirm validation passed
```

**Pipeline Status:**
| Pipeline | Status |
|----------|--------|
| DSAI3202-Project-Phase-II-7 | ✅ Passing |
| RagadZiyada.Project (GitHub) | ✅ Passing |

CI/CD is officially more reliable than our sleep schedules.

---

# Results

### Model Performance

| Metric | Baseline | Personalized | Improvement |
|--------|----------|--------------|-------------|
| MAP@12 | 0.00299 | 0.00806 | **+169.8%** |
| Recall@12 | 0.00802 | 0.02097 | **+161.6%** |

This improvement suggests that even simple personalization strategies can significantly enhance recommendation relevance, making them suitable for scalable production systems.

### System Stats

| Metric | Value |
|--------|-------|
| Total Customers | 1,362,281 |
| Total Transactions | 28,813,419 |
| Features Engineered | 31 |
| Departments Covered | 250 |
| Unique Articles Recommended | 2,862 |
| Batch Inference Time | ~2 seconds |
| Models That Crashed | 1 (RIP ALS) |
| Models That Worked | 1 |

### Reproducibility

| Parameter | Value |
|-----------|-------|
| Random Seed | 42 |
| Train/Test Split | Temporal (2020-09-15 cutoff) |
| Train Transactions | 28,571,904 |
| Test Transactions | 241,515 |
| Test Customers | 75,481 |

---

# Challenges

We encountered many boss fights:

- **31 million rows** — Local machine said no. Azure said "$$$"
- **3 million duplicates** — Gone now. Good riddance.
- **ALS algorithm** — Fought it for 3 hours. Lost.
- **Shared cluster limits** — RDD blocked. Memory limited. Hotel? Trivago.
- **Storage key errors** — Every. Single. New. Notebook.
- **Spark warnings** — "Unpartitioned window operation" — we acknowledged and moved on
- **The deadline** — Approaching while we debugged

---

# How to Run

Run notebooks in order. Skipping steps causes errors and emotional damage.

### Phase 1: Data Pipeline
```
01_load_validate_data.ipynb    → Validate raw data
02_clean_transactions.ipynb    → Clean & transform
03_feature_engineering.ipynb   → Build 31 features
04_eda.ipynb                   → Explore & analyze
```

### Phase 2: ML Pipeline
```
05_model_baseline.ipynb        → Establish baseline
06_model_personalized.ipynb    → Train personalized model
07_model_registration.ipynb    → Register in MLflow
08_deployment.ipynb            → Batch inference
09_deployment_validation.ipynb → Validate deployment
```

**Estimated time:** 15 minutes  
**Realistic time:** 45 minutes (including re-runs and questioning life choices)

---

# Tech Stack

| Layer | Tech |
|-------|------|
| Storage | Azure Data Lake Gen2 |
| Ingestion | Azure Data Factory |
| Compute | Azure Databricks (Spark) |
| ML Tracking | MLflow |
| CI/CD | Azure DevOps |
| Languages | Python, PySpark, SQL |
| Tears | Unlimited |

---

# Lessons Learned

1. **Start simple** — Our best model is a lookup table. It beats crashing.
2. **Infrastructure matters** — Plan around what runs, not what looks cool.
3. **Temporal splits matter** — Random splits on time-series = cheating.
4. **Document everything** — Future you will thank present you.
5. **Deadlines are features** — They force you to ship something that works.
6. **Coffee is essential** — No explanation needed.

---

# What We Would Do Differently

- Get dedicated compute (ALS might have worked)
- Try LightFM or implicit libraries (CPU-based)
- Implement A/B testing framework
- Add real-time scoring endpoint

But also: the current system works, is documented, and was delivered on time. That's not nothing.

---

# Acknowledgments

- H&M for the dataset
- Azure for the free credits (mostly)
- Stack Overflow for existing
- Coffee for everything else
- ALS for teaching us humility

---

![done](https://github.com/user-attachments/assets/fbb07823-d770-4482-b815-422611b5afb3)

**Phase 1 + Phase 2: Complete** ✅

*The model is deployed. The pipeline works. We can finally rest.*

---
