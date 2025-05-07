---
layout: default
permalink: /ns/02-internet-hierarchy
title: The Internet Hierarchy
description: Basics of internet hierarchy
parent: Network Security
nav_order: 2
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

# The Internet Hierarchy 
{: .no_toc}

{:.highlight-title}
> Detailed Learning Objectives
> 
> 1. **Justify the Structure of Internet Hierarchies**
>   - Justify the different **levels** of ISPs including access, regional/national, and global tiers.
>   - Identify the **role** of each ISP level in facilitating global internet connectivity.
>   - Explain **why** the hierarchical structure is **essential** for the **scalable** and **efficient** operation of the internet.
> 2. **Describe the Functions and Significance of Internet Exchange Points (IXPs)**
>   - Describe the purpose of IXPs in internet architecture and how they facilitate peer connections among ISPs.
>   - Explain the **economic** and **operational** benefits of using IXPs for internet traffic exchange.
> 3. **Explain Content Delivery Networks (CDNs)**
>   - Identify major **CDNs** and their function in optimizing internet content delivery.
>   - Explain how CDNs **enhance** user experience by reducing latency and improving content delivery speed.
> 4. **Explain Points of Presence (PoPs)**
>   - Define what a PoP is and its role in network access and connectivity.
>   - Discuss the **advantages** of PoPs including reduced latency, improved bandwidth, and network redundancy.
>
> These learning objectives aim to guide your comprehension and exploration of the structured levels and roles within the internet hierarchy.
  
The internet hierarchy refers to the structured levels of Internet Service Providers (ISPs) that facilitate global internet connectivity, organized into tiers based on their size, reach, and the nature of their network agreements.

It is the network of **networks**. The internet service providers (ISPs) are separated in terms of hierarchy: global ISP, national ISP / regional ISP, access ISP.

<img src="{{ site.baseurl }}//docs/NS/images/02-internet-hierarchy/2024-04-25-10-30-45.png"  class="center_seventy"/>

### Access ISP 

The access ISPs are end systems. PCs, smartphones, Web servers, mail servers, and other devices connect into the Internet via an access ISP. 

### Regional or National ISP 

In any given geographical region, there may be a regional ISP to which the **access** ISPs in the region connect. Access ISPs pay service fees to regional/national ISPs.

### Tier-1 or Global ISP

Each regional ISP then connects to tier-1 ISPs to connect to other parts of the world. Tier-1 ISPs do not need to pay service fees to anyone else since they’re at the top level.

{:.note}
They do not have a presence in every city in the world, but they connect large regions together.

### Internet Exchange Points (IXP)

IXP is a physical infrastructure such as a stand-alone building with its own network of switches,established to serve as a meeting point for nearby ISPs of (typically) the <span class="orange-bold">same level</span> to peer together. IXPs are built so that they can directly connect their networks together and allow traffic to pass **directly** instead of through an upstream intermediaries (and pay fees to upstream intermediaries like Regional / Global ISP).

{:.note}
IXP is usually settlement-free.

They are created to <span class="orange-bold">reduce cost</span>. Typically, downstream  ISPs pay their provider ISPs (we call this *customer ISPs*) to obtain global Internet interconnectivity. The amount that a customer ISP pays a provider ISP <span class="orange-bold">reflects</span> the amount of <span class="orange-bold">traffic</span> it exchanges with the provider. When peering through IXPs, it is settlement free and they can bypass the provider ISPs (of the higher level).

### Content Provider Network / Content Delivery Network

The current major CDNs (among many others) are: Google, OpenConnect (used by Netflix), Akamai, Limelight, and CloudFare. 

They host servers and <span class="orange-bold">data centres</span> placed in many locations around the world. These data centres are all <span class="orange-bold">interconnected</span> via their own private TCP/IP network, which spans the entire globe but is nevertheless <span class="orange-bold">separate</span> from the public Internet. 

{:.important}
The private network of these CDNs only carries traffic to/from its own servers. It results in a <span class="orange-bold">faster</span> network connection for <span class="orange-bold">customers</span> of these companies due to geographical proximity, content optimization, caching, and optimized routing *within* the CDN network.

However, despite the presence of a CDN, end users still connect to the internet through their Internet Service Providers (ISPs) to form an initial connection, routing through ISP, and then accessing the <span class="orange-bold">nearest</span> CDN server (geographically), and then data is delivered back through ISP. 

{:.note}
In essence, CDNs **optimize** the delivery of content by **reducing** the distance and hops needed to transport data, but they *still* operate on top of the foundational internet infrastructure provided by ISPs. The CDN can only speed up delivery *within* the constraints of the existing internet pathways, which are largely controlled and maintained by ISPs.

### Point of Presence 

The PoP (not drawn in the figure above) is a physical access point comprised of a group of one or more routers (at the same location) belonging to the provider’s network where customer ISPs can connect into the provider ISP. Provider ISPs own and manage extensive network infrastructure, including PoPs.

The difference from IXP is that this is for **customer** ISP or users to connect to the <span class="orange-bold">provider</span> ISP or <span class="orange-bold">access</span> ISP (different hierarchy level), even when they are located slightly geographically away from the main building of the provider / access ISP, whereas IXP is for peering between ISPs of the <span class="orange-bold">same</span> level.

For a customer network to connect to a provider’s PoP, it can **lease** a high-speed link from a third-party telecommunications provider to directly connect one of its routers to a router at the PoP. 

{:.note}
In other words, PoP is <span class="orange-bold">not</span> free of charge. 

Nevertheless, PoP serves the following benefits: 
- **Reduced Latency**: PoPs decrease data travel distance, enhancing speed and user experience.
- **Improved Bandwidth**: Local data processing **prevents** bottlenecks, increasing available bandwidth for users.
- **Scalability**: PoPs enable ISPs to expand their reach and capacity efficiently across regions.
- **Reliability and Redundancy**: Multiple PoPs offer fallback options, enhancing network stability and reducing downtime.
- **Customized Services**: Regional PoPs can offer services tailored to local user needs and preferences.
- **Enhanced Security**: Localized security measures at PoPs help protect against **region-specific** threats and enable quicker response to incidents.

## Further Consideration 
Why do we need the internet hierarchy: 
**Presence of competitors**: Competitions allow for many of Tier-1 ISPs in the world, and global/regional ISPs within each region. 


**Multi-home**: Except tier-1 ISPs, any ISPs may choose to multi-home, that is, to connect to two or more ISPs. This is to ensure that network traffic is not crippled in case one of their providers suffer failures. 

**Why do we need the internet hierarchy**: It is <span class="orange-bold">impossible</span> to interconnect each device to one another, since you need to manage $$N^2$$ connections between each device to another, where $$N$$ is a very large number, reaching >200 billions in the year 2024 (see [wigle.net](https://wigle.net) for a visual representation of wireless network mapping).  

Hence by making the internet hierarchy, we:
1. <span class="orange-bold">Support scalability and sustained growth</span>. There’s really a lot of end hosts that are connected to the internet, and this number is growing, explosively. 
2. <span class="orange-bold">Tackle the problem of incommensurate scaling:</span> Different aspects of the network grow at a different rate. 

# Summary 
This chapter dives into the structured hierarchy of Internet Service Providers (ISPs) which is **vital** for global internet connectivity. We categorize ISPs into three levels—access, regional/national, and global. I also explain the role of Internet Exchange Points (IXPs) in enabling direct peering between ISPs, thus enhancing efficiency and reducing costs. Additionally, I discuss the significance of Content Delivery Networks (CDNs) and Points of Presence (PoPs), which are crucial for **optimizing** content delivery and improving network performance. 

Key learning points include:
- **Structure of Internet Service Providers (ISPs)**: Explanation of the hierarchical arrangement of ISPs into global, national/regional, and access levels.
- **Internet Exchange Points (IXPs)**: Discusses the role of IXPs in facilitating efficient traffic exchange between ISPs at similar levels without intermediaries.
- **Content Delivery Networks (CDNs)**: Outlines how CDNs optimize internet content delivery by reducing latency and increasing speed through strategically located servers.
- **Point of Presence (PoPs)**: Describes PoPs as access points that enhance network service quality by improving latency, bandwidth, and reliability.

This hierarchical structure is foundational for scalable and efficient connectivity across the internet.


# Appendix 
## IPv4 Addressing 
The IPv4 is a set of 32-bit numbers, divided into **four** parts (1 byte each). It allows for 4.29 billion unique addresses.

Range of IPv4: `0.0.0.0` to `255.255.255.255`. You can find out your IP address by running `ifconfig`. A sample IPv4:

<img src="{{ site.baseurl }}//docs/NS/images/02-internet-hierarchy/2024-04-25-11-57-38.png"  class="center_thirty"/>

There are two types of addresses in general:
1. Local address, for example the above (also known as private IP)
2. Public gateway address (also known as public IP)
 
The gateway address is what your address looks like to the outside world. A router has an internal and external interface, where the external interface has a different IP than its internal interface. 
- The IP address `192.168.1.254` is usually the default private IP address for some home broadband routers
- The IP address 192.168.1.0 is typically reserved to identify the **network** itself and is not assignable to individual devices within the network.
- The private IP address is **not** globally routable on the <span class="orange-bold">wider</span> Internet. They are however fully routable within your own home or organizational network. 

There’s no direct connection between hosts to the internet. All connection typically must go through the router (for security reasons, e.g: filtering attacks), unless when you enable port forwarding[^2].

<img src="{{ site.baseurl }}//docs/NS/images/02-internet-hierarchy/2024-04-25-12-00-43.png"  class="center_seventy"/>

## Dynamic IP Address

The **public** gateway address is routable, given by IANA. Most ISPs are given a range of routable IP addresses, and employ dynamic IP addresses to their customers. 

The **local** IP address is obtained via **DHCP** protocol, implemented in the network layer of the routers. A new device connected to the local area network will broadcast a request to get an address. Routers will service this request and give the device an unassigned IP address.

Some common local IP addresses:
* `192.168.0.1` to `192.168.255.255` : Linksys and D-Link Routers
* `10.0.0.0` to `10.255.255.255`: Comcast and Xfinity Routers

## Static IP Address

Customers can pay a fee to request their ISPs to assign them a <span class="orange-bold">static public IP</span>. This is especially useful for those hosting web services.

For local IP address: a device can request for static IP addressing as well, if available, so that it is always configured to obtain a fixed IP address. You can do this via your router configuration page.   

{:.info-title}
> Accessing router's configuration page
>
> To access your router's configuration page, first connect to your network and then enter your router's IP address (commonly `192.168.1.1`, `192.168.0.1`, or `10.0.0.1`) into your web browser's address bar. You'll need to log in using the default credentials, which are often "admin" for both username and password, though you should check your router's manual if these don't work. Once logged in, you can modify settings such as the Wi-Fi password, network management, and security features.

{:.note}
If this interests you, you may want to learn more about **subnetting** and IP **masking** yourself after this. Also, educate yourself about IPv6 (the most recent version of Internet Protocol) that uses 128 bits.


[^2]: Port forwarding or port mapping is an application of network address translation (NAT) that redirects a communication request from one address and port number combination to another while the packets are traversing a network gateway, such as a router or firewall.