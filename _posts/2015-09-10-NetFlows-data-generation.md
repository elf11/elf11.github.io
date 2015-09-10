---
layout: post
title: NetFlow data generation with nfdump and softflowd
description: "how to get NetFlow data from your local network"
<!-- modified: 2015-09-10 -->
tags: [netflow, data, netflow generator, nfdump, softflowd]
image:
  feature: abstract-8.jpg

comments: true
share: true
---

Recently I needed some NetFlow data samples, I've looked all over the internet for some of those, but for obvious privacy reasons there were none. No one shares their NetFlow data. Not even a little sample. So what could I do, I had no Cisco equipments to generate traffic on and then to collect it in data flows. So I've improvised by using my laptop as a router in the campus network and collecting the traffic that went through it in data flows. This post is about how to generate and collect Netflow data on your own network.

## What is NetFlow?

Routers and switches have some probe configured, this looks at the packets that go through the equipment and its interfaces and collects those packets into flows. For example, for TCP flows are made out of all the packets from a connection. Of course those flows have some particularities, like they could be terminated after some idle time (usually 15 seconds), or even though if it is a continuous stream of data they are terminated after 30 mins and a new flow is started. Those parameters can usually be set up in the Cisco iOS interface. For UDP packets, a flow is all the packets with the same source and destination address, as well as source and destination ports. Of course it is an over simplification of the problem. When the flows are terminated or completed they are exported to a collector, the collector is the one that uses the NetFlow protocol.

What a collector does is to save all the data sent to him from the probe and write it in files on a disk, it also rotates files after awhile and deletes old capture files. NetFlow is a UDP protocol, so one of the things a NetFlow administrator will want to do is monitor how many UDP packets are being dropped by the collector; this can be in indication that the collector host is either not fast enough or that the collector process is not given a high enough scheduling priority (not something that we will concern ourselves right now).

NetFlow has many versions, but we are going to look at version 5. It is the default version configured on most Cisco routers, and it is perhaps the most common one. The format of NetFlow v5 can be seen [here](https://www.plixer.com/support/netflow_v5.html). There are 7 key fields, that must always be present in a data flow [source ip, destination ip, source port, destination port, layer 3 protocol, type of service, input logical interface], if we have those 7 key fields a data flow can be uniquely identified. There are also newer versions of the protocol, in particular version 9, which is more flexible when it comes to key fields and non-key fields, and it also added support for IPv6 and MPLS.


## softflowd and nfdump

The NetFlow architecture can be implemented using different software for probes, collectors and analysers. We are only focusing on the probing and collecting part. The available choices for collectors are much more variate than those for probes, since probes usually come implemented in the operating system of the routers (Cisco IOS). Some of the available probes are pmacct, nProbe (paid), softflowd. We are going to use softflowd, it is pretty easy to use and it also has support for NetFlow v9.

For the collector part I chose nfdump, another versatile tool which still has support from developers. nfdump has some rather useful querying facilities, including a filter mechanism similar to tcpdump, that supports many of the ad-hoc queries we may wish to make, including Top-N style questions. Therefore, we shall use nfdump for some of our reporting and querying examples as well as our ‘collector’ component.

### softflowd - install, configure and test the probe

The probe is usually part of the operating system of the router, since I had no access to a Cisco router, I used my computer as a router and installed the probe on it. A probe is either on a router, switch or on a "mirror" port on which the traffic from the router/switch is sent to. 

{% highlight bash %}
# apt-get install softflowd
{% endhighlight %}

The software doesn't run automatically, we have to configure it to listen on a particular interface. It is based on the same software as tcpdump (libpcap). To configure it edit /etc/default/softflowd file and define the INTERFACE and OPTIONS variables as follows:

{% highlight bash %}
INTERFACES = "any"
OPTIONS = "-n 127.0.0.1:9995"
{% endhighlight %}

This means the probe is listening on port 9995 of the localhost. To start the software use:

{% highlight bash %}
# /etc/init.d/softflowd start
{% endhighlight %}

To check if it started sending data we can use tcpdump, after we run the following command we have to wait for awhile for some data to expire and be send but that is usually pretty fast.

{% highlight bash %}
# tcpdump -n -i lo port 9995
{% endhighlight %}

If you don't want to wait for the flows to expire, you may force them to by running this:

{% highlight bash %}
# softflowd -c /var/run/softflowd.ctl expire-all
{% endhighlight %}

### nfdump - install and test the collector

The nfdump package is a suite of tools, one of which is nfcapd, which is the collector, and nfdump which is the display and analysis program. There are some other tools included as well, but those are the major commands we need to know about. To install nfdump run:

{% highlight bash %}
# apt-get install nfdump
{% endhighlight %}

The nfcapd - the collector should be started on the same port as the one on which softflowd is running. To do that run:

{% highlight bash %}
# nfcapd -D -l /var/cache/nfdump
{% endhighlight %}

To check if the collector is running on the intended port run:

{% highlight bash %}
# lsof -Pni | grep nfcapd
{% endhighlight %}

We should wait awhile for some data to be produced, but to see what that data looks like we could run the following command:

{% highlight bash %}
# nfdump -R /var/cache/nfdump/ | head -5
Date first seen          Duration Proto      Src IP Addr:Port          Dst IP Addr:Port   Packets    Bytes Flows
2015-09-10 18:45:05.243     0.000 UDP     158.121.23.138:59026 ->   158.121.23.255:8083         1       49     1
2015-09-10 18:45:08.107     0.000 UDP      158.121.22.94:68    ->  255.255.255.255:67           2      656     1
2015-09-10 18:42:29.532   161.807 UDP       158.121.23.0:68    ->  255.255.255.255:67           3      984     1
2015-09-10 18:45:08.458     4.103 UDP      158.121.22.94:137   ->   158.121.23.255:137          4      312     1
{% endhighlight %}

## Conclusion

Using softflowd and nfdump we can generate our own NetFlow data and use it to analyse the traffic in our network, or take out statistics about how much of the bandwidth it is used by each service.
