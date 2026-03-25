---
title: Programming Assignment 2 
permalink: /pa2/intro-c
key: pa2-intro
layout: default
nav_order: 2
parent: Programming Assignment
has_children: true
---

* TOC
{:toc}

**50.005 Computer System Engineering**
<br>
Information Systems Technology and Design
<br>
Singapore University of Technology and Design
<br>
**Natalie Agus (Summer 2026)**

# Programming Assignment 2

{:.info-title}
> Main Task
> 
> In this assignment, you will implement a **secure** file upload application from a **client** to a secure file **server**. You will design and code the secure File Transfer Protocol (FTP) into the application.

_Secure_ means that it fulfiles these **three** requirements simultaneously:
1. **First**, before you do your upload as the client, you should **authenticate** the identity of the file server so you won't leak your data to random entities including criminals.
2. **Secondly**, you want to ensure that you're talking to a **live** server.
3. **Thirdly**, while carrying out the upload you should be able to protect the **confidentiality** of the data against eavesdropping by any curious adversaries.

You may complete this assignment in **groups of 2-3 pax**. Indicate your partner's name in the google sheet provided in our course handout.

There are **three** parts of this assignment:

1. Authentication Protocol (**AP**)
2. Confidentiality Protocol 1 (**CP1**)
3. Confidentiality Protocol 2 (**CP2**)

These three parts form a strict Secure File Transfer protocol. You will be using socket programming (from the first half of the term) and cryptography knowledge (from the second half of the term) to complete this assignment.

{:.note-title}
> Secure FTP != HTTPS
>
> Note that you will be implementing Secure FTP as your own **whole new application layer protocol**. In **NO WAY** we are relying on HTTP/s.
>
> There seem to be some ridiculous misunderstanding from your seniors in the past years that this assignment requires knowledge on HTTP/s and/or DNS. It's totally two different protocol even though they are both application layer protocol. HTTPS is not equal to our SFTP, they are unrelated, as unrelated as 🍊 fruit is to 🌹 flower.

## System Requirements

The starter code provided to you is written in **C** and uses the **OpenSSL** library (`libssl`, `libcrypto`) for all cryptographic operations. You need:

- A C compiler (`gcc` or `clang`)
- OpenSSL development headers
- A **POSIX-compliant OS** (Linux or macOS)

Install the dependencies:

```bash
# Ubuntu / Debian
sudo apt-get install build-essential libssl-dev

# macOS (Homebrew) — the Makefile auto-detects the path
brew install openssl

# Fedora / RHEL
sudo dnf install gcc openssl-devel
```

## Starter Code

You should have joined the GitHub Classroom and obtained the starter code for this assignment there. The link can be found in the Course Calendar portion of your Course Handout.

### PA2 Files

Anything under `source/` is where you will work for this assignment. All files in the same level as `source/` are for **admin purposes**. Do not modify these.

```
.[PROJECT_DIR]
├── files
│   ├── cbc.bmp
│   ├── file.txt
│   ├── image.ppm
│   ├── jsim.jar
│   ├── player.psd
│   ├── squeak.wav
│   ├── vscodejsim.mp4
│   └── week9.html
├── Makefile
├── README.md
├── recv_files
├── recv_files_enc
├── send_files_enc
├── setup.sh
└── source
    ├── auth
    │   ├── cacsertificate.crt
    │   └── generate_keys.sh
    ├── ClientWithoutSecurity.c
    └── ServerWithoutSecurity.c
```

### Run `./setup.sh`

`[PROJ_ROOT_DIR]/recv_files`, `[PROJ_ROOT_DIR]/recv_files_enc`, and `[PROJ_ROOT_DIR]/send_files_enc` are all empty directories that are not added in `.git`. To create them, simply run `./setup.sh` in the **project root**.

You should see exactly the above file structure afterwards. `./setup.sh` is a bash script, in order to **execute** it you must `chmod` it to be executable first.

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa2/images/intro/2026-03-19-15-47-01.png"  class="center_full no-invert"/>


### Custom Crypto Library `common.c` 

All complex OpenSSL calls are **wrapped for you** in `common.h` / `common.c`. You call these wrapper functions and you do NOT need to write raw OpenSSL `EVP_*` code yourself.

Here is the mapping between the Python functions you may be familiar with and the C wrapper functions provided:

| Functionality       | C function (in `common.h`)                                                    |
| ------------------------- | ----------------------------------------------------------------------- |
| Generate random bytes     | `RAND_bytes(buf, len)`                                                  | 
| Send 8-byte int           | `send_int(sockfd, val)`                                                 | 
| Send raw bytes            | `send_all(sockfd, buf, len)`                                            | 
| Read exact bytes          | `read_bytes(sockfd, len)`, must `free()`                               | 
| Convert 8B buf to int      | `bytes_to_int(buf)`                                                     | 
| Parse cert from bytes     | `load_cert_bytes(data, len)`, must `X509_free()`                       | 
| Verify cert vs CA         | `verify_server_cert(cert, ca_path)`, returns 1 or 0                    | 
| Verify RSA-PSS signature  | `verify_message_pss(cert, sig, sig_len, msg, msg_len)`, 1/0            | 
| Sign with RSA-PSS         | `sign_message_pss(key, msg, len, &sig_len)`, must `free()`             | 
| Load private key          | `load_private_key(path)`, must `EVP_PKEY_free()`                       |
| Extract pub key from cert | `X509_get_pubkey(cert)`, must `EVP_PKEY_free()`                        | 
| RSA encrypt one block     | `rsa_encrypt_block(key, pt, pt_len, &ct_len, use_oaep)`, must `free()` |
| RSA decrypt one block     | `rsa_decrypt_block(key, ct, ct_len, &pt_len, use_oaep)`, must `free()` | 
| Generate session key      | `generate_session_key(key_buf)`                                         | 
| Symmetric encrypt         | `session_encrypt(key, pt, pt_len, &ct_len)`, must `free()`             |
| Symmetric decrypt         | `session_decrypt(key, ct, ct_len, &pt_len)`, must `free()`             | 

{:.note-title}
> Memory management in C
>
> Unlike Python, C does not have garbage collection. Every function that returns a `malloc()`'d buffer (marked "must `free()`" above) requires you to call `free()` on the returned pointer when you are done with it. Similarly, `X509*` objects must be freed with `X509_free()` and `EVP_PKEY*` with `EVP_PKEY_free()`. Failure to do so causes memory leaks. Read `common.h` carefully — each function's documentation specifies who must free what.

The documentation for `common.c` library can be found here. Refer to it when completing this assignment.

## Test the Starter Code

### Build

Run the following command in the project root to compile:

```bash
make
```

The Makefile automatically detects macOS vs Linux and locates OpenSSL headers. If the build fails, ensure `libssl-dev` (Linux) or `brew install openssl` (macOS) is installed.

### Using the same machine

The starter code provided to you implements a **simple**, non-secure file transfer protocol. We will explain in detail what the protocol is. For now, let's just ensure that everything runs normally.

Run each of these commands in two separate shell sessions:

```bash
./ServerWithoutSecurity
./ClientWithoutSecurity
```

You can type in the filename you want to send, e.g `files/image.ppm` from the Client's window, and the server will receive it and store it under `[PROJECT_DIR]/recv_files` directory.

You can repeat the above steps multiple times for each file you want to send to the server. If the client would like to close connection to the server, key in `-1`.

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa2/images/intro/2026-03-19-15-49-41.png"  class="center_full no-invert"/>

### Using different machines

You can also host the Server file in another computer:

```bash
./ServerWithoutSecurity [PORT] 0.0.0.0
```

You can use any high port you want, e.g: 12345.

The client computer can connect to it using the command:

```bash
./ClientWithoutSecurity [PORT] [SERVER-IP-ADDRESS]
```

Ensure you use the same port. The server's IP address (private) should be discoverable using the following if you use WiFi (`en0`), or `en1` if you use ethernet:

```bash
# Linux
ipconfig getifaddr en0

# macOS
ipconfig getifaddr en0
```

To get this to work, you most probably need to be on **the same subnet**. Just ensure that both machines are connected to the same WiFi and subnet. If you use SUTD WiFi, there's a chance that you are not in the same subnet. In that case, simply use your phone <span class="orange-bold">hotspot</span> and both machines shall connect to it.

# Important Requirements

{:.important-title}
> Do NOT Modify `common.h` / `common.c`
>
> You are **NOT** allowed to modify `common.h` or `common.c`. These files contain all the cryptographic wrappers and socket helpers you need. You should only `#include "common.h"` in your new source files and call the functions declared there.

{:.error-title}
> No other cryptographic libraries allowed
> 
> You are also not allowed to use any cryptographic library other than OpenSSL (which is what `common.c` already uses). You may use any standard C library headers (`stdio.h`, `stdlib.h`, `string.h`, etc.) as they are already included via `common.h`.
