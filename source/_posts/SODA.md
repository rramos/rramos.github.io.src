---
title: SODA
tags:
   - Data Quality
   - Airflow
   - Docker

---

Bernard Denys kindly share with me the availability of their product for the Open Source Community in [Github](https://github.com/sodadata/soda-sql#readme). [SODA](https://www.soda.io/) provide a Data Monitoring Platform. In this article i will take some time to explore a quick setup on how to use it and final comments around it.

## What does Soda SQL do?

Soda SQL allows you to

Stop your pipeline when bad data is detected
Extract metrics and column profiles through super efficient SQL
Full control over metrics and queries through declarative config files

## 5m tutorial

There is a [5m tutorial](https://docs.soda.io/soda-sql/getting-started/5_min_tutorial.html) available on their official page let's have a go...

## Install

I'm using Linux and i'm lazy so i've just installed the required pip

```bash
pip3 install soda-sql
```

Let's check if it is working

```bash
$ soda 
Usage: soda [OPTIONS] COMMAND [ARGS]...

  Soda CLI version 2.0.0b15

Options:
  --help  Show this message and exit.

Commands:
  analyze  Analyzes tables in the warehouse and creates scan YAML files...
  create   Creates a new warehouse.yml file and prepares credentials in
           your...

  init     Renamed to `soda analyze`
  scan     Computes all measurements and runs all tests on one table.

```

Great now let's create some a dummy warehouse and do some tests:

## Create dummy Datawarehouse

```bash
docker run --name soda_sql_tutorial_db --rm -d \
    -p 5432:5432 \
    -v soda_sql_tutorial_postgres:/var/lib/postgresql/data:rw \
    -e POSTGRES_USER=sodasql \
    -e POSTGRES_DB=sodasql \
    -e POSTGRES_HOST_AUTH_METHOD=trust \
    postgres:9.6.17-alpine
```

And load it with data with the following command

```bash
docker exec soda_sql_tutorial_db \
  sh -c "wget -qO - https://raw.githubusercontent.com/sodadata/soda-sql/main/tests/demo/demodata.sql | psql -U sodasql -d sodasql"
```

According to the tutorial one can remove the created container and volume with the following command :

```bash
docker stop soda_sql_tutorial_db
docker volume rm soda_sql_tutorial_postgres
```

## Create warehouse directory

```bash
mkdir soda_sql_tutorial
cd soda_sql_tutorial/
soda create -d sodasql -u sodasql -w soda_sql_tutorial postgres
```

This created the `warehouse.yml` with connection settings which are stored on your homedir `~/.soda/env_vars.yml` 

## Analyse table scan YAML files

The following command will analyze the exiting tables and fill the `./tables/` directory with large data warehouse it can be inputted manually.

Well, i only have 5m so let's give it a go.

```bash
soda analyze
```

Hum! interesting, queries the information schema and generates a file which i will assume per table with several metrics for validation.

```yaml
table_name: demodata
metrics:
  - row_count
  - missing_count
  - missing_percentage
  - values_count
  - values_percentage
  - valid_count
  - valid_percentage
  - invalid_count
  - invalid_percentage
  - min_length
  - max_length
  - avg_length
  - min
  - max
  - avg
  - sum
  - variance
  - stddev
tests:
  - row_count > 0
columns:
  id:
    valid_format: uuid
    tests:
      - invalid_percentage == 0
  feepct:
    valid_format: number_percentage
    tests:
      - invalid_percentage == 0
```

## Run a scan

Each scan requires a warehouse YAML and a scan YAML as input. The scan command will collect the configured metrics and run the defined tests against them.

```bash
soda scan warehouse.yml tables/demodata.yml
```

And that's it for the tutorial

One can then integrate this with a orchestration tool such as [Airflow](https://docs.soda.io/soda-sql/documentation/orchestrate_scans.html)

It is recommend to `PythonVirtualenvOperator` to prevent any dependency conflicts.

Example dag:

```python
from airflow import DAG
from airflow.models.variable import Variable
from airflow.operators.python import PythonVirtualenvOperator
from airflow.operators.dummy import DummyOperator
from airflow.utils.dates import days_ago
from datetime import timedelta


# Make sure that this variable is set in your Airflow
warehouse_yml = Variable.get('soda_sql_warehouse_yml_path')
scan_yml = Variable.get('soda_sql_scan_yml_path')

default_args = {
    'owner': 'soda_sql',
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}


def run_soda_scan(warehouse_yml_file, scan_yml_file):
    from sodasql.scan.scan_builder import ScanBuilder
    scan_builder = ScanBuilder()
    # Optionally you can directly build the Warehouse dict from Airflow secrets/variables
    # and set scan_builder.warehouse_dict with values.
    scan_builder.warehouse_yml_file = warehouse_yml_file
    scan_builder.scan_yml_file = scan_yml_file
    scan = scan_builder.build()
    scan_result = scan.execute()
    if scan_result.has_failures():
        failures = scan_result.failures_count()
        raise ValueError(f"Soda Scan found {failures} errors in your data!")


dag = DAG(
    'soda_sql_python_venv_op',
    default_args=default_args,
    description='A simple Soda SQL scan DAG',
    schedule_interval=timedelta(days=1),
    start_date=days_ago(1),
)

ingest_data_op = DummyOperator(
    task_id='ingest_data'
)

soda_sql_scan_op = PythonVirtualenvOperator(
    task_id='soda_sql_scan_demodata',
    python_callable=run_soda_scan,
    requirements=["soda-sql==2.0.0b10"],
    system_site_packages=False,
    op_kwargs={'warehouse_yml_file': warehouse_yml,
               'scan_yml_file': scan_yml},
    dag=dag
)

publish_data_op = DummyOperator(
    task_id='publish_data'
)

ingest_data_op >> soda_sql_scan_op >> publish_data_op
```

There is also some work being done to support a SodaScanOperator for Airflow.

## Warehouse types

At the moment SODA is supporting the following techs:

* Snowflake
* AWS Athena
* GCP BigQuery
* PostgreSQL
* Redshift
* Spark SQL -> (Coming Soon)

## Conclusions

This quick tutorial make's it easy to test the tool. But i would like to extended the tests on a larger solution to check how it behaves. Also i would like to see the SparkSQL as a warehouse option as most of my teams work are centered on that technology.

Topics to do extended testing with:

* Schema evolution impact on the quality test setup
* Not using a Warehouse solution but a Datalake
  * Providing schemas from a Datalake and using a processing engine on top of it (ex: spark.schemas and Parquet files)
* Testing with Delta tables
* Define levels of data-quality analysis
* Understanding better how the scans works

The tool seems to have good potential and seems very simple which would speed the adoption. Also by sharing this to the open source community seems to be as a good choice to increase user quorum and still offer enterprise services solution.

## References

* <https://github.com/sodadata/soda-sql#readme>
* <https://docs.soda.io/soda-sql/getting-started/5_min_tutorial.html>
* <https://docs.soda.io/soda-sql/documentation/orchestrate_scans.html>
