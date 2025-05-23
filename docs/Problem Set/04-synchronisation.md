---
layout: default
permalink: /ps/3-synchronisation
title: Synchronizing Processes and Threads
parent: Problem Set 
nav_order:  3
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

# Synchronizing Processes and Threads
{: .no_toc}



## Serving Fairly: Lamport’s Bakery Algorithm

### Background

Most mutual exclusion mechanisms rely on hardware support,  e.g., disabling interrupts, atomic instructions like `Test-And-Set`, or kernel-managed mutexes. However, **Lamport’s Bakery Algorithm** is a **pure software-based** solution that requires no atomic hardware and works for **n processes or threads**.

It was proposed by **Leslie Lamport** in 1974 and is inspired by the **“take-a-number” system used in bakeries**:

* Customers take numbered tickets.
* They are served in order based on their number.

The algorithm uses this analogy to enforce fairness and mutual exclusion:

* Each process “takes a number” using a shared array.
* They wait their turn based on number, then process ID.

Lamport designed the algorithm to work under the assumption of a **single-core processor** with **sequential consistency**, meaning memory operations appear in program order to all threads. On modern multicore systems, it may fail unless **memory fences** or barriers are added, due to instruction reordering or cache visibility issues.

It uses shared memory and two arrays:

* `choosing[n]`: indicates if a thread is picking a number.
* `number[n]`: tickets representing order; a higher number means later turn.

The algorithm satisfies the three classical conditions of correct critical section solutions:

* **Mutual Exclusion**
* **Bounded Waiting**
* **Progress**

### Pseudocode

```c
int choosing[N] = {0};  // initially all false
int number[N] = {0};    // initially all zero

void lock(int i) {
    choosing[i] = 1;
    number[i] = 1 + max(number[0..N-1]); // take a number
    choosing[i] = 0;

    for (int j = 0; j < N; j++) {
        while (choosing[j]);  // wait if j is choosing
        while (number[j] != 0 &&
              (number[j] < number[i] ||
              (number[j] == number[i] && j < i)));
    }
}

void unlock(int i) {
    number[i] = 0;  // release turn
}
```

Each thread `i` calls `lock(i)` before entering the critical section and `unlock(i)` after.


### Scenario

You are reviewing Lamport’s Bakery Algorithm as a potential solution for managing access to a shared queue in a bare-metal, multi-threaded runtime with no kernel or atomic hardware support.

To approve its use, you need to verify that it satisfies all the required properties of a correct critical section protocol.

**Answer the following questions**:
1. What is the purpose of the `choosing[]` array?
2. Prove that the Bakery Algorithm ensures **Mutual Exclusion**.
3. Prove that it provides **Bounded Waiting**,  i.e., starvation is not possible.
4. Prove that it satisfies **Progress**,  if no one is in the critical section, some thread can proceed.
5. What are the **hardware or memory assumptions** required for the algorithm to work correctly?



{:.highlight}
> **Hints**:
> * The `choosing[]` array makes the number assignment atomic.
> * Use contradiction to show mutual exclusion.
> * Lexicographic ordering ensures fairness.
> * Think about how a process knows it has the "smallest" ticket.
> * Reflect on issues like cache coherence and memory visibility in multicore systems.


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
The <code>choosing[]</code> array ensures that the number assignment is atomic in effect. If a process is choosing its number, others wait before reading it to avoid inconsistency. Without this, another process might read a partially updated number.
</p>

<p>
<strong>Mutual Exclusion:</strong> Suppose two processes, i and j, are in the critical section at the same time. That would mean both passed the <code>for</code> loop and believed they had priority. But the check <code>number[j] &lt; number[i]</code> or same number with smaller ID guarantees that one must wait. This contradiction shows that both cannot enter simultaneously. Hence, mutual exclusion holds.
</p>

<p>
<strong>Bounded Waiting:</strong> A process i takes a number greater than any currently assigned. Since all other processes eventually leave the critical section and reset their number to 0, process i will see a moment when no one has a smaller ticket, and will proceed. Therefore, no starvation,  bounded waiting is ensured.
</p>

<p>
<strong>Progress:</strong> If no one is in the critical section, there must exist at least one process with the smallest non-zero ticket. That process will not be blocked and will proceed. Hence, there is no deadlock or unnecessary delay,  the system makes progress.
</p>

<p>
<strong>Assumptions:</strong>
<ul>
  <li>All read and write operations to shared memory are atomic (word-level).</li>
  <li>Sequential consistency: writes by one thread are seen in order by all others.</li>
  <li>No compiler or CPU instruction reordering without barriers.</li>
  <li>Typically safe only on single-core systems without explicit memory fences.</li>
</ul>
On modern multi-core systems, memory barriers (fences) must be used to preserve correctness.
</p>
</p></div><br>

