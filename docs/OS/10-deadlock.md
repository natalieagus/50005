---
layout: default
permalink: /os/deadlock
title: Deadlock 
description: Cause and solution to deadlock 
parent: Operating System
nav_order: 10
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

# Deadlock

{:.highlight-title}
> Detailed Learning Objectives
>
> - **Understand Finite System Resources**
>   - Describe the concept of finite resources within a computing system.
>   - Identify different types of system resources and understand their management.
> - **Resource Management**
>   - Explain the process of requesting, using, and releasing resources.
>   - Distinguish between kernel-managed and user-managed resources.
>   - Understand the mechanisms like semaphores and mutexes used for managing user resources.
> - **Recognize and Define Deadlock**
>   - Define deadlock and understand its implications in systems with finite resources.
>   - Identify the conditions that lead to deadlock.
> - **Deadlock Characteristics and Examples**
>   - Explain through examples how deadlocks can occur in systems.
>   - Discuss the four necessary conditions for deadlock: Mutual Exclusion, Hold and Wait, No Preemption, and Circular Wait.
> - **Deadlock Handling Techniques**
>   - Understand different methods of handling deadlocks: prevention, avoidance, detection, and recovery.
>   - Explore specific strategies within each method to manage deadlocks.
> - **Deadlock Prevention Strategies**
>   - Describe strategies to prevent deadlock by breaking one of the four necessary conditions.
>   - Analyze the advantages and limitations of each prevention strategy.
> - **Deadlock Avoidance Algorithms**
>   - Understand the principle and application of the Banker's Algorithm for deadlock avoidance.
>   - Discuss the Safety and Resource Allocation Algorithms as components of the Banker's Algorithm.
> - **Deadlock Detection and Recovery**
>   - Learn how to detect deadlocks using algorithms and system state analysis.
>   - Explore options for recovering from deadlocks, including process termination and resource preemption.
> - **Evaluate Deadlock Algorithms**
>   - Critically assess the efficiency and practicality of algorithms used for deadlock handling.
>   - Discuss the trade-offs involved in choosing different deadlock handling methods.
>
> These learning objectives are designed to provide a comprehensive understanding of how finite resources are managed in computing systems, the challenges posed by deadlocks, and the various strategies employed to prevent, avoid, detect, and recover from deadlocks.


## Finite System Resources

A system consists of a <span style="color:#f7007f;"><b>finite</b></span> number of resources to be distributed among a number of <span style="color:#f7007f;"><b>competing processes</b></span>. Each type of resource can have several finite instances. Some example include:

- CPU cycles / cores
- I/O devices
- Access to files or memory locations (guarded by locks or semaphores, etc)

A process must <span style="color:#f77729;"><b>request</b></span> a resource before using it and must <span style="color:#f77729;"><b>release</b></span> the resource after using it. 
{: .info}

A process may <span style="color:#f77729;"><b>request as many</b></span> resources as it requires to carry out its designated task. Obviously, the number of resources requested should not exceed the total number of resources available in the system.
{:.info}

For instance: a process cannot request three printers if the system has only two. Usually, a process may utilize a resource in the following sequence:
- <span style="color:#f77729;"><b>Request</b></span>: The process requests the resource. If the request cannot be granted immediately (for example, if the resource is being used by another process), then the requesting process must wait until it can acquire the resource.
- <span style="color:#f77729;"><b>Use</b></span>: The process can operate on the resource (for example, if the resource is a printer, the process can print on the printer).
- <span style="color:#f77729;"><b>Release</b></span>: The process releases the resource.

The request and release of resources may require system calls, depending on who manages the resources.
{: .note}

### Kernel Managed Resources

For each use of a <span style="color:#f77729;"><b>kernel</b></span>-managed resource, the operating system checks to make sure processes who requested these resources has been granted allocation of these resources. Examples are the `request()` and `release()` device, `open()` and `close()` file, and `allocate()` and `free()` memory system calls.

> **Some implementation detail**
> 
> A typical OS manages some kind of system table (data structure) that records whether each resource is <span style="color:#f77729;"><b>free</b></span> or <span style="color:#f77729;"><b>allocated</b></span>. For each resource that is allocated, the table also records the process to which it is allocated. If a process requests a resource that is currently allocated to another process, it can be added to a <span style="color:#f7007f;"><b>queue</b></span> of processes waiting for this resource.

Developers who simply write programs needing kernel-managed resources will simply make the necessary system calls and need not care about _how_ Kernel manages these resources. This is called <span class="orange-bold">abstraction</span>. 
{: .note}

### User Managed Resources

User-managed resources are not protected by the Kernel. For each use of <span style="color:#f77729;"><b>user</b></span>-managed resources, developers need to deliberately "reserve" (guard) them using <span style="color:#f77729;"><b>semaphores</b></span>, <span style="color:#f77729;"><b>mutexes</b></span>, or other critical section solutions learned in the previous chapter. 

Recall that the request and release of semaphores can be accomplished through the `wait()` and `signal()` operations on semaphores, or through `acquire()` and `release()` of a mutex lock. Writing these series of `wait()` and `signal()` are solely the decision of the developers writing that program. They have to be very careful in using them or else it might result in a <span class="orange-bold">deadlock</span>.

# The Deadlock Problem

Deadlock is a situation whereby a set of <span style="color:#f7007f;"><b>blocked</b></span> processes (none can make <span style="color:#f7007f;"><b>progress</b></span>) each holding a resource and waiting to acquire a resource held by another process in the set.
{:.info}

The critical section solutions we learned in the previous chapter _prevents_ race condition, but if not implemented properly it can cause <span style="color:#f7007f;"><b>deadlock</b></span>.
{:.warning}

## A Simple Deadlock Example

For example, consider two mutex locks being initialized:

```cpp
pthread_mutex_t first_mutex;
Pthread_mutex_t second_mutex;
pthread_mutex_init(&first_mutex,NULL);
pthread_mutex_init(&second_mutex,NULL);
```

Two threads doing running these instructions <span class="orange-bold">concurrently</span> *may* (not guaranteed) potentially result in a deadlock:

```cpp
// Thread 1 Instructions
pthread_mutex_lock(&first_mutex);
pthread_mutex_lock(&second_mutex);
/**
* Do some work
*/
pthread_mutex_unlock(&second_mutex);
pthread_mutex_unlock(&first_mutex);
pthread_exit(0);
/************************************/


// Thread 2 Instructions
pthread_mutex_lock(&second_mutex);
pthread_mutex_lock(&first_mutex); /**
/**
* Do some work
*/
pthread_mutex_unlock(&first_mutex);
pthread_mutex_unlock(&second_mutex);
pthread_exit(0);
/************************************/
```

{:.highlight-title}
> Question
> 
> Can you identify the order of execution that causes deadlock?

Consider the scenario where Thread 1 acquires `first_mutex` and then suspended, then Thread 2 acquires `second_mutex`.
* <span style="color:#f77729;"><b>Neither thread will give up</b></span> their currently held `mutex`,
* However they need each other’s mutex lock to <span style="color:#f77729;"><b>continue</b></span>,
* Hence <span style="color:#f7007f;"><b>neither made progress</b></span> and they are in a <span style="color:#f7007f;"><b>deadlock</b></span> situation.

## Deadlock Necessary Conditions

Deadlock situation _may_ arise if the following four conditions hold <span class="orange-bold">simultaneously</span> in a system. These are necessary but <span class="orange-bold">not</span> sufficient conditions:

1. <span style="color:#f77729;"><b>Mutual exclusion</b></span>
   - At least one resource must be held in <span style="color:#f77729;"><b>a non-sharable mode</b></span>.
   - If another process requests that resource that's currently been held by others, the requesting process must be <span style="color:#f77729;"><b>delayed</b></span> until the resource has been released.
2. <span style="color:#f77729;"><b>Hold and Wait</b></span>
   - A process must be <span style="color:#f77729;"><b>holding</b></span> at least one resource and <span style="color:#f77729;"><b>waiting</b></span> to acquire additional resources that are currently being held by other processes.
3. <span style="color:#f77729;"><b>No preemption</b></span>[^1]
   - Resources can only be <span style="color:#f77729;"><b>released</b></span> only <span style="color:#f77729;"><b>after</b></span> that process has <span style="color:#f77729;"><b>completed</b></span> its task, <span style="color:#f7007f;"><b>voluntarily</b></span> by the process holding it.
4. <span style="color:#f77729;"><b>Circular Wait</b></span>
   - There exists a cycle in the _resource allocation graph_ (see next section)

"**Simultaneously**" means all of them must happen to even have a <span style="color:#f77729;"><b>probability</b></span> of deadlock. These conditions are necessary but not sufficient, meaning that if all four are present, it is not 100% guaranteed that there is currently a deadlock.
{:.important}

{:.new-title}
> 4 Necessary Conditions
> 
> Since all 4 conditions are _necessary_ for deadlock to happen, <span style="color:#f7007f;"><b>removing</b></span> just one of them <span style="color:#f7007f;"><b>prevents</b></span> deadlock from happening at all.

## Resource Allocation Graph

Deadlocks can be described more precisely in terms of a directed graph called a system resource-allocation graph. This graph describes the <span class="orange-bold">current</span> system's resource allocation state. It can give us several clues on whether a deadlock is already happening in the system. 

Properties of the graph:
- It has a set of vertices **V** that is partitioned into two different types of **nodes**:
  - Active processes (**circle** node) and
  - All resource types (**square** node) in the system
- It has <span style="color:#f77729;"><b>directed</b></span> edges, from:
  - <span style="color:#f77729;"><b>Process to resource</b></span> nodes: process requesting a resource (<span style="color:#f77729;"><b>request edge</b></span>)
  - <span style="color:#f77729;"><b>Resource to process</b></span> nodes: assignment / allocation of that resource to the process (<span style="color:#f77729;"><b>assignment edge</b></span>)
- Resource <span style="color:#f77729;"><b>instances</b></span> within each resource type node (<span style="color:#f77729;"><b>dots</b></span>)

### Example 1

Suppose a system has the following <span style="color:#f77729;"><b>state</b></span>:

<img src="{{ site.baseurl }}/assets/images/week5/1.png"  class="center_fifty"/>

The resource allocation graph illustrating those states is as follows:

<img src="{{ site.baseurl }}/assets/images/week5/2.png"  class="center_fifty"/>

### Analysing Resource Allocation Graph

You should apply the following preliminary analysis to determine whether deadlock is present in the system: 
1. If the graph <span style="color:#f77729;"><b>has no cycle</b></span>: deadlock will <span style="color:#f7007f;"><b>never</b></span> happen.
2. If there’s _at least_ 1 cycle, three possibilities may occur: 
   - If <span style="color:#f77729;"><b>all</b></span> resources has exactly <span style="color:#f77729;"><b>one</b></span> instance, then <span style="color:#f7007f;"><b>deadlock</b></span> (deadlock necessary and sufficient condition)
   - If cycle involves only <span style="color:#f77729;"><b>a set</b></span> of resource types with <span style="color:#f77729;"><b>single</b></span> instance, then <span style="color:#f7007f;"><b>deadlock</b></span> (deadlock necessary and sufficient condition)
   - Else if cycle involves a set of resource types with <span style="color:#f7007f;"><b>multiple</b></span> instances, then _maybe_ deadlock (we can't say for sure, this is just a necessary but not sufficient condition)

In Example 1 graph above, these **three** processes are deadlocked (process and resource is shortened as `P` and `R` respectively):

- `P1` needs `R1`, which is currently held by `P2`
- `P2` needs `R4`, which is currently held by `P3`
- `P3` needs `R2`, which is currently held by either `P1` or `P2`

Neither process can give up their currently held resources to allow the continuation of the others resulting in a deadlock.

### Example 2

Now consider the another system state below. Although cycles are present, this system is <span style="color:#f7007f;"><b>not</b></span> in a deadlocked state because:
1. `P1` might eventually release `R2` after its done, and 
2. `P3` may acquire it and eventually complete its execution
3. Finally, `P2` may resume to completion after `P3` is done.

<img src="{{ site.baseurl }}/assets/images/week5/3.png"  class="center_fifty"/>

# Deadlock Handling Methods

There are three deadlock handling methods in general:

1. Deadlock <span style="color:#f77729;"><b>Prevention</b></span>
2. Deadlock <span style="color:#f77729;"><b>Avoidance</b></span>
3. Deadlock <span style="color:#f77729;"><b>Detection</b></span>

# Deadlock Prevention

Deadlock prevention works by <span style="color:#f77729;"><b>ensuring</b></span> that at least 1 of the necessary conditions for deadlock <span style="color:#f77729;"><b>never</b></span> happens.
{:.info}


## Mutual Exclusion Prevention

We typically cannot prevent deadlocks by denying the mutex condition, because some resources are intrinsically non-sharable. For example, a mutex lock cannot be simultaneously shared by several processes.

## Hold and Wait Prevention

To ensure that hold and wait never happens, we need to guarantee that whenever a process requests for a resource, it <span style="color:#f7007f;"><b>does not</b></span> currently hold any other resources.

This is possible through such protocols:

- <span style="color:#f77729;"><b>Protocol 1</b></span>: Must have all of its resources at once before beginning execution, otherwise, wait empty handed.
- <span style="color:#f77729;"><b>Protocol 2</b></span>: Only can request for new resources when it has none.

### Example

Consider a process that needs to `write` from a DVD drive to disk and then `print` a document from the disk to the printer machine.

<span style="color:#f77729;"><b>With protocol 1: </b></span>
The process must acquire <span style="color:#f77729;"><b>all</b></span> three resources: DVD drive, disk, and printer before beginning execution, even though the task with the disk and DVD drive has nothing to do with the printer. After it finishes both tasks, it releases both the disk and printer.

<span style="color:#f77729;"><b>With protocol 2: </b></span>
The process requests for DVD drive and disk, and writes to the disk. Then it releases <span style="color:#f77729;"><b>both</b></span> resources. Afterwards, it submits a new request to gain access to the disk (again) and the printer, prints the document, and releases both resources.

### Disadvantages

<span style="color:#f77729;"><b>Starvation</b></span>:
Process with <span style="color:#f77729;"><b>protocol 1</b></span> _may never start_ if resources are <span style="color:#f77729;"><b>scarce</b></span> and there are too many other processes requesting for it.

<span style="color:#f77729;"><b>Low resource utilization</b></span>:
With Protocol 1, resources are allocated at once but it is likely that they <span style="color:#f77729;"><b>aren’t used simultaneously</b></span> (e.g: the sample process requests a printer but it doesn’t utilize it when writing to disk first).
With Protocol 2, lots of time is <span style="color:#f77729;"><b>wasted</b></span> for requesting and releasing many resources in the <span style="color:#f77729;"><b>middle</b></span> of execution.

### No Preemption Prevention (Allow Preemption)

To _allow_ preemption, we need a certain protocols in place to ensure that progress is not lost.

A process is holding some resources and requests another resource that cannot be immediately allocated to it (that is, the process must wait). We can then decide to do either protocols below:

- <span style="color:#f77729;"><b>Protocol A</b></span>: All resources the process is currently holding are _preempted_ -- <span style="color:#f7007f;"><b>implicitly released</b></span>.
- <span style="color:#f77729;"><b>Protocol B</b></span>: We shall perform some checks first if the resources requested are held by other waiting processes or not. Then two possibilities may happen:
  - If held by other <span style="color:#f77729;"><b>waiting</b></span> process: preempts the waiting process and give the resource to the requesting process
  - If neither held by waiting process nor available: requesting process must wait.

These protocols are often applied to resources whose state can be easily <span class="orange-bold">saved</span> and <span class="orange-bold">restored</span> later, such as CPU registers and memory space. It cannot generally be applied to such resources as mutex locks and semaphores.
{:.note}

## Circular Wait Prevention

To prevent circular wait, one must have a protocol that impose a <span style="color:#f7007f;"><b>total ordering of all resource types</b></span>, and require that each process requests resources _according_ to that order. 

### Example

We are free to implement a protocol that ensures total ordering of all resource types however way we want. For instance, we can assign some kind of `id` to each resource and some resource *acquiring* protocols such as:

- The <span style="color:#f77729;"><b>highest</b></span> priority order of currently-held resources should be _less than or equal to_ the current resource being requested, otherwise the process must release the resources that violate this condition.
- Each process can request resources only in an <span style="color:#f77729;"><b>increasing</b></span> order of enumeration.

This burdens the programmer to ensure the order by design to ensure that it doesn't sacrifice resource utilization unnecessarily.
{:.warning}

# Deadlock Avoidance

In deadlock <span style="color:#f7007f;"><b>avoidance</b></span> solution, we need to spend some time to perform an algorithm to <span style="color:#f7007f;"><b>check</b></span> <span style="color:#f7007f;"><b>BEFORE</b></span> granting a resource request, _even if the request is valid and the requested resources are now available_.
{:.note}

This Deadlock Avoidance algorithm is called the <span class="orange-bold">Banker's Algorithm</span>. Its job is to compute and predict whether or not a current request _will_ lead to a deadlock.
- If yes, the current request for the resources will be rejected,
- If no, the current request will be granted.

This algorithm needs to run <span style="color:#f7007f;"><b>each time</b></span> a process request any shared resources, which makes it costly to run. 
{:.warning}

## The Banker's Algorithm

The Banker's Algorithm is comprised of <span style="color:#f77729;"><b>two</b></span> parts: <span class="orange-bold">The Safety Algorithm</span> and the <span class="orange-bold">Resource Allocation Algorithm</span>. The latter utilises the output of the former to determine whether the currently requested resource should be granted or not.

### Required Information

Before we can run the algorithm, we need to have the following information at hand:
1. Number of processes (consumers) in the system (denoted as `N`)
2. Number of resource <span style="color:#f77729;"><b>types</b></span> in the system (denoted as `M`), along with _initial instances_ of each resource type at the start.
3. The <span style="color:#f77729;"><b>maximum</b></span> number of resources required by each process (consumers).

### The System State

Using the above prior known information, the banker's algorithm maintains these four data structures representing the <span style="color:#f7007f;"><b>system state</b></span>: 
1. `available`: 1 by M vector
   - `available[i]`: the available instances of resource `i`
2. `max`: N by M matrix
   - `max[i][j]`: maximum demand of process `i` for resource `j` instances
3. `allocation`: N by M matrix
   - `allocation[i][j]`: current allocation of resource `j` instances for process `i`
4. `need`: N by M matrix
   - `need[i][j]`: how much more of resource `j` instances might be needed by process `i`

### Part 1: Resource Allocation Algorithm

You will be implementing this algorithm during [Lab](https://natalieagus.github.io/50005/labs/05-bankers-algorithm). As such, detailed explanation of the algorithm will not be repeated here.
{:.note}

This algorithm decides whether to give or not give resources to requesting processes over time. As time progresses, more <span style="color:#f77729;"><b>resource request</b></span> or <span style="color:#f77729;"><b>resource release</b></span> can be invoked by any of the processes. <span style="color:#f77729;"><b>Releasing</b></span> resources is a trivial process as we simply update the `allocation`, `available`, and `need` data structures. However, for each resource request received, we need to run the Safety Algorithm (next section) once. The output of the algorithm can be either `Granted` or `Rejected` depending on whether the system will be in a <span style="color:#f77729;"><b>safe state</b></span> _if the resource is granted_

### Part 2: The Safety Algorithm

You will be implementing this algorithm during [Lab](https://natalieagus.github.io/50005/labs/05-bankers-algorithm). As such, detailed explanation of the algorithm will not be repeated here.
{:.note}

This algorithm receives a <span class="orange-bold">copy</span> of `available` (named as `work`), `need`, and `allocation` data structures to perform a _hypothetical situation_ of whether the system <span style="color:#f7007f;"><b>will</b></span> be in a safe state *if* the current request is granted.

If we find that all processes <span style="color:#f77729;"><b>can still finish</b></span> their task (the elements of the `finish` vector is all `True`), then the system is in a safe state and we can grant this current request.

For example, given the following system state:

<img src="{{ site.baseurl }}/assets/images/week5/4.png"  class="center_seventy"/>

Suppose there exist a current request made by `P1`: `[1,0,2]`. You may find that granting this request leads to a <span style="color:#f7007f;"><b>safe state</b></span> and there exist several possible execution sequence (depending on _how_ you iterate through the `finish` vector, from index 0 onwards or from index `N-1` backwards):

1. `P1, P3, P4, P0, P2`,
2. `P1, P3, P4, P2, P0`

If you randomly iterate through the `finish` vector, you might end up with these other sequences too (which are also valid safe sequences):

3. `P1, P3, P2, P4, P0`
4. `P1, P3, P2, P0, P4`
5. `P1, P3, P0, P2, P4`
6. `P1, P3, P0, P4, P2`
7. `P1, P4, P3, P0, P2`
8. `P1, P4, P3, P2, P0`

In this algorithm,  you can also compute the sequence of _possible_ process execution sequence if you store each value of index `i` acquired. This sequence is <span class="orange-bold">not</span> unique.
{:.note}


In summary, the safety algorithm will return two possible output: safe or unsafe:
* **Safe State**: A state is considered safe if there exists a sequence of all processes such that each process can be allocated the maximum resources it may need, then return all the allocated resources back, and this can be done for all processes without causing a deadlock. In other words, there is at least one sequence of process execution that avoids deadlock.

* **Unsafe State**: A state is unsafe if no such sequence exists. However, being in an unsafe state does <span class="orange-bold">not</span> guarantee that a deadlock will occur; it simply means that there is a possibility of a deadlock if future resource requests are not handled properly.

### Unsafe != Guarantee of Deadlock
Entering an unsafe state does <span class="orange-bold">not</span> guarantee a deadlock will occur. Here's why:

* **Potential for Future Resource Availability**: Even if the current state is unsafe, future resource releases by other processes might allow the system to avoid deadlock. For instance, some processes might complete and release their resources, potentially creating a safe sequence that wasn't apparent initially.
* **Future Requests Might Be Different**: Processes might <span class="orange-bold">not</span> request the maximum resources they are theoretically entitled to. They might request fewer resources or none at all, which can also prevent a deadlock.
* **Preemption**: In some systems, certain resources or processes might be **preempted** or **forcibly** taken back from a process, allowing other processes to proceed and potentially avoid a deadlock situation.
* **Resource Release**: Some process might release its resources at one point in time, making it possible for other processes to progress. 


### System State Update

Note that these requests are made sequentially in time, so don’t forget to update the system state as you grant each request. When considering subsequent new requests, we perform the resource allocation algorithm with the <span style="color:#f77729;"><b>UPDATED</b></span> states that’s modified if you have granted the previous request.

However, the previous request is rejected, there's no change in system state is made and you can leave the data structures as-is. This means that the requesting process **must** wait and send the request again in the future. 


### Algorithm Complexity

Deadlock avoidance is <span style="color:#f77729;"><b>time-consuming</b></span>, since due to the expensive `while` loop in the safety algorithm. The time complexity of the safety algorithm is $$O(MN^2)$$ (which is also the complexity of the Banker's Algorithm, since the complexity of the Resource Allocation Algorithm is way smaller).

Carefully think the Banker's Algorithm is time consuming and compute its time complexity. Also, ask yourself: what is the space complexity of the Banker's Algorithm?
{:.note}

# Deadlock Detection

In deadlock detection, we <span style="color:#f7007f;"><b>allow</b></span> deadlock to happen first (we don't deny its possibility), and then detect it later.

Deadlock detection method is <span class="orange-bold">different</span> from deadlock avoidance. The latter is one step ahead since will not grant requests that may lead to deadlocks in the first place. 
{:.important}

Deadlock detection mechanism works using two steps:
- By running an algorithm that <span style="color:#f77729;"><b>examines</b></span> the state of the system to determine whether a deadlock <span style="color:#f7007f;"><b>has occurred</b></span> from time to time,
- And then running algorithm to <span style="color:#f77729;"><b>recover</b></span> from deadlock condition (if any)

## Deadlock Detection Algorithm

This algorithm is similar to the Safety Algorithm (Part 2 of the Banker’s Algorithm), just that this algorithm is performed on the _actual_ system state (not the hypothetical `work`, `need` and `allocation`).
{:.note}

Firstly, we need all the state information as per the Banker’s algorithm: the `available`, `request`(renamed from `need`), and `allocation` matrices (no max/need). The only subtle difference is only that the `need` matrix (in safety algorithm) is replaced by the `request` matrix.

In a system that adapt deadlock <span style="color:#f77729;"><b>detection</b></span> as a solution to the deadlock problem, we don’t have to know the `max`(resource needed by a process), because we will _always_ grant a request whenever whatever’s requested is <span style="color:#f77729;"><b>available</b></span>, and invoke deadlock detection algorithm from time to time to ensure that the system is a good state and not deadlocked.
{:.info}

The `request` matrix contains current requests of resources made by <span style="color:#f77729;"><b>all</b></span> processes at the instance we decide to <span style="color:#f7007f;"><b>detect</b></span> whether or not there’s (already) a deadlock occuring in the system.  

> Row `i` in the `request` matrix stands for: Process `i` requiring some <span style="color:#f77729;"><b>MORE</b></span> resources that’s what’s been allocated for it.

The deadlock detection algorithm works as follows:
- <span style="color:#f77729;"><b>Step 1</b></span>: Initialize these two vectors:
  - `work` (length is #resources `M`) initialized to be == `available`
  - `finish` (length is #processes `N`) initialized to be:
    - `False` if `request[i]` != {0} (empty set)
    - True `otherwise` (means process i doesn’t request for anything else anymore and can be resolved or finished)
  - No update of `allocation, need` needed. We are checking whether the CURRENT state is safe, not whether a HYPOTHETICAL state is safe.
- <span style="color:#f77729;"><b>Step 2</b></span>: find an index `i` such that <span style="color:#f77729;"><b>both</b></span> conditions below are fulfilled,
  - `Finish[i] == False`
  - `request[i] <= work` (element-wise comparison for these vectors)
- <span style="color:#f77729;"><b>Step 3</b></span>:
  - If Step 2 produces such index `i`, update `work` and `finish[i]`,
    - `work += allocation[i]` (vector addition)
    - `finish[i] = True`
    - <span style="color:#f7007f;"><b>Go back to Step 2</b></span>
  - If Step 2 does not produce such index `i`, prepare for exit:
    - If `finish[i] == False` for some `i`, then the system is <span style="color:#f7007f;"><b>already</b></span> in a deadlock (caused by Process `i` getting deadlocked by another Process `j` where `finish[j] == False` too)
    - Else if `finish[i] == True` for all `i<N`, then the system is <span style="color:#f7007f;"><b>not in a deadlock</b></span>.

## Complexity of Deadlock Detection Algorithm
This deadlock detection algorithm requires an order of $$O(MN^2)$$ <span style="color:#f77729;"><b>operations</b></span> to perform.

## Frequency of Detection
When should we invoke the detection algorithm? The answer depends on two factors:
- How often is a deadlock likely to occur?
- How many processes will be affected by deadlock when it happens?

This remains an open issue.
{:.highlight}

## Deadlock Recovery
After deadlock situation is detected, we need to recover from it. To recover from deadlock, we can either:

1. Abort <span style="color:#f77729;"><b>all</b></span> deadlocked processes, the resources held are preempted
2. Abort deadlocked processes <span style="color:#f77729;"><b>one at a time</b></span>, until deadlock no longer occurs

The second method results in an <span style="color:#f7007f;"><b>overhead</b></span> since we need to run the detection algorithm over and over again each time we abort a process. There are a few design choices we have to make: 

- Which <span style="color:#f77729;"><b>order</b></span> shall we abort the processes in?
- Which “victim” shall we select? We need to determine <span style="color:#f77729;"><b>cost</b></span> factors such as which processes have executed the longest, etc
- If we select a “victim”, how much <span style="color:#f77729;"><b>rollback</b></span> should we do to the process?
- How do we ensure that <span style="color:#f77729;"><b>starvation</b></span> does not occur?

Deadlock Recovery remains an open ended issue. Dealing with deadlock is also a <span class="orange-bold">difficult</span> problem. Most operating systems do <span class="orange-bold">not</span> prevent or resolve deadlock completely and users will deal with it when the need arises.
{:.warning}

# Summary

This chapter explores the concept of deadlock in operating systems, a **critical** issue that arises in concurrent programming when two or more processes are unable to proceed because each is waiting for the other to release a resource. It delves into the **necessary** conditions for deadlock to occur, including mutual exclusion, hold and wait, no preemption, and circular wait. The guide discusses strategies for preventing and handling deadlock, such as resource allocation graphs, deadlock detection, and recovery mechanisms. 


Key learning points include:
- **Conditions for Deadlock**: Understanding the specific conditions that must all be present for a deadlock to occur.
- **Deadlock Prevention and Avoidance**: Techniques to prevent deadlock by structurally eliminating the possibility of its conditions, and using algorithms like the Banker’s Algorithm to avoid deadlocks dynamically.
- **Deadlock Detection and Recovery**: Methods to detect deadlocks and strategies to recover from them, such as process termination or resource preemption.


By understanding the causes and solutions to deadlock, programmers can design more robust and efficient concurrent systems.

<hr>


[^1]: **Preemption**: the act of temporarily interrupting a task being carried out by a process or thread, without requiring its cooperation, and with the intention of resuming the task at a later time.

