---
title: Clean all Dockers and configure with btrfs
tags:
  - Docker
  - Linux
  - btrfs
date: 2017-09-19 08:37:29
---


So dockers tend to start growing like weeds. And cost me space i don't have. But one could change the storage driver. You could choose LVM and take advantage of the snapshots or btrfs which i allready have.

## Btrfs  

Btrfs is a next generation copy-on-write filesystem that supports many advanced storage technologies that make it a good fit for Docker. Btrfs is included in the mainline Linux kernel.

Docker’s btrfs storage driver leverages many Btrfs features for image and container management. Among these features are block-level operations, thin provisioning, copy-on-write snapshots, and ease of administration. You can easily combine multiple physical block devices into a single Btrfs filesystem.

## Requirements

* Install btrfs 
* Make sure you have a volume formated as btrfs (not part of the article)

## Clean all dockers and images

```
# Delete all containers
sudo docker rm $(sudo docker ps -a -q)
# Delete all images
sudo docker rmi $(sudo docker images -q)
```


## Setup

Make sure you have btrfs on your system

```
sudo cat /proc/filesystems | grep btrfs
```

1. Stop docker

```
sudo service docker stop
```

2. Create a backup of you docker settings

```
sudo mv /var/lib/docker/ /var/lib/docker.bak
```

3. create a subvolume from an existing btrfs FS

```
btrfs subvolume create /archive/dockers
```

**NOTE:** It's better to use a dedicate disk for it

4. Create the symlink

```
ln -s /archive/dockers /var/lib/docker
```

5. Copy the backup data to the new location

```
cp -rp /var/lib/docker.bak/* /var/lib/docker/
```

6. Configure Docker to use the btrfs storage driver. This is required even though `/var/lib/docker/` is now using a Btrfs filesystem. Edit or create the file `/etc/docker/daemon.json`.

```
{
  "storage-driver": "btrfs"
}
```

7. Start docker service

```
sudo service docker start
```

8. Check if docker is running with docker support

```
sudo docker info|grep  "Storage Driver"
```

You could check the volumes being created with the command

```
sudo btrfs subvolume list /var/liv/docker |grep dockers
```


## References

* https://docs.docker.com/engine/userguide/storagedriver/btrfs-driver/

