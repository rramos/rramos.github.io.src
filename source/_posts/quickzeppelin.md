---
title: Quick Setup Zeppelin Notebook
tags:
  - Zeepelin
  - Scala
  - Docker
  - Spark
date: 2017-09-19 22:06:18
---



In this article i describe a quick way to have zeepelin running so that you could quickly testing some Spark application.

**NOTE:** This procedure shouldn't be used in production environments has you should setup the Notebook with auth and connected to your local infrastructure.

## Requirements

* One should have a docker environment setup. Check my previous {% post_link dockerclean article %} if you need some help with that
* [Docker-compose](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-14-04)

## Setup

1. Create a folder named zeepelin
```
mkdir docker-zeepelin
```

2. Create a `data` where you could put some data to analyse.
```
mkdir -p docker-zeepelin/data
```

3. Create the following `docker-compose.yml` file in dir `docker-zeepelin` :

```
version: '2'
services:
  zeppelin:
    ports:
     - "8080:8080"
    volumes:
     - ./data:/opt/data
    image: "dylanmei/zeppelin"

```

4. Launch docker-compose
```
sudo docker-compose up -d
```

5. That's it you should now be able to access http://localhost:8080 


## Test it

1. Lets download a demo file to our `data` dir. 
```
curl -s https://api.opendota.com/api/publicMatches -o ./data/OpenDotaPublic.json
```

Yeah! I kinda like Dota so this makes sense :D

2. Create a new NoteBook in the web Interface and use the following code

```
%spark

val df = sqlContext.read.json("file:///opt/data/OpenDotaPublic.json")
df.show
```

Hit: **Shift-Enter**

3. Let's register this dataframe as temp table and create some visuals

```
%spark
df.registerTempTable("publicmatches")
```

4. Create the following to generate visualizations
```
%sql
select radiant_win,match_id
 from publicmatches
```


{% asset_img radiant_win.png [PieChart] %}

Guess i need to start playing on Radiant side :D


Well and that's it.

Cheers,
RR

# References

* https://zeppelin.apache.org/docs/0.5.5-incubating/tutorial/tutorial.html
* https://hortonworks.com/tutorial/getting-started-with-apache-zeppelin/
* https://www.digitalocean.com/community/tutorials/how-to-install-docker-compose-on-ubuntu-16-04
* https://docs.docker.com/compose
* https://zeppelin.apache.org/