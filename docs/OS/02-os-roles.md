---
layout: default
permalink: /os/2-os-roles
title: Roles of OS Kernel
description: Various Roles of OS Kernel
parent: Operating System
nav_order:  2
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

# Roles of Operating System Kernel {#roles-of-an-operating-system-kernel}
{: .no_toc}

{:.highlight-title}
> Detailed Learning Objectives
>
> 1. **Summarize the Role of Operating Systems**
>   - Explain the **purposes** of an operating system including **resource allocation**, program **execution** control, and **security** enforcement.
> 2. **Outline Interrupt-Driven I/O Operations**
>   - Define the concept of interrupt-driven I/O and differentiate between **hardware** and **software** interrupts.
>   - Explain the functions and benefits of **vectored** and **polled** interrupt systems.
>   - Describe the process of handling hardware interrupts, including the role of interrupt request lines and interrupt handlers.
>   - Compare vectored and polled interrupt scenarios and their suitability for different system complexities.
> 3. **Examine System Calls and Software Interrupts**
>   - List out the role of **system calls** in software interrupts and their impact on process execution.
>   - Discuss how software interrupts (**traps**) facilitate interaction between user processes and the kernel.
> 4. **Explain Process Management**
>   - Explore how the kernel manages processes to support **multiprogramming** and **timesharing**.
>   - Define the difference between a process and a program.
> 5. **Describe Kernel Reentrancy and Preemptiveness**
>   - Compare reentrant and preemptive kernel operations and their implications for process management.
> 6. **Evaluate Memory and Process Management Techniques**
>   - Discuss the implementation of virtual memory and the configuration of memory management units (MMU).
>   - Analyze cache performance and the role of the kernel in memory and process management.
> 7. **Assess Security and Protection Mechanisms**
>   - Explain the mechanisms operating systems use to provide security and protection, including user identification and access control.
> 8. **Describe computer system organisation (basic architecture)**
>   - Evaluate the differences between symmetric, asymmetric, and clustered system 
>  
> These objectives aim to guide your exploration of operating system concepts, focusing on the interactions between the OS kernel and system hardware, the management of processes and memory, and the crucial role of security within computer systems.

There are several purposes of an operating system: as a <span style="color:#f77729;"><b>resource</b></span> allocator, <span style="color:#f77729;"><b>controls</b></span> program execution, and guarantees <span style="color:#f77729;"><b>security</b></span> in the computer system.

The kernel <span style="color:#f77729;"><b>controls</b></span> and <span style="color:#f77729;"><b>coordinates</b></span> the use of hardware and I/O devices among various user applications. Examples of I/O devices include mouse, keyboard, graphics card, disk, network cards, among many others.

# Resource Allocator and Coordinator: Interrupt-Driven I/O Operations

Interrupt-Driven I/O operations allow the CPU to efficiently handle interrupts without having to waste resources (waiting for asynchronous IO requests). This notes explains how interrupt-driven I/O operations work in a nutshell. There are <span style="color:#f7007f;"><b>two kinds of interrupts</b></span>:

1. <span style="color:#f7007f;"><b>Hardware Interrupt</b></span>: input from external devices activates the interrupt-request-line, thus pausing the current execution of user programs.
2. <span style="color:#f7007f;"><b>Software Interrupt</b></span>: a software generated interrupt that is invoked from the instruction itself because the current execution of user program needs to access Kernel services.

## Hardware Interrupt

<img src="{{ site.baseurl }}//assets/images/week1-3_resource/2023-05-16-10-30-51.png"  class="center_fifty no-invert"/>

The CPU has an <span style="color:#f77729;"><b>interrupt-request line</b></span> that is sensed by its control unit before each instruction execution.

When there's new **input**: from <span style="color:#f77729;"><b>external</b></span> events such as keyboard press, mouse click, mouse movement, incoming fax, or **completion of previous I/O requests** made by the drivers on <span style="color:#f77729;"><b>behalf</b></span> of user processes, the device controllers will invoke an **interrupt** request by setting the **bit** on this <span style="color:#f77729;"><b>interrupt-request line</b></span>.

Remember that this is a <span style="color:#f7007f;"><b>hardware interrupt</b></span>: an interrupt that is caused by setting the interrupt-request line.
{:.error}

This forces the CPU to **transfer** control to the **interrupt handler**. This switch the currently running user-program to enter the <span style="color:#f77729;"><b>kernel mode</b></span>. The interrupt handler will do the following routine:

1. **<span style="color:#f7007f;"><b>Save</b></span> the register states** first (the interrupted program instruction) into the process table
2. And then transferring control to the appropriate interrupt service routine — depending on the device controller that made the request.

### Vectored Interrupt System

Interrupt-driven system may use a <span style="color:#f77729;"><b>vectored interrupt system</b></span>: the interrupt signal that **INCLUDES** the identity of the device sending the interrupt signal, hence allowing the kernel to know exactly which interrupt service routine to execute[^2]. This is more <span style="color:#f7007f;"><b>complex</b></span> to implement, but more <span style="color:#f77729;"><b>useful</b></span> when there is a large number of different interrupt sources that throws interrupts frequently.

This interrupt mechanism accepts an **address**, which is usually one of a small set of numbers for an offset into a table called the **interrupt vector.** This table holds the addresses of routines prepared to process specific interrupts.
{:.info}

With vectored interrupts, each interrupt source is associated with a unique vector address, which allows the processor to directly jump to the appropriate interrupt handler. This eliminates the need for the processor to search or iterate through a list of interrupt requests to identify the source, resulting in faster and more efficient interrupt handling. Therefore, when there are frequent interrupts from various sources, vectored interrupts can help improve the overall system performance.

### Polled Interrupt System

An alternative is to use a <span style="color:#f77729;"><b>polled interrupt system</b></span>:

- The interrupted program enters a general interrupt polling routine <span style="color:#f7007f;"><b>protocol</b></span> (a single entry point), where CPU <span style="color:#f7007f;"><b>scans</b></span> (polls) devices to determine which device made a service request.
- Unlike vectored interrupt, there’s no such _interrupt_ signal that includes the identity of the device sending the interrupt signal.
- The kernel must send a signal out to each controller to **determine** which device made a service request. It may determine the interrupt source by reading the interrupt status registers or other means.

This is simpler to implement, but more time-wasting if there’s frequent, unpredictable I/O requests only from one or some particular device in an array of devices. The CPU need to spend overhead to poll many devices. Does Linux implement a Polled interrupt or Vectored interrupt system? See [here](https://linux-kernel-labs.github.io/refs/heads/master/lectures/interrupts.html) for clues.
{:.info}

This system may be more appropriate for systems with sparse interrupts, where there are only a few interrupt sources or the interrupts occur infrequently. In non-vectored interrupt systems, . While this approach requires more processing overhead for identifying the interrupt source, it may be sufficient and more straightforward for systems with a small number of interrupts.

### Vectored Interrupt vs Polled Interrupt Scenarios

Polled interrupts are **generally** suitable in simple systems with a **limited** number of devices and **low** interrupt rates or **predictable** and moderate interrupt rates. For example, in embedded systems with a single or a few devices generating interrupts infrequently, such as basic sensor input or user interactions, polled interrupts can be sufficient.

Vectored interrupts are commonly used in more **complex** systems with **multiple** devices generating interrupts frequently. They are suitable when efficient handling of multiple interrupts and reduced latency are crucial, such as in high-performance systems, real-time operating systems, or systems with time-sensitive tasks.

### Multiple Interrupts

If there’s more than one I/O interrupt requests from multiple devices, the Kernel may decide which interrupt requests to service first. When the service is done, the Kernel scheduler may **choose** to resume the user program that was interrupted.

### Raw Device Polling

It takes time to perform a **<span style="color:#f7007f;"><b>context switch</b></span>** to handle interrupt requests. In some specifically dedicated servers, raw polling may be faster. The CPU <span style="color:#f7007f;"><b>periodically</b></span> checks each device for requests. If there's no request, then it will return to resume the user programs. Obviously, this method is <span style="color:#f77729;"><b>not suitable</b></span> in the case of sporadic I/O usage, or in general-purpose CPUs since at least a fixed bulk of the CPU load will always be dedicated to perform routine polling. For instance, there's no need to poll for anything periodically when a user is watching a video.

Modern computer systems are **<span style="color:#f7007f;"><b>interrupt-driven</b></span>**, meaning that it <span style="color:#f7007f;"><b>does not wait</b></span> until the input from the requested external device is ready. This way, the computer may <span style="color:#f77729;"><b>efficiently</b></span> fetch I/O data **only when the devices are `ready`{:.error},** and will not waste CPU cycles to "poll" whether there are any I/O requests or not.

## Hardware Interrupt Timeline

The figure below summarises the **interrupt-driven** procedure of **asynchronous I/O handling** during **<span style="color:#f7007f;"><b>hardware interrupt</b></span>**:

<img src="{{ site.baseurl }}/assets/images/week1/9.png"  class="center_fifty"/>

Notes:

1. **I/O Devices**: External I/O Device, e.g: printer, mouse, keyboard. Runs asynchronously, capable of being "smart" and run its own instructions.
2. **Device Controller**: Attached to motherboard. Runs asynchronously, may run simple instructions. Has its own buffers, registers, and simple instruction interpreter. Usually transfer data to and fro the I/O device via standard protocols such as USB A/C, HDMI, DP, etc.
3. **Disk Controller**: Same as device controller, just that it's specific to control Disks

### Receiving asynchronous Input

From the figure above, you may assume that at first the CPU is busy executing user process instructions. At the same time, the device is _idling_.

Upon the presence of external events, e.g: mouse `CLICK()`, the device <strong>records</strong> the input. This triggers <span style="color:#f7007f;"><b>step 1</b></span>: The I/O device sends data to the device controller, and <strong>transfers </strong>the data from the<strong> device buffer </strong>to the<strong> local buffer of the device controller.</strong>

When I/O transfer is complete, this triggers <span style="color:#f7007f;"><b>step 2</b></span>: the device controller makes an <strong>interrupt request</strong> to signal that a <em>transfer is done (and data needs to be fetched)</em>. The simplest way for the device controller to raise an interrupt is by asserting a signal on the interrupt request line (this is why we define I/O interrupts as hardware generated)

The interrupt request line is sensed by the CPU at the beginning of each instruction execution, and when there’s an interrupt, the execution of the <strong>current user program is interrupted.</strong> Its states are <span style="color:#f7007f;"><b>saved</b></span>[^3] by the entry-point of the interrupt handler. Then, this handler determines the <span style="color:#f7007f;"><b>source</b></span> of the interrupt (be it via Vectored or Polling interrupt) and performs the necessary processing. This triggers <span style="color:#f7007f;"><b>step 3</b></span>: the CPU executes the proper I/O service routine to transfer the data <span style="color:#f77729;"><b>from</b></span> the local device controller buffer <span style="color:#f77729;"><b>to</b></span> the physical memory.

After the I/O request is serviced, the handler:

1. <span style="color:#f77729;"><b>Clears</b></span> the interrupt request line,
2. <span style="color:#f77729;"><b>Performs</b></span> a state restore, and executes a **_return from interrupt_** instruction (or `JMP(XP)` as you know it from 50.002).

### Consuming the Input

Two things may happen from here after we have stored the new input to the RAM:

1. If there’s no application that’s currently waiting for this input, then it might be temporarily stored somewhere in kernel space first.
2. If there is **any application** that is waiting (blocked, like Python's `input()`) for this input (e.g: mouse click), that process will be labelled as <span style="color:#f77729;"><b>ready</b></span>. For example, if the application is blocked upon waiting for this new input, then the system call returns. **We will learn more about this in Week 3.**

One thing should be crystal clear: in an interrupt-driven system, upon the presence of new input, a **hardware interrupt** occurs, which invokes the interrupt handler and then the interrupt service routine to service it. <span style="color:#f7007f;"><b>It does not matter whether any process is currently waiting for it or not</b></span>. If there’s a process that’s waiting for it, then it will be scheduled to resume execution since its system call will **return** (if the I/O request is blocking).
{:.info}

# Software Interrupt (Trap) via System Call

Firstly, this image says it all.

<img src="{{ site.baseurl }}/assets/images/week1/10.png"  class="center_fifty no-invert"/>

Sometimes user processes are <span style="color:#f77729;"><b>blocked</b></span> from execution because it requires inputs from IO devices, and it may **not be scheduled** until the presence of the required input arrives. For example, this is what happens if you wait for user input in Python:

```python
inp = input("Enter your name:")
print(inp)
```

Running it will just cause your program to be stuck in the console, _why_:

```console
bash-3.2$ python3 test.py
Enter your name:
```

User processes may <span style="color:#f7007f;"><b>trap</b></span> themselves e.g: by executing an <span style="color:#f7007f;"><b>illegal</b></span> operation (`ILLOP`) implemented as a system call, along with some value in designated registers to <span style="color:#f77729;"><b>indicate</b></span> which system call this program tries to make.

> The details on _which register_, or _what value should be put_ into these registers depends on the architecture of your CPU. <span style="color:#f77729;"><b>This detail is not part of our syllabus</b></span>.

Traps are **software generated interrupts**, that is some special instructions that will <span style="color:#f77729;"><b>transfer</b></span> the mode of the program to the Kernel Mode.
{:.info}

The CPU is forced to go to a special handler that does a state save and then execute (may not be immediate!) on the proper interrupt service routine to handle the <span style="color:#f7007f;"><b>request</b></span> (e.g: fetch user input in the python example above) in kernel mode. Software interrupts generally have <span style="color:#f77729;"><b>low priority</b></span>, as they are not as urgent as devices with limited buffering space.

<img src="{{ site.baseurl }}/assets/images/week1/11.png"  class="center_seventy"/>

During the time between system call request until system call return, the program execution is <span style="color:#f7007f;"><b>paused</b></span>. Examples of system calls are: `chmod(), chdir(), print()`. More Linux system calls can be found [here](http://man7.org/linux/man-pages/man2/syscalls.2.html).

### Combining Hardware Interrupt and Trap

Consider another scenario where you want to open a **very large file** from disk. It takes some time to <span style="color:#f7007f;"><b>load</b></span> (simply transfer your data from disk to the disk controller), and your CPU can proceed to do other tasks in the meantime. Here's a simplified timeline:

<img src="{{ site.baseurl }}/assets/images/week1/12.png"  class="center_full"/>

Imagine that at first, the CPU is busy executing process instructions in user mode. At the same time, the device is idling.

1. The process requests for Kernel services (e.g: load data asynchronously) by making a <span style="color:#f7007f;"><b>system call</b></span>.
   - The **context** of the process are saved by the trap handler
   - Then, the appropriate system call service routine is called. Here, they may require to load appropriate **device drivers** so that the CPU may communicate with the device controller.
2. The **device controller** then makes the instructed I/O request to the device itself on behalf of the CPU, **e.g: a disk,** as instructed.
   - Meanwhile, the service handler returns and may resume the calling process as illustrated.

The I/O device then proceeds on responding to the request and **transfers** the data from the **device** to the **local buffer** of the device controller.
{:.info}

When I/O transfer is <span style="color:#f7007f;"><b>complete</b></span>, the device controller makes an **<span style="color:#f7007f;"><b>hardware interrupt</b></span> request** to signal that the transfer is done (and data needs to be fetched). The CPU may respond to it by saving the states of the currently interrupted process, handle the interrupt, and resume the execution of the interrupted process.

**Note**:

1. SVC <span style="color:#f77729;"><b>delay</b></span> and IRQ <span style="color:#f77729;"><b>delay</b></span>: time elapsed between when the request is invoked until when the request is first executed by the CPU.
2. Before the user program is resumed, its state must be <span style="color:#f77729;"><b>restored</b></span>. Saving of state during the switch between User to Kernel mode is implied although it is not drawn.

## Exceptions {#exceptions}

Exceptions are <span style="color:#f77729;"><b>software interrupts</b></span> that occur due to <span style="color:#f7007f;"><b>errors</b></span> in the instruction: such as division by zero, invalid memory accesses, or attempts to access kernel space illegally. This means that the CPU's hardware may be designed such that it checks for the presence of these <span style="color:#f7007f;"><b>serious</b></span> errors, and immediately invokes the appropriate handler via a pre-built <span style="color:#f7007f;"><b>event-vector table</b></span>.

## Interrupt Vector Table (IVT)

{:.info-title}
> Definition
>
> The IVT is a table of pointers to interrupt service routines (ISRs) used to handle interrupts in computing systems.


**Purpose**: Allows efficient handling of hardware and software interrupts by directing the processor to the appropriate ISR.<br>
**Structure**: Comprises entries, each corresponding to a specific interrupt vector; each entry points to an ISR.<br>
**Location**: Often found at a fixed memory address, particularly in systems like the Intel x86 in real mode.<br>
**Function**: On an interrupt, the system uses the interrupt number to index into the IVT, fetch the ISR address, and execute the routine to address the event.

Generally speaking, CPU must be configured to receive interrupts (IRQs) and invoke correct interrupt handler using the Interrupt Vector Table (IVT). Operating system kernel must provide Interrupt Service Routines (ISRs) to handle interrupts.

Sometimes you might encounter the term IDT instead of IVT. The interrupt descriptor table (IDT) is a _data structure_ used by the x86 architecture to implement an interrupt vector table.
{:.info}

Head to the [appendix](#ivt-in-various-architectures) to learn more about actual implementation of IVT in different architecture.


# Reentrant vs Preemptive Kernel
## Reentrancy

A reentrant kernel is the one which allows <span style="color:#f7007f;"><b>multiple</b></span> processes to be executing in the kernel mode at any given point of time (concurrent), hopefully without causing any consistency problems among the kernel data structures. If the kernel is non re-entrant, the kernel is not designed to handle multiple overlapping system calls. A process could still be suspended in kernel mode, but that would mean that it <span style="color:#f77729;"><b>blocks</b></span> kernel mode execution on <span style="color:#f77729;"><b>all other processes</b></span>. No other processes can make system calls (will be put to wait) until the suspended process in kernel mode resumes and returns from the system call. 

{:.note}
Head to [appendix](#reentrant-kernel-behavior) if you'd like to view a simple example of a reentrant Kernel that supports concurrency. 

For example, consider Process 1 that is <span style="color:#f7007f;"><b>voluntarily</b></span> voluntarily suspended when it is in the middle of handling its `read` system call because the it has to wait for some data from disk to become available. It is <span style="color:#f77729;"><b>suspended in Kernel Mode</b></span> by `yielding` itself. Another Process 2 is now scheduled and wishes to make `print` system call. 

- In a reentrant kernel: Process 2 is currently executed; able to execute its `print` system call.
- In a non-reentrant kernel: Process 2, although currently executed must **wait** for Process 1 to exit from the Kernel Mode if Process 2 wishes to execute its `print` system call.


## Preemption

A pre-emptive Kernel <span style="color:#f77729;"><b>allows the scheduler</b></span> to <span style="color:#f7007f;"><b>interrupt</b></span> processes in Kernel mode to execute the highest priority task that are ready to run, thus enabling kernel functions to be <span style="color:#f7007f;"><b>interrupted</b></span> just like regular user functions. The CPU will be assigned to perform other tasks, from which it later returns to finish its kernel tasks. In other words, the scheduler is permitted to <span style="color:#f7007f;"><b>forcibly</b></span> perform a context switch.

Likewise, in a non-preemptive kernel the scheduler is not capable of rescheduling a task while its CPU is executing in the kernel mode.
{:.info}

Using the same example of Process 1 and 2 above, assume a scenario whereby there's a periodic scheduler interrupt to check _which_ Process may resume next. Assume that Process 1 is in the middle of handling its `read` system call when the timer interrupts.

- In a non-preemptive Kernel: If Process 1 does not voluntarily `yield` while it is still in the middle of its system call, then the scheduler will not be able to forcibly interrupt Process 1.
- In a preemptive Kernel: When Process 2 is ready, and it is time to schedule Process 2 or if Process 2 has a higher priority than Process 1, then the scheduler may **forcibly** suspend Process 1.

A kernel can be reentrant but not preemptive: That is if each process voluntarily `yield` after some time while in the Kernel Mode, thus allowing other processes to progress and enter Kernel Mode (make system calls) as well. However, a kernel <span style="color:#f7007f;"><b>should not</b></span> be preemptive and not reentrant (it doesn't make sense!). 

A **preemptive** kernel allows processes to be interrupted at any time, requiring the kernel to handle multiple **simultaneous** executions of its code safely. A non-reentrant kernel <span class="orange-bold">cannot</span> handle concurrent execution of kernel code, leading to data corruption and instability if preempted. Thus, a kernel must be reentrant to function correctly in a supposed *preemptive* environment.  

We can of course work around it by disallowing any other processes to make system calls when another process is currently suspended in Kernel Mode (eg: forcibly switched by scheduler in the middle of handling its system call). This setup avoids concurrent executions of kernel code but lead to **inefficiencies** and delayed response times: rendering the "preemptive kernel feature" useless. 

**Fun fact:** Linux Kernel is reentrant and preemptive. 
{:.info}

In simpler Operating Systems, incoming hardware interrupts are typically <span style="color:#f7007f;"><b>disabled</b></span> while another interrupt (of same or higher priority) is being processed to prevent a lost interrupt i.e: when user states are currently being saved before the actual interrupt service routine began or various **<span style="color:#f7007f;"><b>reentrancy problems</b></span>**[^4]. Disabling hardware interrupts during critical moments like context switching typically indicates a non-preemptive kernel. 



# Memory Management
The Kernel has to <span style="color:#f77729;"><b>manage</b></span> all memory devices in the system (disk, physical memory, cache) so that they can be shared among many other running user programs. The hierarchical storage structure requires a concrete form of memory management since the same data may appear in different levels of storage system.

## Virtual Memory Implementation

The kernel implements the Virtual Memory. It has to:

1. <span style="color:#f77729;"><b>Support demand paging</b></span> protocol
2. <span style="color:#f77729;"><b>Keep track</b></span> of which parts of memory are currently being used and by whom
3. <span style="color:#f77729;"><b>Decide</b></span> which processes and data to move into and out of memory
4. <span style="color:#f77729;"><b>Mapping</b></span> files into process address space
5. <span style="color:#f77729;"><b>Allocate</b></span> and <span style="color:#f77729;"><b>deallocate</b></span> memory space as needed
   - If RAM is full, <span style="color:#f77729;"><b>migrate</b></span> some contents (e.g: least recently used) onto the <span style="color:#f77729;"><b>swap space</b></span> on the disk
6. <span style="color:#f77729;"><b>Manage</b></span> the <span style="color:#f7007f;"><b>pagetable</b></span> and any operations associated with it.

CPU caches are managed entirely by <span style="color:#f7007f;"><b>hardware</b></span> (cache replacement policy, determining cache `HIT` or `MISS`, etc). Depending on the cache hardware, the kernel may do some initial setup (caching policy to use, etc).
{:.info}

## Configuring the MMU

The MMU (Memory Management Unit) is a computer <span style="color:#f7007f;"><b>hardware</b></span> component that primarily handles translation of virtual memory address to physical memory address. It relies on data on the system's RAM to operate: e.g utilise the <span style="color:#f77729;"><b>pagetable</b></span>. The Kernel sets up the page table and determine rules for the address mapping.

Recall from 50.002 that the CPU always operates on virtual addresses (commonly <span style="color:#f77729;"><b>linear</b></span>, `PC+4` unless branch or JMP), but they're translated to physical addresses by the MMU. The Kernel is aware of the translations and it is solely responsible to <span style="color:#f7007f;"><b>program</b></span> (configure) the MMU to perform them.
{:.info}

## Cache Performance

<span style="color:#f7007f;"><b>Caching</b></span> is an important principle of computer systems. We perform the <span style="color:#f77729;"><b>caching algorithm</b></span> each time the CPU needs to execute a new piece of instruction or fetch new data from the main memory. Remember that cache is a <span style="color:#f7007f;"><b>hardware</b></span>, built in as part of the CPU itself in modern computers as shown in the figure below:

<img src="{{ site.baseurl }}/assets/images/week1/14.png"  class="center_fifty"/>

Note that “DMA” means direct memory access from the device controller onto the RAM[^5] (_screenshot taken from SGG book_).

Data transfer from disk to memory and vice versa is usually controlled by the **kernel** (requires context switching), while data transfer from CPU registers to CPU cache[^6] is usually a <span style="color:#f7007f;"><b>hardware function</b></span> without intervention from the kernel. There is too much <span style="color:#f7007f;"><b>overhead</b></span> if the kernel is also tasked to perform the latter.

Recall that the Kernel is responsible to program and set up the cache and MMU hardware, and manage the entire virtual memory. Kernel memory management routines are <span style="color:#f77729;"><b>triggered</b></span> when processes running in user mode encounter <span style="color:#f77729;"><b>page-fault</b></span> related interrupts. Careful <span style="color:#f7007f;"><b>selection</b></span> of the page size and of a replacement policy can result in a greatly increased performance.

Given a cache <span style="color:#f77729;"><b>hit</b></span> ratio $\alpha$, cache <span style="color:#f77729;"><b>miss</b></span> access time $\epsilon$, and cache <span style="color:#f77729;"><b>hit</b></span> access time $\tau$, we can compute the cache <span style="color:#f7007f;"><b>effective access time as</b></span>:

$$\alpha \tau + (1-\alpha) \times \epsilon$$

### Example

Suppose we have a hit rate of $\alpha = 0.5$, cache access time of 20ms, and RAM access time of 100 ms. What is the effective access time of this system?

In this case, $\epsilon = 20 + 100 = 120ms$ (cache _miss_ time). Cache miss time includes the cache access time because **we would have spent** 20ms to check the cache in the first place to know that it's a cache miss. Therefore, the effective access time is **70ms**.

# Process Management {#process-management-to-support-multiprogramming-and-time-sharing-feature}

The Kernel is also responsible for managing all processes in the system and support <span style="color:#f7007f;"><b>multiprogramming</b></span> and <span style="color:#f7007f;"><b>timesharing</b></span> feature.

Multiprogramming and time-sharing are both concepts in operating systems that aim to improve the efficiency and responsiveness of computer systems by managing how processes are executed. However, they differ in their goals, implementation, and user experience.

## Multiprogramming

**Goal**: Increase CPU utilization and system throughput.

Multiprogramming involves running multiple programs (or processes) on a single processor system, a concept that is needed for <span style="color:#f77729;"><b>efficiency</b></span>. The operating system keeps several jobs in memory simultaneously. When one job waits for I/O operations to complete, the CPU can execute another job. This approach maximizes CPU usage by reducing idle time.

A kernel that supports multiprogramming increases <span style="color:#f77729;"><b>CPU utilization</b></span> by organizing jobs (code and data) so that the CPU always has one to execute. When one job waits for I/O operations to complete, the CPU can execute another job instead of *idling*.
{:.info}



**Simplified implementation**:
- The OS maintains a pool of jobs in the job queue.
- Jobs are selected and loaded into memory.
- The CPU switches between jobs when they are waiting for I/O, ensuring that it is always busy executing some job.
- Jobs are executed until they are complete or require I/O operations, at which point another job is scheduled.
- For example in diagram below, CPU will continue to run Job 3 **as long as** Job 3 does not `wait` for anything else. 
  - When Job 3 waits for some I/O operation in the future, CPU will switch to READY jobs, e.g: either Job 1 or Job 2 (assuming Job 4 is still waiting then)

<img src="{{ site.baseurl }}/docs/OS/images/cse2024-Multiprogramming.drawio.png"  class="center_fourty"/>

### Practical Consideration
Since the clock cycles of a general purpose CPU is very fast (in Ghz), we don't actually need 100% CPU power in most case. It is often _too fast_ to be dedicated for just one program for the entire 100% of the time. Hence, if multiprogramming is not supported and each process has a fixed quantum (time allocated to execute), then the CPU might spend most of its time <span style="color:#f7007f;"><b>idling</b></span>.

**The kernel must organise jobs (code and data) <span style="color:#f77729;"><b>efficiently</b></span> so CPU always has one to execute.** A <span style="color:#f77729;"><b>subset</b></span> of total jobs in the system is kept in memory and swap space of the disk.

Remember: **Virtual memory** allows execution of processes not completely in memory. One job is selected per CPU and run by the scheduler.
{: .info}

When a particular job has to wait (for I/O for example), **context switch** is performed.

- For instance, Process A asked for user `input()`, enters Kernel Mode via supervisor call
- If there's no input, Process A is <span style="color:#f77729;"><b>suspended</b></span> and <span style="color:#f77729;"><b>context switch</b></span> is performed (instead of returning back to Process A)
- If we return to Process A, since Process A cannot progress without the given input, it will invoke another `input()` request again -- again and again until the input is present. This will waste so much resources

{:.note}
Context switch is the routine of <span style="color:#f77729;"><b>saving</b></span> the state of a process, so that it can be <span style="color:#f77729;"><b>restored</b></span> and <span style="color:#f77729;"><b>resumed</b></span> at a later point in time. For more details on what this `state` comprised of, see Week 2 notes.

**User Experience:** Users might experience delays as the system prioritizes CPU utilization over interactive performance. The system is not designed to provide immediate feedback.

## Timesharing

**Goal**: Provide interactive user experience and fast response times.

Naturally, multiprogramming allows for <span style="color:#f7007f;"><b>timesharing</b></span>. Time-sharing **extends** the concept of multiprogramming by allowing multiple users to interact with the system simultaneously. The CPU time is divided into small time slices (or quanta), and each user program is given a slice of time. The OS rapidly switches between user programs, giving the illusion that each user has their own dedicated machine.

{:.info}
Timesharing: context switch that’s performed so <span style="color:#f77729;"><b>rapidly</b></span> that users still see the system as interactive and seemingly capable to run multiple processes despite having limited number of CPUs.


**Simplified implementation**:
- The OS maintains a list of active user processes.
- The CPU scheduler allocates a fixed time slice to each process in a round-robin fashion.
- If a process's time slice expires, the CPU switches to the next process in the queue.
- Processes that need I/O or user input can be interrupted and put into a waiting state, allowing other processes to use the CPU.
- For example in the diagram below, the CPU will run each job for a fixed quanta `t` and switch to another (ready) job once that time slice ends:

<img src="{{ site.baseurl }}/docs/OS/images/cse2024-Timesharing.drawio.png"  class="center_full"/>

**User Experience:** Users experience a responsive system as the OS ensures that each user program gets regular CPU time. Interactive tasks, such as typing or browsing, feel immediate because the OS switches between tasks quickly.



## Further Consideration

Multiprogramming alone <span style="color:#f7007f;"><b>does not</b></span> necessarily provide for user interaction with the computer system. For instance, even though the CPU has <span style="color:#f77729;"><b>something</b></span> to do, it doesn't mean that it _ensures_ that there's always interaction with the user, e.g: a CPU can run background tasks at all times (updating system time, monitoring network traffic, etc) and not run currently opened user programs like the Web Browser and Telegram.

Timesharing is the <span style="color:#f77729;"><b>logical extension of multiprogramming</b></span>. It results in an <span style="color:#f77729;"><b>interactive</b></span> computer system, which provides <span style="color:#f77729;"><b>direct</b></span> communication between the user and the system.
{:.info}

## Key Differences

| Feature                  | Multiprogramming                                | Time-Sharing                                      |
|--------------------------|-------------------------------------------------|---------------------------------------------------|
| **Primary Goal**         | Maximize CPU utilization and throughput         | Provide responsive, interactive user experience    |
| **User Interaction**     | Minimal, often non-interactive (batch processing)| High, interactive user experience                 |
| **CPU Scheduling**       | Job-based, switches when I/O is needed          | Time-based, fixed time slices (quanta)            |
| **Response Time**        | Can be longer, not optimized for interactivity  | Short, optimized for user interaction             |
| **Implementation**       | Multiple jobs loaded in memory, CPU switches on I/O | Multiple user processes, rapid context switching   |
| **Examples**             | Early mainframe systems, batch processing       | Modern operating systems, interactive applications|


While both multiprogramming and time-sharing aim to improve the efficiency of CPU utilization, they serve different purposes. Multiprogramming focuses on maximizing CPU usage by running multiple jobs simultaneously, often in a batch processing environment. Time-sharing, on the other hand, aims to provide a responsive, interactive experience for multiple users by rapidly switching between processes, giving the illusion of concurrent execution.

## Process vs Program {#process-vs-program}

<span style="color:#f7007f;"><b>A process is a program in execution</b></span>. It is a unit of work within the system, and a process has a <span style="color:#f77729;"><b>context</b></span> (regs values, stack, heap, etc) while a program does not.

A program is a _passive entity_, just lines of instructions while a process is an _active entity_, with its state changing over time as the instructions are executed.
{:.info}

A program resides on _disk_ and isn’t currently used. It <span style="color:#f7007f;"><b>does not</b></span> take up <span style="color:#f77729;"><b>resources</b></span>: CPU cycles, RAM, I/O, etc while a process need these resources to run.

- When a process is <span style="color:#f77729;"><b>created</b></span>, the kernel allocate these resources so that it can begin to execute.
- When it <span style="color:#f77729;"><b>terminates</b></span>, the resources are freed by the kernel — such as RAM space is freed, so that other processes may use the resources.
- A typical general-purpose computer runs multiple processes at a time.

If you are interested to find the list of processes in your Linux system, you can type `top` in your terminal to see all of them in real time. A single CPU achieves concurrency by <span style="color:#f77729;"><b>multiplexing</b></span> (perform rapid context switching) the executions of many processes.
{:.info}

### Kernel is not a process by itself

It is easy to perhaps <span style="color:#f7007f;"><b>misunderstand</b></span> that the kernel is a process.
{:.warning}

The kernel is <span style="color:#f7007f;"><b>not</b></span> a process in itself (albeit there are kernel _threads_, which runs in kernel mode from the beginning until the end, but this is _highly specific, e.g: in Solaris OS_).

For instance, I/O handlers are _not_ processes. They do not have <span style="color:#f77729;"><b>context</b></span> (state of execution in registers, stack data, heap, etc) like normal processes do. They are simply a piece of **instructions** that is written and executed to **handle** certain events.

You can think of a the kernel instead as made up of just **instructions** (a bunch of handlers) and **data**. Parts of the kernel deal with memory management, parts of it with scheduling portions of itself (like drivers, etc), and parts of it with scheduling processes.

User-mode processes can execute Kernel instructions to complete a particular <span style="color:#f77729;"><b>service</b></span> or **task** before resuming its own instructions:

- User processes running the kernel code will have its mode changed to <span style="color:#f77729;"><b>Kernel Mode</b></span>
- Once it returns from the handler, its mode is changed back to <span style="color:#f77729;"><b>User Mode</b></span>

Hence, the statement that the <span style="color:#f77729;"><b>kernel is a program that is running at all times</b></span> is technically true because the kernel <span style="color:#f7007f;"><b>IS</b></span> <span style="color:#f7007f;"><b>part of each process</b></span>. Any process may switch itself into Kernel Mode and perform system call routines (software interrupt), or forcibly switched to Kernel Mode in the event of hardware interrupt.

The Kernel <span style="color:#f7007f;"><b>piggybacks</b></span> on any process in the system to run.
{:.info}

If there are no system calls made, and no hardware interrupts, the kernel does nothing at this instant since it is not entered, and there’s nothing for it to do.

### Process Manager {#process-manager}

The **process manager** is part of the kernel code. It is part of the the **scheduler** subroutine, may be called either when there’s timed interrupt by the timed interrupt handler or trap handler when there’s system call made. It manages and keeps track of the system-wide **process table**, a data structure containing all sorts of information about current processes in the system. You will learn more about this in Week 3.

The process manager is responsible for the following tasks:

1. <span style="color:#f77729;"><b>Creation</b></span> and <span style="color:#f77729;"><b>termination</b></span> of both user and system processes
2. <span style="color:#f77729;"><b>Pausing</b></span> and <span style="color:#f77729;"><b>resuming</b></span> processes in the event of **interrupts** or **system calls**
3. <span style="color:#f77729;"><b>Synchronising</b></span> processes and provides <span style="color:#f77729;"><b>communications</b></span> between virtual spaces (Week 4 materials)
4. Provide mechanisms to handle deadlocks (Week 5 materials)

# Providing Security and Protection {#providing-security-and-protection}

The operating system kernel provides **protection** for the computer system by providing a mechanism for <span style="color:#f7007f;"><b>controlling access</b></span> of processes or users to resources defined by the OS.

This is done by identifying the <span style="color:#f7007f;"><b>user</b></span> associated with that process:

1. Using user identities (user IDs, security IDs): name and associated number among all numbers
2. User ID then associated with all files, processes of that user to determine <span style="color:#f7007f;"><b>access control</b></span>
3. For shared computer: group identifier (group ID) allows set of users to be defined and controls managed, then also associated with each process, file

You will learn more about this in Week 6 and Lab 3. During Lab 3, you will learn about <span style="color:#f7007f;"><b>Privilege Escalation</b></span>: an event when a user can change its ID to another effective ID with more rights. You may read on how this can happen in Linux systems [here](https://linux-audit.com/understanding-linux-privilege-escalation-and-defending-against-it/).

Finally, the Kernel also provides <span style="color:#f7007f;"><b>defensive security measures</b></span> for the computer system: protecting itself against internal and external attacks via firewalls, encryption, etc. There’s a huge range of security issues when a computer is connected in a network, some of them include denial-of-service, worms, viruses, identity theft, theft of service. You will learn more about this in Week 10.

# System Structure
## Multiprocessor System {#multiprocessor-system}

Unlike single-processor systems that we have learned before, multiprocessor systems have two or more processors in close communication, sharing the computer bus and sometimes the clock, memory, and peripheral devices.

Multiprocessor systems have three main advantages:

1. Increased <span style="color:#f7007f;"><b>throughput</b></span>: we can execute many processes in parallel
2. <span style="color:#f7007f;"><b>Economy of scale</b></span>: multiprocessor system is cheaper than multiple single processor system
3. Increased <span style="color:#f7007f;"><b>reliability</b></span>: if one core fails, we still have the other cores to do the work

### Symmetric Architecture

There are different architectures for multiprocessor system, such as a <span style="color:#f77729;"><b>symmetric</b></span> architecture — we have multiple CPU chips in a computer system:

<img src="{{ site.baseurl }}/assets/images/week1/15.png"  class="center_fourty"/>

Notice that each processor has its own set of registers, as well as a private or local cache; however, all processors share the <span style="color:#f77729;"><b>same</b></span> physical memory. This brings about design issues that we need to note in symmetric architectures:

1. Need to carefully control I/O o ensure that the data reach the <span style="color:#f77729;"><b>appropriate</b></span> processor
2. Ensure load <span style="color:#f77729;"><b>balancing</b></span>: avoid the scenario where one processor may be sitting idle while another is overloaded, resulting in inefficiencies
3. Ensure cache coherency: if a process supports multicore, makes sure that the data integrity spread among many cache is maintained.

### Multi-Core Architecture

Another example of a symmetric architecture is to have **multiple cores on the <span style="color:#f77729;"><b>same</b></span> chip** as shown in the figure below:

<img src="{{ site.baseurl }}/assets/images/week1/16.png"  class="center_fourty"/>

This carries an advantage that on-chip communication is <span style="color:#f77729;"><b>faster</b></span> than across chip communication. However it requires a more delicate hardware design to place multiple cores on the same chip.

### Asymmetric Multiprocessing

In the past, it is not so easy to add a second CPU to a computer system when operating system had commonly been developed for single-CPU systems. Extending it to handle multiple CPUs <span style="color:#f77729;"><b>efficiently</b></span> and reliably took a long time. To fill this gap, operating systems intended for single CPUs were initially extended to provide <span style="color:#f77729;"><b>minimal</b></span> support for a second CPU.

This is called <span style="color:#f77729;"><b>asymmetric</b></span> multiprocessing, in which each processor is assigned a <span style="color:#f77729;"><b>specific</b></span> task with the presence of a super processor that controls the system. This scheme defines a <span style="color:#f77729;"><b>master–slave</b></span> relationship. The master processor <span style="color:#f77729;"><b>schedules</b></span> and <span style="color:#f77729;"><b>allocates</b></span> work to the slave processors. The other processors either look to the master for instruction or have predefined tasks.


## Clustered System {#clustered-system}
Clustered systems (e.g: AWS) are multiple systems that are working together. They are usually:

1. Share <span style="color:#f77729;"><b>storage</b></span> via a storage-area-network
2. Provides <span style="color:#f77729;"><b>high availability</b></span> service which survives failures
3. Useful for <span style="color:#f77729;"><b>high performance computing</b></span>, require special applications that are written to utilize **parallelization**

There are two types of clustering:

1. <span style="color:#f77729;"><b>Asymmetric</b></span>: has one machine in hot-standby mode. There’s a master machine that runs the system, while the others are configured as slaves that receive tasks from the master. The master system does the basic input and output requests.
2. <span style="color:#f77729;"><b>Symmetric</b></span>: multiple machines (nodes) running applications, they are monitoring one another, requires complex algorithms to maintain data integrity.


# Summary

We have learned that the Operating System is a software that acts as an <span style="color:#f77729;"><b>intermediary</b></span> between a user of a computer and the computer hardware. It is comprised of the Kernel, system programs, and user programs. Each OS is shipped with different flavours of these programs, depending on its design.

In general, an operating system is made with the following goals in mind:

- <span style="color:#f77729;"><b>Execute</b></span> user programs and make solving user problems easier
- Make the computer system <span style="color:#f77729;"><b>convenient</b></span> to use
- Use the computer hardware in an <span style="color:#f77729;"><b>efficient</b></span> manner

It is a huge piece of software, and usually are divided into separate subsystems based on their roles:

1. Process Management
2. Resource Allocator and Coordinator
3. Memory and Storage Management
4. I/O Management,

... and many others. There's too many to go into detail since modern Operating Systems kept getting expanded to improve user experience.

The <span style="color:#f77729;"><b>Kernel</b></span> is the heart of the OS, the central part of it that holds important pieces of instructions that support the roles stated above in the bare minimum. In the next chapter, we will learn how to <span style="color:#f77729;"><b>access</b></span> crucial OS Kernel services.

This chapter elaborates on the critical functionalities and management roles of the OS kernel. It provides a thorough understanding of how the kernel orchestrates interaction between software and hardware, manages processes and memory, and ensures system security and efficiency.

Key learning points include:
- **Interrupt-Driven I/O Operations**: Differentiating between hardware and software interrupts, and explaining vectored versus polled interrupt systems.
- **Process and Memory Management**: How the kernel manages virtual memory, handles multiple processes, and optimizes cache performance.
- **Kernel Modes**: Differences between reentrant and preemptive kernels, and their implications for multitasking.


# Appendix



## Timed Interrupts {#timed-interrupts}

We must ensure that the kernel, as a <span style="color:#f7007f;"><b>resource allocator</b></span> maintains <span style="color:#f77729;"><b>control</b></span> over the CPU. We cannot allow a user program to get stuck in an infinite loop and never return control to the OS. We also cannot trust a user program to voluntarily return control to the OS. To ensure that no user program can occupy a CPU for indefinitely, a computer system comes with a (<span style="color:#f77729;"><b>hardware</b></span>)-based timer. A timer can be set to invoke the hardware interrupt line so that a running user program may transfer control to the kernel after a specified period. Typically, a <span style="color:#f7007f;"><b>scheduler</b></span> will be invoked each time the timer interrupt occurs.

In the hardware, a timer is generally implemented by a **fixed-rate clock** and a counter. The kernel may set the starting value of the counter, just like how you implement a custom clock in your 1D 50.002 project, for instance, here's one simple steps describing how timed interrupts work:

1. Every time the sytem clock ticks, a counter with some starting value `N` is decremented.
2. When the counter reaches a certain value, the timer can be hardware triggered to set the <span style="color:#f77729;"><b>interrupt request line</b></span>

For instance, we can have a 10-bit counter with a 1-ms clock can be set to trigger interrupts at intervals anywhere from 1 ms to 1,024 ms, with precision of 1 ms. Let's say we decide that every 512 ms, timed interrupt will occur, then we can say that the size of a **quantum** (time slice) for each process is 512ms. We can give multiple quantums (quanta) for a user process.

When timed interrupt happens, this transfers control over to the interrupt handler, and the following routine is triggered:

- <span style="color:#f77729;"><b>Save</b></span> the current program's state
- Then call the <span style="color:#f77729;"><b>scheduler</b></span> to perform context switching
- The scheduler may then reset the counter before restoring the next process to be executed in the CPU. This ensures that a proper timed interrupt can occur in the future.
- Note that a scheduler may allocate <span style="color:#f77729;"><b>arbitrary</b></span> amount of time for a process to run, e.g: a process may be allocated a longer time slot than the other. We will learn more about process management in Week 3.

## IVT in various Architectures


Below is an example of ARMv8-M interrupt vector table, which describe which RAM addresses should contain the **handler entry** (also called interrupt service routine, ISR) for all sorts of interrupts (both software and hardware). The table is typically implemented in the <span style="color:#f77729;"><b>lower</b></span> physical addresses in many architecture.

<img src="{{ site.baseurl }}/assets/images/week1/13.png"  class="center_seventy" title="Image taken from https://developer.arm.com/documentation/100701/0200/Exception-properties"/>

Each exception has an ID (associated number), a vector <span style="color:#f77729;"><b>address</b></span> that is the exception <span style="color:#f77729;"><b>entry</b></span> point in memory, and a <span style="color:#f77729;"><b>priority</b></span> level which determines the order in which multiple pending exceptions are handled. In ARMv8-M, the lower the priority number, the higher the priority level.

Based on the instruction set spec of ARMv8, integer division by zero returns zero and not trapped.
{:.info}

In x86 architecture, exception handlers are normally found via so called Interrupt Descriptor Table or **IDT** for short. The IDT is typically located in memory at a specific base address, stored in Interrupt Descriptor Table Register (IDTR, a special register). The value of this base address **depends on the operating system**. In some older hardware like [Intel 8086](https://en.wikipedia.org/wiki/Intel_8086), the interrupt vector table is placed at a fixed location: always located in the first 1024 bytes of memory at addresses `0x000000–0x0003FF` and the more complex config of IDT + IDTR does not exist.

The IDT contains up to **256** entries and each of those entries is 16 bytes in size in 64 bit mode. For instance, a **division by zero** exception exists in x86 (unlike in ARMv8), and can be triggered by the CPU automatically (e.g: hardware is built to check that a divisor is not 0). By convention, this directs the PC to exec RAM address containing that division-by-zero handler **entry** directly (e.g: address `0`, as it is customary to have division by zero as the first handler entry of IVT).

<img src="{{ site.baseurl }}//assets/images/week1-3_resource/2023-05-16-16-17-06.png"  class="center_seventy"/>

## Reentrant Kernel Behavior

Here we explore a kernel-related analogy using simplified examples in C that demonstrate reentrant vs. non-reentrant behavior.

### Non-Reentrant Kernel Function

```c
#include <stdio.h>
#include <pthread.h>

#define BUFFER_SIZE 100
char global_buffer[BUFFER_SIZE];
int buffer_index = 0;

void non_reentrant_syscall(char data) {
    global_buffer[buffer_index++] = data;
    // Simulate processing
    printf("Processing: %s\n", global_buffer);
}

void* thread_function(void* arg) {
    char data = *(char*)arg;
    non_reentrant_syscall(data);
    return NULL;
}

int main() {
    pthread_t thread1, thread2;
    char data1 = 'A', data2 = 'B';

    pthread_create(&thread1, NULL, thread_function, &data1);
    pthread_create(&thread2, NULL, thread_function, &data2);

    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);

    return 0;
}
```

In this example, `non_reentrant_syscall` modifies a <span class="orange-bold">global</span> buffer. If two threads call this function concurrently, it may lead to race conditions and inconsistent states in the `global_buffer`.

### Reentrant Kernel Function

```c
#include <stdio.h>
#include <pthread.h>
#include <string.h>

#define BUFFER_SIZE 100

void reentrant_syscall(char* buffer, char data) {
    int len = strlen(buffer);
    buffer[len] = data;
    buffer[len + 1] = '\0';
    // Simulate processing
    printf("Processing: %s\n", buffer);
}

void* thread_function(void* arg) {
    char data = *(char*)arg;
    char local_buffer[BUFFER_SIZE] = "";
    reentrant_syscall(local_buffer, data);
    return NULL;
}

int main() {
    pthread_t thread1, thread2;
    char data1 = 'A', data2 = 'B';

    pthread_create(&thread1, NULL, thread_function, &data1);
    pthread_create(&thread2, NULL, thread_function, &data2);

    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);

    return 0;
}
```

In this example, `reentrant_syscall` operates on a local buffer passed as an argument. Each thread has its own buffer, avoiding shared state and making the function safe for concurrent execution.

### Analogy to Kernel

- **Non-Reentrant Kernel Function**: Uses global state (e.g., `global_buffer`). If two system calls modify the global state concurrently, it can lead to data corruption or inconsistent system states. This is like a kernel that isn't designed for reentrancy and fails under concurrent execution.
  
- **Reentrant Kernel Function**: Uses local state or parameters (e.g., `local_buffer`). Each system call operates independently on its own data, making it safe for concurrent execution. This is like a kernel deliberately designed for reentrancy, ensuring correct and consistent behavior under concurrent execution.

In an operating system, reentrant kernel functions are <span class="orange-bold">crucial</span> for handling multiple processes or threads safely, ensuring that shared resources are not corrupted and system stability is maintained.
<hr>


[^5]: Interrupt-driven I/O is fine for moving small amounts of data but can produce high overhead when used for bulk data movement such as disk I/O. To solve this problem, direct memory access (DMA) is used. After setting up buffers, pointers, and counters for the I/O device, the device controller transfers an entire block of data directly to or from its own buffer storage to memory, with no intervention by the CPU.
[^6]: The CPU cache is a hardware cache: performing optimization that’s unrelated to the functionality of the software. It handles each and every access between CPU cache and main memory as well. They need to work fast, too fast to be under software control, and _are entirely built into the hardware._


[^2]: Since only a predefined number of interrupts is possible, **a table of pointers to interrupt routines** can be used instead to provide the necessary speed. The interrupt routine is called indirectly through the table, with no intermediate routine needed. Generally, the table of pointers is stored in low memory (the first hundred or so locations), and its values are determined at boot time. These locations hold the addresses of the interrupt service routines for the various devices. This array, or **interrupt vector**, of addresses is then indexed by a unique device number, given with the interrupt request, to provide the address of the interrupt service routine for the interrupting device. Operating systems as different as Windows and UNIX dispatch interrupts in this manner.
[^3]: Depending on the implementation of the handler, <strong>a <em>full </em>context switch (completely saving ALL register states, etc) may not be necessary</strong>. Some OS may implement interrupt handlers in low-level languages such that it knows exactly which registers to use and only need to save the states of these registers first before using them.
[^4]: A reentrant procedure can be interrupted in the middle of its execution and then safely be called again ("re-entered") before its previous invocations complete execution. There are several problems that must be carefully considered before declaring a procedure to be _reentrant_. All user procedures are reentrant, but it comes at a costly procedure to prevent lost data during interrupted execution.