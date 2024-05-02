---
layout: default
permalink: /ns/05-network-security-app
title: Application Scenarios of Cryptography
description: Several scenarios highlighting importance of cryptography
parent: Network Security
nav_order: 5
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


## Case 1: Plain Message 
<img src="{{ site.baseurl }}/docs/NS/images/04-network-security/cse2024-case1.drawio.png"  class="center_seventy"/>

Suppose A would like to communicate with B via the internet. Without any cryptography features, an intruder T can send a fake message to B (claiming that it's sent by A). There’s no way B can differentiate whether the message comes from T or the intended host A:
* Authentication is compromised
* Integrity is also compromised
* Confidentiality is also breached

## Case 2: Plain Message + IP Header 

In an attempt to prove A's identity, A will send a message to B along with their IP. However without any encryption T knows A's IP and will be able to craft a message with A's IP and send it B.  B won't be able to differentiate whether the message comes from T or A:
* Authentication is compromised
* Integrity is also compromised
* Confidentiality is also breached

<img src="{{ site.baseurl }}/docs/NS/images/04-network-security/cse2024-case2.drawio.png"  class="center_seventy"/>

{:.error-title}
> Spoofing
> 
> Faking one’s IP address like what T has done here is called **spoofing**. It's very simple to do. Head to [appendix](#spoofing-ip-address) to see a simple example using Python. 

## Case 3:  Plain message + IP header + “secret” password

In yet another attempt to prove A's identity, suppose A and B both share a secret codeword. A sends their IP + the secret codeword + message to B. Unfortunately, there's <span class="orange-bold">nothing secret</span> on the internet. An intruder T may **eavesdrop** on the conversation.

{:.error-title}
> Eavesdrop Attack
> 
> This attack involves eavesdropping the messages exchanged between A and B and <span class="orange-bold">use the information for malicious intent</span>. 


<img src="{{ site.baseurl }}/docs/NS/images/04-network-security/cse2024-Copy of case2.drawio.png"  class="center_seventy"/>

T can echo back A’s IP + A’s “secret” password to B, but with a new message that does not come from A. Similarly, B cannot differentiate whether the message comes from A or T. Hence still without any form of encryption:
* Authentication is compromised
* Integrity is also compromised
* Confidentiality is also breached


{:.note}
To perform an eavesdrop attack, the attacker T does not even need to be in the same network. If the attacker is on the same local network as the target, it's easier to perform packet sniffing because the attacker can directly intercept network traffic passing through the local network segment. This could be done by connecting to the same Wi-Fi network, being physically connected to the same Ethernet switch, or being on the same subnet. While being on the same network can simplify eavesdropping attacks, attackers can still intercept communication remotely by targeting specific network nodes or exploiting vulnerabilities in network infrastructure.  Head to [appendix](#eavesdropping) if you'd like concreate examples on how eavesdropping is done. 


## Case 4: Encrypted message + encrypted password + IP header

Now suppose both the message and "secret message" between A and B are now encrypted using symmetric key cryptography, such as 3DES. Even though now T cannot tell what the "secret message" between A and B is, nor can it read the content of the message. However, they can just **record** and **replay** or **playback** the cipher to B. This is called the <span class="orange-bold">replay/playback attack</span>.

<img src="{{ site.baseurl }}/docs/NS/images/04-network-security/cse2024-case4.drawio-2.png"  class="center_seventy"/>

B is still not able tell if the incoming message + encrypted secret message comes from A or the intruder T. In this particular example, T does not get an extra $5000, but they damage B by incurring financial loss. The unauthorized duplicate transaction can disrupt legitimate financial transactions, cause financial losses for customers, and erode trust in the banking system and institutions involved. 

{:.note-title}
> Playback vs replay attack
>
> Both replay attacks and eavesdropping attacks involve <span class="orange-bold">intercepting</span> network communication. Replay attacks involve intercepting and later re-transmitting captured network data to impersonate a legitimate user, while eavesdropping attacks involve passive interception of communication to extract sensitive information without altering it. Replay attacks aim to deceive the target system by replaying captured data, whereas eavesdropping attacks focus on listening in on communication to <span class="orange-bold">extract confidential</span> information.


# Appendix

## Spoofing IP Address

To send a message with a spoofed IP address in Python, you can use the `scapy` library, which allows for crafting and sending packets at a low level. Here's a simple example:

```python
from scapy.all import *

# Craft an IP packet with a spoofed source IP
fake_ip = "192.168.1.100"
spoofed_packet = IP(src=fake_ip, dst="www.example.com") / ICMP()

# Send the packet
send(spoofed_packet)

```

In this example:
- We import the `scapy` library, which provides functions for crafting and sending packets.
- We craft an IP packet with a spoofed source IP address (`src="192.168.1.100"`) and a destination IP address (`dst="www.example.com"`).
- We add an ICMP payload to the packet using `/ ICMP()` to demonstrate a simple payload.
- Finally, we use the `send()` function to send the spoofed packet.

## Eavesdropping

An eavesdrop attack, also known as passive eavesdropping or sniffing, involves intercepting and monitoring communications between two parties without their knowledge. Here's how an eavesdrop attack is typically done:

1. **Packet Sniffing**: The attacker uses specialized software or hardware tools to capture network traffic passing through a network interface. This can be done by placing the sniffing device on a network segment or by compromising a network device.

2. **Promiscuous Mode**: The attacker configures the sniffing device to operate in promiscuous mode, allowing it to capture all network packets, including those not intended for the attacker's device. In this mode, the network interface card (NIC) ignores the MAC address filtering and captures all packets on the network segment.

3. **Packet Analysis**: Once the attacker captures network packets, they analyze the captured data to extract sensitive information such as usernames, passwords, credit card numbers, or other confidential data transmitted in plaintext.

4. **Decrypting Encrypted Traffic**: If the intercepted communication is encrypted, the attacker may attempt to decrypt it using various cryptographic attacks, such as brute force attacks, dictionary attacks, or exploiting cryptographic vulnerabilities.

5. **Passive Nature**: One key characteristic of eavesdropping attacks is their passive nature. The attacker does not actively modify the intercepted data but merely listens in on the communication between the legitimate parties.

6. **Stealthy Operation**: Eavesdropping attacks are often difficult to detect since they do not disrupt the normal operation of the network. Unless proper security measures are in place, the attacker can silently intercept sensitive information without raising suspicion.

Overall, eavesdropping attacks pose a significant threat to the confidentiality of communication over a network, highlighting the importance of using encryption and other security measures to protect sensitive data from interception.

### Tools for eavesdropping
Several applications and tools can be used for eavesdropping attacks, commonly known as packet sniffing or network sniffing tools. Here are some examples:

1. **Wireshark**: Wireshark is a widely-used network protocol analyzer that allows users to capture and interactively browse the traffic running on a computer network. It supports hundreds of protocols and can capture data from live networks or read data captured from a file.

2. **tcpdump**: tcpdump is a command-line packet analyzer available for Unix-like operating systems. It captures packets traversing a network interface and displays them in real-time or saves them to a file for later analysis.

3. **Ettercap**: Ettercap is a comprehensive suite for man-in-the-middle attacks on LAN. It features sniffing of live connections, content filtering on the fly, and many other interesting tricks. It supports active and passive dissection of many protocols and includes many features for network and host analysis.

4. **Cain & Abel**: Cain & Abel is a password recovery tool for Microsoft Windows. It allows easy recovery of various kinds of passwords by sniffing the network, cracking encrypted passwords using dictionary, brute-force, and cryptanalysis attacks, recording VoIP conversations, decoding scrambled passwords, and analyzing routing protocols.

5. **dsniff**: dsniff is a collection of tools for network auditing and penetration testing. It includes sniffing tools like dsniff, filesnarf, mailsnarf, msgsnarf, URLsnarf, and others that can be used for eavesdropping on different types of network traffic.

6. **WinPcap**: WinPcap is a library for Windows operating systems that provides low-level network access. It is used by many packet sniffing applications on Windows, including Wireshark and tcpdump for Windows.

These tools are often used by network administrators, security professionals, and attackers alike to analyze network traffic, troubleshoot network problems, or exploit security vulnerabilities. However, it's important to note that using these tools for unauthorized access to network traffic or without proper authorization can be illegal and unethical. Always use these tools responsibly and with the appropriate permissions.