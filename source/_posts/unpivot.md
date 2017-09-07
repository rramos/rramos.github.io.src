---
title: Hive - Unpivot to avoid multiple joins
date: 2017-09-07 17:57:08
tags: 
  - HDFS
  - Hive
  - Optimization
  - BigData
---


In Hive, when a table has multiple similar columns, such as:

| product_id | image_name1 | image_name2 |... | image_name15 |
|------------|-------------|-------------|----|--------------|
|1           | a.jpg       | b.jpg       |... | c.jpg        |
|2           | d.jpg       | e.jpg       |... | f.jpg        |

and we want to join based on all image_names, the naive approach would be to perform 15 joins (in the running example).

Hive does not have an unpivot functionality, but it is possible to use the Hive builtin UDF explode to accomplish this:
Query example

```
select x.product_id, x.image_name from (
  select product_id, map("image_name1", image_name1, ..., "image_name15", image_name15) as image_map
  from table ) x
lateral view explode(image_map) expl as image_nr, image_name
```


This returns an unpivoted table like below, which allows us to perform a single join:

|product_id| image_name|
|----------|-----------|
|1         | a.jpg     |
|1         | b.jpg     |
|1         | c.jpg     |
|2         | d.jpg     | 
|2         | e.jpg     |
|2         | f.jpg     |

Big thanks to Diogo Franco, for this hint ;) check also is [cool projects] (https://github.com/diogoalexandrefranco)


