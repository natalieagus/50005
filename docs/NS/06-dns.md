---
layout: default
permalink: /ns/06-internet-naming-and-addressing
title: Internet Naming and Addressing
description: Basics about DNS
parent: Network Security
nav_order: 6
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



# Domain Name System

{:.info-title}
> Domain Name System
>
> The Domain Name System (DNS) is a naming system for computers, services, or other resources connected to the Internet or a network of computers. 
>
> It is a **hierarchical** and **decentralized** naming system that allows computers, services, or resources connected to the Internet or a private network to be identified and accessed. It translates human-friendly domain names, like example.com, into the numerical IP addresses needed by computers to communicate with each other. This system makes browsing the internet and accessing network resources **easier** for users while handling the technical mapping in the background.
>
> DNS is **both** a naming system and a protocol, read along to find out more.

## Motivation

IP addresses (32 bits or 64 bits) are used to address datagram by routers:
* For instance, IPv4 (32 bits) `172.217.194.99` (dotted notation a.b.c.d in groups of unsigned bytes -- octet)
* This is convenient for hierarchical routing: facilitates routing (remote router uses short prefix of destination IP address only to decide next hop – much smaller routing table)

However, for ease-of-use, users would prefer to identify their traceroute / ping targets, websites to visit by their names and <span class="orange-bold">not</span> IP addresses. For instance: www.google.com, www.netflix.com.  The Domain Name System (DNS) is an essential component of the internet that **translates** human-readable domain names (like example.com) into numerical IP addresses (like 93.184.216.34), which computers use to identify each other on networks.

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-06-17-57-08.png"  class="center_seventy"/>

{:.note}
Think of it like a phonebook. We will remember our friends by their names, and not their contact number. When we want to actually contact them, we need to translate their names into their contact numbers using our phonebook. The DNS is much like our phonebook, where it translates between domain name and IP addresses. 

# Characteristics 

## Distributed Database

The DNS is **implemented** in hierarchy of name servers. We do not centralise DNS because:
* Need to avoid single point of failure, 
* Avoid creating a bottleneck traffic volume
* Avoid distant centralized databases, causing long delays for lookups (remember DNS lookup is only name-IP translation! We haven't even looked up the website yet) 
* Need to support ease of maintenance (add/delete/change translation). We do not want to keep going to one place to do this.

Furthermore, a centralised DNS does not scale.

## Application Layer Protocol 
The Domain Name System (DNS) is vital as one of the core functions of the internet. It enables hosts and name servers to communicate, resolving user-friendly names into numerical IP addresses so computers can locate each other and exchange data. As an application-layer protocol, DNS operates at the top of the Internet Protocol Suite, handling requests from other applications and services to ensure seamless access to network resources.

DNS is implemented at the **network's edge**, managing complexity through a hierarchical and decentralized system. Key features of this structure include:
- **Hierarchical Organization**: The hierarchical arrangement of root servers, top-level domains, and authoritative name servers ensures efficient and organized domain name resolution.
- **Edge Complexity**: By placing complexity at the network's edge, DNS remains scalable and effective in handling the growing demands of the internet.

# DNS Services
Apart from providing hostname to IP address translation, DNS also provides the following services:
* **Host aliasing**: map alias names to canonical name
* **Mail server aliasing**
* **Load distribution**: replicated web servers such that many IP addresses correspond to one name
