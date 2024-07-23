---
layout: default
permalink: /labs/08-dns
title: Lab 8 - Domain Name Server 
description:  Explore how DNS works
parent: Labs
nav_order:  8
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

# Domain Name Server 
{:.no_toc}


In NS Module 4, we learned about the role of the Domain Name System (DNS) in Internet naming and addressing. In this lab exercise, we will go deeper into DNS by using specialised network tools to perform and analyse DNS queries.

At the end of this lab exercise, you should be able to:

- Use `dig` to perform DNS queries (e.g. to look up an IP address)
- <span style="color:#f77729;"><b>Read</b></span> and <span style="color:#f77729;"><b>interpret</b></span> DNS records of different types
- Understand how a DNS query is resolved using <span style="color:#f77729;"><b>hierarchy</b></span> and <span style="color:#f77729;"><b>recursion</b></span>
- Observe and understand the effect of <span style="color:#f77729;"><b>caching</b></span> on DNS lookup times
- Use <span style="color:#f77729;"><b>Wireshark</b></span> to trace and read DNS packets sent to and from a machine

## Submission 
There's only lab questionnaire on eDimension for this lab, no code submission needed. 

## Part 1: Exploring DNS via dig

The Domain Information Groper (`dig`) is commonly used for performing DNS lookups. Here is an example of how it can be used to find information about the host slashdot.org. The results may differ if you run the same query on your machine.

<img src="{{ site.baseurl }}/assets/images/nslab3/1.png"  class="center_seventy no-invert"/>

When the command `dig slashdot.org` is run, dig performs a DNS lookup and displays information about the request and the response it receives. At the bottom of the printout, we can see that the query was sent to the DNS server running on `192.168.2.100`, and that the query took 101 ms to complete. Most of the information that we are interested in can be found in the `ANSWER SECTION`.

The answer section for this query contains a DNS record:

| Server Name    | Expiry (TTL in seconds) | Class | Type | Data           |
| -------------- | ----------------------- | ----- | ---- | -------------- |
| `slashdot.org` | 293                     | IN    | `A`  | `104.18.29.86` |
| `slashdot.org` | 293                     | IN    | `A`  | `104.18.28.86` |

- We can see that the result is of <span style="color:#f77729;"><b>type</b></span> `A`, an address record.
- It tells us that the IP address for the domain name slashdot.org can be either `104.18.29.86` or `104.18.28.86`.
- The expiry time field indicates that this record is valid for 293 seconds.
- The value of the class field is usually IN (Internet) for all records.

If you’d like to know who’s the authoritative NS for the queried domain, you can add the trace option: `dig slashdot.org +trace`:
<img src="{{ site.baseurl }}/assets/images/nslab3/2.png"  class="center_seventy no-invert"/>

The records of type `NS` indicate the names of the DNS servers storing records for a particular <span style="color:#f77729;"><b>domain</b></span>. Here, we can see that the hosts `ns1.dnsmadeeasy.com.` and etc are responsible for providing <span style="color:#f77729;"><b>authoritative responses</b></span> to names in the `slashdot.org` domain.

We can <span style="color:#f77729;"><b>query</b></span> a specific server for information about a host by using the `@` option. For example, to perform a lookup using the DNS server `ns1.dnsmadeeasy.com.`, we can run the command `dig @ns1.dnsmadeeasy.com. slashdot.org.`:

<img src="{{ site.baseurl }}/assets/images/nslab3/3.png"  class="center_seventy no-invert"/>

There are three flags under the header: `qr`, `aa`, and `rd`:

- This means that the message is a query (`qr`), and dig is requesting a recursive lookup (`rd` stands for ‘recursion desired’) and the server is the authoritative name server (`aa` stands for ‘authoritative answer’).
- Not all servers perform recursive lookups due to the heavier load involved, and so you don’t see any `ra` flags here (`ra` stands for ‘recursion available’).



`dig` only prints the <span style="color:#f77729;"><b>final</b></span> result of a recursive search, but you can mimic the individual steps involved by making a query with the `+norecurs` option enabled. For example, to send a non-recursive query to one of the root servers, we enter the command `dig @a.ROOT-SERVERS.NET www.slashdot.org +norecurs`:

<img src="{{ site.baseurl }}/assets/images/nslab3/4.png"  class="center_seventy no-invert"/>

As you can see, the server <span class="orange-bold">does not know the answer</span> (there’s `0 ANSWER`) and instead provides information about the servers _most likely_ to be able to provide an authoritative answer for the question. In this case, the best that the root server knows is the identities of the servers for the `org.` top-level domain.

### `dig` Response Summary

Key sections of a dig response include:

* `QUESTION SECTION`: Shows the query made, including the domain name and the type of DNS record requested (e.g., A, MX, TXT).
* `ANSWER SECTION`: Contains the DNS records that answer the query. For instance, if you queried the A record for a domain, this section would list the IP addresses associated with that domain.
* `AUTHORITY SECTION`: Lists the authoritative name servers for the domain. These servers have the final authority over the domain's DNS records.
* `ADDITIONAL SECTION`: Provides additional information related to the query, often including IP addresses of the servers listed in the AUTHORITY SECTION, to help reduce the number of queries needed to resolve a domain.
* `STATISTICS`: Offers a summary of the query's execution, such as the total query time, server address, when the query was made, and more.

### Task 1

`TASK 1:` Using dig, find the IP address for thyme.lcs.mit.edu. What is the IP address?
{:.task}

### Task 2

`TASK 2:` The dig answer for the previous question includes a record of type CNAME. What does CNAME mean?
{:.task}

### Task 3

`TASK 3:` What is the expiration time for the CNAME record?
{:.task}

### Task 4

`TASK 4:` Run the following commands to find out what your computer receives when it looks up ‘ai’ and ‘ai.’ in the mit.edu domain. What are the two resulting IP addresses?
{:.task}

1. `dig +domain=mit.edu ai`
2. `dig +domain=mit.edu ai.`

{:.highlight-title}
> Ask yourself
> 
> What's the difference between the two commands? Look under the **Question** section of the `dig` response.

### Task 5

`TASK 5:` Find out why the results for both queries are different.
{:.task}

Look up the manual for `dig` to find out what the `+domain` parameter does. Based on the output of the two commands, what is the difference between the DNS searches being performed for `ai` and `ai.`?


# Hierarchy Caching

We will be using `dig` with `+norecurs` option for this section to try to resolve some domain and mimic manual **iterative** query. 
{:.info}

In the previous section, you ran `dig` without changing the default options. This causes `dig` to perform a <span style="color:#f77729;"><b>recursive</b></span> lookup if the DNS server being queried supports it. In this part, you will _trace_ the intermediate steps involved in a performing recursive query by beginning at a `root` server and <span style="color:#f77729;"><b>manually</b></span> going through the DNS hierarchy to resolve a host name. You can obtain a list of all the root servers by running the command `dig . NS`.

### Task 6

`TASK 6:` Use `dig` to query the `c` DNS root server for the IP address of `lirone.csail.mit.edu` <span style="color:#f77729;"><b>without</b></span> using recursion. What is the command that you use to do this?
{:.task}

### Task 7

`TASK 7:` Go through the DNS hierarchy from the root until you have found the IP address of `lirone.csail.mit.edu`.
{:.task}

You should disable recursion each time and follow all the referrals manually. Take note of all the commands that you use, and addresses that you found.

### Iterative Resolution Process
What you just did in the two tasks above mimic an **iterative resolution process**. 
* When a DNS resolver performs an iterative resolution, it **starts** querying the root DNS servers first.
* The root DNS servers provide a **referral** to the top-level domain (TLD) DNS servers (e.g., .com, .org).
* The TLD DNS servers then provide a **referral** to the authoritative DNS servers for the specific domain.
* Finally, the **authoritative** DNS servers provide the **actual** IP address for the domain.

If recursion is not available, you can ask `dig` to perform automatic iterative resolution for you. You can use the `+trace` option to see each step of the iterative process. 

{:.info}
From `dig` manual: `+trace` option performs **iterative** queries and display the entire trace path to resolve a domain name

For instance:

```sh
dig +trace disney.sg 
```

Results in:

```sh
; <<>> DiG 9.10.6 <<>> +trace disney.sg
;; global options: +cmd
.			514391	IN	NS	a.root-servers.net.
.			514391	IN	NS	b.root-servers.net.
.			514391	IN	NS	c.root-servers.net.
.			514391	IN	NS	d.root-servers.net.
.			514391	IN	NS	e.root-servers.net.
.			514391	IN	NS	f.root-servers.net.
.			514391	IN	NS	g.root-servers.net.
.			514391	IN	NS	h.root-servers.net.
.			514391	IN	NS	i.root-servers.net.
.			514391	IN	NS	j.root-servers.net.
.			514391	IN	NS	k.root-servers.net.
.			514391	IN	NS	l.root-servers.net.
.			514391	IN	NS	m.root-servers.net.
.			514391	IN	RRSIG	NS 8 0 518400 20240804200000 20240722190000 20038 . JeAF0TGVahJnkgDNbn5CvqQnUuJkK9Fm3slLNlnyp/HkNdmXTRY0ZFbk 0krZxWEl75fJv0vNDY740s/s02irZnMvKTGqcLS5lLivJNNKr3AA2dBv uWAbhHpgIObo8bNUnZCc8UbBuTvJSuuZ9wZAdahVG7GZBJZirVo2o6XV Ibv+9ElyU4i/ADr6JPN8MM0NSd9PQnt2ITR0HR/7UwfZy0Mt/jLi7tHo 6zA7/lGMMY1fxQJHJnT6WcYoZwAg8PZfvlyudb8ex/CzYN70CIZImlbe UH/9DUozo+qXF4u6C5zptB+qndksgT5tsvnpGsGaQxf+ezFcFQgiz9ub 7BAAIA==
;; Received 525 bytes from 192.168.2.22#53(192.168.2.22) in 7 ms

sg.			172800	IN	NS	ns4.apnic.net.
sg.			172800	IN	NS	pch.sgzones.sg.
sg.			172800	IN	NS	dsany.sgnic.sg.
sg.			172800	IN	NS	dsany2.sgnic.sg.
sg.			172800	IN	NS	dsany3.sgnic.sg.
sg.			86400	IN	DS	26329 8 2 404A15E6CA2F4CE259D8B2BD13894C7823A9513973103C962FF52BFC 57A380AF
sg.			86400	IN	RRSIG	DS 8 1 86400 20240804200000 20240722190000 20038 . uO9Yw4pQKVIa5wuVkLbApWO+PZKsZz6ds0Berm7hYLZaA0sNBu5HL1a+ aNDGmQ+xuBuEZqZrtBCEuibOWOxU4zfv/YyPtxQ3ObBGBBoyBPbZCp/c EK0FKjX9hyYjVzHk+yibioVhbzlmF7H6ExfzXFkRReF4YvvpK9jTE6tw si/QSruNcmjyKr1VLsOemsAIqsFZ5Cu5oDIgIyXj2cFkRLmB0+50yRUA 0DiklkCV2IC5CARDaUeSQLBJ7ccqZLD/ewnLJFBG9RMdYCs6DN//WLp0 c3x84A1S2sW2f5XP5L3RrOB4CoguU53XZR9cSqvVCe5IcKMD4pHfQA1z AnBZcA==
;; Received 718 bytes from 170.247.170.2#53(b.root-servers.net) in 6 ms

disney.sg.		3600	IN	NS	ns4.disneyinternational.net.
disney.sg.		3600	IN	NS	ns2.disneyinternational.net.
disney.sg.		3600	IN	NS	ns1.disneyinternational.net.
disney.sg.		3600	IN	NS	ns3.disneyinternational.net.
sk2017lipfiq108d3npir0g7s9hqufi6.sg. 3600 IN NSEC3 1 1 0 - SM90S2MA02ILRPA9TA4FPIOL4GTPOC77  NS SOA RRSIG DNSKEY NSEC3PARAM
sk2017lipfiq108d3npir0g7s9hqufi6.sg. 3600 IN RRSIG NSEC3 8 2 3600 20240728101424 20240721133051 43735 sg. L+02Z0D1av2/a+15lIhLSwebndFC0KSnp1iOcQiyps9GAdGspcv9xoXg IUUtn4w+29vwSJFXRGxIcR8pbIx1v+bNG4BVOf0dLmW3pGaYL1X7lO8l awPRJuw5F4fh2E5Nd9UIQZx3Sxx0gQ71/r4deIXr7qMnsRCBSBrWI4Qp PkZPX5oPO5YjRDeS93+i2nbgIfbLEzgY0FVlJBIjrQvrk/XY75TRxZ2q LcW3I7OOFuWrIwAUO5ODnpdWkpYc2y0Yyv2/MnR4nLVdmCmUgFBcI/kp 3IDoSFFqBt5hTCPMJBLNv31A/S/mL/uSlVI56D2tocjWJ0UJrcPxi+Yl 17TBUw==
0ghn9o7jkvc2r1r8g24mrmrauvjhks3g.sg. 3600 IN NSEC3 1 1 0 - 0HPO4SGKGC7DN06HPBGKFC6OBJRH9B30  NS DS RRSIG
0ghn9o7jkvc2r1r8g24mrmrauvjhks3g.sg. 3600 IN RRSIG NSEC3 8 2 3600 20240728030408 20240720223051 43735 sg. eSxket84qQILcZttDdhRnGKJ2eX/rIGg6td4g++/ufXNcJAHLo/GjGvu gCe+fs4KDQXJi0lRdkHKyb+lZefxlin/f+AqCQgm8rYO/pomzXj56jsz p9rvRa02usMPB/Y6CeawB4+1ZguoexCDO3DO0f//ui+rl0Ol9xwx5H7m Sf+mhsBpPPHwRCTmRP7dwVVDNOWLo4jI/c2k5+F+WrLoJdwsZ0rpP/EV 1ouKgDKgUMm98AkLRXNT6kzv2/U7v6BlvbX3yl/14eqPA14Y/fiFLCDJ awpAYkiu00LPGtxfbGcMGxffxMg8i2BwMtlOwtd+7gvp71OTarMaLpmC 7dN98A==
;; Received 872 bytes from 120.29.253.11#53(dsany.sgnic.sg) in 5 ms

disney.sg.		300	IN	A	13.248.150.189
disney.sg.		300	IN	A	76.223.18.1
disney.sg.		86400	IN	NS	ns1.disneyinternational.net.
disney.sg.		86400	IN	NS	ns2.disneyinternational.net.
disney.sg.		86400	IN	NS	ns3.disneyinternational.net.
disney.sg.		86400	IN	NS	ns4.disneyinternational.net.
;; Received 341 bytes from 205.251.196.47#53(ns1.disneyinternational.net) in 6 ms
```

Each step in the output shows how dig queries different DNS servers iteratively to resolve the domain name.

## DNS Caching

### Task 8

`TASK 8:` <span style="color:#f77729;"><b>Without</b></span> using recursion, query your default (local) DNS server for information about `www.dmoz.org`. 
{:.task}

Answer the following questions for your own practice:

- What is the command that you used?
- Did your default server have the answer in its cache?<span style="color:#f77729;"><b> </b></span>How did you know?
- How <span style="color:#f77729;"><b>long</b></span> did the query take?

### Task 9

`TASK 9:` Query your default DNS server for information about the host in the previous question, using the recursion option this time. How long did the query take? (is it faster or slower than the previous task?)
{:.task}

### Task 10

`TASK 10:` Query your default DNS server for information about the same host without using recursion. How long did the query take?
{:.task}

{:.highlight-title}
> Question
> 
> Has the cache served its purpose? Think about the reason(s) why.

### Task 11

`TASK 11:` Try to query about `www.singtel.com` as such: `dig www.singtel.com +nocmd +noall +answer`. Take note of the output. Then, <span style="color:#f77729;"><b>wait</b></span> a few seconds and type the query <span style="color:#f77729;"><b>again</b></span>.
{:.task}

Repeat the query a few times after waiting for a few seconds. Also, search what each <span style="color:#f77729;"><b>query option</b></span> does. You might have such output:
<img src="{{ site.baseurl }}/assets/images/nslab3/7.png"  class="center_seventy no-invert"/>

{:.highlight-title}
> Question 
> 
> Do you observe a _pattern_ in the `TTL` field? (e.g: `TTL` value is reducing or increasing, or wrapped around).

# Wireshark

<span style="color:#f77729;"><b>Wireshark</b></span> is a powerful tool used to capture packets sent over a network and analyse the content of the packets retrieved.

## Download dnsrealtrace.pcapng

Download this file: [dnsrealtrace.pcapng](https://drive.google.com/file/d/118Z03KnN7mNchsIs3G-DUdtf1zJV3NVI/view?usp=sharing). We will use it in this lab. It contains a <span style="color:#f77729;"><b>trace</b></span> of the packets sent and received when a web page is downloaded from a web server over the SUTD network.

{:.new-title}
> Fun fact
> 
> In the process of downloading the web page, DNS is used to find the IP address of the server.

If you prefer to download the file from the CLI, enter the command:

```
wget "https://drive.google.com/uc?export=download&id=118Z03KnN7mNchsIs3G-DUdtf1zJV3NVI" --output-document dnsrealtrace.pcapng
```

## Install Wireshark

Wireshark is a network protocol analyzer. Install wireshark from its [official homepage here](https://www.wireshark.org/download.html).

If you use Ubuntu (GUI enabled), run the following commands in install wireshark. You can then run wireshark with `wireshark`.

```
sudo add-apt-repository ppa:wireshark-dev/stable
sudo apt update
sudo apt install wireshark
sudo usermod -aG wireshark $(whoami)
```

If your system doesn't support GUI, you can install [tshark](https://tshark.dev/setup/install/) and [termshark](https://termshark.io) instead:

```
sudo add-apt-repository -y ppa:wireshark-dev/stable
sudo apt install -y tshark
sudo usermod -a -G wireshark $USER
sudo apt install termshark
```

The rest of this lab is written with the assumption that you used Wireshark. Other equivalent network protocol analyser should have similar functionalities.

## Inspect Capture File

Open the `dnsrealtrace.pcapng` in Wireshark and answer the following questions. You can refer to a short Wireshark tutorial [here](https://drive.google.com/file/d/12zi50lKYTf6ebXQNbUJsstc_BBSWO6X6/view?usp=sharing) before proceeding, but most things are self-explanatory.

After opening the file, you should have this interface:

<img src="{{ site.baseurl }}/assets/images/nslab3/5.png"  class="center_full no-invert"/>

If you use `termshark`, you can enter the following command in the directory where the downloaded capture file reside:

```
termshark -r dnsrealtrace.pcapng
```

<img src="{{ site.baseurl }}//assets/images/lab3_3-wireshark/2023-06-28-17-13-32.png"  class="center_seventy no-invert"/>

### Task 12

`TASK 12:` <span style="color:#f77729;"><b>Locate</b></span> the DNS query and response messages. Are they sent over `UDP` or `TCP`?
{:.task}

{:.highlight-title}
> Question
> 
> Which numbers are these DNS query packets? Hint: look under protocol <span style="color:#f77729;"><b>DNS</b></span>

### Task 13

`TASK 13:` What is the <span style="color:#f77729;"><b>destination</b></span> port for the DNS _query_ message? What is the <span style="color:#f77729;"><b>source</b></span> port of the DNS _response_ message?
{:.task}

### Task 14

`TASK 14:` What is the IP address to which the DNS query message was sent? Run `scutil --dns` (macOS) or `cat /etc/resolv.conf` (Ubuntu) to determine the `IPv4` address of your _local_ DNS server. Are these two addresses the same? For Windows users, google it yourself 😄.
{:.task}

### Task 15

`TASK 15:` Examine the <span style="color:#f77729;"><b>second</b></span> DNS query message in the Wireshark capture. What <span style="color:#f77729;"><b>type</b></span> of DNS query is it?
{:.task}

{:.highlight-title}
> Question
> 
> Does the query message contain any answers?

Then examine the second DNS **response** message.

- How many answers are provided?
- What does each of these answers contain?

### Task 16

`TASK 16:` Locate a `TCP SYN` packet sent by your host subsequent to the above (second) DNS response.
{:.task}

This packet opens a `TCP` <span style="color:#f77729;"><b>connection</b></span> between your host and the web server. Does the <span style="color:#f7007f;"><b>destination</b></span> IP address of the `SYN` packet correspond to any of the IP addresses provided in the DNS response message?

## Optional Activity

Capturing packets for packet analysis with wireshark:

1. Once the program is launched, select the <span style="color:#f77729;"><b>network interface</b></span> to capture and click on the _sharkfin_ icon at the top left of the application right under the menu bar to begin capturing packets. If you click on each packet, you can see each layer's header and the application layer payload.
   <img src="{{ site.baseurl }}/assets/images/nslab3/6.png"  class="center_full no-invert"/>

2. To explore the interface, mention the interface (e.g. `eth0`, `wlan`) in the capture option.

3. There are display filters to analyse the packets.
   - <span style="color:#f77729;"><b>Protocols</b></span>: TCP, UDP, ARP, SMTP, etc.
   - Protocol <span style="color:#f77729;"><b>fields</b></span>: port, src.addr, length, etc. (E.g. `ip.src == 192.168.1.1`)
   - For more detailed instructions on Wireshark, refer to its [official homepage](https://www.wireshark.org/)
