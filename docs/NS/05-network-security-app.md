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


# Case 1: Plain Message 
<img src="{{ site.baseurl }}/docs/NS/images/04-network-security/cse2024-case1.drawio.png"  class="center_seventy"/>

Suppose A would like to communicate with B via the internet. Without any cryptography features, an intruder T can send a fake message to B (claiming that it's sent by A). There’s no way B can differentiate whether the message comes from T or the intended host A:
* Authentication is compromised
* Integrity is also compromised
* Confidentiality is also breached

# Case 2: Plain Message + IP Header 

In an attempt to prove A's identity, A will send a message to B along with their IP. However without any encryption T knows A's IP and will be able to craft a message with A's IP and send it B.  B won't be able to differentiate whether the message comes from T or A:
* Authentication is compromised
* Integrity is also compromised
* Confidentiality is also breached

<img src="{{ site.baseurl }}/docs/NS/images/04-network-security/cse2024-case2.drawio.png"  class="center_seventy"/>

{:.error-title}
> Spoofing
> 
> Faking one’s IP address like what T has done here is called **spoofing**. It's very simple to do. Head to [appendix](#spoofing-ip-address) to see a simple example using Python. 

# Case 3:  Plain message + IP header + “secret” password

In yet another attempt to prove A's identity, suppose A and B both share a secret codeword. A sends their IP + the secret codeword + message to B. Unfortunately, there's <span class="orange-bold">nothing secret</span> on the internet. An intruder T may **eavesdrop** on the conversation.

{:.error-title}
> Eavesdrop Attack
> 
> This attack involves eavesdropping the messages exchanged between A and B and <span class="orange-bold">use the information for malicious intent</span>. 


<img src="{{ site.baseurl }}/docs/NS/images/04-network-security/cse2024-case3.png"  class="center_seventy"/>

T can echo back A’s IP + A’s “secret” password to B, but with a new message that does not come from A. Similarly, B cannot differentiate whether the message comes from A or T. Hence still without any form of encryption:
* Authentication is compromised
* Integrity is also compromised
* Confidentiality is also breached


{:.note}
To perform an eavesdrop attack, the attacker T does not even need to be in the same network. If the attacker is on the same local network as the target, it's easier to perform packet sniffing because the attacker can directly intercept network traffic passing through the local network segment. This could be done by connecting to the same Wi-Fi network, being physically connected to the same Ethernet switch, or being on the same subnet. While being on the same network can simplify eavesdropping attacks, attackers can still intercept communication remotely by targeting specific network nodes or exploiting vulnerabilities in network infrastructure.  Head to [appendix](#eavesdropping) if you'd like concreate examples on how eavesdropping is done. 


# Case 4: Encrypted message + encrypted password + IP header

{:.important}
In this scenario, Symmetric Key Cryptography is used since it's faster than Asymmetric Key Cryptography.

Now suppose both the message and "secret message" between A and B are now encrypted using symmetric key cryptography, such as 3DES. Even though now T cannot tell what the "secret message" between A and B is, nor can it read the content of the message. However, they can just **record** and **replay** or **playback** the cipher to B. This is called the <span class="orange-bold">replay/playback attack</span>.

<img src="{{ site.baseurl }}/docs/NS/images/04-network-security/cse2024-case4.drawio-4.png"  class="center_seventy"/>

B is still not able tell if the incoming message + encrypted secret message comes from A or the intruder T. In this particular example, T does not get an extra $5000, but they damage B by incurring financial loss. The unauthorized duplicate transaction can disrupt legitimate financial transactions, cause financial losses for customers, and erode trust in the banking system and institutions involved. 

{:.note-title}
> Playback vs replay attack
>
> Both replay attacks and eavesdropping attacks involve <span class="orange-bold">intercepting</span> network communication. Replay attacks involve intercepting and later re-transmitting captured network data to impersonate a legitimate user, while eavesdropping attacks involve passive interception of communication to extract sensitive information without altering it. Replay attacks aim to deceive the target system by replaying captured data, whereas eavesdropping attacks focus on listening in on communication to <span class="orange-bold">extract confidential</span> information.

# Case 5: Signed message + Signed Nonce

{:.important}
In this scenario, Asymmetric Key Cryptography is used. Assume that confidentiality is **not** a requiremement (not critical)

Case 4's issue is that it is susceptible to replay attack. To tackle this issue, B **wants** to authenticate A and ensure the **liveliness** of A. That is, if A sends the message "Transfer me $5000", B wants to make sure that it is indeed coming from A *right now*. 

{:.info-title}
> Nonce
> 
> To do this, host A should **digitally sign** (encrypts with a private key) a 1-time generated number called **Nonce** (Number used only once).

The scenario goes as follows:
* A generates a public-private key pair
* <span class="orange-bold">B would like to authenticate A</span> (make sure that A is A), and so B sends Nonce N to A
  * Note: B to authenticate A, <span class="orange-bold">NOT</span> the other way around
  * When B <span class="orange-bold">authenticates</span> A, B is sure that A is <span class="orange-bold">live</span> (and B is not communicating with some replayed / stale message)
* A then encrypts Nonce with their private key, producing a signed nonce 
* A also encrypts the message with their private key, producing a signed message
* Then A sends the signed nonce along with A's public key to B + signed message
* B verifies the Nonce using A’s public key, and check that it is indeed the Nonce N sent to A previously
  * This confirms that the message is **fresh** (not a replay message)
* B decrypts the signed message and acts upon it.  
* B may send **confidential** message to A encrypting the reply message m with A's public key. This way, only A can decipher the message.


There's still one issue with this: there's <span class="orange-bold">no way</span> that B can tell that the public key belongs to A. This is susceptible to the <span class="orange-bold">man-in-the-middle attack</span> (MITM). In other words, T  is able to **hijack** the conversation between A and B:
<img src="{{ site.baseurl }}/docs/NS/images/04-network-security/cse2024-case5.drawio-8.png"  class="center_seventy"/>

B's intended "confidential" message to A is also susceptible to MITM:
<img src="{{ site.baseurl }}/docs/NS/images/04-network-security/cse2024-case5b.drawio-2.png"  class="center_seventy"/>

{:.error-title}
> Man in the middle attack
>
> In a Man-in-the-Middle (MITM) attack, an adversary intercepts communication between two parties, allowing them to **eavesdrop** on or **manipulate** the exchanged data. By impersonating one or both parties, the attacker can capture sensitive information and potentially alter the communication without detection.

There are **two solutions** to avoid the MITM attack: to utilise symmmetric key cryptography or to rely on Certificate Authority. 

## Utilise Symmetric Key Cryptography 

This scenario can also be modified using symmetric key cryptography. Host A will encrypt the Nonce using a pre-established “shared key”. Upon receiving the message encrypted by a symmetric key that <span class="orange-bold">B knows only A has</span>, B can confirm (to himself) that A wrote the message. 

T can no longer hijack the conversation because simply it does not have the symmetric key. However, this solution requires A and B to both <span class="orange-bold">meet</span> and share the key in person, so it is not exactly an ideal solution. 


## Certificate Authority 

{:.info}
A Certificate Authority (CA) is a **trusted** entity responsible for issuing digital certificates that verify the identities of individuals, organizations, or devices on the internet. These certificates bind cryptographic keys to entities, ensuring **secure communication** through protocols like SSL/TLS by validating the authenticity of parties and enabling encrypted data exchange.

<img src="{{ site.baseurl }}//docs/NS/images/05-network-security-app/2024-05-02-22-00-46.png"  class="center_seventy"/>

The steps goes as follows: 
* A has to obtain a **certificate** from trusted CA
  * As shown in the figure above, a certificate contains: **public key**, **identity information**, **issuer information**, **validity period**, and more 
  * Refer to [Appendix](#certificate) if you'd like to find out more 
* CA **verifies** that A exists, checks A’s documents etc
* CA **issues a certificate** that the Public Key of A indeed belongs to A
* This certificate is **signed** by CA’s **private** key
* CA’s public key is **widely** known, so we trust that nobody can impersonate the CA
* B can obtain A’s public key from the certificate by decrypting it using the CA’s widely known public key

{:.important-title}
> How is CA's public key widely known? 
>
> 
> CA's certificates (also known as Root certificates) containing CA's public key are typically pre-installed with both web browsers and operating systems, providing users with a <span class="orange-bold">trusted foundation</span> for validating SSL/TLS certificates during secure internet communication. This inclusion ensures that users can securely access websites and services without needing to manually manage trust settings.
>
> The client (e.g., web browser) verifies the SSL/TLS certificate's validity by checking if it's signed by a trusted root certificate. If the certificate chain can be traced back to a root certificate stored in the client's trust store, the certificate is considered valid.
>
> This analogous to us trusting government-issued ID. 

To this end, A CA obviously has to be a trusted entity such as: government (e.g., IDA) or well known provider. Real-life examples of trusted CAs are: **IdenTrust, Comodo, GlobalSign, DigiCert, GoDaddy**, and many more. Modern browsers will warn you if some website’s certificates are <span class="orange-bold">not</span> verified. Below is an example of a legal certificate of netflix.com’s public key, signed by DigiCert as its CA. 

<img src="{{ site.baseurl }}//docs/NS/images/05-network-security-app/2024-05-02-22-01-56.png"  class="center_seventy"/>

On the other hand, below is an example of non-trusted certificate:

<img src="{{ site.baseurl }}//docs/NS/images/05-network-security-app/2024-05-02-22-17-31.png"  class="center_seventy"/>

Let's now review Case 5 again with the addition of CA: 

<img src="{{ site.baseurl }}/docs/NS/images/04-network-security/cse2024-case5c.drawio-4.png"  class="center_seventy"/>

The addition of a CA enables B to be sure that A’s public key indeed belongs to A. However it is *still* forcing A or B to encrypt messages with asymmetric key cryptography. Case 5 + CA nonetheless provides:
* **Authentication**: The Nonce ensures the message signed and sent by A is fresh. In other words, B <span class="orange-bold">authenticates</span> A and ensures the <span class="orange-bold">liveliness</span> of A. The messages B received from A comes from A *right now*.
* **Confidentiality**: From B to A only. Asymmetric Key Cryptography ensures that only A can read the message.
* **Integrity**: A has a CA **signed** certificate, hence B can verify that the signed message was authorized by A without alteration.

<img src="{{ site.baseurl }}/docs/NS/images/04-network-security/cse2024-case5d.drawio-2.png"  class="center_seventy"/>


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

## Certificate

A certificate contains several pieces of information, including:

1. **Public Key**: The public key of the entity (e.g., a website) that the certificate is issued to. This key is used for encryption and verifying digital signatures.

2. **Identity Information**: Information about the entity the certificate is issued to, such as its domain name, organization name, and location.

3. **Issuer Information**: Information about the Certificate Authority (CA) that issued the certificate, including its name and digital signature.

4. **Validity Period**: The period during which the certificate is considered valid, including its start and expiry dates.

5. **Certificate Serial Number**: A unique identifier assigned to the certificate by the issuing CA.

6. **Key Usage**: Specifies the cryptographic operations (e.g., encryption, digital signature) that the public key in the certificate can be used for.

7. **Signature Algorithm**: The algorithm used by the CA to sign the certificate.

8. **Digital Signature**: A digital signature generated by the CA using its private key to bind the certificate's contents to the CA's identity, ensuring the certificate's authenticity.

9. **Extensions**: Additional information or attributes associated with the certificate, such as subject alternative names (SANs) for SSL/TLS certificates.

10. **Certificate Chain**: For SSL/TLS certificates, the certificate may also include the intermediate certificates that link the entity's certificate to the root certificate of the CA.

Overall, a certificate serves as a digital identity card, providing information about the entity it represents and enabling secure communication through cryptographic mechanisms.