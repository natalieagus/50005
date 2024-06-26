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
{: .no_toc}

{:.highlight-title}
> Detailed Learning Objectives
>
> - **Understand the Basics of DNS**
>   - Explain what DNS is and its role in the internet.
>   - Understand the hierarchical and decentralized nature of DNS.
>   - Describe how DNS translates human-friendly domain names into IP addresses.
> - **Comprehend DNS Operations**
>   - Explain the process of accessing a website using DNS.
>   - Understand the role of local DNS servers and authoritative DNS servers.
>   - Describe the steps involved in a DNS lookup, from entering a domain name to accessing the website.
> - **Identify the Structure and Components of DNS**
>   - Understand the structure of DNS, including root servers, top-level domain (TLD) servers, and authoritative name servers.
>   - Describe the purpose and function of each type of DNS server.
>   - Recognize the importance of local domain name servers and their role in DNS resolution.
> - **Analyze DNS Query Types**
>   - Differentiate between iterative and recursive DNS queries.
>   - Explain the resolution order for both iterative and recursive queries.
>   - Understand how DNS queries are processed and the load distribution between clients and servers.
> - **Explore DNS Services and Features**
>   - Identify additional services provided by DNS, such as host aliasing, mail server aliasing, and load distribution.
>   - Understand the concept of DNS caching and its impact on query speed and efficiency.
>   - Recognize the significance of DNS in maintaining strong modularity, fault isolation, and scalability.
> 
> These learning objectives aim to provide a comprehensive understanding of the Domain Name System (DNS), including its operations, structure, query types, services, and security aspects.

{:.info-title}
> Domain Name System
>
> The Domain Name System (DNS) is a naming system for computers, services, or other resources connected to the Internet or a network of computers. 
>
> It is a **hierarchical** and **decentralized** naming system that allows computers, services, or resources connected to the Internet or a private network to be identified and accessed. It translates human-friendly domain names, like example.com, into the numerical IP addresses needed by computers to communicate with each other. This system makes browsing the internet and accessing network resources **easier** for users while handling the technical mapping in the background.
>
> DNS is **both** a naming system and a protocol, read along to find out more.

## Motivation

Imagine you're in your university library, working on a research project, and you type in a website like `example.com` into your browser. The site loads almost instantly. This quick response is thanks to the Domain Name System (DNS), which works like the internet's phone book, converting user-friendly names to the numerical IP addresses computers use to communicate.

Here's what happens when you access a website through the campus network:

1. **Enter a Domain Name**: You type `example.com` into your browser.

2. **University DNS Server**: Your computer queries the local DNS server maintained by the university, which could be located in a data center on campus and maintained by the school's IT team. If the server has recently looked up `example.com`, it knows the IP address and sends it back to your computer. 

3. **Further Lookups**: If the university server doesn't know the IP address, it starts asking other DNS servers on the internet:
   - It first contacts a global root server that directs it to the top-level domain (TLD) servers for `.com`.
   - The TLD servers then point to authoritative servers responsible for `example.com`.

4. **Authoritative DNS Server**: The authoritative DNS server provides the final answer with the correct IP address for `example.com`.

5. **Access the Website**: With the IP address provided, your computer can now connect directly to `example.com`.

In summary, the university's DNS server in the data center acts as an **intermediary** to quickly find the correct IP address for any website you access, enabling seamless browsing on campus.

{:.note-title}
> IP Addresses
> 
> IP addresses (32 bits or 64 bits) are used to address datagram by routers:
> * For instance, IPv4 (32 bits) `172.217.194.99` (dotted notation a.b.c.d in groups of unsigned bytes -- octet)
> * This is convenient for hierarchical routing: facilitates routing (remote router uses short prefix of destination IP address only to decide next hop – much smaller routing table)


<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-06-17-57-08.png"  class="center_seventy"/>


DNS is a system that acts as a phonebook. We will remember our friends by their names, and not their contact number. When we want to actually contact them, we need to translate their names into their contact numbers using our phonebook. The DNS is much like our phonebook, where it translates between domain name and IP addresses. 

{:.info}
Head to this [appendix](#finding-dns-server) section if you'd like to find the IP address of the DNS server your smart devices is currently using. 

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

# DNS Structure

{:.info}
DNS is a **distributed** and **hierarchical** database.

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-07-17-13-04.png"  class="center_seventy"/>

## Root Server

**Root Server** (highest in the hierarchy): contacted by local name servers (end hosts) that cannot resolve names. End hosts get mapping, and return to the local name server. There are 13 root servers in the world. Find the list at [<http://root-servers.org/>](http://root-servers.org/).

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-07-17-14-52.png"  class="center_seventy no-invert"/>

You can look for its individual IP, e.g: a.root-servers.net IP:

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-07-17-15-32.png"  class="center_seventy no-invert"/>

The root servers are basically a <span class="orange-bold">fixed</span> set of name servers that maintain a list of the authoritative (master/slave) name servers for every registered top level domain (e.g: edu., gov., com., etc). They may be located at:
1. Companies contracted to provide the service by ICANN (The Internet Corporation for Assigned Names and Numbers) or 
2. Government institutions. 

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-07-17-16-03.png"  class="center_seventy"/>

## Top Level Domain (TLD)

**TLD Server**: responsible for anything after the last dot, such as `.com`, `.edu`, `.sg` etc. They are maintained by **large** organizations or countries. For example, Verisign global registry services is responsible for `.com`  domain.  

## Authoritative Name Servers 
**Authoritative Name Servers**: organization’s own DNS server, providing authoritative (<span class="orange-bold">final</span>, no need further query) hostname to IP mappings for organization’s named hosts. They're maintained by the organization itself or by internet provider / cloud service management system like AWS.

## Local Domain Name Servers 
**Local DNS**: also called DNS resolvers are used by DNS clients (our computers or smart devices) to resolve domain names to IP addresses. They serve as proxy for hosts to make DNS queries, and forward queries to the hierarchy:
* Local DNS Name Servers have local cache of recent name-to-address translation pairs 
* Need to maintain and update these caches from time to time

Common DNS resolvers include
* ISP-given DNS resolver
* Google DNS resolver: 8.8.8.8
* Cloudflare DNS resolver: 1.1.1.1

# DNS Name Resolution

There are two ways to resolve the hostname-IP translation: iterative and recursive. 

## Iterative Query 

{:.info}
In an iterative DNS query, the DNS client allows the DNS server to return the best answer it can. If the queried server does not have the answer, instead of asking other servers itself, it returns a referral to other more authoritative DNS servers. 

The resolution order is as drawn: 

<img src="{{ site.baseurl }}/docs/NS/images/06-dns/cse2024-iterative-query.drawio.png"  class="center_seventy"/>

{:.highlight}
Majority of the "work" is done by the DNS client.

## Recursive Query 

In **recursive** query, we put the burden of name resolution on the contacted name server. This puts a heavy load at the <span class="orange-bold">upper hierarchy</span>. 

The resolution order is as drawn:

<img src="{{ site.baseurl }}/docs/NS/images/06-dns/cse2024-recursive-query.drawio.png"  class="center_seventy"/>

# Summary

DNS is a **distributed** database that provides name-IP translation. Distributed system of servers provide <span class="orange-bold">scalability</span>. 

The presence of DNS **protects** domains. The same name can point to a different physical machines hence allows for:
* Strong modularity, 
* Strong fault isolation


DNS also provides <span class="orange-bold">indirection</span> (name to IP address) as design principle has many virtues:
* **Late binding at runtime**, e.g: physical server can move around with different IP and keeping the same name
* **Many-to-one mapping**: aliasing. Some people use *multiple* domains aliased to a single site as part of their search engine strategy.
* **One-to-many mapping**: the same domain name can have many IP addresses. This is useful for load balancing. 


# Appendix

## Finding DNS Server

Finding the DNS server your computer is using can be helpful for troubleshooting network issues or simply for informational purposes. Here’s how to do it on different operating systems:

### Windows
1. **Open Command Prompt**: Press `Win + R`, type `cmd`, and hit `Enter`.
2. **Run the `ipconfig /all` Command**: Type `ipconfig /all` in the Command Prompt and press `Enter`. Scroll through the information until you find the "DNS Servers" entry. The IP addresses listed are your DNS servers.

### macOS
1. **Open Terminal**: You can find Terminal in Applications under Utilities, or you can search for it using Spotlight.
2. **Run the `scutil --dns` Command**: Type `scutil --dns` in the Terminal and press `Enter`. Look for "nameserver" entries under the DNS configuration section. These are your DNS servers.

### Linux
1. **Open a Terminal**: Use a shortcut or find it in your applications menu.
2. **Check the resolv.conf File**: Type `cat /etc/resolv.conf` in the Terminal and press `Enter`. This file contains a list of DNS servers; look for lines starting with `nameserver`. The IP addresses following `nameserver` are your DNS servers.

### On a Smartphone
- **Android**:
  - Go to `Settings` > `Network & Internet` > `Wi-Fi`.
  - Tap on the Wi-Fi network you are connected to.
  - Look for DNS settings, which might be under `Advanced options`.
- **iOS**:
  - Go to `Settings` > `Wi-Fi`.
  - Tap the information (`i`) icon next to the network you are connected to.
  - Scroll to the DNS section to see the DNS servers.

These steps will help you identify the DNS server addresses your device is currently using.

## Common Terminologies
Below are some terminologies that are related to the topic and might be useful to fully appreciate the chapter: 
**DNS Namespace**: represent the entire DNS structure tree (all nodes)

**Domain**:
* Logical division of DNS namespace
* Represents a the entire set of names / machines that are contained under an organizational domain name (subtree), eg:
  * “.com” websites are part of the “com” domain, 
  * en.wikipedia.com and id.wikipedia.com websites are part of the “wikipedia.com" domain.

**Zone**: **physical** division of DNS namespace 
* DNS namespace can be **broken** into zones for which individual DNS servers are responsible. Each zone has its own zone file. 
{:.info}
A zone file is a plain text file stored in a DNS server that contains an actual representation of the zone and contains all the records for every domain within the zone. Zone files must always start with a Start of Authority (SOA) record, which contains important information including contact information for the zone administrator.
* Unlike “domain” that’s a <span class="orange-bold">subtree</span>, a zone can just be a bunch of sibling nodes. 
* A DNS zone is not necessarily associated with one server, so two servers or more can store the same copy of the zone. 
* A DNS server can also contain multiple zones.
* If a domain is not subdivided to subdomains, then domain is equivalent to the zone

### Example about DNS Zone

Imagine you own a domain `example.com`. This domain can have various subdomains, such as `www.example.com`, `mail.example.com`, or `shop.example.com`. Each of these can technically be part of a different DNS zone, but they might also all be in one zone, depending on how you choose to organize your DNS records.

#### Example 1: Single DNS Zone for All Records
- **Domain**: `example.com`
- **DNS Zone**: In this scenario, you might have a single DNS zone for `example.com` that includes all the DNS records for the domain and its subdomains. The zone file could include:
  - `A` record for `example.com` pointing to `192.168.1.1`
  - `CNAME` record for `www.example.com` pointing to `example.com`
  - `MX` record for `example.com` to manage email services
  - `A` record for `shop.example.com` pointing to `192.168.1.2`

#### Example 2: Multiple DNS Zones for Different Purposes
- **Domain**: `example.com`
- **DNS Zone 1**: Main zone for `example.com`
  - `A` record for `example.com` pointing to `192.168.1.1`
  - `MX` record for `example.com`
- **DNS Zone 2**: Separate zone for the subdomain `shop.example.com`
  - `A` record for `shop.example.com` pointing to `192.168.1.2`
  - Separate administrative settings specific to the e-commerce platform

#### How DNS Zones Work
The key point about DNS zones is that they are <span class="orange-bold">administrative</span> blocks that allow you to manage how different parts of your domain's namespace are handled. Each zone can be hosted on the same or different DNS servers. If you delegate a subdomain like `shop.example.com` to a different DNS zone, you can specify different DNS servers for it, which might be managed by a different team or a third-party service, helping distribute the load and administrative responsibilities.

#### Practical Application
In practice, if you manage a large organization, you might want DNS zones for different departments or functions to separate administrative control or enhance security. For example, IT might manage the main domain's DNS records, while marketing manages a subdomain for promotional sites in a different DNS zone.

Understanding DNS zones as these distinct entities that contain DNS records for specific namespaces under your domain can help you manage and delegate DNS more effectively.

