# Data Lineage

## Source System

Kaggle Dataset:
H&M Personalized Fashion Recommendations

---

## Data Flow Pipeline

```text
Source (CSV Files)
    ↓
Raw Layer (raw-p1)
    ↓
Landing Layer (processed-p1)
    ↓
Clean Layer (processed-p1)
    ↓
Curated Layer (curated-p1)
```

---

## Transformation Mapping

### Articles

```text
articles.csv
→ articles_landing
→ articles_clean
→ article_features
```

### Customers

```text
customers.csv
→ customers_landing
→ customers_clean
→ customer_features
```

### Transactions

```text
transactions_train.csv
→ transactions_landing
→ transactions_clean
→ customer_article_interactions
→ recommendation_dataset
```

---

## Output Layer

Final datasets:

* customer_article_interactions
* customer_features
* article_features
* recommendation_dataset

These datasets are used for analytics and future recommendation models.
