---
layout: post
title: Analyse a tcpdump capture using libpcap in C
description: "how to analyze a tcpdump capture using lipcap and C programming"
<!-- modified: 2015-09-10 -->
tags: [c, programming, tcpdump, libpcap, pcap, analysis]

comments: true
share: true
---

In the past I have taken some security courses, and during one of them we had as assignment to use lipcap to sniff and spoof the DNS inside a network. That gave me an idea for this article, more like a gentle introduction to libpcap and how to use it to analyze the type and number of packets you are getting in the network at a particular moment. This could be done very easily with Wireshark and a series of filters, but the purpose of this article is an educational one and a basis for understanding and developing greater and better things with libpcap.

So what will you need for this is a Wireshark capture of the traffic, if you don't want to capture anything at this moment for the purpose of this tutorial you can download a pcap file [here][pcapfile], and some basic to medium C programming knowledge.

Before we start we should define two terms that might not be familiar to everyone, even though if you got to this tutorial you should be pretty familiar with both of them:

1. Packet Capture means to grab a copy of packets off of the wire before they are processed by the operating system. The capture can be used in network security tools to analyze raw traffic and detect malicious behaviours, networks scans or attacks, for sniffing, fingerprinting and other purposes.
2. libpcap is the library that we are going to use to grab those network packets right as they come off of the network card.

## Intro - how to get libpcap and how to compile with it

Compiling a pcap program requires linking with the pcap library. You can install it in Debian based distributions like this:

{% highlight bash %}
sudo apt-get install libpcap0.8-dev
{% endhighlight %}

If you already have the library installed then you can compile programs in the following way:

{% highlight bash %}
gcc <source-file> -lpcap
{% endhighlight %}

## Offline packet processing with libpcap

What we are going to do in this tutorial is open a pcap file captured with Wireshark, or other capture software that exports files in pcap format, and do a short analysis of the packets that have been captured, number of HTTP packets, number ICMP packets, number of UDP packets - or as I call it a protocol summary, number of DNS packets seen in the network, the IP address that sends the most SYN packets - this could be a sign of an attacker in the network if there are too many SYN from this address for which it doesn't wait for an ACK, and the IP address to which the most HTTP/HTTPS traffic goes to. The HTTP/HTTPS traffic will be analysed looking at the bandwidth/bytes and not number of packets.

So now if we look through the source of the [stats.c][source] file we can figure out how are we going to do this.

{% highlight C %}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include <netinet/in.h>
#include <netinet/ip.h>
#include <net/if.h>
#include <netinet/if_ether.h>
#include <net/ethernet.h>
#include <netinet/tcp.h>
#include <netinet/udp.h>
#include <arpa/inet.h>


#include <pcap.h>

#define PCAP_BUF_SIZE	1024
#define PCAP_SRC_FILE	2

int icmpCount = 0;
int tcpCount = 0;
int udpCount = 0;
int dnsCount = 0;
int synCount[PCAP_BUF_SIZE];
int synIdx = 0;
char synIP[PCAP_BUF_SIZE][INET_ADDRSTRLEN];
int httpCount[PCAP_BUF_SIZE];
int httpIdx = 0;
char httpIP[PCAP_BUF_SIZE][INET_ADDRSTRLEN];

void packetHandler(u_char *userData, const struct pcap_pkthdr* pkthdr, const u_char* packet);
{% endhighlight %}

The first lines are the includes necessary for reading a pcap file, the ones coming from netinet and net packages are used for parsing and transforming data found in packets. Then we have a series of global variable declarations where we are going to save the number of packets for each of the categories we enumerated above.

{% highlight C %}
int main(int argc, char **argv) {

    pcap_t *fp;
    char errbuf[PCAP_ERRBUF_SIZE];
    char source[PCAP_BUF_SIZE];
    int i, maxCountSyn = 0, maxCountHttp = 0, maxIdxSyn = 0, maxIdxHttp = 0;

    if(argc != 2) {
        printf("usage: %s filename\n", argv[0]);
        return -1;
    }

=
    fp = pcap_open_offline(argv[1], errbuf);
    if (fp == NULL) {
	    fprintf(stderr, "\npcap_open_offline() failed: %s\n", errbuf);
	    return 0;
    }
{% endhighlight %}

After entering the main execution, we check if the number of arguments is what we are expecting, we expect to get the pcap file name from the standard input, and tehn we open our target packet. To do this we use pcap_open_offline() and give it the capture filename and an error buffer as parameters. If all goes well, we get a pcap_t descriptor returned. If not, check the error buffer for details.

{% highlight C %}
	if (pcap_loop(fp, 0, packetHandler, NULL) < 0) {
        fprintf(stderr, "\npcap_loop() failed: %s\n", pcap_geterr(fp));
        return 0;
    }

    for (i = 0; i < synIdx; i++) {
        if (maxCountSyn < synCount[i]) {
            maxCountSyn = synCount[i];
            maxIdxSyn = i;
        }
    }

    for (i = 0; i < httpIdx; i++) {
        if (maxCountHttp < httpCount[i]) {
            maxCountHttp = httpCount[i];
            maxIdxHttp = i;
        }
    }

    printf("Protocol Summary: %d ICMP packets, %d TCP packets, %d UDP packets\n", icmpCount, tcpCount, udpCount);
    printf("DNS Summary: %d packets.\n", dnsCount);
    printf("IP address sending most SYN packets: %s\n", synIP[maxIdxSyn]);
    printf("IP address that most HTTP/HTTPS traffic goes to (in terms of bandwidth, NOT packet count): %s\n", httpIP[maxIdxHttp]);
    return 0;
{% endhighlight %}

We use pcap_loop() to set up a handler callback for each packet in our capture to be processed. The handler function needs a file descriptor - that we just created with pcap_open_offline(), the number of packets we want to process - we will use 0 to indicate that there is no limit to it, the callback function name - in our case is packetHandler, userdata - in our case it will be NULL because we will pass no uder defined data to the callback function.

The pcap_loop() function returns when the entire capture file has been processed, at that point our counters have been already updated and we can printf the statistics that we are interested in.

{% highlight C %}
void packetHandler(u_char *userData, const struct pcap_pkthdr* pkthdr, const u_char* packet) {

    const struct ether_header* ethernetHeader;
    const struct ip* ipHeader;
    const struct tcphdr* tcpHeader;
    const struct udphdr* udpHeader;
    char sourceIP[INET_ADDRSTRLEN];
    char destIP[INET_ADDRSTRLEN];
    u_int sourcePort, destPort;
    u_char *data;
    int dataLength = 0;
    int i;

    ethernetHeader = (struct ether_header*)packet;
    if (ntohs(ethernetHeader->ether_type) == ETHERTYPE_IP) {

        ipHeader = (struct ip*)(packet + sizeof(struct ether_header));
        inet_ntop(AF_INET, &(ipHeader->ip_src), sourceIP, INET_ADDRSTRLEN);
        inet_ntop(AF_INET, &(ipHeader->ip_dst), destIP, INET_ADDRSTRLEN);

        if (ipHeader->ip_p == IPPROTO_TCP) {
            tcpCount = tcpCount + 1;
            tcpHeader = (struct tcphdr*)(packet + sizeof(struct ether_header) + sizeof(struct ip));
            sourcePort = ntohs(tcpHeader->source);
            destPort = ntohs(tcpHeader->dest);
            data = (u_char*)(packet + sizeof(struct ether_header) + sizeof(struct ip) + sizeof(struct tcphdr));
            dataLength = pkthdr->len - (sizeof(struct ether_header) + sizeof(struct ip) + sizeof(struct tcphdr));
            if (sourcePort == 80 || sourcePort == 443 || destPort == 80 || destPort == 443) {
                for (i = 0; i < httpIdx; i++) {
                    if (strcmp(destIP, httpIP[i]) == 0) {
                        httpCount[i] = httpCount[i] + dataLength;
                    }
                }
                strcpy(httpIP[httpIdx], destIP);
                httpCount[httpIdx] = dataLength;
                httpIdx = httpIdx + 1;
            }

            if (tcpHeader->th_flags & TH_SYN) {
                for (i = 0; i < synIdx; i++) {
                    if (strcmp(sourceIP, synIP[i]) == 0) {
                        synCount[i] = synCount[i] + 1;
                    }
                }
                strcpy(synIP[synIdx], sourceIP);
                synCount[synIdx] = 1;
                synIdx = synIdx + 1;
            }

        } else if (ipHeader->ip_p == IPPROTO_UDP) {
            udpCount = udpCount + 1;
            udpHeader = (struct udphdr*)(packet + sizeof(struct ether_header) + sizeof(struct ip));
            sourcePort = ntohs(udpHeader->source);
            destPort = ntohs(udpHeader->dest);
            if (sourcePort == 53 || destPort == 53) {
                dnsCount = dnsCount + 1;

            }
        } else if (ipHeader->ip_p == IPPROTO_ICMP) {
            icmpCount = icmpCount + 1;
        }
    }
}
{% endhighlight %}


In the packet handler function we are parsing the ethernet header from the packet and using its type to determine if it is an IP packet or not. We use the ntohs() to convert the type from network byte order to host byte order. If it is an IP packet, we parse out the IP header and use the inet_ntop() function to convert the IP addresses found in the IP header into a human readable format (i.e., xxx.xxx.xxx.xxx). We check if the tcpheader has the syn flag on and if so then we update a counter. We update the http counters as well if the source port or destination port of the packets is either 80 or 443.

If the packet is not a TCP packet then we check if it's an UDP packet and check if the port is 53 for DNS service, if so then we update the counter for that.

If it's not either a TCP or UDP packet we check if it's an ICMP packet and update the corresponding counter.

## Conclusion

You can download the C program in its entire form [here][source] and try it out. Happy Hacking! :) 


[pcapfile]: /files/capture.pcap
    "Download a stable release of HBase"
[source]: /files/stats.c
	"Download the source for statistics"
