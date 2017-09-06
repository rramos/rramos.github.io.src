---
title: SparkSummit2016
tags:
  - Conferences
  - Spark
  - SparkSummit
  - Hadoop
  - Streamming
  - Kafka
date: 2016-11-03 22:09:40
---


# SparkSummit 2016

So i've attended 25-27 OCT in Bruxels to SparkSummit 2016 in Europe. So it kind makes sence to write down some impressions regarding the event.

The Event it self was pretty well organized, and the distribution of spaces was well planned.

Regarding Spark it-self, is one of the ecosystem players (https://hadoopecosystemtable.github.io/) that's been getting more traction from the industry, partially because of the Entreprise support from Databricks and other major BigData contributors.

This Summit, had huge increase of participants, compared to last year, what garanties that this is really here to stay.

I allways try to make of this conferences with two key points:

* What's new ?
* What are the other Companies doing, that's working for them ?

## What new 

AMPLab, the creators of Spark from UC Berkley presented some interesting projects for the future that would increase SparkEngine to a new level. They presented some comparation charts with execution times of this new engine compared with project Flink which uses real streaming and not micro-batching, which may come as a very good news, because there are more and more Streaming applications being implemented. Also some new layor of secure, the projects are called Drizzle and Opaque, let's hope they get this ready soon :)

Also one of the big anouncements, was the new TensorFrames object. this output come's from a join work of TensorFlow and Databricks in order to bring DeepLearning to Spark. What this means is that in the Future one cloude simple use the Spark Framework in order to take advantage of GPU's, also that comes into alignment as you could expect with the new feature in Databricks Notebooks to support this.

It is still in Beta, but if you are using both TensorFlow and Spark, this is obviously good news.

IBM is also entering the race with a SaaS solution (https://console.ng.bluemix.net/catalog/services/apache-spark) similar to Databricks, in terms of Notebook offering but it runs on their own infrastructure, which could be good and have good stability and all, but when dealing with bigdata and moving all that data from provider to provider, you better to the maths.

For that matter still prefeer the Databricks model, and they have good news that until the end of the year they are planning to support a new CloudProvider.


## Use Cases

There where lot's of use cases, in the several industries, even in the [Milk Industry](https://spark-summit.org/eu-2016/events/mmmooogle-from-big-data-to-decisions-for-dairy-cows/) that's right :)

But looking on the overall presented cases, is clear that more and more companies are adopting Streaming solutions for they business, as [Jacek Laskowski](https://youtu.be/mVP9sZ6K__Y) refered in his presentation. (Really recommend that one, a really appealing one)

Kafka is becoming a key-role figure in the several presented architectures.

And it seems HDFS is becoming mostly used as a DataLake only solution, and starting using more high-speed memory layer as DataGrid, Aluxio solutions.

They also presented some comparations regarding direct access to Cloud Blob Storage compared with tradicional FS [Steve Loughran](https://spark-summit.org/eu-2016/events/spark-and-object-stores-what-you-need-to-know/). Interesting values presented regarding Microsoft Cloud Solution vs AWS.

On other aspect that call my attention, was the lack of monitoring solutions for Spark workloads, and there where in fact several presentations regarding this subject, mostly use cases of in-house developments that fill this gap. [Simon Whitear](https://spark-summit.org/eu-2016/events/sparklint-a-tool-for-monitoring-identifying-and-tuning-inefficient-spark-jobs-across-your-cluster/) 
was one of the best in my opinion.

Lambda,Gamma and Omega. With all these Streaming high-speed architectures showing on, one has to play it safe rigth. That's also one of the tendencies for the Industry to have these types of archs with a speed layer and slow-layer. [William Benton](https://spark-summit.org/eu-2016/speakers/william-benton/) made a very enjoyable presentation regarding this topic.

# Slides and Videos

It seems they updated the site and the videos and slides seem available, which is good news. 

Check them if you like on the site Schedule page:

https://spark-summit.org/eu-2016/schedule/


Cheers and keep Sparking