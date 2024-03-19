---
layout: default
title: Operating System 
permalink: /os/intro
has_children: true
nav_order: 3
---

# Operating System
{: .no_toc}

This section contains the syllabus and materials for the first part of 50.005. You can find detailed Weekly learning objectives below. 

{: .note-title}
> Textbook
> 
> Silberschatz, Galvin, Gagne. **Operating Systems Concepts with Java**, 8th Ed. Wiley. ISBN 978-0-470-50949-4. (required) - Abbreviated as “SGG” in this document

## Week 1: SGG Ch. 1.1 - 1.6, 1.8.3, 1.13.3
* Define what an operating system is
* Describe basic roles of an operating system
* Know the difference between User vs Kernel mode, and how to switch between each mode using system calls and interrupt
* Describe computer system organisation (basic architecture)
* Describe how I/O interrupts occur and how the Kernel handles the interrupt
* Describe and explain the basic structure of storage / memory device hierarchy and derive computations for cache performance
* Explain the meaning of timesharing, context switch, scheduling, and multiprogramming - all of which is part of OS process management 
* Explain the basic difference between process and program
 
## Week 2: SGG Ch. 2.1 - 2.6, 2.7.1 - 2.7.3, 2.9.2
* Explain a variety of OS services and their applications
* Describe ways on how to access OS services
* Explain the difference between making direct system calls vs through API
* Describe the possible ways to pass parameters using system calls
* Describe and explain different types of system calls and their purposes
* Explain the difference between user programs and system programs
* Describe and explain different types of system programs and their purposes
* Explain the basic principles behind OS design: simple, layered, microkernel using  examples: macOS, JX, MSDos, and UNIX.
 
## Week 3: SGG Ch. 3.1-3.3, 3.4.1, 3.6.1, 4.1-4.2, 4.3.1, 4.4, 4.5.1 - 4.5.2
* Explain the concept of processes, and the difference between codes, programs and processes
* Explain the difference between concurrency and parallelism
* Describe and explain various process lifecycle, scheduling states and their transitions
* Know the process management terms: process control block, and context switch. 
* Describe different types of process queues and I/O queues
* Know how to create a new process, e.g: using fork() in C or ProcessBuilder in Java
* Explain how interprocess communication (IPC) works
* Explain the concept of threads and its difference with process
* Describe two types of threads: user and kernels and their models and their applications
* Compute amount of speedup in multicore programming
 
## Week 4: SGG Ch. 6.1-6.4, 6.5.1, 6.5.2, 6.6.1, 6.8.1 - 6.8.2, 6.8.4
* Explain issues of multiprogramming and concurrency: the race condition
* Describe and explain the concept of critical section
* Discuss how Peterson’s solution solve the critical section problem and prove its correctness
* Explain how synchronisation hardware works in general
* Describe and explain the concepts of mutex and condition synchronization
* Explain other critical section solutions that do not require busy waiting: semaphores
* Explain how semaphores can be used as a mutex and condition synchronization
* Explain how Java implements synchronization using synchronized methods and named condition variables
 
## Week 5: SGG Ch. 7.1-7.4, 7.5.1, 7.5.3, 7.6.2, 7.7
* Explain the problem of deadlock
* Describe necessary conditions for deadlock
* Draw and explain resource allocation graphs
* Discuss ways to handle deadlocks through prevention, detection, and recovery
* Describe and explain deadlock avoidance algorithm: Banker’s algorithm
* Describe and explain deadlock detection algorithm
* Explain the difference between deadlock avoidance algorithm and deadlock detection algorithm
 
## Week 6: SGG Ch.10.1, 10.2, 10.3.2 - 10.3.7, 10.6, 11.2.1
* Explain what is a file system and the file system namespace
* Explain file format, permission, and type and how they are used in the file system
* Describe and explain possible operations on file
* Describe the concepts and the difference between file data and metadata (Attributes)
* Learn the concepts of file descriptors and file system data structure, i.e: how the OS stores files in the memory
* Explain UNIX file system data structures and mapping between table entries
* Describe the concepts of various directory structures and how directory works: e.g tree structures (acyclic and cyclic)
* Discuss the difference between symbolic and hard links and its purpose 
