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

**Root Server** (highest in the hierarchy): contacted by local name servers (end hosts) that cannot resolve names. End hosts get mapping, and return to the local name server. There are 13 root servers in the world. Find the list at http://root-servers.org/.

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-07-17-14-52.png"  class="center_seventy"/>

You can look for its individual IP, e.g: a.root-servers.net IP:

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-07-17-15-32.png"  class="center_seventy"/>

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

# DNS Records

In this section we are going to analyse how some types of DNS records that are stored in DNS servers look like, in particular, these record types: A, MX, and NS. 

{:.note-title}
> Resource Records
>
> DNS resource records (RR) are <span class="orange-bold">entries</span> stored in a DNS zone file that provide information about a domain. These records are used to facilitate the translation of human-readable domain names into IP addresses (which are necessary for locating and identifying computer services and devices on the Internet) and to provide other essential information associated with the domain.
> 
> In short, the DNS is basically a distributed database storing these RRs.

The **data structure** of a resource record is as follows. Each time a client query for hostname resolution, the servers consults these records. 

<img src="{{ site.baseurl }}/docs/NS/images/06-dns/cse2024-RR.drawio.png"  class="center_seventy"/>

An RR in general contains the following information (but not limited to): name, value, type, and TTL. The table below explains what each information means:

Name | Value | Type | TTL 
---------|----------|---------|---------
 Hostname, e.g: sutd.edu.sg | IP Address | A | N seconds
 Domain, e.g: sutd.edu.sg | The nameserver that is authoritative to this domain | NS | N seconds
 Alias name for some canonical (real) hostname | Real hostname | CNAME | N seconds
 Domain, e.g: sutd.edu.sg | mailserver name, e.g: sutd-edu-sg.mail.protection.outlook.com. | MX | N seconds

{:.note}
Note that sutd.edu.sg are both domain and hostname. However this is just a specific example. Not all domains are hostnames, and not all hostnames are domains. See this [appendix](#domain-vs-hostname) section to understand more. 

# DNS Caching

The Local Name Servers will <span class="orange-bold">cache</span> hostname-ip mapping once a query is made for a certain TTL (time to live). Due to this caching mechanism, TLD servers are typically cached in DNS local name servers, allowing for faster resolution. On the other hand, <span class="orange-bold">Root servers are often not visited</span>.

## Handling Cache Update
Cached entries may be out of date, e.g: if a host changes IP address, it may not be known Internet-wide until all TTLs (time-to-live) expire. 
* The authoritative name server decides the DNS record TTL, for example, Netflix's authoritative name server decides the TTL for its DNS records 
* The local nameserver re-queries when TTL expires
* The update or notify mechanisms of these caches are proposed by IETF standard, in particular, [RFC 2136 -- Dynamic Updates in the Domain Name System](https://datatracker.ietf.org/doc/html/rfc2136) (DNS UPDATE)


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

## SOA 

{:.info}
The SOA (Start of Authority) record is a type of DNS (Domain Name System) record that contains administrative information about a DNS zone. It is one of the essential records in a DNS zone file, acting as a blueprint that provides details about the zone and how it should be managed. The SOA record is mandatory in every DNS zone and is used to convey properties about the zone to the DNS infrastructure.

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-07-17-28-02.png"  class="center_seventy"/>

When you use the `dig` command to query DNS and receive a reply containing just an "SOA" (Start of Authority) record, this provides crucial information about the DNS zone, but it might also indicate specific query contexts or configurations.

### Understanding the SOA Record

The SOA record is one of the most important types of DNS records and appears at the <span class="orange-bold">beginning</span> of a zone file. It contains administrative details about the zone, including:

1. **Primary DNS server** - The authoritative name server for the domain.
2. **Responsible party** - The email address of the person responsible for administering the domain's DNS zone.
3. **Serial number** - A sequence number that increments whenever the zone file is updated. This is used by secondary DNS servers to check if they need to update their copies.
4. **Refresh rate** - How frequently secondary DNS servers should check if updates are needed.
5. **Retry rate** - How long a secondary server should wait before retrying after a failed refresh.
6. **Expire time** - How long the zone will be considered valid if the secondary server can’t reach the primary server.
7. **Negative caching TTL** - Time to Live value for negative results, indicating how long a DNS resolver should cache the absence of a record.

### Why You Might See Only an SOA Record

If your `dig` query returns only the SOA record, here are a few reasons why this might happen:

1. **Query for Non-Existent Subdomain**: If you query a subdomain or record that doesn't exist, the DNS server might return the SOA record to indicate the authoritative server for that zone, especially if the query is for a non-existent type (e.g., asking for an MX record where none exists).
   
2. **Zone Transfer Attempt**: If you attempt a zone transfer (e.g., using `dig AXFR`) and it's not allowed by the server configuration, the server may return the SOA record as a part of its refusal to provide a zone transfer.

3. **Specific DNS Query**: If your query explicitly asks for the SOA record (e.g., `dig SOA example.com`), then naturally, the SOA record will be the response.

4. **DNS Configuration Issues**: Sometimes, misconfigurations in the DNS setup might lead to unexpected results like only returning an SOA record even if other records exist.

### Example of a `dig` Command Showing an SOA Record

```bash
dig SOA google.com
```

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-07-17-29-02.png"  class="center_seventy"/>

This command will provide you with the SOA record for `google.com`, listing the details as explained above. If this wasn't the intended result and you were expecting other types of DNS records, you might want to specify the record type directly in your `dig` query, such as `A`, `MX`, `CNAME`, etc., or check if the domain or subdomain queried is correctly configured in the DNS.

## Domain vs Hostname

The terms "domain" and "hostname" can sometimes be used interchangeably, but they have specific meanings that are worth distinguishing, especially in the context of how a domain like `sutd.edu.sg` can also function as a hostname.

### Domain
A domain is part of the Domain Name System (DNS) hierarchy. It is a subset of the broader internet address space that is reserved or registered under a specific name. For example, in `sutd.edu.sg`, `edu.sg` is the second-level domain (part of the `.sg` country code top-level domain), and `sutd` is a third-level domain under `edu.sg`.

### Hostname
A hostname is a specific identifier used for a device or a node on a network. It is the label assigned to a device connected to a computer network and is used to distinguish one device from another on that network. Hostnames are part of fully qualified domain names (FQDNs), which provide specific addresses for individual nodes.

### How Domains Can Also Be Hostnames
In many cases, especially with organizational or institutional websites, the domain name can also serve as the hostname. For instance, `sutd.edu.sg` can be set up to be both the domain name of the Singapore University of Technology and Design and the hostname for the main server where their primary website is hosted.

#### Example Breakdown
- **Domain**: `sutd.edu.sg`
- **Hostname**: In network terms, the hostname could simply be `sutd.edu.sg` if that's how the primary server is identified within the network.
- **FQDN**: The FQDN might simply be `sutd.edu.sg`, pointing directly to the server that hosts the university’s main website.

### Practical Implementation
When you enter `sutd.edu.sg` in a web browser:
- DNS resolves `sutd.edu.sg` as a hostname to an IP address where the server hosting the university's main website can be reached.
- Technically, the hostname can have prefixes like `www`, so `www.sutd.edu.sg` might also point to the same server or to different services as needed.

Thus, while a domain is part of the DNS hierarchy and serves as a human-readable address for a section of the internet, a hostname is a specific address for a network node (often a server). A domain like `sutd.edu.sg` can serve as <span class="orange-bold">both</span>, fulfilling the roles of domain and hostname, especially if it directly points to a specific server hosting services under that domain.