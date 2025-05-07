---
layout: default
permalink: /labs/06-ping-and-traceroute
title: Ping and Traceroute
description:  Experiment with ICMP packets
parent: Labs
nav_order:  6
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

# Ping and Traceroute
{:.no_toc}


In this lab exercise, you will learn how to use `ping` and `traceroute` programs to measure round trip times and find network routes.

<span style="color:#f77729;"><b>Learning Objectives</b></span>:

- <span style="color:#f77729;"><b>Explain</b></span> how ping and traceroute utilities work.
- Use the ping utility to <span style="color:#f77729;"><b>measure</b></span> network round trip times.
- Use the traceroute utility to find network <span style="color:#f77729;"><b>routes</b></span>.
- Observe and understand the effects of <span style="color:#f77729;"><b>varying</b></span> packet sizes on delays experienced.

You will need both ping and traceroute to be installed on your OS. Most Ubuntu or macOS installations should already include ping by default. You can install traceroute by running `sudo apt install traceroute` from the command line.

# Submission
You are to complete this lab's questionnaire on eDimension as you complete the tasks.


No checkoff required for this lab. 

# RTT Measurement using ping

The `ping` utility is one of the most widely-used network utilities. It enables you to <span style="color:#f77729;"><b>measure</b></span> the time that it takes for a packet to travel through the Internet to a remote host _and back_.

The `ping` utility works by sending a **short** message, known as an <span style="color:#f77729;"><b>echo request</b></span> to a remote host using the Internet Control Message Protocol (ICMP).When a host that supports ICMP receives an echo-request message, it replies by sending an echo-response message back to the originating host.
{:.info}


In this first part of this lab exercise, you will use the `ping` utility to send echo requests to a number of _different_ hosts. In many of the exercises, you will be referring to hosts using their <span style="color:#f77729;"><b>domain name</b></span> rather than their IP addresses[^1]. For more information about ping, you can look up its manual page by running `man ping` from the command line.

The following info is relevant for the next few tasks:

- `ping netflix.com` is the easiest and simplest way to ping a server. It will continuously send packets and print out the response (if any). You may press `ctrl+c` to terminate it.
- It supports several options:
  - `-c [no_of_packets]`: specify the number of packets that should be sent by `ping` before terminating (otherwise it will continue forever until `SIGINT`[^2] is sent by `ctrl+c`).
  - `-s [packet_size]`: set packet size. The default is 56.
  - `-i [seconds_interval]`: interval of ping packets sent to the destination

## Round-Trip Time

The `ping` utility can be used to measure the round-trip time (RTT).

Round-trip time (<span style="color:#f77729;"><b>RTT</b></span>) is the duration, measured in milliseconds, <span style="color:#f77729;"><b>from</b></span> when a browser sends a request <span style="color:#f77729;"><b>to</b></span> when it receives a response from a server.
{:.info}

RTT is one of the <span style="color:#f77729;"><b>key performance metric</b></span> for web applications.

### Task 1

`TASK 1:` Use `ping` to send 10 packets (56 bytes each) to each of the following hosts, and there should be an interval of 5 seconds between each packet sent,
{:.task}

- www.csail.mit.edu
- www.berkeley.edu
- www.usyd.edu.au
- www.kyoto-u.ac.jp

The size of each packet is 56 bytes by default, but you may observe that the actual size of the packet is <span style="color:#f77729;"><b>larger</b></span> than 56 bytes. You can look up the manual for ping to understand <span style="color:#f77729;"><b>why</b></span> such a discrepancy exists.

Fill up the table below in edimension to key in your answer.

| Website           | Successfull % | Min RTT | Ave RTT | Max RTT |
| ----------------- | ------------- | ------- | ------- | ------- |
| www.csail.mit.edu |               |         |         |
| www.berkeley.edu  |               |         |         |
| www.usyd.edu.au   |               |         |         |
| www.kyoto-u.ac.jp |               |         |         |

Also, go to this [online ping test site](https://tools.keycdn.com/ping) and ping www.csail.mit.edu

{:.highlight-title}
> Question
> 
> From whom do you receive replies? You can get the IP address and use the command `whois [ip_address]`

### Task 2

`TASK 2:` Repeat the exercise from Task 1 using packet sizes of 512 and 1024 bytes. Record the <span style="color:#f77729;"><b>minimum</b></span>, <span style="color:#f77729;"><b>average</b></span>, and <span style="color:#f77729;"><b>maximum</b></span> round trip times for each of the packet sizes with a table like the previous task, and head to edimension to key in your answer.
{:.task}

{:.highlight-title}
> Question
> 
> <span style="color:#f77729;"><b>Why</b></span> are the minimum round-trip times to the same hosts _different_ when using 56, 512, and 1024–byte packets?

## Unanswered pings

### Task 3

`TASK 3:` Use ping to send 100 packets to the following host: www.wits.ac.za
{:.task}

Each packet should have a size of 56 bytes, and there should be an interval of 5 seconds between each packet sent.

<span style="color:#f77729;"><b>Record</b></span> the percentage of the packets sent that resulted in a successful response for each host.

{:.highlight-title}
> Question
> 
> What are some possible reasons why you may not have received a response? (Be sure to check the host in a <span style="color:#f77729;"><b>web browser</b></span>).

# Traceroute 

The `traceroute` utility is another useful network utility. It enables you to trace the route taken by a packet from your machine to a remote host.

Note that if traceroute doesn’t work on your VM, you may:

- Add the `-I` option: `traceroute -I [ip]`
- Or, use results from `tracert` (assuming Windows is your host OS).

Here is an example of the output produced when traceroute is used to trace the route taken by a packet to www.mit.edu:

<img src="{{ site.baseurl }}/assets/images/nslab1/1.png"  class="center_full no-invert"/>

The first line of the traceroute output describes what the command is set for. It lists the <span style="color:#f77729;"><b>destination</b></span> system (e9566.dscb.akamaiedge.net), destination IP address (184.50.104.236), and the maximum number of hops that will be used in the traceroute (64).

The remainder of the output shows information on <span style="color:#f77729;"><b>each hop</b></span>, where each line is a reply from (typically) a router, _in the path between the sender and the final destination_.

It is <span style="color:#f77729;"><b>important</b></span> to note that the number of hops is <span class="orange-bold">not</span> an important factor that affects latency.
{:.note}

Each of these lines begins with a host (e.g router) <span style="color:#f77729;"><b>IP</b></span> on the route from your computer to www.mit.edu, followed by the round-trip time (<span style="color:#f77729;"><b>RTT</b></span>) for <span style="color:#f7007f;"><b>3</b></span> packets sent to that host.

For more information about `traceroute`, you can look up its manual page by running `man traceroute` from the command line.

### Task 4

`TASK 4:` Find out how `traceroute` works. You will need this to answer several questions on eDimension.
{:.task}

{:.new-title}
> Hint  
> 
> `traceroute` sends a UDP packet to the destination host's (highly likely) _unusable_ port, with increasing TTL. The routers that reduces the TTL to 0 will send an ICMP TTL Exceeded reply. The end host will send an ICMP Port unreachable reply.

## Route Asymmetries

The route taken to send a packet from your machine to the remote host machine is <span style="color:#f77729;"><b>not always the same</b></span> with the route taken to send a packet from the remote machine _back_ to you.

In this exercise, you will run traceroute in two <span style="color:#f77729;"><b>opposite</b></span> directions. First, you will run `traceroute` on a remote host to see the route taken <span style="color:#f77729;"><b>to your network</b></span>. Then, you will also run `traceroute` from your computer to see the route taken to that host.

### Task 5

`TASK 5:` Find out your computer’s public IP address. (Hint: You can use a website like [this](http://www.whatismypublicip.com/), or search for “what is my ip” using Google’s search engine.)
{:.task}

### Task 6

`TASK 6:` Visit [this link](https://ping.eu/traceroute/) using your web browser.
{:.task}

Then do the following:
- Enter your computer’s public IP address as shown in the site 
- Enter the captcha code, then press enter to start a traceroute to your computer's public IP
- Take a <span style="color:#f77729;"><b>screenshot</b></span> of the output

If the output shows that the packet _does not reach your IP_ (request timed out), think about a reason or two on why this is so.

The school might block the website. You can utilise your phone hotspot instead. You’re free to use other similar sites that performs traceroute from their server to your computer's public IP address. If you have two devices with different IPs (e.g: one uses VPN), then you can also traceroute each other’s IP addresses.
{:.warning}

### Task 7

`TASK 7:` After `traceroute` finishes running, you should be able to view the route taken from <span style="color:#f77729;"><b>specified locations</b></span> to your network.
{:.task}

Record the IP address of the <span style="color:#f77729;"><b>first</b></span> visible hop which will be used in the next step.

In the screenshot below, that will be `213.239.245.241` for example.

<img src="{{ site.baseurl }}/docs/Labs/images/06-Lab6-Ping/2024-04-06-13-33-24.png"  class="center_full"/>


You can <span style="color:#f77729;"><b>check</b></span> who that remote host is using the command `whois [ip address]`, for instance, 	`213.239.245.241` is indeed described as being in Germany (DE).

<img src="{{ site.baseurl }}//docs/Labs/images/06-Lab6-Ping/2024-04-06-13-35-52.png"  class="center_full no-invert"/>
### Task 8

`TASK 8:` On your computer, run `traceroute` using the IP address recorded in the previous step as the remote destination.
{:.task}

<img src="{{ site.baseurl }}//docs/Labs/images/06-Lab6-Ping/2024-04-06-13-37-18.png"  class="center_full no-invert"/>


{:.highlight-title}
> Question
> 
> Are the <span style="color:#f77729;"><b>same</b></span> routers traversed in both directions? If no, could you think of a reason why? 

# Summary 
In this lab, we have explored two network utilities: ping and traceroute. This shall help you explain the effects of varying packet sizes on delays experienced. 

<hr>

[^1]: A domain name is an easy-to-remember alias used to access websites. For example, we access netflix by typing netflix.com and not the actual netflix server's public IP address. For more information, see [here](https://www.cloudflare.com/en-gb/learning/dns/glossary/what-is-a-domain-name/).
[^2]: In POSIX-compliant OS, the default action for `SIGINT`, `SIGTERM`, `SIGQUIT`, and `SIGKILL` is to terminate the process. However, `SIGTERM`, `SIGQUIT`, and `SIGKILL` are defined as signals to terminate the process, but `SIGINT` is defined as an <span style="color:#f77729;"><b>interruption</b></span> requested by the <span style="color:#f77729;"><b>user</b></span>.

