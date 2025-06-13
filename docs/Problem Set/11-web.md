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
    These are the distinct domains:
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
    These are the shared domains:
    <br>• <code>img1.png</code> and <code>img5.png</code> → <code>www.campus.sg</code>  
    <br>• <code>img2.png</code> and <code>img4.png</code> → <code>cdn.campuscdn.com</code>
    <br><br>
    In this scenario, each image is fetched using a **separate connection**, so no reuse is applied. Even if domains are the same, handshakes and transfers are repeated.
  </p>

  <p>
    Since all connections are opened in parallel, the **slowest image** determines total page latency. All successful images take the same time (400 ms), so:
    <br><strong>Total time = 400 ms</strong>
  </p>

  <p>
    A DNS failure for <code>analytics.thirdparty.io</code> returns in 1 RTT. The image cannot load, but this does not delay other images. Overall page load time remains <strong>400 ms</strong>, though a browser warning may be shown for the missing object.
  </p>

  <p>
    On a repeat visit:
    <br>• DNS is cached → 0 RTT  
    <br>• TLS connections are reused → 0 RTT  
    <br>• Only 1 RTT per image for transfer
    <br><br>
    Since transfer can begin immediately, and requests can be sent in parallel:
    <br><strong>Total page load time = 1 RTT = 100 ms</strong>
  </p>
</div>




## The Sticky Connection

### Background: Persistent Connections and `keep-alive`

In HTTP/1.0, each request-response pair required a **new TCP connection**. This was inefficient: every request incurred TCP (and possibly TLS) setup cost.

HTTP/1.1 introduced **persistent connections** by default. Using the `Connection: keep-alive` header, clients and servers can reuse a TCP connection for multiple requests, avoiding repeated handshakes. The connection stays open until explicitly closed or timed out.

**Reusing a connection saves RTTs**, but requires the client and server to agree to keep it open, and the server must support multiple sequential or pipelined requests on the same socket.

Connection reuse is common in HTTPS, where each new connection setup costs **2 RTTs** (TCP + TLS 1.3), making reuse even more beneficial.

### Scenario

A user visits `www.sticky.sg`, which returns an HTML page referencing these five small objects from the **same domain**:

* `logo.png`
* `banner.jpg`
* `style.css`
* `script.js`
* `font.woff`

Assume:

* RTT to server = **100 ms**
* Each object is small and takes **1 RTT** to transfer
* TLS 1.3 is used over TCP (i.e., HTTPS)
* TCP setup = 1 RTT; TLS handshake = 1 RTT
* No proxy or CDN involved
* All requests are made **sequentially**, not pipelined
* No caching or reuse unless specified


**Answer the following questions**:
1. If the browser opens a **new connection per object**, how many RTTs are required in total?
2. If the browser reuses a **single TCP+TLS connection** for all 5 objects (with `Connection: keep-alive`), how many RTTs are required in total?
3. Explain how `keep-alive` reduces latency in this scenario. Why does connection reuse matter more for HTTPS than for plain HTTP?
4. Suppose the server times out the connection after the third object. What happens when the client tries to fetch the fourth object? What is the new total latency?
5. Now assume the client pipelined the requests over the keep-alive connection. What effect would this have on latency, and why is pipelining rarely used in practice?

{: .highlight}
> **Hints**:
> * Each new connection = 1 RTT (TCP) + 1 RTT (TLS) + 1 RTT (object)
> * Reused connection = only 1 RTT per object after setup
> * Pipelining lets the client send multiple requests before responses return


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    When the browser opens a new connection for each object, it incurs 1 RTT for TCP, 1 RTT for TLS, and 1 RTT for the object transfer. That means 3 RTTs per object. Since there are five objects and no connection reuse, the total latency is 5 times 3, which is <strong>15 RTTs</strong> (1500 ms).
  </p>

  <p>
    With a single persistent connection (enabled by the <code>Connection: keep-alive</code> header), the client performs the TCP and TLS handshake only once. That costs 2 RTTs in total. Each object then takes 1 RTT to transfer, assuming the requests are sent one after another. This results in a total of 2 RTTs for setup and 5 RTTs for transfer, giving <strong>7 RTTs</strong> (700 ms).
  </p>

  <p>
    This reduction in latency is due to avoiding repeated connection setup. Each new connection would require a full TCP and TLS handshake, which adds 2 RTTs. In plain HTTP, the saving would be smaller, because only the TCP setup is needed. But in HTTPS, reusing the connection avoids more overhead and improves overall performance.
  </p>

  <p>
    If the server closes the connection after sending the third object, the client must start new connections for the fourth and fifth objects. Each of these requires 2 RTTs for setup and 1 RTT for transfer. The total becomes 2 RTTs for the first setup, 3 RTTs for the first three transfers, and then 2 setups and 2 transfers for the last two objects. This gives <strong>11 RTTs</strong> in total (1100 ms).
  </p>

  <p>
    If the client pipelines the requests over the keep-alive connection, it sends all five requests right after the TLS handshake. The server responds to them in order. This still results in 2 RTTs for setup and 5 RTTs to receive all responses, giving <strong>7 RTTs</strong> in total. The benefit is reduced idle time between requests, although it does not reduce total RTTs. In practice, pipelining is rarely used. Many servers do not handle it correctly, and because responses must come back in order, one slow response can block all the others. HTTP/2 addresses this by allowing multiplexing with independently scheduled streams.
  </p>
</div>


### Conclusion 

Here's an illustration to compare the three behaviors. Time flows from top to bottom. 

#### TCP + TLS Setup and Object Transfer Timing

```
Client                          Server
  |--- TCP + TLS Setup --------->|   RTTs 1–2
  |--- GET img1 ---------------->|   RTT 3
  |<-- img1 ---------------------|

  |--- TCP + TLS Setup --------->|   RTTs 4–5
  |--- GET img2 ---------------->|   RTT 6
  |<-- img2 ---------------------|

  |--- TCP + TLS Setup --------->|   RTTs 7–8
  |--- GET img3 ---------------->|   RTT 9
  |<-- img3 ---------------------|

  |--- TCP + TLS Setup --------->|   RTTs 10–11
  |--- GET img4 ---------------->|   RTT 12
  |<-- img4 ---------------------|

  |--- TCP + TLS Setup --------->|   RTTs 13–14
  |--- GET img5 ---------------->|   RTT 15
  |<-- img5 ---------------------|
```

**Total: 15 RTTs**

#### Keep-alive with Sequential Requests 

```
Client                          Server
  |--- TCP + TLS Setup --------->|   RTTs 1–2
  |--- GET img1 ---------------->|   RTT 3
  |<-- img1 ---------------------|

  |--- GET img2 ---------------->|   RTT 4
  |<-- img2 ---------------------|

  |--- GET img3 ---------------->|   RTT 5
  |<-- img3 ---------------------|

  |--- GET img4 ---------------->|   RTT 6
  |<-- img4 ---------------------|

  |--- GET img5 ---------------->|   RTT 7
  |<-- img5 ---------------------|
```

**Total: 7 RTTs**


#### Keep-alive with Pipelining

```
Client                          Server
  |--- TCP + TLS Setup --------->|   RTTs 1–2
  |--- GET img1 ---------------->|
  |--- GET img2 ---------------->|
  |--- GET img3 ---------------->|
  |--- GET img4 ---------------->|
  |--- GET img5 ---------------->|

                                   (Server responds in order)

  |<-- img1 ---------------------|   RTT 3
  |<-- img2 ---------------------|
  |<-- img3 ---------------------|
  |<-- img4 ---------------------|
  |<-- img5 ---------------------|
```

**Total: still 7 RTTs because of bandwidth limitation**, but less client-side idle time.


## The Port Mismatch

### Background: Application Demultiplexing by Port Number

On the internet, many services can run on the **same host** (same IP address) but be distinguished by their **transport-layer port number**. This is known as **application-layer demultiplexing**.

When a TCP or UDP segment arrives at a host, the OS looks at the **destination port** to determine which process or application should receive the data. Each listening application binds to a specific port number using a socket.

Some ports are standardized (e.g., 80 for HTTP, 443 for HTTPS, 53 for DNS), but a server can also use **custom ports** (e.g., 8080, 3000) if it wants to run multiple services or avoid conflicts.

If two different applications attempt to bind to the **same port and protocol** on the same IP address, the operating system will reject the second bind. Each (IP, port, protocol) combination must be unique for active listeners.


### Scenario

You run the following two commands on your laptop, which has a single public IP:

```bash
python3 -m http.server 8080
```

and in another terminal:

```bash
nc -l 8080
```

You receive an error from the second command saying “address already in use.”

Later, you instead try:

```bash
python3 -m http.server 8080
```

and in another terminal:

```bash
nc -l 9090
```

*Both servers run without issue.*


**Answer the following questions**:
1. Why did the second command fail when both attempted to use port 8080?
2. What exactly is the role of the port number in directing traffic to the correct process?
3. If a packet arrives at your machine on port 8080, how does the OS know which application to deliver it to?
4. If two users connect from the same laptop to the same server IP and port (e.g., `http://website.com:80`), how are their return traffic flows kept separate?
5. You run a web server on port 443 and a DNS server on port 53, both on the same machine. Why is this allowed? How does the OS distinguish between them?

{: .highlight}
> **Hints**:
> * A port number, together with IP and protocol, identifies the destination socket.
> * Servers listen on fixed ports; clients usually use ephemeral (random) source ports.
> * TCP connections are identified by (source IP, source port, dest IP, dest port).

{:.note-title}
> Ephemeral Port
>
> An **ephemeral port** is a short-lived port number automatically assigned by the operating system when a client application initiates an outgoing TCP or UDP connection.
> 
> It acts as the **source port** for that connection and allows the OS to distinguish between different sockets, even if they’re all talking to the same server and port.
> 
> Ephemeral ports are usually chosen from a specific range (e.g., 49152–65535 on most systems) and are released when the connection ends. They are essential for enabling **multiple simultaneous client connections** without port conflicts, especially when connecting repeatedly to the same server.


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The second command failed because the first command was *already* using TCP port 8080 on your IP address. Each application that wants to receive traffic must bind to a specific combination of protocol, IP address, and port number. If another process is already listening on that exact (IP, port, protocol) tuple, the OS will not allow a second one to do the same.
  </p>

  <p>
    Port numbers serve as identifiers for application-layer services. When a segment arrives at the host, the transport layer checks the destination port to decide which application should receive the data. The port number acts like a delivery slot at the transport layer.
  </p>

  <p>
    If a packet arrives at your machine destined for TCP port 8080, the OS looks for a socket that has bound to port 8080 over TCP. It will deliver the packet to the process associated with that socket. If no such socket exists, the OS will drop the packet or send back a TCP RST.
  </p>

  <p>
    When two users on the same client machine connect to the same server IP and port, the OS keeps the connections separate by assigning each client connection a unique source port. For example, one might use port 51500 and the other 53000. Even though they connect to the same server and server port, the full connection is identified by the source IP, source port, destination IP, and destination port. The OS uses this 4-tuple to distinguish each flow.
  </p>

  <p>
    Running a web server on port 443 and a DNS server on port 53 works fine because each uses a different port number. Even though they are on the same host and IP, they bind to different port numbers and may use different protocols (TCP for HTTPS, UDP for DNS). The OS keeps their sockets separate by port and protocol.
  </p>
</div>



### Summary: Clients Connecting to the Same Server Port

Here's a **Wireshark-style ladder diagram** to illustrate how the **operating system uses port numbers** to demultiplex traffic and how it handles multiple clients talking to the same server port. Given a server with two clients connecting to it, we assume that: 
* Server IP: `203.0.113.1`
* Server port: `80` (HTTP)
* Client A IP: `10.0.0.1`, ephemeral port `51500`
* Client B IP: `10.0.0.2`, ephemeral port `53000`


{:.note}
An ephemeral port is a **temporary** port number *automatically* assigned by the operating system to a client application **when it initiates an outgoing connection**.

```
Client A                          Server                        Client B
  |                                |                              |
  |--- SYN (10.0.0.1:51500 → 80) -->|                              |
  |                                |                              |
  |<-- SYN-ACK (80 → 10.0.0.1:51500)--|                            |
  |--- ACK ------------------------>|                              |

  |                                |                              |
  |                                |<-- SYN (10.0.0.2:53000 → 80)--|
  |                                |-- SYN-ACK (80 → 10.0.0.2:53000)-->
  |                                |<-- ACK ------------------------|

  |--- GET /index.html ----------->|                              |
  |<-- HTTP Response -------------|                               |
                                  |<-- GET /page.html ------------|
                                  |--> HTTP Response ------------>|
```



**How the OS handles this**: 
* On the **server**, the OS uses the full 4-tuple to distinguish connections:
  `(src IP, src port, dst IP, dst port)`
* Although both clients are connecting to the same **destination port 80**, the connections are **not the same**, because the source IP and port are different.
* Each client can run its own web browser using its own ephemeral port.
  The server keeps a separate socket entry for each active connection.


**In short**: 
* **Server port 80** is reused across many clients.
* Each **client–server pair** is *uniquely* identified by the connection 4-tuple.
* The OS routes incoming TCP segments to the correct socket using this 4-tuple.
* The diagram and explanation provided earlier are <span class="orange-bold">specifically for TCP</span>.

{:.important}
If two servers on the same machine try to bind to port 80 with TCP, the OS will <span class="orange-bold">block</span> the second one, because the listening socket `(IP, port, protocol)` must be unique, as highlighted in this question. 




## The Stateless Listener

### Background: UDP and Demultiplexing

UDP is a **connectionless transport protocol**. It does not perform a handshake or track connection state. Each UDP packet is treated independently and carries its own addressing information.

When a UDP datagram arrives, the operating system looks only at the **destination port number** and **protocol** to decide which application should receive it. The OS finds a matching socket that has bound to that `(IP address, port number, protocol)` combination.

Unlike TCP, there is no "established connection" and no 4-tuple for ongoing state. Applications that use UDP (like DNS or NTP) must implement their own logic to match replies to requests.


### Scenario

You start a DNS server on your laptop using:

```bash
sudo named -p 5353
```

Then you send a DNS query from a custom script:

```bash
dig @127.0.0.1 -p 5353 example.com
```

Your script opens a UDP socket on an ephemeral port and sends a datagram to 127.0.0.1 port 5353. The DNS server replies to the source IP and source port from your packet.

Later, you try sending two DNS queries in rapid succession using different scripts. Both use UDP and send to port 5353, but only one gets a reply.


**Answer the following questions**:
1. How does the operating system decide which application should receive an incoming UDP datagram?
2. Why is it possible for many clients to send datagrams to the same UDP port (e.g., 5353), but only one server process can bind to that port at a time?
3. When the DNS server receives your request, how does it know where to send the reply?
4. Why is there no concept of "connection reuse" in UDP?
5. What happens if two client scripts on your laptop try to bind to the same UDP source port when sending requests? Can the replies get misrouted?

{: .highlight}
> **Hints**:
> * UDP has no session state or handshake
> * Replies go to (source IP, source port) in the request
> * Multiple clients can send to the same destination port, but only one listener can bind to it



### UDP Demultiplexing: No Connections, Just Packets

The diagram below shows how UDP demux works:

```
Client A                      Server (DNS @ port 5353)                     Client B
  |                                |                                        |
  |-- UDP: 10.0.0.1:50001 → 5353 -->|                                        |
  |                                |                                        |
  |<-- UDP: 5353 → 10.0.0.1:50001 --|                                        |

  |                                |<-- UDP: 10.0.0.2:50100 → 5353 ---------|
  |                                |-- UDP: 5353 → 10.0.0.2:50100 --------->|
```


* Each **incoming UDP datagram** is matched by the OS using only:
  * **Destination port**
  * **Protocol (UDP)**
  * **Local IP (optional)**

The OS finds a **single matching socket** bound to UDP port 5353 and delivers the datagram to it. The DNS server sends replies by copying the **source IP and port** from the request into the **destination** of the response.

{:.note}
Unlike TCP, there is **no persistent connection**, no session tracking, and **no 3-way handshake**.


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The operating system uses the destination UDP port number to decide which application should receive an incoming datagram. When a packet arrives, it looks for a socket bound to the specified port and protocol. If one is found, the packet is delivered to that socket. If no socket is listening on that port, the datagram is dropped.
  </p>

  <p>
    Multiple clients can send UDP packets to the same destination port because that port is on the receiving side. The server listens on a fixed port (like 5353), and many different clients with unique source ports can send packets to it. However, only one server process can bind to a given port on the same IP at the same time. This ensures that there is no ambiguity about who should receive incoming packets.
  </p>

  <p>
    When your DNS server receives the query, it inspects the source IP and source port fields in the UDP header. These fields tell the server where to send the reply. The response packet is constructed using the server’s own port 5353 as the source, and the client’s IP and port as the destination.
  </p>

  <p>
    UDP does not have any built-in concept of connection reuse or session state. Each message is independent. There is no handshake, no teardown, and no timeout that marks the end of a connection. Applications that need to track ongoing interactions must implement that logic themselves.
  </p>

  <p>
    If two client scripts try to bind to the same source UDP port on the same machine, the second one will likely fail unless the socket is opened in a special sharing mode. In most cases, only one process can bind to a particular port at a time. If both succeed using shared binding, there is a risk that incoming replies will be delivered nondeterministically, depending on how the OS distributes datagrams. This can cause one script to receive the other's response or for both to miss the reply entirely.
  </p>
</div>


## The Tunnel That Talks

### Background: UNIX Domain Sockets (Local Inter-Process Communication)

Most networked applications use **INET sockets**, which communicate over IP using TCP or UDP. But not all socket-based communication leaves the machine. For processes on the **same host**, the operating system provides an efficient alternative called a **UNIX domain socket**.

UNIX domain sockets are part of the socket API, just like TCP and UDP, but they do not use IP addresses or port numbers. Instead, they use **file system paths** to identify endpoints (e.g., `/var/run/docker.sock` or `/tmp/myapp.sock`).

Key properties of UNIX domain sockets:

* They are **used for local communication** only.
* They provide **fast, low-latency IPC** (no IP stack overhead).
* They use **the same system calls** (`socket()`, `bind()`, `listen()`, `accept()`, `connect()`) as TCP sockets.
* Permissions are managed via the **file system**, using standard UNIX file permissions (read, write, owner).
* They can operate in both **stream (SOCK\_STREAM)** and **datagram (SOCK\_DGRAM)** modes.

UNIX domain sockets are widely used by background services, daemons, and system utilities that expose an API for local clients to access. For example, Docker CLI talks to the Docker daemon via a socket file like `/var/run/docker.sock`. No TCP port is involved.


### Scenario

A student runs a command to start a local development database:

```bash
pg_ctl -D /usr/local/var/postgres start
```

Then, they try to connect to it using:

```bash
psql
```

Surprisingly, it works — even though they haven’t specified a port or host.

The student inspects the database log and sees:

```
connection received: host=[local]
```

Later, they explicitly run:

```bash
psql -h localhost
```

and the log now says:

```
connection received: host=127.0.0.1
```

This <span class="orange-bold">surprises</span> them. Both commands seem local, but the logs show different paths. They wonder what protocol is being used, and what exactly the difference is.


**Answer the following questions**:
1. How can a program like `psql` connect without specifying a host or port?
2. What is a UNIX domain socket, and how does it differ from a regular TCP socket?
3. When the log shows `host=[local]`, what does that imply about the transport layer?
4. Why does explicitly specifying `-h localhost` cause it to switch to using IP instead?
5. How does the OS determine which process should receive a message sent to a UNIX domain socket?
6. Why might a system choose to use UNIX sockets instead of TCP sockets for local access?

{: .highlight}
> **Hints**:
> * UNIX domain sockets are file-based, not port-based
> * The socket file must already exist and be bound by the server process
> * When using `-h localhost`, the program switches to TCP/IP loopback (127.0.0.1)
> * IPC using UNIX sockets avoids the overhead of the IP stack


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The `psql` client is able to connect without specifying a host or port because it defaults to using a UNIX domain socket. PostgreSQL creates a special socket file (often at `/tmp/.s.PGSQL.5432` or a similar path) which acts as the communication endpoint. If no `-h` is provided, `psql` connects to that socket file directly.
  </p>

  <p>
    A UNIX domain socket is a form of inter-process communication that stays entirely within the host machine. Unlike TCP sockets, which use IP addresses and port numbers, UNIX sockets are identified by file system paths. They use the same system calls and socket interface, but there is no network stack involved. As a result, they provide lower latency and tighter permission control.
  </p>

  <p>
    When the log says `host=[local]`, it means the connection was made using a UNIX domain socket. There was no TCP handshake or IP address used. The PostgreSQL server is simply accepting local connections via the socket file it created.
  </p>

  <p>
    When the user specifies `-h localhost`, the client uses the TCP/IP stack and connects to `127.0.0.1` on the standard PostgreSQL port (5432). This results in a real TCP connection, which is why the server log shows `host=127.0.0.1`. The presence of `-h` overrides the default UNIX socket behavior.
  </p>

  <p>
    The operating system manages UNIX domain socket communication by using the file path to look up the listening socket. If the path exists and a process has bound to it, the OS routes incoming data directly to that process. The permissions on the socket file determine who is allowed to connect.
  </p>

  <p>
    Systems often prefer UNIX domain sockets for local services because they avoid the overhead of TCP and IP, reduce attack surface, and support fine-grained permission control using standard UNIX file access rules. They also eliminate the need to reserve and manage TCP ports for local-only communication.
  </p>
</div>




### 🔧 UNIX Domain Socket vs TCP Loopback

Here is a **ladder diagram** that contrasts a **UNIX domain socket connection** with a **TCP loopback connection** for local database access.
#### Example 1: `psql` connects using **UNIX domain socket** (no `-h`)

```
psql                              PostgreSQL Server
  |                                     |
  |-- connect(/tmp/.s.PGSQL.5432) ----->|   ← uses socket file
  |                                     |
  |<------------- ACK ------------------|   ← no handshake, local pipe
  |-- query --------------------------->|
  |<-- response ------------------------|
```

* No IP or port involved
* OS checks permissions on the socket file
* Low latency, stays within kernel


#### Example 2: `psql -h localhost` connects using **TCP over loopback**

```
psql                              PostgreSQL Server
  |                                     |
  |-- SYN (127.0.0.1:50432 → 127.0.0.1:5432) -->
  |<-- SYN-ACK (5432 → 50432) -------------|
  |-- ACK -------------------------------->|
  |-- TCP query -------------------------->|
  |<-- TCP response -----------------------|
```

* Full TCP 3-way handshake
* Uses the loopback network interface
* Identified by IP and port


In a nutshell:
* UNIX domain sockets are used when no `-h` is given
* TCP loopback is used when `localhost` or an IP is specified
* Both allow local communication, but differ in transport layer and efficiency



## The Unexpected SYN

### Background: TCP Connection Lifecycle and State Transitions

TCP is a **connection-oriented protocol**. Before data can be exchanged, both parties must perform a **3-way handshake**, which transitions the connection through states:

1. **LISTEN** (server waiting for incoming connections)
2. **SYN_SENT** (client sends initial SYN)
3. **SYN_RECEIVED** (server received SYN, sends SYN-ACK)
4. **ESTABLISHED** (both sides have acknowledged)

After communication is done, both sides must **gracefully close** the connection. This leads through states like **FIN_WAIT**, **CLOSE_WAIT**, **TIME_WAIT**, and **CLOSED**.

A TCP socket must track the **connection state**, including sequence numbers, expected ACKs, and timers. If a **SYN** (which is supposed to begin a new connection) is received when the socket is already in another state, this is treated as **unexpected** and may be dropped or responded to defensively.



### Scenario

A server is running an application that accepts TCP connections on port 9090. A client connects, exchanges some data, and closes the connection. The server enters **TIME_WAIT** state to ensure all stray packets are handled.

Seconds later, the same client tries to reconnect quickly. The new SYN packet from the client reaches the server, but the server log shows:

```
[WARN] Dropped unexpected SYN from 192.0.2.5: disconnected state
```

This is puzzling. Isn’t SYN supposed to always start a new connection? Why would the server reject it?



**Answer the following questions**:
1. What does it mean for the server to be in TIME_WAIT, and why is it not ready to accept a new SYN from the same client?
2. How can a SYN packet be considered unexpected?
3. Why does TCP maintain the TIME_WAIT state after closing a connection?
4. What happens if a duplicate SYN arrives while a previous connection to the same (IP, port) is still in TIME_WAIT?
5. What would be different if the client used a new ephemeral port for the second connection?

{: .highlight}
> **Hints**:
> * TIME_WAIT helps avoid confusion from delayed or duplicate packets
> * The socket is identified by a 4-tuple: (src IP, src port, dst IP, dst port)
> * Reusing the same 4-tuple before TIME_WAIT expires can trigger conflicts

This is what actually happened:

```
Client (192.0.2.5:51000)       Server (203.0.113.10:9090)
        |                                |
        |-- SYN ------------------------>|
        |<-- SYN-ACK --------------------|
        |-- ACK ------------------------>|
        |       [Connection ESTABLISHED] |
        |-- FIN ------------------------>|
        |<-- ACK ------------------------|
        |       [Server enters TIME_WAIT]|
        |                                |
   [Client attempts reconnect using same ephemeral port]
        |-- SYN ------------------------>|
        |<-- (Dropped: port tuple still in TIME_WAIT) 
        |                                |
```


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The TIME_WAIT state exists to prevent old or duplicate TCP segments from interfering with new connections. When a connection is closed, the side that sends the final ACK enters TIME_WAIT for a period (typically 2 × Maximum Segment Lifetime, or about 60 seconds). During this time, the OS keeps the connection entry alive, even though the connection appears closed to the application.
  </p>

  <p>
    When a SYN arrives during this period, it may appear to be a new connection attempt, but if it uses the same 4-tuple (source IP, source port, destination IP, destination port) as the one still in TIME_WAIT, the server may reject it or drop it silently. This prevents ambiguity between old segments from the previous connection and new segments from a reused connection.
  </p>

  <p>
    TCP expects that sequence numbers from the old connection might still be floating in the network. If the server immediately accepts a new connection using the same 4-tuple, those old packets could be misinterpreted as part of the new session. The TIME_WAIT state ensures that any stray or duplicated packets from the previous session will be discarded before allowing reuse.
  </p>

  <p>
    If the client reuses the same ephemeral port to reconnect too quickly, the server sees the same 4-tuple as before and refuses the SYN. This is why the SYN is considered unexpected — the OS is still holding state from the previous connection and cannot safely start a new one with the same identifiers.
  </p>

  <p>
    If the client had instead selected a different ephemeral port, the new SYN would have a different 4-tuple. The server would treat it as a fresh connection unrelated to the one in TIME_WAIT and accept it normally. This is why most client OSes rotate ephemeral ports for each new outgoing connection.
  </p>
</div>


