---
layout: default
permalink: /os/processes
title: Processes 
description: How are processes managed by the OS?
parent: Operating System
nav_order:  5
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

# Processes

{:.highlight-title}
> Detailed Learning Objectives
> 
> - **Understand the Difference Between Process and Program**
>   - Recognize that a process is a dynamic execution instance of a static program.
>   - Acknowledge that a program is a passive entity containing the code, whereas a process is an active entity with executing code and associated resources.
> - **Explore Process Context**
>   - Identify what constitutes a process' context including the program counter, processor's registers, address space, and more.
>   - Understand how multiple instances of a program can run as separate processes.
> - **Concurrency and Protection in Processes**
>   - Grasp how processes allow for concurrency and provide protection, enabling processes to operate as if they are the only ones running on the system.
> - **Introduction to Process Scheduling States**
>   - Learn about different process states (New, Running, Waiting, Ready, Terminated) and the lifecycle of a process.
>   - Understand the concept of a process table and a Process Control Block (PCB) which store process attributes.
> - **Process Management and Scheduling**
>   - Comprehend how processes are managed and scheduled by the operating system, including the role of the scheduler.
>   - Familiarize with context switching and its impact on system performance.
>
> These objectives are structured to provide a comprehensive overview of the concepts related to processes versus programs, their management, and the system's handling of multiple processes.

## Process vs Program {#the-concept-of-process-vs-program}

A <span style="color:#f77729;"><b>process</b></span> is formally defined as a program in execution. Process is an active, <span style="color:#f7007f;"><b>dynamic</b></span> entity -- i.e: it changes state overtime during execution, while a program is a passive, <span style="color:#f7007f;"><b>static</b></span> entity. A program is the code and static data as they exist on the disk. A process is *so much more* than just a program code (text or instruction section)
{:.info}

Before we proceed, please remember that <span style="color:#f7007f;"><b>a program is not a process</b></span>.

### Process Context

A single process includes all of the following information, which we call the process' *context*:

1. The <span style="color:#f77729;"><b>text</b></span> section (code or instructions)
2. Value of the Program Counter (<span style="color:#f77729;"><b>PC</b></span>)
3. Contents of the processor’s <span style="color:#f77729;"><b>registers</b></span>
4. Dedicated[^1] <span style="color:#f77729;"><b>address space</b></span> (block of location) in memory
5. <span style="color:#f77729;"><b>Stack</b></span> (temporary data such as function parameters, return address, and local variables, grows downwards),
6. <span style="color:#f77729;"><b>Data</b></span> (allocated memory during compile time such as global and static variables that have predefined values)
7. <span style="color:#f77729;"><b>Heap</b></span> (dynamically allocated memory -- typically by calling `malloc `in C -- during process runtime, grows upwards)[^2]
   These information are also known as a process <span style="color:#f7007f;"><b>state</b></span>, or a process <span style="color:#f7007f;"><b>context</b></span>:

The same program can be run `n` times to create `n` processes simultaneously.

- For example, separate tabs on some web browsers are created as <span style="color:#f77729;"><b>separate processes</b></span>.
- The <span style="color:#f77729;"><b>program</b></span> for all tabs are the same: which is the part of the web browser code itself.

A program becomes a process when an <span style="color:#f77729;"><b>executable</b></span> (binary) file is <span style="color:#f77729;"><b>loaded</b></span> into memory (either by double clicking them or executing them from the command line)
{:.note}

### Concurrency and Protection

A process couples <span style="color:#f7007f;"><b>two abstractions</b></span>: <span style="color:#f7007f;"><b>concurrency</b></span> and <span style="color:#f7007f;"><b>protection</b></span>.
{:.note}

Each process runs in a different address space and sees itself as running in a virtual machine -- unaware of the presence of other processes in the machine. Multiple processes execution in a single machine is <span style="color:#f7007f;"><b>concurrent</b></span>, managed by the <span style="color:#f7007f;"><b>kernel scheduler</b></span>.

## Piggybacking

From this point onwards, we are going to refer simply to the scheduler as the kernel. Remember this is just a running code (service routine) with kernel privileges whose task is to manage processes. Scheduler in itself, is part of the kernel, and is <span style="color:#f7007f;"><b>not</b></span> a process.

Recall how the system calls <span style="color:#f7007f;"><b>piggyback</b></span> the currently running user-process which mode has been changed to kernel mode. Likewise, the scheduler is just a set of instructions, part of the kernel realm whose job is to <span style="color:#f77729;"><b>manage</b></span> other processes. It is inevitable for the scheduler to be executed as well upon invoking certain system calls such as `read` or get `stdin` input which requires some <span style="color:#f7007f;"><b>waiting</b></span> time.

# Process Management
## Process Scheduling States {#process-scheduling-state}

As a process executes, it changes its <span style="color:#f7007f;"><b>scheduling state</b></span> which reflects the <span style="color:#f7007f;"><b>current</b></span> activity of the process.
{:.warning}

In general these states are:

1. <span style="color:#f77729;"><b>New</b></span>: The process is being created.
2. <span style="color:#f77729;"><b>Running</b></span>: Instructions are being executed.
3. <span style="color:#f77729;"><b>Waiting</b></span>: The process is waiting for some event to occur (such as an I/O completion or reception of a signal).
4. <span style="color:#f77729;"><b>Ready</b></span>: The process is waiting to be assigned to a processor
5. <span style="color:#f77729;"><b>Terminated</b></span>: The process has finished execution.

The figure below shows the scheduling state transition diagram of a typical process:
<img src="{{ site.baseurl }}/assets/images/week3/1.png"  class="center_fifty no-invert"/>

## Process Table and Process Control Block {#process-control-block}

The system-wide process table is <span style="color:#f77729;"><b>data structure</b></span> maintained by the Kernel to facilitate **context switching** and **scheduling**. Each process metadata is <span style="color:#f77729;"><b>stored</b></span> by the Kernel in a particular data structure called the process control block (PCB). A process table is made up of an array of PCBs, containing information about of current processes[^3] in the system.
{:.warning}



The PCB contains many pieces of information associated with a specific process. These information are up<span style="color:#f77729;"><b></b></span>dated each time when a process is <span style="color:#f77729;"><b>interrupted</b></span>:

1. Process <span style="color:#f77729;"><b>state</b></span>: any of the state of the process -- new, ready, running, waiting, terminated
2. <span style="color:#f77729;"><b>Program counter</b></span>: the address of the _next instruction_ for this process
3. CPU <span style="color:#f77729;"><b>registers</b></span>: the contents of the registers in the CPU when an interrupt occurs, including stack pointer, exception pointer, stack base, linkage pointer, etc. These contents are saved each time to allow the process to be continued correctly afterward.
4. <span style="color:#f77729;"><b>Scheduling </b></span> information: access priority, pointers to scheduling queues, and any other scheduling parameters
5. <span style="color:#f77729;"><b>Memory-management</b></span> information: page tables, MMU-related information, memory limits
6. <span style="color:#f77729;"><b>Accounting</b></span> information: amount of CPU and real time used, time limits, account numbers, process id (<span style="color:#f77729;"><b>pid</b></span>)
7. <span style="color:#f77729;"><b>I/O status</b></span> information: the list of I/O devices allocated to the process, a list of open files

PCB is also called a <span style="color:#f77729;"><b>task control block</b></span> in some textbooks.
{: .note}

### Linux task_struct

In Linux system, the PCB is created in C using a data structure called `task_struct`. The diagram below[^4] illustrates some of the contents in the structure:
<img src="{{ site.baseurl }}/assets/images/week3/2.png"  class="center_fifty"/>

Do not memorize the above, it's just for illustration purposes only. The task_struct is a relatively large data structure, at around 1.7 kilobytes on a 32-bit machine.
{:.error}

Within the Linux kernel, all active processes are represented using a <span style="color:#f77729;"><b>doubly linked list</b></span> of `task_struct.` The kernel maintains a `current_pointer` to the process that's currently <span style="color:#f77729;"><b>running</b></span> in the CPU.

<img src="{{ site.baseurl }}/assets/images/week3/3.png"  class="center_fourty no-invert"/>

### Context Switching

When a CPU switches execution between one process to another, the Kernel has to <span style="color:#f77729;"><b>store</b></span> all of the process states onto its corresponding PCB, and <span style="color:#f77729;"><b>load</b></span> the new process’ information from its PCB before resuming them as shown below, (_image screenshot from SGG book_):

<img src="{{ site.baseurl }}/assets/images/week3/4.png"  class="center_fifty"/>

## Rapid Context Switching and Timesharing {#rapid-context-switching-and-timesharing}

<span style="color:#f77729;"><b>Context switch</b></span> definition: the mechanism of <span style="color:#f77729;"><b>saving</b></span> the states of the current process and <span style="color:#f77729;"><b>restoring</b></span> (loading) the state of a different process when switching the CPU to execute another process.
{:.warning}

### Timesharing Support

Timesharing requires <span style="color:#f77729;"><b>interactivity</b></span>, and this is done by performing rapid context switching between execution of multiple programs in the computer system.

When an interrupt occurs, the kernel needs to save the current context of the process running on the CPU so that it can restore that context when its processing is done, essentially <span style="color:#f77729;"><b>suspending</b></span> the process and then resuming it at the later time. The suspended process context is stored in the <span style="color:#f77729;"><b>PCB</b></span> of that process.

#### Benefits of Context Switching

Rapid Context switching is beneficial because it gives the <span style="color:#f77729;"><b>illusion</b></span> of concurrency in uniprocessor system.

- Improve system <span style="color:#f77729;"><b>responsiveness</b></span> and intractability, ultimately allowing timesharing (users can interact with each program when it is running).
- To support <span style="color:#f77729;"><b>multiprogramming</b></span>, we need to <span style="color:#f77729;"><b>optimise</b></span> CPU usage. We cannot just let one single program to run all the time, especially when that program <span style="color:#f77729;"><b>blocks</b></span> execution when waiting for I/O (idles, have nothing important to do).

### Drawback of Context Switching

Context-switch time is pure <span style="color:#f7007f;"><b>overhead</b></span>, because the system <span style="color:#f7007f;"><b>does no useful work</b></span> while switching. To minimise downtime due to overhead, context-switch times are highly dependent on hardware support -- some hardware supports rapid context switching by having a <span style="color:#f7007f;"><b>dedicated</b></span> unit for that (effectively bypassing the CPU).

## Mode Switch Versus Context Switch {#mode-switch-versus-context-switch}

Mode switch and Context switch -- although similar in name; they are two separate concepts.
{:.warning}

### Mode switch

- The <span style="color:#f77729;"><b>privilege</b></span> of a process changes
- Simply escalates privilege from user mode to kernel mode to access kernel services.
- Done by either: hardware interrupts, system calls (traps, software interrupt), exception, or reset
- Mode switch <span style="color:#f7007f;"><b>may not always lead to context switch</b></span>. Depending on implementation, Kernel code decides whether or not it is necessary.

### Context switch

- Requires <span style="color:#f77729;"><b>both</b></span> saving (all) states of the old process and loading (all) states of the new process to resume execution
- Can be caused either by timed interrupt or system call that leads to a `yield()`, e.g: when waiting for something.

{:.highlight-title}
> Ask yourself
> 
> Think about scenarios that requires <span style="color:#f77729;"><b>mode switch</b></span> but <span style="color:#f7007f;"><b>not</b></span> context switch.

## Process Scheduling Detail {#process-scheduling}

### Motivation

The objective of <span style="color:#f77729;"><b>multiprogramming</b></span> is to have some process running at all times, to maximize CPU utilization.

The objective of <span style="color:#f77729;"><b>time sharing</b></span> is to switch the CPU among processes so frequently that users can interact with each program while it is running.

To meet <span style="color:#f77729;"><b>both</b></span> objectives, the process scheduler selects an available process (possibly from a set of several available processes) for program execution on the CPU.

- For a single-processor system, there will never be more than one actual running process at any instant.
- If there are more processes, the rest will have to wait in a queue until the CPU is free and can be rescheduled.

For Linux scheduler, see the man page [here](https://man7.org/linux/man-pages/man7/sched.7.html). We can set scheduling policies: `SCHED_RR`, `SCHED_FIFO`, etc. We can also set <span style="color:#f77729;"><b>priority</b></span> value and `nice` value of a process (the latter used to control the priority value of a process in user space[^5].

### Process Scheduling Queues

There are three <span style="color:#f77729;"><b>queues</b></span> that are maintained by the process <span style="color:#f77729;"><b>scheduler</b></span>:

1. <span style="color:#f7007f;"><b>Job</b></span> queue – set of all processes in the system (can be in both main memory and swap space of disk)
2. <span style="color:#f7007f;"><b>Ready</b></span> queue – set of all processes residing in main memory, ready and waiting to execute (queueing for CPU)
3. <span style="color:#f7007f;"><b>Device</b></span> queues – set of processes waiting for an I/O device (one queue for each device)

Each queue contains the <span style="color:#f7007f;"><b>pointer</b></span> to the corresponding PCBs that are waiting for the <span style="color:#f7007f;"><b>resource</b></span> (CPU, or I/O).

#### Example

The diagram below shows a system with ONE ready queue, and FOUR device queues (_image screenshot from SGG book_):

<img src="{{ site.baseurl }}/assets/images/week3/5.png"  class="center_seventy"/>

#### Queueing Diagram

A common representation of process scheduling is using a queueing diagram as shown below:

<img src="{{ site.baseurl }}/assets/images/week3/6.png"  class="center_seventy"/>

A note on the diagram's symbols:
- <span style="color:#f77729;"><b>Rectangular</b></span> boxes represent queues,
- <span style="color:#f77729;"><b>Circles</b></span> represent resources serving the queue
- All types of queue: job queue, ready queue and a set of device queues (I/O queue, I/O Req, time exp, fork queue, and wait irq)

A new process is initially put in the <span style="color:#f77729;"><b>ready</b></span> queue. It waits there until it is selected for execution, or dispatched. Once the process is allocated the CPU, a few things <span style="color:#f77729;"><b>might</b></span> happen afterwards (that causes the process to leave the CPU):

- If the process issue an I/O request, it will be placed onto the <span style="color:#f77729;"><b>I/O queue</b></span>
- If the process `forks` (<span style="color:#f77729;"><b>create</b></span> new process), it may queue (wait) until the child process finishes (terminated)
- The process could be forcily removed from the CPU (e.g: because of an interrupt), and be put back in the <span style="color:#f77729;"><b>ready</b></span> queue.

{:.info}
Process scheduler is typically divided into two parts: long term and short term. Refer to the [appendix](#long-and-short-term-scheduler) section if you're interested to find out more. 


# Operations on Processes
We can perform various <span style="color:#f77729;"><b>operations</b></span> on a process: spawning child processes, terminate the process, set up inter-process communication channels, change the process priority, and many more. All of these operations require a system call (switching to Kernel Mode). In this example, we use the <span style="color:#f77729;"><b>C API</b></span> to make the system call.

# Process Creation {#process-creation}

We can create new processes using <code>fork()</code> system call.

1. The process creator is called a <span style="color:#f7007f;"><b>parent</b></span> process, the new processes are called the <span style="color:#f7007f;"><b>children</b></span> of that process.
2. Each of these new processes may in turn create more child processes, forming a <span style="color:#f7007f;"><b>tree</b></span> of processes.

## Process Tree

We can illustrate multiple process creation as a <span style="color:#f77729;"><b>process tree</b></span>:

<img src="{{ site.baseurl }}/assets/images/week3/8.png"  class="center_thirty"/>

In the example above, there are 5 processes in total. Process 2, 3, and 4 are direct <span style="color:#f77729;"><b>children</b></span> of Process 1. Process 5 is created by Process 2.

### Process id

Each process is identified by an integer called the process id (`pid`). Pid is <span style="color:#f7007f;"><b>unique</b></span> in the system.
{:.warning}

You can type the command `ps [options]` to observe all running processes in your system, along with the `pid` of each process. For instance,

<img src="{{ site.baseurl }}/assets/images/week3/9.png"  class="center_fifty no-invert"/>

### Child Process vs Parent Process

The <span style="color:#f77729;"><b>new</b></span> process consists of the entire copy of the address space (code, stack, process of execution, etc) of the <span style="color:#f77729;"><b>original</b></span> parent process at the point of `fork()`.

The child process inherits the parent process' state <span style="color:#f7007f;"><b>at the point of</b></span> `fork()`.
{: .important}

Parent and child processes operate in <span style="color:#f77729;"><b>different address space</b></span> (isolation). Since they are different processes, parent and children processes execute <span style="color:#f77729;"><b>concurrently</b></span>.

Practically, a parent process <span style="color:#f77729;"><b>waits</b></span> for its children to terminate (using `wait()` system call) to read the child process’ exit status and <span style="color:#f77729;"><b>only then</b></span> its PCB entry in the process table can be removed.

Child processes <span style="color:#f77729;"><b>is unable</b></span> `wait` for their parents to terminate (there's no system call for that in UNIX systems). Since children processes are a duplicate of their parents (inherits the whole address space), they can either

1.  <span style="color:#f77729;"><b>Execute</b></span> the same instructions as their parents concurrently, or
2.  <span style="color:#f77729;"><b>Load</b></span> a new program into its address space

## How fork works {#code-how-fork-works}

It is best to explain how `fork()` process creation works by example.

```java
#include <sys/wait.h>
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>

int main(int argc, char const *argv[])
{
   pid_t pid;

   pid = fork();
   printf("pid: %d\n", pid);

   if (pid < 0)
   {
       fprintf(stderr, "Fork has failed. Exiting now");
       return 1; // exit error
   }
   else if (pid == 0)
   {
       execlp("/bin/ls", "ls", NULL);
   }
   else
   {
       wait(NULL);
       printf("Child has exited.\n");
   }
   return 0;
}
```

The simple C program above is executed and when the execution system call `fork()` returns, <span style="color:#f77729;"><b>two processes are present</b></span>.

<img src="{{ site.baseurl }}/assets/images/week3/10.png"  class="center_seventy no-invert"/>

Both have the <span style="color:#f77729;"><b>same</b></span> copy of the text (code) and resources (any opened files, etc). The parent process is <span style="color:#f77729;"><b>cloned</b></span>, resulting in the child process. They're at a <span style="color:#f77729;"><b>different</b></span> address space, executed concurrently by the system.

### `fork` return value

<code>fork()</code>returns 0 in the child process while in the parent process it returns the pid of the child (>0).
{:.warning}

We can write just <span style="color:#f77729;"><b>one instruction</b></span> for <span style="color:#f77729;"><b>both parent and child process</b></span> but each will take a different <span style="color:#f77729;"><b>branch</b></span> of the `if` clause. In the example code above:

- The child executes the line if-clause: `execlp`
- The parent process executes the `else` clause where it `wait` for the child process to `exit`

<img src="{{ site.baseurl }}/assets/images/week3/11.png"  class="center_seventy"/>

### `execlp`

`execlp` is a system call that loads a new program called `ls` onto the child process’ address space, effectively <span style="color:#f7007f;"><b>replacing</b></span> its text (code), data, and stack content.

{:.note}
`execlp` belongs to the `exec` system call family. Give this [appendix](#exec-system-call) a read to find out more. 

### `wait`

Concurrently, the parent process executes `wait(NULL)`, which is a system call that <span style="color:#f7007f;"><b>suspends</b></span> the parents’ execution until this child process that is executing `ls `has returned.

### Another example

Run this program to further understand how `fork` works: 

```c
#include <sys/wait.h>
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

int main(int argc, char const *argv[])
{
   pid_t pid;

   pid = fork();
   printf("Return value of fork stored in variable pid is: %d\n", pid);

   if (pid < 0)
   {
       fprintf(stderr, "Fork has failed. Exiting now");
       return 1; // exit error
   }
   else if (pid == 0)
   {
    // child
    printf("This is child process with pid %d\n", getpid());
    exit(0);
   }
   else
   {
    // parent
    wait(NULL); // wait for any child to return
    printf("This is parent process with pid %d\n", getpid());
   }
   return 0;
}
```

Carefully observe the output and do **not** confuse between variable `pid` to store the **return value** of `fork` vs **actual process id** returned by `getpid`.
{:.warning}

**Expected Output:**
* You will see the `PID` returned by `fork()` stored in the **variable** `pid` printed by the parent process. It will be a positive number.
* The child process will print its message first (or second, depending on the scheduler) showing that it is the child and its `PID`.
* The parent process will print its message after the child process has exited because it waits for the child to terminate with wait(NULL);. It will show that it is the parent and its `PID`.

## Program: The fork tree {#code-the-fork-tree}

<span style="color:#f77729;"><b>Compile</b></span> and <span style="color:#f77729;"><b>run</b></span> the C program below.

{:.new-title}
> Compute
> 
> How many processes are created in total? (excluding the parent process). Can you draw the process tree?

```cpp
#include <sys/wait.h>
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>

int main(int argc, char const *argv[])
{
   int level = 3;
   pid_t pid[level];

   for (int i = 0; i < level; i++)
   {
       pid[i] = fork();

       if (pid[i] < 0)
       {
           fprintf(stderr, "Fork has failed. Exiting now");
           return 1; // exit error
       }
       else if (pid[i] == 0)
       {
           printf("Hello from child %d \n", i);
       }
   }

   return 0;
}
```

The following shows the fork-tree resulted from running the above program: 

<img src="{{ site.baseurl }}/docs/OS/images/cse2024.drawio.png"  class="center_fifty"/>

## Process Termination {#process-termination}

A process needs certain resources (CPU time, memory, files, I/O devices) to run and accomplish its task. These resources are <span style="color:#f77729;"><b>limited</b></span>.

A process can terminate <span style="color:#f7007f;"><b>itself</b></span> using <code>exit()</code> system call. A process can also terminate <span style="color:#f77729;"><b>other processes</b></span> using the `kill(pid, SIGKILL)` system call.
{:.warning}

Once a process is terminated, these resources are <span style="color:#f77729;"><b>freed</b></span> by the kernel for <span style="color:#f77729;"><b>other</b></span> processes. Parent processes may <span style="color:#f7007f;"><b>terminate</b></span> or <span style="color:#f7007f;"><b>abort</b></span> its children as it knows the `pid` of its children.

### Orphaned processes

If a parent process with live children is terminated, the children processes become <span style="color:#f7007f;"><b>orphaned</b></span> processes:

- Some operating system is designed to either <span style="color:#f7007f;"><b>abort</b></span> all of its orphaned children (cascading termination) or
- <span style="color:#f7007f;"><b>Adopt</b></span> the orphaned children processes (`init` process usually will adopt orphaned processes)

#### About `init`

In UNIX-like OS, `init` is the first process started by the kernel during booting of the computer system. `Init` is a <span style="color:#f7007f;"><b>daemon</b></span> process that continues running until the system is shut down. It is the direct or indirect <span style="color:#f7007f;"><b>ancestor</b></span> of all other processes, and automatically adopts all orphaned processes.

`Init` is a <span style="color:#f7007f;"><b>user</b></span> process like any other processes, and hence it is using virtual memory. The only special thing about `init` is that it is one of the two processes that the kernel started initially. When `init` is started by the kernel, it goes into user mode. When `init` calls system call `fork(),` it traps into the kernel mode, and the kernel does certain things to _create the new process,_ and the new process _will be scheduled in the future._ When the `fork() `returns, the original process is back to user mode. The equivalent of `init` in macOS is [`launchd`](https://en.wikipedia.org/wiki/Launchd).

### Zombie Processes {#zombie-processes}

<span style="color:#f7007f;"><b>Zombie</b></span> processes are processes that are <span style="color:#f7007f;"><b>ALREADY TERMINATED</b></span>, and memory as well as other resources are <span style="color:#f7007f;"><b>freed</b></span>, but its exit status is not read by their parents, hence its <span style="color:#f7007f;"><b>entry</b></span> (PCB) in the process table remains.
{:.warning}

A parent process <span style="color:#f7007f;"><b>must</b></span> call `wait` or `waitpid` to read their children’s exit status. A call to `wait` or `waitpid` <span style="color:#f7007f;"><b>blocks</b></span> the calling process until one of its child processes exits or a signal is received. <span style="color:#f7007f;"><b>Otherwise, their child process becomes a zombie process. </b></span>

Children processes can <span style="color:#f77729;"><b>terminate</b></span> themselves after they have finished executing their tasks using `exit(int status)` system call.
1. The kernel will <span style="color:#f77729;"><b>free</b></span> the memory and other resources from this process, <span style="color:#f7007f;"><b>but not the PCB entry</b></span>.
2. Parent processes are supposed to call `wait` or `waitpid` to obtain the exit status of a child.
3. Only after `wait` or `waitpid` in the parent process returns, the kernel can <span style="color:#f7007f;"><b>remove</b></span> the child PCB entry from the system wide process table.
4. If the parents didn’t call `wait` or `waitpid` and instead continue execution of other things, then children’s entry in the pcb remains; <span style="color:#f7007f;"><b>a zombie process remains</b></span>.

**A zombie process generally takes up very little memory space**, but `pid` of the child remains. Recall that pid is <span style="color:#f7007f;"><b>unique</b></span>, so for example in a 32-bit system, there're only 32768 available pids (thats what set by the OS). Well, actually it is 32767, because pid `0` and `1` are reserved, so user processes get 2 to 32767. On 64-bit systems, the maximum PID is `2^22`.

{:.info}
You can find the maximum PID value for your system in Linux using `cat /proc/sys/kernel/pid_max`:

<img src="{{ site.baseurl }}//assets/images/week3-2_operations/2023-06-04-15-02-06.png"  class="center_seventy no-invert"/>

Having too many zombie processes might result in inability to create new processes in the system, simply because we may run out of pid.
{:.warning}


All processes transition to this zombie state when they terminate, but generally they exist as zombies only <span style="color:#f7007f;"><b>briefly</b></span>. Once the parent calls <code>wait, waitpid</code> the `pid` of the zombie process and its entry in the process table are released.

If a parent process has died, then all the zombie children will be <span style="color:#f7007f;"><b>cleared</b></span> by the kernel. To <span style="color:#f7007f;"><b>observe</b></span> zombie children, you need to artifically suspend the parent process after the children have terminated.
{: .note}

## Program: Zombie making {#code-zombie-making}

<span style="color:#f77729;"><b>Compile</b></span> and <span style="color:#f77729;"><b>run</b></span> the C program below. It will suspend itself at `scanf`, waiting for input at `stdin`. Do not type anything, leave it hanging there.

{:.highlight-title}
> Question
> 
> What’s the (possible) maximum number of zombies created by this process?

```cpp
#include <sys/wait.h>
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>

int main(int argc, char const *argv[])
{
    int level = 5;
    pid_t pid[level];

    for (int i = 0; i < level; i++)
    {
        pid[i] = fork();

        if (pid[i] < 0)
        {
            fprintf(stderr, "Fork has failed. Exiting now");
            return 1; // exit error
        }
        else if (pid[i] == 0)
        {
            printf("Hello from child %d \n", i);
            return 0;
        }
    }

    int testInteger;
    printf("Enter an integer: "); // artificially blocking parent
    scanf("%d", &testInteger);

    return 0;
}
```

We can enter the `ps aux | grep 'Z'` command to list all zombie processes in the system caused by running the program above.
<img src="{{ site.baseurl }}/assets/images/week3/12.png"  class="center_seventy no-invert"/>

<hr>
# Appendix

## Long and Short Term Scheduler

In operating systems, the management of processes and their execution by the CPU is handled by various types of schedulers. Among these, there exist the Long-term and Short-term schedulers. They manage each process scheduling queue accordingly as shown:

<img src="{{ site.baseurl }}/assets/images/week3/7.png"  class="center_seventy"/>

### Long-term Scheduler (or Job Scheduler)

The long-term scheduler is responsible for controlling the degree of multiprogramming, which is the number of processes in memory. This scheduler selects processes from the pool of submitted jobs and loads them into memory for execution. Its main objectives are:

- **Load Balancing:** Ensuring that enough processes are active in the system to make efficient use of CPU and I/O resources without overloading the system.
- **Process Mix:** Managing a good mix of CPU-bound and I/O-bound processes. This diversity helps in optimal utilization of system resources.

The long-term scheduler can decide to limit the number of jobs admitted to the system to avoid excessive competition for resources, which might lead to inefficiency.

### Short-term Scheduler (or CPU Scheduler)

The short-term scheduler, on the other hand, is much more frequent in its operations. It decides which of the processes in the ready queue will be executed next by the CPU. The main attributes of the short-term scheduler are:

- **Speed:** It operates very frequently and must make decisions rapidly to ensure efficient CPU utilization.
- **Decision Making:** It uses algorithms like Round Robin, First-Come-First-Serve, Priority Scheduling, and others to decide the order of process execution.

The short-term scheduler is crucial for achieving efficient time-sharing and is involved in the fundamental operation of the CPU.

### Differences

The primary difference between the long-term and short-term schedulers lies in their frequency of execution and their impact on system performance. The long-term scheduler has a significant impact on the overall degree of multiprogramming but runs less frequently, whereas the short-term scheduler makes rapid decisions and directly affects the performance of individual processes.

### Medium-term Scheduler

Additionally, some systems use a medium-term scheduler to enhance performance. This scheduler temporarily removes processes from active contention for the CPU (usually by swapping them out) and reduces the degree of multiprogramming. When conditions are deemed favorable, the medium-term scheduler can bring these processes back into main memory. This helps in balancing the load and managing the system resources more effectively.

Together, these schedulers manage system resources dynamically to optimize the performance of the operating system and the applications running on it.

# Appendix

## Exec System Call
The `exec` family of functions in Unix-like operating systems is used to replace the current process image with a new process image. The functions `execlp`, `execvp`, and `execv` are variants of this family, each serving slightly different purposes based on how they handle arguments and the environment. Here's a breakdown of their differences:

1. **`execlp`**:
   - **Function Prototype**: `int execlp(const char *file, const char *arg, ..., NULL);`
   - **Behavior**: Searches for the executable named `file` in the directories listed in the environment's PATH variable. It does not require the full path to the executable if it's located in one of these directories.
   - **Arguments**: Takes the name of the program to execute as the first argument, followed by a NULL-terminated list of character strings that represent the arguments to the program. This list must be terminated by a `NULL` pointer.
   - **Use Case**: Useful when the number of arguments is known at compile time and the program to execute is in the system's path.

2. **`execvp`**:
   - **Function Prototype**: `int execvp(const char *file, char *const argv[]);`
   - **Behavior**: Similar to `execlp` in that it searches for the executable in the PATH. However, instead of taking a variable number of arguments, it accepts an array of pointers to null-terminated strings that represent the arguments to the program.
   - **Arguments**: Takes the name of the program and an array of strings as arguments. The array of strings must be NULL-terminated.
   - **Use Case**: Useful when the number of arguments is not known at compile time, and the program to execute is in the system's path.

3. **`execv`**:
   - **Function Prototype**: `int execv(const char *path, char *const argv[]);`
   - **Behavior**: Executes the program pointed to by `path`. Unlike `execlp` and `execvp`, `execv` does not search the PATH. You need to specify the full path of the executable.
   - **Arguments**: Takes the full path of the program and an array of strings as arguments. The array of strings must be NULL-terminated.
   - **Use Case**: Useful when the program's location is known and fixed, and you have an array of arguments to pass to the executable.

Each of these functions does not return to the calling process upon successful execution because the calling process's image is completely replaced by the new program. If there's an error (e.g., the executable is not found), the function returns -1.

[^1]: Because each process is isolated from one another and runs in different address space (forming virtual machines)
[^2]: Heap and stack grows in the opposite direction so that it maximises the space that both can have and minimises the chances of overlapping, since we do not know how much they can dynamically grow during runtime. If the heap / stack grows too much during runtime, we are faced with stack/heap overflow error. If there’s a heap overflow, `malloc` will return a `NULL` pointer.
[^3]: You can list all processes that are currently running in your UNIX-based system by typing `ps aux` in the command line.
[^4]: Image referenced from IBM RedBooks – Linux Performance and Tuning Guidelines.
[^5]: The relation between nice value and priority is: Priority_value = Nice_value + 20
