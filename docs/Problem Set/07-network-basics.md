---
layout: default
permalink: /ps/7-network-basics-and-hierarchy
title: Network Basics and Hierarchy 
parent: Problem Set 
nav_order:  7
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

# Network Basics and Hierarchy 
{: .no_toc}

## The Vanishing Packet

### Background

Data on the Internet travels in packets through multiple routers. These packets are routed independently and may be dropped along the way. The Internet Protocol (IP) is responsible for delivering packets across networks, but it follows a best-effort delivery model. This means the network makes no guarantees: packets may be lost, duplicated, delayed, or arrive out of order and IP will <span class="orange-bold">not</span> try to detect or fix these issues. 

{:.note}
Higher layers (like TCP in network layer) can add reliability, but at the IP layer, delivery is not assured. Many users often expect the network to be reliable by default, which creates confusion.

### Scenario

A student builds a program to send short messages between two devices across networks. Locally, messages arrive instantly. But when testing between cities, some messages never appear. The student assumes the network is broken.

**Answer the following questions**:
* What does it mean that IP provides “best-effort” delivery?
* Why might a packet not reach its destination?
* What happens inside a router when traffic exceeds capacity?
* Why do longer-distance messages experience more loss?
* What network design decisions reduce the impact of packet loss?

{: .highlight}
> **Hints**:
> * IP makes no delivery guarantees
> * Routers have limited buffer space
> * More distance = more routers = more risk
> * Loss can occur without notification
> * Loss recovery is a separate layer responsibility

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>“Best-effort” delivery means the Internet tries to forward every packet, but does not guarantee it will arrive, arrive in order, or arrive only once. There are no built-in mechanisms to confirm success or detect failure at this level.</p>

<p>A packet might not reach its destination if it is dropped at any point along its route. This can happen due to congestion, hardware failure, or routing issues.</p>

<p>Routers forward packets using **queues**. When incoming traffic exceeds a router’s processing or forwarding capacity, its queue fills up, and excess packets are dropped. This is a common cause of packet loss in the Internet.</p>

<p>Messages sent over longer distances traverse more routers and links. Each additional hop introduces another potential drop point, especially during congestion or poor routing conditions.</p>

<p>To reduce the impact of packet loss, networks use practices such as traffic shaping, queue management, and building redundancy into higher-level protocols. Handling loss typically requires mechanisms outside of basic IP forwarding.</p>


</p></div><br>



## The Disconnected Port

### Background: Ports

Hosts communicate using logical *ports*, which identify services (like web or email) on a machine. However, being “online” doesn't mean others can reach you: routers, firewalls, and NATs (Network Address Translators) can block or hide traffic, especially for incoming connections.

Recall that on the Internet, data travels through routers, which **forward** packets based on **destination IP** addresses. Many home and campus networks use NAT (Network Address Translation) to let multiple devices share one public IP. 


{:.note-title}
> Network Address Translators (NAT)
> 
> NAT tracks outgoing traffic and rewrites addresses so replies can find their way back, but unsolicited incoming packets are typically blocked. 
> 
> NAT is typically implemented on a router or gateway device at the edge of a private network. It modifies packet headers as traffic leaves the network, replacing private IP addresses with the router’s public IP and tracking the mapping in a translation table.

<span class="orange-bold">Firewalls</span> add another layer of protection by filtering traffic based on port numbers and IP addresses, often defaulting to a “deny incoming” rule. Just because a program is listening on a port does not mean it is reachable from outside.

### Scenario

A student runs a web server on their laptop and shares its IP address and port 8080 with a friend. It works fine on campus, but fails when the friend tries to access it from home. The student insists their laptop is “connected” and running fine.

**Answer the following questions**:
* What does it mean for a port to be “reachable” from another network?
* Why might a service be accessible locally but not externally?
* What role does NAT play in blocking inbound traffic?
* How do firewalls impact incoming connection attempts?
* What does it take for a service to be truly accessible over the public Internet?

{: .highlight}
> **Hints**:
> * Private IPs are not directly reachable from the public Internet
> * NAT rewrites IPs and ports to allow outgoing traffic
> * Firewalls often block unknown or unsolicited inbound packets
> * Listening on a port is necessary but not sufficient for global access

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>For a port to be “reachable,” packets sent to that IP:port combination must be delivered all the way to the target machine without being blocked or redirected. This requires proper routing and no interference from NAT or firewall rules.</p>

<p>A service might work locally because devices on the same network can reach each other directly. However, external devices may be unable to reach the private IP address, or may be blocked by NAT or firewall settings.</p>

<p>NAT (Network Address Translation) allows multiple devices to share one public IP address. It typically allows outgoing connections but blocks unsolicited incoming ones unless explicit port forwarding is set up.</p>

<p>Firewalls filter network traffic based on rules. Many default configurations block all incoming connections that aren’t part of an existing outbound session, preventing unexpected external access.</p>

<p>To make a service publicly accessible, the host must have a public IP address or a forwarded port through NAT, the firewall must allow the connection, and the service must be actively listening on that port.</p>


</p></div><br>



## The Recursive Route

### Background: TTL in Network Packets

Routers forward packets using routing tables that map destination IP ranges to next-hop addresses. Each router uses only local information. It does <span class="orange-bold">not</span> know the full end-to-end path. Misconfigured static routes or faulty dynamic updates can create loops, where a packet cycles indefinitely between routers. The Time-To-Live (TTL) field in the IP header prevents this from lasting forever by limiting the number of hops a packet can take.

How router forwarding works in a nutshell:
* Routers forward packets based on a routing table, which maps destination IP address ranges to a next-hop router. 
* Each packet includes an IP header containing metadata used for delivery 
* This includes a field called TTL 
* TTL starts at a **fixed** value and is decreased by 1 at each router. 

If TTL reaches 0, the packet is <span class="orange-bold">discarded</span>. This protects the network from infinite loops, which can happen if routers are misconfigured to forward traffic in circles. We will learn more about this in the following week's lab.

### Scenario

A student sets up two routers in a lab to connect three subnets. They accidentally configure both routers to forward unknown traffic to each other. When testing a ping between devices across subnets, packets vanish and no replies come back.

**Answer the following questions**:
* What causes a routing loop in a packet-switched network?
* How does the TTL field prevent infinite loops?
* Why does the packet “vanish” instead of returning an error?
* How could the student detect that a loop occurred?
* What are safer alternatives to manual static routing?

{: .highlight}
> **Hints**:
> * Routers forward based on longest-prefix match
> * TTL is decremented at each hop and drops the packet at 0
> * Loops may not generate errors unless explicitly logged
> * Diagnostic tools rely on TTL to map routes
> * Dynamic routing protocols detect and resolve inconsistencies

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>A routing loop occurs when two or more routers forward packets to each other in a cycle because each believes the other is a valid next hop for a destination. This creates a forwarding path with no exit.</p>

<p>The TTL (Time-To-Live) field is a counter in each IP packet that decreases by one at every router. When it reaches zero, the packet is discarded. This mechanism prevents indefinite looping.</p>

<p>The packet “vanishes” because it is repeatedly forwarded until its TTL expires, at which point it is silently dropped. Unless a router is configured to generate an error message, the sender may receive no feedback.</p>

<p>To detect a loop, the student could use tools like `traceroute`, which rely on observing TTL expiration at each hop. Repeating hops or timeouts are signs of a loop or black hole.</p>

<p>Safer alternatives to manual static routes include using dynamic routing protocols such as RIP, OSPF, or BGP. These protocols share reachability information and detect loops using path metrics or route poisoning.</p>

</p></div><br>


## The Phantom ISP

### Background

The Internet is built from many independently managed networks called *Autonomous Systems (ASes)*. These are operated by Internet Service Providers (ISPs), which form a global hierarchy. At the bottom are *Access ISPs*, which connect homes, campuses, and businesses. These access networks rely on *Regional* or *National ISPs*, which in turn connect to *Tier-1 ISPs*: large backbone providers with global reach. ISPs exchange traffic using business agreements called *peering* or *transit*. If one ISP path becomes unavailable, traffic may reroute through alternative upstream networks. However, not all paths are symmetrical or available to all regions.

### Scenario
During a regional ISP outage, you and your peers find that they cannot access the university website. However, friends in another country report that it is working fine. The student concludes the university server is down, even though it remains online and operational from other networks.

**Answer the following questions**:
* Why might a website be reachable from one network but not another?
* What role does peering and ISP hierarchy play in this outcome?
* Why does the issue not lie with the university’s server?
* How could the student confirm the website is still online?
* What does this reveal about the structure of the Internet?

{: .highlight}
> **Hints**:
> * ISPs connect through upstream transit providers
> * Not all access networks share the same paths
> * Traffic may reroute around failures only if alternate agreements exist
> * The server can be up even if it is unreachable to some users

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>A website might be reachable from one network but not another due to differences in routing paths between ISPs. If an access ISP loses connectivity to its upstream provider and lacks alternative routes, it cannot reach certain parts of the Internet.</p>

<p>Peering and ISP hierarchy determine how traffic flows between networks. If a regional ISP fails and its upstream Tier-1 connection is lost, traffic may not reroute unless peering agreements or alternate upstreams exist. This limits reachability from specific locations.</p>

<p>The university server is not at fault if it remains reachable from other networks. The issue lies in the student’s ISP path failing to reach the university’s network, possibly due to an upstream outage or missing peering relationship.</p>

<p>The student could confirm the site is online by using external tools like global ping or web proxies, or asking someone on another ISP to try accessing it. These steps test connectivity from outside the affected region.</p>

<p>This situation highlights how the Internet is a loosely connected system of networks. Availability of a site depends not only on the server’s status, but also on the paths between Autonomous Systems and the peering structure among ISPs.</p>
</p></div><br>




## The Hijacked Handshake

### Background: Connection-Oriented Communication

The Internet supports *connection-oriented* communication through protocols that perform handshakes before data transfer begins. A common example is a *three-step handshake*, where the sender and receiver exchange setup messages before a connection is established. Each server listens on a specific port number, which identifies the service it offers (for example, web servers use port 80). When a handshake begins, the operating system creates a temporary record of the connection in a half-open connection queue. This queue is managed by the OS, and has a fixed maximum size. If many handshakes are started but not completed, the queue fills up, preventing the server from accepting new connections. These incomplete attempts can come from unreachable clients or even malicious sources.

{:.note}
In short, if the handshake is never completed, these resources can remain **occupied**. Repeated incomplete handshakes from forged or unreachable sources can exhaust a server’s connection queue and make it unavailable to legitimate users.



### Scenario

A server accepts connections on a well-known port. Suddenly, its connection queue becomes full and it stops responding to new clients. Logs show many connection attempts, but almost none complete the handshake. No large data transfers are observed.

**Answer the following questions**:
* What is the purpose of a three-step handshake in connection setup?
* Why would incomplete handshakes consume server resources?
* How can forged or unreachable IP addresses contribute to the problem?
* What makes this situation a potential denial-of-service attack?
* What countermeasures exist at the system level?

{: .highlight}
> **Hints**:
> * Connection state is reserved early in the handshake
> * Servers maintain a queue of half-open connections
> * Spoofed IP addresses never complete the exchange
> * SYN floods exploit this behavior
> * Operating systems may use techniques to mitigate resource exhaustion

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>A three-step handshake establishes a reliable connection between two hosts. It ensures both sides are ready to communicate and have synchronized state before exchanging data.</p>

<p>When a server receives the first handshake message, it allocates memory and reserves a slot in a pending connection queue. If the handshake is not completed, this slot remains occupied until it times out.</p>

<p>If the handshake request comes from a forged or unreachable IP address, the server waits for a reply that will never arrive. This results in multiple half-open connections that waste server resources.</p>

<p>This behavior can be exploited as a denial-of-service attack known as a SYN flood. Attackers send many fake handshake requests, filling the queue and preventing real clients from connecting.</p>

<p>To mitigate this, operating systems can use techniques such as SYN cookies, which defer resource allocation until the handshake completes, or limit the number of half-open connections per client or globally.</p>

</p></div><br>



## The Lost Domain

### Background

Most Internet communication relies on *Domain Name System (DNS)* resolution. DNS translates human-readable names (like `example.com`) into IP addresses used for routing. When a device joins a network, it usually receives DNS server settings via DHCP. The operating system uses this DNS server to resolve names and may cache results to reduce lookup times. If a device moves between networks or the DNS configuration changes unexpectedly, name resolution may fail even though the Internet connection seems fine.

#### DNS and DHCP 

To access most websites, computers must convert domain names (like example.com) into IP addresses. This is done using the Domain Name System (DNS): a global, distributed lookup *service* (anyone can host a DNS server and that becomes the *service*). When a device joins a network, it is usually assigned a DNS server automatically using a setup protocol called DHCP (Dynamic Host Configuration Protocol). The operating system stores this DNS server setting and uses it for all future lookups. To improve performance, DNS responses are often cached temporarily. However, if the device moves to a different network or keeps outdated DNS settings, name resolution may break even though the Internet connection appears active.

{:.note}
We will learn more about DNS in class in the weeks to come. 

### Scenario

A machine switches from a campus network to a mobile hotspot. It maintains an active connection and can reach known IP addresses, but fails to access websites by name. DNS resolution times out or returns “not found” errors, even for popular sites.

**Answer the following questions**:
* Why does a device need DNS to access most websites?
* What could cause DNS lookups to fail after switching networks?
* Why can known IP addresses still be reached directly?
* How does the OS manage and cache DNS information?
* What can be done to restore working name resolution?

{: .highlight}
> **Hints**:
> * DNS is required to map domain names to IPs
> * DHCP provides DNS server settings during network join
> * DNS responses may be cached per process or system-wide
> * Old configurations may persist after moving networks
> * Flushing cache or renewing DHCP can resolve issues


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>Most websites are accessed using domain names, not raw IP addresses. DNS is used to translate these names into IP addresses that can be routed by the network. Without DNS, the system cannot locate hosts by name.</p>

<p>DNS lookups may fail after switching networks if the system continues to use outdated DNS server settings from the previous network. This can happen if the DHCP renewal does not update the resolver configuration properly.</p>

<p>Known IP addresses can still be reached directly because DNS is only needed for name resolution. If the application uses an IP address directly, no DNS query is required.</p>

<p>The operating system typically caches DNS results to speed up future lookups. Depending on the OS, these entries may persist temporarily even after a network change. Some browsers and applications also maintain their own DNS caches.</p>

<p>To fix DNS issues, the user can flush the DNS cache, restart the network interface, or renew the DHCP lease to get fresh DNS server settings. Manually setting a known working DNS server is another possible solution.</p>



</p></div><br>





## The Unknown Middleman

### Background: IXPs

Internet traffic often passes through shared infrastructure known as *Internet Exchange Points (IXPs)*. These are physical facilities where multiple networks, including ISPs and content providers, interconnect to exchange traffic. While the logical view of the Internet may suggest a direct connection between source and destination, actual traffic paths depend on routing agreements, geography, and cost. Packets may travel through distant or unexpected locations before reaching their target. This routing behavior is invisible to end users unless explicitly traced.

### Scenario

A device in Singapore connects to a nearby cloud server, expecting a low-latency path. However, measurements show the traffic is routed through Tokyo before returning to Singapore. The application experiences lag despite both endpoints being in the same country.

**Answer the following questions**:
* Why might traffic between two nearby devices be routed through a distant location?
* What is the role of IXPs in shaping Internet paths?
* How do ISP agreements influence routing decisions?
* Why does the physical location of a server not always guarantee the shortest path?
* What tools can be used to observe or confirm the path taken?

{: .highlight}
> **Hints**:
> * Routing decisions depend on ISP policies and peering
> * IXPs serve as connection hubs between networks
> * Some paths are chosen for cost or reliability, not distance
> * Traceroute reveals actual packet hops
> * Traffic paths may not match geographic expectations

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>Traffic between two nearby devices can be routed through a distant location due to the structure of ISP interconnections. The routing decision is not always based on physical distance, but on peering relationships and network policies.</p>

<p>IXPs are physical exchange points where multiple networks connect and hand off traffic. They help optimize routes and reduce latency, but are only effective if the involved ISPs participate in the same exchange point.</p>

<p>ISP agreements — including peering, transit, and pricing — heavily influence which path traffic takes. ISPs may prefer cheaper or more stable routes over shorter geographic ones.</p>

<p>Even if a server is physically nearby, its traffic may enter a different network that does not peer locally with the user’s ISP. This forces packets to take a longer international detour before reaching the destination.</p>

<p>Tools such as `traceroute` can show each hop a packet takes to reach its destination, helping users understand why traffic might appear indirect or experience unexpected latency.</p>




</p></div><br>


## The Borrowed Address

### Background: Public and Private IP Addresses

Every device on a network must have a unique *IP address* to communicate. These addresses are divided into public and private ranges. *Public IP addresses* are globally unique and routable on the Internet. *Private IP addresses* are reserved for local use within private networks (such as homes or offices) and are not routable over the Internet. To connect private devices to the wider Internet, routers use *Network Address Translation (NAT)*, which rewrites packet headers so many internal devices can share one public IP. If two networks assign the same private IP range without coordination, conflicts can occur when connecting between them.


#### IP Addresses Details 
IP addresses are numerical labels assigned to devices in a network. The most common format is IPv4, which uses **32 bits** divided into four 8-bit sections called *octets*, written in *dot-decimal notation* (e.g., `192.168.1.5`). Each octet ranges from 0 to 255, allowing for about 4.3 billion unique combinations globally. IP addresses identify both a device and the *network* it belongs to. This is done using a *subnet mask*, which separates the **network portion** (the higher-order bits) from the **host portion** (the lower-order bits). For example, the address `192.168.1.5/24` means the first 24 bits (`192.168.1`) identify the network, and the last 8 bits identify the host within that network. The `/24` is called *CIDR (Classless Inter-Domain Routing)* notation. Devices must have unique IP addresses within the same network to avoid address conflicts.


### Scenario

Two machines on different networks are both assigned the same private IP address range. When one machine connects to the other through a VPN, neither can reach the other by IP, and routing fails silently.

**Answer the following questions**:
* Why must IP addresses be unique for reliable communication?
* What is the difference between public and private IP ranges?
* Why does address duplication cause problems across networks?
* How does NAT interact with private address spaces?
* What strategies can prevent this kind of IP conflict?

{: .highlight}
> **Hints**:
> * Private addresses are reused in many networks
> * NAT hides private IPs behind one public IP
> * Overlapping private ranges confuse routing tables
> * VPNs may use address translation or reallocation to avoid conflict

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>IP communication relies on each device having a unique address so that packets can be routed correctly. If two devices on different networks use the same address, routing becomes ambiguous and can fail.</p>

<p>Public IP addresses are globally assigned and reachable from any Internet-connected network. Private IP ranges, such as 192.168.0.0/16 or 10.0.0.0/8, are meant only for internal use and are not visible on the global Internet.</p>

<p>When two connected networks use the same private IP space, the system cannot determine which device is meant when routing to a given IP. This results in dropped packets or misrouted traffic.</p>

<p>NAT allows many devices in a private network to share one public IP address. It works by rewriting addresses and tracking connections. However, NAT cannot resolve conflicts when two networks have overlapping internal addresses.</p>

<p>To avoid such conflicts, VPNs often use distinct address pools or reassign IPs during connection. Network administrators can also coordinate to ensure different sites use non-overlapping private ranges.</p>
```

Let me know if you're ready to continue with **#9 – The Shrinking Mask** (subnetting).

</p></div><br>


## The Shrinking Mask

### Background

Subnetting is the process of dividing a larger IP network into smaller segments. This is controlled using a *subnet mask*, which specifies how many bits of an IP address represent the network and how many bits represent the host. Subnet masks are written using CIDR notation, such as `/24`, which means the **first 24 bits define the network** and the remaining **8 bits define individual hosts**. A larger subnet (like `/22`) supports more hosts, but requires all devices to agree on the boundary between network and host bits. If subnet masks are misconfigured or inconsistent, devices may misclassify peers as being “outside” the network and fail to communicate directly.

{:.note}
It does <span class="orange-bold">not</span> create more IP addresses. The total number of addresses is fixed by the IP version and network assignment. 

Instead, subnetting changes how those addresses are *grouped* and *interpreted*. The subnet mask defines the boundary between the network and host portions of an address. For example, the address block `192.168.0.0/22` contains 1024 IP addresses, from `192.168.0.0` to `192.168.3.255`. This same range could also be split into four smaller `/24` subnets (`192.168.0.0/24`, `192.168.1.0/24`, etc.), each with 256 addresses. Whether grouped as one large subnet or several smaller ones, the total number of IPs remains the same. 

This example should illustrate clearly how changing the mask from `/24` to `/22` does <span class="orange-bold">not</span> generate more addresses it just combines what was previously segmented.

Mismatched subnetting leads to misclassification of peers, causing communication failures.

### Scenario

A device is reconfigured to use a `/22` subnet mask instead of the original `/24`, to "expand the network." After the change, some hosts become unreachable and local communication fails unexpectedly.

**Answer the following questions**:
* What does a subnet mask control in IP addressing?
* How does changing from `/24` to `/22` affect address assignment?
* Why would inconsistent masks across devices cause communication issues?
* What types of problems can arise from overlapping or misaligned subnets?
* How can subnetting be verified and corrected?

{: .highlight}
> **Hints**:
> * The subnet mask defines the boundary between network and host
> * Larger subnets include more IPs but require coordination
> * Devices may disagree on whether an IP is “local” or “remote”
> * Routing tables depend on correct mask interpretation
> * Tools like `ip addr`, `ip route`, and ping can help diagnose

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>A subnet mask determines which portion of an IP address represents the network and which portion identifies a host within that network. It controls how devices interpret address ranges.</p>

<p>Changing from `/24` to `/22` expands the addressable range from 256 to 1024 possible hosts. This allows more devices to be grouped under the same network, but only if all devices agree on the new mask.</p>

<p>If subnet masks are inconsistent, devices may treat the same IP address differently — one may think it is on the local network, while another believes it requires routing. This causes miscommunication and failed local delivery.</p>

<p>Misaligned subnets can lead to unreachable hosts, incorrect routing, or silent packet drops. These problems are difficult to detect without examining subnet configurations closely.</p>

<p>To resolve these issues, ensure all hosts in the same subnet use the same CIDR mask. Use diagnostic tools to confirm interface settings and test connectivity within the expected address range.</p>
</p></div><br>



## The Misrouted Reply

### Background

NAT (Network Address Translation) is commonly performed by a **router** or gateway that connects a private network to the public Internet. 

In most homes and offices, the *Wi-Fi router acts as the NAT device*. It allows **multiple** internal devices using private IP addresses to share a **single** public IP. When a device on the private network sends data to the Internet, the NAT router *rewrites* the source IP and port to its own public address and remembers the mapping in a connection table. When the reply comes back, the NAT uses this mapping to forward the response to the correct internal machine. This only works if the reply returns through the same device that created the mapping. If the reply comes back through a different NAT router that does not have the entry, it cannot forward the packet, and the reply is silently dropped. This failure is caused by asymmetric routing, where the outgoing and incoming paths do not match.

{:.note}
Unlike subnetting, which only restructures how IP addresses are grouped, NAT allows many devices to share a single public IP, effectively **increasing** the number of usable addresses for everyday use. For example, most home routers assign internal devices addresses like `192.168.0.x`, which belong to a private IP range. Every house can reuse the same `192.168.0.1` to `192.168.0.254` addresses without conflict, because NAT **hides** these private IPs behind the router’s one public IP. The router keeps track of each connection and rewrites addresses as needed, making it appear to the Internet as if all the traffic came from one source. 

This makes NAT a practical solution for conserving the limited pool of IPv4 (32-bits) addresses.


### Scenario

A private network connects to the Internet using two NAT gateways for redundancy. A packet is sent through one gateway, but the reply is routed through the second gateway. Although the external server sends a valid reply, the original sender never receives it.

**Answer the following questions**:
* How does NAT track and forward reply traffic correctly?
* Why does asymmetric routing confuse NAT behavior?
* What happens to a packet that arrives at a NAT without a mapping?
* Why are replies dropped even if the destination IP looks correct?
* What solutions exist to handle redundant NAT setups safely?

{: .highlight}
> **Hints**:
> * NAT uses connection tracking tables for each outbound packet
> * Replies must return through the same gateway
> * Unrecognized replies are dropped for security
> * Load balancing across gateways must preserve session state
> * Symmetric routing ensures NAT consistency


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>NAT tracks outbound connections by storing a mapping between the internal IP and port and their translated public equivalents. This mapping lets the NAT device recognize incoming replies and forward them to the correct internal host.</p>

<p>Asymmetric routing confuses NAT behavior because the reply returns through a different gateway than the one that created the original mapping. The second gateway does not recognize the connection and cannot forward the packet properly.</p>

<p>When a NAT device receives a reply for which it has no corresponding entry in its translation table, it drops the packet. This protects the network from unsolicited or mismatched traffic.</p>

<p>Even if the destination IP in the reply matches the public address, NAT requires more information to identify the correct internal device. Without the original source port and connection context, the reply cannot be routed correctly.</p>

<p>To prevent this issue, network administrators can enforce symmetric routing using source-based routing rules or configure state sharing between NAT gateways. This ensures all packets for a session enter and exit through the same device.</p>

</p></div><br>


