---
layout: default
permalink: /ps/11-web
title: HTTP and The Web
parent: Problem Set 
nav_order:  11
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

# HTTP and The Web
{: .no_toc}




## The HOL Diner

### Background: Head-of-Line Blocking in HTTP Protocols

When a web browser loads a webpage, it must retrieve many different resources: the main HTML document, stylesheets, JavaScript files, images, and more. All of these are fetched using HTTP, an application layer protocol that relies on transport protocols like TCP or UDP.

In HTTP/1.0, the browser would open a new TCP connection for each object. This created unnecessary overhead and slowed things down. HTTP/1.1 improved this by allowing multiple objects to be requested over the same TCP connection. With pipelining, the browser could send several requests back-to-back without waiting for the previous replies to return. However, the server was still required to send responses in the same order the requests were received.

{:.important}
This ordering constraint led to a problem called **head-of-line blocking (HOL blocking)**. If the first requested object is large or delayed for any reason, all subsequent objects, *even small ones that are ready to go* must wait for it. The queue is blocked by the “head” of the line.

HTTP/2 solves this problem at the application layer. It introduces multiplexing: each object is broken into <span class="orange-bold">frames</span>, and the frames can be *interleaved* over the same TCP connection. The client can receive part of object 1, then part of object 2, and so on, even if object 1 is not yet complete. This means small objects no longer get stuck behind large ones. However, HTTP/2 still runs over TCP, which enforces in-order delivery of bytes. If a single packet is lost, TCP waits until that packet is retransmitted before delivering any further data, even if it belongs to a different object. This is **transport-layer HOL blocking**.

HTTP/3 eliminates this issue by running over QUIC, which is built on UDP. QUIC manages each stream independently. A packet loss on one stream does not delay the others, since there is no enforced global byte ordering. This avoids both application-layer and transport-layer HOL blocking.


### Scenario

You are the systems engineer for HOL Diner, an experimental restaurant that sends food to customers through pneumatic tubes. A customer’s order includes five items: one main dish and four side dishes. The main dish is large and takes time to prepare. The others are quick and easy.

You simulate three different delivery configurations that match HTTP protocols:

**Setup A** uses HTTP/1.1 with pipelining. All items are queued in order and must be sent one by one, even if some are ready earlier.

**Setup B** uses HTTP/2. Items are split into segments and interleaved in transit, allowing side dishes to arrive while the main dish is still in progress.

**Setup C** uses HTTP/3. Each item is sent over its own independent stream, and the loss of one segment does not delay the others.

{:.note}
All clients order the **same** five items at the same time. You observe how long they wait and *which* items arrive first.


### Questions
1. What causes the smaller side dishes to be delayed in Setup A?
2. Why does Setup B improve the delivery time for the side dishes, even if the main dish is still cooking?
3. Suppose one segment of the main dish is lost in Setup B. What happens to the delivery of the side dishes?
4. How does Setup C allow the side dishes to arrive independently, even in the presence of packet loss?
5. For a modern interactive website with many small components, which setup provides the best user experience? Justify your answer.

{: .highlight}
> **Hints**:
> * HTTP/1.1 pipelining delivers responses in strict order.
> * TCP guarantees in-order delivery, so one lost packet blocks the entire byte stream.
> * HTTP/2 multiplexes objects over a single TCP connection but still suffers from transport-layer HOL.
> * HTTP/3 uses QUIC, where each stream has its own loss recovery and encryption.

<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    In Setup A, the smaller side dishes are delayed because the main dish is at the head of the line. Since HTTP/1.1 pipelining requires the server to respond in order, the side dishes cannot be delivered until the main dish is complete. This is an example of application-layer HOL blocking.
  </p>

  <p>
    Setup B improves delivery time because HTTP/2 allows multiple objects to be split into frames and sent in an interleaved fashion. While the main dish is still being transmitted, the server can send parts of the side dishes too. This removes application-layer HOL blocking and improves responsiveness for small items.
  </p>

  <p>
    However, HTTP/2 still runs on top of TCP. If one packet in the stream is lost (say a segment of the main dish) TCP will delay the delivery of all subsequent data until the lost packet is retransmitted and received. Even if the side dishes are ready and their data is in later packets, they are blocked by TCP’s in-order delivery mechanism. This is transport-layer head-of-line blocking.
  </p>

  <p>
    Setup C avoids this by using HTTP/3, which is built on QUIC over UDP. QUIC manages loss recovery and encryption separately for each stream. If a packet carrying part of the main dish is lost, only the stream for that dish is affected. The side dishes can continue arriving without delay. This allows more reliable and efficient delivery, especially in lossy networks.
  </p>

  <p>
    For a modern interactive website with many small resources, HTTP/3 provides the best user experience. It avoids both application-layer and transport-layer head-of-line blocking, supports faster connection establishment with 0-RTT, and handles packet loss more gracefully. Users see faster load times and smoother performance overall.
  </p>
</div>


Acknowledged. Below is the revised version of **The Stalled Gallery**, using a proper **space-time-style diagram** (with vertical flow for time), no em-dashes, and no ambiguous horizontal timelines. The object delivery is shown with vertical progression (time flows down), and clear sequencing across protocols.


## The Stalled Gallery

### Background: Object Fetching and Protocol Behavior

Web browsers load pages by requesting many objects from a server. These include HTML files, images, scripts, and stylesheets. All of them are retrieved using **HTTP**, the application-layer protocol that sits above transport protocols like **TCP** and **UDP**.

Here's a recap, refer to notes for further details.

#### HTTP/1.1 

With **HTTP/1.1**, the browser can send multiple requests using a persistent TCP connection. But even if the browser sends all requests at once, the server must respond in order. This causes a problem called **head-of-line blocking**. If the first response is large or delayed, all subsequent responses must wait, even if they are ready.

#### HTTP/2 
**HTTP/2** improves this by allowing the server to break objects into frames and interleave their transmission. This allows smaller objects to begin arriving earlier, which improves perceived speed. However, HTTP/2 still uses TCP, which enforces in-order delivery. If one packet is lost, the entire connection must pause until that packet is recovered. This results in **transport-layer head-of-line blocking**.

#### HTTP/3
**HTTP/3**, built on QUIC over UDP, avoids this entirely. Each stream is handled independently. Packet loss in one stream does not affect others. This makes HTTP/3 especially effective for sites with many small, independent objects.


### Scenario

A photo gallery webpage requests five images at the same time:

* `img1` is a high-resolution banner image (2 MB), which takes time to generate
* `img2` to `img5` are small thumbnails (50 KB each), which are ready immediately

You test the gallery using three protocols:

* **Setup A**: HTTP/1.1 over TCP (with pipelining)
* **Setup B**: HTTP/2 over TCP (with multiplexing)
* **Setup C**: HTTP/3 over QUIC (with independent streams)

The following diagram shows an illustration of object transmission over time. Time flows downward. Each column represents a different image stream. A filled block indicates transmission of a segment of that object.


```
Setup A: HTTP/1.1
(img1 blocks others)

img1 | ███████████████████████████████████████████
img2 |                                           ▒▒▒▒
img3 |                                               ▒▒▒▒
img4 |                                                   ▒▒▒▒
img5 |                                                       ▒▒▒▒

Setup B: HTTP/2
(interleaved, but blocked by TCP loss)

img1 | ███ ███ ███ ███ ███ ███ ███ ███
img2 |   ░░  ░░  ░░
img3 |     ░░  ░░
img4 |         ░░
img5 |           ░░

Setup C: HTTP/3
(independent streams, no blocking)

img1 | ███ ███ ███ ███   ███ ███ ███
img2 | ░░░░░░░░░░░░
img3 | ░░░░░░░░░░░░
img4 | ░░░░░░░░░░░░
img5 | ░░░░░░░░░░░░
```

Legend:
`█` = transmission of a chunk of `img1`
`░` or `▒` = transmission of a thumbnail (`img2` to `img5`)
Each row shows time moving downwards from top to bottom.
Setup A blocks small images until the large image is done.
Setup B interleaves, but a TCP-level packet loss (not shown) can stall later chunks.
Setup C delivers thumbnails smoothly, unaffected by issues in the banner image.


**Answer the following questions**:
1. Why are `img2` to `img5` delayed in Setup A, even though they are small and ready?
2. Why do `img2` to `img5` appear earlier in Setup B, and what limitation still exists?
3. What happens if a segment of `img1` is lost during transmission in Setup B?
4. In Setup C, how does the use of QUIC avoid blocking of unrelated images?
5. For a modern page with many thumbnails and one banner image, which setup gives the best experience and why?

{: .highlight}
> **Hints**:
> * HTTP/1.1 requires in-order replies from the server.
> * TCP enforces strict byte ordering.
> * HTTP/2 interleaves streams at the application layer.
> * HTTP/3 handles stream delivery and loss independently.


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    In Setup A, the server must send responses in the order requests were received. Since `img1` is large and takes time to generate or send, the server cannot send `img2` to `img5` until `img1` is completely transmitted. This creates application-layer head-of-line blocking. Even though the small images are ready, they are stuck behind the large one.
  </p>

  <p>
    In Setup B, HTTP/2 allows multiple streams to be multiplexed. The server can break objects into frames and send parts of different images interleaved. This means thumbnails can begin arriving even while the large image is still being delivered. The user sees content sooner. However, because HTTP/2 still uses TCP, any lost packet on the connection forces the client to wait before it can process later frames. This is transport-layer head-of-line blocking.
  </p>

  <p>
    If a segment of `img1` is lost in Setup B, TCP pauses the delivery of all following bytes in the connection. This includes parts of other images, even if those objects are unrelated. This delay is caused by the requirement for in-order byte delivery in TCP. So thumbnails may freeze even if their data was transmitted right after the loss.
  </p>

  <p>
    In Setup C, HTTP/3 uses QUIC, which is built on top of UDP. Each stream is handled independently. If a packet carrying part of `img1` is lost, only that stream is delayed. The streams for `img2` to `img5` are unaffected and can continue transmitting and arriving normally. This results in faster and smoother loading, especially when there are many small objects and occasional packet loss.
  </p>

  <p>
    HTTP/3 provides the best experience in this scenario. It avoids both types of head-of-line blocking. Users can see thumbnails load quickly without being delayed by the banner image. Even in lossy networks, QUIC ensures that the impact of lost packets is limited to the affected stream. This makes page loads faster and more resilient.
  </p>
</div>



## The Stalled Gallery (Follow-Up)

### Background: Packet Loss and Stream Behavior

You continue testing the photo gallery under Setup B (HTTP/2 over TCP) and Setup C (HTTP/3 over QUIC). All five images are requested at the same time.

The large image `img1` is transmitted over multiple segments. In both HTTP/2 and HTTP/3, image objects are broken into frames or chunks, but the transport layer handles packet delivery differently.

You capture a trace of the network during the transmission of `img1` to `img5`. One segment of `img1` is deliberately dropped in the middle of the transfer.

Below is a simplified **Wireshark-style event log** showing packet arrival order at the receiver.


### Packet Trace

#### Setup B: HTTP/2 over TCP

```
[1] TCP ACK       (connection established)
[2] HTTP2 STREAM  img1 - frame 1
[3] HTTP2 STREAM  img2 - frame 1
[4] HTTP2 STREAM  img1 - frame 2
[5] HTTP2 STREAM  img3 - frame 1
[6] HTTP2 STREAM  img1 - frame 3  ← LOST
[7] HTTP2 STREAM  img4 - frame 1  ← RECEIVED, but held back
[8] HTTP2 STREAM  img5 - frame 1  ← RECEIVED, but held back
[9] TCP RETRANSMIT img1 - frame 3
[10] HTTP2 STREAM  img1 - frame 4
```

#### Setup C: HTTP/3 over QUIC

```
[1] QUIC HANDSHAKE    (connection established)
[2] QUIC STREAM img1 - frame 1
[3] QUIC STREAM img2 - frame 1
[4] QUIC STREAM img1 - frame 2
[5] QUIC STREAM img3 - frame 1
[6] QUIC STREAM img1 - frame 3  ← LOST
[7] QUIC STREAM img4 - frame 1  ← DELIVERED IMMEDIATELY
[8] QUIC STREAM img5 - frame 1  ← DELIVERED IMMEDIATELY
[9] QUIC RETRANSMIT img1 - frame 3
[10] QUIC STREAM img1 - frame 4
```



**Answer the following questions**:
1. In the TCP trace (Setup B), why are frames 7 and 8 held back even though they arrived successfully?
2. In the QUIC trace (Setup C), why are frames 7 and 8 delivered immediately?
3. What specific TCP design principle causes the delay seen in Setup B?
4. In the QUIC design used in Setup C, how does the stream-level independence affect perceived latency for the user?
5. From this trace, what can you conclude about HTTP/2’s and HTTP/3’s robustness in high-latency or lossy networks?

{: .highlight}
> **Hints**:
> * TCP enforces global in-order delivery of the byte stream.
> * HTTP/2 packets from different streams still travel in a single TCP stream.
> * QUIC allows independent encryption, sequencing, and recovery per stream.


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    In the TCP trace for Setup B, frames 7 and 8 are part of different streams (`img4` and `img5`), and they arrive successfully. However, TCP delivers data as a continuous ordered byte stream. Since frame 6 (a segment of `img1`) was lost, the TCP receiver holds back later data until that segment is retransmitted and received. This affects all streams, even though their data was not lost.
  </p>

  <p>
    In the QUIC trace for Setup C, frames 7 and 8 belong to streams unrelated to the lost segment. Since QUIC treats streams independently, it delivers `img4` and `img5` immediately. There is no global sequencing requirement across streams. Packet loss in `img1` does not interfere with delivery of other streams.
  </p>

  <p>
    The delay in Setup B is caused by TCP’s strict in-order delivery guarantee. TCP assigns sequence numbers to bytes in the stream. If any byte is missing, subsequent bytes cannot be delivered to the application, even if they arrived.
  </p>

  <p>
    QUIC avoids this by implementing loss detection, retransmission, and ordering at the stream level. Each stream has its own flow control and packet recovery. This allows unrelated content to proceed without delay. As a result, users see thumbnails and other small objects arrive on time even if one image is delayed.
  </p>

  <p>
    From the trace, we can conclude that HTTP/3 over QUIC is more robust in lossy networks. It reduces user-visible delays by isolating the impact of packet loss. In contrast, HTTP/2 over TCP can cause noticeable stalling even when only one packet is lost, since all streams share the same TCP connection and sequencing.
  </p>
</div>



## The Stalled Gallery (RTT Edition)

### Background: Round-Trip Time and Protocol Overheads

When a client fetches resources from a server, the time required depends not just on bandwidth, but on the number of **round-trip times (RTTs)** involved in the process. One RTT is the time it takes for a message to go from client to server and back.

Different protocols incur different numbers of RTTs depending on how they establish connections and how they schedule object transmissions. 

### Scenario 

This scenario is the same as the previous two questions. Here's the details for convenience:


#### Online Photo Gallery 
A client visits an online photo gallery hosted on a website. The page includes five images:

* `img1` is a **large image** (2 MB), taking **4 RTTs worth of data** to transmit
* `img2` to `img5` are **small images** (50 KB each), each taking **1 RTT** to transmit

Assume the following conditions:
* All images are fetched from the **same origin server directly**, with **no CDN, proxy, or cache**
* All HTTP requests are made **simultaneously**, immediately after the connection is ready
* The network is **ideal**: no congestion, no packet loss, no reordering, and no queueing delay
* The **round-trip time (RTT)** between client and server is **100 ms**
* **ACKs and application requests are piggybacked** wherever possible
* **DNS lookup is not included** in timing
* For HTTP/1.1 and HTTP/2, **TLS 1.3** is used over TCP:
  * TCP 3-way handshake: 1 RTT
  * TLS 1.3 handshake: 1 RTT
* For HTTP/3, **QUIC** is used with **integrated TLS 1.3**:
  * QUIC handshake: 1 RTT
  * With 0-RTT, requests can be sent during handshake
* Assume that the available bandwidth per RTT is <span class="orange-bold">sufficient to interleave the frames</span> of all five images such that the small images <span class="orange-bold">do not add transmission delay</span>.

{:.note}
This question assumes the common real-world deployment of **HTTP** over **TLS** (i.e., <span class="orange-bold">HTTPS</span>). This affects the number of RTTs required for connection setup. However, in this course we focus on HTTP as an application-layer protocol, and do not cover the TLS layer in depth. The inclusion here is purely to help contextualize RTT overheads in practical scenarios.


**Answer the following questions**:
1. In HTTP/1.1 over TCP (with pipelining), how many RTTs are required to complete the full load of all 5 images?
2. In HTTP/2 over TCP, how many RTTs are needed, and why is it better than HTTP/1.1?
3. In HTTP/3 with 1-RTT handshake, how many RTTs are needed? What changes with 0-RTT?
4. Compute and compare the total wall-clock time (in ms) for HTTP/1.1, HTTP/2, HTTP/3 with 1-RTT, and HTTP/3 with 0-RTT.
5. How do pipelining, multiplexing, and handshake design contribute to latency in each case?


To aid your understanding, here's a simple diagram showing how each RTT is derived from TCP + TLS:
```
Client                          Server
  |                                |
  |--- TCP SYN ------------------->|   
  |                                |
  |<-- SYN-ACK -------------------|    ← RTT 1 ends
  |--- ACK + TLS ClientHello ---->|   ← TCP + TLS piggybacked

  |                                |
  |<-- TLS EncryptedExtensions ---|   
  |    TLS ServerCert, Finished   |    ← RTT 2 ends
  |--- TLS Finished + HTTP GET -->|   ← TLS finish + HTTP request piggybacked

  |                                |
  |<-- HTTP Response -------------|    ← RTT 3 ends
```

**Piggybacking**:
* **ACK + ClientHello**: The TCP ACK and the TLS ClientHello are sent in the same packet, reducing wait time. No extra RTT needed.
* **TLS Finished + HTTP Request**: Once the client finishes TLS, it immediately sends the HTTP request in the same packet as the TLS Finished message

{: .highlight}
> **Hints**:
> * In HTTP/1.1, all responses must arrive in **order**. Later responses wait for earlier ones.
> * In HTTP/2 and HTTP/3, small images can complete while the large image is still in progress.
> * QUIC allows early data during handshake in 0-RTT mode.


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    <strong>1.</strong> In HTTP/1.1, connection setup includes 1 RTT for TCP and 1 RTT for the TLS 1.3 handshake. Requests are sent right after setup. However, the server must respond in order. Since <code>img1</code> takes 4 RTTs, the small images must wait. Each of <code>img2</code> to <code>img5</code> takes 1 RTT but cannot begin until <code>img1</code> finishes. So we have:
    <br>• 2 RTTs (setup)  
    <br>• 4 RTTs for <code>img1</code>  
    <br>• 4 RTTs for <code>img2–5</code> (1 each, sequential)  
    <br><strong>Total = 10 RTTs = 1000 ms</strong>
  </p>

  <p>
    <strong>2.</strong> In HTTP/2, setup is the same (TCP + TLS = 2 RTTs). But once the connection is ready, all streams are multiplexed over one TCP connection. The server can interleave data. <code>img2</code> to <code>img5</code> each need 1 RTT’s worth of data, but since they are sent in parallel with <code>img1</code>, they *do not add time*.  This is not “magic parallelism”, it's a model where the network can carry the total load in 4 RTTs **as per our assumption** (read questions carefully in exams). The dominant transfer time is that of the largest object:
    <br>• 2 RTTs (setup)  
    <br>• 4 RTTs (total transmission window, parallel)  
    <br><strong>Total = 6 RTTs = 600 ms</strong>
  </p>

  <p>
    <strong>3.</strong> HTTP/3 uses QUIC, which merges transport and crypto handshake into 1 RTT. Object transfers begin immediately after. Streams are independent, so small images are unaffected by delays in large objects:
    <br>• 1 RTT (setup)  
    <br>• 4 RTTs (parallel transmission, dominated by <code>img1</code>)  
    <br><strong>Total = 5 RTTs = 500 ms</strong>
    <br>
    With 0-RTT, the requests are sent together with the handshake. This overlaps the first RTT of transfer with setup. So only 4 RTTs are needed from start to finish:
    <br><strong>Total = 4 RTTs = 400 ms</strong>
  </p>

  <p>
    <strong>4.</strong> Total times:
    <br>• HTTP/1.1 = 1000 ms  
    <br>• HTTP/2 = 600 ms  
    <br>• HTTP/3 (1-RTT) = 500 ms  
    <br>• HTTP/3 (0-RTT) = 400 ms  
    <br><br>
    HTTP/3 with 0-RTT is fastest because it overlaps handshake with data transfer and avoids blocking entirely.
  </p>

  <p>
    <strong>5.</strong> HTTP/1.1 suffers from both setup delay and forced response ordering. HTTP/2 fixes application-layer head-of-line blocking by multiplexing streams, allowing early delivery of small objects. However, it still incurs 2 RTTs of setup. HTTP/3 further reduces latency with a faster handshake and independent stream recovery. With 0-RTT, requests begin immediately, improving perceived page load time, especially on repeat visits.
  </p>
</div>




## The Split Page

### Background: Multiple Origins and DNS Overhead

Modern webpages often include content from **multiple domains** — for example, images from a CDN, analytics scripts from a third-party, and ads from an ad network. Each of these domains requires a **separate DNS resolution** and typically a **separate HTTPS connection** unless a prior one is cached or reused.

DNS resolution usually takes **1 RTT**, and HTTPS connection setup (TCP + TLS 1.3) takes **2 RTTs**. When objects come from multiple domains, the browser may issue DNS queries and open connections **in parallel**, but each object can only be fetched once its own connection is ready.


### Scenario

The diagram below shows how a browser loads six images from a webpage:

```
Client                    DNS Resolver           Origin Server(s)

 |---DNS www.campus.sg------------------------------>|
 |<--Response----------------------------------------|  [1 RTT]
 |---TCP SYN---------------------------------------->|
 |<--SYN-ACK----------------------------------------|  [1 RTT]
 |---TLS 1.3 Hello---------------------------------->| 
 |<--TLS Finished-----------------------------------|  [1 RTT]
 |---GET img1--------------------------------------->|
 |<--img1 Response----------------------------------|  [1 RTT]

 |---DNS cdn.campuscdn.com-------------------------->|
 |<--Response----------------------------------------|  [1 RTT]
 |---TCP SYN---------------------------------------->|
 |<--SYN-ACK----------------------------------------|  [1 RTT]
 |---TLS 1.3 Hello---------------------------------->| 
 |<--TLS Finished-----------------------------------|  [1 RTT]
 |---GET img2--------------------------------------->|
 |<--img2 Response----------------------------------|  [1 RTT]

 |---DNS ads.adprovider.net------------------------->|
 |<--Response----------------------------------------|  [1 RTT]
 |---TCP SYN---------------------------------------->|
 |<--SYN-ACK----------------------------------------|  [1 RTT]
 |---TLS 1.3 Hello---------------------------------->| 
 |<--TLS Finished-----------------------------------|  [1 RTT]
 |---GET img3--------------------------------------->|
 |<--img3 Response----------------------------------|  [1 RTT]

 |---DNS cdn.campuscdn.com-------------------------->|
 |<--Response----------------------------------------|  [CACHED]
 |---TCP SYN---------------------------------------->|
 |<--SYN-ACK----------------------------------------|  [1 RTT]
 |---TLS 1.3 Hello---------------------------------->| 
 |<--TLS Finished-----------------------------------|  [1 RTT]
 |---GET img4--------------------------------------->|
 |<--img4 Response----------------------------------|  [1 RTT]

 |---DNS www.campus.sg------------------------------>|
 |<--Response----------------------------------------|  [CACHED]
 |---TCP SYN---------------------------------------->|
 |<--SYN-ACK----------------------------------------|  [1 RTT]
 |---TLS 1.3 Hello---------------------------------->| 
 |<--TLS Finished-----------------------------------|  [1 RTT]
 |---GET img5--------------------------------------->|
 |<--img5 Response----------------------------------|  [1 RTT]

 |---DNS analytics.thirdparty.io-------------------->|
 |<--NXDOMAIN----------------------------------------|  [1 RTT]
```

Assumptions:
* RTT = 100 ms
* All DNS queries and TCP+TLS handshakes happen in **parallel**
* Each domain requires: 1 RTT (DNS) + 1 RTT (TCP) + 1 RTT (TLS) + 1 RTT (image)
* DNS lookups for repeated domains are not reused in this scenario
* All image objects are small (transfer in 1 RTT)
* HTTPS is used (HTTP over TLS 1.3)
* No caching, reuse, or connection pooling unless otherwise stated
* The network is ideal (no congestion, loss, or queuing)

{:.note}
This scenario assumes **HTTPS**, where HTTP is layered on TLS. Although this course focuses on HTTP as an application-layer protocol, TLS impacts overall latency and is modeled here for realism.


**Answer the following questions**:
1. List all the **distinct domains** involved. For each, compute the time taken from **initial DNS query to the completion of image transfer**.
2. Which images share a domain? Can they reuse a connection? Why or why not in this scenario?
3. What is the **total time** needed for the **slowest image** to finish loading, assuming all DNS lookups and connections happen in parallel?
4. Suppose the DNS lookup for `analytics.thirdparty.io` fails and returns `NXDOMAIN`. How does this affect image loading and overall page load latency?
5. If the browser implements a **connection pool** per domain, how would that change the number of RTTs needed if all six images were to be fetched again immediately (i.e., in a repeat visit with DNS cached and connections still alive)?

{: .highlight}
> **Hints**:
> * Each image requires: DNS (1 RTT) + TCP (1 RTT) + TLS (1 RTT) + image transfer (1 RTT)
> * Parallel actions don’t sum in time
> * Shared domains can reuse connections, but this scenario assumes they do not


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    <strong>1.</strong> Distinct domains:
    <br>• <code>www.campus.sg</code>  
    <br>• <code>cdn.campuscdn.com</code>  
    <br>• <code>ads.adprovider.net</code>  
    <br>• <code>analytics.thirdparty.io</code>
    <br><br>
    Each domain requires:  
    • 1 RTT (DNS) + 1 RTT (TCP) + 1 RTT (TLS) + 1 RTT (image) = <strong>4 RTTs = 400 ms</strong>  
    Images from each domain take 400 ms to load if no caching is used.
  </p>

  <p>
    <strong>2.</strong> Shared domains:
    <br>• <code>img1.png</code> and <code>img5.png</code> → <code>www.campus.sg</code>  
    <br>• <code>img2.png</code> and <code>img4.png</code> → <code>cdn.campuscdn.com</code>
    <br><br>
    In this scenario, each image is fetched using a **separate connection**, so no reuse is applied. Even if domains are the same, handshakes and transfers are repeated.
  </p>

  <p>
    <strong>3.</strong> Since all connections are opened in parallel, the **slowest image** determines total page latency. All successful images take the same time (400 ms), so:
    <br><strong>Total time = 400 ms</strong>
  </p>

  <p>
    <strong>4.</strong> A DNS failure for <code>analytics.thirdparty.io</code> returns in 1 RTT. The image cannot load, but this does not delay other images. Overall page load time remains <strong>400 ms</strong>, though a browser warning may be shown for the missing object.
  </p>

  <p>
    <strong>5.</strong> On a repeat visit:
    <br>• DNS is cached → 0 RTT  
    <br>• TLS connections are reused → 0 RTT  
    <br>• Only 1 RTT per image for transfer
    <br><br>
    Since transfer can begin immediately, and requests can be sent in parallel:
    <br><strong>Total page load time = 1 RTT = 100 ms</strong>
  </p>
</div>



