# Schema Dictionary

## Dataset: transactions_clean

| Field Name       | Data Type | Description                           | Example    |
| ---------------- | --------- | ------------------------------------- | ---------- |
| customer_id      | string    | Unique identifier for each customer   | 000123abc  |
| article_id       | int       | Unique identifier for each product    | 706016001  |
| t_dat            | date      | Transaction date                      | 2020-09-22 |
| price            | float     | Price of purchased item               | 29.99      |
| sales_channel_id | int       | Sales channel (1 = store, 2 = online) | 2          |

---

## Dataset: customer_features

| Field Name                    | Data Type | Description                             |
| ----------------------------- | --------- | --------------------------------------- |
| customer_id                   | string    | Unique customer identifier              |
| customer_purchase_count       | int       | Total number of purchases               |
| customer_total_spend          | float     | Total amount spent                      |
| customer_avg_price            | float     | Average purchase price                  |
| customer_unique_articles      | int       | Number of distinct products purchased   |
| customer_unique_departments   | int       | Number of departments interacted with   |
| customer_recency_days         | int       | Days since last purchase                |
| customer_avg_basket_size      | float     | Average number of items per transaction |
| customer_preferred_department | string    | Most frequently purchased department    |

---

## Dataset: article_features

| Field Name               | Data Type | Description                                |
| ------------------------ | --------- | ------------------------------------------ |
| article_id               | int       | Unique product identifier                  |
| article_total_sales      | int       | Total number of times the article was sold |
| article_unique_customers | int       | Number of unique customers                 |
| article_avg_price        | float     | Average selling price                      |
| article_sales_last_7d    | int       | Sales in last 7 days                       |
| article_sales_last_30d   | int       | Sales in last 30 days                      |
| article_popularity_rank  | int       | Ranking based on total sales               |
| article_department_name  | string    | Product department                         |

---

## Dataset: recommendation_dataset

| Field Name                 | Data Type | Description                              |
| -------------------------- | --------- | ---------------------------------------- |
| customer_id                | string    | Customer identifier                      |
| article_id                 | int       | Product identifier                       |
| price                      | float     | Transaction price                        |
| sales_channel_id           | int       | Sales channel                            |
| customer_avg_price         | float     | Customer average spending                |
| article_total_sales        | int       | Product popularity metric                |
| same_preferred_department  | int       | 1 if matches customer preference, else 0 |
| same_preferred_index_group | int       | Preference match indicator               |
| price_vs_customer_avg      | float     | Price difference vs customer average     |
| is_online_channel          | int       | 1 if online purchase                     |
