---
title: hive Queries crash when inserting GC Exception
tags:
  - hadoop
  - hive
  - GC
  - exception
  - beeline
date: 2016-10-19 00:05:00
---


When running some queries with hive sometimes we get a very nice java exception of overhead limit

```
Exception in thread "main" java.lang.OutOfMemoryError: GC overhead limit exceeded
```

I didn't know but it turns out you can supply directly on the jdbc connection of beeline the parameters to increase this values.


```
jdbc:hive2://localhost:10000/default?mapreduce.map.memory.mb=3809;mapreduce.map.java.opts=-Xmx3428m;mapreduce.reduce.memory.mb=2560;mapreduce.reduce.java.opts=-Xmx2304m;
```

It is important to understand the size of the containers in your cluster and this is usefull for some adhoc procedures.

The best way is to configure the containers memory size in hadoop, but this kind of quick solutions are usefull, expecially for testing parameters of new workflows.

There is some good info regarding this configuration in altiscale documentation:

https://documentation.altiscale.com/heapsize-for-mappers-and-reducers

Cheers,
RR
