---
layout: default
permalink: /ps/4-synchronisation
title: Synchronizing Processes and Threads
parent: Problem Set 
nav_order:  4
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



## The Bakery Dilemma

### Background

Now that you’ve learned how Lamport’s Bakery Algorithm works and why it satisfies the conditions for correct mutual exclusion, it’s time to evaluate its practical usefulness.

There are many ways to solve the critical section problem, including hardware-supported instructions (e.g., `Test-And-Set`), kernel-level mutexes, spinlocks, and other software solutions like **Peterson’s Algorithm** (for two threads). Each has trade-offs in complexity, scalability, performance, and assumptions.

The Bakery Algorithm is remarkable for being:

* Purely software-based
* Scalable to *n* threads
* Fair and starvation-free

However, its real-world deployment is rare due to hardware assumptions and inefficiencies on modern multi-core architectures.


### Scenario

You are designing synchronization primitives for a small embedded operating system that supports up to 8 concurrent threads. The platform does not support atomic instructions, but it does guarantee sequentially consistent memory (single-core). You implemented Lamport’s Bakery Algorithm successfully, but your teammate suggests replacing it with Peterson’s Algorithm, citing simplicity.

You must defend your design, but also understand when Bakery is or isn’t the better option.

**Answer the following questions:**
1. **Conceptual**:
   Explain how the Bakery Algorithm guarantees **mutual exclusion**, **fairness**, and **bounded waiting** (briefly recap key ideas).
2. **Comparison**:
   Compare **Bakery Algorithm** and **Peterson’s Algorithm** with respect to:
    * Number of threads supported
    * Fairness
    * Assumptions and hardware requirements
    * Implementation complexity
3. **Scalability**: What performance issues might arise as the number of threads increases? How does Bakery’s use of arrays affect overhead?
4. **Modern Systems**: Why is the Bakery Algorithm rarely used in modern multi-core systems? What kinds of failures could occur without memory barriers?
5. **Practical Design**: Would a simple spinlock using `Test-And-Set` (assuming hardware allows) be more practical in this case? Justify based on system characteristics.


{:.highlight}
> **Hints:**
> * Think about Bakery’s use of shared arrays and lexicographic ordering.
> * Peterson’s is for 2 threads only but uses simpler logic.
> * Memory visibility is a major challenge on multi-core systems.
> * A Test-And-Set lock is fast but can waste CPU with busy waiting.
> * Consider energy, determinism, and fairness in embedded contexts.


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
The Bakery Algorithm ensures <strong>mutual exclusion</strong> by having each process wait until no others have a smaller ticket (or equal ticket and smaller ID). It provides <strong>fairness</strong> by granting access in strict lexicographic order, and guarantees <strong>bounded waiting</strong> since each process eventually sees all smaller tickets cleared and can proceed.
</p>

<p>
<strong>Bakery vs. Peterson:</strong>
<ul>
  <li><strong>Thread support:</strong> Bakery supports <em>n</em> threads, Peterson only 2.</li>
  <li><strong>Fairness:</strong> Bakery is strictly fair due to ticketing; Peterson may allow overtaking.</li>
  <li><strong>Assumptions:</strong> Both require atomic reads/writes and sequential memory consistency. Bakery is more sensitive to memory visibility issues as <code>n</code> grows.</li>
  <li><strong>Complexity:</strong> Bakery is more complex and harder to reason about and implement correctly for large <code>n</code>.</li>
</ul>
</p>

<p>
<strong>Scalability:</strong> Bakery requires two shared arrays of size <code>n</code>. As <code>n</code> increases, each thread must loop through <code>n</code> entries to decide if it can enter, making it <code>O(n)</code> time per entry and exit. This becomes costly in systems with many threads.
</p>

<p>
<strong>Modern Systems:</strong> Bakery assumes sequential memory. On multi-core systems, CPUs may reorder instructions and cache values locally. Without memory barriers, a process might read stale or partially updated values, violating mutual exclusion. Thus, Bakery is unreliable without explicit fences or memory ordering constraints.
</p>

<p>
<strong>Spinlock with Test-And-Set:</strong> If atomic instructions are available, a spinlock is often more practical. It uses constant space and provides fast entry under low contention. However, it can waste CPU cycles when waiting. For small embedded systems with known thread count and fairness requirements, Bakery remains a viable and starvation-free alternative if hardware limits exclude atomic ops.
</p>


</p></div><br>



## Broken Counter

### Background

In multithreaded programs, race conditions occur when multiple threads read-modify-write shared variables without <span class="orange-bold">proper</span> synchronization. Even a simple increment operation like `counter++` is not atomic. It involves reading the current value, incrementing it, and writing it back, potentially allowing interleaved access that corrupts the result.

Languages like C and C++ allow you to create threads using `pthread_create`, but unless you use locks (e.g., `pthread_mutex_t`), **race conditions** may occur.



### Scenario

You and a teammate are writing a benchmarking tool that spawns 10 threads to increment a shared counter 100,000 times each. You expect the result to be exactly 1,000,000. However, the output is often much lower and non-deterministic.

Here’s the simplified C code you're using:

```c
#include <stdio.h>
#include <pthread.h>

#define NUM_THREADS 10
#define NUM_INCREMENTS 100000

int counter = 0;

void* increment(void* arg) {
    for (int i = 0; i < NUM_INCREMENTS; i++) {
        counter++;  // not atomic!
    }
    return NULL;
}

int main() {
    pthread_t threads[NUM_THREADS];

    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_create(&threads[i], NULL, increment, NULL);
    }

    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
    }

    printf("Final counter: %d\n", counter);
    return 0;
}
```


**Answer the following questions:**
1. Explain why `counter++` causes a race condition. What are the read-modify-write steps behind it?
2. Why does this lead to a final value that is less than expected?
3. Suggest two ways to fix this race condition in C using the pthreads library.
4. If the increments were performed in a signal handler, would a mutex still be safe? Why or why not?
5. Rewrite the `increment` function using a mutex to make it correct.


{:.highlight}
> **Hints:**
> * Think in terms of instruction interleaving between threads.
> * `counter++` expands to a load, increment, and store.
> * Consider using `pthread_mutex_t` or `__sync_fetch_and_add` (if allowed).
> * Signal handlers have limitations on which functions are safe to use.
> * Mutexes must be initialized before use and destroyed afterward.


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
The expression <code>counter++</code> is not atomic. It involves three steps: reading the current value of <code>counter</code>, incrementing it, and writing the new value back. When multiple threads execute this concurrently without coordination, two or more threads might read the same value, leading to lost updates.
</p>

<p>
This race condition means that some increments overwrite others. For example, two threads may both read <code>counter = 5</code>, increment it to 6, and write back 6. Only one increment takes effect even though both threads ran their code.
</p>

<p>
Two solutions using pthreads:
<ul>
  <li>Wrap the increment with a mutex lock:</li>
  <pre>
pthread_mutex_lock(&lock);
counter++;
pthread_mutex_unlock(&lock);
  </pre>

  <li>Use atomic built-ins like <code>__sync_fetch_and_add</code> (GCC):</li>
  <pre>
__sync_fetch_and_add(&counter, 1);
  </pre>
</ul>
</p>

<p>
Mutexes are not safe inside signal handlers because they are not async-signal-safe; using them may lead to deadlock if a signal interrupts a locked section. For signal handlers, use atomic types or avoid shared state entirely.
</p>

<p>
Here is the corrected <code>increment</code> function with a mutex:
</p>

<pre>
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

void* increment(void* arg) {
    for (int i = 0; i < NUM_INCREMENTS; i++) {
        pthread_mutex_lock(&lock);
        counter++;
        pthread_mutex_unlock(&lock);
    }
    return NULL;
}
</pre>

</p></div><br>



## Fast but Wrong, Slow but Safe

### Background

Fixing a race condition with locks ensures correctness, but may introduce performance bottlenecks, especially when many threads contend for the same lock. Low-level operations like `__sync_fetch_and_add()` or C11 atomic types may avoid locks entirely, but they are still not "free."

The `__sync_fetch_and_add()` function is a GCC-provided built-in that performs an **atomic fetch-and-add operation**. It is often used in multithreaded programs to safely increment shared counters without requiring explicit locks. This operation reads the current value of a variable, adds a specified amount to it, and writes the result back, **all as a single atomic step**. It ensures that no other thread can interleave its access between the read and write, thus preventing race conditions. 

{:.note}
Unlike mutexes, `__sync_fetch_and_add()` avoids the overhead of blocking and kernel-level context switching, making it much faster for simple counter updates under light to moderate contention. However, under heavy contention, it still suffers from memory bus contention and potential cache line bouncing, since all threads are updating the same memory location.


### Scenario

You fixed the counter race condition by protecting `counter++` with a mutex. But you notice that your benchmark runs dramatically slower, especially with more threads. Your teammate switches to `__sync_fetch_and_add`, which improves performance. However, you later redesign the benchmark to use per-thread counters, one for each thread, and aggregate them only at the end.

{:.highlight}
To your surprise, this last version is not only correct, it’s much faster!


**Answer the following questions:**
1. Why does adding a mutex to `counter++` slow the program down as threads increase?
2. How does `__sync_fetch_and_add()` improve performance compared to mutexes?
3. What is false sharing, and how can it slow down multithreaded code?
4. How does using per-thread counters avoid false sharing and improve performance?
5. In the code below, why might this still suffer from false sharing, and how can you fix it?

```c
int counters[NUM_THREADS];

void* increment(void* arg) {
    int tid = *(int*)arg;
    for (int i = 0; i < NUM_INCREMENTS; i++) {
        counters[tid]++;
    }
    return NULL;
}
```


{:.highlight}
> **Hints:**
> * Mutex contention causes cache line bouncing and kernel transitions.
> * Atomic ops avoid the kernel but still serialize at the hardware level.
> * On most CPUs, adjacent integers are packed into a single 64-byte cache line.
> * Use padding or cache alignment to separate frequently written variables.
> * Think of data locality and thread-private storage.



<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
Using a mutex around <code>counter++</code> slows the program because threads constantly compete for the same lock. Each <code>pthread_mutex_lock()</code> and <code>unlock()</code> involves kernel-mode coordination and cache invalidation, especially when many threads contend for the same memory location.
</p>

<p>
<code>__sync_fetch_and_add()</code> is faster because it avoids locking. It is a hardware-level atomic operation. It still enforces serialization on the memory bus but has much lower overhead than mutexes, especially in user space.
</p>

<p>
False sharing occurs when multiple threads modify different variables that happen to lie on the same cache line. Even if the variables are unrelated, the CPU cache treats them as a unit, causing performance loss due to frequent cache invalidation and memory traffic.
</p>

<p>
Per-thread counters avoid contention and false sharing by allowing each thread to increment its own private variable. There is no coordination during updates, so threads can operate independently and cache their values locally. Aggregation is done once at the end, in a serial step.
</p>

<p>
In the code shown, <code>counters[tid]</code> for different threads may still lie on the same cache line. To fix this, add padding between entries to separate them by at least 64 bytes (common cache line size). For example:
</p>

<pre>
typedef struct {
    int value;
    char pad[64 - sizeof(int)];
} PaddedCounter;

PaddedCounter counters[NUM_THREADS];
</pre>

<p>
Now each <code>counters[i].value</code> is isolated on its own cache line, avoiding false sharing and preserving performance.
</p>

</p></div><br>


## Sleepy Barber And Spinning Chef

### Background

There are many synchronization primitives available for managing concurrent access in operating systems: **mutexes**, **condition variables**, and **spinlocks**, to name a few. Each has trade-offs in responsiveness, CPU usage, and complexity.

* **Spinlocks** are useful for short, fast critical sections. They keep the thread active in a loop while waiting.
* **Condition variables** put a thread to sleep and allow it to wait for a specific condition to become true, often used when coordinating work between producer and consumer threads.



### Scenario

You are building a multithreaded kitchen simulation. In one task, a **chef** repeatedly checks for new orders and prepares them. In another, a **barber** sleeps until a customer arrives, then services them.

The chef currently uses a spinlock to protect the queue of orders and continuously polls it. The barber uses a mutex and condition variable to sleep until a customer wakes him.

You are asked to review these two designs and evaluate which synchronization primitive is more appropriate for each case.


**Answer the following questions:**
1. Why is using a spinlock appropriate or inappropriate for the chef scenario? When would it be acceptable?
2. Why does the barber benefit from using a condition variable instead of a spinlock?
3. What happens if the chef’s critical section is long or gets delayed by other threads? How does this affect spinlock behavior?
4. Describe the difference in CPU usage between a thread using a spinlock and one waiting on a condition variable.
5. Could the chef’s polling be replaced with a condition variable? If so, what trade-off would that involve?


{:.highlight}
> **Hints**:
> * A spinlock avoids sleeping but wastes CPU while waiting.
> * Condition variables block the thread, allowing CPU reuse by others.
> * Think about how "active waiting" and "passive waiting" behave under load.
> * Consider real-time responsiveness vs. system efficiency.
> * Long critical sections make spinlocks problematic.



<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
Using a spinlock in the chef scenario may be appropriate only if the wait time is extremely short and predictable. For example, if new orders arrive very frequently and processing them is fast, a spinlock avoids the overhead of sleeping and waking. However, if there is any delay or unpredictability in order arrival, the chef ends up wasting CPU cycles while spinning, reducing overall system efficiency.
</p>

<p>
The barber benefits from using a condition variable because customer arrivals are sporadic. Instead of constantly checking for customers (which wastes CPU), the barber thread can sleep and be notified only when a customer arrives. This is an example of passive waiting, which is more energy-efficient and avoids unnecessary CPU usage.
</p>

<p>
If the chef's critical section becomes long due to lock contention or delays in order processing, then the spinlock creates significant performance problems. Other threads trying to acquire the lock will spin uselessly, potentially starving the system of CPU resources for useful work. Spinlocks assume that critical sections are short and infrequent.
</p>

<p>
A thread waiting on a spinlock continuously consumes CPU cycles by actively polling the lock variable. In contrast, a thread waiting on a condition variable yields control to the OS and does not consume CPU until it is woken up. Therefore, condition variables are far more CPU-efficient for longer waits.
</p>

<p>
Yes, the chef’s polling loop could be replaced with a condition variable. This would eliminate busy-waiting and conserve CPU, but at the cost of some latency and increased complexity. The trade-off is between responsiveness (spinlocks give faster reactions) and system load (condition variables reduce CPU usage). If near-instant reaction is not critical, the condition variable is usually the better choice.
</p>
</p></div><br>

### Further Notes

The diagram below compares the behavior of spinlocks (active waiting) and condition variables (passive waiting), showing what a thread do during each case. 

```
Thread State Timeline

| Time → → →

Case A: Spinlock (Chef)
────────────────────────────────────────────────────────────
[ Check Lock ]→[ Check Lock ]→[ Check Lock ]→[ Acquire Lock ]
(Consumes CPU) (Consumes CPU) (Consumes CPU) (Finally enters)

Notes:
- Thread stays active, continuously polls
- CPU is busy even if lock is not available
- Wasteful under contention

Case B: Condition Variable (Barber)
────────────────────────────────────────────────────────────
[   Wait (Sleep)   ]────────────→[  Wake & Acquire Lock  ]
(No CPU used while sleeping)     (CPU used only on wake)

Notes:
- Thread sleeps until notified
- CPU can be used by other threads
- More scalable and efficient under low activity
```

Condition Variable puts the waiting thread to sleep until lock is available (resource is available), hence preventing redudant usage of the CPU like in the Spinlock case. 



## Signal, Wait, Or Die Trying

### Background

**Semaphores** are a fundamental synchronization primitive used to control access to shared resources. A counting semaphore uses two atomic operations:

* `wait()` (or `P()`): decrements the counter and may block the thread if the value is below zero.
* `signal()` (or `V()`): increments the counter and may unblock a waiting thread.

Semaphores are more flexible than mutexes but trickier to use correctly. Misordered or missing operations can lead to deadlock, starvation, or undefined behavior. Interleavings of `wait()` and `signal()` across multiple threads can lead to surprising outcomes.



### Scenario

You have a counting semaphore `sem` initialized to 1, and three threads A, B, and C trying to use it. They each execute code similar to this:

```c
wait(sem);
// critical section
signal(sem);
```

However, during debugging, you observe the following behavior:

* Sometimes all three threads make progress eventually.
* Other times, one thread seems to never proceed.
* Occasionally, two threads enter the critical section at once.

You suspect there is a <span class="orange-bold">bug</span> either in your semaphore implementation or how it’s being used.


**Answer the following questions:**
1. Describe what should happen if all three threads execute the `wait()` and `signal()` pairs properly, assuming a correct semaphore.
2. What could cause more than one thread to enter the critical section at once?
3. Could starvation happen even if the semaphore is working correctly? Why or why not?
4. How would initializing `sem` to 0 affect behavior? Give an example use case for this.
5. How can priority inversion affect semaphore-based scheduling?


{:.highlight}
> **Hints:**
> * Check whether `wait()` and `signal()` are atomic.
> * Semaphores can have fairness issues if wakeups are not queued.
> * A semaphore initialized to 0 means the first `wait()` always blocks.
> * Consider real-time systems where lower-priority threads hold semaphores.



<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
If all three threads call <code>wait(sem)</code> and <code>signal(sem)</code> correctly, and the semaphore is initialized to 1, then at most one thread should be in the critical section at any time. The semaphore acts like a gatekeeper: the first thread enters, and the others block until the semaphore is signaled.
</p>

<p>
If more than one thread is in the critical section simultaneously, it likely indicates a bug in the semaphore implementation. This can happen if <code>wait()</code> and <code>signal()</code> are not atomic or if there's a race condition between the check and update of the counter. It may also happen if one thread skips the <code>wait()</code> entirely.
</p>

<p>
Starvation is possible even with a correct semaphore if it does not enforce FIFO ordering or fairness. For example, a thread could be perpetually delayed if others repeatedly acquire and release the semaphore before it is scheduled. This depends on the OS scheduler and the semaphore's internal policy.
</p>

<p>
Initializing <code>sem</code> to 0 means that all threads calling <code>wait(sem)</code> will block until another thread first signals. This is useful for cases like thread synchronization barriers or one-time event coordination, where threads must wait for a trigger to proceed. For example, a main thread might initialize <code>sem = 0</code>, and signal it only after setup is complete.
</p>

<p>
Priority inversion can occur when a low-priority thread holds a semaphore that a high-priority thread is waiting on. If the scheduler doesn't boost the priority of the holder, the high-priority thread may be delayed indefinitely. Some systems use priority inheritance to avoid this issue.
</p>



</p></div><br>




## Kernel Or User: Who Should Control?

### Background

Synchronization can be implemented in **user space** (e.g. spinlocks) or **kernel space** (e.g. mutexes that cause a thread to sleep). Both approaches provide mutual exclusion but have different performance characteristics depending on the context and workload.

* **Spinlocks** perform well when the wait time is short because they avoid context switches, but they consume CPU cycles while spinning.
* **Blocking locks** (like `pthread_mutex_lock`) may incur the overhead of system calls and context switches, but they free up the CPU for other work while waiting.

Operating systems often use **spinlocks** for short kernel critical sections and **blocking locks** for user programs. 


### Scenario

You are designing a shared-memory message-passing system for inter-thread communication in a real-time control application. Message queues must be accessed with mutual exclusion, but responsiveness is critical. You initially use a `pthread_mutex_t`, but performance profiling reveals delays in message delivery under high contention.

Your teammate suggests <span class="orange-bold">replacing</span> the blocking mutex with a spinlock implemented using atomic instructions. You must evaluate the correctness and efficiency of this change.


**Answer the following questions:**
1. Why might a user-space spinlock reduce latency compared to a blocking mutex?
2. What are the downsides of using spinlocks in user applications, especially under contention?
3. Under what conditions would a blocking mutex be more appropriate than a spinlock?
4. Why do operating systems often implement hybrid locks (e.g. spin-then-sleep)?
5. If the spinlock is implemented with an atomic compare-and-swap loop, what issues should you watch for?

{:.highlight}
> **Hints**:
> * Context switches cost time and involve kernel interaction.
> * Spinlocks make sense only if locks are held briefly.
> * Think about CPU starvation and scheduler fairness.
> * Consider responsiveness vs. CPU efficiency.



<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
A user-space spinlock avoids the overhead of a system call and context switch that occurs with a blocking mutex. If the lock is only held briefly, spinning may allow the thread to acquire the lock quickly without giving up the CPU, improving latency in high-frequency, low-contention scenarios.
</p>

<p>
The downside of spinlocks is that they waste CPU while waiting. If the thread holding the lock is delayed or preempted, spinning threads may consume an entire core doing no useful work. Under heavy contention or long critical sections, this leads to poor performance and potential starvation of other threads.
</p>

<p>
A blocking mutex is more appropriate when the lock is expected to be held for a non-trivial duration or when contention is high. Sleeping allows the OS to schedule other threads and ensures better fairness and resource sharing across the system.
</p>

<p>
Operating systems often use hybrid locks that spin briefly and then block if the lock is not available. This balances responsiveness (quick reaction if the lock is released soon) with efficiency (avoid long spinning if not). Examples include futex-based locks in Linux.
</p>

<p>
When implementing a spinlock with an atomic compare-and-swap loop, watch for:
<ul>
  <li>Livelock: threads repeatedly try and fail to acquire the lock</li>
  <li>Memory bus contention from cache line bouncing</li>
  <li>Lack of backoff: adding a delay between retries helps reduce pressure</li>
  <li>Fairness issues: no ordering guarantees among competing threads</li>
</ul>
Use memory barriers to prevent compiler or CPU reordering of lock-related instructions.
</p>



</p></div><br>




## Visual Trace Exercise: Interleaving Madness

### Background

When multiple threads execute code that manipulates shared state, the exact order in which operations occur determines the program’s behavior. Without proper synchronization, these operations can **interleave** in many ways, leading to data races and inconsistent results.

Understanding this interleaving is key to **debugging** and **reasoning** about concurrent programs.


### Scenario

You are given the following C program with two threads accessing a shared variable:

```c
int x = 0;

void* thread1(void* arg) {
    x = x + 1;
    return NULL;
}

void* thread2(void* arg) {
    x = x + 1;
    return NULL;
}
```

You expect `x` to be 2 at the end, but sometimes it’s 1.

You are asked to list possible interleavings of the two threads' operations and explain how they lead to different outcomes.


**Answer the following questions:**
1. What low-level operations make up `x = x + 1`? (Assume it expands into three steps.)
2. Construct a timeline showing an interleaving where the final value of `x` is 1.
3. Show a timeline where the final value is 2.
4. How many possible interleavings are there if both threads' operations are broken into three steps?
5. What is the only way to make the result consistently 2?


{:.highlight}
> **Hints:**
> * Think in terms of `LOAD x`, `ADD 1`, `STORE x`.
> * Interleavings can overlap these steps.
> * No synchronization = no guarantees.
> * You can use mutexes or atomic operations to protect critical sections.

### Visual Timeline Example

```text
Step-by-step Breakdown of: x = x + 1

Thread 1:        LOAD x → ADD 1 → STORE x
Thread 2:        LOAD x → ADD 1 → STORE x

Interleaving that leads to x == 1:

T1: LOAD x  (x = 0)
T2: LOAD x  (x = 0)
T1: ADD 1   (temp1 = 1)
T2: ADD 1   (temp2 = 1)
T1: STORE x (x = 1)
T2: STORE x (x = 1) ← overwrites T1’s result

Final value: x == 1 (lost update)

Interleaving that leads to x == 2:

T1: LOAD x  (x = 0)
T1: ADD 1   (temp1 = 1)
T1: STORE x (x = 1)
T2: LOAD x  (x = 1)
T2: ADD 1   (temp2 = 2)
T2: STORE x (x = 2)

Final value: x == 2
```

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
Each <code>x = x + 1</code> expands to three steps: <code>LOAD x</code>, <code>ADD 1</code>, and <code>STORE x</code>. These are not atomic.
</p>

<p>
When threads interleave without synchronization, a race condition occurs. In the example where both threads load <code>x</code> as 0 and store 1, one increment is lost: the result is 1 instead of 2.
</p>

<p>
The number of possible interleavings grows quickly. If each thread has 3 steps, there are <code>6 choose 3 = 20</code> ways to interleave their operations.
</p>

<p>
The only way to ensure the final value is consistently 2 is to synchronize the critical section. This can be done by wrapping the increment in a <code>pthread_mutex_lock()</code> and <code>pthread_mutex_unlock()</code>, or by using an atomic operation like <code>__sync_fetch_and_add(&x, 1)</code>.
</p>
</p></div><br>



## Visual Trace Exercise: Missed Wakeups

### Background

**Condition variables** are used to block a thread until a certain condition is true. They are used in combination with a mutex:

* `pthread_cond_wait(&cond, &mutex)` releases the mutex and puts the thread to sleep.
* `pthread_cond_signal(&cond)` wakes one waiting thread.
* `pthread_cond_broadcast(&cond)` wakes all waiting threads.

{:.error}
However, condition variables are **not message queues**. If no thread is waiting when `signal()` is called, the signal is lost. This leads to the classic **missed wakeup problem**.


### Scenario

You implement a producer-consumer system with a shared buffer and one slot. You use a condition variable to notify the consumer when the producer inserts an item. But sometimes the consumer seems to block forever.

Here is the simplified code:

```c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

int ready = 0;

void* producer(void* arg) {
    pthread_mutex_lock(&mutex);
    ready = 1;
    pthread_cond_signal(&cond); // wake consumer
    pthread_mutex_unlock(&mutex);
    return NULL;
}

void* consumer(void* arg) {
    if (ready == 0) {
        pthread_cond_wait(&cond, &mutex); // wait for signal
    }
    pthread_mutex_lock(&mutex);
    // consume
    pthread_mutex_unlock(&mutex);
    return NULL;
}
```


**Answer the following questions:**
1. What happens if the producer signals before the consumer starts waiting?
2. Why must `pthread_cond_wait()` always be used inside a `while` loop, not an `if`?
3. What is a **spurious wakeup** and how does the loop guard handle it?
4. Construct a visual timeline where the consumer blocks forever due to a missed signal.
5. **Rewrite** the consumer code to be safe against missed signals and spurious wakeups.


{:.highlight}
> **Hints:**
> * A condition variable does not remember past signals.
> * Signals are not queued, they are **lost** if no thread is waiting.
> * Always recheck the condition after waking up.




<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
If the producer signals the condition variable before the consumer begins waiting, and <code>pthread_cond_wait()</code> is not yet called, the signal is lost. The consumer then calls wait and blocks forever because the condition is already true, but no thread will signal again. The visual trace is as follows: 
</p>

<pre><code>T1 (Producer): LOCK → (CONTEXT SWITCHED)
T2 (Consumer): if (ready == 0) → (CONTEXT SWITCHED)
T1 (Producer): ready = 1 → SIGNAL cond → UNLOCK 
T2 (Consumer): WAIT cond → blocks forever</code></pre>

<p><strong>Explanation:</strong>
<ul>
    <li>The variable "ready" changes after consumer checks it, just before it calls Wait </li>
    <li>There's a gap between time-of-check and time-of-use </li>
    <li>When consumer calls wait, the condition is already true and SIGNAL is already sent</li>
    <li>Consumer sleeps forever</li>
</ul>
</p>

<p>
<code>pthread_cond_wait()</code> must always be used inside a <code>while</code> loop that checks the predicate (e.g., <code>while (ready == 0)</code>). This ensures the thread waits only if the condition is false and rechecks after waking up. It also must be called inside the critical section (protected by mutex).
</p>

<p>
A spurious wakeup is when <code>pthread_cond_wait()</code> returns even though no signal or broadcast occurred. This can happen due to scheduler decisions or race conditions in kernel internals. Using a <code>while</code> guard ensures correctness even in such cases.
</p>

<p>
In the broken interleaving, the producer sets <code>ready = 1</code> and signals <code>cond</code> before the consumer starts waiting. Since the consumer checks for the condition outside of a mutex, it checked a stale value and misses the signal so it blocks forever. The <code>if</code> check will also cause incorrect behavior due to spurious wakeup.
</p>

<p>
The corrected consumer logic is:
</p>

<pre>
pthread_mutex_lock(&mutex);
while (ready == 0) {
    pthread_cond_wait(&cond, &mutex);
}
// consume
pthread_mutex_unlock(&mutex);
</pre>

<p>
This ensures that `ready` is not stale and consumer only waits when needed. It also always rechecks the shared condition after being woken up, handling both missed signals and spurious wakeups correctly.
</p>



</p></div><br>




## Visual Trace Exercise: The Silent Corruption

### Background

The **bounded buffer problem** is a classic producer-consumer scenario where multiple threads insert and remove items from a shared buffer of fixed size. To coordinate access safely and efficiently, the solution typically uses:

* A **counting semaphore `empty`**, initialized to the buffer size, which tracks how many empty slots are available.
* A **counting semaphore `full`**, initialized to 0, which tracks how many items are available to consume.
* A **mutex**, which ensures mutual exclusion during access to the buffer itself.

While the semaphores regulate *when* threads can proceed, the mutex protects *how* they access shared memory.


### Correct Implementation (Reference)

```c
#define BUFFER_SIZE 5
int buffer[BUFFER_SIZE];
int in = 0, out = 0;

sem_t empty, full;
pthread_mutex_t mutex;

void* producer(void* arg) {
    while (1) {
        int item = produce_item();

        sem_wait(&empty);
        pthread_mutex_lock(&mutex);

        buffer[in] = item;
        in = (in + 1) % BUFFER_SIZE;

        pthread_mutex_unlock(&mutex);
        sem_post(&full);
    }
}

void* consumer(void* arg) {
    while (1) {
        sem_wait(&full);
        pthread_mutex_lock(&mutex);

        int item = buffer[out];
        out = (out + 1) % BUFFER_SIZE;

        pthread_mutex_unlock(&mutex);
        sem_post(&empty);

        consume_item(item);
    }
}
```

This implementation is safe because:

* **Only one thread at a time accesses the buffer.**
* **Producers block when the buffer is full.**
* **Consumers block when the buffer is empty.**


### The Silent Corruption

Your teammate attempts to simplify the code by removing the mutex, arguing that the semaphores already control access:

```c
sem_t empty, full; // mutex removed

void* producer(void* arg) {
    while (1) {
        int item = produce_item();

        sem_wait(&empty);
        buffer[in] = item;
        in = (in + 1) % BUFFER_SIZE;
        sem_post(&full);
    }
}

void* consumer(void* arg) {
    while (1) {
        sem_wait(&full);
        int item = buffer[out];
        out = (out + 1) % BUFFER_SIZE;
        sem_post(&empty);

        consume_item(item);
    }
}
```

{:.warning}
The program runs and does not crash, but consumers occasionally receive corrupted or duplicated data. The bug is subtle, intermittent, and hard to trace.


**Answer the following questions:**
1. Why does the code *seem* to work most of the time, even without a mutex?
2. Explain why `empty` and `full` alone are not enough to ensure safe access to the buffer.
3. Show a timeline where two producers write to the buffer concurrently and cause data corruption.
4. What are some ways this bug could manifest in a real system?
5. What is the minimal fix required to make this implementation correct?


{:.highlight}
> **Hints:**
> * Semaphores only count availability, not protect consistency.
> * Shared variables like `buffer`, `in`, and `out` need to be accessed atomically.
> * The bug may not appear on single-core machines or under light load.


### Visual Timeline Example
```text
Initial: empty = 5, full = 0, in = 0

T1 (Producer):
    sem_wait(empty) → proceeds
    writes buffer[0] = A

T2 (Producer):
    sem_wait(empty) → proceeds
    writes buffer[0] = B  ← overlaps with T1

→ Consumer:
    sem_wait(full)
    reads buffer[0] = B or corrupted value
    consumes wrong or partial data
```


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
The code appears to work because semaphores successfully prevent buffer overflow and underflow. That is, producers and consumers wait for slots to be available before proceeding. However, this does not protect against concurrent access to shared memory.
</p>

<p>
The semaphores <code>empty</code> and <code>full</code> control <em>when</em> threads can proceed, but not <em>how</em> they access the buffer. If two producers enter the critical section together, they may both modify the <code>in</code> index or write to the same slot, leading to lost writes or corrupted data.
</p>

<p>
A possible interleaving is:
</p>

<pre>
T1: sem_wait(empty)
T1: buffer[in] = A        // in == 0
T2: sem_wait(empty)
T2: buffer[in] = B        // in == 0 (same)
T1: in = 1
T2: in = 1
</pre>

<p>
Both producers write to <code>buffer[0]</code>. Only one of the writes is preserved, and the item from the other is lost. Also, <code>in</code> is incorrectly incremented twice.
</p>

<p>
In a real program, this bug may result in consumers receiving duplicated data, skipped items, or uninitialized values. These errors may be rare and timing-dependent, making them difficult to debug.
</p>

<p>
The minimal fix is to restore a <code>pthread_mutex_t</code> to protect the buffer and index variables. The <code>insert</code> and <code>remove</code> sections must be guarded with <code>pthread_mutex_lock</code> and <code>pthread_mutex_unlock</code>, ensuring mutual exclusion during modification.
</p>
```



</p></div><br>



## Visual Trace Exercise: Find The Missing Semaphore

### Background

In a bounded buffer with multiple producers and consumers, correct synchronization requires managing both **access safety** (via a mutex) and **resource availability** (via semaphores). Omitting either can cause subtle bugs.

This exercise presents code with **a mutex but no semaphores**. You must identify what goes wrong and fix it.

### Scenario

The following code is used to implement a bounded buffer between a producer and consumer. It uses a shared circular buffer of size 10. A mutex ensures only one thread accesses the buffer at a time.

However, <span class="orange-bold">no semaphores</span> are used to track available slots or items. Occasionally, the consumer reads invalid data, or the producer writes past the buffer limit.

```c
#define BUFFER_SIZE 10
int buffer[BUFFER_SIZE];
int in = 0, out = 0;

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

void* producer(void* arg) {
    while (1) {
        int item = produce_item();

        pthread_mutex_lock(&mutex);
        buffer[in] = item;
        in = (in + 1) % BUFFER_SIZE;
        pthread_mutex_unlock(&mutex);
    }
}

void* consumer(void* arg) {
    while (1) {
        pthread_mutex_lock(&mutex);
        int item = buffer[out];
        out = (out + 1) % BUFFER_SIZE;
        pthread_mutex_unlock(&mutex);

        consume_item(item);
    }
}
```


**Answer the following questions:**

1. What is the role of the mutex in this code, and what problems does it prevent?
2. Why does this code fail even though mutual exclusion is enforced?
3. Describe two different failure scenarios: one from buffer overrun, one from under-read.
4. Which semaphores are missing? What should their initial values be?
5. Rewrite the core logic of both `producer` and `consumer` using semaphores correctly.


{:.highlight}
> **Hints**:
> * A mutex only protects access. <span class="orange-bold">It doesn’t count how much data is in the buffer</span>.
> * Use `empty` to block producers when the buffer is full.
> * Use `full` to block consumers when the buffer is empty.
> * Always acquire semaphores in the correct order.


### Visual Timeline Example

```text
Initial: in = 9, out = 9

T1 (Producer):
    LOCK
    buffer[9] = A
    in = 0
    UNLOCK

T2 (Producer):
    LOCK
    buffer[0] = B
    in = 1
    UNLOCK

→ buffer now contains 11 items (overflowed)

T3 (Consumer):
    LOCK
    item = buffer[0] // maybe stale or overwritten
    out = 1
    UNLOCK
```


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>

<p>
The mutex ensures that only one thread accesses the buffer at a time, preventing data races and inconsistent updates to <code>in</code> and <code>out</code>. It protects correctness of structure but not logic of usage limits.
</p>

<p>
The code fails because there is no mechanism to prevent the producer from writing when the buffer is full, or the consumer from reading when it is empty. The mutex only guards the buffer from concurrent modification, but it does not manage availability.
</p>

<p>
Two failure scenarios:
<ul>
  <li><strong>Overrun:</strong> If producers insert more than 10 items without any being consumed, <code>in</code> will loop and overwrite older values, causing data loss.</li>
  <li><strong>Underrun:</strong> If a consumer runs before any items are produced, it will read uninitialized or stale data, leading to garbage being consumed.</li>
</ul>
</p>

<p>
To fix this, two semaphores are needed:
<ul>
  <li><code>sem_t empty</code>, initialized to 10, representing available slots.</li>
  <li><code>sem_t full</code>, initialized to 0, representing available items.</li>
</ul>
</p>

<p>
Corrected logic:

<pre>
void* producer(void* arg) {
    while (1) {
        int item = produce_item();

        sem_wait(&empty);
        pthread_mutex_lock(&mutex);
        buffer[in] = item;
        in = (in + 1) % BUFFER_SIZE;
        pthread_mutex_unlock(&mutex);
        sem_post(&full);
    }
}

void* consumer(void* arg) {
    while (1) {
        sem_wait(&full);
        pthread_mutex_lock(&mutex);
        int item = buffer[out];
        out = (out + 1) % BUFFER_SIZE;
        pthread_mutex_unlock(&mutex);
        sem_post(&empty);

        consume_item(item);
    }
}
</pre>
</p>
</p></div><br>





## Visual Trace Exercise: Unintended Garbage Collection

### Background

In producer-consumer synchronization using semaphores, the order of operations matters. If a **producer signals the consumer too early**, it may allow a consumer to proceed before the shared data is actually written. This causes a classic race condition: the consumer reads a **garbage value**, a default-initialized variable, or stale memory.

Condition variables are often paired with a loop to guard against this. But with semaphores, the **ordering alone must guarantee correctness**.


### Scenario

Your teammate makes the following optimization: they move `sem_post(&full)` to just before inserting the item. They believe this improves responsiveness by unblocking consumers early.

```c
sem_wait(&empty);
sem_post(&full); // moved up
buffer[in] = item;
in = (in + 1) % BUFFER_SIZE;
```

{:.note}
It “seems to work”, but in some runs, the consumer logs an invalid item or reads garbage values.


**Answer the following questions:**
1. Why is it incorrect to post `full` before inserting the item?
2. Show a timeline where the consumer reads from the buffer before the producer writes to it.
3. Why might this bug be rare in tests but critical in production?
4. How would you fix this bug?
5. In general, when is it safe to signal a semaphore that releases another thread?


{:.highlight}
> **Hints:**
> * Semaphores should reflect the *logical state* of the resource.
> * The signal must mean “the item is fully ready.”
> * Use timeline thinking: what happens between signal and actual write?


### Visual Timeline Example

```text
Initial state: empty = 3, full = 0

T1 (Producer):
    sem_wait(empty) → OK (empty = 2)
    sem_post(full)  → early signal (full = 1)

T2 (Consumer):
    sem_wait(full)  → OK (full = 0)
    reads buffer[in] before producer writes
    → reads uninitialized or stale value

T1:
    writes buffer[in] = 42
```



<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
Posting <code>sem_post(&full)</code> before writing the item allows a consumer to wake up and read the buffer slot before the producer actually inserts the new value. This breaks the synchronization guarantee that the consumer should only proceed once an item is ready.
</p>

<p>
A race occurs if the consumer thread is scheduled immediately after <code>sem_post(&full)</code> and before the producer writes to the buffer. The consumer reads from <code>buffer[in]</code> which still holds uninitialized or stale data.
</p>

<p>
This bug may not appear in light-load testing where threads run sequentially. But under real-world conditions with context switches, multicore CPUs, and unpredictable timing, the issue can cause data corruption or logic errors.
</p>

<p>
To fix the bug, move <code>sem_post(&full)</code> after the item is fully inserted. Only signal when the resource (in this case, the item) is truly ready:
</p>

<pre>
sem_wait(&empty);
buffer[in] = item;
in = (in + 1) % BUFFER_SIZE;
sem_post(&full);
</pre>

<p>
In general, you should signal a semaphore only when the condition associated with it has become true. In producer-consumer systems, that means signaling <code>full</code> after an item is inserted, and <code>empty</code> after one is removed.
</p>

</p></div><br>