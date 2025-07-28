---
layout: default
permalink: /ps/10-dns
title: Internet Naming and Addressing 
parent: Problem Set 
nav_order:  10
---
* TOC
{:toc}

**50.005 Computer System Engineering**
<br>
Information Systems Technology and Design
<br>
Singapore University of Technology and Design
<br>
**Natalie Agus (Summer 2025)**

# Internet Naming and Addressing 
{: .no_toc}

## The Diagrammed Delay

### Background: DNS Query Types

DNS resolution can happen in two main query types:

* **Recursive Query**: The client asks a DNS server to resolve the full query and return the final answer. The contacted DNS server does all the querying work.
* **Iterative Query**: The server returns the best answer it has, typically a referral to a more authoritative DNS server. The client must continue the resolution process.

Clients like browsers typically send **recursive** queries to a **local DNS server**, which then performs **iterative** queries on their behalf. Understanding who queries whom (and in what order) is key to debugging DNS resolution issues.

### Scenario

You are shown the following partial time-space diagram of DNS query resolution for `www.moviehub.net`.

```
Time ↓

User's Browser        Local DNS         Root DNS        .net TLD        ns1.moviehub.net

    |                     |                 |               |                   |
1.  |--- Query: www.moviehub.net ---------->|               |                   |
2.  |                     |--- Query: .net? --------------->|                   |
3.  |                     |                (Responds .net NS: 192.0.2.2)        |
4.  |                     |--- Query: moviehub.net ------->|                   |
5.  |                     |                (Responds with NS: ns1.moviehub.net)|
6.  |                     |--- Query: www.moviehub.net ------------------------>|
7.  |                     |<-- IP: 203.0.113.77 --------------------------------|
8.  |<-- IP: 203.0.113.77 ------------------|               |                   |
```

{: .note}
This diagram ends at the point the final IP address is returned. You are told the user typed the domain in a browser, and did not use any CLI tool.

**Answer the following questions**:
1. Is the query initiated by the browser recursive or iterative? Justify your answer using steps from the diagram.
2. Which component in this diagram performs the majority of the DNS resolution work? Explain.
3. What is the role of the `.net` TLD server in this flow?
4. Suppose the local DNS resolver does **not** support recursion.
   a. How would the browser have to behave differently?
   b. Draw the new message flow between the browser and external DNS servers.
5. Briefly explain two tradeoffs between using a recursive DNS resolver vs. having the client perform iterative queries.

{: .highlight}
> **Hints**:
> * Focus on whether the *client* receives a final answer vs. multiple referrals
> * Look at who contacts the TLD and authoritative servers
> * Think about network overhead and caching potential


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The query from the browser is a recursive query. This is clear from the fact that the browser sends a single request to the local DNS server and receives the final IP address as a response, without directly communicating with any other DNS servers. All the intermediate steps are handled by the local DNS server, which performs the full resolution on the client’s behalf.
  </p>

  <p>
    Most of the DNS resolution work is carried out by the local DNS server. It queries the root server, the TLD server for <code>.net</code>, and the authoritative server for <code>moviehub.net</code>. It collects the necessary referrals and ultimately retrieves the final A record for the requested hostname.
  </p>

  <p>
    The <code>.net</code> TLD server plays a critical role in directing the query closer to its destination. It does not provide the IP address of <code>www.moviehub.net</code> directly, but it informs the resolver which name server is authoritative for the <code>moviehub.net</code> domain by returning an NS record.
  </p>

  <p>
    If the local DNS server does not support recursion, then the client (in this case, the browser) must take over the resolution process. It would receive referrals instead of final answers, and would need to send its own queries to each DNS server along the path. The flow would look like this: the browser queries the root server for the TLD, receives the address of the TLD server, then queries the TLD server for the domain, receives the authoritative server’s name, and finally queries that server for the IP address of the hostname.
  </p>

  <p>
    Using recursive resolution offloads work from the client and centralizes caching and query logic in the local DNS server. This improves performance and reduces complexity on the client side. However, it also places more responsibility and load on the resolver. In contrast, iterative queries distribute the workload and give the client more control, but they require more DNS knowledge, network traffic, and careful handling of responses. Each method has tradeoffs in terms of simplicity, scalability, and fault tolerance.
  </p>
</div>



## A Misleading Alias

### Background: CNAME Resolution

A **CNAME (Canonical Name)** DNS record maps one domain name to another. It functions as an alias. When a DNS client receives a CNAME record, it must perform a new DNS query to resolve the canonical name into an IP address.

CNAMEs simplify DNS configuration by avoiding duplication of A records. However, they introduce additional queries. If the canonical name does not resolve properly, the alias will fail as well.

### Scenario

A user tries to visit `news.campusnet.sg` in a browser. The DNS response is:

```
news.campusnet.sg    CNAME    internal.campusnet.sg
```

The browser displays **"Server Not Found."**

The student is confused. They believe the DNS worked, since the name resolved to another name.

**Answer the following questions**:

1. After receiving this CNAME record, what does the client need to do next?
2. List two reasons why the browser would still fail to load the page, even though the initial DNS query returned a result.
3. Suppose `internal.campusnet.sg` does not have an A record but instead has its own CNAME pointing elsewhere. What happens in that case? Is there a limit to how many CNAMEs can be chained?
4. If `internal.campusnet.sg` is an internal-only name that can only be resolved within the campus network, what happens when someone off-campus tries to access `news.campusnet.sg`?
5. Why might network administrators choose to use CNAME records instead of duplicating A records across multiple subdomains?

{: .highlight}
> **Hints**:
> * The CNAME is not the final answer. The client must resolve the new name as well.
> * DNS resolution can fail at any step in the chain.
> * Internal hostnames are often not exposed to public DNS resolvers.



<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The presence of the SOA record shows that <code>mystuff.xyz</code> is a valid DNS zone. This record provides essential administrative details about how the zone is maintained, including which server is considered the primary authority and who is responsible for managing the domain.
  </p>

  <p>
    In this specific record, <code>ns1.registrar.com.</code> is the primary authoritative DNS server, and <code>admin.mystuff.xyz.</code> is the zone administrator's contact email (formatted with a dot in place of the @ symbol). The number <code>2025061201</code> is the serial number, which increases each time the zone file is updated. This helps secondary DNS servers detect changes and trigger a zone transfer when needed.
  </p>

  <p>
    The values <code>7200</code>, <code>3600</code>, and <code>1209600</code> represent the refresh interval, retry interval, and expiry time for secondary servers. These control how frequently secondaries check for updates, how long they wait before retrying after failure, and how long they retain data if the primary becomes unreachable. The final number, <code>300</code>, is the negative TTL, which tells resolvers how long they may cache a negative (non-existent domain) response.
  </p>

  <p>
    Even when a subdomain does not exist, DNS responses often include the SOA record of the zone. This allows clients to cache the “not found” result for the duration specified by the negative TTL, which reduces repeated unnecessary queries.
  </p>

  <p>
    The SOA record is always placed at the beginning of a DNS zone file because it defines the zone's identity and controls how other DNS servers interact with it. Without an SOA record, the zone would not support replication, change tracking, or coordinated administration.
  </p>
</div>





## Rooting Your Domain From Home 
### Background: Domain Registration and Hosting Your Own DNS

To make a domain like `pixelpeek.dev` point to your own home-hosted website, you need two things:

1. **A domain registrar**: to buy a domain name **and** inform the TLD (`.dev`) which DNS server is authoritative for it.
2. **An authoritative DNS server**: running somewhere publicly reachable, to respond to queries about your domain.

You can run a DNS server from your home network but that requires a **public static IP address** and port forwarding. Otherwise, DNS resolvers won't be able to find or reach your name server.

### Scenario

You buy `pixelpeek.dev` from **Namecheap**, a popular domain registrar.

You have fiber broadband at home and pay for a **static public IP address** from your ISP: `203.0.113.42`.

You install **BIND9** on your Linux desktop at home, planning to run an authoritative DNS server at `ns1.pixelpeek.dev` from your own IP.

Your goal:

* Map `www.pixelpeek.dev` and `admin.pixelpeek.dev` to your home server
* Serve all DNS queries yourself using your personal DNS server

**Answer the following questions**:
1. What steps must you take on Namecheap to make your domain use your DNS server?
2. Why must you provide a glue record for `ns1.pixelpeek.dev`?
3. What entries must exist in your zone file for `pixelpeek.dev`?
4. How do DNS resolvers worldwide learn that `ns1.pixelpeek.dev` is in charge of your domain?
5. What are the limitations or risks of hosting your own DNS server from home?


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    On Namecheap’s dashboard, you need to go to your domain settings and select “Custom DNS” under Nameservers. Enter <code>ns1.pixelpeek.dev</code> as your authoritative name server. Because <code>ns1.pixelpeek.dev</code> is part of the domain it serves, Namecheap will also ask for the IP address. This is where you provide the <span class="orange-bold">glue record</span>: <code>203.0.113.42</code>.
  </p>

  <p>
    A glue record is required because resolvers cannot resolve <code>ns1.pixelpeek.dev</code> unless they already know its IP. The TLD <code>.dev</code> stores and serves this glue IP as part of the NS delegation, so resolvers can reach your server directly.
  </p>

  <p>
    Your zone file should include:
    <code><br />
    pixelpeek.dev.         IN  NS   ns1.pixelpeek.dev.<br />
    ns1.pixelpeek.dev.     IN  A    203.0.113.42<br />
    www.pixelpeek.dev.     IN  A    203.0.113.42<br />
    admin.pixelpeek.dev.   IN  A    203.0.113.42<br />
    </code>
  </p>

  <p>
    When someone queries <code>www.pixelpeek.dev</code>, their resolver first checks the root servers, which refer them to the <code>.dev</code> TLD. The TLD server sees your NS record and glue, then tells the resolver to ask <code>ns1.pixelpeek.dev</code> at <code>203.0.113.42</code>. From there, your home server handles the rest.
  </p>

  <p>
    Hosting DNS from home is <span class="orange-bold">fragile</span>. If your power, internet, or computer goes down, your domain becomes unreachable. Residential ISPs may also block port 53 (used for DNS), and many don’t provide truly static IPs. In production, DNS is usually hosted on redundant servers across different networks.
  </p>
</div>



## Lonely Authority

### Background: SOA Record

Every DNS zone must begin with a **Start of Authority (SOA)** record. This record contains administrative information about the zone, including:

* The **primary authoritative name server**
* The **email address** of the zone administrator (formatted with `.` instead of `@`)
* The **serial number** to track updates
* The **refresh**, **retry**, and **expire** intervals for secondary DNS servers
* The **negative TTL**, which defines how long to cache non-existent domain responses

SOA records are **not used for hostname resolution**, but are **essential** for zone transfer, update coordination, and administrative control.

### Scenario

A student queries `dig SOA blog.mystuff.xyz` and receives the following response:

```
;; ANSWER SECTION:
mystuff.xyz.    3600  IN  SOA  ns1.registrar.com.  admin.mystuff.xyz.  2025061201  7200  3600  1209600  300
```

They ask:
"This doesn’t give me any IP address. Why is this even useful?"

**Answer the following questions**:
1. What does the presence of an SOA record indicate about `mystuff.xyz`?
2. Interpret the fields in this SOA record. What do the numbers represent?
3. What is the purpose of the **serial number**, and how is it used?
4. Suppose you try to resolve a subdomain that does not exist. Why might the DNS response still contain the zone’s SOA record?
5. Why is the SOA record always the first entry in a zone file?

{: .highlight}
> **Hints**:
> * SOA is about **administration**, not resolution.
> * DNS zones use the SOA to coordinate updates and caching behavior.
> * Negative answers often include SOA for **negative caching**.


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p><strong>1.</strong> It shows that <code>mystuff.xyz</code> is a defined DNS zone, and the record provides administrative metadata about that zone.</p>

  <p><strong>2.</strong> 
    <ul>
      <li><code>ns1.registrar.com.</code>: Primary authoritative DNS server</li>
      <li><code>admin.mystuff.xyz.</code>: Administrator email (written with <code>.</code> instead of <code>@</code>)</li>
      <li><code>2025061201</code>: Serial number (used to indicate updates to the zone)</li>
      <li><code>7200</code>: Refresh time (how often secondary servers check for updates)</li>
      <li><code>3600</code>: Retry time (how long to wait before retrying if refresh fails)</li>
      <li><code>1209600</code>: Expire time (how long to keep data if primary is unreachable)</li>
      <li><code>300</code>: Negative TTL (how long to cache a "not found" result)</li>
    </ul>
  </p>

  <p><strong>3.</strong> The serial number helps secondary DNS servers determine whether the zone file has changed. If the number is higher than before, they trigger a zone transfer to sync their copy.</p>

  <p><strong>4.</strong> DNS responses to non-existent domains often include the zone’s SOA record so that the resolver can apply **negative caching** based on the SOA’s TTL settings.</p>

  <p><strong>5.</strong> The SOA record is required as the first record in a zone file. It defines who controls the zone and how it should behave. Without it, the zone cannot function properly or support transfers.</p>
</div>






## Silent Switch

### Background: Indirection and Late Binding

DNS provides a level of **indirection** by translating names into IP addresses only when needed. This allows for **late binding**, where the actual server behind a domain name can be changed at runtime without affecting the domain itself. This is useful for load balancing, redundancy, and maintenance.

The DNS **Time to Live (TTL)** controls how long a resolved record stays cached. Once the TTL expires, the name must be resolved again, potentially giving a different answer.

### Scenario

On Monday, `library.univ.edu` resolves to IP address `192.168.5.10`. On Tuesday, it resolves to `192.168.5.88`. No configuration changes were made on your laptop between these days. You did not flush the cache manually.

Yet, when you run `dig library.univ.edu`, you get the new IP.

**Answer the following questions**:
1. Why did the IP address change, even though you made no changes locally?
2. How is this behavior related to the concept of **late binding**?
3. Suppose both IPs point to load-balanced servers. How can DNS help distribute traffic across them?
4. What would happen if the TTL for this domain was set to 86400 seconds? Would you still have seen the change the next day?
5. Why is indirection via DNS a good design for maintaining large-scale systems?

{: .highlight}
> **Hints**:
> * TTL determines how long a DNS answer is reused.
> * DNS records can be updated at the authoritative server.
> * Clients only re-query when TTL expires.


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The change in IP address happened because the authoritative DNS server updated the A record for <code>library.univ.edu</code>. Your device did not make any local changes, but once the previously cached DNS record expired due to its TTL, your system automatically queried the DNS again and received the new IP address.
  </p>

  <p>
    This behavior is an example of <strong>late binding</strong>, where the mapping between a name and its actual IP address is determined at the time of use. This gives the system flexibility to move services, replace servers, or distribute load without requiring users to update configurations.
  </p>

  <p>
    If both IP addresses point to functional backend servers, DNS can be configured to return multiple A records using round-robin or geolocation-based methods. This allows incoming traffic to be spread across servers for better performance and reliability.
  </p>

  <p>
    However, if the TTL of the DNS record was set to a very long duration, such as 86400 seconds (24 hours), your system would have continued to use the cached old IP address until the TTL expired. The new IP would only be visible after that point.
  </p>

  <p>
    Indirection through DNS is a powerful design tool. It allows services to change addresses behind the scenes, supports load balancing and failover, and helps decouple users from the underlying infrastructure. This design is especially useful in large, distributed systems where services need to be maintained, scaled, or restructured without user disruption.
  </p>
</div>




## A Fragmented Campus

### Background: Zone Delegation and Glue Records

In the Domain Name System (DNS), large organizations can divide their domain into **zones** for easier administration. A **zone delegation** allows subdomains to be handled by different teams or servers, each managing their own records. For this to work globally, the parent zone must provide **NS records** pointing to the child zone's authoritative servers, and sometimes also include **glue records**  (A or AAAA records) that provide IP addresses for those name servers.

{:.note}
If glue records are missing, DNS resolution may fail due to circular dependencies or the inability to contact the authoritative server.

### Scenario

SUTD's IT department delegates `research.sutd.edu.sg` to a separate research unit. Internally, everyone can access `research.sutd.edu.sg` and `lab1.research.sutd.edu.sg` without problems. But external users report that these addresses do not resolve at all.

> The IT team says, “We’ve already delegated it by adding NS records.”

**Answer the following questions**:
1. Why might external users still fail to resolve the `research` subdomain?
2. What is the purpose of a glue record, and when is it required?
3. Suppose the NS records for `research.sutd.edu.sg` point to `ns.research.sutd.edu.sg`. What problem might this cause during resolution?
4. How can this issue be detected and fixed?
5. Why is zone delegation important for managing large or distributed organizations?

{: .highlight}
> **Hints**:
> * If a name server is *inside* the subdomain it serves, **glue** records are likely needed.
> * Look at where authoritative records live, and whether IP addresses are reachable.
> * Zone = administrative boundary. Domain = logical name structure.


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    External users may fail to resolve <code>research.sutd.edu.sg</code> because their resolvers cannot reach the authoritative name server for the subdomain. Although NS records have been delegated, the parent zone may not have provided the IP address of the delegated server. Without an A or AAAA record for the name server, external resolvers have no way to contact it.
  </p>

  <p>
    A glue record is an A or AAAA record placed in the parent zone that tells resolvers how to reach a child zone’s name server. Glue records are necessary when the NS target lies within the subdomain being delegated. For example, if the NS for <code>research.sutd.edu.sg</code> is <code>ns.research.sutd.edu.sg</code>, you cannot resolve the NS without already being able to resolve the subdomain (a circular dependency). Glue breaks this loop.
  </p>

  <p>
    In this case, the NS record points to <code>ns.research.sutd.edu.sg</code>, but external resolvers cannot get its IP address because that A record is stored inside the same zone that cannot be reached yet. As a result, the chain of resolution fails before it can even reach the delegated server.
  </p>

  <p>
    This can be fixed by inserting a <span class="orange-bold">glue</span> record, specifically, the A record for <code>ns.research.sutd.edu.sg</code> directly into the <code>sutd.edu.sg</code> zone, alongside the NS record. This gives resolvers the IP address they need to proceed with querying the subdomain’s authoritative server.
  </p>

  <p>
    Zone delegation helps break down DNS administration across departments, offices, or subsidiaries. It allows each unit to control its own records independently, improving flexibility and scalability in large or complex organizations. But proper delegation must include both NS and glue records where necessary to ensure global resolvability.
  </p>
</div>





## The Long Path to Mail

### Background: MX Records and Indirection

DNS **MX (Mail Exchange)** records map a domain name to the mail server responsible for receiving email. However, the value of an MX record is not an IP address. Instead, it is another hostname that must be resolved further, typically through an additional A or AAAA lookup.

This layering provides **indirection**, allowing mail hosting to be managed independently of the original domain. However, it also means **email delivery** <span class="orange-bold">depends</span> on multiple successful DNS resolutions.

### Scenario

You run the command:

```bash
nslookup -type=MX deptX.org
```

The result is:

```
deptX.org    MX preference = 10, mail exchanger = mail.protection.outlook.com
```

The student says, "Now we know where to send email." But your team replies, "Not quite yet."

**Answer the following questions**:
1. Why is this response not yet sufficient to send an email?
2. What steps must the client perform after receiving an MX record like this?
3. Could mail delivery fail even though the MX record itself is correct? Explain.
4. How does this indirection help organizations that outsource their email services?
5. How does this setup relate to the idea of DNS as a modular and extensible system?

{: .highlight}
> **Hints**:
> * MX points to a name, not an IP.
> * Another DNS lookup is required.
> * Look for separation of concerns in system design.


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    Although the MX record was successfully retrieved, it only provides a hostname, in this case, <code>mail.protection.outlook.com</code>. This is not an IP address, so additional steps are required. The client must perform another DNS query to resolve this hostname to one or more IP addresses using A or AAAA records before any email connection can be established.
  </p>

  <p>
    Without this second lookup, the email client or SMTP relay has no information about how to reach the actual mail server. Only after resolving <code>mail.protection.outlook.com</code> can the system initiate a connection to deliver email.
  </p>

  <p>
    Mail delivery can still fail even if the MX record exists and is correctly configured. For example, the target hostname may not have a valid A or AAAA record, or the mail server could be down or blocking traffic. In such cases, email delivery attempts will time out or bounce back.
  </p>

  <p>
    This form of indirection is useful because it allows an organization like <code>deptX.org</code> to outsource its email handling to a provider like Microsoft. The domain’s DNS zone only needs to contain an MX record pointing to the external mail service. The email provider can manage its own server addresses and infrastructure separately without needing access to <code>deptX.org</code>'s DNS zone.
  </p>

  <p>
    This design reinforces the modularity of DNS. Each part of the system is responsible for a specific mapping (names to other names) and eventually names to IPs. Indirection helps DNS remain scalable, flexible, and easy to maintain across independent administrative domains.
  </p>
</div>



## Fastest Response Lie

### Background: DNS Caching and Recursion

As you've learned in class, to reduce latency and network traffic, DNS resolvers cache answers for a period of time determined by the **TTL (Time to Live)**. When a user repeats a query, the local resolver may return the answer from cache instead of performing a fresh lookup.

Tools like `dig` support the `+norecurse` flag to bypass recursive querying. If a resolver does not already have the answer cached, and recursion is disabled, the query may return no result at all.

### Scenario

A student says, “`dig nus.edu.sg` always replies instantly, so DNS must be really fast.”

You tell them to try this instead:

```bash
dig nus.edu.sg +norecurse
```

Now the response is either missing, delayed, or empty.

The student is surprised.

**Answer the following questions**:
1. Why was the first query fast, but the second one was slower or empty?
2. What does the `+norecurse` flag change about how the resolver behaves?
3. What determines whether the second query will succeed or fail?
4. What does this experiment reveal about the role of caching in DNS?
5. How can this trick be used to test whether a DNS answer is fresh or from cache?

{: .highlight}
> **Hints**:
> * Caching only works when recursion is allowed or recent answers exist.
> * Local DNS servers do not always have answers unless explicitly asked to fetch them.
> * Try with both common and uncommon domains.


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The first `dig` query returned an answer instantly because the local DNS server already had the result cached from a previous request. It simply reused the stored mapping without performing any new queries across the DNS hierarchy.
  </p>

  <p>
    The `+norecurse` flag tells the resolver not to perform any recursive resolution. This means the resolver will only answer if it already has the record cached. It will not reach out to root, TLD, or authoritative servers to resolve the query if the record is missing.
  </p>

  <p>
    Whether the second query succeeds depends entirely on whether the resolver already has a valid cached entry for `nus.edu.sg`. If it does, it can return the answer immediately. If not, the response will be empty because recursion is disabled and the resolver refuses to fetch the answer on its own.
  </p>

  <p>
    This experiment shows how important caching is in DNS performance. Most DNS queries seem fast because they rely on previously stored answers. Without caching, each resolution would require multiple network hops and referrals, increasing delay.
  </p>

  <p>
    Using `dig +norecurse` is a practical way to test whether a DNS answer is served from cache or generated by a recursive lookup. If the record is missing, and no answer is returned, it confirms that the resolver had to work to fetch the answer the first time.
  </p>
</div>



## Authoritative Enough for You?

### Background: Authoritative vs. Non-Authoritative Responses

DNS servers can either return an **authoritative** answer or a **non-authoritative** answer. Authoritative responses come directly from the domain’s configured DNS servers, while non-authoritative responses are typically cached results returned by a local resolver. 

In `dig`, the presence of the **aa** flag indicates that the response is <span class="orange-bold">authoritative</span>. Querying specific name servers directly allows us to *bypass* local caches and verify the true state of the domain.

### Scenario

You run this command:

```bash
dig google.com
```

You receive six IP addresses, but there is no `aa` (authoritative answer) flag in the response.

You then run:

```bash
dig google.com @ns1.google.com
```

This time, the answer contains fewer IPs, and the `aa` flag is present.

A student asks, “Why are the answers different? Aren’t they both correct?”

**Answer the following questions**:
1. Why did the first query not show the `aa` flag, and what does that imply?
2. Why are the answers different when querying a specific authoritative name server?
3. What does the second response prove that the first one cannot?
4. Why might the local resolver return more IPs than the authoritative server?
5. In what situations is it useful to check for authoritative answers?

{: .highlight}
> **Hints**:
> * Local resolvers aggregate and cache answers.
> * Authoritative servers return only the data they manage directly.
> * Checking `@ns1.google.com` forces a direct query.


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The first query to `dig google.com` returned a non-authoritative response. This means the answer came from a local DNS resolver that had the result cached. Because it did not originate from Google’s actual authoritative servers, the `aa` flag is not set.
  </p>

  <p>
    When you query `@ns1.google.com`, you are directly asking one of Google’s authoritative name servers. This bypasses any local cache and returns the live, authoritative data from Google’s DNS infrastructure. The `aa` flag is set to indicate that this answer is from the source of truth.
  </p>

  <p>
    The second response proves what Google’s name servers are currently serving. This is especially important when verifying whether DNS changes have propagated, or when investigating caching issues. The first response only shows what your local DNS server remembered from a previous query.
  </p>

  <p>
    It is possible that the local resolver returns more IP addresses than the authoritative server in one query. This can happen due to load balancing strategies, round-robin response rotation, or cached records from multiple queries combined. Authoritative servers may limit the number of A records returned per query.
  </p>

  <p>
    Checking authoritative answers is useful during DNS debugging, especially after DNS changes, domain migrations, or suspected misconfigurations. It helps confirm what the domain owner has configured at the source, independently of what might be cached elsewhere.
  </p>
</div>



## Hollow Zones 

### Background: Missing Records and Incomplete Zones

In DNS, a domain is only reachable if all parts of its resolution chain are intact. This includes:

* A record or AAAA record pointing to an IP address
* NS records indicating where the domain is hosted
* A complete zone definition if the domain is delegated

A DNS zone might contain an NS record for a subdomain but omit the necessary A record or zone file. In such cases, the domain appears to exist but fails to resolve.

### Scenario

You try to visit `projects.deptx.edu`, but the browser reports “Server not found.” Curious, you run:

```bash
dig projects.deptx.edu
```

The result shows:

```
projects.deptx.edu.  NS  ns1.deptx.edu.
```

But there is no A record, and the response seems incomplete.

**Answer the following questions**:
1. Why does the presence of an NS record not guarantee that a domain is reachable?
2. What is missing from the DNS configuration in this case?
3. Could the problem be due to improper delegation? How?
4. Why might DNS tools show “some” response, even when resolution ultimately fails?
5. What best practices can prevent this kind of misconfiguration?

{: .highlight}
> **Hints**:
> * NS records without glue or zone data are insufficient.
> * A zone must have an A/AAAA record for each reachable name.
> * Test the authoritative server directly to verify completeness.

<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The presence of an NS record for <code>projects.deptx.edu</code> suggests that the domain has been delegated to a specific name server. However, delegation alone does not make the domain reachable. Without an A record pointing to the actual IP address of the resource, the name cannot be resolved fully.
  </p>

  <p>
    What is missing here is either the A record for <code>projects.deptx.edu</code> itself or a working DNS zone on <code>ns1.deptx.edu</code> that contains that A record. If the name server does not respond authoritatively or does not have any records for the name, resolution will fail even though the NS record exists.
  </p>

  <p>
    Improper delegation can also cause this issue. For example, if <code>projects.deptx.edu</code> was delegated but the delegated server was never set up properly or does not serve the right zone file, then it will not return any useful records. The domain is effectively broken despite appearing in DNS.
  </p>

  <p>
    DNS tools may show partial answers such as NS records, because those exist in the parent zone. However, unless the authoritative server provides the final A record, the lookup remains incomplete. From the client’s point of view, the domain still cannot be resolved into an IP address.
  </p>

  <p>
    To prevent these kinds of errors, DNS administrators should test delegated domains by querying their authoritative servers directly and confirming that A records and supporting data exist. Tools like <code>dig +trace</code> or <code>dig @ns1.deptx.edu projects.deptx.edu</code> help verify that the delegation and data are complete.
  </p>
</div>


## Looks Legit yet Totally Fake 

### Background: DNS Spoofing and Cache Poisoning

DNS responses are not always secure. In traditional DNS, the resolver accepts the first valid-looking response it receives. Attackers can exploit this by sending a fake DNS reply faster than the real one. This is a method known as **DNS spoofing** or **cache poisoning**. Once poisoned, a resolver may serve incorrect answers to all users relying on it.

To prevent this, DNSSEC (DNS Security Extensions) allows resolvers to verify the authenticity of responses using cryptographic signatures.

### Scenario

A user visits `bank.securepay.com`, and their browser loads a suspicious site. On checking with `dig`, the IP address returned is `5.5.5.5`. But when you query an external DNS like Cloudflare’s:

```bash
dig @1.1.1.1 bank.securepay.com
```

You get a completely different IP address.

**Answer the following questions**:
1. What kind of attack might be happening here?
2. How could the attacker make this happen without hacking the actual domain?
3. Why does asking a public DNS like 1.1.1.1 return a different result?
4. How can DNSSEC help prevent this situation?
5. Why is it dangerous to trust DNS responses blindly?

{: .highlight}
> **Hints**:
> * The resolver’s cache can be poisoned without breaching the authoritative server.
> * Responses can be faked if no signature verification is enforced.
> * DNSSEC adds validation, not encryption.


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    This situation is likely caused by a **DNS spoofing or cache poisoning** attack. An attacker may have sent a forged DNS reply to the user's local resolver, tricking it into caching a fake IP address for <code>bank.securepay.com</code>. From then on, all users querying that resolver receive the malicious address.
  </p>

  <p>
    The attacker does not need to compromise the real <code>securepay.com</code> domain. Instead, they just need to guess the transaction ID of a DNS query and race to send a fake response before the real authoritative server does. If the forged response looks valid and arrives first, the resolver may accept it and store the incorrect mapping.
  </p>

  <p>
    Querying a public resolver like <code>1.1.1.1</code> returns the correct address because it uses a separate, unpoisoned DNS path. This confirms that the poisoning is local: it affects the user’s resolver, not the global DNS infrastructure.
  </p>

  <p>
    DNSSEC helps prevent this by allowing resolvers to verify the authenticity of DNS records using digital signatures. If a DNSSEC-enabled resolver receives a forged or altered response, it will detect the mismatch and reject it. This makes poisoning attacks much harder to execute successfully.
  </p>

  <p>
    Blindly trusting DNS responses without validation is risky, especially when the data may come from a compromised cache or intercepted network. Without integrity checks, attackers can reroute traffic, impersonate services, and harvest sensitive information.
  </p>
</div>


## Phantom Email Server

### Background: Mail Delivery and Nonexistent Targets

MX records tell email systems where to send messages for a given domain. However, an MX record that points to a hostname with no corresponding A or AAAA record leads to delivery failure. The DNS may technically be “configured,” but the infrastructure is broken because the final resolution step fails.

This problem is subtle. DNS queries *might* succeed for the MX record itself, but email delivery will still **fail** silently or bounce back.

### Scenario

You configure an MX record for your new domain `paperlaunch.ai`:

```
paperlaunch.ai.  MX  10 mail.paperlaunch.ai.
```

However, emails to `admin@paperlaunch.ai` bounce back with a “host not found” error. You try:

```bash
dig mail.paperlaunch.ai
```

And the response is:

```
;; ANSWER: 0
```

**Answer the following questions**:
1. Why does the MX record alone **not** guarantee successful email delivery?
2. What is the critical missing piece in this configuration?
3. How can you confirm whether a mail exchanger hostname is reachable?
4. Why is this a common misstep for people configuring custom domains?
5. What steps must you take to fix this issue?

{: .highlight}

> **Hints**:
> * MX → hostname → must resolve to an IP.
> * Mail servers depend on final A/AAAA record resolution.
> * DNS completeness matters, not just the presence of MX.

<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The MX record tells sending mail servers to deliver mail to <code>mail.paperlaunch.ai</code>, but this hostname must be resolvable to an IP address for delivery to work. Without an A or AAAA record for that name, the SMTP client cannot establish a connection to send the email.
  </p>

  <p>
    The critical missing piece is the A record for <code>mail.paperlaunch.ai</code>. The DNS setup has declared that this is the mail server, but has not told anyone where that server is, essentially naming a destination that does not exist.
  </p>

  <p>
    You can confirm this by using <code>dig mail.paperlaunch.ai</code> or <code>nslookup</code>. If no IP is returned, then mail clients will fail to resolve the address and will abort the delivery. Even though the MX record itself exists and is correct syntactically, the rest of the resolution chain is broken.
  </p>

  <p>
    This is a common mistake, especially for administrators new to DNS or email hosting. They assume setting an MX record is enough, forgetting that the mail exchanger must also be defined and reachable. The mistake is not immediately obvious unless tested end-to-end.
  </p>

  <p>
    To fix the issue, the administrator must create an A (or AAAA) record for <code>mail.paperlaunch.ai</code> pointing to the actual IP address of the mail server. Once that record is in place, the MX path will be complete and mail delivery should succeed.
  </p>
</div>





## Still Not Found 

### Background: TTL and Negative Caching

DNS records have a **TTL (Time To Live)** that determines how long they should be cached. Once expired, a new lookup must be performed. For non-existent domains, resolvers cache the negative response using the TTL defined in the zone’s **SOA record** (specifically, the “negative TTL” field). This avoids repeated lookups for invalid names.

However, if the SOA record has an *excessively long negative TTL* or is misconfigured, even newly added subdomains may remain inaccessible for some time.

### Scenario

You try to access `newbranch.company.biz`, but receive a “domain not found” error. Later, the admin says, “We added the A record hours ago.”

Still, your laptop refuses to resolve the name.

You run:

```bash
dig newbranch.company.biz
```

And get:

```
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN
;; AUTHORITY SECTION:
company.biz.  86400  IN  SOA  ns1.company.biz. admin.company.biz. ...
```

**Answer the following questions**:
1. Why is your system still showing the domain as nonexistent?
2. What role does the SOA record play in this situation?
3. How is negative caching different from regular TTL caching?
4. How can you force a refresh of this stale negative answer?
5. Why is careful tuning of TTL values important in DNS?

{: .highlight}
> **Hints**:
> * The SOA's last field controls how long “NXDOMAIN” is remembered.
> * Even after a valid A record is added, old NXDOMAIN results may persist.
> * Tools like `dig` show the TTL of cached failures.


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    Your system is still showing the domain as non-existent because it previously received a negative response (NXDOMAIN) when the record truly did not exist. That result was cached, and your resolver is still relying on it, even though the record has since been created.
  </p>

  <p>
    The SOA record plays a crucial role here. It defines the “negative TTL,” which tells resolvers how long to cache the absence of a record. In this case, the <code>86400</code> value means the system may remember the domain as missing for up to 24 hours.
  </p>

  <p>
    Negative caching differs from regular caching in that it stores the fact that a record does not exist. This improves efficiency by avoiding repeated lookups for invalid names. However, it also means that when records are newly added, clients may not see them until the negative TTL expires.
  </p>

  <p>
    To work around this, you can flush your DNS cache using system tools like <code>sudo dscacheutil -flushcache</code> on macOS or <code>ipconfig /flushdns</code> on Windows. This forces your resolver to discard the cached NXDOMAIN response and re-query the DNS hierarchy.
  </p>

  <p>
    TTL values in the SOA should be tuned carefully. Long TTLs reduce traffic and load, but they also slow down propagation of new changes. Shorter TTLs improve responsiveness to updates but may increase query volume. Striking a balance is key to maintaining a responsive and efficient DNS setup.
  </p>
</div>



## Timing a DNS Resolution 

### Background: Recursive Query and Time-Space Diagram

DNS name resolution involves a series of requests across multiple servers. When a user initiates a DNS query, the local resolver may perform a **recursive query**, resolving the name by contacting multiple DNS servers in sequence. Time-space diagrams help visualize the order and timing of these interactions.

### Scenario

A user types `moviehub.net` in their browser. The local DNS resolver performs **recursive resolution** to retrieve the IP address. Assume:

* The resolver has no cache.
* The browser uses the OS resolver, which sends the query to a recursive local DNS server.
* The resolution path involves the root, TLD (.net), and authoritative name server for `moviehub.net`.

**Answer the following questions**:
1. Use the time-space diagram below to identify which server provides the final IP address for `moviehub.net`.
2. Explain how the recursive DNS resolver processes the query based on this diagram.
3. Describe what happens if the resolver had previously cached the result.
4. Why are root and TLD servers designed to only answer iteratively?
5. Why is the direction of slanted arrows important in the diagram?

{: .highlight}
> **Hint**: 
> * In recursive resolution, the client sees only the local resolver. 
> * That resolver performs iterative queries behind the scenes, so responses trickle down from the top hierarchy. 
> * Root and TLD servers typically do not support recursion.

```
Time ↓

Client         Resolver       Root Server     TLD Server      moviehub.net NS
  |                |               |               |                 |
  |───────────────▶|               |               |                 |  (1) Query: moviehub.net
  |                |──────────────▶|               |                 |  (2) Ask root for .net
  |                |               |◀──────────────|                 |  (3) Root replies with TLD info
  |                |──────────────▶|               |                 |  (4) Ask TLD for moviehub.net
  |                |               |               |◀─────────────── |  (5) TLD replies with NS info
  |                |──────────────▶|               |                 |  (6) Ask NS for moviehub.net
  |                |               |               |                 |◀───────────────|  (7) NS replies with A record
  |◀───────────────|               |               |                 |  (8) Resolver returns IP to client
  |                |               |               |                 |
```

<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The final IP address is provided by the <strong>authoritative name server</strong> for <code>moviehub.net</code>, shown in step (7).
  </p>
  <p>
    The resolver receives the user's query and performs a recursive lookup. It queries the root server for .net, then the TLD server, and finally the authoritative server for <code>moviehub.net</code>. Each arrow represents a new iterative query. Once the resolver obtains the A record, it returns the answer to the client.
  </p>
  <p>
    If the resolver had cached the answer, it would have returned the result immediately to the client in step (2), skipping all upstream queries. This demonstrates the importance of DNS caching for performance.
  </p>
  <p>
    Root and TLD servers only answer iteratively to reduce load and keep responsibilities minimal. They return pointers to other servers rather than performing recursion.
  </p>
  <p>
    Slanted arrows indicate the flow of time and request-response delays between entities. They help clarify which server initiates each interaction and how long the resolution path is.
  </p>
</div>

## A Deep Dive to Space and Time

### Background: Recursive Queries and Time-Space Behavior

When a client queries a name that is not cached, the local DNS resolver performs a full **recursive resolution**. It contacts root servers, TLD servers, and authoritative servers step by step, collecting referrals and eventually returning the final answer to the client.

This process can be illustrated using a **time-space diagram**, showing who contacts whom and in what order. Recursive resolution is centralized in the local DNS server, making the client experience simpler at the cost of some delay.

### Scenario

You try to visit `www.moviehub.net`. The name is not cached locally, so a full resolution process occurs. Assume your local DNS server supports recursion.

Here’s a simplified time-space diagram (ASCII art format). We reuse the one from the previous question:

```
Time ↓

Client         Resolver       Root Server     TLD Server      moviehub.net NS
  |                |               |               |                 |
  |───────────────▶|               |               |                 |  (1) Query: moviehub.net
  |                |──────────────▶|               |                 |  (2) Ask root for .net
  |                |               |◀──────────────|                 |  (3) Root replies with TLD info
  |                |──────────────▶|               |                 |  (4) Ask TLD for moviehub.net
  |                |               |               |◀─────────────── |  (5) TLD replies with NS info
  |                |──────────────▶|               |                 |  (6) Ask NS for moviehub.net
  |                |               |               |                 |◀───────────────|  (7) NS replies with A record
  |◀───────────────|               |               |                 |  (8) Resolver returns IP to client
  |                |               |               |                 |


```

{:.note}
For simplicity, we omit the "slantness" of each query/response. There's still network delay for each query and response. 

**Answer the following questions**:
1. Is this query recursive or iterative from the browser’s point of view?
2. Who is doing most of the work in this resolution process?
3. What role does the `.net` TLD server play?
4. What happens if the local resolver does not support recursion?
5. What are the pros and cons of recursive vs. iterative DNS queries?

{: .highlight}
> **Hints**:
> * Recursive = you ask once, someone else does all the lookup steps.
> * Iterative = each referral must be followed manually by the client.
> * Think about caching and client-side simplicity.


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    From the browser’s point of view, this is a **recursive query**. It sends a single request to the local DNS resolver and expects a final answer. The resolver is responsible for handling the multiple lookups behind the scenes.
  </p>

  <p>
    The **local DNS resolver** does most of the work. It contacts the root server, then the `.net` TLD server, and finally the authoritative server for `moviehub.net`, following the chain of delegation to get the final IP address.
  </p>

  <p>
    The `.net` TLD server provides a **referral**. It does not return the final answer, but points the resolver to the authoritative name server for the `moviehub.net` domain. It is one of the steps in the recursive path.
  </p>

  <p>
    If the local resolver does **not support recursion**, the browser or client must handle each referral manually: first querying the root, then the TLD, then the authoritative server. This is called **iterative resolution** and places more burden on the client.
  </p>

  <p>
    Recursive resolution is **simpler for clients** and allows better centralization of caching and logic. However, it makes the resolver more complex and resource-intensive. Iterative resolution is more **flexible and transparent**, but harder for client software to implement well.
  </p>
</div>




## Who Resolves the Resolver?  

### Background: Delegation and Glue Records

DNS delegation allows a parent zone to assign responsibility for a subdomain to another name server. However, if the delegated name server lies within the very subdomain it is meant to serve, a **glue record** (an A record placed in the parent zone) is required. Otherwise, the resolver ends up needing to resolve the same name it's trying to delegate. This results in a circular lookup that cannot complete. You have seen similar question [above](#the-fragmented-campus)

### Scenario

You attempt to access `status.campusnet.org`, but DNS fails. Running:

```bash
dig status.campusnet.org
```

returns:

```
status.campusnet.org.  NS  ns1.campusnet.org.
;; connection timed out; no servers could be reached
```

Checking further:

```bash
dig ns1.campusnet.org
```

yields:

```
ns1.campusnet.org.  NS  ns1.campusnet.org.
```

No A record is returned. No glue record exists either.

**Answer the following questions**:
1. What has gone wrong with this DNS configuration?
2. Why can’t the resolver figure out where `ns1.campusnet.org` is located?
3. What exactly is a glue record, and when is it needed?
4. How should the administrator fix this circular dependency?
5. Would this issue occur if `ns1.campusnet.org` pointed to an external domain like `aws-dns.com`?



<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The problem is that the name server <code>ns1.campusnet.org</code> is inside the domain it is supposed to serve. This creates a circular dependency: to resolve <code>status.campusnet.org</code>, the resolver needs to query <code>ns1.campusnet.org</code>, but resolving <code>ns1.campusnet.org</code> also requires knowing the address of <code>ns1.campusnet.org</code>.
  </p>

  <p>
    The resolver is stuck because there is no A record for <code>ns1.campusnet.org</code>. It has no idea **what** IP address to use to send the query. The delegation points to a name it cannot resolve, effectively breaking the DNS chain.
  </p>

  <p>
    A glue record is an A (or AAAA) record for the delegated name server, included in the parent zone (in this case, <code>org</code> or <code>campusnet.org</code>). It provides the resolver with the IP address of the name server so it can query it directly without having to resolve its name first. Glue records are only needed when the name server is part of the delegated domain itself.
  </p>

  <p>
    To fix the issue, the administrator must provide a glue record in the parent zone. This means adding an A record for <code>ns1.campusnet.org</code> in the <code>campusnet.org</code> zone file, alongside the NS record. Once the IP is available during delegation, the resolver can proceed correctly.
  </p>

  <p>
    This issue would not occur if the NS record pointed to a name outside the domain, such as <code>ns1.aws-dns.com</code>. In that case, resolving the name server does not require access to the zone being delegated, and no glue record is necessary.
  </p>
</div>




## TXT Me If You Can

{:.info}
This question combines knowledge from network security and DNS. 

### Background: DNS Tunneling and TXT Records

DNS is often allowed through firewalls, making it an **attractive** target for covert communication. Attackers can encode data into DNS queries or responses: a technique known as **DNS tunneling**. TXT records are especially vulnerable, since they are designed to carry arbitrary strings and are often ignored by security monitoring tools.

Some malware uses DNS to send stolen data or receive commands, bypassing firewalls by embedding payloads into the domain name or the response data.


### Recap: DNS Record Types
The following are common DNS records that you've seen so far. 
* **A Record**: Maps a domain name to an IPv4 address.
  `www.example.com → 192.0.2.1`

* **AAAA Record**: Like an A record, but for IPv6 addresses.
  `www.example.com → 2001:db8::1`

* **NS Record**: Points to the authoritative name server for a domain.
  `example.com → ns1.dnsprovider.net`

* **CNAME Record**: Creates an alias from one name to another.
  `login.example.com → auth.example.net`

* **MX Record**: Specifies which mail servers handle email for the domain.
  `example.com → mail.example.com (priority 10)`

* **SOA Record**: Stores metadata about the zone, including admin contact, serial number, and TTL rules.

* **TXT Record**: Stores arbitrary text, often for domain verification or service configuration and sometimes misused for covert communication.

### What is a TXT record?

A **TXT (Text) record** is a type of DNS resource record used to store human-readable or machine-readable text in association with a domain name. Originally designed for administrative notes, it is now commonly used to hold structured data for verification or policy purposes.

Examples of TXT record usage include:

* **SPF (Sender Policy Framework):**
  Specifies which mail servers are allowed to send email on behalf of a domain.

  ```
  TXT "v=spf1 include:_spf.google.com ~all"
  ```

* **DKIM (DomainKeys Identified Mail):**
  Stores a public key used to verify signed email headers.

  ```
  TXT "v=DKIM1; k=rsa; p=MIIBIjANBgkqh..."
  ```

* **Domain ownership verification:**
  Used by services like Google, AWS, or GitHub Pages to confirm domain control.

  ```
  TXT "google-site-verification=abc123xyz"
  ```

* **Custom metadata or service hooks:**
  Applications and services may define their own TXT-based configurations.

{: .note}
A domain can have <span class="orange-bold">multiple</span> TXT records. Each record can contain up to 255 characters, but long strings can be split across quoted segments.


### Example: DKIM TXT Record

Suppose your domain is `example.com`, and your mail server signs outgoing emails using a DKIM selector called `mail2025`. It is a label (string) you choose that confims with valid DNS labels (no emojis!). If you're managing your own mail server, you choose the selector which helps identify which key is being used. If you're using a third-party email provider (like Google Workspace, Microsoft 365, or Mailchimp), they usually *assign* the selector and **tell** you what DNS TXT record to add to your domain.

Your DNS would include this **TXT record**:

```dns
mail2025._domainkey.example.com. IN TXT "v=DKIM1; k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQD..."
```

What does this mean? 

* `mail2025` is the **selector** that identifies which DKIM key to use.
* `_domainkey.example.com` is the **required prefix** for DKIM lookups.
* The value inside the TXT record includes:

  * `v=DKIM1`: version
  * `k=rsa`: key type
  * `p=...`: the **public key** used to verify the signature

When a receiving mail server sees an email from `example.com` signed with DKIM, it uses the selector (`mail2025`) to look up this TXT record and validate the signature.


### Why are TXT records risky?

Because TXT records accept arbitrary strings and are rarely parsed strictly, they can be abused:

* Attackers can **exfiltrate data** in chunks using TXT responses.
* Malware can **embed commands or status** into TXT fields.
* Security tools may **ignore TXT records** unless explicitly configured to monitor them.


### Scenario

An analyst notices suspicious DNS traffic from an internal laptop. Instead of querying for typical websites, the laptop sends repeated TXT queries like:

```
query: b2xhdGtleWluZm8xLm1hbHdhcmUudmljdGltLnhtcGF5LmV4ZmlsLmRhdGE.badguy.cn
```

The response contains encoded strings like:

```
TXT "ACK: chunk 3 received"
```

**Answer the following questions**:

1. What is likely happening on this machine?
2. Why would an attacker choose DNS as their communication channel?
3. What makes TXT records useful for exfiltration?
4. How can security teams detect and block this kind of activity?
5. How would DNSSEC or DoH affect such tunneling?


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The machine is likely infected with malware that is using DNS tunneling to exfiltrate data or communicate with a command-and-control server. The queries encode data (possibly in base64) into the subdomains of a domain controlled by the attacker.
  </p>

  <p>
    DNS is often not filtered as strictly as HTTP or SMTP traffic, especially on internal networks. It also works even in locked-down environments where other outbound protocols are blocked. This makes it an attractive covert channel for attackers.
  </p>

  <p>
    TXT records can hold arbitrary strings and are a legitimate part of the DNS protocol (e.g., for SPF). Since they’re often overlooked by security appliances, they can be abused to carry commands or receive acknowledgments from malicious servers.
  </p>

  <p>
    Security teams can detect this activity by monitoring DNS logs for unusually long domain names, high volumes of TXT queries, or queries to domains known to be untrustworthy. DNS rate limiting, blacklisting suspicious domains, and inspecting query content for encoded data can also help.
  </p>

  <p>
    DNSSEC would not stop tunneling, since it focuses on authenticity, not content filtering. However, DNS over HTTPS (DoH) can make it harder for security teams to inspect DNS traffic, as it encrypts queries. In such cases, endpoint monitoring becomes more important.
  </p>
</div>



