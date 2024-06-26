---
layout: default
permalink: /ns/03-network-performance
title: Network Performance 
description: Various factors affecting network performance
parent: Network Security
nav_order: 3
---


* TOC
{:toc}

**50.005 Computer System Engineering**
<br>
Information Systems Technology and Design
<br>
Singapore University of Technology and Design
<br>
**Natalie Agus (Summer 2024)**

# Network Performance 
{: .no_toc}

{:.highlight-title}
> Detailed Learning Objectives
>- **Understand Network Performance Metrics**
>  - Grasp the concept of network performance including efficiency and reliability in data transmission.
>  - Identify the factors that impact network performance such as latency and throughput.
>- **Learn about Sources of Packet Delay**
>  - Describe the four main types of packet delays: processing, queueing, transmission, and propagation.
>  - Understand how each type of delay affects the overall network performance.
>- **Explore Tools for Measuring Network Delays**
>  - Gain familiarity with network diagnostic tools like traceroute and their functionalities.
>  - Learn how traceroute uses ICMP to map the route data packets take across the network.
>- **Understand ICMP and its Role in Network Diagnostics**
>  - Comprehend how ICMP works and its significance in error messaging and network diagnostics.
>  - Analyze the practical application of ICMP in tools like traceroute and ping.
>- **Analyze Packet Loss and Its Implications**
>  - Identify the causes and consequences of packet loss within a network.
>  - Learn how to compute end-to-end loss rate probability based on per-link loss rates.
>- **Distinguish Between Throughput and Bandwidth**
>  - Define and differentiate between throughput and bandwidth in network contexts.
>  - Explore the calculations and factors affecting throughput, including network topology and link technologies.
>- **Visualize Network Performance**
>  - Understand the use of time-space diagrams in visualizing network data flow and performance.
>  - Analyze how changes in network parameters like bandwidth and packet size affect the data transmission represented in time-space diagrams.
>
> These learning objectives are designed to provide a comprehensive understanding of network performance, including how to measure, analyze, and optimize the efficiency and reliability of data transmission across networks.

{:.info}
Network performance refers to the **efficiency** and **reliability** of data transmission across a network. Packet delays, which can stem from factors such as network congestion, routing inefficiencies, and packet processing delays, **directly impact** the overall performance of a network by influencing latency and throughput. 

When packets arrive at network nodes (i.e. router) they <span class="orange-bold">queue</span> in the router's local buffers, waiting for their turn to be dispatched towards the destination host. 

{:.note}
An end host has a complete network stack, whereas a node can have partial network stack. 

There are four sources of <span class="orange-bold">packet delay</span> that affects network performance: 

<img src="{{ site.baseurl }}//docs/NS/images/03-network-performance/2024-04-25-15-55-21.png"  class="center_seventy"/>

## Processing Delay 

<img src="{{ site.baseurl }}//docs/NS/images/03-network-performance/2024-04-25-16-03-32.png"  class="center_fifty"/>

**Duration**: < milliseconds, usually is bounded or fixed. 

**Cause**: Needs time to examine packet header.
* Check for bit errors (checksum), 
* Compute output link as denoted by destination IP address. 

{:.info}
Routers typically have multiple output ports, each leading to a different network or device. These output ports allow routers to forward data packets to the appropriate destination based on their routing tables and the destination IP addresses of the packets

## Queueing Delay

**Duration**: varies, depending on traffic (congestion level). It is <span class="orange-bold">non trivial</span> to compute. 

**Cause**: packet input rate > link output rate, therefore packet needs to wait in the node buffer to get to the front of the queue to reach the output link. 

<img src="{{ site.baseurl }}//docs/NS/images/03-network-performance/2024-04-25-16-05-05.png"  class="center_fifty"/>

## Transmission Delay 

**Duration**: varies, depends on bandwidth (link technology). The amount of transmission delay is computed as: $$d_{trans} = \frac{L}{R}$$, where $$L$$ is packet length in **bits** and $$R$$ is link bandwith in bits/s.

**Cause**: Need time to push the whole packet bits, e.g: 512 Bytes from the router to the link itself. 

<img src="{{ site.baseurl }}//docs/NS/images/03-network-performance/2024-04-25-16-06-38.png"  class="center_fifty"/>

## Propagation Delay
**Duration**: depends on the physical length of the link. The amount of propagation delay is computed as: $$d_{prop} = \frac{d}{S}$$, where $$d$$ is the link length in metres, and $$S$$ is the propagation speed of bits on medium, it can be approximated to $$2\times10^8$$ m/s.

## Total Nodal Delay 

The total nodal delay (from one node/router) to another node/router is the sum of all types of delays: $$d_{nodal} = d_{proc} + d_{queue} + d_{trans} + d_{prop}$$. 

## Average Link Utilisation 

The average link utilisation model is obtained from observation. The model below is assumed based on random and bursty packet arrivals: 

<img src="{{ site.baseurl }}//docs/NS/images/03-network-performance/2024-04-25-16-10-03.png"  class="center_fourty"/>

The average queueing delay is related to the average link utilization (traffic intensity). It approaches infinity as the average link utilization approaches 1. 

The formula for average link utilization is $$\frac{La}{R}$$ where $$L$$ is packet length in bits, $$a$$ is average packet arrival rate (packets/second), and $$R$$ is the link's bandwith in bits/s. 

{:.note-title}
> Analysis
>
> Queueing delay is a convex increasing  function of utilization:
> * If average link utilization 0, average queueing delay is small
> * If average link utilization is near 1, average queueing delay is large
> * If average link utilization > 1, average queueing delay is infinite (there’s more work arriving that can be serviced)

<span class="orange-bold">Key takeaway</span>: should not try to use every bit of the link capacity.

# Real Internet Delays and Routes

## Traceroute 

There exists multiple hops (on routers) before data from a source host arrives at the destination host. You can measure delay from source to each router along the Internet path towards destination using `traceroute` utility.

<img src="{{ site.baseurl }}//docs/NS/images/03-network-performance/2024-04-25-16-12-35.png"  class="center_seventy"/>


A sample output of `traceroute` to `google.com` is as follows:

<img src="{{ site.baseurl }}//docs/NS/images/03-network-performance/2024-04-25-16-14-23.png"  class="center_seventy no-invert"/>

Here we see that there are at least 10 **probes** in total (result truncated). 

### How traceroute works 

Traceroute is a network diagnostic tool used to trace the path that data packets take from one point to another on a network. raceroute relies on ICMP (Network Layer protocol) to work. 
{:.info}

It works by sending a series of packets, each forming a **probe** with an incrementally increasing time-to-live (TTL) value, towards the destination. The TTL value determines how many network hops (routers) the packet can pass through before being discarded. Each router <span class="orange-bold">decrements</span> the TTL value by one. When the TTL reaches zero, the router discards the packet and sends an ICMP "Time Exceeded" message back to the sender.

#### ICMP 

ICMP stands for Internet Control Message Protocol, a supporting protocol in the Internet Protocol Suite. 

It is a protocol for error messages and operational information (standard messages), mainly not used by the application itself but used by the network operators to identify errors, perform diagnostics, and generally are used for <span class="orange-bold">control</span> purposes.

It is not typically critical for normal operation, hence it is not usual for it to be <span class="orange-bold">disabled</span>, misconfigured, or buggy. This is why you see some `* * *` (no response, timeout) in the traceroute output. 

{:.info}
ICMP is actually implemented at the “top” of layer 3 (network layer). It relies on IP protocol to deliver data to a remote host. In other words, ICMP messages <span class="orange-bold">must</span> be encapsulated in IP packets. 

#### Probe 
At each probe *i*, traceroute sends 3 packets that will reach router *i* (means the router reached after i hops) on path towards destination. These packets are **ICMP Echo packets** with TTL (time to live) value of *i*. Each router in the path will decrease the TTL by  1. 

{:.info}
The final value of *i* is unknown in the beginning. It starts from 1 and it will be increased by 1 at each iteration. It's maximum is set to be certain number, e.g: 64 in example above. 

Router at hop *i* will be the last router in the path that decreases the TTL (TTL will reach 0 at this point), and will respond to the sender, the `traceroute` process, by sending <span class="orange-bold">ICMP TTL Exceeded packet</span> because it is <span class="orange-bold">not</span> the destination router. Remember tin the example above: the destination is `google.com`. 

The other routers at hop *<i* will simply decrement the TTL as explained above. 

#### RTT 
The `traceroute` process measures the time from packet sent until packet is received back from router *i* for each probe: RTT (round trip time). RTT depends on various factors: network infrastructure, distance between nodes, network conditions,  and packet size.

Traceroute listens for these ICMP messages, recording the IP addresses of the routers along the path. By repeatedly sending packets with increasing TTL values and recording the responding routers, traceroute builds a map of the network path to the destination.

#### Termination 
The traceroute program (in UNIX-like OS) sends packets to a likely unusable port in the destination, e.g: 50000. When the packet reaches the destination, the destination host finds out that the packet is destined for a port that is <span class="orange-bold">unusable</span>. 

The destination host will send back an <span class="orange-bold">ICMP Port Unreachable message</span>, where upon receiving this the traceroute program stops sending more packets.  

{:.info-title}
> Port 
> 
> A port is a 16 bits logical representation of a service or application in the host system. Upon receiving the incoming packet, the network management subsystem of your OS will try to forward the packet to the application that’s bound to the particular port. 

## Ping 

{:.info}
Ping is a another network utility used to test the reachability of a host on an Internet Protocol (IP) network and measure the round-trip time for messages sent from the originating host to a destination computer and back. It also relies on ICMP to work.  

**How it works**: it sends ICMP echo request / reply packets to the destination host. However, some hosts block ICMP echo requests for security purposes. 


<img src="{{ site.baseurl }}//docs/NS/images/03-network-performance/2024-04-25-16-28-29.png"  class="center_seventy"/>
no-invert

Here's a sample output of the `ping` utility: 
<img src="{{ site.baseurl }}//docs/NS/images/03-network-performance/2024-04-25-16-29-09.png"  class="center_seventy no-invert"/>

# Packet Loss 

<img src="{{ site.baseurl }}//docs/NS/images/03-network-performance/2024-04-25-16-29-44.png"  class="center_seventy"/>

Earlier we learned that packets will queue in the local buffer of a network node if packet arrival in the input link exceeds the output link capacity. 

{:.important}
Packets that arrive at a full queue (full buffer) will be <span class="orange-bold">dropped</span> (lost).  They may be retransmitted by the previous node, source system, or not at all. Loss is <span class="orange-bold">random</span>, where each link has an average loss rate or probability.

You can compute end-to-end loss rate probability given per-link loss rate probability. For example, given the following network topology and per-link loss rate: 

<img src="{{ site.baseurl }}//docs/NS/images/03-network-performance/2024-04-25-16-30-28.png"  class="center_fifty"/>

We can compute the end-to-end loss rate between A to C as: $$(1 - (0.8 - 0.9)) * 100\% = 28\%$$

# Throughput and Bandwidth

{:.info}
**Throughput** is defined as the rate (bits/time unit) at which bits are actually transferred between sender/receiver. **Bandwidth** is defined as the <span class="orange-bold">maximum</span> amount of data that can travel through a link. 

Throughput computation is <span class="orange-bold">application specific</span>, depending on factors such as latency, SNR (hardware limitations), etc. For example, at TCP level, the throughput depends on RTT (round trip time) and window size.

{:.warning}
Throughput is different from bandwidth. Bandwidth is the theoretical maximum capacity of a network link, while throughput is the actual observed data transfer rate under real-world conditions.

There are **two** ways to calculate throughput: instantaneous and averaged. The number seconds used for the measurement of throughput separates between the two computation:
* **Instantaneous** throughput: if the measurement is taken over a very short time interval
* **Average** throughput: if the measurement is taken over a long time interval, for example, the transfer a bunch of large files.

{:.note}
Average throughput is usually more consistent than instantaneous throughput. Average throughput represents the data transfer rate over a longer period, smoothing out fluctuations and providing a more stable indication of network performance compared to instantaneous throughput, which may vary significantly due to factors like congestion, bursty traffic, or network conditions.

Link bandwidth varies depending on its material/technology:
* T1 Line (1.5 Mbps)
* Fast Ethernet (100 Mb/s)
* T3 Line (43 Mbps)
* Gigabit Ethernet (1Gb/s)
* OC192 (9.95 Gb/s)

We can compute end-to-end throughput based on the <span class="orange-bold">bottleneck</span> link. For instance, given the following network topology and bandwidth: 

<img src="{{ site.baseurl }}//docs/NS/images/03-network-performance/2024-04-25-16-35-58.png"  class="center_fifty"/>

If Host A wish to send packets to host C, where the *throughput* of each link is as shown in the diagram above, Host A can only do so with throughput of 3 MBps.

{:.note}
Note that b stands for bit, while B stands for Byte.

{:.new-title}
> Bandwidth or throughput in calculations? 
> 
> It's common to use the term "bandwidth" when discussing network topology and link capacities in throughput computation questions. This is because in **theoretical** or **simplified** network scenarios, the bandwidth of a link represents its <span class="orange-bold">maximum</span> capacity, which is a key factor in determining the *potential throughput* of data transmission across that link.
> 
> Therefore, while "throughput" refers to the <span class="orange-bold">actual</span> observed data transfer rate, in theoretical scenarios or when designing networks, it's often assumed that the link operates at its maximum capacity (bandwidth) under ideal conditions. Therefore, using "bandwidth" in such contexts is appropriate for simplicity and clarity in calculations and network design discussions.


Now consider another network topology:

<img src="{{ site.baseurl }}//docs/NS/images/03-network-performance/2024-04-25-16-37-43.png"  class="center_fifty"/>

When a link is <span class="orange-bold">shared</span> among N connections, each connection may (unless otherwise stated) get an equal share of the transmission rate, i.e: the transmission rate is divided equally and each connected host (A, B, and D) gets 2.5 MBps each. Therefore the throughput from A to C becomes 2.5 MBps instead.

# Network Performance Visualisation 
## The Time-Space Diagram

The time-space diagram as shown below can be used to visualise network performance. 

Network performance is <span class="orange-bold">throughput and delay limited</span>, and the only way to improve it is by using faster links with bigger bandwidth.


<img src="{{ site.baseurl }}//docs/NS/images/03-network-performance/2024-04-25-16-38-40.png"  class="center_fifty"/>

**Vertical** direction represents **time**, while **horizontal** direction represents **space** between source and destination points (distance). You can compute the last bit transmitted given $$L$$ (packet length) and $$R$$ (bandwidth). 



{:.important}
Ask yourself: will the <span class="orange-bold">slope</span> of the transmission line in the space-time diagram ever change between the same source and destination? Why or why not? What does it represent? 

{:.highlight}
Please head to Class Activities for more practice. 

# Summary

In this chapter on network performance, we explore the critical aspects that influence the efficiency and reliability of data transmission across networks. We present you with different types of packet delays: processing, queueing, transmission, and propagation; and their impact on **overall** network performance. The chapter also covers essential diagnostic tools like traceroute and ping, which help map network paths and measure connectivity and speed. Furthermore, we explain concepts like throughput versus bandwidth and the implications of packet loss, providing a thorough understanding of how to **analyze** and **optimize** network performance.

Key learning points include:
- **Types of Network Delays**: Breaks down the key contributors to network latency: processing, queueing, transmission, and propagation delays.
- **Diagnostic Tools**: Highlights the use of tools like traceroute and ping to measure these delays and diagnose network performance issues.
- **Performance Metrics**: Discusses critical metrics such as packet loss, throughput, and bandwidth that affect and reflect the performance of network systems.

For a more detailed examination, you can view the complete note [here](https://natalieagus.github.io/50005/ns/03-network-performance).
