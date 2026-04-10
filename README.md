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

### README Organization

1. [Phase 1: Data Pipeline](#phase-1-data-pipeline-bronze--silver--gold-but-make-it-fashion)
2. [Phase 2: ML Pipeline](#phase-2-ml-pipeline-the-suffering-arc)
3. [Results](#results)
4. [Challenges](#challenges-boss-fights)
5. [How to Run](#how-to-run)

---

# Phase 1: Data Pipeline (Bronze → Silver → Gold but make it fashion)

### The Data

| Dataset | Rows | Vibe |
|---------|------|------|
| Transactions | 31,788,324 | Pain |
| Customers | 1,371,980 | People |
| Articles | 105,542 | Clothes |

31 million rows. Our laptops said "absolutely not" so we used Azure.

---

### Architecture

| Service | Purpose |
|---------|---------|
| Azure Data Lake Gen2 | Store the chaos |
| Azure Data Factory | Move the chaos |
| Azure Databricks | Clean the chaos |

**Storage Zones:**
- `raw-p1` → Raw CSV files (untouched, like our mental health)
- `processed-p1` → Cleaned Parquet files
- `curated-p1` → Feature tables + models + predictions

---

### ETL Process

**Notebook:** `02_clean_transactions.ipynb`

We cleaned the data. 3 million rows were duplicates.

| Dataset | Before | After | Removed |
|---------|--------|-------|---------|
| Transactions | 31,788,324 | 28,813,419 | 2,974,905 |

Those 3 million rows will not be missed. They knew what they did.

**Transformations applied:**
- Removed duplicates (there were... so many)
- Fixed ages outside 12-100 (15,861 people were lying about their age)
- Normalized text fields (because `"regularly."` ≠ `"regularly"` apparently)
- Removed null prices (free clothes? in THIS economy?)

![chaos](https://github.com/user-attachments/assets/1116d5ef-f8e3-476a-9c38-ef1b83bb174d)

*us vs 31 million rows*

---

### Feature Engineering

**Notebook:** `03_feature_engineering.ipynb`

We asked ourselves: *"What makes someone buy a shirt?"*  
Then we turned the answer into 31 features.

**Customer Features (14):** purchase count, total spend, preferred department, etc.  
**Article Features (13):** popularity rank, price tier, sales trends, etc.  
**Interaction Features (4):** same department flag, price difference, channel, etc.

**Output:**
| Dataset | Rows |
|---------|------|
| `customer_features` | 1,362,281 |
| `article_features` | 104,547 |
| `recommendation_dataset` | 28,813,419 |

![understanding](https://github.com/user-attachments/assets/4fb63b4e-2ab0-443f-b074-998429a72aa5)

*"wait… WE actually understand the data now???"*

---

# Phase 2: ML Pipeline (The Suffering Arc)

*"We thought Phase 1 was hard. We were so young. So naive."*

---

### The ALS Incident (A Tragedy)

**The Plan:** Use ALS (Alternating Least Squares), a fancy collaborative filtering algorithm. Very sophisticated. Very recommended by every tutorial.

**The Reality:**
```
MEMORY ERROR: Your cluster has died.
```

We tried:
- Sampling to 10% → Crashed
- Sampling to 5% → Crashed  
- Sampling to 1% → Finally ran, results were garbage
- Crying → Did not help

**The Pivot:** Fine. We'll build something that actually works.

![fine](https://github.com/user-attachments/assets/e39f4a53-8d46-42d7-afcb-a8c08a83de37)

---

### Baseline Model

**Notebook:** `05_model_baseline.ipynb`

Strategy: Recommend the 12 most popular items to everyone.  
It's basically peer pressure as a service.

| Metric | Value |
|--------|-------|
| MAP@12 | 0.00299 |
| Recall@12 | 0.00802 |

These numbers are small because recommending the same 12 items to 75,481 different customers is about as personalized as a mass email that starts with "Dear Valued Customer."

---

### Personalized Model

**Notebook:** `06_model_personalized.ipynb`

Strategy: Recommend top 12 popular items **from the customer's preferred department**.

You like swimwear? Here's popular swimwear.  
You like blouses? Here's popular blouses.  
Revolutionary? No. Does it work? Yes.

| Model | MAP@12 | Recall@12 | Improvement |
|-------|--------|-----------|-------------|
| Baseline | 0.00299 | 0.00802 | — |
| **Personalized** | **0.00806** | **0.02097** | **+170%** |

**+170% improvement.** Not bad for a model that runs in 30 seconds instead of crashing.

---

### Why Not Keep Trying ALS?

| Reason | Explanation |
|--------|-------------|
| Shared cluster | RDD operations blocked |
| Memory | Even 10% sampling crashed |
| Deadline | Approaching faster than cluster could process |
| Our sanity | Already gone |

The deadline does not care about your preferred algorithm.

---

### Model Registration & Deployment

**Notebooks:** `07_model_registration.ipynb`, `08_deployment.ipynb`

| Metric | Value |
|--------|-------|
| Customers Served | 1,362,281 |
| Inference Time | ~2 seconds |
| Departments Covered | 250 |
| MLflow Experiment | `/Shared/hm_recommendation_60304248` |

Output stored at: `curated-p1/predictions/latest/`

---

### Deployment Validation

**Notebook:** `09_deployment_validation.ipynb`

| Check | Result |
|-------|--------|
| All customers have predictions | ✅ PASS |
| Predictions match model | ✅ PASS |
| No null recommendations | ✅ PASS |
| Single lookup latency | ~600ms |

**Status:** It works. We can finally sleep.

---

### CI/CD Pipeline

**Azure DevOps + GitHub**

We set up automated pipelines because manually running notebooks is for Phase 1.

| Pipeline | Status |
|----------|--------|
| DSAI3202-Project-Phase-II-7 | ✅ Passing |
| RagadZiyada.Project | ✅ Passing |

CI/CD is officially more reliable than our sleep schedules.

---

# Results

### Model Performance

| Metric | Baseline | Personalized | Change |
|--------|----------|--------------|--------|
| MAP@12 | 0.00299 | 0.00806 | +170% |
| Recall@12 | 0.00802 | 0.02097 | +162% |

### System Stats

| Metric | Value |
|--------|-------|
| Total Customers | 1,362,281 |
| Total Transactions | 28,813,419 |
| Features Engineered | 31 |
| Models That Crashed | 1 (RIP ALS) |
| Models That Worked | 1 |

---

# Challenges (Boss Fights)

- **31 million rows** — Local machine said no. Azure said "that'll be $$$"
- **3 million duplicates** — They're gone now. Good riddance.
- **ALS algorithm** — Fought it for 3 hours. Lost.
- **Shared cluster limits** — RDD operations? Blocked. Memory? Limited. Hotel? Trivago.
- **Storage key errors** — Every. Single. New. Notebook.
- **The deadline** — Approaching like a freight train while we debugged

---

# How to Run

Run notebooks in order. Skipping steps causes errors and emotional damage.

### Phase 1
```
01_load_validate_data.ipynb    → Validate
02_clean_transactions.ipynb    → Clean
03_feature_engineering.ipynb   → Features
04_eda.ipynb                   → Explore
```

### Phase 2
```
05_model_baseline.ipynb        → Baseline
06_model_personalized.ipynb    → Model
07_model_registration.ipynb    → Register
08_deployment.ipynb            → Deploy
09_deployment_validation.ipynb → Validate
```

**Estimated time:** 15 minutes  
**Realistic time:** 45 minutes (including re-runs and questioning life choices)

---

# Tech Stack

| Layer | Tech |
|-------|------|
| Storage | Azure Data Lake Gen2 |
| Ingestion | Azure Data Factory |
| Compute | Azure Databricks |
| ML Tracking | MLflow |
| CI/CD | Azure DevOps |
| Tears | Unlimited |

---

# Lessons Learned

1. **Start simple** — Our best model is a lookup table. It beats crashing.
2. **Infrastructure matters** — Plan around what actually runs, not what looks cool.
3. **Deadlines are features** — They force you to ship something that works.
4. **Document everything** — Future you will thank present you.
5. **Coffee is essential** — No explanation needed.

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
