---
title: Portainer a UI for docker management
tags:
  - Docker
  - Portainer
date: 2017-09-23 17:22:37
---


[Portainer](https://portainer.io/) is:

{% blockquote Offical Website Definition%}

PORTAINER IS AN OPEN-SOURCE LIGHTWEIGHT MANAGEMENT UI WHICH ALLOWS YOU TO EASILY MANAGE YOUR DOCKER HOSTS OR SWARM CLUSTERS
{% endblockquote %}

In order to manage the dockers where it is running one should pass the following option

```
sudo docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer
```


## Snapshot

{% asset_img portainer.io.png [Portainer.io] %}

# Setup

Create the following `docker-compose.yml` file 

```
version: '2'
services:
  portainer:
    restart: always
    ports:
     - "9000:9000"
    volumes:
     - ./data:/opt/data
     - /var/run/docker.sock:/var/run/docker.sock
    image: "portainer/portainer"

```

and execute `docker-compose up -d`

You can then access the interface at: http://localhost:9000


Cheers,
RR