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

We will be using:
+ Docker
+ Airflow in Docker environment
+ Python pandas
+ Postgres, SQL
+ Apache Arrow
+ GCP
+ NYC Taxi dataset   
In this module we will create a more complex pipeline:
```
(web)
  ↓
DOWNLOAD
  ↓
(csv)
  ↓
PARQUETIZE
  ↓
(parquet) ------→ UPLOAD TO S3
  ↓
UPLOAD TO GCS
  ↓
(parquet in GCS)
  ↓
UPLOAD TO BIGQUERY
  ↓
(table in BQ)
```
_Parquet_ is a [columnar storage datafile format](https://parquet.apache.org/) which is more efficient than CSV.  
This ***Data Workflow*** has more steps and even branches. This type of workflow is often called a ***Directed Acyclic Graph*** (DAG) because it lacks any loops and the data flow is well defined.

The steps in capital letters are our ***jobs*** and the objects in between are the jobs' outputs, which behave as ***dependencies*** for other jobs. Each job may have its own set of ***parameters*** and there may also be global parameters which are the same for all of the jobs.

A ***Workflow Orchestration Tool*** allows us to define data workflows and parametrize them; it also provides additional tools such as history and logging.

The tool we will focus on in this course is **[Apache Airflow](https://airflow.apache.org/)**, There are several tools available for workflow orchestration in data engineering, some of them are:
`Apache Airflow`, 'Luigi', 'AWS Glue', 'Prefect', 'Google cloud composer'(by GCP), 'Microsoft Azure Data Factory'(BY AZURE).   
   https://airflow.apache.org/docs/apache-airflow/2.0.1/concepts.html

## Airflow Architecture
[source](https://airflow.apache.org/docs/apache-airflow/2.0.1/concepts.html)
![image](https://airflow.apache.org/docs/apache-airflow/2.0.1/_images/arch-diag-basic.png)
* A **metadata database** (Postgres) used by the scheduler, the executor and the web server to store state. The backend of Airflow.
* The **scheduler** and **web server** handles both triggering scheduled workflows as well as submitting _tasks_ to the executor to run. The scheduler is the main "core" of Airflow.
* The **executor** handles running tasks. In a default installation, the executor runs everything inside the scheduler but most production-suitable executors push task execution out to _workers_.
* A **worker** simply executes tasks given by the scheduler.
* A **DAG directory**; a folder with _DAG files_ which is read by the scheduler and the executor (an by extension by any worker the executor might have) **DAG** contains python code, representing the data pipelines to be run by  airflow. The location is specified in the Airflow configuration file, which need to be accessible by the web server, scheduler, and workers.

* Additional components (not shown in the diagram):
  * `redis`: a _message broker_ that forwards messages from the scheduler to workers.
  * `flower`: app for monitoring the environment, available at port `5555` by default.
  * `airflow-init`: initialization service which we will customize for our needs.

Airflow will create a folder structure when running:
* `./dags` - `DAG_FOLDER` for DAG files
* `./logs` - contains logs from task execution and scheduler.
* `./plugins` - for custom plugins

Additional definitions:
* ***DAG***: Directed acyclic graph, specifies the dependencies between a set of tasks with explicit execution order, and has a beginning as well as an end. (Hence, “acyclic”). A _DAG's Structure_ is as follows:
  * DAG Definition
  * Tasks (eg. Operators)
  * Task Dependencies (control flow: `>>` or `<<` )  
* ***Task***: a defined unit of work. The Tasks themselves describe what to do, be it fetching data, running analysis, triggering other systems, or more. Common Types of tasks are:
  * ***Operators*** (used in this workshop) are predefined tasks. They're the most common.
  * ***Sensors*** are a subclass of operator which wait for external events to happen.
  * ***TaskFlow decorators*** (subclasses of Airflow's BaseOperator) are custom Python functions packaged as tasks.
* ***DAG Run***: individual execution/run of a DAG. A run may be scheduled or triggered.
* ***Task Instance***: an individual run of a single task. Task instances also have an indicative state, which could be `running`, `success`, `failed`, `skipped`, `up for retry`, etc.
    * Ideally, a task should flow from `none`, to `scheduled`, to `queued`, to `running`, and finally to `success`.
## Setting up Airflow with Docker
[video source: Lightweight Local Setup for Airflow](https://www.youtube.com/watch?v=A1p5LQ0zzaQ&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb&index=24)
## Setting up Mage with Docker
```
git clone https://github.com/mage-ai/mage-zoomcamp.git mage-zoomcamp
cd mage-zoomcamp

# rename dev.env to .env
cp dev.env .env

docker compose build
docker compose up
```
## Ingesting data to local Postgres with Mage

We have a new profile in io_config.yaml and define some variables:
```yaml
dev:
    # PostgresSQL
  POSTGRES_CONNECT_TIMEOUT: 10
  POSTGRES_DBNAME: "{{ env_var('POSTGRES_DBNAME') }}"
  POSTGRES_SCHEMA: "{{ env_var('POSTGRES_SCHEMA') }}" # Optional
  POSTGRES_USER: "{{ env_var('POSTGRES_USER') }}"
  POSTGRES_PASSWORD: "{{ env_var('POSTGRES_PASSWORD') }}"
  POSTGRES_HOST: "{{ env_var('POSTGRES_HOST') }}"
  POSTGRES_PORT: "{{ env_var('POSTGRES_PORT') }}"

```

We will create a new pipeline:
- Go to http://localhost:6789/pipelines
we will start a new ETL Pipeline:
- Data loader:
To a start we have to declare data types:
```python
    taxi_dtypes = {
        'VendorID': pd.Int64Dtype(),
        'passenger_count': pd.Int64Dtype(),
        'trip_distance': float, 
        'RatecCodeID': pd.Int64Dtype(),
        'store_and_fwd_flag': str,
        'PULocationID': pd.Int64Dtype(),
        'DOLocationID': pd.Int64Dtype(),
        'payment_type': pd.Int64Dtype(),
        'fare_amount': float, 
        'mta_tax': float,
        'tip_amount': float,
        'tolls_amount': float, 
        'improvement_surcharge': float,
        'total_amount': float,
        'congestion_surcharge': float

    }
```

And we have to feed dates columns to parsing by pandas:
```python
parse_dates = ['tpep_pickup_datetime', 'tpep_dropoff_datetime']
```

Finally our Data Loader would be:
```python
import io
import pandas as pd
import requests
if 'data_loader' not in globals():
    from mage_ai.data_preparation.decorators import data_loader
if 'test' not in globals():
    from mage_ai.data_preparation.decorators import test


@data_loader
def load_data_from_api(*args, **kwargs):
    """
    Template for loading data from API
    """
    url = 'https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz'

    taxi_dtypes = {
        'VendorID': pd.Int64Dtype(),
        'passenger_count': pd.Int64Dtype(),
        'trip_distance': float, 
        'RatecCodeID': pd.Int64Dtype(),
        'store_and_fwd_flag': str,
        'PULocationID': pd.Int64Dtype(),
        'DOLocationID': pd.Int64Dtype(),
        'payment_type': pd.Int64Dtype(),
        'fare_amount': float, 
        'mta_tax': float,
        'tip_amount': float,
        'tolls_amount': float, 
        'improvement_surcharge': float,
        'total_amount': float,
        'congestion_surcharge': float

    }

    parse_dates = ['tpep_pickup_datetime', 'tpep_dropoff_datetime']

    return pd.read_csv(url, sep=",", compression="gzip", dtype=taxi_dtypes, parse_dates=parse_dates)


@test
def test_output(output, *args) -> None:
    """
    Template code for testing the output of the block.
    """
    assert output is not None, 'The output is undefined'

```

And then we'll be do some transformation block. We will clean data with zero "passenger_count"
```python
if 'transformer' not in globals():
    from mage_ai.data_preparation.decorators import transformer
if 'test' not in globals():
    from mage_ai.data_preparation.decorators import test


@transformer
def transform(data, *args, **kwargs):
    """
    Template code for a transformer block.

    Add more parameters to this function if this block has multiple parent blocks.
    There should be one parameter for each output variable from each parent block.

    Args:
        data: The output from the upstream parent block
        args: The output from any additional upstream blocks (if applicable)

    Returns:
        Anything (e.g. data frame, dictionary, array, int, str, etc.)
    """
    # Specify your transformation logic here

    return data[data['passenger_count'] > 0]


@test
def test_output(output, *args) -> None:
    """
    Template code for testing the output of the block.
    """
    assert output['passenger_count'].isin([0]).sum() == 0, 'Ther are rides with zero passengers'

```

Then we will do some Export Block using profile created in previous step (dev), and declaring schema name and table name:

```python
from mage_ai.settings.repo import get_repo_path
from mage_ai.io.config import ConfigFileLoader
from mage_ai.io.postgres import Postgres
from pandas import DataFrame
from os import path

if 'data_exporter' not in globals():
    from mage_ai.data_preparation.decorators import data_exporter


@data_exporter
def export_data_to_postgres(df: DataFrame, **kwargs) -> None:
    """
    Template for exporting data to a PostgreSQL database.
    Specify your configuration settings in 'io_config.yaml'.

    Docs: https://docs.mage.ai/design/data-loading#postgresql
    """
    schema_name = 'ny_taxi'  # Specify the name of the schema to export data to
    table_name = 'yellow_cab_data'  # Specify the name of the table to export data to
    config_path = path.join(get_repo_path(), 'io_config.yaml')
    config_profile = 'dev'

    with Postgres.with_config(ConfigFileLoader(config_path, config_profile)) as loader:
        loader.export(
            df,
            schema_name,
            table_name,
            index=False,  # Specifies whether to include index in exported table
            if_exists='replace',  # Specify resolution policy if table name already exists
        )

```
finally, we will check the result:
![result](https://github.com/teenbress/DataEngineeringZoomCamp/blob/main/images/mage%20result.png)
### Ingesting data to GCP


