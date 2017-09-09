---
title: Airflow PoC
tags:
    - HDFS
    - Airflow
    - Orchestration
    - BigData
    - Hive
    - Sqoop
    - Docker
    - Spark
    - Metrics
---


In this article i'll describe the procedure to create a [Airflow](https://airflow.incubator.apache.org/) environment running in docker supporting several BigData Operators.

I'll explain how it works, how to setup and try to document my personal thoughts on the software compared to other solutions.

# Introduction

Airflow is a python platform to create, schedule and monitor workflows.

{% blockquote Offical Website Definition%}

Use airflow to author workflows as directed acyclic graphs (DAGs) of tasks. The airflow scheduler executes your tasks on an array of workers while following the specified dependencies. Rich command line utilities make performing complex surgeries on DAGs a snap. The rich user interface makes it easy to visualize pipelines running in production, monitor progress, and troubleshoot issues when needed.

{% endblockquote %}

# PoC architecture

For this PoC i'm considering the following architetcure:

* 1 docker : Scheduler
* 1 docker : Worker
* 1 docker : Flower
* 1 docker : DB
* 1 docker : Webserver
* 1 docker : Metrics exporter 

For a production environment, it should be taken into account a 
better HA-Solution.

# Dockers Setup

TODO

# Launch the Environment

TODO

# Create a initial Workflow

TODO

# Operations

TODO

# Tests

TODO

# Conclusion

TODO














