# Data Catalog

## Storage Zones
| Zone | Container | Description |
|------|-----------|-------------|
| Raw | raw-p1 | Original CSV files, never modified |
| Processed | processed-p1 | Landing parquet + cleaned parquet per dataset |
| Curated | curated-p1 | Feature tables ready for modeling |

## Dataset: articles (104,547 rows)
| Field | Type | Description |
|-------|------|-------------|
| article_id | string | Unique article identifier |
| product_code | string | Product group code |
| prod_name | string | Product display name |
| product_type_name | string | Type of product (e.g. Vest top, Bra) |
| product_group_name | string | High-level product group |
| graphical_appearance_name | string | Pattern type (e.g. Solid, Stripe) |
| colour_group_name | string | Colour group |
| department_name | string | Department (e.g. Jersey Basic) |
| index_group_name | string | Index group (e.g. Ladieswear) |
| garment_group_name | string | Garment category |
| detail_desc | string | Free-text product description (416 nulls → filled "Unknown") |

## Dataset: customers (1,371,980 rows)
| Field | Type | Description |
|-------|------|-------------|
| customer_id | string | Unique hashed customer identifier |
| FN | int | Fashion news subscriber flag (1=yes, 0=no, null→0) |
| Active | int | Active customer flag (1=yes, 0=no, null→0) |
| club_member_status | string | Membership status (ACTIVE / null) |
| fashion_news_frequency | string | Newsletter frequency (regularly, monthly, none, unknown) |
| age | int | Customer age — values outside 12–100 set to null (~15,861 nulls) |
| postal_code | string | Hashed postal code |

## Dataset: transactions (28,813,419 rows after dedup)
| Field | Type | Description |
|-------|------|-------------|
| t_dat | date | Transaction date |
| customer_id | string | Foreign key → customers |
| article_id | string | Foreign key → articles |
| price | double | Normalized price (not raw currency) |
| sales_channel_id | int | 1 = store, 2 = online |
| year | int | Derived from t_dat |
| month | int | Derived from t_dat |
| week | int | Week of year, derived from t_dat |
| day_of_week | int | Day of week, derived from t_dat |

## Lineage
raw-p1 (CSV) → ADF pipeline → processed-p1/[dataset]_landing (parquet) → notebook 02 → processed-p1/[dataset]_clean (parquet) → notebook 03 → curated-p1/[feature tables]
