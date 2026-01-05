---
date: 2025-12-19 06:20:35 +0301
title: Data engineering with Azure & Databricks
subtitle: ETL using Azure and Databricks with a medallion architecture (In Progress)
image: '/images/data-engineering-project.svg'
hide_image: true
---

## 📋 Project Overview

When I decided on my first portfolio project — the [UK data job market insights](https://www.graemeboulton.com/project/job-insights-project-copy) — one of its main purposes was to better understand where demand and value existed in the market, and to use those insights to help guide my own upskilling.

From a data engineering perspective, **Azure** appeared very dominant in the UK market, with services such as **Databricks** and **Synapse** mentioned consistently. From a BI perspective, **Tableau** emerged as the second most commonly requested tool. Based on this, I decided to incorporate all three into this project.

I was also keen to work with a different type of data source, so for this project I chose an **on-premises SQL Server** and used the [AdventureWorks sample database](https://learn.microsoft.com/en-us/sql/samples/adventureworks-install-configure?view=sql-server-ver17&tabs=ssms). In terms of transformations, I wanted to follow an industry-standard **Bronze → Silver → Gold** pattern to reflect how modern data platforms are typically structured in production environments.

---

## 🏗️ Architecture

![Jobs Pipeline](/images/data-engineering-pipeline.svg)

---

## 🛠️ Technical Stack

- **Azure Data Factory** - Orchestration and data integration
- **Azure Data Lake Storage Gen2** - Scalable data lake storage
- **Azure Databricks** - Data transformation and analytics
- **Azure Synapse Analytics** - Serverless SQL querying
- **Azure Key Vault** - Secure secrets management
- **Microsoft Entra ID** - Identity and access management
- **Tableau** - Business intelligence and visualization

---

## 💡 💡 Implementation Overview

### 🔌 Data Ingestion

Data ingestion was handled using **Azure Data Factory** to securely extract data from an **on-premises SQL Server** environment and land it in Azure in a scalable, repeatable way.

Because the source database was hosted locally, a **Self-Hosted Integration Runtime (SHIR)** was installed and configured on the machine running SQL Server. This allowed Azure Data Factory to communicate securely with the on-prem environment without exposing the database publicly.

Rather than hard-coding individual tables, the ingestion pipeline was designed to be **fully dynamic**:

- A **Lookup activity** queries SQL Server system tables to discover all tables within the `SalesLT` schema
- The results are returned as JSON and passed into a **ForEach activity**
- Each table is extracted using a dynamically generated `SELECT * FROM schema.table` statement

This approach ensures the pipeline automatically adapts if tables are added or removed, without requiring code changes.

All extracted data is written to **Azure Data Lake Storage Gen2** in **Parquet format**, organised into a Bronze layer using a structured folder layout:

```
bronze/{schema}/{table}/{table}.parquet
```

---

### 🔄 Data Transformation

Data transformation was implemented using **Azure Databricks** and follows a clear **Bronze → Silver → Gold** pattern to separate raw data from curated and business-ready datasets.

#### Bronze → Silver

The Silver layer focuses on **data cleanliness and consistency**. Databricks notebooks read the raw Parquet files from the Bronze layer and the following transformations were applied:

- Standardising data types (e.g. converting timestamps to dates where appropriate)
- Cleaning and normalising columns
- Removing technical inconsistencies from the source system
- Ensuring schemas are consistent across tables

The cleaned data is written to the Silver layer using **Delta Lake**, which adds transaction support, schema enforcement, and version history on top of Parquet.

#### Silver → Gold

The Gold layer represents **analytics-ready business data**. At this stage, the focus shifts from cleaning to modelling:

- Joining related tables into meaningful datasets
- Applying business logic and calculations
- Renaming columns into consistent, human-readable formats
- Producing fact and dimension-style outputs optimised for BI tools

Delta Lake continues to be used at this layer, enabling reliable overwrites, easy debugging via version history, and resilience to schema evolution.

---

### ⚙️ Orchestration

End-to-end orchestration is handled by **Azure Data Factory**, which coordinates ingestion, transformation, and downstream processing.

The pipeline flow is:

1. Extract data from on-prem SQL into the Bronze layer  
2. Trigger Databricks notebooks to transform Bronze → Silver  
3. Trigger Databricks notebooks to transform Silver → Gold  

Databricks notebooks are integrated into Data Factory using **linked services**, with sensitive access tokens stored securely in **Azure Key Vault**.

A **daily schedule trigger** ensures the entire pipeline runs automatically, keeping analytics data up to date without manual intervention.

Operational visibility is provided through:

- Pipeline run monitoring
- Activity-level success and failure tracking
- The ability to **rerun failed steps** without reprocessing the entire pipeline

This orchestration design closely mirrors how production data pipelines are built and operated in real-world Azure environments.

---

## 📊 Serving & Reporting

The curated Gold datasets are exposed via **Azure Synapse Analytics (Serverless SQL)** and consumed directly by **Tableau**.

Using a serverless SQL layer allows Tableau to query the data lake without duplicating data, while still providing a familiar relational interface for analytics and visualisation.

Note: Dashboards are currently in progress

---

## 🔒 Security & Governance

- **Managed identities** used across Azure services
- **Secrets** stored securely in Azure Key Vault
- **RBAC** enforced via Microsoft Entra ID
- **Security groups** used to manage access at scale

---

## ✨ Outcome

- ✅ **Fully automated** end-to-end data pipeline
- ✅ **Analytics-ready** data model optimised for Tableau
- ✅ **Daily refreshed** dashboards without manual intervention
- ✅ **Scalable**, production-aligned Azure architecture
