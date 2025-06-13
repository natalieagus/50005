---
layout: default
permalink: /os/synchronization
title: Processes and Threads Synchronization 
description: Methods on how processes and threads can be made cooperative
parent: Operating System
nav_order: 8
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

# Processes and Threads Synchronization

{:.highlight-title}
> Detailed Learning Objectives
>
> 1. **Explain the Producer-Consumer Problem**
>   - Explain **issues** of multiprogramming and concurrency: the race condition
>   - Identify the challenges of managing two asynchronous processes or threads (producer and consumer) that interact through a shared bounded buffer.
>   - Distinguish between the terms "**asynchronous**," "**concurrent**," and "**parallel**" and their implications on process execution and scheduling.
>   - Identify **real-life examples** of producer-consumer scenarios such as a compiler and assembler interaction, and web servers and browsers.
>   - Explain the **precedence constraints**: the  necessary conditions to prevent the producer from overfilling the buffer and the consumer from reading from an empty buffer.
> 2. **Explain Critical Section and Race Conditions**
>   - Define the critical section where processes access shared resources that **require mutual exclusion**.
>   - Explain race conditions, particularly how they can disrupt program correctness in the context of shared variables like counters.
> 3. **Explain Various Synchronization Mechanisms**
>   - **Prove** how various race condition solution supports mutual exclusion, progress, and bounded waiting
>   - **Explore** different synchronization **techniques** including software and hardware solutions, semaphores, and condition variables.
>   - Discuss how these solutions **enforce** mutual exclusion, **manage** process execution order (bounded waiting), and maintain **progress** and avoid starvation.
>   - Discuss how **Peterson’s solution** solve the critical section problem and prove its correctness.
>   - Summarize the difference between solutions requiring and not requiring to **busy wait**.
>   - Explain how **semaphores** function as counters for resource management, capable of solving complex synchronization problems beyond simple mutual exclusion with minimised **busy waiting**
>   - Explain how condition variables work with mutexes to provide a way for threads to wait for specific conditions to proceed, enabling efficient resource management without busy waiting.
> 4. **Evaluate the Efficiency and Applicability of Each Synchronization Mechanisms**
>   - Assess **when** to use each type of synchronization technique based on the specific requirements and context of the problem, considering factors like system architecture, overhead, and the nature of the critical section.
> 5. **Explore Practical Implementations and Code Examples**
>   - Examine C code samples that implement mutexes, condition variables, and semaphores to handle synchronization in multi-threaded environments.
> 
> These objectives guide through understanding the theoretical concepts and practical implementations, emphasizing the importance of correct synchronization in concurrent programming to maintain data integrity and program stability.

## Background: Asynchrony vs Concurrency vs Parallelism

{:.note}
**Asynchronous** processing involves **non-blocking** tasks that run independently without waiting for others, **concurrent** processing manages <span class="orange-bold">multiple</span> tasks making progress over time without simultaneous execution, while **parallel** processing executes multiple tasks at the same time on multiple cores or processors.

### Asynchronous Processing
Asynchronous processing allows a program to perform tasks without waiting for others to complete. This is particularly useful in I/O operations where waiting for a resource could block other tasks. Instead, tasks can be started and the program can move on to other tasks while waiting for the initial tasks to complete.

**Example**: Making an HTTP request. The program can initiate the request and then continue executing other code. When the response is received, a <span class="orange-bold">callback</span> function can be executed to handle the response.

{:.note}
Asynchrony does <span class="orange-bold">not</span> necessarily require multi-threading. 

### Concurrent Processing
Concurrent processing involves multiple tasks making progress over time, but not necessarily simultaneously. This can be achieved through various methods like time slicing, where each task is given a small time slot to execute before moving to the next one. In order to *achieve concurrency*, there has to be overlapping management, suspension, coordination, or scheduling (need task switching *or* interleaving).

**Example**: Running multiple threads on a single-core processor. Each thread gets a time slice to run, and the operating system switches between them, creating the illusion of simultaneous execution.

{:.note}
> Asynchrony *enables* concurrency, but does **not** guarantee it because in the code we can still `await` to ensure one task runs after another, thus effectively disabling concurrency. Multithreading **also** enables concurrency but does not guarantee it because scheduler might just schedule these threads *in sequence*. 
> 
> However concurrency <span class="orange-bold">cannot</span> happen without either asynchrony or multithreading. 

### Parallel Processing
Parallel processing involves performing multiple tasks simultaneously. This requires multiple processors or cores. Each core can execute a different task at the same time, leading to true simultaneous execution.

**Example**: Running multiple processes on a multi-core processor. Each process can run on a different core, truly executing in parallel.

### Key Differences
1. **Execution Style**:
   - **Asynchronous**: <span class="orange-bold">Non-blocking</span>, tasks are started and the program continues without waiting.
   - **Concurrent**: Multiple tasks are in **progress**, but they may not be running at the exact same time.
   - **Parallel**: Multiple tasks are executed **simultaneously**, requiring multiple cores or processors.

2. **Typical Use Cases**:
   - **Asynchronous**: I/O-bound tasks, such as reading/writing files, network requests.
   - **Concurrent**: Multithreading in applications, such as GUI applications where you **don’t** want the interface to **freeze** while performing background tasks.
   - **Parallel**: CPU-bound tasks, such as scientific calculations, data processing on large datasets.

3. **Hardware Requirements**:
   - **Asynchronous**: Does not necessarily require multiple cores or processors.
   - **Concurrent**: Can be achieved with a single-core processor through time slicing.
   - **Parallel**: Requires **multiple** cores or processors.

### Summary
- **Asynchronous** processing allows tasks to be executed **without** blocking the main program flow, often used for I/O operations.
- **Concurrent** processing enables **multiple** tasks to make <span class="orange-bold">progress over time</span>, often through multithreading or multiprocessing.
- **Parallel** processing executes multiple tasks **simultaneously**, requiring multi-core processors or multiple processors.

{:.info}
Each of these concepts can be implemented using different techniques and technologies depending on the specific requirements and constraints of the application. Understanding when and how to use each can significantly impact the performance and efficiency of software systems.

## The Producer Consumer Problem

Consider the basic producer-consumer problem, where two <span style="color:#f77729;"><b>asynchronous</b></span> processes or threads are trying to write to and read from the same bounded N-character buffer <span style="color:#f77729;"><b>concurrently</b></span>.

**Asynchronous** and **concurrent** means that both processes or threads make progress, but we cannot assume anything about their relative speed of execution.
{:.important}

<img src="{{ site.baseurl }}/assets/images/week4/1.png"  class="center_fifty"/>

Confused about the term **asynchronous** or **concurrent** or **parallel**? Read [this burger ordering analogy](https://fastapi.tiangolo.com/async/) that was found by one of your CSD seniors.
{:.new}

### Real Life Examples

In real life situations, a producer process produces information that is consumed by a consumer process. For example:

- A <span style="color:#f77729;"><b>compiler</b></span> (producer) producing assembly code that is consumed by an <span style="color:#f77729;"><b>assembler</b></span> (consumer).
- The assembler can also be a producer with respect to the <span style="color:#f77729;"><b>loader</b></span>: it produces object modules that are consumed by the loader.
- A web <span style="color:#f77729;"><b>server</b></span> produces (that is, provides) HTML files and images, which are consumed (that is, read) by the client web <span style="color:#f77729;"><b>browser</b></span> requesting the resource.

## Precedence Constraints

We require the following <span class="orange-bold">precedence constraints</span> for these asynchronous processes:

- <span style="color:#f7007f;"><b>Producer cannot produce too much before consumer consumes </b></span>
- <span style="color:#f7007f;"><b>Consumer cannot consume before producer produces</b></span>

## What happens if there's no synchronization?

To highlight what happens if we don’t synchronize both processes / threads, lets see a very simple <span style="color:#f77729;"><b>producer</b></span> program. Let `counter` and `buffer` of size `N` be a <span style="color:#f77729;"><b>shared</b></span> variable between the producer and consumer. `counter` keeps track of how many items there are in the `buffer`.

```cpp
while (true) {
    /* produce an item in next produced */
    while (counter == BUFFERSIZE); /* do nothing */
    buffer[in] = nextproduced;
    in = (in + 1) % BUFFERSIZE;
    counter++;
}
```

And the following very simple <span style="color:#f77729;"><b>consumer</b></span> program:

```cpp
while (true) {
    while (counter == 0); /* do nothing */
    next consumed = buffer[out];
    out = (out + 1) % BUFFERSIZE;
    counter--;
    /* consume the item in next consumed */
}
```

Each code is correct on its own, but incorrect when executed <span style="color:#f7007f;"><b>concurrently</b></span>, meaning that without any proper synchronisation attempts, the <span style="color:#f7007f;"><b>precedence</b></span> condition will be <span style="color:#f7007f;"><b>violated</b></span>.
{:.error}

This is due to the presence of <span class="orange-bold">race condition</span>.

# The Race Condition

Assume `buffer` and `counter` are shared between the two processes / threads. The instructions `counter ++` and `counter -- `are not implemented in a single clock cycle (it is <span style="color:#f77729;"><b>not atomic</b></span>).

## Atomic Operation

{:.info-title}
> Atomicity
>
> An atomic operation is an indivisible operation that completes in a single step relative to other threads. It appears instantaneous and is executed without any interference from other operations, ensuring consistency and preventing race conditions.

Note that atomic operation does **NOT** necessarily mean it is non-interruptible. An atomic operation ensures that the operation is completed as a single, indivisible step **with respect to other threads/processes**, meaning that once the operation begins, it will complete without **interference** from other operations. 

This guarantees consistency and prevents race conditions. However, remember that it does not imply that the operation cannot be interrupted at the hardware or operating system level; it means that from the **perspective** of other threads or processes, the operation appears to happen instantaneously and without interruption.

## Non-atomic `++` and `--`

`counter++` may be implemented as follows in assembly language:

```armasm
LDR(counter, R2)
ADDC(R2, 1, R2) || or SUBC for counter--
ST(R2, counter)
```

This is **non-atomic** because it is composed of multiple **separate** instructions that are <span class="orange-bold">not</span> executed as a single, indivisible operation. More precisely, each of these instructions can be **interleaved** by the execution of other threads, leading to inconsistencies (race condition) in the result.


## Race Condition Outcome 1

The execution between `counter ++ `and `counter --` can therefore be <span style="color:#f7007f;"><b>interleaved</b></span>. For example in a uniprocessor system, when the value of `counter` is `4` and the producer is now writing the 5th item, it could get interrupted while executing `counter++` for consumer to consume the 5th item and cause the following interleaved execution between the two:

```armasm
LDR(counter, R2) | Producer executes, then interrupted, R2’s content:4
...IRQ on producer, save state, restore consumer
LDR(counter, R2) | Consumer executes, R2 contains 4
SUBC(R2, 1, R2)  | R2 contains 3, then consumer is interrupted
...IRQ on consumer, save state, restore producer
ADDC(R2, 1, R2)  | R2 contains 5
ST(R2, counter)  | value 5 is stored at counter, then IRQ
...IRQ on producer, save state, restore consumer
ST(R2, counter)  | value 3 is stored at counter
```

Therefore the final value of the `counter` is `3` when it should be `4`.

## Race Condition Outcome 2

We can try another combination of interleaved execution:

```armasm
LDR(counter, R2) | Producer executes, then interrupted R2’s content:4
...IRQ on producer, save state, restore consumer
LDR(counter, R2) | Consumer executes, R2 contains 4
SUBC(R2, 1, R2)  | R2 contains 3
ST(R2, counter)  | value 3 is stored at counter, then consumer is interrupted
...IRQ on consumer, save state, restore producer
ADDC(R2, 1, R2)  | R2 contains 5
ST(R2, counter)  | value 5 is stored at counter
```

Therefore in the case above, the final value of the `counter` is `5` which is still incorrect.

## Other Race Condition Outcomes

You may easily find that the value of the counter can be 4 as well through other combinations of interleaved execution of `counter ++` and `counter --`.

Hence the value of the counter <span style="color:#f7007f;"><b>depends on the order of execution</b></span> that is out of the user's control (the order of execution depends on the kernel’s scheduling handler, unknown to the user).

In such a situation where several processes access and manipulate the same data concurrently and the outcome of the execution depends on the particular order in which the access takes place, is called a <span style="color:#f7007f;"><b>race condition</b></span>.
{: .note}

A race condition is <span class="orange-bold">NOT desirable</span>. We need to perform process <span style="color:#f7007f;"><b>synchronization</b></span> and <span style="color:#f7007f;"><b>coordination</b></span> among cooperating processes (or equivalently, threads cooperation) to avoid the race condition.

# The Critical Section Problem

We define the <span style="color:#f7007f;"><b>regions</b></span> in a program whereby <span class="orange-bold">atomicity</span> must be guaranteed as the <span style="color:#f7007f;"><b>critical section</b></span>.
{:.note}

Consider a system consisting of `n` processes `{P0, P1,..., Pn−1}`. Each process may have a <span style="color:#f77729;"><b>segment</b></span> of instructions, called a `critical section` (CS). The important <span style="color:#f7007f;"><b>feature</b></span> of the system is that <span style="color:#f77729;"><b>when one process is executing its critical section, no other process is allowed to execute its critical section</b></span>.

In the consumer producer sample code above, the critical section in the producer’s code is the instruction `counter++` while the critical section in the consumer’s code is `counter-—`.

In the critical section the (asynchronous) processes may be:

- Changing <span style="color:#f77729;"><b>common</b></span> variables,
- Updating a <span style="color:#f77729;"><b>shared</b></span> table,
- Writing to a <span style="color:#f77729;"><b>common</b></span> file, and so on.

To support the CS, we need to design a <span style="color:#f77729;"><b>protocol</b></span> that the processes can use to cooperate or synchronize.

It is a challenging problem to <span style="color:#f77729;"><b>protect</b></span> a critical section. Therefore, having a critical section in your program is a <span style="color:#f77729;"><b>problem</b></span>, and it requires complex synchronisation solutions.
{: .warning}

There are <span style="color:#f77729;"><b>two</b></span> basic forms of synchronization:

- <span style="color:#f77729;"><b>Mutual exclusion</b></span>: No other processes can execute the critical section if there is already one process executing it.
- <span style="color:#f77729;"><b>Condition synchronization</b></span>: Synchronize the execution of a process in a CS based on certain conditions instead.

## Requirements for CS Solution

A solution to guarantee a critical-section must satisfy the following three requirements:

- <span style="color:#f77729;"><b>Mutual exclusion</b></span> (mutex): No other processes can execute the critical section if there is already one process executing it (in the case of condition synchronization, this is adjusted accordingly)
- <span style="color:#f77729;"><b>Progress</b></span>: If there’s no process in the critical section, and some other processes wish to enter, we need to grant this permission and we cannot postpone the permission indefinitely.
- <span style="color:#f77729;"><b>Bounded waiting</b></span>: If process A has requested to enter the CS, there exists a bound on the number of times other processes are allowed to enter the CS before A. This implies that CS is also of a finite length, it cannot loop forever and will exit after a finite number of instructions. It is a requirement that ensures **fairness** by guaranteeing that every process will eventually get a chance **to** enter its critical section (CS) after a finite number of other processes have done so.

{:.note}
If the progress condition is violated, meaning no process can ever enter the critical section again, discussing bounded waiting becomes moot. This is because progress is the property that ensures at least one of the processes not in its critical section but wishing to enter is eventually able to do so without interference. Without progress, the system can get stuck in a state where no further entries into critical sections are possible, making the assurance of bounded waiting irrelevant.

## Properties

The requirements above result in the following property to a CS solution:

- <span style="color:#f77729;"><b>Safety</b></span> property: no race condition
- <span style="color:#f77729;"><b>Liveness</b></span> property: a program with proper CS solution will not hang forever (because technically no progress IS mutex).

## Solution Template

The solution template to a CS problem is as follows:

```cpp
while(true){

   [ENTRY SECTION]
      CRITICAL SECTION ...
      ...
   [EXIT SECTION]
      REMAINDER SECTION ...
      ...
}
```

The protocol to approach a CS in general causes the process to:

- <span style="color:#f77729;"><b>Request</b></span> for permission to enter the section (entry section).
- <span style="color:#f77729;"><b>Execute</b></span> the critical section when the request is granted
- <span style="color:#f77729;"><b>Exit </b></span> the CS solution
- There may exist an _remainder_ section

The rest of the program that is not part of the critical section is called the <span style="color:#f77729;"><b>remainder</b></span> section.
{:.note}



In the next few sections, we discuss several known solutions to CS problems. They are generally divided into these categories:

- <span style="color:#f77729;"><b>Software Mutex </b></span>Algorithm (purely software only, possible only under restricted conditions, busy waits)
- <span style="color:#f77729;"><b>Hardware</b></span> Supported <span style="color:#f77729;"><b>Spinlocks</b></span> (hardware supported, only 1 process in the CS at a time, busy waits)
- <span style="color:#f77729;"><b>Software Spinlocks and Mutex Locks</b></span>
- <span style="color:#f77729;"><b>Semaphores</b></span> (does not busy wait, generalization of mutex locks -- able to protect two or more identical shared resources)
- <span style="color:#f77729;"><b>Condition variables</b></span> (does not busy wait, wakes up on condition)

# Software Mutex Algorithm

## Peterson's Solution

Peterson’s solution is a <span style="color:#f77729;"><b>software-based</b></span> approach that solves the CS problem, but two restrictions apply:

- Strictly <span style="color:#f77729;"><b>two</b></span> processes that <span style="color:#f77729;"><b>alternate</b></span> execution between their critical sections and remainder sections (can be [generalised to `N` processes](https://www.geeksforgeeks.org/n-process-peterson-algorithm/) with proper data structures, out of syllabus)
  - Practically applied in a <span style="color:#f77729;"><b>single</b></span> core environment only
- Architectures where `LD` and `ST` are <span style="color:#f77729;"><b>atomic</b></span>[^1] (i.e: executed in 1 clk cycle, or not interruptible).
  - **Definition**: atomically storing a value `x` into a memory location with initial value `y` means that the state of that location shall either be `x` or `y` when an attempt to read it at any point in time is done. No _intermediary value_ shall ever be observed.

The solution works by utilizing two shared global variables:

```cpp
int turn;
bool flag[2];
```

- If `turn == i`, process `Pi` is allowed to <span style="color:#f77729;"><b>enter</b></span> the critical section. Similar otherwise.
- If `flag[i] == true`, then process `Pi` is <span style="color:#f77729;"><b>ready</b></span> to enter the critical section.

### Initialisation

We can initialize the `flag` into `false` for all `i`, and set `turn` into <span style="color:#f77729;"><b>arbitrary number</b></span> (`i` or `j` to index two processes `Pj` and `Pi` -- two processes only for our syllabus) in the beginning.

### Algorithm

```cpp
do{
   flag[i] = true;
   turn = j;
   while (flag[j] && turn == j); // this is a while LINE
   // CRITICAL SECTION HERE
   // ...
   flag[i] = false;
   // REMAINDER SECTION HERE
   // ...
}while(true)
```

The algorithm above is the solution for process `Pi`. The code for `Pj` is equivalent, just swap all the `i` to `j` and vice versa.

- In the `while`-loop, `Pi` busy waits (means <span style="color:#f77729;"><b>try</b></span> and keep <span style="color:#f77729;"><b>retrying</b></span> until succeed, thus wasting the quanta given to the process),
- `Pi` will be <span style="color:#f7007f;"><b>stuck</b></span> at the while-line (notice it's NOT a while-loop, there’s a semicolon at the end), for as long as `flag[j] == true` <span style="color:#f7007f;"><b>and</b></span> `turn == j`.

## Proof of Correctness

To prove that this solution is correct, we need to show that:

- <span style="color:#f77729;"><b>Mutual exclusion</b></span> is preserved.
- The <span style="color:#f77729;"><b>progress</b></span> requirement is satisfied.
- The <span style="color:#f77729;"><b>bounded-waiting</b></span> requirement is met.

We prove that the Peterson’s solution satisfies all <span style="color:#f77729;"><b>three</b></span> properties above by tracing how it works:

- When `Pi` wants to enter the critical section, it will set its own flag; `flag[i]` into `true`.
- Then, it sets `turn=j` (instead of itself, i).

You can think of a process as being <span style="color:#f77729;"><b>polite</b></span> hence requesting the _other one_ (`j`) to be executed <span style="color:#f77729;"><b>first</b></span>.
{: .note}

Now two different scenarios might happen.

### Scenario 1: Proceed to CS

`Pi`might break from the `while`-loop under two possible conditions.

<span style="color:#f77729;"><b>Condition 1:</b></span> `flag[j] == false`, meaning that the other `Pj` is <span style="color:#f7007f;"><b>not ready</b></span> to enter the CS and is <span style="color:#f7007f;"><b>also not in the critical section</b></span>. This ensures <span style="color:#f77729;"><b>mutex</b></span>.

<span style="color:#f77729;"><b>Condition 2:</b></span> OR, IF `flag[j] == true` but `turn == i`. This means the other process `Pj` is also <span style="color:#f77729;"><b>about</b></span> to enter the critical section. No process is in the Critical Section, but it is `Pi`'s turn, so `Pi` gets to enter the CS first (ensuring <span style="color:#f77729;"><b>progress</b></span>).

### Scenario 2: Busy-wait

`Pi` will be stuck with the `while`-line if `flag[j] == true` <span style="color:#f7007f;"><b>AND</b></span> `turn == j`, meaning that `Pj`is inside the CS.

This satisfies <span style="color:#f77729;"><b>mutual exclusion</b></span>: `flag[i]` and `flag[j]` are both `true`, but `turn` is an <span style="color:#f77729;"><b>integer</b></span> and cannot be both `i` and `j`.

- No two processes can simultaneously execute the CS.

<span style="color:#f77729;"><b>Bounded waiting is guaranteed</b></span>: After `Pj` is done, it will set its own flag: `flag[j] = false`, hence ensuring `Pi` to <span style="color:#f77729;"><b>break</b></span> out of the `while`-line and enter CS in the future.

- `Pi` only needs to wait a <span style="color:#f7007f;"><b>MAXIMUM</b></span> of 1 cycle before being able to enter the CS.

You might want to <span style="color:#f77729;"><b>interleave</b></span> the execution of instructions between `Pi` and `Pj`, and convince yourself that this solution is indeed a legitimate solution to the CS problem. Try to also modify some parts: not to use `turn`, set `turn` for `Pi` as itself (`i`) instead of `j`, not to use `flag`, etc and convince yourself whether the 3 <span style="color:#f77729;"><b>correctness</b></span> property for the original Peterson's algorithm still apply.
{:.info}

## Will Peterson's work in modern CPUs out of the box?

The short answer is <span class="orange-bold">not really</span>. This is not part of the syllabus, so head to the [appendix](#petersons-in-modern-cpus) to find out more. 


# Synchronization Hardware

Peterson’s solution is a software solution that is not guaranteed to work on modern computer architectures where `LD/ST` might not be atomic (e.g: there are many processors accessing the same location).

We can also solve the CS Problem using hardware solution. All these solutions are based on the premise of <span style="color:#f77729;"><b>hardware locking</b></span> to protect the critical sections. This is significantly different from software locks. The number of hardware locks per system is <span style="color:#f77729;"><b>physically</b></span> limited.

## Preventing Interrupts

We can intentionally prevent interrupts[^2] from occurring while a shared variable was being modified.

This only works in <span style="color:#f77729;"><b>single-core</b></span> systems and is <span style="color:#f7007f;"><b>not</b></span> feasible in multiprocessor environments because:

- Time consuming, need to pass message to all processors
- Affects system clocks
- Decreases efficiency, defeats the purpose of multiprocessors.

The ability to temporarily inhibit interrupts and ensuring that the currently running process cannot be context switched need to be supported at hardware levels in many modern architectures. If no interrupt can occur during a particular CS, then mutual exclusion is <span style="color:#f77729;"><b>trivial</b></span>.

## Atomic Instructions

We can also implement a set of <span style="color:#f77729;"><b>atomic</b></span> instructions by <span style="color:#f77729;"><b>locking the memory bus</b></span>. Mutual exclusion is therefore <span style="color:#f77729;"><b>trivial</b></span>.
{:.warning}

This is typically used to provide <span style="color:#f77729;"><b>mutual exclusion</b></span> in symmetric multiprocessing systems. The common instructions are:

- Swap two memory words: `compare_and_swap()`
- Test original value of memory word and set its value: `test_and_set()`

### Example

Lets say **CPU1** issues `test_and_set()`, the modern DPRAM (dual port ram) makes an internal note by <span style="color:#f77729;"><b>storing</b></span> the address of the memory location in its own dedicated place. If **CPU2** also issues `test_and_set()` to the <span style="color:#f7007f;"><b>same</b></span> location then DPRAM checks its internal note and notices that CPU1 is accessing it, so it will issue a <span style="color:#f77729;"><b>busy</b></span> interrupt to **CPU2**.

**CPU2** might be busy waiting (spinlock) to <span style="color:#f7007f;"><b>retry</b></span> again (until succeeds). This happens at <span style="color:#f77729;"><b>hardware speed</b></span> so it is actually not very long at all. After CPU1 is done, the DPRAM erases the internal note, thus allowing CPU2 to execute its `test_and_set()`.

Below are the common (but not limited to) atomic instructions supported at the hardware level. They are used <span style="color:#f77729;"><b>directly</b></span> by compiler and operating system <span style="color:#f77729;"><b>programmers</b></span> but are also abstracted and exposed as bytecodes and library functions in higher-level languages like C:

- atomic `read` or `write`
- atomic `swap`
- `test-and-set`
- `fetch-and-add`
- `compare-and-swap`
- `load-link`/`store-conditional`

# Software Spinlocks and Mutex Locks

We need hardware support for certain <span style="color:#f77729;"><b>special</b></span> atomic assembly-language instructions like `test-and-set` above. This can be used by application programmers to <span style="color:#f77729;"><b>implement</b></span> software locking without the need to switch to kernel mode or perform context switch (unless otherwise intended). On architectures without such hardware support for atomic operations, a non-atomic locking algorithm like Peterson's algorithm can be used.

However, such an implementation may require more memory than a hardware spinlock.

## Spinlocks

A spinlock provides mutual exlusion. It is simply a variable that can be initialized, e.g `pthread_spinlock_t` implemented in C library and then obtained or released using two <span style="color:#f77729;"><b>standard</b></span> methods like `acquire()` and `release()`. An attempt to `acquire()` the lock causes a process or thread trying to acquire it to wait in a loop ("spin") while repeatedly checking whether the lock is available.

This is also called <span style="color:#f77729;"><b><span class="orange-bold">busy waiting</span></b></span>, hence wasting the caller's quantum until the caller acquires the lock and can progress.
{:.note}

Example implementation of `acquire()` in C library:

```cpp
acquire() {
   /* this test is performed using atomic test_and_set and busy wait for the process' CS, hardware supported */
     while (!available);
     available = false;
}

release(){
	available = true;
}
```

You can create spinlock using C POSIX library: `pthread_spin_lock()`

```cpp
static pthread_spinlock_t spinlock;
pthread_spin_init(&spinlock,0);
pthread_spin_lock(&spinlock); // no context switch, no system call, busy waits if not available
// CRITICAL SECTION ...
pthread_spin_unlock(&spinlock);
// REMAINDER SECTION ...
```

{:.note-title}
> Sharing locks between Processes
>
> In C, the function pthread_spin_init is used to initialize a spinlock. The function signature is `int pthread_spin_init(pthread_spinlock_t *lock, int pshared);`. 
>
> The `pshared` parameter can take one of two values: 
> * `PTHREAD_PROCESS_SHARED` (1): The spinlock can be shared between processes.
> * `PTHREAD_PROCESS_PRIVATE` (0):  The spinlock is only used within a single process.
>
> When you use `0` as the value for the `pshared` parameter, it is equivalent to using `PTHREAD_PROCESS_PRIVATE`. This means the spinlock will not be shared outside the process in which it is created. 
> 
> Therefore, `pthread_spin_init(&spinlock, 0);` initializes `spinlock` for use within the same process, ensuring it is not shared between multiple processes.
>
> To share a spinlock outside of processes, read this [appendix](#share-spinlock-between-processes) section.



### Busy Waiting

Busy waiting <span style="color:#f77729;"><b>wastes</b></span> CPU cycles -- some other process might be able to use productively, and it affects efficiency tremendously when a CPU is shared among many processes. The spinning caller will utilise 100% CPU time just waiting: repeatedly checking if a spinlock is available.

Does it make much sense to use spinlocks in single-core environment? The spinlock polling is blocking the only available CPU core, and as a result no other process can run. Since no other process can run, the lock won't be unlocked either.
{: .highlight}

Nevertheless, spinlocks are mostly useful in places where anticipated waiting time is <span style="color:#f77729;"><b>shorter</b></span> than a quantum and that multicore is present. <span style="color:#f7007f;"><b>No context switch is required</b></span> when a process must wait on a lock, the calling process simply utilise the special assembly instruction.
{:.warning}

## Mutex Lock

Some other libraries (like C POSIX library) implement another type of lock called <span style="color:#f77729;"><b>mutex lock</b></span> that does not cause busy wait.

Mutex locks are also supported by special atomic assembly instructions implemented at hardware level, and requires <span style="color:#f7007f;"><b>integration</b></span> with the Kernel Scheduler because it will put the requesting thread/process to <span style="color:#f77729;"><b>sleep</b></span> if the lock has already been acquired by some other thread.

Example of POSIX Mutex usage:

```cpp
// initialize mutex
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

pthread_mutex_lock(&mutex); // this is  acquire() equivalent, atomic and might invoke kernel scheduler and context switch if mutex not available
// CRITICAL SECTION HERE ...
pthread_mutex_unlock(&mutex); // this is  release() equivalent, atomic and might invoke kernel scheduler as well to wake up other waiting processes/threads
// REMAINDER SECTION HERE ...
```

The problem with mutexes is that putting threads to sleep and waking them up again are both rather <span style="color:#f7007f;"><b>expensive</b></span> operations, they'll need quite a lot of CPU instructions <span style="color:#f7007f;"><b>overhead</b></span> due to the integration with the Kernel Scheduler.
{:.warning}

If the CS is short, then the overhead of using mutexes might be <span style="color:#f77729;"><b>more</b></span> than the time taken to "spin" when using a spinlock. It is up to the programmer which one to use for their purposes.

`pthread_mutex_t` is typically used among threads in the **same process**. The mutexes won't be valid across processes after `fork` unless we **put** them in **shared memory** and we explicitly initialize the mutex attribute with `pthread_mutexattr_setpshared` set to `PTHREAD_PROCESS_SHARED`. Semaphore is more commonly used to synchronise between processes.
{: .note}

# Semaphores

The semaphore can be seen as a <span style="color:#f77729;"><b>generalised mutex lock</b></span>. It provides <span style="color:#f77729;"><b>mutual exclusion</b></span>.

It is implemented at the <span style="color:#f77729;"><b>kernel</b></span> level, meaning that its execution requires the calling process to change into the kernel mode via trap (SVC) semaphore instructions. Upon initialisation, the semaphore **descriptor** is **inherited** across a `fork`.

A parent process can **create** a semaphore, **open** it, and `fork`. The child process does not need to open the semaphore and can close the semaphore if the application is finished with it. Note that this means you need to **destroy** a semaphore (just like shared memory) after you're completely done with it (no other processes or threads using it). You can type the command `ipcs -s` to see semaphores created in your system.

This is a high-level <span style="color:#f77729;"><b>software</b></span> solution that relies on <span style="color:#f7007f;"><b>synchronization hardware</b></span> (like those special atomic instructions), and is considered a more <span style="color:#f77729;"><b>robust</b></span> tool than mutex lock.

Peterson’s solution and the hardware-assisted solutions all <span class="orange-bold">require busy waiting</span>. However using semaphore, you can express a solution to the CS problem <span class="orange-bold">without</span> busy waiting.
{:.info}

## Definition

Semaphore is defined as:

- An <span style="color:#f77729;"><b>integer variable</b></span> that is maintained in the kernel, initialized to a certain value that represents the number of <span style="color:#f77729;"><b>available</b></span> resources for the sharing processes
- Is accessed only through two standard <span style="color:#f77729;"><b>atomic</b></span> System Call operations: `wait()` and `signal()`.

Equivalently, we have `acquire()` and `release()` too in some other libraries. `acquire()` is the same as `wait()` and `release()` is the same as `signal()`.
{:.note}

Since calls to `signal()` or `wait()` for semaphores must be performed atomically, they are often implemented using one of the synchronization hardware mechanisms (to support multiple cores synchronization) or software mutual exclusion algorithms such as Peterson’s algorithm, when restriction applies.

## Usage

Semaphore can be used in two ways:

- <span style="color:#f77729;"><b>Binary</b></span> semaphore: integer value can be a `0` or `1`.
- <span style="color:#f77729;"><b>Counting</b></span> semaphore: integer can take any value between `0` to `N`.

## Implementation

How semaphores avoid busy waiting (well, _mostly_): it integrates implementation with CPU scheduler, hence the need to make system calls throughout. It will put the calling process that attempts to `acquire()` on suspension if the semaphore is not currently available.

There are two common CPU scheduling operations (System Calls): `block()` and `wakeup()`. Both are used to implement `wait()` and `signal()` semaphore functions.

- Whenever the process/thread needs to `wait()`, it <span style="color:#f77729;"><b>reduces</b></span> the semaphore and if the current semaphore value is negative, it blocks itself.
- It is added to a <span style="color:#f77729;"><b>waiting queue</b></span> of processes/threads associated with that semaphore. This is done through the system call `block()` on that process.

```cpp
wait(semaphore *S)
{  S->value--;
   if (S->value < 0)
   {
       add this process to S->list; // this will call block()
   }
}
```

- When a process is completed it calls a `signal()` function and one process in the queue is resumed, by using the `wakeup()` system call.

```cpp
signal(semaphore *S)
{
   S->value++;
   if (S->value <= 0)
   {
       remove a process P from S->list;
       wakeup(P);
   }
}
```

Further notes about the above simple implementation of Semaphore `wait` and `signal` System Calls:

- Semaphore values may be <span style="color:#f77729;"><b>negative</b></span>, but this is typically <span style="color:#f77729;"><b>hidden</b></span> from user
  - On the surface, semaphore values are never negative under the classical definition of semaphores with busy waiting.
- If the semaphore value is <span style="color:#f77729;"><b>negative</b></span>, <span style="color:#f7007f;"><b>its magnitude</b></span> is the number of processes waiting on that semaphore
- The list (a queue of processes waiting to acquire the semaphore) can be easily implemented by a link field in each process control block (PCB).
  - Each semaphore data structure for example, can contain an <span style="color:#f77729;"><b>integer</b></span> value and a <span style="color:#f77729;"><b>pointer to a list of PCBs.</b></span>

Note that semaphore implementation may vary between different libraries, but the idea remains the same.
{:.warning}

## Circular Dependency?

How can we implement `signal()` and `wait()` atomically <span style="color:#f77729;"><b>without</b></span> busy waiting, if it relies on synchronization hardware in multiprocessor systems or even basic software mutex algorithms (e.g: if on uniprocessor systems) that <span style="color:#f77729;"><b>requires</b></span> busy waiting?
{:.highlight}

The answer is that semaphore <span style="color:#f7007f;"><b>DOES NOT</b></span> completely eliminates busy waiting. Specifically, it <span style="color:#f7007f;"><b>ONLY busy waits in the critical section of semaphore</b></span> function itself: `wait()`, `signal()` that is relatively <span style="color:#f7007f;"><b>SHORT</b></span> if implemented properly. We do <span class="orange-bold">NOT</span> busy-wait in the CS of the program itself (which can be considerably longer).

In practice, the critical section of the `wait()` and `signal()` implementation of the semaphore in the library is almost never occupied (meaning it’s rare that two processes or threads are making the same wait and signal call on the same semaphore), and busy waiting occurs rarely, or if it does happen, it happens for only a short time.
{:.note}

In this specific situation, busy waiting is <span style="color:#f77729;"><b>completely acceptable</b></span> and still remains efficient.

## Applying Semaphore to MPC Problem

Now that we know how semaphore works, it’s useful to think about how they can be applied to tackle the <span style="color:#f77729;"><b>multiple</b></span> <span style="color:#f77729;"><b>producer-consumer</b></span> (MPC in short) problem that we analyze earlier in this section above.

The pseudocode below illustrates the idea on how the semaphore can be used to replace `counter`, and <span style="color:#f77729;"><b>protect</b></span> the consumer/producer code against multiple (more than 1 of each) consumer/producer threads racing to execute their respective instructions, and sharing resources:

- write index `in` (shared among consumer processes)
- read index `out` (shared among producer processes)
- `char buf[N]` (shared among all processes)
- **Semaphores**: two counting semaphores, `chars` and `space` to keep track of free space and number of characters in the buffer, and two mutex semaphores, `mutex_p` and `mutex_c` to provide mutex among producers and among consumers

Shared resources:

```cpp
char buf[N];
int in = 0; int out = 0;

semaphore chars = 0;
semaphore space = N;
semaphore mutex_p = 1;
semaphore mutex_c = 1;
```

Producer program:

```cpp
void send (char c){
   wait(space);
   wait(mutex_p);

   buf[in] = c;
   in = (in + 1)%N;

   signal(mutex_p);
   signal(chars);
}
```

Consumer program:

```cpp
char rcv(){
   char c;
   wait(chars);
   wait(mutex_c);

   c = buf[out];
   out = (out+1)%N;

   signal(mutex_c);
   signal(space);
}
```

# Condition variables

Condition variables allow a process or thread to wait for completion of a given <span style="color:#f7007f;"><b>event</b></span> on a particular object (some shared state, data structure, anything). It is used to <span style="color:#f77729;"><b>communicate</b></span> between processes or threads when certain conditions become `true`.

The "event" is the _change_ in state of some condition that thread is interested in. Until that is satisfied, the process waits to be awakened later by a signalling process/thread (that actually <span style="color:#f77729;"><b>changes</b></span> the condition).

Condition variables are and <span style="color:#f77729;"><b>should</b></span> always be implemented with <span style="color:#f77729;"><b>mutex</b></span> locks. When implemented properly, they provide <span style="color:#f7007f;"><b>condition synchronization</b></span>.
{:.error}

For example, we can initialize a mutex guarding certain CS:

```cpp
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
```

And in this example, we assume that a particular CS in Process/Thread 2 cannot be executed if `bool cond_x == false`. Therefore we can create a condition to represent this:

```cpp
pthread_cond_t p_cond_x = PTHREAD_COND_INITIALIZER;
```

Now consider Process/Thread 1 instructions:

```cpp
pthread_mutex_lock(&mutex);
// CRITICAL SECTION
// ...
cond_x = true;
pthread_cond_signal(&p_cond_x);
pthread_mutex_unlock(&mutex);
```

... and Process/Thread 2 instructions:

```cpp
pthread_mutex_lock(&mutex);
while (cond_x == false){
   pthread_cond_wait(&p_cond_x, &mutex);  // yields mutex, sleeping
}
// CRITICAL SECTION, can only be executed iff cond_x == true
// ...
pthread_mutex_unlock(&mutex);
```

It is clear that the `mutex` guards the CS. However, Process 2 should <span style="color:#f77729;"><b>not</b></span> proceed to its CS if `cond_x` is `false` in this example.

Process 2 can be put to `sleep` by calling `condition_wait()`; the implementation of this function is integrated into the process scheduler which does the following:

- It <span style="color:#f77729;"><b>gives</b></span> up the mutex <span style="color:#f77729;"><b>AND</b></span>
- <span style="color:#f77729;"><b>Sleeps</b></span>, will not busy wait

When Process 1 has set the variable `cond_x` into `true`, it signals Process 2 to <span style="color:#f77729;"><b>continue</b></span> before <span style="color:#f77729;"><b>giving</b></span> up the mutex.

- It is important to call `wait()` after acquiring the mutex and checking the condition.
  - You cannot use `unlock()` to release the mutex.
- When Process 2 `waits` and eventually sleep, it will <span style="color:#f77729;"><b>give</b></span> up the lock so as to allow other threads needing the lock to proceed.
- When Process 2 has woken up from the `wait()`, it <span style="color:#f77729;"><b>automatically</b></span> reacquires the mutex lock
  - If Process 1 hasn’t given up the lock, Process 2 will not execute although `cond_x` has been fulfilled

This is crucial because it will re-check the state of `cond_x` again before continuing to the CS. It is also crucial to re-check `cond_x` before continuing to the CS. <span style="color:#f77729;"><b>Why?</b></span>
{:.error}

# Final Note

A condition variable is effectively a <span style="color:#f77729;"><b>signalling</b></span> mechanism under the context of a given mutex lock. With mutex lock alone, we cannot easily block a process out of its CS based on any <span style="color:#f77729;"><b>arbitrary condition</b></span> even when the mutex is <span style="color:#f77729;"><b>available</b></span>.

By default, condition variables and mutexes cannot be shared across processes. However, you can use condition variables **across** processes if you make the condition variable **process-shared**, using condition variable attribute configured with the `pthread_condattr_setpshared` function and a value of `PTHREAD_PROCESS_SHARED`. You will also have to make the **associated** mutex process-shared, using a mutex attribute configured with `pthread_mutexattr_setpshared`.

Here's an excerpt from the Linux `man` page:

> The process-shared attribute is set to `PTHREAD_PROCESS_SHARED` to permit a condition variable to be operated upon by any thread that has access to the memory where the condition variable is allocated, even if the condition variable is allocated in memory that is shared by multiple processes. If the process-shared attribute is `PTHREAD_PROCESS_PRIVATE`, the condition variable shall only be operated upon by threads created within the same process as the thread that initialized the condition variable; if threads of differing processes attempt to operate on such a condition variable, the behavior is undefined. The **default** value of the attribute is `PTHREAD_PROCESS_PRIVATE`.

An example skeleton code for mutex and condition variables shared among processes is as follows:

```c
/***** MUTEX *****/
pthread_mutex_t * pmutex = NULL;
pthread_mutexattr_t attrmutex;

/* Initialise attribute to mutex */
pthread_mutexattr_init(&attrmutex);
pthread_mutexattr_setpshared(&attrmutex, PTHREAD_PROCESS_SHARED);

/* Allocate SHARED memory to pmutex here. */

/* Initialise mutex. */
pthread_mutex_init(pmutex, &attrmutex);

/***** CONDITION VARIABLE *****/
pthread_cond_t * pcond = NULL;
pthread_condattr_t attrcond;

/* Initialise attribute to condition */
pthread_condattr_init(&attrcond);
pthread_condattr_setpshared(&attrcond, PTHREAD_PROCESS_SHARED);

/* Allocate memory to pcond here */

/* Initialise condition */
pthread_cond_init(pcond, &attrcond);

/* Use the mutex */
/* Use the condition */

/* Clean up when done */
pthread_cond_destroy(pcond);
pthread_condattr_destroy(&attrcond);
pthread_mutex_destroy(pmutex);
pthread_mutexattr_destroy(&attrmutex);
```

# Summary

This chapter discusses the essentials of synchronization between processes and threads, a critical aspect of operating systems. It covers various synchronization mechanisms including mutexes, semaphores, and condition variables to manage access to **shared** resources and **prevent** race conditions. The text also explores **practical** scenarios like the **producer-consumer problem**, illustrating how synchronization prevents data inconsistencies and ensures orderly execution. 


Key learning points include:
- **Producer-Consumer Problem**: Explains synchronization needs between two processes using a shared buffer.
- **Race Conditions**: Discusses how improper access and manipulation of shared data can lead to unpredictable outcomes.
- **Critical Section Problem**: Introduces solutions for managing access to shared resources, ensuring system stability and correctness.
- **Synchronization Techniques**: Covers software and hardware methods, including mutexes, semaphores, and condition variables, illustrating their roles in preventing deadlock and ensuring orderly execution of processes.


# Appendix

## Peterson's in Modern CPUs 


This section is not required in our syllabus. It's written to satisfy some curiosity that may arise due to the specific requirements of Peterson's solution: atomic LD/ST, and used in single-core CPU. Only proceed to read these sections if you are prepared to feel 🤯. Otherwise, skip to Synchronisation Hardware.
{:.error}

Try compiling Peterson's in plain C, and it will <span style="color:#f77729;"><b>NOT</b></span> work in modern architecture without:

1. Ensuring atomic LD/ST
2. Ensuring cache coherency in multicore systems
3. Applying memory barriers to enforce <span style="color:#f77729;"><b>sequential consistency</b></span> (not mentioned above to not overcomplicate the syllabus)

First of all, let's begin with a definition of <span style="color:#f77729;"><b>atomicity</b></span>:

When an atomic store is performed on a shared variable, _no other thread_ can observe the modification half-complete. _“Atomic” in this context means “all or nothing”_. When an atomic load is performed on a shared variable, it reads the entire value as it appeared at a single moment in time. Non-atomic loads and stores do not make those guarantees. You can read more about it [here](https://preshing.com/20130618/atomic-vs-non-atomic-operations/).
{:.info}

Note that atomicity does not guarantee you to get the value _most recently written_ by any thread. If you always want to read the most recently written value, then that means you want [<span class="orange-bold">strict consistency</span>](https://en.wikipedia.org/wiki/Consistency_model#Strict_consistency), and strict consistency is a more _strict_ version of sequential consistency (not relevant here).

### Non-atomic example

Not all `LD`/`ST` instructions are guaranteed to be atomic, we take this for granted. To understand this better, let's use an analogy. Consider a scenario of a non-atomic database containing student grades that were initialised to `null` and these two actions performed:

1. Professor A is keying in the grade of student `I`. Total grade is 75, but he did it via multiple steps: key in `5` (LSB), then key in `7` (MSB). This storing of student grade is non-atomic (requires two steps that can be preempted)
2. Professor B realised that the student's grade is not 75 but 91, so he would like to update the grade of student `I`. He did it via multiple steps as well: key in `1` (LSB) then `9` (MSB)

By right, the grade of student `I` should be 91. However, since the database is not atomic, Professor B action might preempt the intermediary value stored by Professor A (which means the update done by Professor A is not complete yet):

1. Professor A update the LSB: `5`, grade of the student is `05` now.
2. Professor A second update is preempted, Professor B update the LSB: `1`, grade of the student is `01` now
3. Professor B update the MSB `9`, grade of the student is `91` now.
4. Professor A update the MSB: `7`, grade of the student is `71` now.

Student `I` end up having a grade of 71 instead of 91 (got a C instead of an A!). This grade 9` _does not belong to anybody_, it's just a catastrophe resulted from non-atomic store made by Professor A.

### Torn Write

What happened?
{:.highlight}

The `STORE` done by Professor A is not `ATOMIC`. Professor B indeed executed his action _after_ Professor A, but didn't realise that Professor A has not finished. Therefore the end state of the student grade is <span style="color:#f77729;"><b>neither</b></span> the grades that are meant to be set by either professor (neither `75` nor `91`). This is known as <span style="color:#f77729;"><b>torn write</b></span>.

In Peterson's solution, both processes might attempt to `STORE` the value of `turn` in an interleaved fashion (set `turn=i`, and `turn=j`). With atomic `STORE`, we need to be entirely sure that the value stored in `turn` is ENTIRELY `j`, or ENTIRELY `i`, and not some undefined behavior, especially when `turn=j` and `turn=i` in both Process i and j are executed near each other, concurrently. If `turn` is set to be neither `i` or `j` due to non-atomic store, mutex guarantee is lost.

### Torn Read

The same must happen with the `LOAD`. Let's go back to our grade analogy.

Suppose we have Professor C reading the value of student `I` grade: read `LSB` then read `MSB` (non atomic). It is posssible that Professor C reads the student's grade as `95`.

1. Professor A update the LSB: `5`, grade of the student is `05` now.
2. <span style="color:#f77729;"><b>Meanwhile, Professor C read the `LSB`</b></span> grade of the student: `5`
3. Professor A second update is preempted, Professor B update the LSB: `1`, grade of the student is `01` now
4. Professor B update the MSB `9`, grade of the student is `91` now.
5. <span style="color:#f77729;"><b>Meanwhile, Professor C read the `MSB`</b></span> grade of the student: `9`
6. Professor A update the MSB: `7`, grade of the student is `71` now.

Professor C read <span class="orange-bold">neither</span> the old value of the student grade: `75` or the new value of student grade: `91`. `95` is the result of a <span style="color:#f77729;"><b>torn read</b></span>.

In Peterson's, two instructions: `turn==j` (`LOAD`) and `turn=i` (`STORE`) must complete atomically. We do not want to read some "neither `i` nor `j`" value in `turn`because as we were _loading_ the value of `turn`, the other process modifies it.

In the past, we have an 8-bit or even 16-bit architecture. Any `LOAD` or `STORE` involving more than supported bits will result in non-atomic operations by default. We don't have to worry about this anymore nowadays in our 64-bit architecture, unless our values [are not boundary aligned](https://rigtorp.se/isatomic/).
{:.note}

## Sequential Consistency

This is actually another requirement for Peterson's Solution to work, but omitted in the section above to simplify our syllabus. <span style="color:#f77729;"><b>Sequential consistency</b></span> means (simplified):

The operations of each individual processor appear in a specific sequence in the order specified by its program, and will not be re-ordered by the compiler or executed out-of-order by the CPU.
{:.info}

The Peterson's Algorithm <span style="color:#f77729;"><b>also required</b></span> that the <span style="color:#f77729;"><b>order</b></span> of the instructions executed is <span style="color:#f77729;"><b>consistent</b></span>,

1. Setting of `flag[i] = True` must happen <span style="color:#f77729;"><b>before</b></span> for `turn = j` for Process i
2. Setting of `flag[j] = True` must happen <span style="color:#f77729;"><b>before</b></span> for `turn = i` for Process j
3. Step (1) and (2) must precede the `LOAD` access in the `while` line.

Not all instructions are executed _in order_. A compiler may try to be _smart_ and recompile the Peterson's solution for Process i as follows, reordering the instructions completely since they involve access to different memory location. One possible reordering is: 

```cpp
// Process i code
turn = j 
while(flag[j] == True and turn == j); // LOAD from flag[j]
flag[i] = True  // STORE to flag[i]
// CS...
// ...
flag[i] = False
```

The above violates **mutual exclusion**:
1. Suppose Pi has just finished setting `turn = j`, goes to the `while` check and failed the check, hence proceeding to store to `flag[i]` but was context switched before being able to do so 
2. Pj starts, and does the same: sets `turn=i`, fails the `while` check because at this point nobody sets the flags yet, and then sets `flag[j] = True`, goes to CS and then context switched 
3. When Pi is resumed, it *has already* passed the `while` check, hence both processes are now in the CS

Another possibility is:

```cpp
// Process i code
turn = j 
flag[i] = True  // STORE to flag[i]
while(flag[j] == True and turn == j); // LOAD from flag[j]
// CS...
// ...
flag[i] = False
```

The above violates **mutual exclusion** too: 
1. Suppose Pi has *just* set `turn=j` and then context switched 
2. Suppose Pj then resume and set `turn=i, flag[j]=True`, then proceed to `while` check. The check returns `False` because `flag[i]` is still `False`
3. Pj then enters CS and context switched 
4. Now Pi is resumed, sets `flag[i]=True` (at this point turn *is* `i` due to step 2 above), goes to the `while` check which also returns `False`, enabling Pi to enter the CS as well while Pj is still in it

On <span style="color:#f77729;"><b>modern</b></span> operating system where you have multiple processors, the <span style="color:#f77729;"><b>order</b></span> of `LOAD` and `STORE` instructions can change if these instructions are not dealing with same memory addresses. You can obviously find out why the above is disastrous.
{: .info}

In order to prevent this, you must implement [memory barrier](https://en.wikipedia.org/wiki/Memory_barrier). You can look at [this post](https://bartoszmilewski.com/2008/12/01/c-atomics-and-memory-ordering/) to learn how this is done in C++ as an example.

### Optimising local variables (e.g: caching in Register)

Also, a compiler may decide that it is unnecessary to read the same variable more than once, especially if they don't change during the entire lifetime of the program. It may create a snapshot copy of `flag[j]` in the register local to Process i, because how could it possibly change if there are no _store_ operations in between?

The store to `flag[j]` is done in Process j (unknown to Process i). As a result, `flag[j]` in Process i may always be `False`.

For instance, assume that Process i is initially scheduled (initialised), and immediately suspended. Then, Process j is scheduled until it reached its CS before preempted. When Process i is scheduled and check `flag[j]`, this will return `False` because its compiler decided to optimise the instructions and make a copy in Process i's register on the value of the old `flag[j]` upon initialisation. As a result, Process i also enter the CS and mutex is violated.

In other words, there are _three_ copies of `flag[j]` --> one in Process i's register, one in Process j's register, and another in the RAM. The value of `flag[j]` is therefore <span style="color:#f77729;"><b>not consistent</b></span>.
{:.note}

You can use the `volatile` keyword in C to let the compiler know that it is possible for a variable's value to be <span style="color:#f77729;"><b>changed</b></span> by instructions in _another_ program, or by another Thread executing the same set of instructions, and therefore disable the above optimisations because it is no longer safe to do so.

Objects that are declared as <span class="orange-bold">volatile</span> are <span class="orange-bold">not</span> used in certain optimizations because their values can change at any time. The system always reads the <span style="color:#f77729;"><b>current</b></span> value of a volatile object when it is requested, even if a previous instruction asked for a value from the same object. Also, the value of the object is <span style="color:#f77729;"><b>written</b></span> immediately on assignment.

For example:

```cpp
volatile int turn;
volatile int flag[2] = {0,0}
```

## Cache Coherency

Attempting to run Peterson's in multiple core requires <span style="color:#f77729;"><b>cache coherency</b></span>. Cores have individual caches, and for performance efficiency, each process might cache `turn` and `flag` value in its individual caches. This violates <span style="color:#f77729;"><b>sequential consistency</b></span> requirement explained above -- there are multiple copies of `turn` and `flag` with different values. We can try to synchronise between the caches but it comes at a <span style="color:#f77729;"><b>huge</b></span> performance cost (impractical).

## Final Caveats for Peterson's Solution

In summary, Peterson’s solution rests on the assumption that the instructions are executed in a <span style="color:#f77729;"><b>particular</b></span> order (sequential consistency) and memory accesses can be achieved atomically. Both of these assumptions can <span style="color:#f77729;"><b>fail</b></span> with modern hardware. Due to complexities in the design of <span style="color:#f77729;"><b>pipelined</b></span> CPUs, the instructions may be executed in a _different_ order (called [out-of-order](https://en.wikipedia.org/wiki/Out-of-order_execution) execution). Additionally, if the threads are running on different cores that do not guarantee immediate <span style="color:#f77729;"><b>cache coherency</b></span>, the threads may be using <span style="color:#f77729;"><b>different</b></span> memory values.

In fact, Peterson's algorithm cannot be implemented correctly in C99, as explained in [this article](http://bartoszmilewski.com/2008/11/05/who-ordered-memory-fences-on-an-x86/). We need to make sure that we use a sequentially consistent memory order and compilers do not perform additional optimisation hoisting[^2] or sinking[^3][^4] load and stores. Also, we need to make sure that the CPU hardware itself does <span style="color:#f7007f;"><b>not perform out-of-order execution</b></span>.

End of brain-tearing sections.
{:.info}

## Sample C Code
### Mutex

In the example below, we attempt to increase a shared variable `counter`, guarded by a `mutex` to prevent race conditions.

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

void *functionC();
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
int counter = 0;

int main()
{
  int rc1, rc2;
  pthread_t thread1, thread2;

  /* Create independent threads each of which will execute functionC */

  if ((rc1 = pthread_create(&thread1, NULL, &functionC, NULL)))
  {
     printf("Thread 1 creation failed: %d\n", rc1);
  }

  if ((rc2 = pthread_create(&thread2, NULL, &functionC, NULL)))
  {
     printf("Thread 2 creation failed: %d\n", rc2);
  }

  // Main thread waits until both threads have finished execution

  pthread_join(thread1, NULL);
  pthread_join(thread2, NULL);

  return 0;
}

void *functionC()
{
  pthread_mutex_lock(&mutex);
  counter++;
  printf("Counter value: %d\n", counter);
  pthread_mutex_unlock(&mutex);
}
```

### Condition Variables

Consider a main function with a shared variable `count`, two `mutexes` and one `condition`:

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

pthread_mutex_t count_mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t condition_mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t condition_cond = PTHREAD_COND_INITIALIZER;

void *functionCount1();
void *functionCount2();
int count = 0;
#define COUNT_DONE 10
#define COUNT_HALT1 3
#define COUNT_HALT2 6

int main()
{
  pthread_t thread1, thread2;

  pthread_create(&thread1, NULL, &functionCount1, NULL);
  pthread_create(&thread2, NULL, &functionCount2, NULL);
  pthread_join(thread1, NULL);
  pthread_join(thread2, NULL);

  return 0;
}
```

Suppose an example where we Thread 1 running `functionCount1()` is to be <span style="color:#f77729;"><b>halted</b></span> whenever the `count` value is between 3 (`COUNT_HALT1`) and 6 (`COUNT_HALT2`).

- Otherwise, either thread can increment the counter.
- We can use the condition variable and condition wait to ensure this behavior in `functionCount1()`:

```cpp
void *functionCount1()
{
  for (;;) // equivalent to while(true)
  {
     pthread_mutex_lock(&count_mutex);
     while (count >= COUNT_HALT1 && count <= COUNT_HALT2)
     {
        pthread_cond_wait(&condition_cond, &count_mutex);
     }

     count++;
     printf("Counter value functionCount1: %d\n", count);
     pthread_mutex_unlock(&count_mutex);

     if (count >= COUNT_DONE)
        return (NULL);
  }
}
```

We can then use `cond_signal` in `functionCount2()` executed by Thread 2:

```cpp
void *functionCount2()
{
  for (;;) // equivalent to while(true)
  {
     pthread_mutex_lock(&count_mutex);
     if (count < COUNT_HALT1 || count > COUNT_HALT2)
     {
        pthread_cond_signal(&condition_cond);
     }
     count++;
     printf("Counter value functionCount2: %d\n", count);
     pthread_mutex_unlock(&count_mutex);

     if (count >= COUNT_DONE)
        return (NULL);
  }
}
```

### Producer-Consumer Problem

In this sample, we try to tackle the single producer single consumer problem with counting semaphore. The shared resources will be an integer array named `buffer` in this example, of size `10`.

The idea is to ensure that producer <span style="color:#f77729;"><b>does not overwrite</b></span> unconsumed values, and to ensure that consumer <span style="color:#f77729;"><b>does not consume anything before</b></span> producer puts anything new into the buffer.

Main process to initialise Producer and Consumer Threads:

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>


#define RESOURCES 10
#define REPEAT 100

sem_t *blank_space;
sem_t *content;

int buffer[RESOURCES];
int write_index = 0;
int read_index = 0;

int main()
{
   // instantiate named semaphore, works on macOS
   // sem_t *sem_open(const char *name, int oflag,
   //                   mode_t mode, unsigned int value);
   // @mode: set to have R&W permission for owner, group owner, and other user
   blank_space = sem_open("blank_space", O_CREAT,
                          S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP | S_IROTH | S_IWOTH,
                          RESOURCES);
   printf("%p \n", (void *)blank_space);
   if (blank_space == (sem_t *)SEM_FAILED)
   {
       printf("Sem Open Failed.\n");
       exit(1);
   }
   content = sem_open("content", O_CREAT,
                      S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP | S_IROTH | S_IWOTH,
                      0);
   printf("%p \n", (void *)content);
   if (content == (sem_t *)SEM_FAILED)
   {
       printf("Sem Open Failed.\n");
       exit(1);
   }

   pthread_t producer, consumer;
   pthread_create(&producer, NULL, producer_function, NULL);
   pthread_create(&consumer, NULL, consumer_function, NULL);

   printf("Joining threads\n");
   pthread_join(producer, NULL);
   pthread_join(consumer, NULL);

   // if you don't destroy, it persists in the system
   // run the command: ipcs -s
   // to remove: ipcrm -s <sem_id>
   sem_unlink("blank_space");
   sem_unlink("content");
   return 0;
}
```

Producer Thread instruction:

```cpp
void *producer_function(void *arg)
{
   for (int i = 0; i < REPEAT; i++)
   {
       // wait
       sem_wait(blank_space);
       // write to buffer
       buffer[write_index] = i;
       // advance write pointer
       write_index = (write_index + 1) % RESOURCES;
       // signal
       sem_post(content);
   }

   return NULL;
}
```

Consumer Thread instruction:

```cpp
void *consumer_function(void *arg)
{
   for (int i = 0; i < REPEAT; i++)
   {
       // wait
       sem_wait(content);
       // read from buffer
       int value = buffer[read_index];
       printf("Consumer reads: %d \n", value);
       // advance write pointer
       read_index = (read_index + 1) % RESOURCES;
       // signal
       sem_post(blank_space);
   }

   return NULL;
}
```

Paste the two functions above before `main()`. After you compile and run the code, you should have an output as such where consumer thread nicely prints out the numbers put into the buffer by producer in sequence (and stops at 100):

<img src="{{ site.baseurl }}/assets/images/week4/2.png"  class="center_fifty no-invert"/>

### Cleanup

To remove unused semaphores, shared memory, and message queues in your system created by you, use the bash script:

```
#!/bin/bash
ipcs -m | grep `whoami` | awk '{ print $2 }' | xargs -n1 ipcrm -m
ipcs -s | grep `whoami` | awk '{ print $2 }' | xargs -n1 ipcrm -s
ipcs -q | grep `whoami` | awk '{ print $2 }' | xargs -n1 ipcrm -q
```

Don't forget to `chmod +x [your-bash-script-name].sh` before executing it.
{: .note}

## Share Spinlock Between Processes

To share a `pthread_spinlock_t` outside the process, you need to use the `PTHREAD_PROCESS_SHARED` (this value is `1`) option during **initialization**. This allows the spinlock to be used in a **shared** memory region accessible by multiple processes.

Here is how you can do it:

1. Initialize the spinlock with `PTHREAD_PROCESS_SHARED`.
2. **Allocate** the spinlock in a shared memory segment.

Here's an example using POSIX shared memory:

### Step-by-Step Example


```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <sys/mman.h>
#include <pthread.h>
#include <unistd.h>

#define SHM_NAME "/my_shared_memory"

int main() {
    int shm_fd;
    pthread_spinlock_t *spinlock;

    // Create shared memory object
    shm_fd = shm_open(SHM_NAME, O_CREAT | O_RDWR, 0666);
    if (shm_fd == -1) {
        perror("shm_open");
        exit(EXIT_FAILURE);
    }

    // Set size of the shared memory object
    if (ftruncate(shm_fd, sizeof(pthread_spinlock_t)) == -1) {
        perror("ftruncate");
        exit(EXIT_FAILURE);
    }

    // Map shared memory object
    spinlock = mmap(NULL, sizeof(pthread_spinlock_t), PROT_READ | PROT_WRITE, MAP_SHARED, shm_fd, 0);
    if (spinlock == MAP_FAILED) {
        perror("mmap");
        exit(EXIT_FAILURE);
    }

    // Initialize spinlock with PTHREAD_PROCESS_SHARED
    if (pthread_spin_init(spinlock, PTHREAD_PROCESS_SHARED) != 0) {
        perror("pthread_spin_init");
        exit(EXIT_FAILURE);
    }

    // Use the spinlock in the parent process
    pthread_spin_lock(spinlock);
    printf("Parent: Acquired the spinlock\n");

    // Fork a child process to demonstrate sharing
    pid_t pid = fork();
    if (pid == -1) {
        perror("fork");
        exit(EXIT_FAILURE);
    } else if (pid == 0) {
        // Child process
        if (pthread_spin_lock(spinlock) == 0) {
            printf("Child: Acquired the spinlock\n");
            pthread_spin_unlock(spinlock);
        } else {
            perror("pthread_spin_lock");
        }
        exit(EXIT_SUCCESS);
    }

    // Wait for the child to finish
    wait(NULL);

    // Release the spinlock in the parent process
    pthread_spin_unlock(spinlock);

    // Cleanup
    pthread_spin_destroy(spinlock);
    munmap(spinlock, sizeof(pthread_spinlock_t));
    shm_unlink(SHM_NAME);

    return 0;
}
```

Detailed **steps**: 
1. **Creating Shared Memory:**
   - `shm_open`: Creates or opens a shared memory object.
   - `ftruncate`: Sets the size of the shared memory object.
   - `mmap`: Maps the shared memory object into the process's address space.

2. **Initializing the Spinlock:**
   - `pthread_spin_init`: Initializes the spinlock with `PTHREAD_PROCESS_SHARED`, allowing it to be shared between processes.

3. **Using the Spinlock:**
   - The parent process locks the spinlock.
   - A child process is created using `fork`.
   - The child process attempts to lock the spinlock. If successful, it demonstrates that the spinlock is shared.

4. **Cleanup:**
   - The spinlock is unlocked and destroyed.
   - The shared memory is unmapped and unlinked.

This example demonstrates how to share a spinlock between processes using shared memory. The child process is able to lock and unlock the same spinlock initialized by the parent process.
<hr>

[^1]: An operation acting on shared memory is atomic if it completes in a single step relative to other threads. For example, when an atomic store is performed on a shared variable, no other thread/process can observe the modification half-complete.
[^2]: You know this as a non-preemptive approach, and some kernels are non-preemptive (non-interruptible) and therefore will not face the race condition in the kernel level itself.
[^3]: Loop-invariant expressions can be hoisted out of loops, thus improving run-time performance by executing the expression only once rather than at each iteration. Example can be found [here](https://compileroptimizations.com/category/hoisting.htm).
[^4]:Code Sinking is a term for a technique that reduces wasted instructions by moving instructions to branches in which they are used.