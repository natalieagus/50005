---
layout: default
permalink: /ps/9-network-security
title: Network Security 
parent: Problem Set 
nav_order:  9
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

# Network Security  
{: .no_toc}


{:.warning}
The amount of questions for this topic is more than usual. Please explore them if you're interested. Some questions are more difficult and tricky, and may not be applicable to 50.005 (more applicable to cybersecurity), but they are interesting and has been stripped out of many technicalities. Hence we select them to be in this list. 


## Which Pillar Fails?

### Background: CIA

Cryptographic systems aim to ensure three main security goals:

* **Confidentiality**: Only the intended party can read the message.
* **Integrity**: The message wasn’t tampered with or altered in transit.
* **Authentication**: You’re communicating with the entity you think you are.

Sometimes, protocols break just one of these, <span class="orange-bold">and that’s all it takes</span>.

{:.important}
Recognize that **“secure” ≠ secure in all dimensions**.

### Scenario

For each situation below, identify **which security goal is violated**: Confidentiality, Integrity, or Authentication. <span class="orange-bold">More than one may apply</span>.


1. A message is encrypted using AES-CBC, but with a fixed IV reused every time.
2. A system uses SHA-256 to check for tampering but doesn't use any keys.
3. A client receives a valid-looking TLS certificate from a server, but doesn’t check if it's signed by a trusted CA.
4. A device receives a message with a valid HMAC, but the shared key is used by many other clients.
5. A report is encrypted and MACed, but the server doesn’t know who sent it or which client it came from.
6. A software update is signed by the vendor but downloaded over HTTP.
7. An IoT sensor encrypts and sends its data to the server. The key was hardcoded into the firmware and is now known.

<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p><strong>1.</strong> <u>Confidentiality</u>: Fixed IVs in AES-CBC reveal message structure or repeated plaintexts.</p>
  <p><strong>2.</strong> <u>Integrity</u>: SHA-256 is not a keyed function; attackers can modify the message and recompute the hash.</p>
  <p><strong>3.</strong> <u>Authentication</u>: The client cannot verify the server’s identity without validating the certificate chain.</p>
  <p><strong>4.</strong> <u>Authentication</u>: If multiple clients share the same MAC key, the server can't tell who sent the message.</p>
  <p><strong>5.</strong> <u>Authentication</u>: The server has no idea which client sent the message, even if the message is secure.</p>
  <p><strong>6.</strong> <u>Integrity + Confidentiality</u>: Signature ensures authenticity, but HTTP allows tampering or eavesdropping in transit.</p>
  <p><strong>7.</strong> <u>Confidentiality + Integrity</u>: If the key is exposed, an attacker can decrypt or forge messages entirely.</p>
</div>




## Just Enough Security?

Security is <span class="orange-bold">not</span> about applying *all* protections everywhere: it’s about **choosing** the **right protections** for the **right risks**. A messaging protocol might only need integrity. A broadcast alert might need only authentication. Encrypting everything doesn’t help if anyone can impersonate the sender.

In this problem, you’ll evaluate different **CIA property combinations** for real-world systems.

### Scenario

For each of the systems below, assume the following protections are applied. Your task is to determine:

* Is this **enough** security for the purpose?
* If not, **what’s missing** and **why**?


| System                                     | C | I | A | Is this enough? Why or why not? |
| ------------------------------------------ | - | - | - | ------------------------------- |
| 1. IoT broadcast alerts                    | ✗ | ✗ | ✔ |                                 |
| 2. Login form sent over HTTPS              | ✔ | ✔ | ✗ |                                 |
| 3. Digital voting (encrypted vote storage) | ✔ | ✗ | ✔ |                                 |
| 4. Anonymous whistleblower system          | ✔ | ✔ | ✗ |                                 |
| 5. API request with HMAC                   | ✗ | ✔ | ✔ |                                 |
| 6. Software update signed with vendor key  | ✗ | ✔ | ✔ |                                 |
| 7. Messaging app using AES-CTR only        | ✔ | ✗ | ✗ |                                 |
| 8. Medical record database with TLS        | ✔ | ✔ | ✔ |                                 |


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p><strong>1.</strong> <u>Enough</u>: IoT alerts are public, so confidentiality and integrity aren't critical if we only care that alerts come from a known source. Authentication is sufficient.</p>
  <p><strong>2.</strong> <u>Not enough</u>: Without authentication, the server doesn’t know who is logging in. Anyone could intercept the form and replay it.</p>
  <p><strong>3.</strong> <u>Not enough</u>: Without integrity, vote data might be silently altered. Confidentiality hides the vote, but integrity ensures it wasn't changed.</p>
  <p><strong>4.</strong> <u>Enough</u>: Authentication is intentionally omitted. The system is designed to preserve anonymity while ensuring privacy and tamper-resistance.</p>
  <p><strong>5.</strong> <u>Enough</u>: This setup ensures both message authenticity and integrity using HMAC. Confidentiality may not be necessary for public or low-sensitivity APIs.</p>
  <p><strong>6.</strong> <u>Enough</u>: The signature verifies source and ensures the update wasn’t modified. No confidentiality is needed since software is public.</p>
  <p><strong>7.</strong> <u>Not enough</u>: AES-CTR provides confidentiality, but without integrity or authentication, the message can be silently tampered or forged by attackers.</p>
  <p><strong>8.</strong> <u>Enough</u>: TLS provides all three: encrypted channel, tamper detection, and mutual or server-side authentication. This is appropriate for sensitive data.</p>
</div>


## The Curious Ciphertext

### Background: ECB 

Encryption is supposed to conceal the structure of a message. However, not all modes of encryption achieve this. **Electronic Codebook (ECB)** is one of the simplest block cipher modes. It encrypts each block of plaintext independently using the same key. While this makes it easy to implement and parallelize, it has a serious flaw: if two plaintext blocks are identical, their ciphertexts will also be identical.

This weakness was famously demonstrated using the **ECB Penguin**. A bitmap image of the Linux penguin (Tux) was encrypted using AES in ECB mode. Although the image data was encrypted, the resulting encrypted file still visibly resembled the original image. The structure was preserved because repeating image blocks were encrypted to *identical* ciphertext blocks. This became a widely used cautionary example of why **ECB mode is insecure for any structured data**.

<img src="{{ site.baseurl }}/docs/Problem Set/images/ecb-penguin.png" class="center_seventy"/>

{:.note}
You will also have hands-on experience with this issue during the Python Cryptography Lab.

### Scenario

You are analyzing traffic between a legacy client and a server. Messages are encrypted using **AES-128 in ECB mode** with a shared symmetric key. You capture two encrypted messages:

* `C1 = [0x5af3, 0x8c21, 0x5af3, 0x92b0, 0x5af3]`
* `C2 = [0x7da1, 0x5af3, 0x8c21, 0x8c21, 0x7da1]`

**You do not know the AES key or the plaintext, but the ciphertexts reveal repeating patterns.**

**Answer the following questions**:
1. What can you infer about the plaintexts of `C1` and `C2`?
2. Which security property is violated in this situation?
3. Would this problem still occur if the client switched to AES-CBC or AES-GCM mode with a new IV per message? Why or why not?
4. Why might a developer still use ECB mode despite its flaws?
5. How would you improve this system to ensure both confidentiality and integrity?

{: .highlight}
> **Hints**:
> * ECB encrypts each block independently.
> * Identical ciphertext blocks mean identical plaintext blocks.
> * Random IVs and chaining can break this pattern.
> * Integrity requires more than just encryption.


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The block <code>0x5af3</code> appears three times in C1 and once in C2, indicating repeated plaintext blocks. Similarly, <code>0x8c21</code> appears in both messages and twice in C2, suggesting a repeated message segment. The pattern <code>0x7da1</code> appears at both ends of C2, possibly framing the message.
  </p>
  <p>
    This reveals a violation of confidentiality. ECB mode leaks information about plaintext structure because it produces identical ciphertext blocks for identical plaintext blocks.
  </p>
  <p>
    This issue would not occur in CBC or GCM mode. Both introduce randomness via an initialization vector, and CBC chains ciphertext blocks together, so even repeated plaintexts result in different ciphertexts. GCM also adds authentication.
  </p>
  <p>
    Developers may still use ECB mode for its simplicity. It is easy to implement, requires no IV management, and supports parallel encryption. In legacy systems or low-security applications, this might be mistakenly deemed acceptable.
  </p>
  <p>
    The system should be updated to use AES-CBC or AES-GCM with a fresh IV per message. For full protection, AES-GCM is preferred because it provides both confidentiality and integrity. Key management and IV reuse prevention are essential.
  </p>
  <p>
    In a secure system, <span class="orange-bold">it is not enough to hide the content of a message</span>. The <strong>patterns</strong> within the data, such as repeated blocks or structures must also be <span class="orange-bold">obscured</span>, or attackers may infer sensitive information **without** ever decrypting the payload.
  </p>
</div>


{:.info}
> One of the factors that contributed to the cracking of the **Enigma** cipher during World War II was the consistent and predictable use of phrases like *"Heil Hitler"* at the end of many military messages. This **repeated** plaintext, known as a crib, allowed Allied cryptanalysts at Bletchley Park to make educated guesses about parts of the plaintext and align them with known ciphertext segments. 
> Because Enigma encrypted each letter **deterministically** and did not use secure modes to obscure repetition or structure, these known phrases provided critical footholds for deducing key settings. This demonstrates that even strong ciphers can be undermined when patterns in the plaintext are exposed, reinforcing the modern principle that secure systems must obscure structure as well as content.



## The Graph of Trust

### Background: Public Key Infrastructure Trust Chain

Public Key Infrastructure (PKI) allows users to verify the authenticity of entities using **digital certificates**. At the top of the hierarchy sits a **Certificate Authority (CA)**, whose public key is trusted implicitly. Every certificate issued by the CA (or by another entity with delegated signing rights) creates a **chain of trust**.

This can be modeled as a **directed graph**, where each node represents a public key, and an edge from A to B means "A has signed B’s public key." A public key is considered trusted if there exists a path from the root CA to that key. If no such path exists, the key and the identity it belongs to is not trusted by the system.

For example, in the graph below:

```
    [Root CA]
        ↓
       [X]
        ↓
       [Y]

```
If the client trusts only the Root CA, it can also trust X (because Root signed X) and Y (because X signed Y). But if a key is not reachable from the Root CA, the client cannot *verify* it and must treat it as **untrusted**.

{:.info}
In modern operating systems and browsers, a **chain of trust** is established through a built-in list of **trusted root certificate authorities (CAs)**. These root CAs are <span class="orange-bold">hardcoded</span> or <span class="orange-bold">bundled</span> with the **OS** or **browser** and are **implicitly trusted**. When a user connects to a website over HTTPS, the browser receives a digital certificate issued to that website, along with intermediate certificates if needed. The browser then verifies that this certificate was signed by a trusted CA or by an intermediate CA that itself has **a valid signature chain back to a trusted root**. If such a valid path exists, the certificate is considered trustworthy, and the connection proceeds securely. If the chain is broken or unverifiable, the browser warns the user or blocks the connection entirely.


### Scenario

You are given the following trust graph extracted from a distributed certificate system:

```
        [Root CA]
           |
          [A]
         /   \
      [B]    [C]
       |      |
      [D]    [E]
             /
          [F]
```

{:.note}
Each edge means the upper node **digitally signed the public key** of the lower node.

A client only trusts the **Root CA** initially. Your task is to evaluate which keys are trusted and which are not.

**Answer the following questions**:
1. Which of the nodes' public keys are trusted from the client's perspective?
2. Can the client verify a message signed by F? Why or why not?
3. Suppose a new signature is added: D signs C. How does this change the trust graph? Is C now trusted?
4. What is the risk of relying solely on this graph-based trust model in open networks?
5. How does a compromised node like A or C affect the rest of the graph?

{: .highlight}
> **Hints**:
> * Paths must originate from the Root CA to be trusted.
> * Signing someone’s key doesn’t imply the reverse.
> * Trust is transitive only if the signature chain is valid and verifiable.
> * Cycles do not break trust, but they don’t substitute for a root path.

<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The trusted keys are those with a valid signature path from the Root CA. These include A (signed by Root CA), B and C (signed by A), D (signed by B), and E (signed by C). F is <strong>not</strong> trusted because although E signed F, the client has no way to verify E’s key unless it trusts C, which it does.
  </p>
  <p>
    The client cannot verify a message signed by F, because there is no complete chain of signatures from the Root CA down to F. Although F’s key is signed by E and E by C, the client does not necessarily trust C unless the full path is valid and known.
  </p>
  <p>
    If D signs C, and D is already trusted via Root → A → B → D, then there exists a new path: Root → A → B → D → C. This makes C trusted by transitivity. Consequently, E and F would also become trusted, as they now fall within a trusted path.
  </p>
  <p>
    In open networks, relying only on transitive trust graphs poses risks. An attacker could insert fake keys or trick trusted nodes into signing unverified keys. Without strict policy enforcement or revocation mechanisms, the trust model can be exploited to elevate untrusted entities.
  </p>
  <p>
    If a node like A is compromised, then all entities it signed (B and C), and those signed downstream (D, E, F), become potentially untrustworthy. This is called <strong>trust propagation</strong>. A single compromised node can corrupt the integrity of a large subtree of the trust graph.
  </p>
</div>




## RSA in Action

### Background: RSA Arithmetic

RSA is a public key cryptosystem based on modular arithmetic. It uses a public key `(n, e)` for encryption or signature verification, and a private key `(n, d)` for decryption or signing. The security relies on the difficulty of factoring large numbers.

The keys are derived from two large primes, `p` and `q`:

* Compute `n = p × q`
* Compute Euler's totient `φ(n) = (p - 1)(q - 1)`
* Choose public exponent `e` such that `gcd(e, φ(n)) = 1`
* Compute private exponent `d` such that `e × d ≡ 1 (mod φ(n))`

Encryption: `c = m^e mod n`
Decryption: `m = c^d mod n`
Signing: `s = m^d mod n`
Verification: `m = s^e mod n`

{:.note}
Small examples can be used to practice and verify understanding, even though real RSA uses numbers with hundreds or thousands of bits.

### Scenario

A server uses RSA to secure client data. The public key is:
* `n = 221`
* `e = 11`

A client sends the encrypted message `c = 55`.

**Answer the following questions**:
1. Given that `n = 221` was generated using `p = 13` and `q = 17`, compute the private exponent `d`.
2. Decrypt the ciphertext `c = 55` to recover the original plaintext message `m`.
3. If a client wants to sign a message `m = 25`, what is the resulting signature `s`?
4. Why is RSA not used directly to encrypt large files or streams of data?
5. How does padding (like OAEP) strengthen RSA encryption?

{: .highlight}
> **Hints**:
> * Use Euler’s totient φ(n) = (p − 1)(q − 1)
> * Compute d such that (e × d) mod φ(n) = 1
> * For small numbers, try brute-force or use extended Euclidean algorithm
> * Modular exponentiation is key to RSA security




<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    First, compute φ(n) = (13 - 1)(17 - 1) = 12 × 16 = 192. We want to find d such that 11 × d ≡ 1 (mod 192). Testing small values, we find d = 35, because 11 × 35 = 385 and 385 mod 192 = 1.
  </p>
  <p>
    To decrypt c = 55, compute m = 55^35 mod 221. Using modular exponentiation, we find m = 48. This is the original plaintext.
  </p>
  <p>
    To sign m = 25, compute s = 25^35 mod 221. This results in s = 104. This is the digital signature.
  </p>
  <p>
    RSA is inefficient for large files because encryption and decryption involve expensive exponentiation with large numbers. Also, RSA can only operate on messages smaller than the modulus n. In practice, RSA is used to encrypt small secrets like symmetric keys, and the rest of the data is encrypted with a faster symmetric cipher.
  </p>
  <p>
    Padding schemes like OAEP add randomness and structure to the message before encryption, preventing attackers from exploiting deterministic output. Without padding, RSA is vulnerable to dictionary attacks and other forms of cryptanalysis.
  </p>
</div>


## The Compromised Broadcast

### Background: Digital Signatures

In many systems, messages are **digitally signed** but not encrypted. This ensures the receiver can verify the **authenticity** of the sender and the **integrity** of the message but the **contents remain visible** to anyone listening.

This model works well when confidentiality is not critical, such as in **software updates** or **public news feeds**. However, if an attacker can inject or replay old signed messages, systems that do not perform **freshness checks** may behave incorrectly.

Digital signatures verify that a message was not modified and was signed by the correct private key. They do not guarantee that the message is **recent**, **relevant**, or **safe to reuse**.

### Scenario

A command center broadcasts signed control messages to multiple unmanned aerial vehicles (UAVs). Each message includes the command and a digital signature using the control center’s private key. The messages are **not encrypted**, and each UAV accepts any message with a valid signature.

An attacker records the message:

```
"Return to base"  
Signature: valid
```


Hours later, the attacker **replays** this signed message <span class="orange-bold">during an ongoing operation</span>.

**Answer the following questions**:

1. What guarantees does the digital signature provide in this system?
2. Why is the replayed message still accepted by the UAVs?
3. What property is missing that allows this attack to succeed?
4. Suggest a design fix that preserves integrity but also prevents replay attacks.
5. If confidentiality was also required, what additional changes would you make?

{: .highlight}
> **Hints**:
> * Signatures verify origin and integrity, but not timeliness
> * Adding nonces, sequence numbers, or timestamps can help
> * Replay attacks exploit stateless or timestamp-free systems
> * Encryption can hide the message but does not replace signature checks


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The digital signature guarantees that the message was created by the control center and was not modified in transit. It ensures authenticity and integrity, but not freshness or context.
  </p>
  <p>
    The message is accepted because it still has a valid signature from the trusted sender. The UAVs do not distinguish whether the message is new or old, since there is no timestamp, nonce, or sequence number to indicate freshness.
  </p>
  <p>
    The missing property is <strong>message freshness</strong>. Without a way to detect whether a message is outdated or already processed, the system is vulnerable to replay attacks.
  </p>
  <p>
    A good fix would be to include a timestamp or sequence number in the message and require each UAV to track the most recent message it has processed. The signature must cover the timestamp or sequence number to prevent tampering.
  </p>
  <p>
    If confidentiality is also needed, the messages should be encrypted using a symmetric session key established beforehand, possibly via public key exchange. This ensures that only the intended UAVs can read the message, while digital signatures continue to ensure authenticity and integrity.
  </p>
</div>


## Replay Riches

### Background: Message Authentication Codes

Message Authentication Codes (MACs) are used to ensure **data integrity** and **authentication** over insecure channels. A MAC is computed over the message using a **shared secret key**, and the receiver verifies it to ensure the message has not been tampered with.

For example, when the client prepares the message "Pay SGD 100 to Merchant A", it computes a MAC using a cryptographic hash function (like SHA-256) combined with the shared secret. This produces a **fixed-length tag** that acts like a **fingerprint** for the message. The message and its MAC are sent together to the server. 
* Upon receiving the message, the server uses the same secret key to <span class="orange-bold">recompute</span> the MAC and compares it with the one received. 
* If they match, the server knows the message was not tampered with and came from someone who knows the key, *presumably* the legitimate client. 
 
However, if an attacker copies the exact same (message, mac) pair and sends it again, the server will perform the same MAC check and accept it, unless it has a way to detect that this is not a new request.

{:.important}
This shows that MACs **do not prevent replay attacks** by themselves. If a valid `(message, MAC)` pair is captured and sent again later, a system that doesn’t track freshness or ordering will accept it again, even if the original message was only intended to be used once.

### Scenario

An online payment system uses HMAC-SHA256 with a shared secret between the client and server. When a customer clicks “Pay SGD 100 to Merchant A,” the client sends:

```
message: "Pay 100 to Merchant A"  
mac: HMAC(secret, message)
```

The server **verifies** the MAC and **processes** the payment.

An attacker sniffs the network and captures this `(message, mac)` pair. Later, the attacker **resends** it to the server. Since the MAC is still valid, the server **processes the same SGD 100 payment again**.

**Answer the following questions**:
1. Why does the MAC still verify correctly on the second attempt?
2. What important security property is violated in this attack?
3. How can the server distinguish between original and replayed messages?
4. Suggest two different protocol changes that would prevent this replay.
5. Would encrypting the message fix the problem? Explain.

{: .highlight}
> **Hints**:
> * MACs only prove integrity and origin, <span class="orange-bold">not uniqueness or timing</span>
> * Nonces, timestamps, or counters are often used to track freshness
> * Encryption without integrity does not prevent replays
> * Think about stateful vs stateless designs


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The MAC verifies correctly because it was computed using the correct secret and message. Since the attacker simply replayed the exact same message and MAC, the server sees it as valid.
  </p>
  <p>
    The attack violates <strong>authentication freshness</strong>. While the MAC ensures that the message originated from the legitimate client and was not tampered with, it does not prove that the message is recent or intended to be processed again.
  </p>
  <p>
    The server needs a way to detect whether the message has already been processed. This can be done using <strong>unique transaction IDs, sequence numbers, or timestamps</strong> included in the message and verified against a record of previously seen values.
  </p>
  <p>
    One solution is to add a unique nonce or payment ID to each transaction and reject duplicates. Another is to include a timestamp in the message and enforce an expiration window, combined with MAC verification. In both cases, the freshness data must be included in the MAC computation.
  </p>
  <p>
    Encrypting the message without changing its contents or adding freshness data does not help. The attacker can replay the same ciphertext. Confidentiality does not imply integrity or uniqueness. Without tracking freshness, the system remains vulnerable to replays.
  </p>
</div>



## When Integrity Isn’t Enough

In some systems, data is encrypted or MACed correctly: ensuring confidentiality and integrity. But there’s no **authentication**: no proof of who sent the data. Sometimes this is acceptable (e.g. anonymous whistleblowing). In other cases, it leaves the system open to spoofing, impersonation, or replay attacks.

Understanding **what security property matters** in a context is as important as applying crypto correctly! In exam settings, you should read the requirements properly. 

### Scenario

An anonymous reporting tool allows users to submit incident reports via an HTTPS POST request. To protect privacy and detect tampering, the client:

* Compresses the report.
* Encrypts it using the server’s **public RSA key**.
* Computes a **SHA-256 hash** of the plaintext and includes it in a header.
* Sends the encrypted report and the hash to the server.

The server decrypts the message using its private key and checks the SHA-256 hash to ensure the report wasn’t tampered with.

**Answer the following questions**:
1. Does this scheme provide confidentiality? Why or why not?
2. Does it provide integrity? Are there any caveats?
3. Is authentication required in this use case? Why or why not?
4. What attacks, if any, are still possible?
5. Suggest modifications if this system were used for sensitive commands instead of anonymous reporting.

{: .highlight}
> **Hints**:
> * Think about: who can send? who is the recipient? does it matter?
> * SHA-256 alone doesn’t stop intentional forgery
> * Public-key encryption only proves receiver identity, not sender


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The system provides confidentiality because the message is encrypted using the server’s public RSA key. Only the server can decrypt it with its private key, so eavesdroppers cannot read the report.
  </p>
  <p>
    It offers basic integrity via the SHA-256 hash, but this is not sufficient to prevent intentional tampering. Anyone can recompute the hash over altered content and submit a forged pair. A secure integrity check would require a MAC using a secret key.
  </p>
  <p>
    Authentication is not required in this use case because the goal is anonymous reporting. The system is designed to protect the reporter’s identity, not verify it.
  </p>
  <p>
    However, the system is vulnerable to abuse — anyone (including attackers or bots) can submit fake reports or spam the server. The server also cannot trace repeated reports from the same source or limit malicious use.
  </p>
  <p>
    If this system were used for sensitive actions like issuing commands or submitting transactions, authentication would be essential. In that case, each client should authenticate using a digital signature or client-side certificate, and integrity should be enforced using HMAC or AEAD rather than plain hashes.
  </p>
</div>



## Sign Before You See

Digital signatures are used to assert that a piece of data originated from a trusted signer and has not been tampered with. However, **signing only proves that a private key was used**, and it does not imply that the signer agrees with or even understands the content unless proper verification steps are in place.

If a system blindly signs data without reviewing its contents, it can unintentionally become a **signing oracle**, enabling attackers to forge valid-looking claims *with the system’s authority behind them*.

In class, we learned that digital signatures provide <span class="orange-bold">non-repudiation</span>, meaning the signer cannot later **deny** having signed a message. But this property only proves that a particular private key was used, not that the signer read, understood, or agreed with the contents. In systems where users can submit arbitrary data for signing, non-repudiation **does not imply endorsement**. 

{:.important}
This distinction is critical: a signature may confirm origin and integrity, but without content verification, it says nothing about intent or consent.

### Scenario

A startup builds a blockchain-based document notarization service. Clients submit any document, and the server returns a **signed hash** of that document using the service’s private key. This is meant to prove that “the document existed at a certain time.”

One day, an attacker submits the following message:

```
"I, the owner of the service, hereby transfer 100% control of the company to the bearer of this document."
```

The server returns a valid signature over the hash of this message.

The attacker now presents the signed message in court, claiming it is legally binding, because it was **signed by the service’s private key**.

**Answer the following questions**:
1. What does the signature prove, and what does it not prove?
2. Why is this behavior dangerous for the server?
3. How could an attacker abuse such a system repeatedly?
4. What policy or technical measure could prevent this misuse?
5. If signing cannot be avoided, what disclaimers or structural safeguards should be added?

{: .highlight}
> **Hints**:
> * Signatures verify **authenticity** and **integrity**, but <span class="orange-bold">not endorsement</span>
> * In real life, systems must separate signing from *attestation* or *approval*
> * Think about user-submitted input and legal implications
> * Consider the difference between "notarizing" and "agreeing"


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The signature proves that the service's private key was used to sign the hash of the document. It confirms the document existed in that exact form at the time of signing. However, it does not prove that the service agrees with or endorses the content.
  </p>
  <p>
    This is dangerous because third parties — including courts — may interpret a digital signature as an endorsement or approval. If the service does not verify the contents, it can be tricked into signing malicious, misleading, or fraudulent claims.
  </p>
  <p>
    An attacker could repeatedly submit documents with deceptive claims or forged declarations, obtaining valid signatures that appear to come from a legitimate authority. These can then be used for impersonation, contract fraud, or misinformation.
  </p>
  <p>
    The server should never sign arbitrary user content without human or automated verification. Instead, it could require a formal request format, include metadata stating that the document was user-submitted, or use a separate signing key for unverified materials.
  </p>
  <p>
    If signing cannot be avoided, the signature should include a clear disclaimer like “This document was notarized but not reviewed or approved by the service.” Structurally, the service can sign only a standard header or hash block with explicit boundaries and timestamps to limit liability.
  </p>
</div>



## Timing the Leak


Encryption protects data by scrambling its contents, making it unreadable without the correct key. However, encryption often **does not hide metadata**, such as packet sizes, transmission timing, or connection frequency. In some cases, this metadata can be exploited to infer sensitive user behavior, even if the payload remains encrypted.

This is known as a **side-channel attack**, where information is leaked not from the decrypted message, <span class="orange-bold">but from how and when messages are sent</span>. Timing attacks, packet size analysis, and traffic shape inference are all examples.

This is especially relevant in high-performance systems like VPNs, secure routers, or TLS connections that still emit observable patterns.

### Scenario

A secure medical web app encrypts all its traffic using HTTPS. However, an attacker positioned between a patient and the server captures the following:

* Requests are **small** and **sporadic** during idle periods
* Suddenly, a large `POST` request is made
* Followed by a series of regularly sized responses over 15 minutes


The attacker, despite not decrypting any messages, guesses that the user has just started a **video consultation** with a doctor!

**Answer the following questions**:
1. How was the attacker able to infer user activity despite HTTPS?
2. What exactly was protected by encryption, and what was exposed?
3. Suggest a protocol-level mitigation to obscure such traffic patterns.
4. How does this relate to performance and buffering tradeoffs?
5. Would padding all packets to the same size solve this problem? Why or why not?

{: .highlight}
> **Hints**:
> * TLS protects content, not size, timing, or frequency
> * Traffic shaping or timing obfuscation can help
> * Bandwidth, latency, and CPU use are often affected by padding
> * Metadata can leak behavioral or contextual information


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The attacker used traffic patterns. Specifically, the sudden increase in payload size and consistent response timing to infer the **beginning** of a video session. This side-channel observation reveals user behavior even though the message content is encrypted.
  </p>
  <p>
    HTTPS protects the payload contents from being read or altered, but it does not conceal packet sizes, transmission times, or flow frequency. These aspects remain visible to any observer on the path between client and server.
  </p>
  <p>
    One mitigation is to use traffic shaping, such as sending fixed-size packets at regular intervals regardless of actual data, or adding dummy traffic to obscure the true communication pattern. VPNs and onion routing protocols sometimes apply this technique.
  </p>
  <p>
    These countermeasures reduce information leakage but introduce performance costs. Constant-rate transmission can increase latency and waste bandwidth, especially for low-traffic users. Systems must balance privacy with resource efficiency.
  </p>
  <p>
    Padding all packets to the same size only hides the size of individual messages, but not timing or frequency. It helps mitigate some pattern leaks but cannot eliminate all side channels unless combined with timing controls and randomized dummy traffic.
  </p>
</div>

{:.info}
> 🧅 Onion Routing 
>
> Onion routing is a technique used to achieve anonymous communication over a network by **layering encryption**, like the layers of an onion. In this model, a message is encrypted multiple times, each layer intended for a different node in a chain of relays. As the message passes through each relay, one layer of encryption is removed, revealing only the identity of the next node. No single relay knows both the sender and the final destination. This layered design prevents eavesdroppers and intermediaries from linking source and destination directly, making onion routing a core technology behind systems like Tor, which aim to provide privacy, anonymity, and resistance to traffic analysis.
>
> Read more about it [here](https://nordvpn.com/blog/onion-routing/) if you are interested.




## Fast Enough?

**Not all systems require full confidentiality, integrity, and authentication**. Sometimes, performance is critical, and you only need one or two of the CIA properties: especially for low-power embedded systems, real-time messaging, or public broadcasts.

{:.note}
Choosing the right **cryptographic tool** requires understanding both **what you need to protect** and **how fast different primitives perform**.

### Scenario

A company is designing a **high-frequency event stream** from IoT sensors (like temperature spikes or door opens). These alerts are **broadcast** to a shared cloud server every 50 ms from hundreds of devices. The data is **not secret**, but the server must know:

* The message came from a legitimate device (authentication).
* The message wasn’t tampered with (integrity).
* But speed is <span class="orange-bold">critical</span>! The devices are low-power and the cloud must handle high load.

The team is evaluating the following combinations:

| Option | Crypto Used                           | Provides                  | Throughput |
| ------ | ------------------------------------- | ------------------------- | ---------- |
| A      | HMAC-SHA256                           | Integrity, Authentication | Medium     |
| B      | AES-GCM with random nonce             | Conf, Integrity, Auth     | Low        |
| C      | Ed25519 signature + plaintext message | Authentication only       | High       |
| D      | CRC32 checksum + AES-CTR              | Conf only                 | Very High  |
| E      | Plaintext + SHA-256 digest            | Integrity (unkeyed)       | High       |

**Answer the following questions**:
1. Which option satisfies the system’s needs without over-protecting?
2. Which option is **cryptographically insufficient**? Why?
3. If option C is used, how can the server verify that a message was not replayed?
4. Suppose the message includes a timestamp and device ID — how does that affect security?
5. What tradeoffs would justify choosing option A over option C?

{: .highlight}
> **Hints**:
> * Think about whether **integrity** without keys is enough
> * Signatures are slower than HMACs but provide public verifiability
> * Does AES make sense if you don’t need secrecy?


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    Option <strong>C (Ed25519 signature + plaintext)</strong> is the best match. It provides <strong>authentication</strong> (proves the sender’s identity) and implicit integrity (a valid signature won't verify over a tampered message), without encrypting or adding unnecessary overhead. Its throughput is high and fits the system’s need for speed and authenticity.
  </p>
  <p>
    Option <strong>E</strong> is cryptographically insufficient. A SHA-256 digest is not keyed, so an attacker can tamper with the message and recompute the digest. There's no way to detect forgery or impersonation.
  </p>
  <p>
    To prevent replays in option C, each message can include a <strong>monotonic timestamp or nonce</strong>. The server can keep track of recently seen values to reject duplicates.
  </p>
  <p>
    Including a timestamp and device ID helps the server trace events and prevent replay attacks, but the timestamp must be signed — otherwise, an attacker could forge a message with fake timing.
  </p>
  <p>
    Option <strong>A (HMAC-SHA256)</strong> is also valid but uses a symmetric key. It's suitable when both parties share a secret key securely. It’s slightly slower but more efficient on constrained devices that can't do public-key cryptography. It would be chosen if the signer must be very lightweight or if key rotation is tightly managed.
  </p>
</div>



## The Hijacked Port

### Background: Port Numbers

At the transport layer, protocols like TCP use **port numbers** to distinguish between different services on the same host. A port is like a logical **endpoint** on a machine (or virtual machine) as we learned before, for example, port 443 is used for HTTPS, while port 22 is for SSH.

When an application binds to a port, it listens for incoming connections. The **OS kernel maps packets arriving at that port to the application socket** that owns it.

However, if an attacker manages to bind to a port **before the legitimate service**, they may receive sensitive incoming traffic. This is known as a **port hijacking** or **pre-binding attack**. <span class="orange-bold">Even if encryption is in place, the traffic is misrouted</span>.

### Scenario

A hospital's internal system includes a backend service that encrypts sensitive patient reports and sends them to a reporting client on port 7000. The client program is supposed to be launched first and binds to port 7000 to receive the data.

However, a local attacker runs a malicious program that binds to port 7000 **before** the legitimate client. The server now sends the encrypted reports to the attacker's process.

{:.note}
Although the attacker cannot decrypt the reports, they **store all incoming traffic** for future cryptanalysis or potential key compromise.

**Answer the following questions**:
1. Why is the attacker able to receive the data despite the server using encryption?
2. What OS-level mistake allowed this hijack to succeed?
3. Suggest a design that prevents unauthorized processes from hijacking critical ports.
4. How could a process authenticate the receiving party before sending encrypted data?
5. What general security principle is violated in this scenario?

{: .highlight}
> **Hints**:
> * Encryption doesn't stop traffic from going to the wrong destination
> * Port access is typically first-come, first-served unless controlled
> * Consider capabilities, user privilege, or handshake authentication
> * Think about endpoint validation, not just encryption


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The attacker is able to receive the data because they bound to the expected port before the legitimate client did. The server does not verify who is listening, as it simply sends data to port 7000 on the client machine, assuming it’s correct.
  </p>
  <p>
    The OS allowed any user-level process to bind to port 7000. There was no access control or reservation mechanism preventing unprivileged or malicious programs from occupying the port first.
  </p>
  <p>
    One solution is to run the legitimate client as a privileged service that reserves the port at boot time. Alternatively, the OS could enforce capability-based controls, only allowing certain programs or users to bind to specific ports.
  </p>
  <p>
    To prevent this class of attack, the sending process should not blindly trust the port. It should require an authenticated handshake before transmitting sensitive data. Mutual TLS, signed challenges, or application-layer tokens can validate the receiver’s identity before sending encrypted payloads.
  </p>
  <p>
    This scenario violates the principle of **secure binding** and **trust minimization**. Encryption alone is not sufficient as the sender must ensure that messages are delivered to the **correct**, **authenticated** recipient before assuming the channel is secure.
  </p>
</div>



## One Hash Too Many

### Background: Collision in Hashing

Most digital signature algorithms (like RSA or ECDSA) do not sign the full message directly. They sign a **cryptographic hash (digest)** of the message. This is faster and allows signatures to handle variable-length input.

However, this approach assumes the **hash function is collision-resistant**, that is, it should be computationally infeasible to find two different messages `m1 ≠ m2` such that `hash(m1) = hash(m2)`.

If a signer **blindly signs the hash** without knowing what the original message was, and if the hash function is weak, an attacker could create a **collision pair**: one harmless document and one malicious document with the same digest. The signer sees and signs the harmless one, but the attacker presents the malicious one with the valid signature.

{:.note}
This has occurred in the real world with **MD5** and **SHA-1**.

### Scenario

A notary service agrees to sign any file by computing its SHA-1 hash and returning an RSA signature over that hash. A user submits:

> **File A**: “Agreement between Alice and Bob to pay \$1.”

The service signs `SHA1(File A)` and returns the signature.

Later, Bob presents:

> **File B**: “Agreement between Alice and Bob to pay \$1,000,000.”

The two files have different contents but were **carefully crafted to collide under SHA-1**. The signature for File A is now valid for File B.

**Answer the following questions**:
1. Why does this attack succeed, even though RSA is mathematically secure?
2. What specific property of SHA-1 was exploited?
3. How could the service have prevented this attack, even while still using RSA?
4. Should hash functions be trusted forever once approved? Why or why not?
5. How do modern signing standards defend against this kind of forgery?

{: .highlight}
> **Hints**:
> * Think about **collision resistance** vs **pre-image resistance**. Search it online or read [this](https://natalieagus.github.io/50005/ns/05-network-security-app#hash-function--requirements) section of the notes.
> * Signing a digest assumes trust in the hash function
> * Consider double hashing or domain separation in modern schemes
> * Look up SHA-2, SHA-3, and collision exploits in MD5 or SHA-1


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The attack succeeds because the notary signed only the SHA-1 hash of the file, and the attacker submitted a file that was part of a known collision pair. RSA correctly signed the hash, but that hash was shared by both the harmless and malicious documents.
  </p>
  <p>
    This exploited a weakness in SHA-1's <strong>collision resistance</strong>, which means an attacker can generate two different messages with the same hash. Once a collision is found, the same signature validates both.
  </p>
  <p>
    The service should have stopped using SHA-1 and moved to a stronger hash function like SHA-256 or SHA-3. It should also sign structured messages or use modern signing protocols that include metadata to prevent interchangeable content.
  </p>
  <p>
    Hash functions should not be trusted indefinitely. Cryptanalysis improves over time, and once collisions or weaknesses are discovered, continued use opens systems to forgery attacks like this. MD5 and SHA-1 are now considered insecure.
  </p>
  <p>
    Modern digital signature standards (like RSASSA-PSS or ECDSA over SHA-256) use padding, randomized hashing, or structured input formats that prevent attackers from creating meaningful collisions. They also typically include protocol-specific context to bind the signature to its intended use.
  </p>
</div>


## Padding Oracle

### Background: CBC and the Attack

In block cipher modes like **CBC (Cipher Block Chaining)**, encryption requires plaintext to be a multiple of the block size (e.g., 128 bits). When the message is shorter, it’s padded (e.g., using PKCS#7). On decryption, the receiver removes the padding and checks if it's valid.

If an attacker can send **tampered ciphertexts** to a server and observe whether the **padding check fails or succeeds**, they can learn information about the plaintext, one byte at a time. This is known as a **padding oracle attack**.

Even though the encryption algorithm (e.g., AES) is secure, **leaking error information** gives attackers a powerful side channel.

### Padding Oracle Attack Example

Assume AES-CBC with 16-byte blocks and **PKCS#7** padding. The server receives **two ciphertext blocks**:

```
C1: [AA BB CC DD EE FF 00 11 22 33 44 55 66 77 88 99]
C2: [FA 5B 91 00 4C 8D 3A 7E 21 76 9A 0B 17 29 5C 01]   ← Last byte ends in 0x01
```

Let’s say `C2` is the **final block**, and `C1` is the **IV for decrypting C2**.

On decryption:

1. The server computes `P2_raw = AES_decrypt(C2)`. Suppose this yields (not known to attacker): `P2_raw = [CB A1 9E 77 5A 33 B2 C7 1B C4 12 A0 1D 55 4F 04]`
2. Then it computes `P2 = P2_raw XOR C1`. (This produces the actual plaintext block).
3. The server then checks the **padding**. The last byte of `P2` is `0x04`. If all of the last 4 bytes are `0x04`, the padding is valid.


#### The Attack

The attacker **cannot decrypt** the block directly but **can modify C1** and resend the ciphertext to the server.

They:

* Flip the **last byte** of `C1` (e.g., change `0x99` → `0x98`)
* Send `[modified C1] + C2` to the server
* Observe the response:
  * If the server returns **“invalid padding”**, it means the new last byte of `P2` is not a valid `0x01`
  * If it returns **“MAC failed”** (but padding passed), attacker knows they hit a valid padding (`0x01`)

By doing this repeatedly, they figure out what byte must be in `C1` to make `P2[-1] = 0x01` (the last byte, i.e. index -1, of that P2 plaintext block), allowing them to compute:

```
P2_raw[-1] = C1[-1] XOR P2[-1]
           = known_value XOR 0x01
```

Since attacker knows what they flipped `C1[-1]` to, they recover `P2_raw[-1]`, and from there compute `P2[-1]`.

They repeat for second-last byte (`0x02 0x02`), third-last, etc., recovering the entire last block **byte by byte**, without decrypting AES.

### Scenario

A server receives encrypted data over HTTPS. Internally, it decrypts incoming ciphertext using AES-CBC and returns an error if the padding is invalid:

* `Error 1: Invalid padding`
* `Error 2: MAC verification failed`

An attacker sends a modified ciphertext and observes the error message. By systematically tweaking ciphertext blocks and watching for **“Invalid padding”** vs **“MAC failed”**, the attacker recovers one byte of plaintext at a time without ever knowing the encryption key.

**Answer the following questions**:
1. What design flaw allows the attacker to decrypt the message without the key?
2. How does the attacker learn about the plaintext from the padding error?
3. What change to the server’s behavior could prevent this attack?
4. Would switching to AES in ECB mode solve the problem? Why or why not?
5. What modern cipher modes avoid this vulnerability?

{: .highlight}
> **Hints**:
> * Look at how CBC decrypts and applies XOR with previous block
> * PKCS#7 padding is predictable and can be brute-forced
> * Don’t reveal whether padding or MAC failed
> * Authenticated encryption (AEAD) is designed to prevent this


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    The attack works because the server leaks internal validation steps. By returning different error messages for padding failure and MAC failure, it creates a decryption oracle that the attacker can probe.
  </p>
  <p>
    The attacker manipulates the ciphertext and resends it. If the error is “invalid padding,” the guess was wrong. If the error changes, they know the padding is valid, revealing information about the original plaintext through CBC’s XOR structure. Repeating this narrows down the correct byte.
  </p>
  <p>
    The server should return a generic error message regardless of whether the padding or MAC check failed. It should process both steps (decrypt, then MAC verify) before failing, without revealing where it failed.
  </p>
  <p>
    Switching to AES-ECB would prevent the padding oracle, but at the cost of leaking plaintext structure. ECB mode is insecure because it produces identical ciphertext blocks for identical plaintext blocks.
  </p>
  <p>
    A better fix is to use a modern authenticated encryption mode like AES-GCM or ChaCha20-Poly1305 (please search it online on your own if you are interested), which combines encryption and integrity checking in a way that avoids such side channels entirely.
  </p>
</div>


## Man in the Middle Manager

### Background: MITM

Encryption ensures that data cannot be read by third parties, but it does **not** guarantee who you're talking to. Without authentication, such as verifying a **TLS certificate** or **SSH host key**, a user can be tricked into connecting to an attacker who intercepts the communication, decrypts it, and passes it along. This is a **man-in-the-middle (MITM)** attack.

These attacks are especially dangerous in corporate or public Wi-Fi environments, where attackers control the local network and can impersonate legitimate services.

{:.note-title}
> TLS Certificate
>
> When your browser connects to a secure website, it receives a TLS certificate that contains the website’s public key, signed by a Certificate Authority (CA), which you have learned in class. The CA’s signature assures the browser that this public key really belongs to the claimed domain. The private key, which proves ownership of that public key, stays hidden on the server and is never sent over the network. The client never sees the private key it only trusts that the public key is valid because a trusted CA signed it.

{:.note-title}
> SSH TOFU
> 
> In SSH, trust is established differently than TLS. When you connect to a server for the first time, your SSH client receives the server’s **host public key** and typically asks if you want to trust it. If you accept, the key is saved to a local `known_hosts` file. On future connections, the client checks that the server presents the **same key**. If it changes, the client warns you that the server might be compromised or spoofed. 
> 
> This model is called **Trust On First Use (TOFU)**. You trust the key the first time you see it and watch for unexpected changes afterward. More secure setups preload trusted host keys or use SSH certificates signed by a known authority.



### Scenario

A startup employee connects to the company’s internal admin dashboard over HTTPS. The domain `internal.startup.com` resolves locally, and the browser warns:

> ⚠ Your connection is not private. Proceed anyway?

The employee clicks "Proceed" to get their work done. Unbeknownst to them, a disgruntled IT manager has set up a rogue server with a **self-signed certificate** and is intercepting all requests. Because the employee **skipped certificate validation**, all login credentials, internal configs, and messages are now exposed — despite the use of HTTPS.

**Answer the following questions**:
1. Why did encryption fail to protect the user’s credentials?
2. What should the client have verified before sending sensitive data?
3. How does a self-signed certificate enable MITM?
4. Suggest two ways to prevent such attacks in real-world networks.
5. Why is SSH’s “first time connect” model risky? How can it be hardened?

{: .highlight}
> **Hints**:
> * **TLS** needs both encryption and authentication
> * Browsers use CA trust stores; SSH uses known\_hosts
> * Always verify the identity of the other party
> * Public key pinning, HSTS, or TOFU models can help




<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    Encryption failed because the client skipped the certificate validation step. Without authenticating the server’s identity, the user had no guarantee they were speaking to the legitimate server, and the encrypted connection was established with the attacker instead.
  </p>
  <p>
    The client should have verified that the server presented a valid TLS certificate signed by a trusted certificate authority (CA), matching the domain `internal.startup.com`. This ensures the encryption is not just secure, but also with the correct party.
  </p>
  <p>
    A self-signed certificate allows the attacker to act as their own CA. Since the attacker controls the network and DNS resolution, they can present a certificate for the expected domain, and unless the client rejects it, the connection appears legitimate.
  </p>
  <p>
    To prevent such attacks, organizations should use internal CAs and install them on client devices, or configure strict HSTS and certificate pinning policies. Networks should also monitor for unauthorized TLS endpoints and enforce mutual TLS when possible.
  </p>
  <p>
    SSH uses a "trust on first use" (TOFU) model where the host key is stored after the first connection. This is risky if an attacker intercepts the initial connection. It can be hardened by preloading known host keys, verifying fingerprints manually, or using SSH certificates with a trusted authority.
  </p>
</div>




## Dual Use Danger

### Background: Danger of Reusing Keys

In secure systems, it's common to use both **encryption** (for confidentiality) and **MACs** (for integrity/authentication). Best practice is to use **separate keys**: one for encryption, and another for the MAC. This separation ensures that the operations remain **independent** and resistant to key-reuse attacks.

However, if the same key is used for both purposes, for example, AES for encryption and HMAC-SHA256 for integrity, the security of both mechanisms can be compromised. 

{:.important}
This is called a **dual-use key** and is explicitly warned against in cryptographic standards.

### Scenario

An IoT device uses a symmetric key `K` shared with the server to both:

* Encrypt sensor data using **AES-CBC with K**
* Compute a **MAC over the ciphertext using HMAC-SHA256 with K**

To save memory, the designers reuse the same key `K` for both. An attacker captures many encrypted messages along with their MACs. They cannot decrypt the ciphertext but begin crafting their own messages and observing how the MAC behaves under trial guesses.

Eventually, they exploit the key reuse to forge a valid ciphertext-MAC pair, which is accepted by the server, bypassing both **confidentiality** and **integrity** checks.

**Answer the following questions**:
1. What’s wrong with using the same key for both encryption and MAC?
2. How does key reuse increase the risk of forgery?
3. What’s the cryptographic principle violated here?
4. Suggest a safer design using symmetric cryptography.
5. Why do AEAD ciphers like AES-GCM avoid this issue entirely?

{: .highlight}
> **Hints**:
> * Think about domain separation: MAC and encryption have different input/output structures
> * Reusing keys gives attackers more known input/output pairs
> * AEAD combines encryption and authentication securely in one mode


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    Using the same key for both encryption and MAC violates the principle of key separation. The operations have different mathematical structures and security assumptions, and combining them under a single key creates unintended interactions.
  </p>
  <p>
    Key reuse gives the attacker more information: they can observe how the same key behaves under both encryption and MAC, and use chosen-ciphertext or chosen-message attacks to narrow down the key space or forge valid outputs.
  </p>
  <p>
    This breaks the principle of <strong>domain separation</strong>: different cryptographic functions should use independent keys or at least ensure they’re distinguishable by context.
  </p>
  <p>
    A safer design would use two separate keys derived from a master secret: one for AES-CBC encryption (`K_enc`) and one for HMAC-SHA256 (`K_mac`). Even better, keys should be generated or derived using a secure KDF with labels to indicate their role.
  </p>
  <p>
    AEAD ciphers like AES-GCM or ChaCha20-Poly1305 (please search online on your own) handle both encryption and authentication in one integrated process using a single key, but they’re carefully **designed** to avoid **collisions** and interaction flaws. This eliminates the risk of dual-use key mistakes.
  </p>
</div>




## Crypto Soup

{:.info}
This question is challenging. It requires knowledge from various domains of cybersecurity where you are given a scenario and are supposed to find cryptographic weakness in the design. 


In real-world systems, cryptographic components are rarely isolated. They’re mixed together: encryption, signatures, MACs, key exchanges, often under **time pressure** and <span class="orange-bold">without full understanding</span> of their interactions. Even if individual parts seem "secure," poor integration can undermine everything.

Understanding how to **audit a design** holistically is a crucial skill in system security.

### Scenario

Your team is reviewing a proposed design for an encrypted messaging protocol used in a legacy IoT platform:
> 1. Each device has a shared symmetric key `K` with the server.
> 2. Messages are encrypted using **AES-CBC with PKCS#7 padding**.
> 3. The **same key `K`** is also used to compute **HMAC-SHA1** over the ciphertext.
> 4. IVs are generated using the system time in seconds.
> 5. Devices cache a signed “firmware OK” certificate from the server — signed using **RSA with SHA-1**.
> 6. Devices accept the certificate as long as the signature is valid, even if it's **expired or revoked**.

You’re asked to comment on the design before deployment.

**Answer the following questions**:
1. Identify **at least three serious cryptographic weaknesses** in this design.
2. For each weakness, explain how an attacker could exploit it.
3. Propose specific improvements or modern alternatives for each component.
4. How do these flaws interact to make the system more vulnerable than the parts suggest?
5. Why is **secure integration** just as important as secure algorithms?

{: .highlight}
> **Hints**:
> * Think about randomness, key reuse, padding oracles, hash collisions
> * Think about system lifecycle: key rotation? certificate expiry?
> * Consider what an attacker can replay, forge, or predict
> * Evaluate the entire stack, not just the cipher


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    First, using the <strong>same key</strong> `K` for both AES-CBC encryption and HMAC violates key separation. An attacker who collects enough ciphertext-MAC pairs may mount forgery or chosen-ciphertext attacks, especially if padding oracles are present.
  </p>
  <p>
    Second, generating IVs from the <strong>system time</strong> makes them predictable. If two messages are sent within the same second, they may reuse the IV — breaking semantic security and potentially revealing patterns in the ciphertext.
  </p>
  <p>
    Third, the use of <strong>SHA-1 for HMAC and RSA signatures</strong> is deprecated due to collision vulnerabilities. An attacker might craft multiple inputs with the same hash, potentially forging a signature or a MAC.
  </p>
  <p>
    Fourth, the system <strong>does not check certificate revocation or expiry</strong. Even if a key is compromised or a certificate is outdated, the device continues trusting it — opening the door to persistent attacks or firmware forgery.
  </p>
  <p>
    These flaws reinforce each other: predictable IVs make ciphertext patterns visible; reused keys make forged ciphertexts more likely; and missing certificate checks allow the attacker to persist in the system once any part is compromised.
  </p>
  <p>
    The system should use <strong>separate keys</strong> for encryption and MAC, switch to <strong>AES-GCM or ChaCha20-Poly1305</strong> for AEAD, use <strong>cryptographically random IVs</strong>, migrate to <strong>SHA-256 or SHA-3</strong>, and implement <strong>revocation checking</strong> (e.g., OCSP or short-lived certs). Secure integration means applying each primitive correctly, and ensuring they work together under realistic threat models.
  </p>
</div>




## Patch the Handshake

{:.info}
This question is challenging. It requires knowledge from various domains of cybersecurity where you are given a scenario and are supposed to find cryptographic weakness in the design <span class="orange-bold">and then fix</span> the broken protocol. 

### Background

Designing a secure protocol is difficult. Even if you use standard crypto primitives like RSA or AES, incorrect sequencing, missing authentication, or assumptions about trust can make the system vulnerable. Many real-world attacks exploit these flaws — not broken algorithms, but **broken glue**.

Being able to spot and repair insecure protocols is a key skill in applied cryptography.

### Scenario

Below is a simplified handshake protocol between a client and a server to establish a secure session key `K_session`:

1. Client → Server: `Hello`
2. Server → Client: `RSA_enc(K_session, Server_PublicKey)`
3. Client and server both use `K_session` for AES encryption.

The goal is to establish a shared symmetric key using the server’s public RSA key. However, the protocol is vulnerable to several attacks. There’s **no client authentication**, no signature or certificate involved, and no proof that the server actually owns the private key.

**Answer the following questions**:
1. Identify two major vulnerabilities in this protocol.
2. How could a man-in-the-middle attacker exploit this exchange?
3. Propose a revised handshake that defends against these attacks.
4. Should the server sign anything? Should the client verify anything?
5. How do TLS and SSH avoid these design mistakes?

{: .highlight}
> **Hints**:
> * Think about certificate validation, forward secrecy, and key confirmation
> * Can you replace RSA with a modern key exchange like ECDHE?
> * What happens if an attacker intercepts and rewrites the handshake?


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    One major vulnerability is the lack of server authentication. The client accepts any RSA public key without verifying its origin. A man-in-the-middle attacker can replace the server’s public key with their own and decrypt the session key.
  </p>
  <p>
    A second issue is the lack of key confirmation. The client sends an encrypted session key and assumes the server received it, but there’s no handshake step where the server proves it possesses the private key or the session key.
  </p>
  <p>
    An attacker could intercept the "Hello" message, substitute their own key, and forward the rest of the protocol as a proxy — fully decrypting and re-encrypting traffic. This is a textbook MITM attack.
  </p>
  <p>
    To fix the protocol, the server should send an <strong>X.509 certificate</strong> signed by a trusted CA to prove its identity. The client should verify this certificate before encrypting anything. Even better, use an ephemeral key exchange (like ECDHE), with both parties contributing randomness, and include signatures to authenticate the exchange.
  </p>
  <p>
    TLS uses signed ephemeral key exchanges and certificates to authenticate the server, with optional client authentication. SSH uses known host keys or certificates, and includes mutual key confirmation during setup. Both protocols defend against passive and active MITM attacks by authenticating all public parameters and confirming possession of private keys.
  </p>
</div>




## The Slippery Protocol

### Background

A good cryptographic protocol should ensure **confidentiality** (no one else can read), **integrity** (no one can tamper undetected), and **authentication** (you know who you're talking to). Many flawed designs violate one or more of these goals: not through broken ciphers, but due to bad assumptions or missing steps.

Understanding what each part of a protocol contributes, and which goal it secures, is essential.

### Scenario

A custom-designed secure file transfer system uses this protocol:
1. The client compresses a file.
2. The client encrypts the file using **AES-CBC with a pre-shared key `K`** and a **fixed IV of all zeroes**.
3. The client sends the ciphertext and a **SHA-256 hash of the plaintext file**.
4. The server decrypts the ciphertext and compares the hash to check for tampering.

**Answer the following questions**:
1. Does this protocol guarantee confidentiality? Why or why not?
2. Does it provide integrity? What weaknesses exist?
3. Is authentication ensured between client and server?
4. What are the specific technical flaws in this design?
5. Propose improvements that enforce all three goals correctly.

{: .highlight}
> **Hints**:
> * Fixed IV + CBC → **predictable** ciphertexts
> * SHA-256 is not a MAC
> * Who proves they’re who they claim to be?
> * How do modern protocols bundle encryption + integrity?


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">
  <p>
    Confidentiality is weakened due to the use of a <strong>fixed IV</strong> in AES-CBC. If the same file is encrypted twice, the ciphertext will be identical. An attacker can detect file reuse or infer structural patterns, breaking semantic security.
  </p>
  <p>
    Integrity is not properly enforced. A <strong>SHA-256 hash of the plaintext</strong> is not a secure message authentication code (MAC). An attacker can tamper with the ciphertext, and unless the tampered data results in the same hash post-decryption, the server won’t detect the change.
  </p>
  <p>
    There is <strong>no authentication</strong. The server doesn’t verify the client’s identity, nor does the client verify the server. Anyone who knows the shared key `K` (or intercepts the file and hash) could send spoofed messages.
  </p>
  <p>
    Specific technical flaws include the use of <strong>deterministic IVs</strong>, lack of <strong>keyed integrity checking</strong (no HMAC or AEAD), and absence of any <strong>identity verification mechanism</strong> such as certificates or digital signatures.
  </p>
  <p>
    To fix this, use a modern <strong>AEAD scheme</strong> like AES-GCM or ChaCha20-Poly1305, which ensures both encryption and integrity in one operation. Replace SHA-256 with an HMAC if separate integrity is needed. To add authentication, include client and server certificates or signed tokens as part of the handshake. The IV should be randomly generated for each encryption session and transmitted safely.
  </p>
</div>
