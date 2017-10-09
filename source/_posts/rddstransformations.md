---
title: RDD Basic Transformations Operations
tags:
  - Spark
  - RDD
  - CheatSheet
date: 2017-10-09 20:54:12
---


Just a simple Basic CheatSheet on Spark RDD's

* Basic Transformations on a RDD containing: **{1,2,3,3}**

| Function Name | Example                   | Result           |
|---------------|---------------------------|------------------|
| map()         | rdd.map(x => x +1)        | {2,3,4,4}        |
| flatmap()     | rdd.flatMap(x => x.to(3)) | {1,2,3,2,3,3,3}  |
| filter()      | rdd.filter(x => x != 1 )  | {2,3,3}          |
| distinct()    | rdd.distinct()            | {1,2,3}          |
| sample()      | rdd.sample(false,0.5)     | Nondeterministic |

* Basic two-RDD transformations on RDDs: **{1,2,3}** and **{3,4,5}**

| Function Name | Example                   | Result               |
|---------------|---------------------------|----------------------|
| union()       | rdd.union(other)          | {1,2,3,3,4,5}        |
| intersection()| rdd.intersection(other)   | {3}                  |
| subtract()    | rdd.subtract(other)       | {1,2}                |
| cartesian()   | rdd.cartesian(other)      | {(1,3),(1,4)...(3,5)}|

* Basic Actions on RDD containing: **{1,2,3,3}**

| Function Name | Example                   | Result              |
|---------------|---------------------------|---------------------|
| collect()     | rdd.collect()             | {1,2,3,3}           |
| count()       | rdd.count()               | 4                   |
| countByValue()| rdd.countByValue()        | {(1,1),(2,1),(3,2)} |
| take()        | rdd.take(2)               | {1,2}               |
| top()         | rdd.top(2)                | {3,3}               |
| takeOrdered() | rdd.takeOrdered(2)(myOrdering)| {3,3}           |
| takeSample()  | rdd.takeSample(false,1)   | Nondeterministic    |
| reduce()      | rdd.reduce((x,y) => x + y )| 9                  |
| fold()        | rdd.fold(0)((x,y) => x + y)| 9                  |
| foreach()     | rdd.foreach(func)          | Nothing            |









