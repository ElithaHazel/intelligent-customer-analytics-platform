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

• Implements a Medallion Lakehouse Architecture using Delta Lake

• Processes over 1.5 million records from 9 relational datasets

• Engineers RFM features for customer segmentation

• Predicts high-value customers using XGBoost

• Tracks experiments using MLflow

• Uses Retrieval-Augmented Generation (RAG) to answer business questions from customer reviews

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

Recency was identified as the strongest predictor of Customer Lifetime Value.

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

# Business Value

The platform enables organizations to:

- Identify high-value customers
- Improve retention strategies
- Analyze customer sentiment
- Support marketing decisions
- Provide business users with natural language access to customer insights

---

# Future Enhancements

- Hyperparameter Optimization
- Model Deployment
- Databricks Workflows
- Interactive Business Dashboard
- Production RAG Deployment

---

# Author

**Elita Hazel Gorimanikonda**

M.S. Data Science

Stony Brook University

May 2026
