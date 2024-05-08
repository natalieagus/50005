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

**Types:**
* **A**: Address, it contains the IP address of the hostname in question.
* **NS**: Name Server, it tells which nameservers are authoritative for a that domain in question. You can have multiple NS records for load distribution (increasing availability)
* **CNAME**: Canonical Name, maps an alias name to a true or canonical domain name. Requires more probing (queries) to resolve to an IP eventually. It is like the same website with two or more URLs. 
* **MX**: Mail server, points you to the mail server name responsible for the domain in question. 

{:.note}
Notice that out of all the 4 types that we learn, only the **A record** has an IP address in its “value” field. 

## DNS Query Solving Mock Example

### Find your local nameserver
Suppose you want to know the IP of `b.example.com`. You ask your local DNS server (A local DNS nameserver whose IP you know, usually a public knowledge or setup by your ISP, or by yourself if you have a DNS server at home). You can find your local DNS server using the command `scutil --dns`:

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-08-11-21-19.png"  class="center_seventy no-invert"/>

### Sample local nameserver records 
Now suppose your local DNS server has the following records:

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-08-11-22-57.png"  class="center_seventy no-invert"/>

This means that your local DNS server does <span class="orange-bold">not</span> know what `b.example.com`'s IP address is, since it contains only its NS record. We should then ask `NS2.server.com` regarding `b.example.com`’s IP address. 

{:.note}
Since `b.example.com` NS record points to `NS2.server.com`, it means that `NS2.server.com` is the **authoritative** nameserver for `b.example.com`.

### Further probe to `b.example.com` authoritative nameserver 
The local DNS server can then do the job of translating  `NS2.server.com` into its IP address: `111.222.125.124` since it already has the A record of `NS2.server.com`. Otherwise, we have to find the IP for NS2.server.com first by querying either the root server or the .com TLD server.

Afterwhich, we (or the local DNS server, depending on whether the query is iterative or recurisve) shall ask `111.222.125.124` (`NS2.server.com`)  if it has the A record of `b.example.com`. 

### Sample authoritative nameserver records
Now suppose NS2.server.com has the following record: 

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-08-11-24-02.png"  class="center_seventy no-invert"/>

The first query to `NS2.server.com` will show that  `b.example.com` is a `CNAME` record, so it is simply an **alias** of another hostname called `a.example.com`.

### Resolving `b.example.com`
The local DNS server *or* the NS server (depending on whether it is an iterative or recursive query) has to continue the probe and send another DNS query to resolve the IP address for `a.example.com`, and finally `NS2.server.com` returns `a.example.com`'s A record with IP of `10.12.14.145`. 

Now that hostname-IP translation for `b.example.com` is resolved, we can start sending requests to `10.12.14.145` (our intended `b.example.com` host). 

# DNS Query and Reply Protocol

DNS queries and replies follow a **specific** message format outlined in [RFC 1035](https://datatracker.ietf.org/doc/html/rfc1035), which structures the data to be transmitted over the network.

The format is as follows. Below we explain certain parts of the message: 

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-08-12-04-07.png"  class="center_fourty"/>


**Identification**: 16 bit number for query, reply uses the same number

**Flags**:
* Query or Reply?
* Recursion desired or available?
* Reply is authoritative?

**Size**: Varies, depending on number of RRs in each section. Generally, replies are larger than queries.


Components of the variable lengths of DNS query / reply:
* **Questions**: name, type fields for a query
* **Answers**:  RRs in response to the query
* **Authority**: Records for authoritative servers
* **Additional information**: possibly helpful information 

{:.note}
Tools like `dig` and `nslookup` are designed to craft DNS query messagse and query DNS servers, then *render* out the information contained in DNS reply messages. They provide different levels of detail and formats, helping users and administrators understand DNS responses and diagnose network issues.

## Example of DNS query via nslookup

Let’s say we want to resolve the name google.com. We can do so using system programs such as `dig` or `nslookup`:

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-08-11-29-36.png"  class="center_seventy no-invert"/>

There are 6 different IP addresses for the same domain name: `google.com`. This is probably done by Google to ensure <span class="orange-bold">load balancing</span> if there’s a surge of query made to access `google.com` website. We can ping one of them to check if they are alive:

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-08-11-30-24.png"  class="center_seventy no-invert"/>


The answer we obtain is <span class="orange-bold">non-authoritative</span> because these records are obtained from the local name server instead: 192.168.2.22 (cached).

### Obtaining authoritative answer

Hdo we find the IP translation from google.com’s authoritative name server? First, we need to find the NS-type record of `google.com` using nslookup:

```sh
nslookup -type=ns google.com
```

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-08-11-31-55.png"  class="center_seventy no-invert"/>

There are **four** NS-type records, which means that all these servers are the authoritative name servers for `google.com`. What we have to do know is to query *one of them* instead about the IP address of `google.com`:

```sh
nslookup google.com ns1.google.com
```
<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-08-11-33-32.png"  class="center_seventy no-invert"/>


{:.note}
Notice that we have two names as arguments to nslookup. `nslookup` has to **resolve** `ns1.google.com` first (obtain its IP address), and then resolve `google.com` by sending a DNS query to `ns1.google.com`.

You can also do this manually:
1. Resolve IP address of `ns1.google.com` 
2. Use that IP address to query about `google.com`

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-08-11-35-16.png"  class="center_seventy no-invert"/>

You can also obtain the MX record type to obtain the names of the mail servers in google.com domain:

```sh
nslookup google.com 216.239.32.10 -type=MX
```

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-08-11-35-41.png"  class="center_seventy no-invert"/>

{:.important}
In `nslookup`, the absence of the "Non-authoritative answer" message typically indicates that the response is authoritative.

## Example of DNS query via dig 

We can also use another tool: `dig` (Domain information groper) to query the IP address for `google.com`: 

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-08-12-07-49.png"  class="center_seventy no-invert"/>

The output is a little different because dig decodes the dns response and presents it in another format to us, but there’s some things we can relate to, and compare with the simplified DNS query and reply protocol we learned above. 
1. Look at the **header**: we have id (transaction id) 22259.

2. The **flags** section gives us more information about the response. 
   * `qr` (query response): indicates that the message was a **response**, not a query. dig always renders out the DNS response, not queries, so `qr` flat will always be present. 
   * `rd` (recursion desired): Most of the time `rd` flag is set in the query so that we don't have to iteratively resolve our query and avoid the manual work
   * `aa` (authoritative answer): if present, the response is obtained from an authoritative nameserver. In this example, there's no `aa` flag so our answer is *not* obtained from the authoritative nameserver of `google.com`
   * `ra` (recursion available): this flag indicates that recursion was available from the remote name server. 

3. Next we have **numbers** of queries, answers, authority and additional section. 
   * The **authority** section typically indicates the server(s) that are authoritative for answering DNS queries about that domain. In this example, we don’t have any answer in the authority section.
   * The other sections are self explanatory 

4. Next is the **answer** section, we have 6 A records for `google.com`. 
   * In `dig`, we know that this is non-authoritative because otherwise there’s supposed to be an `aa` flag set (indicating authoritative answer). * Therefore, this answer comes from the local nameserver's cache. 
   * Each column in the ANSWER section indicates: TTL, CLASS, TYPE, and VALUE.
   * 48 means TTL in seconds, and IN stands for INTERNET.

{:.info}
At the bottom of the screenshot, the SERVER means our local DNS server making the query for us. Our local DNS server has an IP of 192.168.2.22. The transport layer protocol utilised by DNS is UDP and the port used is #53. We will learn more about UDP and TCL in the next chapter. 

### Obtaining authoritative answer 

Suppose we want to obtain an authoriative answer to resolve `google.com` using dig, we can query one of its authoritative nameservers like before  instead: 

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-08-12-16-14.png"  class="center_seventy no-invert"/>

* This response contains an <span class="orange-bold">authoritative</span> answer, with the `aa` flag set. 
* There’s no `ra` flag because this nameserver has **recursion disabled**, probably for security reasons. 


# DNS Caching

The Local Name Servers will <span class="orange-bold">cache</span> hostname-ip mapping once a query is made for a certain TTL (time to live). Due to this caching mechanism, TLD servers are typically cached in DNS local name servers, allowing for faster resolution. On the other hand, <span class="orange-bold">Root servers are often not visited</span>.

## Handling Cache Update
Cached entries may be out of date, e.g: if a host changes IP address, it may not be known Internet-wide until all TTLs (time-to-live) expire. 
* The authoritative name server decides the DNS record TTL, for example, Netflix's authoritative name server decides the TTL for its DNS records 
* The local nameserver re-queries when TTL expires
* The update or notify mechanisms of these caches are proposed by IETF standard, in particular, [RFC 2136 -- Dynamic Updates in the Domain Name System](https://datatracker.ietf.org/doc/html/rfc2136) (DNS UPDATE)

## Inspecting cached queries 

Try running `dig [SOME-DOMAIN]` twice or thrice, and see if the subsequent query is **faster**. That means your subsequent query (not the first one) is cached, while the first one is not. 

```sh
# run this twice, consecutively
dig nus.edu.sg 
```

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-08-12-18-25.png"  class="center_seventy no-invert"/>

In the example above, the second query about `nus.edu.sg` is **faster** (just 1ms) compared to the first query. 

Alternatively, you may **disable** recursion, for instance:

```sh
dig google.com +norecurs
```

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-08-12-19-40.png"  class="center_seventy no-invert"/>

If you still have an answer like the above, then the answer was from **cache**. 

Try some website you rarely visited, e.g:

```sh
dig selfridges.com +norecurs
```

This time round there's no answer provided, meaning that this record was <span class="orange-bold">*not* cached</span>: 

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-08-12-20-34.png"  class="center_seventy no-invert"/>

# DNS Record Insertion 

{:.info-title}
> DNS Record Insertion 
> 
> DNS record insertion refers to the process of <span class="orange-bold">adding</span> new records to a DNS zone to manage the resolution of domain names into IP addresses or other information. For example, if you manage your own DNS server at home (authoritative to sites you host at home), then you will need to insert some record (information about your authoritative DNS server at home) to the TLD.  
> 
> Even though you manage your own DNS zone, it is still part of a larger DNS hierarchy. 

Suppose you want your website to be accessible via the internet using your own domain name. You can **insert** (or register) DNS records to ensure that they are recognized Internet-wide, allowing other hosts to find your website. The steps are as follows:
1. Find a DNS Registrar to register the domain name of your site 
2. Find authoritative nameservers to responsible for solving your domain name 
3. Insert DNS records (A type) into the authoritative nameservers 
4. Insert DNS records (NS types) of your authoritative nameservers to the TLD 

## DNS Registrar 

{:.info}
A DNS registrar, also known simply as a domain registrar, is a company or organization accredited to manage the reservation of domain names on the internet.

Example registrar: Amazon Domain Registrar, GoDaddy. GoDaddy and AWS Route 53 allows you to register new domain names. You can search for and purchase domain names directly through GoDaddy website or the AWS Route 53 console.'

Note that a registrar is <span class="orange-bold">not</span> a registry (see [appendix](#registry) for definition of registry). A Registrar interfaces with the general public regarding the registration of their domain names. **In short, they sell domain names**. See more examples of registrars [here](https://www.icann.org/en/accredited-registrars?filter-letter=a&sort-direction=desc&sort-param=name&page=1). 

Therefore, suppose  for you want your website to be reachable at `examplesite.com`. You then need to **buy** this particular domain name (if it is still available) for any of the registrars above.   

## Authoritative Name Servers

You will then **need** to provide the **names** and **IP addresses** of the authoritative name server (primary and secondary server as backup) responsible for resolving this domain name `examplesite.com`. 

{:.info}
Authoritative name servers can be maintained by you (if you know how to manage a DNS server), or an ISP or other related companies (DNS hosting providers). For example, SUTD has 3 Authoritative nameservers hosted by starhub, and also SUTD herself. 

<img src="{{ site.baseurl }}//docs/NS/images/06-dns/2024-05-08-17-31-46.png"  class="center_seventy"/>

You can engage services like [AWS Route 53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-configuring.html) to *be* your domain's authoritative nameservers. You will need at least <span class="orange-bold">two</span> authoritative name servers. For instance, suppose AWS Route 53 assign you the following authoritative nameservers: 
* ns-1312.awsdns-36.org: 20.50.23.12
* ns-788.awsdns-34.net: 53.25.22.11

This means that you shall insert the A record of your domain `examplesite.com` into **both** of these authoritative nameservers. Suppose your website is hosted at IP address: 111.222.111.222, then:
1. Insert A record: `examplesite.com, 111.222.111.222, A, TTL` into ns-1312.awsdns-36.org DNS server
2. Insert the same A record into ns-788.awsdns-34.net DNS Server 

{:.note}
You can also insert MX record type to both authoritative name server if your organization also has a mail server, e.g: suppose you rely on Microsoft's email services, you shall insert MX record: `examplesite.com, examplesitemailserver.mail.protection.outlook.com, MX, TTL`.

## TLD Update 

Since you rely on AWS Route 53 for your authoritative nameservers, then Amazon is responsible for inserting the NS records for `examplesite.com` into the appropriate TLD server (.com TLD server in this example). These NS records are useful to indicate the Authoritative Name Server responsible for your domain:
* `examplesite.com, ns-1312.awsdns-36.org, NS, TTL`
* `examplesite.com, ns-788.awsdns-34.net, NS, TTL` 

The A record for these two nameservers must also be present and reachable from the internet. For instance, The IP addresses of `ns-1312.awsdns-36.org` and `ns-788.awsdns-34.net` are registered within the respective TLDs (.org and .net). This means that the .org and .net registries contain the `A` records for these nameservers. For example:
* `ns-1312.awsdns-36.org,  20.50.23.12, A, TTL`
* `ns-788.awsdns-34.net,  53.25.22.11, A, TTL`

{:.note}
If the authoritative nameserver happen to be in the `.com` domain, then the `.com` TLD should contain glue records to avoid circular dependency. Head to [appendix](#glue-records) if you'd like to find out more. 


# DNS Attack

Since DNS is a crucial part of the internet, it is prone to various attacks. Example of some DNS attacks are as follows:
* **DDoS attack** (Denial of Service): done either by bombarding root servers with traffic or bombarding TLD servers with traffic. The latter is potentially more dangerous since they are cached by local DNS servers.
  * Root server bombarding is not successful to date due to traffic filtering	and that local DNS servers cache IPs of TLD servers, hence we typically bypass the root servers
* **Redirect attack** (Man in the Middle): done via **DNS Cache Poisoning**, which is to send bogus replies to local DNS server, which caches the replies
  * Intercept DNS queries, direct it to a bogus server (not the actual server)
  * Client wont't be able to reach the actual server
* **Exploit DNS for DDoS**: 
  * Send queries with spoofed source address (that’s our target victim IP)
  * Amplify these queries
  * DNS servers that receive queries will answer to our target victim IP and flood it, causing its domain to be inaccessible


# Summary

DNS is a **distributed** database that provides name-IP translation. Distributed system of servers provide <span class="orange-bold">scalability</span>. 

The presence of DNS **protects** domains. The same name can point to a different physical machines hence allows for:
* Strong modularity, 
* Strong fault isolation


DNS also provides <span class="orange-bold">indirection</span> (name to IP address) as design principle has many virtues:
* **Late binding at runtime**, e.g: physical server can move around with different IP and keeping the same name
* **Many-to-one mapping**: aliasing. Some people use multiple domains aliased to a single site as part of their search engine strategy.
* **One-to-many mapping**: e.g: like google.com example above, for load balancing.


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

## Registry

A **Registry** only interfaces with the Registrar and is responsible for **delegating** IP addresses and the DNS infrastructure. A registry manages top-level domain (TLD)  names, meaning it maintains an authoritative database regarding top level domains including zone files. They create domain name extensions (see [here](https://en.wikipedia.org/wiki/List_of_Internet_top-level_domains), there’s many domain name extensions out there, not just .com), set the rules for that domain name, and work with registrars to sell domain names to the public. 

Example: Verisign, Comcast, etc. They are all accredited by ICANN. See more examples [here](https://www.icann.org/resources/pages/listing-2012-02-25-en). 

## Glue Records

A glue record is an A (IPv4) or AAAA (IPv6) record that provides the IP address of a nameserver. Glue records are used to prevent circular dependencies and ensure that DNS resolvers can find the nameserver's IP address. They are necessary when the nameserver's domain is the same as, or a subdomain of, the domain it serves.

### Example of a Glue Record

Consider the domain `examplesite.com` with nameservers `ns1.examplesite.com` and `ns2.examplesite.com`. Without glue records, there would be a circular dependency:

1. To resolve `examplesite.com`, the DNS resolver needs to contact `ns1.examplesite.com`.
2. To resolve `ns1.examplesite.com`, the DNS resolver needs to contact `examplesite.com`.

This creates a situation where the resolver cannot find the IP address of the nameserver without already knowing the IP address.

### How Glue Records Work

Glue records are provided by the parent zone (in this case, the `.com` TLD) to break this circular dependency. Here’s how it works:

1. **Domain Registration**: When you register `examplesite.com`, you specify `ns1.examplesite.com` and `ns2.examplesite.com` as nameservers.
2. **Registrar Submission**: Your registrar submits the NS records and the corresponding glue records to the `.com` TLD.

### NS Records:

```text
examplesite.com.  NS  ns1.examplesite.com.
examplesite.com.  NS  ns2.examplesite.com.
```

### Glue Records (A records for the nameservers):

```text
ns1.examplesite.com.  A  192.0.2.1
ns2.examplesite.com.  A  198.51.100.1
```

These glue records are stored in the `.com` TLD zone file. When a resolver queries the `.com` TLD for `examplesite.com`, it receives the NS records along with the glue records:

### Response from the `.com` TLD:

```text
examplesite.com.  NS  ns1.examplesite.com.
examplesite.com.  NS  ns2.examplesite.com.
ns1.examplesite.com.  A  192.0.2.1
ns2.examplesite.com.  A  198.51.100.1
```

The resolver now knows the IP addresses of the nameservers and can contact them directly to resolve `examplesite.com`.

### Conclusion

Glue records are essential for breaking circular dependencies when a nameserver’s domain is within the same zone it serves. They are provided by the parent zone to ensure that DNS resolvers can find the IP addresses of the nameservers.

### Visual Representation

Here’s a step-by-step visual of the process:

1. **Querying for `examplesite.com`**:
   - Resolver queries `.com` TLD for `examplesite.com`.
   - `.com` TLD responds with NS records and glue records.

2. **Contacting Nameservers**:
   - Resolver now has IP addresses for `ns1.examplesite.com` and `ns2.examplesite.com` from the glue records.
   - Resolver contacts `ns1.examplesite.com` or `ns2.examplesite.com` directly to resolve `examplesite.com`.

This ensures that the nameservers are reachable, and the DNS resolution process can proceed smoothly.
