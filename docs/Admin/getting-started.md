---
layout: default
title: Getting Started
permalink: /admin/getting-started
parent: Administration
nav_order: 11
---

* TOC
{:toc}

# Getting Started
{:.no_toc}

This course uses a POSIX-compliant OS throughout all labs and lecture examples. You need to have one set up before your first lab session in Week 1. We also assume working knowledge of C; if you are rusty, work through the preparatory materials below at your own pace during the first few weeks.

## Setting Up a POSIX-Compliant OS

POSIX-compliant means UNIX or UNIX-like: Linux distributions, macOS, or WSL on Windows. Anything mainstream that is not native Windows will work. We recommend Ubuntu 22 or 24 if you are a beginner. If you are feeling adventurous, ArchLinux is a good learning experience.

### macOS Users

macOS is already POSIX-compliant, so you can do most labs and run all lecture code as-is. The one exception is [**Lab TOCTOU Race Condition Attack**]({{ site.baseurl }}/labs/02-toctou), which requires a Linux environment. You have two options:

1. Pair with a teammate who has Linux for that lab.
2. Install UTM with Ubuntu 22. This works out of the box on Apple Silicon and takes about 5 GB. If you want the full CLI experience, the server version (no desktop environment) is sufficient. Download links are available on eDimension.

### Windows Users

Pick one of the following:

1. **WSL (recommended for beginners):** Windows Subsystem for Linux comes built into Windows. This is the easiest option if you have no prior experience with Linux. Recent WSL updates (late 2024 onwards) have good support for everything we do in this course.
2. **Virtual machine:** Install VirtualBox or VMware Fusion with Ubuntu 22/24. There are plenty of YouTube guides for this.
3. **Dual boot:** Install Ubuntu alongside Windows. More involved but gives the best native performance.
4. **Cloud:** Use AWS EC2 with Cloud9 or a similar cloud IDE. Free tier is available but requires a credit card. See [this in-house](https://hackmd.io/6U8U5dYrSU2Ny-jBatAKiw) EC2/Cloud9 setup guide if interested.

You need at least <span class="orange-bold">one</span> of these working before Week 1's lab session.

## C Language Preparation

The labs and many lecture examples use C. These refresher materials below are optional but will make your life significantly easier, especially if you plan to (and should!) do the labs and assignments on your own instead of asking AI to do it.

| Material | Topics Covered | When to Read |
|---|---|---|
| [Introduction to C Part 1](https://docs.google.com/document/d/1hcMLXwKqblB9UtalLnbPjIHmxrdAnZaK7Haadhql9jo/edit?usp=sharing), [recording](https://www.youtube.com/watch?v=TBX35MUoq_w) | Structs, pointers, malloc, other basics | Before Week 1 |
| [Introduction to C Part 2](https://docs.google.com/document/d/1KcY8mW6D9XM2l-I73KUXmYdaGgYILoRBdkFPcVJBjpM/edit?usp=sharing), [recording](https://youtu.be/7H5roj-6JRo) | Advanced memory allocation, static variables, function pointers, stack vs heap | Before Week 2 |
| [Introduction to C Part 3](https://docs.google.com/document/d/1OfXAu2Zaap_pDR4oLrByyVMv2YyUP0WonJTiVuvWsHs/edit), [recording](https://youtu.be/PB3d6VCd1Cc) | Shared memory, signal handling, fork, process status handling, process termination | Before Week 3 |

If you are comfortable with C from your own experience, you can skip these. If you find yourself struggling with pointer arithmetic or memory management during labs, come back and work through them.

## Textbooks

| Abbreviation | Title | Status |
|---|---|---|
| SGG | Silberschatz, Galvin, Gagne. *Operating Systems Concepts with Java*, 8th Ed. Wiley. | Required (Weeks 1 to 6) |
| KR | Kurose and Ross. *Computer Networking: A Top-Down Approach*, 7th Ed. Pearson. | Required (Weeks 8 to 12) |
| K&R | Kernighan and Ritchie. *The C Programming Language*, 2nd Ed. Prentice Hall. | Optional |