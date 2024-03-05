### Module 2: Workflow Orchestration   
- [Data Ingestion](#data-ingestion)
- [Data Lake](#data-lake)
  - [Data Lake vs Data Warehouse](#data-lake-vs-data-warehouse)
  - [ETL vs ELT](#etl-vs-elt)

- [Introduction to Workflow Orchestration](#introduction-to-workflow-orchestration)
  - [Airflow architecture](#airflow-architecture)
  - [Setting up Airflow with Docker](#setting-up-airflow-with-docker)
  - [Running DAGs](#running-dags)
  - [Airflow and DAG tips and tricks](#airflow-and-dag-tips-and-tricks)
- [Mage in action](#mage-in-action)
  - [Setting up Mage with Docker](#setting-up-mage-with-docker)
  - [Ingesting data to local Postgres with Mage](#ingesting-data-to-local-postgres-with-mage)
  - [Ingesting data to GCP](#ingesting-data-to-gcp)
- [GCP's Transfer Service](#gcps-transfer-service)
  - [Creating a Transfer Service from GCP's web UI](#creating-a-transfer-service-from-gcps-web-ui)
  - [Creating a Transfer Service with Terraform](#creating-a-transfer-service-with-terraform)
______
## Data Ingestion
Data ingestion is the process of importing large, assorted data files from multiple sources into a single could-based storage mudium- a data warehouse, data lake or database--where it can be accessed and analyzed.  
There are three ways to ingest data: batch(ETL pipelines), real-time, lambda.
## Data Lake

A **data lake** is a central repository that holds big data from many sources.  
The data in data lake could be structured, unstructured or a mix of both.  
The main goal behind a data lake is being able to ingest data as quickly as possible. A data lake should be: 
+ Secure
+ Scalable
+ Able to run on inexpensive hardware   
There are several popular data lake solutions available, some of them are:
`Amazon S3`, `Azure Data Lake Storage`, `Google Cloud Storage`, `Haddop HDFS`, `Snowflake Data Warehouse`   

There are several popular data warehouse solutions available, some of them are:
`Amazon Redshift`, `Azure Snapse Analytics`, `Google BigQuery`, `Teradata`, `Snowflake` `PostgreSQL` for BI, aNALYTICS AND reporting.   

## Data Lake vs Data Warehouse

A Data Lake (DL) is not to be confused with a Data Warehouse (DW). There are several differences:

* Data Processing:
  * DL: The data is **raw** and has undergone minimal processing. The data is generally unstructured.
  * DW: the data is **refined**; it has been cleaned, pre-processed and structured for specific use cases.
* Size:
  * DL: Data Lakes are **large** and contains vast amounts of data, in the order of petabytes. Data is transformed when in use only and can be stored indefinitely.
  * DW: Data Warehouses are **small** in comparison with DLs. Data is always preprocessed before ingestion and may be purged periodically.
* Nature:
  * DL: data is **undefined** and can be used for a wide variety of purposes.
  * DW: data is historic and **relational**, such as transaction systems, etc.
* Users:
  * DL: Data scientists, data analysts.
  * DW: Business analysts.
* Use cases:
  * DL: Stream processing, machine learning, real-time analytics...
  * DW: Batch processing, business intelligence, reporting.

Data Lakes came into existence because as companies started to realize the importance of data, they soon found out that they couldn't ingest data right away into their DWs but they didn't want to waste uncollected data when their devs hadn't yet finished developing the necessary relationships for a DW, so the Data Lake was born to collect any potentially useful data that could later be used in later steps from the very start of any new projects.

## ETL vs ELT

When ingesting data, DWs use the ***Export, Transform and Load*** (ETL) model whereas DLs use ***Export, Load and Transform*** (ELT).

The main difference between them is the order of steps. In DWs, ETL (Schema on Write) means the data is _transformed_ (preprocessed, etc) before arriving to its final destination, whereas in DLs, ELT (Schema on read) the data is directly stored without any transformations and any schemas are derived when reading the data from the DL.
## Introduction to Workflow orchestration 
_[Video source](https://www.youtube.com/watch?v=0yK7LXwYeD0&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb&index=17)_
- What is an Orchestration Pipeline? 
There are several tools available for workflow orchestration in data engineering, some of them are:
`Apache Airflow`, 'Luigi', 'AWS Glue', 'Prefect', 'Google cloud composer'(by GCP), 'Microsoft Azure Data Factory'(BY AZURE).   
- Directed Acyclic Graph(DAG)

## Airflow Architecture
[note source](https://github.com/ziritrion/dataeng-zoomcamp/blob/main/notes/2_data_ingestion.md#data-ingestion)
## Mage in Action
We will be using:
+ Docker
+ Mage in Docker environment
+ Python pandas
+ Postgres, SQL
+ APACHE ARROW
+ GCP
+ NYC Taxi dataset

![Mage workflow](https://github.com/teenbress/DataEngineeringZoomCamp/blob/main/images/mage%20workflow.png)

**Mage is an open-source pipeline tool for orchestrating, transforming and integrating data.**
Mage's main components:   
Projects --> Pipelines--> Blocks --> ETL   
   
Other types of built-in Mage blocks:   

+ Sensors - trigger on some event
+ Conditionals
+ Dynamics - can create dynamic children
+ Webhooks

Other notable functionality:
   
+ Data Integration
+ Unified Pipeline
+ Multi-user events
+ Templating
## Mage Set Up
