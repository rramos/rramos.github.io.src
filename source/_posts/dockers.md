---
title: Dockers
tags:
  - Docker
  - docker-compose
  - utils
date: 2017-10-18 01:26:17
---


This article is just to share a list of usefull dockers, in case one wants to speed test some feature and required some of this software to be up and running.

# Why ?

The main ideia is that i can fireup quickly without having to worry on the ports that are allready mapped on my local machine, so the several dockers should have colision ports taken care of.

One should just need to execute `docker-compose up` and that's it.

You could also use other repo, but i'll try to make my local configurations in a way that i could interuse several dockers.

# Dockers repo

You just need to clone the repo https://github.com/rramos/dockers

```
git clone https://github.com/rramos/dockers.git
```


# Requirements

Make sure you have docker and docker compose properly configured

# Setup

Just fire-up `docker-compose` and that's it 

```
cd docker-portainer
docker-compose up -d
```

Each docker has it's own README file with more instructions.

# Docker List

* docker-grafana  
* docker-hdp      
* docker-ksql    
* docker-nifi         
* docker-portainer  
* docker-zeppelin  
* docker-hbase    
* docker-jenkins  
* docker-moodle  
* docker-openproject  
* docker-rundeck 

This is the available list at the moment of this article, i will add up more as i'm using and testing new stuff.

**NOTE:** Use this containers at your own risk 


Cheers,
RR
