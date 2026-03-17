# Assumptions 

| Category          | Assumption                                                               |
| ----------------- | ------------------------------------------------------------------------ |
| Data Completeness | Records with missing critical IDs (customer_id, article_id) were removed |
| Data Types        | All dates converted to standard date format                              |
| Pricing           | Prices converted to numeric values                                       |
| Recency           | Latest transaction date used as reference                                |
| Customer Behavior | Preference based on most frequent purchases                              |
| Processing Mode   | Batch processing used (no streaming)                                     |
| Dataset Nature    | Historical dataset, not real-time                                        |
