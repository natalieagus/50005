---
layout: default
permalink: /ns/05-network-security-app
title: Application Scenarios of Cryptography
description: Several scenarios highlighting importance of cryptography
parent: Network Security
nav_order: 4
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



# Application Scenarios of Cryptography

Previously, we list out four security properties that are essential for secure communication over the network:
1. Confidentiality
2. Integrity
3. Authentication
4. Access and Availability

In this section, we are going to learn *how* cryptography can be applied to provide for property 1-3. To provide for access and availability, cryptography alone will be unable to prevent attacks that cripples access and availability. We need other algorithms, such as **DNS caching** to prevent DDoS attack to provide for access and availability in our system. We will learn about this in the next chapter. 

For each of the cases described below, consider two end hosts, `A` and `B` who would like to communicate via the internet, and another `T`, who is a <span class="orange-bold">malicious</span> intruder. 


 
