---
title: Nifi Parcel for CDH
tags:
  - Cloudera
  - Nifi
  - CDH Parcel
  - Hadoop
date: 2017-09-29 02:01:48
---


I've been using [Apache Nifi](https://nifi.apache.org/) for some time now. But has a Cloudera user it would be nice to have it managed centralized in Cloudera Manager.

I found this [repo]() which had only one commit for doing exactly this, so i decided to give it a try, to see if it worked.

# Requirements

* Virtual machine with Cloudera quickstart pre-installed
* I had to patch the available Nifi version with [NIFI-3466](https://github.com/apache/nifi/pull/1604)
* And parcel repo also with [Issue1](https://github.com/prateek/nifi-parcel/issues/1)

So i'll be using my own forks that have this changes allready, but will be following the author (prateek) instructions.

# Instructions 

## Install Requirements

* cloudera/cm_ext

```sh
cd /tmp
git clone https://github.com/cloudera/cm_ext
cd cm_ext/validator
mvn install
```

* Create parcel & CSD:

```sh
$ cd /tmp
$ git clone git@github.com:rramos/nifi.git
$ cd nifi
$ mvn clean install
$ cd /tmp
$ git clone git@github.com:rramos/nifi-parcel.git
$ cd nifi-parcel
$ POINT_VERSION=5 VALIDATOR_DIR=/tmp/cm_ext ./build-parcel.sh /tmp/nifi/nifi-assembly/target/nifi-*-SNAPSHOT-bin.tar.gz
$ VALIDATOR_DIR=/tmp/cm_ext ./build-csd.sh
```

# Test the Parcel

Cloudera quickstart images comes with java version 1.7. But i need version 1.8 for Nifi otherwise i'll get minor major version missmatch issue. So i had to make some changes.

* First start your cloudera quickstart VM and copy the folders:
    * `build-csd` 
    * `build-parcel` 
    
there in my case i had IP `192.168.122.132` (change for your case).

```sh
scp -r build-parcel cloudera@192.168.122.132:/tmp
scp -r build-csd cloudera@192.168.122.132:/tmp
```

* SSH in the machine and install java8

```sh
yum install java-1.8.0-openjdk
```

* Run the following command in `build-parcel` folder

```sh
cd build-parcel
python -m  SimpleHTTPServer 14641
```

Now open your browser at `http://192.168.122.132:7180` and access Cloudera Manager (cloudera:cloudera).

Navigate to -> Parcels -> Edit Settings. 

Add `http://192.168.122.132:14641` 

{% asset_img parcel01.png [NIFI Parcel] %}

Download and install the Nifi Parcel.

* Copy NIFI home

```sh
cp -r /tmp/build-parcel/NIFI-0.0.5.nifi.p0.5 /opt/cloudera/parcels/NIFI/
```

* Correct java path in file `/opt/cloudera/parcels/NIFI/bin/nifi-env.sh`

```sh
# The java implementation to use.
export JAVA_HOME=/usr/lib/jvm/jre-1.8.0
```

* Copy the csd jars 

```sh
cp /tmp/build-csd/NIFI-1.2.0.jar /opt/cloudera/csd/
mkdir /opt/cloudera/csd/NIFI-1.2.0
cp /tmp/build-csd/NIFI-1.2.0.jar /opt/cloudera/csd/NIFI-1.2.0/
cd /opt/cloudera/csd/NIFI-1.2.0/
jar xvf NIFI-1.2.0.jar
rm -f NIFI-1.2.0.jar
```

* Correct ownership

```
chown -R cloudera-scm:cloudera-scm /opt/cloudera
```

* Apply Configuration changes to Cloudera Manager and restart the service

* Move CSD to Cloudera Manager's CSD Repo
 
```sh
sudo service cloudera-scm-server restart
```

* After login again in CM wait for zookeeper to be running normal, and add a new service (Nifi)

{% asset_img parcel02.png [Add NIFI Service] %}

After terminating the wizard you should have access to Nifi Interface


{% asset_img parcel03.png [Nifi Web UI] %}

 
# Conclusion

I must say that prateek repo with only that commit worked pretty well and the instructions where also clear. There where some minor adjustments because of the java version, but we can start/stop the service via Cloudera Manager.

There are some pending items refered in his github page:
* Currently `NiFi` runs under the `root` user
* Expose config options under Cloudera Manager
* Conf folder from parcels is used, this needs to be migrated to ConfigWriter
* Expose metrics from NiFi

I haven't tested the configuration in cluster mode as i has using the quickstart VM.

The configuration options in CM would be very good improvement. Auto configuring zookeeper and the other nifi options. I'll try to contribute to prateek excelent work if i manage to get some time.

Cheers,
RR

# References

* https://nifi.apache.org
* https://github.com/prateek/nifi-parcel
* https://github.com/cloudera/cm_ext
* https://www.cloudera.com/downloads/quickstart_vms/5-12.html
