---
title: Polybase Configuration with Cloudera 5
tags:
  - Cloudera
  - Polybase
  - HDFS
  - Sqoop
  - SQLServer
  - Data Ingestion
date: 2017-09-09 01:06:30
---


In this article i'll try to describe the required configuration steps to setup Polybase in SQLServer 2016 for [Cloudera](https://www.cloudera.com) CDH5. 

# Intro

Before starting the the configuration steps, let's just try to understand why this is being done.

## Sqoop

Sqoop is one of the most used tools to transfer data from the Relational World to BigData infrastructures. It relies on JDBC connectors to transfer data between SQLServer and HDFS.

## Performance

The transfer process to SQLServer via sqoop is taking quite a lot, so the objective of this PoC is to verify if PolyBase alternative for dumping data in/out the cluster and understand if there is a improvement on existing processes.

# Setup Process

## Obtain the cluster configuration files

In order to configure Polybase for you Cloudera cluster one should first gather from CM the client configurations for HDFS and YARN.

After downloading the configs you need to update the following files:

* yarn-site.xml
* mapred-site.xml
* hdfs-site.xml

Copy this files to your SQLServer instance where Polybase will be installed.

The usual path is

```
C:\Program Files\Microsoft SQL Server\MSSQL13.MSSQLSERVER\MSSQL\Binn\Polybase\Hadoop\conf  
```

**Note:** Please note, that when PolyBase authenticates to a Kerberos secured cluster, we require the `hadoop.rpc.protection` setting to be set to authentication. This will leave the data communication between Hadoop nodes unencrypted.

##  Activate the required Feature

On the SQLServer you are going to activate Polybase make sure you have the required pre-requisites

* 64-bit SQL Server Evaluation edition
* Microsoft .NET Framework 4.5.
* Oracle Java SE RunTime Environment (JRE) version 7.51 or higher (64-bit) 
* Minimum memory: 4GB
* Minimum hard disk space: 2GB
* TCP/IP must be enabled for Polybase to function correctly. 

Follow Microsoft guide to activate the feature [PolyBase Install Guide](https://docs.microsoft.com/en-us/sql/relational-databases/polybase/polybase-installation)

One could test if Polybase is correctly installed by running the following command

{% codeblock lang:SQL %}
SELECT SERVERPROPERTY ('IsPolybaseInstalled') AS IsPolybaseInstalled;  
{% endcodeblock %}

## Configure External data source

Execute the following T-SQL to create Hadoop connectivity to CDH5

{% codeblock lang:SQL %}
-- Values map to various external data sources.  
--  Option 6: Cloudera 5.1, 5.2, 5.3, 5.4, 5.5, 5.9, 5.10, 5.11, and 5.12 on Linux
sp_configure @configname = 'hadoop connectivity', @configvalue = 6;   
GO   

RECONFIGURE   
GO  
{% endcodeblock %}

You could change the option in case you use a different Hadoop Cluster check the [Option Mapping](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/polybase-connectivity-configuration-transact-sql)

**Note:** After running RECONFIGURE, you must stop and restart the SQL Server service.

## Create the T-SQL objects

Follow the example configuration described in [Getting Started with Polybase](https://docs.microsoft.com/en-us/sql/relational-databases/polybase/get-started-with-polybase)

{% codeblock lang:SQL %}
-- 1: Create a database scoped credential.  
-- Create a master key on the database. This is required to encrypt the credential secret.  

CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'S0me!nfo';  

-- 2: Create a database scoped credential  for Kerberos-secured Hadoop clusters.  
-- IDENTITY: the Kerberos user name.  
-- SECRET: the Kerberos password  

CREATE DATABASE SCOPED CREDENTIAL HadoopUser1   
WITH IDENTITY = '<hadoop_user_name>', Secret = '<hadoop_password>';  

-- 3:  Create an external data source.  
-- LOCATION (Required) : Hadoop Name Node IP address and port.  
-- RESOURCE MANAGER LOCATION (Optional): Hadoop Resource Manager location to enable pushdown computation.  
-- CREDENTIAL (Optional):  the database scoped credential, created above.  

CREATE EXTERNAL DATA SOURCE MyHadoopCluster WITH (  
        TYPE = HADOOP,   
        LOCATION ='hdfs://10.xxx.xx.xxx:xxxx',   
        RESOURCE_MANAGER_LOCATION = '10.xxx.xx.xxx:xxxx',   
        CREDENTIAL = HadoopUser1        
);  

-- 4: Create an external file format.  
-- FORMAT TYPE: Type of format in Hadoop (DELIMITEDTEXT,  RCFILE, ORC, PARQUET).    
CREATE EXTERNAL FILE FORMAT TextFileFormat WITH (  
        FORMAT_TYPE = DELIMITEDTEXT,   
        FORMAT_OPTIONS (FIELD_TERMINATOR ='|',   
                USE_TYPE_DEFAULT = TRUE)  

-- 5:  Create an external table pointing to data stored in Hadoop.  
-- LOCATION: path to file or directory that contains the data (relative to HDFS root).  

CREATE EXTERNAL TABLE [dbo].[CarSensor_Data] (  
        [SensorKey] int NOT NULL,   
        [CustomerKey] int NOT NULL,   
        [GeographyKey] int NULL,   
        [Speed] float NOT NULL,   
        [YearMeasured] int NOT NULL  
)  
WITH (LOCATION='/Demo/',   
        DATA_SOURCE = MyHadoopCluster,  
        FILE_FORMAT = TextFileFormat  
);  

-- 6:  Create statistics on an external table.   
CREATE STATISTICS StatsForSensors on CarSensor_Data(CustomerKey, Speed)  
{% endcodeblock %}

## Example Queries

* Import external Data

{% codeblock lang:SQL %}
-- PolyBase Scenario 2: Import external data into SQL Server.  
-- Import data for fast drivers into SQL Server to do more in-depth analysis and  
-- leverage Columnstore technology.  

SELECT DISTINCT   
        Insured_Customers.FirstName, Insured_Customers.LastName,   
        Insured_Customers.YearlyIncome, Insured_Customers.MaritalStatus  
INTO Fast_Customers from Insured_Customers INNER JOIN   
(  
        SELECT * FROM CarSensor_Data where Speed > 35   
) AS SensorD  
ON Insured_Customers.CustomerKey = SensorD.CustomerKey  
ORDER BY YearlyIncome  

CREATE CLUSTERED COLUMNSTORE INDEX CCI_FastCustomers ON Fast_Customers;  
{% endcodeblock %}

* Export External Data

{% codeblock lang:SQL %}
-- PolyBase Scenario 3: Export data from SQL Server to Hadoop.  

-- Enable INSERT into external table  
sp_configure ‘allow polybase export’, 1;  
reconfigure  

-- Create an external table.   
CREATE EXTERNAL TABLE [dbo].[FastCustomers2009] (  
        [FirstName] char(25) NOT NULL,   
        [LastName] char(25) NOT NULL,   
        [YearlyIncome] float NULL,   
        [MaritalStatus] char(1) NOT NULL  
)  
WITH (  
        LOCATION='/old_data/2009/customerdata',  
        DATA_SOURCE = HadoopHDP2,  
        FILE_FORMAT = TextFileFormat,  
        REJECT_TYPE = VALUE,  
        REJECT_VALUE = 0  
);  

-- Export data: Move old data to Hadoop while keeping it query-able via an external table.  
INSERT INTO dbo.FastCustomer2009  
SELECT T.* FROM Insured_Customers T1 JOIN CarSensor_Data T2  
ON (T1.CustomerKey = T2.CustomerKey)  
WHERE T2.YearMeasured = 2009 and T2.Speed > 40;  
{% endcodeblock %}

# Tests

Initial tests are quite good actually, even with the identified issues. Polybase seems quite limited but for the objective in hands migth present like a very viable solution.

Some more tests would be required.


# Issues

* It seems one cannot truncate external tables so an extra process would be required if you plan to use this as part of an ETL process that should support re-runs
* It seems that `hadoop_user_name` is being ignored and polybase still uses `pwc_user` account in cluster.
* Take care on the compression levels you choose as they consume quite a lot CPU on your SQLServer
* The metadata of the tables is allways stored on SQLServer. And when you choose parquet files has source format it stores in parquet meta the colunms as `col-0,col-1,col-3,...` if you map thoose files to a Hive table would require a view with the respective column name mapping.
* Not sure if this can be change but the dumped files to HDFS are splitted in 8, for the initial tests, only bad for small tables.



# Conclusion

- Work in progress

# References

* https://docs.microsoft.com/en-us/sql/relational-databases/polybase/polybase-guide
* https://docs.microsoft.com/en-us/sql/relational-databases/polybase/get-started-with-polybase
* https://docs.microsoft.com/en-us/sql/relational-databases/polybase/polybase-installation