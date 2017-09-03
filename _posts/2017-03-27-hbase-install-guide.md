---
layout: post
title: HBase Installation Guide
description: "how to install HBase on a cluster"
<!-- modified: 2015-09-10 -->
tags: [hbase, big data, hadoop, cluster]

comments: true
share: true
---

As we have discussed in the previous post for my research we needed an open source solution that could deal with Big Data, we stopped on the Hadoop Ecosystem and we went through the componets of HDFS and MapReduce as well as through the steps needed to install Hadoop on a cluster. Now we are going to discuss about another component of the Hadoop Ecosystem, HBase. Accessing data in the HDFS system is slow, so we needed a way to be able to get to the particular data we were interested in - in the TBs of data we had saved on the cluster - in a fast manner. HBase allows random access to the data in a convenient time, being a NoSQL database.

## What is a HBase?

Apache HBase is a NoSQL database that runs on top of Hadoop as a distributed and scalable big data store. This means that HBase can leverage the distributed processing paradigm of the Hadoop Distributed File System (HDFS) and benefit from Hadoop’s MapReduce programming model. It is meant to host large tables with billions of rows with potentially millions of columns. HBase allows you to query for individual records as well as derive aggregate analytic reports across a massive amount of data. So in short, HBase:

1. Stores key/value pairs in columnar fashion (columns are clubbed together as column families).
2. Provides low latency access to small amounts of data from within a large data set.
3. Provides flexible data model.


## HBase architectural components

HBase is composed of three types of servers in a master slave type of architecture. Region servers serve data for reads and writes. When accessing data, clients communicate with HBase RegionServers directly. Region assignment, create, delete tables operations are handled by the HBase Master process. Zookeeper, which is part of HDFS, maintains a live cluster state.
The Hadoop DataNode stores the data that the Region Server is managing. All HBase data is stored in HDFS files. Region Servers are collocated with the HDFS DataNodes, which enable data locality, putting the data close to where it is needed, for the data served by the RegionServers. HBase data is local when it is written, but when a region is moved, it is not local until compaction.
The NameNode maintains metadata information for all the physical data blocks that comprise the files.

So the 3 types of serves that can be found in HBase are:
1. RegionServer
2. HBase Master
3. Zookeeper

### How do those components work together?

Each Region Server creates an ephemeral node. The HMaster monitors these nodes to discover available region servers, and it also monitors these nodes for server failures. HMasters vie to create an ephemeral node. Zookeeper determines the first one and uses it to make sure that only one master is active. The active HMaster sends heartbeats to Zookeeper, and the inactive HMaster listens for notifications of the active HMaster failure.
If a region server or the active HMaster fails to send a heartbeat, the session is expired and the corresponding ephemeral node is deleted. Listeners for updates will be notified of the deleted nodes. The active HMaster listens for region servers, and will recover region servers on failure. The Inactive HMaster listens for active HMaster failure, and if an active HMaster fails, the inactive HMaster becomes active.

## Setting up HBase on top of a HDFS cluster

HBase is usually installed on top of a HDFS cluster and needs a couple of prerequisites before it can fully function. Since we are going to install HBase on top of the HDFS cluster set up in the last post most of those prerequisites are already met, but we still have to make sure that everything works properly.

### Prerequisites

1. a running HDFS cluster
2. install JAVA, since we have a running HDFS cluster we already have JAVA installed
3. passwordless login, we have already set this up from the master to the slaves when we set up the HDFS cluster
4. DNS resolution, we have to check that both forward and reverse DNS resolution is possible for all hostnames, we have set up the /etc/hosts file to help us with this when we set up the HDFS cluster

### Setting up HBase

We are going to use the same cluster as before, with 1 master and 2 slaves, as a reminder the IP addresses that we are going to use are:

{% highlight bash %}
NameNode        10.2.0.10
DataNode1        10.2.0.1
DataNode2        10.2.0.2
{% endhighlight %}

The NameNode is going to be our HMaster and the DataNodes are going to be the RegionServers.

1. download a stable release of HBase from the [link][download] and move it to "/usr/local" under the "hbase" folder name.

2. edit the /usr/local/hbase/conf/hbase-env.sh file
{% highlight bash %}
root@NameNode:~# vim /usr/local/hbase/conf/hbase-env.sh
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64 ----> check the java path in your system.
export HBASE_MANAGES_ZK=false
{% endhighlight %}

3. edit /usr/local/hbase/conf/hbase-site.xml

{% highlight bash %}
root@NameNode:~# vim /usr/local/hbase/conf/hbase-site.xml
<configuration>

 <property>
 <name>hbase.master</name>
 <value>10.2.0.10:60000</value>
 </property>

<property>
<name>hbase.cluster.distributed</name>
<value>true</value>
</property>

<property>
<name>hbase.rootdir</name>
<value>hdfs://10.2.0.10:9000/hbase</value>
 </property>

 <property>
 <name>hbase.zookeeper.quorum</name>
 <value>DataNode1,DataNode2</value>
 </property>

</configuration>
{% endhighlight %}

The hbase.zookeeper.quorum property tells HBASE, which will be the ZOOKEEPER servers in the cluster.

4. edit /usr/local/hbase/conf/regionservers and add the IP addresses of the regionservers to the file

{% highlight bash %}
root@NameNode:~# vim /usr/local/hbase/conf/regionservers
# It should be something like this
10.2.0.1
10.2.0.2
{% endhighlight %}

5. now, before we can start the HBase application we have to copy all those files we have modified to the regionservers

{% highlight bash %}
root@NameNode:~# scp -r /usr/local/hbase root@DataNode1:/usr/local
root@NameNode:~# scp -r /usr/local/hbase root@DataNode2:/usr/local
{% endhighlight %}

6. to start the HBase application

{% highlight bash %}
root@NameNode:~# /usr/local/hbase/bin/start-hbase.sh
{% endhighlight %}


[download]:https://archive.apache.org/dist/hbase/
    "Download a stable release of HBase"
