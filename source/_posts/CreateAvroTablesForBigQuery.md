---
title: Create Avro Tables For Google BigQuery
tags:
  - Hive
  - BigQuery
  - Avro
  - HDFS
  - Parquet
  - GCS
date: 2017-09-19 22:59:58
---


This article describes one way to create tables in BigQuery From HDFS parquet Data.

## Requirements

For this article assumes the following requisites are meet:

* You have a Google Cloud Platform account
* You have created a Google Cloud Storage bucket
* You have HDFS configured
* You have Parquet data that you could materialize in Hive
* Must have [bq](https://cloud.google.com/sdk/docs/) util install 

## Definitions

* **Hive Parquet Table:** parquet_table
* **HDFS Parquet Location:** `/user/hive/warehouse/test.db/parquet_table`
* **Hive Avro Table Name:** avro_table
* **HDFS Avro Location:** `/user/hive/warehouse/test.db/avro_table`

## Process

1. Create a avro table from parquet data
2. Copy avro files to GCS
3. Create Bigquery Table from avro in gs bucket

## Setup

### Create the Avro Table

Let's start by creating the new table based on the existing parquet data

```
SET hive.exec.compress.output=true;
SET avro.output.codec=snappy;

CREATE TABLE avro_table STORED AS AVRO
  AS (SELECT * FROM parquet_table);
```

One could specify in the `SELECT` statement the columns we would like that could be obtain using `parquet-tools` command  ex:

```
parquet-tools meta <parquet_file>
```

You would still need to get the parquet file to obtain that.

### Why Avro File and that format ?

You could update data to BigQuery by streaming or from Google Cloud Storage as a batch process. A bulk import from HDFS seems logical to use a batch process so why avro ?
Acording to the latest info on the google BigQuery Site it's possible to:

* Load from Google Cloud Storage, including CSV, JSON (newline-delimited), and Avro files, as well as Google Cloud Datastore backups.
* Load directly from a readable data source.
* Insert individual records using streaming inserts.

> Compressed Avro files are not supported, but compressed data blocks are. BigQuery supports the DEFLATE and Snappy codecs.
> 

Also there is the following Avro mapping that could be usefull

|Avro data type|  BigQuery data type |
|--------------|---------------------|
|null          | - ignored -         |   
|boolean       |BOOLEAN              |
|int           |INTEGER              |
|long          |INTEGER              |
|float         |FLOAT                |
|double        |FLOAT                |
|bytes         |BYTES                |
|string        |STRING               |
|record        |RECORD               |
|enum          |STRING               |
|array         | - repeated fields - |
|map<T>        |RECORD               |
|union         |RECORD               |
|fixed         |BYTES                |

Check the full spec on GCP [Page](https://cloud.google.com/bigquery/data-formats#avro_format)

The other advantage of using avro is that BigQuery infers the schema so you don't have to describe the columns of you table.

### Copy Avro file from HDFS to GCS

The best approach for this is to add the GCS connector to your HDFS config

Follow the instructions in the following [link](https://github.com/GoogleCloudPlatform/bigdata-interop/tree/master/gcs) or download the jar for Hadoop 2.x [here](https://storage.googleapis.com/hadoop-lib/gcs/gcs-connector-latest-hadoop2.jar)

1. Add that jar on a valid location for you cluster `HADOOP_CLASSPATH`
2. Generate a `service account` in the GCP console and get JSON key ([follow this instructions](https://cloud.google.com/storage/docs/authentication#service_accounts))
3. Copy that JSON file to a location in your cluster
4. Add the following properties to your cluster `core-site.xml`

```
  <property>
    <name>fs.gs.project.id</name>
    <value>your-project-name</value>
    <description>
      Required. Google Cloud Project ID with access to configured GCS buckets.
    </description>
  </property>
  <property>
  <name>google.cloud.auth.service.account.json.keyfile</name>
  <value>/path/to/your/JSON-keyfile</value>
  <description>
    The JSON key file of the service account used for GCS
    access when google.cloud.auth.service.account.enable is true.
  </description>
  </property>
    <property>
    <name>fs.gs.working.dir</name>
    <value>/</value>
    <description>
      The directory relative gs: uris resolve in inside of the default bucket.
    </description>
  </property>
```

Extended options are available in [gcs-core-default](https://github.com/GoogleCloudPlatform/bigdata-interop/blob/master/gcs/conf/gcs-core-default.xml) example

5. Create a new bucket on GCP and make sure you can access to with via `hdfs` command.

```
hdfs dfs -ls gs://my-bucket-name/
```

If that works, you can now execute a distcp to sync the avro files directly to GCS.

```
hdfs mkdir gs://my-bucket-name/my_table
hdfs distcp /user/hive/warehouse/test.db/avro_table/* gs://my-bucket-name/my_table/
```

### Load GCS avro data to a BigQuery table

Execute

```
bq load ds.table gs://my-bucket-name/my_table/* --autodetect
```

And that's it you now have a table with data in BigQuery.

It is recommended to have this process managed by some type of orchestrator. There are several solutions for this. The next article i'll be writting  about one of those [Airflow](https://airflow.incubator.apache.org/)

Cheers,
RR

## References

* https://www.cloudera.com/documentation/enterprise/5-4-x/topics/cdh_ig_hive.html
* https://github.com/apache/parquet-mr/tree/master/parquet-tools
* https://cloud.google.com/bigquery/loading-data
* https://github.com/GoogleCloudPlatform/bigdata-interop/blob/master/gcs/INSTALL.md
* https://cloud.google.com/bigquery/docs/loading-data-cloud-storage