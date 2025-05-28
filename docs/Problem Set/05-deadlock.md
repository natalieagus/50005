---
layout: default
permalink: /ps/5-deadlock
title: Deadlock
parent: Problem Set 
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
**Natalie Agus (Summer 2025)**

# Deadlock
{: .no_toc}



## Visual Trace Exercise: Deadlock When Full

### Background

In producer-consumer systems using semaphores, the **order in which semaphores are acquired** is critical. If a thread holds one semaphore while waiting on another, and other threads are waiting to acquire the first one, the system can enter a **deadlock**: a circular wait where no thread can proceed.

Let's demonstrate how a subtle change in ordering can cause the system to freeze when the buffer becomes full.


### Scenario

You attempt to optimize a bounded buffer implementation by locking access early. You reason: “Since the thread will use both `mutex` and `empty`, let’s acquire `mutex` first to avoid holding `mutex` too long after.”

Here’s the producer code:

```c
sem_wait(&mutex);
sem_wait(&empty);
// insert into buffer
sem_post(&full);
sem_post(&mutex);
```

At runtime, the system works for a while. But occasionally, when the buffer becomes full, all producers block indefinitely, even though consumers are still running.


**Answer the following questions:**
1. What is the mistake in the order of semaphore operations?
2. Show how deadlock can occur if one producer holds `mutex` and is blocked on `empty`, while other threads are waiting for `mutex`.
3. Why does this problem only show up under full buffer conditions?
4. How should the semaphore order be fixed to prevent deadlock?
5. Why is it safe to call `sem_wait(&empty)` before locking the mutex?


{:.highlight}
> **Hints:**
> * Think about the scenario where the buffer has no more empty slots.
> * A thread holding `mutex` must never block on another semaphore.
> * Blocking must happen *before* acquiring exclusive access.


### Visual Timeline Example

```text
Initial state: buffer is full (empty = 0)

T1 (Producer):
    sem_wait(mutex) → succeeds (mutex = 1 → 0)
    sem_wait(empty) → blocks (empty = 0)

T2 (Producer):
    sem_wait(mutex) → blocks (mutex = 0)

T3 (Consumer):
    sem_wait(full) → succeeds
    tries to acquire mutex → blocks

→ All threads waiting on each other → deadlock
```


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
The mistake is that the producer acquires <code>mutex</code> before checking whether there is space in the buffer using <code>sem_wait(&empty)</code>. If the buffer is full, the thread blocks on <code>empty</code> while still holding the mutex.
</p>

<p>
This causes deadlock because other threads (e.g. consumers) need to acquire the mutex to make progress but they can't, since it's held by a thread that is blocked. This creates a circular wait.
</p>

<p>
This issue only arises when the buffer is full (<code>empty == 0</code>). If there is space, <code>sem_wait(&empty)</code> returns immediately, and the problem doesn’t occur. But under high load, the blocking behavior becomes visible.
</p>

<p>
To fix this, reverse the order: call <code>sem_wait(&empty)</code> before acquiring the mutex. This way, a thread blocks before it holds any exclusive resources, avoiding circular waits.
</p>

<p>
It's safe to call <code>sem_wait(&empty)</code> before locking the mutex because <code>empty</code> is not protecting the buffer structure. It's only a count of how many slots are available. The mutex is what ensures safe access to shared memory, and should only be held when actually modifying the buffer.
</p>

<p>
Correct order:

<pre>
sem_wait(&empty);
sem_wait(&mutex);
// insert into buffer
sem_post(&mutex);
sem_post(&full);
</pre>
</p>

</p></div><br>
