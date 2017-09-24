---
title: Streaming SQL for Kafka (KSQL)
tags:
  - KSQL
  - Kafka
  - SQL
  - Streaming
  - Docker
date: 2017-09-24 15:54:00
---


In this article i'll explore Apache Kafka KSQL

## Requirements

 * Docker
 * docker-compose

## Description

{% blockquote Offical Website Definition%}

KSQL is an open source streaming SQL engine that implements continuous, interactive queries against Apache Kafka™. It allows you to query, read, write, and process data in Apache Kafka in real-time, at scale using SQL commands. KSQL interacts directly with the Kafka Streams API, removing the requirement of building a Java app.

{% endblockquote %}

## PoC Setup

Create the following file `docker-compose.yml`

```
---
version: '2'
services:
  zookeeper:
    image: "confluentinc/cp-zookeeper:latest"
    hostname: zookeeper
    ports:
      - '32181:32181'
    environment:
      ZOOKEEPER_CLIENT_PORT: 32181
      ZOOKEEPER_TICK_TIME: 2000
    extra_hosts:
      - "moby:127.0.0.1"

  kafka:
    image: "confluentinc/cp-enterprise-kafka:latest"
    hostname: kafka
    ports:
      - '9092:9092'
      - '29092:29092'
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:32181
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_METRIC_REPORTERS: io.confluent.metrics.reporter.ConfluentMetricsReporter
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      CONFLUENT_METRICS_REPORTER_BOOTSTRAP_SERVERS: kafka:29092
      CONFLUENT_METRICS_REPORTER_ZOOKEEPER_CONNECT: zookeeper:32181
      CONFLUENT_METRICS_REPORTER_TOPIC_REPLICAS: 1
      CONFLUENT_METRICS_ENABLE: 'true'
      CONFLUENT_SUPPORT_CUSTOMER_ID: 'anonymous'
    extra_hosts:
      - "moby:127.0.0.1"

  schema-registry:
    image: "confluentinc/cp-schema-registry:latest"
    hostname: schema-registry
    depends_on:
      - zookeeper
      - kafka
    ports:
      - '8081:8081'
    environment:
      SCHEMA_REGISTRY_HOST_NAME: schema-registry
      SCHEMA_REGISTRY_KAFKASTORE_CONNECTION_URL: zookeeper:32181
    extra_hosts:
      - "moby:127.0.0.1"

  # Runs the Kafka KSQL data generator for topic called "pageviews"
  ksql-datagen-pageviews:
    image: "confluentinc/ksql-examples:latest"
    hostname: ksql-datagen-pageviews
    depends_on:
      - kafka
      - schema-registry
    # Note: The container's `run` script will perform the same readiness checks
    # for Kafka and Confluent Schema Registry, but that's ok because they complete fast.
    # The reason we check for readiness here is that we can insert a sleep time
    # for topic creation before we start the application.
    command: "bash -c 'echo Waiting for Kafka to be ready... && \
                       cub kafka-ready -b kafka:29092 1 20 && \
                       echo Waiting for Confluent Schema Registry to be ready... && \
                       cub sr-ready schema-registry 8081 20 && \
                       echo Waiting a few seconds for topic creation to finish... && \
                       sleep 2 && \
                       java -jar /usr/share/java/ksql-examples/ksql-examples-0.1-SNAPSHOT-standalone.jar
                       quickstart=pageviews format=delimited topic=pageviews bootstrap-server=kafka:29092 maxInterval=100 iterations=1000 && \
                       java -jar /usr/share/java/ksql-examples/ksql-examples-0.1-SNAPSHOT-standalone.jar
                       quickstart=pageviews format=delimited topic=pageviews bootstrap-server=kafka:29092 maxInterval=1000'"
    environment:
      KSQL_CONFIG_DIR: "/etc/ksql"
      KSQL_LOG4J_OPTS: "-Dlog4j.configuration=file:/etc/ksql/log4j-rolling.properties"
      STREAMS_BOOTSTRAP_SERVERS: kafka:29092
      STREAMS_SCHEMA_REGISTRY_HOST: schema-registry
      STREAMS_SCHEMA_REGISTRY_PORT: 8081
    extra_hosts:
      - "moby:127.0.0.1"

  # Runs the Kafka KSQL data generator for topic called "users"
  ksql-datagen-users:
    image: "confluentinc/ksql-examples:latest"
    hostname: ksql-datagen-users
    depends_on:
      - kafka
      - schema-registry
    # Note: The container's `run` script will perform the same readiness checks
    # for Kafka and Confluent Schema Registry, but that's ok because they complete fast.
    # The reason we check for readiness here is that we can insert a sleep time
    # for topic creation before we start the application.
    command: "bash -c 'echo Waiting for Kafka to be ready... && \
                       cub kafka-ready -b kafka:29092 1 20 && \
                       echo Waiting for Confluent Schema Registry to be ready... && \
                       cub sr-ready schema-registry 8081 20 && \
                       echo Waiting a few seconds for topic creation to finish... && \
                       sleep 2 && \
                       java -jar /usr/share/java/ksql-examples/ksql-examples-0.1-SNAPSHOT-standalone.jar
                       quickstart=users format=json topic=users bootstrap-server=kafka:29092 maxInterval=100 iterations=1000 && \
                       java -jar /usr/share/java/ksql-examples/ksql-examples-0.1-SNAPSHOT-standalone.jar
                       quickstart=users format=json topic=users bootstrap-server=kafka:29092 maxInterval=1000'"
    environment:
      KSQL_CONFIG_DIR: "/etc/ksql"
      KSQL_LOG4J_OPTS: "-Dlog4j.configuration=file:/etc/ksql/log4j-rolling.properties"
      STREAMS_BOOTSTRAP_SERVERS: kafka:29092
      STREAMS_SCHEMA_REGISTRY_HOST: schema-registry
      STREAMS_SCHEMA_REGISTRY_PORT: 8081
    extra_hosts:
      - "moby:127.0.0.1"

  # Runs the Kafka KSQL application
  ksql-cli:
    image: "confluentinc/ksql-cli:latest"
    hostname: ksql-cli
    depends_on:
      - kafka
      - schema-registry
      - ksql-datagen-pageviews
      - ksql-datagen-users
    command: "perl -e 'while(1){ sleep 99999 }'"
    environment:
      KSQL_CONFIG_DIR: "/etc/ksql"
      KSQL_LOG4J_OPTS: "-Dlog4j.configuration=file:/etc/ksql/log4j-rolling.properties"
      STREAMS_BOOTSTRAP_SERVERS: kafka:29092
      STREAMS_SCHEMA_REGISTRY_HOST: schema-registry
      STREAMS_SCHEMA_REGISTRY_PORT: 8081
    extra_hosts:
      - "moby:127.0.0.1"

```

Prepare the enviroment with the following command

```
sudo docker-compose up -d
```

This will create the minimal services to test ksql
* zookeeper
* schema-registry
* kafka
* ksql-cli

It also will create 2 dockers which will generate data in order to test:
* ksql-datagen-pageviews
* ksql-datagen-users

Thoose dockers are kafka producers, which we will use to test KSQL.


## KSQL

Execute the CLI in order to test

```
sudo docker-compose exec ksql-cli ksql-cli local --bootstrap-server kafka:29092
```

You will see something like this

```
                       ======================================
                       =      _  __ _____  ____  _          =
                       =     | |/ // ____|/ __ \| |         =
                       =     | ' /| (___ | |  | | |         =
                       =     |  <  \___ \| |  | | |         =
                       =     | . \ ____) | |__| | |____     =
                       =     |_|\_\_____/ \___\_\______|    =
                       =                                    =
                       =   Streaming SQL Engine for Kafka   =
Copyright 2017 Confluent Inc.                         

CLI v0.1, Server v0.1 located at http://localhost:9098

Having trouble? Type 'help' (case-insensitive) for a rundown of how things work!

ksql>
```


Now let's create a new **STREAM**

```
CREATE STREAM pageviews_original (viewtime bigint, userid varchar, pageid varchar) WITH (kafka_topic='pageviews', value_format='DELIMITED');
```

Let's describe the created object

```
ksql> DESCRIBE pageviews_original;

 Field    | Type            
----------------------------
 ROWTIME  | BIGINT          
 ROWKEY   | VARCHAR(STRING) 
 VIEWTIME | BIGINT          
 USERID   | VARCHAR(STRING) 
 PAGEID   | VARCHAR(STRING) 
```

Now lets create a users **TABLE**

```
CREATE TABLE users_original (registertime bigint, gender varchar, regionid varchar, userid varchar) WITH (kafka_topic='users', value_format='JSON');
```

And confirm the DDL

```
ksql> DESCRIBE users_original;

 Field        | Type            
--------------------------------
 ROWTIME      | BIGINT          
 ROWKEY       | VARCHAR(STRING) 
 REGISTERTIME | BIGINT          
 GENDER       | VARCHAR(STRING) 
 REGIONID     | VARCHAR(STRING) 
 USERID       | VARCHAR(STRING) 
```

So there are two main objects **TABLES** and **STREAMS**. To see the existing ones, execute:

```
ksql> SHOW STREAMS;

 Stream Name        | Kafka Topic | Format    
----------------------------------------------
 PAGEVIEWS_ORIGINAL | pageviews   | DELIMITED 
```

and 

```
ksql> SHOW TABLES;

 Table Name     | Kafka Topic | Format | Windowed 
--------------------------------------------------
 USERS_ORIGINAL | users       | JSON   | false    
```

### Differences between STREAMS and TABLES

**Stream**

A stream is an unbounded sequence of structured data ("facts"). For example, we could have a stream of financial transactions such as "Alice sent $100 to Bob, then Charlie sent $50 to Bob". Facts in a stream are immutable, which means new facts can be inserted to a stream, but existing facts can never be updated or deleted. Streams can be created from a Kafka topic or derived from existing streams and tables.

**Table**

A table is a view of a stream, or another table, and represents a collection of evolving facts. For example, we could have a table that contains the latest financial information such as "Bob’s current account balance is $150". It is the equivalent of a traditional database table but enriched by streaming semantics such as windowing. Facts in a table are mutable, which means new facts can be inserted to the table, and existing facts can be updated or deleted. Tables can be created from a Kafka topic or derived from existing streams and tables.

### Extending the Model

Lets create a new pageviews stream with key `pageid` and joi it with table `users`.

First create the new stream

```
CREATE STREAM pageviews \
   (viewtime BIGINT, \
    userid VARCHAR, \
    pageid VARCHAR) \
   WITH (kafka_topic='pageviews', \
         value_format='DELIMITED', \
         key='pageid', \
         timestamp='viewtime');
```

And users table

```
CREATE TABLE users \
   (registertime BIGINT, \
    gender VARCHAR, \
    regionid VARCHAR, \
    userid VARCHAR, \
    interests array<VARCHAR>, \
    contact_info map<VARCHAR, VARCHAR>) \
   WITH (kafka_topic='users', \
         value_format='JSON');

```

#### Aggregations

Let's test some aggregations

```
SELECT count(*),userid FROM pageviews GROUP BY userid;
```

As data arrives in stream this is constantly updating , but we can verify that the count values begins from the start of the query, this could be usefull to dump to another topic with transformed data.

Let's do that.

```
CREATE TABLE user_counts AS select count(*),userid from pageviews group by userid;
```

We can use the statement `SHOW TOPICS` which is usefull and confirmed that the USER_COUNTS TOPICS is created.

The following aggregations are available

|Function    | Example    | Description |
|------------|------------|--------------|
|COUNT       | COUNT(col1)| Count the number of rows|
|MAX         | MAX(col1)  | Return the maximum value for a given column and window|
|MIN         |MIN(col1)  | Return the minimum value for a given column and window|
|SUM         | SUM(col1) |  Sums the column values|

## Window

The WINDOW clause lets you control how to group input records that have the same key into so-called windows for operations such as aggregations or joins. Windows are tracked per record key. KSQL supports the following WINDOW types:

* TUMBLING
* HOPPING
* SESSION

### Window Tumbling

TUMBLING: Tumbling windows group input records into fixed-sized, non-overlapping windows based on the records' timestamps. You must specify the window size for tumbling windows. Note: Tumbling windows are a special case of hopping windows where the window size is equal to the advance interval.

Example:
```
SELECT item_id, SUM(quantity)
  FROM orders
  WINDOW TUMBLING (SIZE 20 SECONDS)
  GROUP BY item_id;
```


### Window HOPPING

HOPPING: Hopping windows group input records into fixed-sized, (possibly) overlapping windows based on the records' timestamps. You must specify the window size and the advance interval for hopping windows.

Example:

```
SELECT item_id, SUM(quantity)
  FROM orders
  WINDOW HOPPING (SIZE 20 SECONDS, ADVANCE BY 5 SECONDS)
  GROUP BY item_id;
```

### Window SESSION

SESSION: Session windows group input records into so-called sessions. You must specify the session inactivity gap parameter for session windows. For example, imagine you set the inactivity gap to 5 minutes. If, for a given record key such as "alice", no new input data arrives for more than 5 minutes, then the current session for "alice" is closed, and any newly arriving data for "alice" in the future will mark the beginning of a new session.

Example:

```
SELECT item_id, SUM(quantity)
  FROM orders
  WINDOW SESSION (20 SECONDS)
  GROUP BY item_id;
```

## Transformations

Let's create a new stream with a column transformation

```
CREATE STREAM pageviews_transformed \
  WITH (timestamp='viewtime', \
        partitions=5, \
        value_format='JSON') AS \
  SELECT viewtime, \
         userid, \
         pageid, \
         TIMESTAMPTOSTRING(viewtime, 'yyyy-MM-dd HH:mm:ss.SSS') AS timestring \
  FROM pageviews \
  PARTITION BY userid;
```

## Joining

And joining the STREAM with enriched data from TABLE

```
CREATE STREAM pageviews_enriched AS \
   SELECT pv.viewtime, \
          pv.userid AS userid, \
          pv.pageid, \
          pv.timestring, \
          u.gender, \
          u.regionid, \
          u.interests, \
          u.contact_info \
   FROM pageviews_transformed pv \
   LEFT JOIN users u ON pv.userid = users.userid;
```

# Use Cases

Common KSQL use cases are:

* Fraud detection - identify and act on out of the ordinary data to provide real-time awareness.
* Personalization - create real-time experiences and insight for end users driven by data.
* Notifications - build custom alerts and messages based on real-time data.
* Real-time Analytics - power real-time dashboards to understand what’s happening as it does.
* Sensor data and IoT - understand and deliver sensor data how and where it needs to be.
* Customer 360 - provide a clear, real-time understanding of your customers across every interaction.

### Streaming ETL

KSQL makes it simple to transform data within the pipeline, readying messages to cleanly land in another system

### Anomaly Detection

KSQL is a good fit for identifying patterns or anomalies on real-time data. By processing the stream as data arrives you can identify and properly surface out of the ordinary events with millisecond latency.

### Monitoring

Kafka's ability to provide scalable ordered messages with stream processing make it a common solution for log data monitoring and alerting. KSQL lends a familiar syntax for tracking, understanding, and managing alerts.

# Conclusion

On a first glance KSQL seems pretty easy to start using it. It's still pretty new, i would give it a time to mature before start using it on production.

For small transformations, data-quality control and analytical queries that don't take into account a large windows this seems a good solution. For more complex queries i still prefer to keep Kafka to it's core business and let the computation work to Spark.  


## More Information

* KSQL is available as a developer preview on [Github](http://go2.confluent.io/Q0KX0lH0Qq2kx0t3r00U8X0)
* Read the [documentation](http://go2.confluent.io/m000tV2080xr3kXlHKr0QX0) for KSQL
* Join the conversation in the #ksql channel in the [Confluent Community Slack](http://go2.confluent.io/m000tW2080xr3kXlHKs0QX0). 

## References

* https://github.com/confluentinc/ksql
* https://github.com/confluentinc/ksql/tree/0.1.x/docs/quickstart#quick-start
* https://github.com/confluentinc/ksql/tree/0.1.x/ksql-clickstream-demo#clickstream-analysis





