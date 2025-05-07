---
layout: default
permalink: /os/1-os-intro
title: Introduction to Operating System
description: Background motivation on Operating System and its basic components
parent: Operating System
nav_order:  1
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

# Introduction to Operating System
{: .no_toc}

{:.highlight-title}
> Detailed Learning Objectives
>
> 1. **Basics of Operating Systems**
>   - Define what an operating system is
>   - Define an operating system and its basic role as an **intermediary** between hardware and users.
>   - Describe computer system organisation (basic architecture) including hardware, operating system, application programs, and users.
> 2. **Examine the Functions of an Operating System**
>   - Identify the operating system as a **resource allocator**, program **execution** controller, and **security** enforcer.
>   - Articulate the **division** of operating system **components** into the kernel, system programs, and user programs.
> 3. **Explore the Kernel**
>   - Explain the high-level **functions** and **privileges** of the operating system kernel.
>   - Discuss the kernel's role in **managing** the computer's hardware and memory hierarchy.
>   - Explain dual mode operation and hardware support for kernel and user modes.
> 4. **Analyze Computer System Architecture**
>   - **Describe** the components and operations of the memory hierarchy including registers, caches, main memory, and secondary storage.
>   - Outline the **booting** process and the role of firmware or BIOS in system startup.
> 5. **Investigate I/O Operations and Device Management**
>   - Outline the roles of **device controllers** and device **drivers** in managing I/O operations.
>   - Explain how device drivers interface with hardware devices and the implications of running drivers in kernel or user mode.
>
> These learning objectives are designed to guide your study of operating systems, ensuring a comprehensive understanding of their structure, functions, and roles within computer systems.

An operating system (OS) is a program that **manages computer hardware**.
{:.info}

The figure below shows the hardware components of a common general purpose computer. There are many user programs that are running in a computer, and the OS acts as an intermediary application that enables many user programs to share the same set of hardware, such as the mouse, printer, keyboard, display monitor, etc.

<img src="/50005/assets/images/week1/1.png"  class="center_seventy"/>

## The Operating System {#the-operating-system}

An operating system is a **special program** that acts as an intermediary between users of the computer and the computer hardware.

The goal of an operating system is such that we have a **dedicated program** to fulfil the following essential roles:

1. **Resource allocator and coordinator**:
   - Controls hardware and input/output requests
   - Manage resource conflicting requests
   - Manage interrupts
2. **Controls program execution**:
   - Storage hierarchy manager
   - Process manager
3. Limits program execution and ensure **security**:
   - Preventing illegal access to the hardware or improper usage of the hardware

Once we have an _operating system_, it makes things easier for users to **use** a program or **write** another program for other purposes within a **computer system.**

There are a lot of things that make up an operating system, but they are generally divided into three categories:

1. **The Kernel**
2. **System programs**
3. **User programs**

Definition and role of **<span style="color:#f77729;"><b>operating system kernel</b></span>** can be found in another section below, and it is the only program with <span style="color:#f7007f;"><b>full</b></span> privileges, i.e: absolute access to control all the hardware in the computer system.

Both system programs and user programs run in <span style="color:#f7007f;"><b>user mode</b></span>, with limited privileges, i.e: any of these programs have to send a _request_ to the kernel each time they require access to the I/O or hardware devices.

## The Computer System {#the-computer-system}

A computer system can be roughly divided into **four** components: the hardware, the operating system, the application programs (also known as **user programs**), and the users as shown in the figure below.

<img src="/50005/assets/images/week1/2.png"  class="center_seventy"/>

The operating system is part of the computer system and is analogous to a **government**.
{:.info}

The OS provides an _<span style="color:#f77729;"><b>environment</b></span>_ such that user programs such as the text editor, web browser, compiler, database system, music player, video editor, etc can do useful work. Since each user program runs in a <span style="color:#f77729;"><b>virtual machine</b></span> (i.e: it is written in a manner that the _entire_ machine belongs to itself), there has to be some sort of _manager_ program that **has higher privileges** and **oversees** all programs that live on the RAM and reside on disk, as well as managing the **memory hierarchy**.

This special program is part of the operating system called the **<span style="color:#f7007f;"><b>kernel</b></span>**.
{:.info}

## The Memory Hierarchy {#the-memory-hierarchy}

The storage structure in a typical computer system is made of **registers**, **caches**, **main** **memory**, and **non-volatile secondary storage** such as magnetic disk. The wide variety of storage systems can be arranged in terms of hierarchy according to **speed and cost** (increasing speed and increasing cost from bottom up):

<img src="/50005/assets/images/week1/3.png"  class="center_seventy"/>

- The CPU can load instructions only from memory, so any programs to run must be stored there.
- General-purpose computers run most of their programs from rewritable memory, called main memory (RAM).
- At each CPU clock cycle, instructions are fetched from the main memory to the CPU.

### The RAM

Ideally, we want the programs and data to reside in main memory (also known as physical memory, or RAM) <span style="color:#f7007f;"><b>permanently</b></span>. This arrangement usually is **not possible** for two reasons:

1. Main memory is usually too small to store all needed programs and data permanently.
2. Main memory is a <span style="color:#f7007f;"><b>volatile</b></span> storage device that loses its contents when power is turned off or otherwise lost.

Recall that the memory unit sees only a stream of memory addresses; it does not know how they are generated (by the instruction counter, indexing, indirection, literal addresses, or some other means) or what they are for (instructions or data).
{:.info}

### Cache

Cache devices are typically used to <span style="color:#f77729;"><b>speed</b></span> up the performance of the computer. They are storing (typically) a few of the most recently used instruction pages. Cache devices are wired directly to the CPU so that the CPU has direct access to it (unlike secondary storages), just like how the CPU can directly access the RAM.

You may think of it as a more-expensive, smaller-but-faster RAM.
{:.info}

Because caches have limited size, cache management is an important design problem. The **kernel** dictates details pertaining to cache management, such as which supported <span style="color:#f77729;"><b>cache replacement policies</b></span> should be used for the system.

# Kernel
The one **program** that is running at all times in the computer is the kernel.
{:.info}

The Kernel is the **<span style="color:#f7007f;"><b>heart</b></span>** of an operating system.

- For example, Ubuntu OS is about 2.7GB, but its Kernel (Linux) size is only about 70MB).
- It operates on the <span style="color:#f7007f;"><b>physical space</b></span> — meaning that it has full knowledge of all <span style="color:#f7007f;"><b>physical</b></span> addresses instead of <span style="color:#f77729;"><b>virtual</b></span> addresses, and has the <span style="color:#f7007f;"><b>complete privilege</b></span> over all the hardware of the computer system.
- The only way to access the kernel code is when a process runs in the Kernel mode, via very specific <span style="color:#f7007f;"><b>controlled entry points</b></span>.

You have learned this before: for instance via `ILLOP`, `IRQ`, and `RESET`.

## Kernel Mode

The kernel runs with special privileges, called the <span style="color:#f7007f;"><b>kernel mode</b></span>. It can do what normal user program cannot do:

1. Ultimate access and control to all hardware in the computer system (mouse, keyboard, display, network cards, disk, RAM, CPU, etc)
2. Know (and lives in) the physical address space and manages the memory hierarchy
3. Interrupt other user programs
4. Receive and manage I/O requests
5. Manage other user program locations on the RAM, the MMU, and schedule user program executions

In order for the **kernel** to have more _privileges_ than **other user mode programs**, the computer hardware has to support **dual mode operation.**
{:.info}

You have learned this before as well, i.e: `PC31` in the Beta CPU indicates whether the current instruction is run in the Kernel mode or the User mode. Note that using PC31 as Kernel/User mode indicator is specific to Beta CPU. Other CPU architecture such as the x86 and ARM uses special registers ([FLAGS](https://en.wikibooks.org/wiki/X86_Assembly/X86_Architecture) register for x86 and [CPSR for certain ARM architecture](https://developer.arm.com/documentation/den0013/d/ARM-Processor-Modes-and-Registers)) for this purpose. The details about other CPU architecture is out of our syllabus, but the concept is similar.

## Hardware Support for Dual Mode Operation {#dual-mode-operation}

<img src="/50005/assets/images/week1/4.png"  class="center_fifty"/>

The dual mode is possible <span style="color:#f7007f;"><b>iff</b></span> it is supported by the hardware. The kernel is also typically **uninterruptible** in older CPU designs, and this interruptible feature is also supported by the hardware. There are some more complex CPU designs that allows interrupt even when the CPU is in kernel mode (based on some kind of priority) but this requires more complex interrupt handling routine (need to save progress, etc).
{:.info}

### Differences between architectures

In 50.002, we have learned that the control logic unit **prevents** the PC to `JMP` to memory address with MSB bit of `1` (where the kernel program resides) when it is at memory address with MSB bit 0 (where user programs reside).

Also, the control logic unit does not _trap_ the PC onto the handler when an interrupt signal is present if the PC is running in kernel mode (MSB of the PC is 1).

In the Linux system, low (virtual) memory region is dedicated for the kernel and high memory is assigned for user processes. It is essentially the same as what we have learned before: the concept of having _dual_ mode and _hardware support_.

There has to be some kind of <span style="color:red; font-weight: bold;">hardware-dependent implementation</span> that protects the Kernel memory region, e.g: prevents the PC from jumping _illegally_ (not via handlers) to a lower memory address (MSB = 0) when it was from a higher memory address (MSB = 1).

Note that using MSB of PC register as a flag to indicate kernel/user mode is only specific to our Beta CPU implementation. Actual implementation vary widely. For instance, x86 architecture uses [dedicated FLAGS registers](https://en.wikipedia.org/wiki/FLAGS_register). Such detail is outside of our syllabus.
{: .error}

### The Big Picture

A general purpose CPU has at least dual mode operation that should be supported by its hardware:

1. **<span style="color:#f7007f;"><b>The Kernel mode</b></span>** (privileged) : the executing code has complete and unrestricted access to the underlying hardware.
2. **<span style="color:#f77729;"><b>The User mode</b></span>** (unprivileged) : all user programs such as a web browser, word editor, etc and also system programs such as compiler, assembler, file explorer, etc runs on _virtual machine_

User programs have to perform **system calls** (supervisor call) when they require services from the kernel, such as access to the hardware or I/O devices. When they perform **system calls**, the user program changes its mode to **the kernel mode** and began executing the kernel instructions handling that call instead of their own program instructions. When the system call returns, the `PC` resumes the execution of the user program.

# Booting {#booting}

Booting is the process of starting up a computer. **It is usually hardware initiated** — (by the start button that users press) — meaning that users physically initiate simple hardwired procedures to kickstart the chain of events that loads the firmware (BIOS) and eventually the entire OS to the main memory to be executed by the CPU. This process of loading basic software to help kickstart operation of a computer system after a hard reset or power on is called **bootstrapping**.

Recall that programs (including the operating system kernel) must **load** into the main memory before it can be executed. However, at the instance when the start button is pressed, there’s no program that resides in the RAM yet and therefore nothing can be executed in the CPU.

We require a software to load another software into the RAM — this results in a <span style="color:#f7007f;"><b>paradox</b></span>.
{:.info}

## The Booting Paradox

To solve this paradox, the bare minimum that should be done in the hardware level upon pressing of the start button is to load a _special_ program onto the main memory from a dedicated input unit: **a <span style="color:#f77729;"><b>read-only-memory</b></span> (ROM) that comes with a computer when it is produced and that cannot be erased.** This special program is generally known as **firmware or BIOS**[^1].

After the firmware is loaded onto the main memory through hardwired procedures, the CPU may execute it and **initialise all aspects of the system, such as:**

1. <span style="color:#f77729;"><b>Prepare</b></span> all attached devices in a state that is ready to be used by the OS
2. <span style="color:#f77729;"><b>Loads</b></span> other programs — which in turn loads more and more complex programs,
3. <span style="color:#f77729;"><b>Loads</b></span> the Kernel from disk
4. When the system boots, the hardware starts in the **kernel mode**. After being loaded, the Kernel will perform the majority of system setups (driver init, memory management, interrupts, etc). Afterwards, the rest of the OS is loaded and then user processes are started in _<span style="color:#f77729;"><b>user mode</b></span>_.

The figure below summarises the booting process:

<img src="/50005/assets/images/week1/5.png"  class="center_seventy"/>

Note that the figure is heavily simplified for illustration purposes only.

# Computer System I/O Operation {#computer-system-i-o-operation}

There are **two** types of hardware in the computer system that are capable of running instructions:

1. The <span style="color:#f7007f;"><b>CPU</b></span> (obviously!)
2. <span style="color:#f77729;"><b> I/O device controllers</b></span>

Each I/O device is managed by an autonomous hardware entity called the **<span style="color:#f77729;"><b>device controllers</b></span>** as shown in the figure below:

<img src="/50005/assets/images/week1/6.png"  class="center_seventy"/>

In other words, I/O devices and the CPU can execute **instructions** in parallel. They are independent of one another and are <span style="color:#f7007f;"><b>asynchronous</b></span>
{:.info}

## Device Drivers {#device-drivers}

A system must have **device drivers installed** for each device type. This driver is a specific program to <span style="color:#f7007f;"><b>interpret</b></span> the behavior of each device type. We typically install/download _device drivers_ when we plug in **new** I/O units to our computers through the USB port.

<img src="/50005/assets/images/week1/7.png"  class="center_seventy no-invert"/>

It provides a software **interface** to hardware devices so that the device controller is able to communicate with the OS or an application program.
{:.info}

### Running Device Drivers

Some default device drivers are part of the kernel code, and may consist of interfaces that control one or more common devices that can be attached to a system, such as hard disks, GPUs, keyboards, mouses, monitors, and network interfaces. In other words, device drivers are modules that can be <span style="color:#f77729;"><b>plugged</b></span> into an OS to handle a particular device or category of similar devices. Many drivers run in kernel mode (therefore some requires _reboot upon installation_), but there are also drivers that can run in user mode.

Those drivers that run in user mode will be _<span style="color:#f77729;"><b>slower</b></span>_ in comparison, since <span style="color:#f77729;"><b>frequent switching</b></span> to kernel mode is required to access the **serial ports** of the device controller that’s connected to the external devices. However, if it was <span style="color:#f7007f;"><b>poorly</b></span> written, <span style="color:#f77729;"><b>it will not endanger the system </b></span>by, for example, accidentally overwriting the kernel memory.

For drivers that run in kernel mode, <span style="color:#f7007f;"><b>vulnerabilities</b></span> in these drivers can pose a serious threat as they can allow an attacker to escalate privileges to the highest level and become highly <span style="color:#f7007f;"><b>persistent</b></span>.

One should only install drivers from <span style="color:#f77729;"><b>trusted</b></span> vendors.
{:.error}

## Device Controllers {#device-controllers}

Device controllers are **<span style="color:#f7007f;"><b>electronic components</b></span>** inside a computer that are in charge of specific types of devices. Components that make up the **device controllers:**

1. <span style="color:#f77729;"><b>Registers</b></span> — contains instructions that can be read by an appropriate **device driver program at the CPU**
2. Local memory <span style="color:#f77729;"><b>buffer</b></span> — contains instructions and data that will be fetched by the CPU when executing the device driver program, and ultimately loaded onto the RAM.
3. A <span style="color:#f77729;"><b>simple</b></span> program to _communicate_ with the device driver

### I/O
I/O operation happens when there’s transfer of data between the local memory buffer of the device controller and the device itself.
{:.info}

The I/O operation  means:

1. Output: <span style="color:#f77729;"><b>Move</b></span> data from the device controller's buffer to the output device, or
2. Input: <span style="color:#f77729;"><b>From</b></span> the input device to the device controller’s buffer.

Since the device controller and our CPU are <span style="color:#f7007f;"><b>asynchronous</b></span> (can operate independently, in parallel), we need to devise a way to <span style="color:#f7007f;"><b>coordinate</b></span> between servicing I/O requests and executing other user programs. This <span style="color:#f77729;"><b>I/O handling</b></span> issue is handled by our OS Kernel (read along to understand _how_).

# Summary

This notes provide foundational knowledge for understanding how operating systems function as critical components in computing. It details the role of operating systems in managing hardware resources, executing user programs, and ensuring system security. Operating systems (OS) serve as a **crucial** intermediary between a computer's hardware and its users, facilitating the management of hardware resources, program execution, and system security. The OS's core component, the kernel, plays a pivotal role in handling hardware operations and memory management, ensuring efficient processing and resource allocation. This introduction also discusses the architecture of computer systems, the memory hierarchy, and the process of booting, providing a **foundational** understanding of how operating systems **integrate** and **manage** computing resources.


Key learning points include:
- **Role and Functions**: Operating systems act as intermediaries between computer applications and the hardware, optimizing resource use and user interaction.
- **System Structure**: Emphasis on how different components like the kernel, user space, and system calls interact within a computing system.
- **Process Management**: Explains how operating systems manage multiple processes, ensuring efficient execution and resource sharing.
- **I/O and Device Management**: Discusses methods for handling input/output operations and managing various hardware devices connected to the system.


<hr>

[^1]: Firmware is not equivalent to BIOS, but unfortunately some resources and PC manufacturers might just use them interchangeably. Firmware generally refers to software stored on the motherboard (of any devices like computers, routers, switches, etc), containing basic settings of the device at startup. Some firmwares are upgradable, while some are Read-Only. BIOS is a term generally used specifically to refer to computer’s motherboard firmware in older computers. Modern computers use other Firmwares such as UEFI, also stored on chips on the motherboard. Note that UEFI / BIOS don’t form the entirety of a motherboard’s firmware.
