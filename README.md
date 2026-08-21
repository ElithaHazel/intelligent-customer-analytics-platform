# Intelligent Customer Analytics Platform

### End-to-End Customer Lifetime Value Prediction using Databricks Lakehouse

PySpark • Delta Lake • MLflow • XGBoost • FAISS • RAG • Generative AI

---

# Project Overview

The Intelligent Customer Analytics Platform is an end-to-end Lakehouse analytics solution built on Databricks to identify high-value customers, predict Customer Lifetime Value (CLV), and provide natural language business insights from customer reviews.

The project integrates modern Data Engineering, Machine Learning, and Retrieval-Augmented Generation (RAG) into a single scalable analytics platform.

---

# Business Problem

Large e-commerce platforms generate massive volumes of customer, transaction, payment, and review data.

Although this information is valuable, it often exists across multiple relational tables without a unified analytics workflow.

As a result, business users struggle to:

- Identify high-value customers
- Predict future customer value
- Understand customer feedback
- Make proactive business decisions

---

# Solution

This project builds an end-to-end Customer Analytics Platform that:

- Implements a Medallion Lakehouse Architecture using Delta Lake

- Processes over 1.5 million records from 9 relational datasets

- Engineers RFM features for customer segmentation

- Predicts high-value customers using XGBoost

- Tracks experiments using MLflow

- Uses Retrieval-Augmented Generation (RAG) to answer business questions from customer reviews

---

# Dataset

| Attribute | Value |
|------------|-------|
| Dataset | Brazilian E-Commerce Public Dataset by Olist |
| Source | Kaggle |
| Orders | 100,000+ |
| Tables | 9 |
| Total Records | 1,557,851 |
| Time Period | 2016 – 2018 |

---

# System Architecture

```
Raw CSV Files
        │
        ▼
Bronze Layer
(Raw Delta Tables)
        │
        ▼
Silver Layer
(Cleaned & Enriched Data)
        │
        ▼
Exploratory Data Analysis
        │
        ▼
Gold Layer
(RFM Feature Engineering)
        │
        ▼
Customer Segmentation
        │
        ▼
XGBoost CLV Prediction
        │
        ▼
MLflow Experiment Tracking
        │
        ▼
RAG Knowledge Assistant
        │
        ▼
Business Insights

```

---

# Project Structure

| Notebook | Description |
|-----------|-------------|
| 00_Project_Setup | Project documentation and architecture |
| 01_Data_Ingestion | Raw data ingestion into Bronze Layer |
| 02_Bronze_Layer | Data validation and profiling |
| 03_Silver_Layer | Data cleaning and transformation |
| 04_EDA | Exploratory Data Analysis |
| 05_Gold_Layer_RFM | Feature engineering and customer segmentation |
| 06_CLV_Model | Customer Lifetime Value prediction using XGBoost |
| 07_RAG_Knowledge_Assistant | Semantic search over customer reviews |
| 08_Model_Deployment | Batch inference and CLV tier deployment |
| 09_Hyperparameter_Tuning | Hyperopt-based tuning of the CLV model |
| 10_Temporal_CLV_Target | Leakage-free target using a temporal train/test split |

---

# Technology Stack

## Platform

- Databricks
- Delta Lake

## Data Engineering

- PySpark
- Spark SQL

## Machine Learning

- Scikit-Learn
- XGBoost
- MLflow

## Generative AI

- Sentence Transformers
- FAISS
- HuggingFace Transformers

## Programming

- Python
- SQL

---

# Key Results

## Data Engineering

- Processed **1,557,851** records
- Implemented Bronze → Silver → Gold Lakehouse
- Built scalable Delta Tables

---

## Customer Segmentation

| Segment | Customers |
|----------|----------:|
| High Value | 33,703 |
| Medium Value | 37,051 |
| Low Value | 22,502 |

Total Customers

93,256

---

## Machine Learning

- XGBoost Customer Lifetime Value Classification
- MLflow Experiment Tracking
- Feature Importance Analysis

Key Finding

Recency was initially identified as the strongest predictor of Customer Lifetime Value. Further investigation (below) found this result to be inflated by target leakage — see the Model Development section for the corrected, honestly-measured finding.

---

## Model Development — Leakage Investigation & Fix

The first modeling pass (V1/V2) predicted `is_high_value`, a label derived directly from percentile-ranked recency/frequency/monetary scores — the same features used to predict it. This produced near-perfect but meaningless metrics (V3, tuned: **ROC AUC 1.0000, accuracy 99.86%**), and a feature importance profile where `frequency` scored exactly 0% across every version — a clear signature of leakage rather than genuine signal.

**The fix (V4):** redefined the target as `is_repeat_customer_90d` — whether a customer placed an order in the 90/180 days *after* a fixed cutoff date, using only recency/frequency/monetary computed *before* that cutoff. Features and label are now strictly time-separated.

| Model | Target | ROC AUC | Notes |
|-------|--------|--------:|-------|
| V1/V2 | is_high_value (same-snapshot RFM threshold) | ~1.00 | Leaked — target derived from features |
| V3 (tuned) | is_high_value | 1.0000 | Hyperopt-tuned, still leaked |
| V4 (90-day window) | is_repeat_customer_90d | 0.5293 | Honest, weak signal |
| V4 (180-day window) | is_repeat_customer_90d | 0.5461 | Honest, marginal improvement |

**Conclusion:** once measured honestly, recency/frequency/monetary alone carry weak predictive power for *future* repeat purchase (AP lift of ~2.3–2.9x over base rate, ROC AUC only marginally above random). This is a legitimate finding, not a modeling failure — it indicates that a stronger CLV model would need richer features (review sentiment, product category, delivery experience, seasonality) rather than further tuning of the existing three features. Notably, `frequency` became the top feature in V4 (52.8% importance) once the leakage was removed — a meaningful behavioral signal that was completely invisible in the leaked V1–V3 models.

---

## Retrieval-Augmented Generation (RAG)

Implemented an end-to-end semantic search pipeline that:

- Generates multilingual sentence embeddings
- Indexes customer reviews using FAISS
- Retrieves relevant reviews through semantic similarity
- Produces structured business insights from natural language questions

Example Questions

- Why are customers unhappy with delivery?
- What do customers love about the products?
- What are the main complaints about product quality?

---

# Pipeline Orchestration (Databricks Workflows)

The full pipeline is orchestrated as a scheduled Databricks Workflow, structured as two branches from a shared feature layer:

`ingest_bronze` → `bronze_layer` → `silver_layer` → `gold_layer_rfm`

&nbsp;

**After Gold Layer, the pipeline branches into:**

| Production Path | Experimentation Path |
|---|---|
| `clv_model` | `experimental_hyperparameter_tuning` |
| ↓ | ↓ |
| `model_deployment` | `experimental_temporal_target` |

- **Production path** deploys the current validated model (V2).
- **Experimentation path** produces candidate models (V3 tuned, V4 leakage-free) without affecting what's deployed — a standard MLOps pattern for separating validated production models from active experimentation.
- Scheduled to run **daily at 6:00 AM (America/New_York)**.
- **Failure-only email notifications** configured — no noise on successful runs.
- Verified with two full manual runs, all tasks across both branches succeeding, including the MLflow hand-off from the tuning task to the temporal-target task.

Note: Free Edition supports Workflows with limits (5 concurrent job tasks, 1 active pipeline per type) — this project's DAG runs within those limits.

---

# Business Value

The platform enables organizations to:

- Identify high-value customers
- Improve retention strategies
- Analyze customer sentiment
- Support marketing decisions
- Provide business users with natural language access to customer insights

---

# Future Enhancements

**Completed:**
- ~~Hyperparameter Optimization~~ — Hyperopt/TPE tuning (09)
- ~~Model Deployment~~ — batch CLV tier deployment (08)
- ~~Databricks Workflows~~ — scheduled, branched DAG with production/experimentation split
- ~~Leakage-free CLV target~~ — temporal train/test split (10)

**Pending:**
- Real-time model serving (MLflow Model Serving REST endpoint, wrapping V4)
- LLM synthesis for RAG (replace structured extraction with a real LLM generation call, exposed as an endpoint)
- Drift monitoring (track recency/frequency/monetary distribution drift against training baseline)
- Richer feature engineering for CLV (review sentiment, product category, delivery experience, seasonality) — the natural next step identified by the V4 leakage investigation
- Interactive Business Dashboard

---

# Author

**Elita Hazel Gorimanikonda**

M.S. Data Science

Stony Brook University

May 2026