---
title: Dr. Elephant Overview
tags:
  - dr-elephant
  - Hadoop
  - Performance
  - Monitoring
  - Tunning
  - Spark
date: 2017-10-13 02:09:12
---


This article would be about [dr-elephant](https://github.com/linkedin/dr-elephant) A Performance and Monitoring tool for Hadoop and Spark.

{% blockquote Offical Website Definition%}

Dr. Elephant is a performance monitoring and tuning tool for Hadoop and Spark. It automatically gathers all the metrics, runs analysis on them, and presents them in a simple way for easy consumption. Its goal is to improve developer productivity and increase cluster efficiency by making it easier to tune the jobs. It analyzes the Hadoop and Spark jobs using a set of pluggable, configurable, rule-based heuristics that provide insights on how a job performed, and then uses the results to make suggestions about how to tune the job to make it perform more efficiently.

{% endblockquote %}

# Requirements

* Install mysql-server and create a BD for dr-elephant
```sh
sudo apt-get install mysql-server 
```

* MySQl preparation

```mysql
mysql> create database drelephant;
Query OK, 1 row affected (0.00 sec)

mysql> grant all on drelephant.* to drelephant@localhost identified by 'drelephant';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

* Install zip command
```sh
sudo apt-get install zip
```

* Install sbt
```sh
echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823
sudo apt-get update
sudo apt-get install sbt
```


# Setup

* Clone the repo

```sh
git https://github.com/linkedin/dr-elephant.git
```

* Compile

```sh
sbt package
sbt dist
cp ./target/universal/dr-elephant-2.0.3-SNAPSHOT.zip .
./compile.sh
cd dist
unzip dr-elephant-2.0.3-SNAPSHOT.zip
```

* Starting the service

```sh
export ELEPHANT_CONF_DIR=../../app-conf
./bin/start.sh 
```

One can now access the web interface at: http://localhost:8080

{% asset_img DrElephant-2-dashboard.jpg [Dashboard] %}

# Conclusion

This tool seems very powerfull. At the moment i haven't tested changing the recommendations it provided, but will try them soon. Spark 2.x applications don't seem to be working at the momment

# Extended tests

* Test Oozie Scheduler integration
* Test Airflow integration
* Define a deployment strategy
* Test recommended changes


# References
 
  * https://github.com/linkedin/dr-elephant
  * https://github.com/linkedin/dr-elephant/commit/7d9e34ecdbe36a652ca4ad2db852df08da57050a
  * https://engineering.linkedin.com/blog/2016/04/dr-elephant-open-source-self-serve-performance-tuning-hadoop-spark
