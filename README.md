# H&M Fashion Recommendation System

**DSAI3202 Cloud Computing - Phase 2**

## Team Information
| Name | Student ID |
|------|------------|
| Rawand Elraba | 60304948 |
| Ragad Ziyada | 60301042 |
| **Combined ID** | **60304248** |

## Project Overview
An end-to-end machine learning pipeline for personalized fashion recommendations using the H&M Kaggle dataset, deployed on Azure Databricks.

## Architecture
- **Storage:** Azure Data Lake Gen2 (ragadziyada)
- **Compute:** Azure Databricks (amazon-dbx-60301042)
- **Tracking:** MLflow
- **CI/CD:** Azure DevOps

## Dataset
- **Source:** H&M Personalized Fashion Recommendations (Kaggle)
- **Transactions:** 28,813,419 purchases
- **Customers:** 1,362,281 unique
- **Articles:** 104,547 products
- **Date Range:** 2018-09-20 to 2020-09-22

## Phase 2 Notebooks

| Notebook | Description | Output |
|----------|-------------|--------|
| 05_model_baseline | Global popularity baseline | MAP@12: 0.00299 |
| 06_model_personalized | Department-based recommendations | MAP@12: 0.00806 (+170%) |
| 07_model_registration | MLflow model registry | Model registered |
| 08_deployment | Batch inference for 1.36M customers | predictions/latest/ |
| 09_deployment_validation | Production validation | All checks passed |

## Model Performance

| Model | MAP@12 | Recall@12 | Improvement |
|-------|--------|-----------|-------------|
| Popularity Baseline | 0.00299 | 0.00802 | — |
| **Personalized Popularity** | **0.00806** | **0.02097** | **+170%** |

The personalized model significantly outperforms the baseline, achieving a 170% improvement in MAP@12. This demonstrates that incorporating user-specific preferences leads to more relevant recommendations compared to a global popularity approach.

## Reproducibility
- **Random Seed:** 42
- **Train/Test Split:** Temporal (cutoff: 2020-09-15)
- **Train:** 28,571,904 transactions
- **Test:** 241,515 transactions (75,481 customers)

## Deployment

The model was deployed using Azure Databricks as a batch inference pipeline. 
Customer transaction data is processed using the same feature engineering steps as in training, ensuring consistency between offline and production predictions. 
The output recommendations are stored in the predictions/latest/ directory for downstream use.

## How to Run
1. Import notebooks to Azure Databricks
2. Configure storage key in each notebook
3. Run notebooks in order (05 → 09)

## Azure Resources
| Resource | Name |
|----------|------|
| Storage Account | ragadziyada |
| Databricks Workspace | amazon-dbx-60301042 |
| MLflow Experiment | /Shared/hm_recommendation_60304248 |

---
*DSAI3202 Cloud Computing | Team 60304248*