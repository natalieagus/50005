---
layout: default
permalink: /os/threads
title: Threads 
description: Details about threads as lightweight processes
parent: Operating System
nav_order: 7
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

# Threads

{:.highlight-title}
> Detailed Learning Objectives
>
> - **Understanding Threads**
>   - Define a thread as a segment of a process, and recognize that every process has at least one thread.
>   - Identify the components of a thread: thread ID, program counter, register set, and stack.
> - **Concurrency and Resource Sharing**
>   - Learn how threads within the same process share resources like code, data sections, and system resources.
>   - Understand that threads can execute the same or different parts of the process code concurrently.
> - **Multithreading Benefits**
>   - Explore how multithreading can improve the responsiveness of applications by allowing continued operation even if part of the application is blocked.
>   - Discuss the efficiency of resource sharing among threads compared to processes, highlighting the reduced overhead in creating and switching contexts between threads.
> - **Multithreading vs. Multiprocessing**
>   - Compare the concurrency and protection aspects of processes versus threads.
>   - Analyze the communication, parallel execution, and synchronization differences between using multiple processes and multiple threads.
> - **Types of Threads and Their Management**
>   - Differentiate between user-level threads and kernel-level threads.
>   - Examine how each type of thread is scheduled, the level of overhead involved, and the implications for parallel execution on multicore systems.
> - **Practical Implementation of Threads**
>   - Implement threads in Java using Runnable interfaces and by extending the Thread class.
>   - Use the pthread library in C to create, manage, and synchronize threads.
> - **Advanced Concepts in Threading**
>   - Understand thread mapping models like many-to-one, one-to-one, and many-to-many, and discuss their advantages and disadvantages.
>   - Learn about Intel’s Hyper-Threading technology and its impact on performance and parallel computing.
> - **Thread Safety and Synchronization**
>   - Address the challenges of thread safety and the need for synchronization to avoid race conditions.
>   - Implement basic synchronization mechanisms to manage access to shared resources between threads.
> - **Evaluating Performance in Multithreaded Applications**
>   - Apply Amdahl's Law to determine the potential speedup from parallelizing an application on a multicore system.
>   - Consider the fraction of the application that can be parallelized and the diminishing returns of adding more processing cores.
> 
> These objectives are structured to provide a comprehensive overview of multithreading, highlighting its advantages, challenges, and practical applications in programming.

Threads are defined as a <span style="color:#f77729;"><b>segment</b></span> of a process. A process has <span style="color:#f7007f;"><b>at least</b></span> one thread.
{:.note}

A process can have <span style="color:#f77729;"><b>multiple</b></span> threads, and we can define thread as a basic unit of CPU utilization; it comprises of:

- A <span style="color:#f7007f;"><b>thread ID</b></span>,
- A <span style="color:#f7007f;"><b>program counter</b></span>,
- A <span style="color:#f7007f;"><b>register</b></span> set, and
- A <span style="color:#f7007f;"><b>stack</b></span>.

A thread can execute <span style="color:#f7007f;"><b>any</b></span> part of the process code, including parts currently being executed by another thread.

The figure below _(taken from SGG book)_ shows the illustration of a single-threaded and multi threaded processes. As shown, it <span style="color:#f7007f;"><b>shares</b></span> with other threads belonging to the same process its code section, data section, and other operating-system resources, such as open files and signals. Multiple threads in the same process operates in the <span style="color:#f77729;"><b>same address space</b></span>.

<img src="{{ site.baseurl }}/assets/images/week3/17.png"  class="center_seventy"/>

# Multithreading Benefits

## Responsiveness

Threads make the program seem more <span style="color:#f7007f;"><b>responsive</b></span>. Multithreading an interactive application may allow a program to continue running even if <span style="color:#f7007f;"><b>part</b></span> of it is <span style="color:#f7007f;"><b>blocked</b></span> or is performing a lengthy operation, thereby increasing <span style="color:#f7007f;"><b>responsiveness</b></span> to the user.
{:.important}

For example, a multithreaded <span style="color:#f7007f;"><b>server architecture</b></span> below _(taken from SGG book)_ allows the server to remain responsive in accepting new clients while servicing existing clients' requests:
<img src="{{ site.baseurl }}/assets/images/week3/18.png"  class="center_seventy"/>

Multithreading us to reap the benefits of true <span style="color:#f7007f;"><b>parallel</b></span> programming in <span style="color:#f7007f;"><b>multiprocessor</b></span> architecture, where threads may be running in <span style="color:#f7007f;"><b>parallel</b></span> on different processing cores.

## Easy Resource Sharing and Communication

Easier means for resource sharing and communication (since code, data, and files are shared among threads of the same process and they have the <span style="color:#f7007f;"><b>same</b></span> address space), as compared to creating multiple processes. On the other hand, multiple processes can only share resources through techniques such as shared memory and message passing.

## Threads are Lighter

Creating new threads requires <span style="color:#f7007f;"><b>cheaper</b></span> resources than creating new processes. Performing thread context switch is also a <span style="color:#f77729;"><b>cheaper</b></span> process.

- Allocating memory and resources for process creation is <span style="color:#f7007f;"><b>costly</b></span>.
- Because threads <span style="color:#f7007f;"><b>share</b></span> the resources of the process to which they belong, it is more economical to <span style="color:#f7007f;"><b>create</b></span> and <span style="color:#f7007f;"><b>context-switch</b></span> threads.

Remember, context switch is <span style="color:#f77729;"><b>pure</b></span> overhead.
{:.warning}

# Multithreading vs Multiprocessing


## Protection and Concurrency

<span style="color:#f77729;"><b>Processes</b></span> have both <span class="orange-bold">concurrency and protection</span>. One process is isolated from the other, hence providing fault isolation.

<span style="color:#f77729;"><b>Threads</b></span> only have concurrency but <span class="orange-bold">not</span> protection since code, data, and files are shared between threads of the same process. There’s no fault isolation.

## Communication

<span style="color:#f77729;"><b>IPC</b></span> is <span class="orange-bold">expensive</span>, requires system calls and context switch between processes has high overhead.

<span style="color:#f77729;"><b>Thread</b></span> communication has <span class="orange-bold">low overhead</span> since they share the same address space. Context switching (basically thread switching) between threads is much cheaper.

## Parallel Execution

Parallel process execution is always available on multicore system.

However, parallel thread execution is <span style="color:#f77729;"><b>not</b></span> always available. It depends on the [types of threads](#thread-types).

## Synchronisation

We can rely on OS services (<span class="orange-bold">system calls</span>) to synchronise execution between <span style="color:#f77729;"><b>processes</b></span>, i.e: using semaphores

However, <span class="orange-bold">careful programming</span> is required (burden on the developer) to synchronise between <span style="color:#f77729;"><b>threads</b></span>.

## Overhead cost

<span style="color:#f77729;"><b>Processes</b></span> are entirely managed by Kernel Scheduler:

- To switch between process executions, <span style="color:#f77729;"><b>full</b></span> (<span class="orange-bold">costly</span>) context switch is required + <span style="color:#f7007f;"><b>system call</b></span>
- The context of a process is large, including flushing the cache, managing the MMU since each process lives in different virtual space

<span style="color:#f77729;"><b>Threads</b></span> are managed by Thread Scheduler (user space, depending on the language):

- To switch between thread executions, lighter context switch is sufficient
- The context of a thread is lighter, since threads live in the same virtual space. Contents that need to be switched involve mainly only registers and stack.
- <span style="color:#f7007f;"><b>May or may not need to perform system call</b></span> to access Thread API, depending on the types of thread

# Thread Examples: Java and C pthread {#thread-examples-java-and-pthread}

## Java Thread {#java-thread}

Java threads are managed by the JVM. You can create Java threads in two ways.

### Runnable Interface

The first way is by implementing a <span style="color:#f77729;"><b>runnable interface</b></span> and call it using a thread:

1. Implement a runnable interface and its `run()` function:

   ```cpp
   public class MyRunnable implements Runnable {
       public void run(){
          System.out.println("MyRunnable running");
       }
     }
   ```

2. Create a new Thread instance and pass the runnable onto its constructor. Call the `start() `function to begin the execution of the thread:

   ```cpp
   Runnable runnable = new MyRunnable();
   Thread thread = new Thread(runnable);
   thread.start();

   ```

### Thread subclass

The second way is to create a <span style="color:#f77729;"><b>subclass</b></span> of `Thread` and override the method `run():`

1. For example,

   ```cpp
   public class CustomThread extends Thread {
     public void run(){
        System.out.println("CustomThread running");
     }
   }
   ```

2. Create and start CustomThread instance to start the thread:

   ```cpp
   CustomThread ct = new CustomThread();
   ct.start();

   ```

## C Thread {#c-thread}

We can also equivalently create threads in C using the `pthread` library. The code below shows how we can create threads in C:

1. Create a <span style="color:#f77729;"><b>function</b></span> with void\* return type (generic return type)
2. Use `pthread_create` to <span style="color:#f77729;"><b>create</b></span> a thread that executes this function
3. Use `pthread_join` to <span style="color:#f77729;"><b>wait</b></span> for a thread to finish

### Program: pthread creation {#code-c-threading}

We can pass <span style="color:#f77729;"><b>arguments</b></span> to a thread, <span style="color:#f77729;"><b>wait</b></span> for its completion and get its <span style="color:#f77729;"><b>return</b></span> value.

Threads from the same process share heap and data. A global variable like `shared_integer` in the example below is <span style="color:#f77729;"><b>visible</b></span> to both main thread and created thread.

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>

int shared_integer; // instantiate integer in fixed memory

void *customThreadFunc(void *vargp)
{
   sleep(1);
   char argument = *(char *)vargp;
   printf("Printing helloWorld from spawned thread with argument: %c\n", argument);
   shared_integer++;

   char answer = 'a'; // in stack
   char *answer_heap = malloc(sizeof(char)); // in heap

   *answer_heap = 'a';
   pthread_exit((void *)answer_heap);

   // pthread_exit((void *) &answer); // will result in segfault
}

int main()
{
   shared_integer = 10;
   pthread_t thread_id;

   char input_argument = 'b'; // to pass to thread
   char *result;              // to hold return value

   // pthread_create(pthread_t* thread_id, NULLABLE thread attributes, void* function_for_thread, NULLABLE void* argument)
   pthread_create(&thread_id, NULL, customThreadFunc, &input_argument);

   sleep(1);
   printf("shared_integer: %d\n", shared_integer); // potential race condition

   // pthread_join(pthread_t thread_id, void return_value);
   // Second argument of type void : pass the address of "result", which is a pointer to the supposed return value of the thread

   // blocking
   pthread_join(thread_id, (void * )&result);
   printf("After thread join...\n");

   printf("shared_integer: %d\n", shared_integer);
   printf("result: %c\n", *result);

   exit(0);
}
```

### Return value

In the example above, we created `answer_heap` using `malloc`, therefore this answer resides in the `heap` (shared with all threads in the entire process, persists for as long as the process is not terminated).

However, if we <span style="color:#f77729;"><b>return</b></span> a pointer to variable that’s initialized in the thread’s <span style="color:#f77729;"><b>stack</b></span>, then you will end up with a <span style="color:#f7007f;"><b>segfault error</b></span>.

# Types of Threads {#thread-types}

We can create two types of threads, <span style="color:#f77729;"><b>kernel</b></span> level threads and <span style="color:#f77729;"><b>user</b></span> level threads.

## User level threads

User level threads (sometimes known as green threads) are threads that are:

1. Scheduled by a <span style="color:#f77729;"><b>runtime library</b></span> or virtual environment instead of by the Kernel.
2. They are <span style="color:#f77729;"><b>managed in user space</b></span> instead of kernel space, enabling them to work in OS that do not have native thread support.
3. <span style="color:#f7007f;"><b>This is what programmers typically use. </b></span> (e.g when we create Thread in C or Java above).

## Kernel level threads

To augment the need for running background operations, the kernel spawns **threads**, similar to regular processes in that they are represented by a `task_struct` and assigned a PID. Unlike user processes, they do not have any address space mapped, and run exclusively in kernel mode, which makes them non-interactive.

Kernel level threads (sometimes known as OS-level threads) are threads that are:

1. Scheduled by the Kernel, having its own stack and register values, but share data with other Kernel threads.
2. Can be viewed as <span style="color:#f77729;"><b>lightweight processes</b></span>, which perform a certain task asynchronously.
3. A kernel-level thread need not be associated with a process; a kernel can create them whenever it needs to perform a particular task.
4. Kernel threads cannot execute in user mode.
   - Processes that create kernel-level threads use it to <span style="color:#f7007f;"><b>implement background tasks in the kernel</b></span>.
   - E.g: handling <span style="color:#f7007f;"><b>asynchronous</b></span> events or <span style="color:#f7007f;"><b>waiting</b></span> for an event to occur.

All modern operating systems support kernel level threads, allowing the kernel to perform multiple simultaneous tasks and/or to <span style="color:#f77729;"><b>service</b></span> multiple kernel system calls simultaneously.

Most programming languages will provide an <span style="color:#f77729;"><b>interface</b></span> to create kernel-level threads, and the API for kernel-level threads are system-dependent. The C-API for creating kernel threads in Linux:

```cpp
#include <kthread.h>
kthread_create(int (*function)(void *data), void *data, const char name[], ...)
```


All kernel threads are descendants of `kthreadd` (pid 2), which is spawned by the kernel (pid 0) during boot. The `kthreadd` is a perpetually running thread that looks into a list called `kthread_create_list` for data on new kthreads to be created.
{: .new}

You can view it via `ps -ef` command, they are shown within square brackets `[]`:

<img src="{{ site.baseurl }}//assets/images/week3-4_threads/2023-06-04-11-49-39.png"  class="center-full no-invert"/>

They each have a specific task, for instance `[migration/0]` handles distributes processes across cores, `[cpuhp/0]` supports physically adding and removing CPUs from the system, etc. You can read some info about other kernel threads and what [they might do from here](https://everylinuxprocess.com). The are alot of kernel threads that are not well documented, just like your 1D assignments, so don't take it too hard.

## Use case

Many device drivers utilize the services of kernel threads to implement **assistant** or **helper** tasks. Suppose you would like the Kernel to asynchronously invoke a **user** mode program to send you an email alert whenever it senses that the health of certain key kernel data structures is deteriorating. For instance, free space in network receive buffers has dipped below a certain healthy threshold, risking losing incoming packets.

This need warrants a creation of Kernel Thread because:

- It's a background task because it has to wait for asynchronous events (depending on the network buffer state).
- It needs access to kernel data structures because the actual detection of events is done by other parts of the kernel.
- It has to invoke a user mode helper program (time consuming, requires resources to complete).

This Kernel thread will _sleep_ until it gets woken up by parts of the Kernel responsible for monitoring the network receive buffers. When awake, it invokes a user mode helper program and passes appropriate identity codes in its environment.

## Kernel vs User Level Threads

| <span style="color:#f7007f;"><b>Kernel</b></span> Level Threads                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | <span style="color:#f77729;"><b>User</b></span> Level Threads                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <span style="color:#f7007f;"><b>Known to OS Kernel</b></span>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | <span style="color:#f77729;"><b>Not known to OS Kernel</b></span>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| <span style="color:#f7007f;"><b>Runs</b></span> in Kernel mode. All code and data structures for the Thread API exist in Kernel space.                                                                                                                                                                                                                                                                                                                                                                                                                                                          | <span style="color:#f7007f;"><b>Runs</b></span> entirely in User mode. All code and data structures for the Thread API exist in user space.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| Invoking a function in the API results in <span style="color:#f7007f;"><b>trap</b></span> (system call) to the Kernel Mode (requires a change of mode)                                                                                                                                                                                                                                                                                                                                                                                                                                          | Invoking a function in the API results in <span style="color:#f7007f;"><b>local</b></span> function call in user space.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| <span style="color:#f7007f;"><b>Scheduled directly by Kernel scheduler</b></span>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | <span style="color:#f77729;"><b>Managed and scheduled entirely by the run time system</b></span> (user level library).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Scheduler maintains one <span style="color:#f7007f;"><b>thread control block</b></span> per thread in the system in a system wide thread table.<br><br>Scheduler also manages <span style="color:#f7007f;"><b>one process control block</b></span> per process in the system in a system wide process table.<br><br>From the OS's point of view, these are what is scheduled to run on a CPU. Kernel <span style="color:#f7007f;"><b>must</b></span> manage the scheduling of both <span style="color:#f7007f;"><b>threads</b></span> and <span style="color:#f7007f;"><b>processes</b></span>. | The thread scheduler is running in <span style="color:#f77729;"><b>user</b></span> mode.<br>It exists in the <span style="color:#f77729;"><b>thread library </b></span>(e.g: phtread of Java thread) linked with the process.                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| Multiple Kernel threads can run on <span style="color:#f7007f;"><b>different</b></span> processors.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | Multiple user threads on the same user process <span style="color:#f77729;"><b>may not be run</b></span> on multiple cores (since the Kernel isn’t aware of their presence).<br>It depends on the mapping model (see next section).                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| Take up <span style="color:#f77729;"><b>Kernel</b></span> data structure, <span style="color:#f77729;"><b>specific</b></span> to the operating system.                                                                                                                                                                                                                                                                                                                                                                                                                                          | Take up <span style="color:#f77729;"><b>thread</b></span> data structure (depends on the library). More <span style="color:#f77729;"><b>generic</b></span> and can run on any system, even on OS that doesn’t support multithreading                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Requires more <span style="color:#f7007f;"><b>resources</b></span> to allocate / deallocate.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | <span style="color:#f77729;"><b>Cheaper</b></span> to allocate / deallocate.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| Significant <span style="color:#f7007f;"><b>overhead</b></span> and increase in kernel complexity.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | <span style="color:#f77729;"><b>Simple management</b></span>, creating, switching and synchronizing threads done in user-space without kernel intervention.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| Switching threads is as <span style="color:#f7007f;"><b>expensive</b></span> as making a system call.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | <span style="color:#f77729;"><b>Fast and efficient</b></span>, switching threads not much more expensive than a function call                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| Since kernel has full <span style="color:#f7007f;"><b>knowledge</b></span> of all kernel threads, the scheduler may be more efficient.<br><br><span style="color:#f7007f;"><b>Example</b></span>: <br>• Give <span style="color:#f7007f;"><b>more</b></span> CPU time for processes with larger number of threads<br>• <span style="color:#f7007f;"><b>Schedule</b></span> threads that are currently holding a lock.                                                                                                                                                                           | Since <span style="color:#f77729;"><b>OS is not aware</b></span> of user-level threads, Kernel may forcibly make poor decisions.<br><br><span style="color:#f77729;"><b>Example:</b></span><br>• Scheduling a process with <span style="color:#f77729;"><b>idle</b></span> threads<br>• <span style="color:#f77729;"><b>Blocking</b></span> a process due to a blocking thread even though the process has other threads that can run<br>• Giving a process as a whole one time slice <span style="color:#f77729;"><b>irrespective</b></span> of whether the process has 1 or 1000 threads<br>• <span style="color:#f77729;"><b>Unschedule</b></span> a process with a thread holding a lock |

# Thread Mapping {#thread-mapping}

Since there are two types of threads: kernel and user thread, there should exist some kind of mapping between the two. The mapping of user threads to kernel threads is done using virtual processors[^8]. We can think of kernel level threads as <span style="color:#f7007f;"><b>virtual processors</b></span> and user level threads as simply <span style="color:#f77729;"><b>threads</b></span>.

There are three types of mapping.

## Many to One

Maps <span style="color:#f77729;"><b>many</b></span> user-level threads to <span style="color:#f77729;"><b>one</b></span> kernel thread.
{:.note}

<span style="color:#f77729;"><b>Advantage</b></span>:

- Thread management is done by the thread library in user space, so it is more <span style="color:#f77729;"><b>efficient</b></span> as opposed to kernel thread management.
- Developers may create as many threads as they want
- Entire thread context-switching is maintained by the user-level thread library <span style="color:#f77729;"><b>entirely</b></span>

<span style="color:#f77729;"><b>Disadvantage</b></span>:

- <span style="color:#f77729;"><b>The entire process will block </b></span>if a thread makes a blocking system call since kernel isn’t aware of the presence of these user threads
- Since only one thread can access the kernel at a time, multiple threads are unable to run in parallel on multicore systems

<img src="{{ site.baseurl }}/assets/images/week3/19.png"  class="center_fifty "/>

## One to One

Maps <span style="color:#f77729;"><b>each</b></span> user thread to a kernel thread. You can simply think of this as process threads to be <span style="color:#f77729;"><b>managed</b></span> entirely by the Kernel.
{:.note}

<span style="color:#f77729;"><b>Advantage</b></span>:

- Provides <span style="color:#f77729;"><b>more concurrency</b></span> than the many-to-one model by allowing another thread to run when a thread makes a blocking system call.
- Allows <span style="color:#f77729;"><b>multiple</b></span> threads to run in parallel on multiprocessors.

<span style="color:#f77729;"><b>Disadvantage</b></span>:

- Creating a user thread requires creating the corresponding kernel thread (a lot of <span style="color:#f77729;"><b>overhead</b></span>, system call must be made)
- <span style="color:#f77729;"><b>Limited</b></span> amount of threads can be created to not burden the system
- <span style="color:#f77729;"><b>May</b></span> have to involve the kernel when there is a context switch between the threads (overhead)

The modern Linux implementation of pthreads uses a 1:1 mapping between pthread threads and kernel threads, so you will always get a kernel-level thread with `pthread_create().`
<img src="{{ site.baseurl }}/assets/images/week3/20.png"  class="center_fifty "/>

## Many to Many

<span style="color:#f77729;"><b>Multiplexes</b></span> many user-level threads to a smaller or equal number of kernel threads.
{:.note}

This is the best of both worlds:

- Developers can <span style="color:#f77729;"><b>create</b></span> as many user threads as necessary,
- The corresponding kernel threads can run in <span style="color:#f77729;"><b>parallel</b></span> on a multiprocessor.

<img src="{{ site.baseurl }}/assets/images/week3/21.png"  class="center_fifty "/>

A variation of many-to-many mapping, the <span style="color:#f77729;"><b>two-level model</b></span>: both multiplexing user threads and allowing some user threads to be mapped to just one kernel thread.

<img src="{{ site.baseurl }}/assets/images/week3/22.png"  class="center_fifty "/>

In some systems, especially in many-to-many or two-level models, there is some way for the user threading library to communicate (coordinate) with kernel threads via scheduler activation, i.e: a mechanism whereby kernel can allocate more threads to the process <span style="color:#f7007f;"><b>on demand</b></span>.
{:.warning}

# Intel Hyper-Threading {#intel-hyper-threading}

Hyper-threading was Intel's first effort (proprietary method) to bring parallel computation to end user's PCs by fooling the kernel to think that there exist more than 1 processors in a single processor system:

- A <span style="color:#f77729;"><b>single</b></span> CPU with <span style="color:#f7007f;"><b>hyper-threading</b></span> appears as two or more <span style="color:#f7007f;"><b>logical</b></span> CPUs (with all its resources and registers) for the operating system kernel
- This is a process where a [CPU ](https://www.tomshardware.com/reviews/cpu-buying-guide,5643.html)splits each of its physical [cores](https://www.tomshardware.com/news/cpu-core-definition,37658.html) into virtual cores, which are known as threads.
- Hence the kernel assumes two CPUs for each single CPU core, and therefore <span style="color:#f7007f;"><b>increasing the efficiency of one physical core</b></span>.
  - It lets a single CPU to fetch and execute instructions from two memory locations <span style="color:#f7007f;"><b>simultaneously</b></span>

This is <span style="color:#f77729;"><b>possible</b></span> if a big chunk of CPU time is wasted as it waits for data from cache or RAM.

# Multicore Programming {#multicore-programming}

On a system with multiple cores and supported kernel mapping, concurrency means that the threads can run in parallel, because the system can assign a separate thread to each core.
{:.warning}

A possible parallel execution sequence on a multicore system is as shown (Ti stands for Thread i, 4 threads in total in this example):
<img src="{{ site.baseurl }}/assets/images/week3/23.png"  class="center_seventy"/>

This is faster from concurrent execution with a single core as shown:
<img src="{{ site.baseurl }}/assets/images/week3/24.png"  class="center_seventy"/>

There are two types of parallelism that we can achieve via multicore programming: <span style="color:#f77729;"><b>data</b></span> parallelism and <span style="color:#f77729;"><b>task</b></span> parallelism.

<span style="color:#f77729;"><b>Data parallelism </b></span>focuses on distributing subsets of the same data across multiple computing cores and performing the same operation on each core.

<span style="color:#f77729;"><b>Task parallelism</b></span> involves distributing not data but tasks (<span style="color:#f7007f;"><b>threads</b></span>) across multiple computing cores.

- Each thread is performing a unique operation.
- Different threads may be operating on the same data, or they may be operating on different data.

# Amdahl's Law

Not all programs can be 100% parallelised. Some programs required a portion of its instructions to be <span style="color:#f77729;"><b>serialised</b></span>. We need to identify potential performance gains from adding additional computing cores to an application that has both serial (nonparallel) and parallel components.
{:.warning}

Given that $\alpha$ is the <span style="color:#f77729;"><b>fraction</b></span> of the program that must be executed serially, the <span style="color:#f7007f;"><b>maximum speedup</b></span> that we can gain when running this program with $N$ CPU cores (processors) is:

$$\frac{1}{\alpha + \frac{1-\alpha}{N}}$$

{:.highlight-title}
> Ask yourself
> 
> What is the maximum speedup we can gain when $\alpha=30$% and $N$ is $\infty$?

# Summary

This chapter covers the essentials of threads in operating systems, explaining that threads are lighter, more efficient segments of processes designed to execute tasks. It differentiates between user-level and kernel-level threads, discusses their advantages, and outlines their management. The text also explores the benefits of multithreading, such as increased responsiveness and efficient resource sharing, alongside practical implementation in Java and C. 


Key learning points include:
- **Basics of Threads**: Defines threads as segments of a process that include a unique thread ID, program counter, register set, and stack.
- **Multithreading Benefits**: Highlights improvements in responsiveness and resource sharing, contrasting with the heavier resource demands of multiprocessing.
- **Types of Threads**: Differentiates between user-level and kernel-level threads, discussing their scheduling and use cases.
- **Thread Safety and Synchronization**: Addresses the necessity for mechanisms like mutexes and semaphores to prevent race conditions.
- **Performance Considerations**: Applies concepts like Amdahl’s Law to evaluate the benefits of parallel execution on multicore systems.


# Appendix
## Daemon Processes {#appendix-daemon-processes}

A <span style="color:#f77729;"><b>daemon</b></span> is a background process that performs a specific function or system task. It is created in user mode.
{:.warning}

In keeping with the UNIX and Linux philosophy of modularity, daemons are programs rather than parts of the kernel. Many daemons start at boot time and continue to run as long as the system is up. Other daemons are started when needed and run only as long as they are useful.

The daemon process is designed to run in the background, typically managing some kind of ongoing service. A daemon process might listen for an incoming request for access to a service. For example, the `httpd` daemon listens for requests to view web pages. Or a daemon might be intended to initiate activities itself over time. For example, the `crond` daemon is designed to launch cron[^9] jobs at preset times.

When we refer to a daemon process, we are referring to a process with these characteristics:

- Generally <span style="color:#f77729;"><b>persistent</b></span> (though it may spawn temporary helper processes like xinetd does)
- <span style="color:#f77729;"><b>No controlling terminal</b></span> (and the controlling `tty` process group (`tpgid`) is shown as -1 in `ps`)
- <span style="color:#f77729;"><b>Parent</b></span> process is generally init (process 1)
- Generally has its own process group `id` and session `id` (daemon group)


## More on Linux Startup Process {#appendix-a-more-detailed-look-on-linux-startup-process}

The Linux startup process very much depends on the hardware architecture, but it can be simplified into the following steps:

1. <span style="color:#f77729;"><b>Firstly</b></span>, the BIOS performs setup tasks that are specific to the hardware of the system. This is to prepare the hardware for the bootup process. After setup is done successfully, the bios loads the boot code (bootloader) from the boot device (the disk)
2. The bootloader will <span style="color:#f77729;"><b>load</b></span> the default option of the operating system. If there’s more than 1 OS, then the user is presented with an option of selecting an OS to run.
3. After an OS is selected, the bootloader <span style="color:#f77729;"><b>loads the kernel into memory</b></span>, and the CPU starts execution in kernel mode.
4. The kernel will <span style="color:#f77729;"><b>setup</b></span> system functions that are crucial for hardware and memory paging, perform the majority of system setups pertaining to interrupts, memory management, device, and driver initialization.
5. It then start up a bunch of _daemon_ processes:
   - The `idle` process and the `init` process for example, that runs in <span style="color:#f77729;"><b>user space</b></span>
6. The `init` either consists of scripts that can be executed by the shell (sysv, bsd, runit) or configuration files that are executed by the binary components (systemd, upstart).
   - The `init` process bascally invokes a specific set of services (daemons).
   - These provide various <span style="color:#f77729;"><b>background</b></span> system services and structures and form the user <span style="color:#f77729;"><b>environment</b></span>.
7. Desktop environment is eventually launched

## Desktop environment

The desktop environment begins with a daemon, called the <span style="color:#f77729;"><b>display manager</b></span> that starts a GUI environment which consists of:

1. Graphical <span style="color:#f77729;"><b>server</b></span> that provides basic <span style="color:#f77729;"><b>rendering</b></span> and manages input from user
2. <span style="color:#f77729;"><b>Login manager</b></span> that provides the ability to enter credentials and select a session (basically our login page).

After the user has entered the correct credentials, the session manager starts a <span style="color:#f7007f;"><b>session</b></span>.

A session is a set of programs such as UI elements (panels, desktops, applets, etc.) which together can form a complete desktop environment.
{:.warning}

<hr> 

[^7]: Notice the distinction between <strong><em>parallelism </em></strong>and <strong><em>concurrency </em></strong>mentioned in this table. A system is <strong>parallel</strong> if it can perform more than one task simultaneously (in time). In contrast, a concurrent system supports more than one task by allowing all the tasks to make progress. Thus, it is possible to have concurrency without parallelism.
[^8]: A virtual processor (VP) is a library entity that is usually implicit. For a user thread, the VP behaves like a CPU. In the library, the VP is a kernel thread or a structure bound to a kernel thread. (taken from https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/generalprogramming/thread_models_virtual.html)
[^9]: cron is a Linux utility which schedules a command or script on your server to run automatically at a specified time and date. A cron job is the scheduled task itself.
