# Intelligent Customer Analytics Platform

### End-to-End Customer Analytics, Predictive Modeling & RAG using Databricks Lakehouse

**PySpark • Delta Lake • MLflow • XGBoost • Hyperopt • Databricks Workflows • FAISS • RAG • Generative AI**

---

## Project Overview

The **Intelligent Customer Analytics Platform** is an end-to-end Lakehouse solution built on Databricks for customer segmentation, predictive modeling, and natural-language analysis of customer feedback.

The platform integrates:

- Data Engineering
- Delta Lake Medallion Architecture
- Customer Segmentation
- Machine Learning
- MLflow Experiment Tracking
- Databricks Workflow Orchestration
- Retrieval-Augmented Generation (RAG)

The project also documents the evolution of the machine learning pipeline from an initially leaked RFM-derived target to a temporally separated future repeat-purchase prediction problem with richer behavioral features.

---

## Business Problem

Large e-commerce platforms generate customer information across orders, payments, products, deliveries, and reviews.

Without a unified analytics platform, organizations may struggle to:

- Identify high-value customer segments
- Understand customer purchasing behavior
- Predict future customer engagement
- Analyze customer satisfaction and feedback
- Operationalize machine learning pipelines
- Provide business users with accessible insights

This project brings these capabilities together within a Databricks Lakehouse architecture.

---

## Solution

The platform:

- Implements a **Bronze → Silver → Gold Medallion Architecture** using Delta Lake
- Processes **1,557,851 records** across 9 relational datasets
- Engineers **Recency, Frequency, and Monetary (RFM)** features
- Segments customers into value groups
- Trains XGBoost models for customer classification and future repeat-purchase prediction
- Uses **Hyperopt/TPE** for hyperparameter optimization
- Tracks model experiments and artifacts using **MLflow**
- Introduces temporal feature/label separation to reduce target leakage
- Extends RFM with review, delivery, and product-category features
- Uses cross-validation for more reliable model comparison
- Orchestrates the pipeline using **Databricks Workflows**
- Implements semantic retrieval over customer reviews using **Sentence Transformers + FAISS**

---

## Dataset

| Attribute | Value |
|---|---|
| Dataset | Brazilian E-Commerce Public Dataset by Olist |
| Source | Kaggle |
| Orders | 100,000+ |
| Tables | 9 |
| Total Records | 1,557,851 |
| Time Period | 2016–2018 |

The dataset contains information across customers, orders, order items, payments, products, sellers, geolocation, and customer reviews.

---

## System Architecture

```text
                         Raw Olist CSV Files
                                  │
                                  ▼
                         ┌─────────────────┐
                         │  Bronze Layer   │
                         │ Raw Delta Data  │
                         └────────┬────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │  Silver Layer   │
                         │ Cleaned Data    │
                         └────────┬────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │   Gold Layer    │
                         │  RFM Features   │
                         └────────┬────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
                    ▼                           ▼
          Customer / ML Pipeline         Review / RAG Pipeline
                    │                           │
                    ▼                           ▼
          Customer Segmentation          Sentence Embeddings
                    │                           │
                    ▼                           ▼
          XGBoost Modeling                    FAISS
                    │                           │
                    ▼                           ▼
          MLflow Experiments            Semantic Retrieval
                    │                           │
                    ▼                           ▼
          Batch Predictions             Business Q&A
```

Pipeline execution is orchestrated through **Databricks Workflows**.

---

## Project Structure

| Notebook | Description |
|---|---|
| `00_Project_Setup` | Project documentation and architecture |
| `01_Data_Ingestion` | Raw data ingestion into the Bronze Layer |
| `02_Bronze_Layer` | Data validation and profiling |
| `03_Silver_Layer` | Data cleaning and transformation |
| `04_EDA` | Exploratory Data Analysis |
| `05_Gold_Layer_RFM` | RFM feature engineering and customer segmentation |
| `06_CLV_Model` | Initial XGBoost customer-value classification models |
| `07_RAG_Knowledge_Assistant` | Semantic retrieval over customer reviews |
| `08_Model_Deployment` | Batch inference and customer-tier deployment |
| `09_Hyperparameter_Tuning` | Hyperopt/TPE tuning with MLflow tracking |
| `10_Temporal_CLV_Target` | Temporal future repeat-purchase target and leakage-reduced baseline |
| `11_V5_Review_Score` | Review-score feature experiment |
| `12_V6_Combined_Features` | Combined behavioral features with 5-fold cross-validation |

---

# Technology Stack

## Platform & Lakehouse

- Databricks
- Delta Lake
- Databricks Workflows

## Data Engineering

- PySpark
- Spark SQL
- Delta Tables
- Medallion Architecture

## Machine Learning

- Scikit-learn
- XGBoost
- Hyperopt
- MLflow

## Generative AI / NLP

- Sentence Transformers
- FAISS
- Hugging Face Transformers
- Retrieval-Augmented Generation (RAG)

## Programming

- Python
- SQL

---

# Data Engineering

The platform processes raw Olist datasets through a three-layer Lakehouse architecture.

### Bronze Layer

Stores ingested source data as Delta tables while preserving the original structure for reproducibility and downstream processing.

### Silver Layer

Performs data cleaning, transformation, validation, and enrichment.

### Gold Layer

Creates analytics-ready customer-level features, including RFM metrics used for segmentation and machine learning.

### Key Result

**1,557,851 records** were processed across the source datasets.

---

# Customer Segmentation

RFM analysis is used to represent customer purchasing behavior through:

- **Recency** — how recently a customer purchased
- **Frequency** — how frequently a customer purchased
- **Monetary** — total customer spending

The resulting customer segments were:

| Segment | Customers |
|---|---:|
| High Value | 33,703 |
| Medium Value | 37,051 |
| Low Value | 22,502 |
| **Total** | **93,256** |

Customer segmentation is retained as a descriptive analytics capability separate from the later forward-looking repeat-purchase prediction models.

---

# Machine Learning Development

## Initial Modeling

The initial XGBoost models classified customers using RFM-derived features.

MLflow was used to track:

- Model parameters
- Evaluation metrics
- Model artifacts
- Feature importance
- Hyperparameter experiments

During later model validation, the initial target design was found to contain **target leakage**.

---

## Target Leakage Investigation

The original target, `is_high_value`, was derived from percentile-ranked RFM scores.

The model then used those same underlying RFM variables as predictive features.

Conceptually:

```text
RFM Features
     │
     ├──────────────► Model Inputs
     │
     └──────────────► High-Value Target
```

Because the target was mathematically derived from the model inputs, the resulting near-perfect performance did not represent genuine future predictive capability.

The Hyperopt-tuned V3 model reached:

- **ROC AUC: 1.0000**
- **Accuracy: 99.86%**

Rather than treating this as a successful performance result, the pipeline was redesigned to create a forward-looking prediction problem.

---

# Temporal Target — V4

V4 introduced strict temporal separation between model features and the prediction target.

```text
Historical Period                     Future Period

<--------------------------|-------------------------------->
                           │
                        Cutoff
                           │
        RFM Features       │       Repeat Purchase Target
        calculated here    │       calculated here
```

Features are calculated using transactions available **before the cutoff date**.

The target represents whether an existing customer places another order during the subsequent prediction window.

This prevents future customer behavior from being directly incorporated into historical model features.

### Initial V4 Results

| Experiment | ROC AUC |
|---|---:|
| V4 – 90-day window | 0.5293 |
| V4 – 180-day window | 0.5461 |

These results were substantially lower than the leaked models but represented a more realistic forward-looking evaluation.

> **Target naming note:** Earlier notebooks retain the column name `is_repeat_customer_90d`. The 180-day experiment changes the configured prediction window but retains this legacy column name. V6 replaces it with the window-agnostic `is_repeat_customer`. Backfilling this rename to earlier notebooks remains a code-cleanup item.

---

# Hyperparameter Optimization — V3

Notebook 09 implements Hyperopt's **Tree-structured Parzen Estimator (TPE)** search.

The tuning pipeline includes:

- 40 Hyperopt trials
- 5-fold cross-validation
- ROC-AUC optimization
- XGBoost hyperparameter search
- Nested MLflow runs
- Best-configuration tracking

Parameters explored include:

- `max_depth`
- `learning_rate`
- `n_estimators`
- `subsample`
- `colsample_bytree`
- `min_child_weight`
- `gamma`

The tuning exercise demonstrates automated experiment management and MLflow integration.

However, V3 still uses the original leaked target and its performance is therefore **not presented as valid evidence of predictive performance**.

---

# Richer Feature Engineering

After establishing the temporal baseline, the next experiments tested whether customer-experience and behavioral information provides predictive signal beyond RFM.

---

## V5 — Review Score

V5 adds:

```text
avg_review_score
```

A key leakage-control requirement is that reviews must be filtered by **review creation date**, not simply order purchase date.

A customer may purchase before the cutoff but submit a review after the cutoff. Such a review would not have been known when the prediction was made.

Therefore:

```text
review_creation_date < feature_cutoff
```

is required before review information can enter the feature set.

### Results

| Metric | V4 RFM | V5 + Review |
|---|---:|---:|
| ROC AUC | 0.5461 | 0.5345 |
| Average Precision | 0.0286 | 0.0366 |

The result was mixed:

- ROC-AUC decreased slightly
- Average Precision increased
- Review score contributed to feature importance

Because V5 relied on a single train/test split with relatively few positive examples, the experiment was treated as **inconclusive rather than a confirmed improvement**.

This motivated the more rigorous V6 experiment.

---

# V6 — Combined Behavioral Features

V6 combines RFM with several additional customer-experience features.

### Feature Set

**RFM**

- Recency
- Frequency
- Monetary value

**Review**

- Average review score

**Delivery**

- Average delivery days
- Late-delivery rate

**Product Behavior**

- Product-category diversity

The final V6 feature set contains:

```text
recency
frequency
monetary
avg_review_score
avg_delivery_days
late_delivery_rate
category_diversity
```

---

## Temporal Leakage Controls

Each feature is restricted according to when that information would actually have been available.

### Reviews

Only reviews created before the cutoff are included.

```text
review_creation_date < cutoff
```

### Delivery Experience

Delivery information is only knowable once the order has actually been delivered.

Therefore delivery features require:

```text
order_delivered_customer_date < cutoff
```

rather than simply:

```text
order_purchase_timestamp < cutoff
```

### Product Category

Category diversity is calculated only from historical orders before the cutoff.

These controls preserve the temporal separation introduced in V4 while allowing richer behavioral information to be incorporated.

---

# Cross-Validated Model Comparison

V5 demonstrated that a single train/test split could produce unstable conclusions.

V6 therefore uses **5-fold Stratified Cross-Validation**.

Both the RFM baseline and V6 are evaluated using:

- The same customers
- The same folds
- The same model family
- The same hyperparameter configuration
- The same class-imbalance treatment

The primary experimental difference is the feature set.

### Results

| Model | Features | CV ROC AUC |
|---|---|---:|
| V4 Baseline | RFM | **0.4995 ± 0.0218** |
| V6 | RFM + Review + Delivery + Category | **0.5232 ± 0.0312** |

V6 achieved a higher ROC-AUC than the RFM baseline in **all 5 cross-validation folds**.

This provides consistent, though modest, evidence that the richer behavioral feature set contains additional predictive information beyond RFM alone.

Importantly, the cross-validated V4 result also demonstrates why the earlier single-split ROC-AUC of 0.5461 should not be interpreted as the definitive baseline.

---

# Model Evolution

The project intentionally preserves the complete modeling history rather than reporting only the best result.

| Version | Main Change | Evaluation | Interpretation |
|---|---|---:|---|
| V1/V2 | RFM → `is_high_value` | ~1.00 ROC AUC | Target leakage |
| V3 | Hyperopt tuning | 1.0000 ROC AUC | Tuned but still leaked |
| V4 | Temporal future target | 0.4995 CV mean* | Honest RFM baseline |
| V5 | Added review score | 0.5345 single split | Inconclusive |
| V6 | Review + delivery + category | **0.5232 CV mean** | Best current honest feature set |

\*V4 originally produced 0.5461 on a single 180-day train/test experiment. For the controlled V6 comparison, V4 was re-evaluated using the same 5-fold CV procedure and produced 0.4995 ± 0.0218.

### Key Modeling Lesson

```text
Perfect metric
      ↓
Investigate validity
      ↓
Identify target leakage
      ↓
Redesign target temporally
      ↓
Obtain realistic baseline
      ↓
Engineer richer features
      ↓
Use stronger evaluation
      ↓
Measure modest but defensible improvement
```

This progression is a central outcome of the project.

---

# MLflow Experiment Tracking

MLflow is used throughout the machine learning lifecycle for:

- Experiment organization
- Parameter tracking
- Metric logging
- Hyperparameter-trial tracking
- Model artifact storage
- Model comparison

Example experiment progression:

```text
V1 / V2
   ↓
V3 — Hyperopt
   ↓
V4 — Temporal Baseline
   ↓
V5 — Review Feature
   ↓
V6 — Combined Features
```

V6 is logged as the current strongest honestly evaluated experimental model.

---

# Class Imbalance Handling

Future repeat purchase is highly imbalanced.

The modeling pipeline therefore calculates:

```python
scale_pos_weight = negative_examples / positive_examples
```

and supplies the value to XGBoost.

For cross-validation, **StratifiedKFold** is used to preserve the positive/negative class distribution across folds.

Because of the imbalance, model evaluation focuses on metrics such as:

- ROC-AUC
- Average Precision
- Precision
- Recall
- F1 Score

rather than relying on accuracy alone.

---

# Retrieval-Augmented Generation (RAG)

The project also implements semantic retrieval over customer reviews.

```text
Business Question
        │
        ▼
Sentence Transformer
        │
        ▼
Review Embeddings
        │
        ▼
FAISS Vector Index
        │
        ▼
Semantic Retrieval
        │
        ▼
Relevant Reviews
        │
        ▼
Structured Business Insight
```

The pipeline:

- Generates sentence embeddings
- Stores vectors in FAISS
- Performs similarity-based retrieval
- Returns relevant customer reviews
- Produces structured business insights

### Example Questions

- Why are customers unhappy with delivery?
- What do customers like about the products?
- What are the main complaints about product quality?

> The current implementation focuses on semantic retrieval and structured insight extraction. Full LLM-based response synthesis remains a planned enhancement.

---

# Pipeline Orchestration — Databricks Workflows

The project is orchestrated using a scheduled Databricks Workflow.

```text
ingest_bronze
      │
      ▼
bronze_layer
      │
      ▼
silver_layer
      │
      ▼
gold_layer_rfm
      │
      ├─────────────────────────────────────┐
      │                                     │
      ▼                                     ▼
Production Path                    Experimentation Path
      │                                     │
      ▼                                     ▼
clv_model                  experimental_hyperparameter_tuning
      │                                     │
      ▼                                     ▼
model_deployment             experimental_temporal_target
```

### Production Path

```text
Gold Features
     ↓
CLV Model
     ↓
Batch Deployment
```

The existing production branch continues to use the validated earlier deployment workflow.

### Experimentation Path

```text
Gold Features
     ↓
Hyperparameter Tuning
     ↓
Temporal Model Experiment
```

This separates active experimentation from the existing deployment pipeline.

Notebooks 11 and 12 are currently evaluated independently and are **not yet connected to the scheduled DAG**.

V6 is the current candidate for the next deployment iteration after the required integration and validation work.

---

## Workflow Automation

The Databricks Workflow is configured with:

- Dependency-based task execution
- Production/experimentation separation
- **Daily execution at 6:00 AM America/New_York**
- Failure-only email notifications
- End-to-end task monitoring

The DAG was validated through two complete manual runs with all configured tasks completing successfully.

---

# Business Value

The platform demonstrates how a unified Lakehouse architecture can support multiple customer-analytics use cases.

### Customer Strategy

- Segment customers by purchasing behavior
- Identify high-value customer groups
- Analyze repeat-purchase patterns

### Customer Experience

- Incorporate review information
- Measure delivery experience
- Analyze product-category behavior

### Predictive Analytics

- Build forward-looking repeat-purchase models
- Compare transactional and behavioral features
- Track experiments systematically

### Business Intelligence

- Retrieve relevant customer feedback using natural-language questions
- Surface customer-review insights through semantic search

### MLOps

- Track experiments using MLflow
- Separate production and experimentation workflows
- Automate pipeline execution using Databricks Workflows

---

# Key Findings

1. **Near-perfect model performance can indicate a modeling problem rather than a successful model.**

   The original RFM-derived target produced misleadingly strong results because the target was constructed from the same variables used as features.

2. **Temporal validation fundamentally changed the model interpretation.**

   Once historical features were separated from future behavior, RFM alone provided little reliable predictive signal.

3. **Single train/test splits were unstable for this imbalanced prediction problem.**

   Cross-validation produced a substantially more conservative baseline than the original single-split experiment.

4. **Behavioral and customer-experience features added incremental signal.**

   Review, delivery, and category-diversity features consistently improved the RFM baseline across the five CV folds, although the absolute performance remains modest.

5. **Model-development rigor mattered more than maximizing the headline metric.**

   The strongest outcome of the project is not a high ROC-AUC, but a progressively more defensible modeling and evaluation pipeline.

---

# Current Project Status

| Component | Status |
|---|---|
| Bronze/Silver/Gold Lakehouse | ✅ Completed |
| RFM Customer Segmentation | ✅ Completed |
| Initial XGBoost Modeling | ✅ Completed |
| MLflow Experiment Tracking | ✅ Completed |
| Hyperopt Tuning | ✅ Completed |
| Temporal Target Redesign | ✅ Completed |
| Review Feature Experiment | ✅ Completed |
| Combined Behavioral Features | ✅ Completed |
| 5-Fold Model Comparison | ✅ Completed |
| Batch Model Deployment | ✅ Completed |
| RAG Semantic Retrieval | ✅ Completed |
| Databricks Workflow | ✅ Completed |
| Automated Scheduling | ✅ Completed |
| V6 Workflow Integration | ✅ Completed |
| Real-Time Model Serving | ⏳ Planned |
| Drift Monitoring | ⏳ Planned |
| LLM Response Synthesis | ⏳ Planned |
| Interactive Dashboard | ⏳ Planned |

---

# Future Enhancements

## 1. V6 Production Integration

Integrate the V6 feature-engineering and inference pipeline into the Databricks Workflow and evaluate replacing the existing production model.

## 2. Temporal Hyperparameter Retuning

Retune XGBoost directly against the temporal V6 feature set and target rather than relying on hyperparameters originally selected during the earlier modeling stage.

## 3. Real-Time Model Serving

Expose the selected model through a model-serving endpoint for low-latency prediction.

```text
Application
    ↓
Model Serving Endpoint
    ↓
Customer Features
    ↓
Repeat-Purchase Probability
```

## 4. Drift Monitoring

Track changes in important feature distributions such as:

- Recency
- Frequency
- Monetary value
- Review score
- Delivery behavior
- Category diversity

and compare production distributions against the training baseline.

## 5. LLM-Based RAG Synthesis

Extend the existing retrieval pipeline from:

```text
Question → Embeddings → FAISS → Relevant Reviews
```

to:

```text
Question
   ↓
Embeddings
   ↓
Vector Retrieval
   ↓
Relevant Reviews
   ↓
LLM
   ↓
Grounded Business Answer
```

## 6. Additional Predictive Signals

Potential future signals include:

- Seasonality
- Customer-service interactions
- Browsing behavior
- Promotion engagement
- Product-level behavioral history

Some of these variables are not available in the current Olist dataset and would require additional data sources.

## 7. Interactive Business Dashboard

Develop an analytics interface for:

- Customer segments
- Repeat-purchase probabilities
- Model monitoring
- Customer-review insights
- Business KPIs

---

# Repository Highlights

This project demonstrates practical experience with:

- Databricks Lakehouse architecture
- PySpark data engineering
- Delta Lake
- Feature engineering
- Temporal machine learning
- Target-leakage detection
- Imbalanced classification
- XGBoost
- Hyperopt
- Cross-validation
- MLflow
- Databricks Workflows
- MLOps concepts
- Embeddings
- FAISS
- Retrieval-Augmented Generation

---

# Author

**Elita Hazel Gorimanikonda**

M.S. Data Science  
Stony Brook University  
May 2026