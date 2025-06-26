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

### Background: Circular Wait with Semaphores

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

{:.warning-title}
> Problem
> 
> At runtime, the system works for a while. But occasionally, when the buffer becomes full, all producers block indefinitely, even though consumers are still running.


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



## Dining Philosphers: Not So Hungery After All 
### Background

The **Dining Philosophers Problem** is a classic illustration of deadlock and resource contention. Five philosophers sit around a table. Between each pair of philosophers is a single fork. To eat, a philosopher needs to pick up both the left and right fork. Philosophers alternate between thinking and eating.

In the naïve solution, each philosopher:

1. Picks up the left fork.
2. Picks up the right fork.
3. Eats.
4. Puts down both forks.

<img src="{{ site.baseurl }}/docs/Problem Set/images/an_illustration_of_the_dining_philosophers_problem.png"  class="center_seventy no-invert"/>

If all philosophers pick up their left fork at the same time, <span class="orange-bold">no one can pick up their right fork</span>, and **deadlock** occurs.

Deadlock arises when four conditions are met: 

* **Mutual exclusion** (forks cannot be shared)
* **Hold and wait** (each philosopher holds one fork while waiting for the other)
* **No preemption** (forks are only released voluntarily)
* **Circular wait** (each philosopher waits on the next in a cycle)


### Scenario

You are given a C-like pseudocode implementation of the philosophers’ behavior. Each philosopher runs the following:

```c
while (true) {
    think();
    pick_up(left_fork);
    pick_up(right_fork);
    eat();
    put_down(left_fork);
    put_down(right_fork);
}
```

{:.note}
Forks are represented as mutexes. All philosophers start simultaneously. Deadlock sometimes occurs during execution.

Several alternate designs are proposed:
* **Design A:** Allow at most four philosophers to sit at the table at once.
* **Design B:** Impose a resource hierarchy by requiring philosophers to always pick up the lower-numbered fork first.
* **Design C:** Use a waiter (monitor) who grants permission to pick up forks only if both are available.
* **Design D:** Replace blocking `pick_up()` with try-lock. If the second fork is unavailable, release the first and retry after thinking.


**Answer the following questions:**
1. Identify which of the four Coffman deadlock conditions are satisfied in the original code.
2. For each of the proposed designs (A–D), explain how they address the deadlock problem.
3. Which designs also improve fairness or avoid starvation? Why?
4. What are the trade-offs between centralized (Design C) and decentralized (Design D) control?


{:.highlight}
> **Hints:**
> * Think about how each design breaks one or more deadlock conditions.
> * Consider whether philosophers may starve even if deadlock is avoided.
> * For resource hierarchy, assume fork numbers range from 0 to 4 clockwise.


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>The original implementation satisfies all four Coffman conditions:</p>
<ul>
  <li><strong>Mutual exclusion:</strong> Forks are mutexes, only one philosopher can hold each.</li>
  <li><strong>Hold and wait:</strong> Each philosopher holds one fork and waits for the other.</li>
  <li><strong>No preemption:</strong> Forks are only released voluntarily.</li>
  <li><strong>Circular wait:</strong> Each philosopher waits for the fork held by their neighbor.</li>
</ul>

<p>Here's how each design address the problems:</p>
<ul>
  <li><strong>Design A:</strong> By limiting access to four philosophers, at least one can eat, preventing a full circular wait. This breaks the circular wait condition.</li>
  <li><strong>Design B:</strong> Enforcing a global resource ordering (always pick lower-numbered fork first) ensures there can be no cycles in the wait-for graph. This breaks circular wait.</li>
  <li><strong>Design C:</strong> A waiter grants both forks or none, so no philosopher holds one fork while waiting for the other. This breaks hold and wait.</li>
  <li><strong>Design D:</strong> Philosophers do not hold one fork while waiting for the other. If they fail to acquire both, they release and retry later. This breaks hold and wait.</li>
</ul>

<p>About fairness and starvation:</p>
<ul>
  <li><strong>Design B:</strong> Can cause starvation if unlucky philosophers always have higher-numbered forks.</li>
  <li><strong>Design C:</strong> Can be made fair if the waiter uses a queue to grant forks.</li>
  <li><strong>Design D:</strong> May lead to starvation if some philosophers retry too often without success.</li>
</ul>

<p>Tradeoffs between Design C and D:</p>
<ul>
  <li><strong>Design C (centralized):</strong> Easier to enforce fairness, but adds a single point of failure and potential bottleneck.</li>
  <li><strong>Design D (decentralized):</strong> More scalable and avoids central coordination, but harder to analyze for starvation and fairness.</li>
</ul>

</p></div><br>





## Missed Notify with Separate Lock and Premature Signal

### Background

In Java, `wait()` and `notify()` are used within `synchronized(lock)` blocks to allow threads to coordinate using an object's monitor. However, unlike semaphores, `notify()` does **not persist**. If no thread is currently waiting on the monitor when `notify()` is called, the signal is **lost**.

{:.note}
This means if a thread calls `wait()` **after** a `notify()` has already occurred, it will block **forever**.

### Recap: Java Monitors and Intrinsic Synchronization

In Java, a **monitor** is a high-level concurrency construct that provides both **mutual exclusion** and **condition synchronization**. Java monitors are implemented as an intrinsic part of every object in the Java Virtual Machine (JVM), meaning that **every object implicitly supports monitor-based coordination**.


#### Mutex
From the lecture notes, recall that Java achieves mutual exclusion through the use of the `synchronized` keyword. When a thread enters a `synchronized` block or method, it must acquire the monitor associated with the target object or class.
* For instance methods, the monitor is associated with the instance (`this`).
* For static methods, the monitor is associated with the class object (`ClassName.class`).
* For `synchronized(obj)` blocks, the monitor is associated with the specific object `obj`.

{:.note}
Only one thread may hold a given monitor at a time. All other threads attempting to enter a synchronized region guarded by that monitor are **blocked** until the monitor is released.

#### Condition Synchronization

Java monitors also support condition synchronization through the methods `wait()`, `notify()`, and `notifyAll()`: all of which must be called while holding the monitor (i.e., within a synchronized block or method).

* `wait()` causes the current thread to **release the monitor** and suspend execution until another thread invokes `notify()` or `notifyAll()` on the same object.
* `notify()` wakes up **one** thread waiting on that monitor.
* `notifyAll()` wakes up **all** threads waiting on that monitor.

The typical pattern for condition synchronization involves a shared predicate variable and a `while` loop guarding a `wait()` call:

```java
synchronized (lock) {
    while (!condition) {
        lock.wait();
    }
    // Proceed when condition is true
}
```

This usage ensures correctness in the presence of **spurious wakeups**, where a thread may return from `wait()` even if it was not explicitly notified.

**It has the following key properties**:
* A thread must hold the monitor before calling `wait()`, `notify()`, or `notifyAll()`. Otherwise, a `java.lang.IllegalMonitorStateException` is thrown.
* The monitor’s condition queue is shared for all waiting threads. `notify()` does not distinguish which thread to wake; it may wake a thread that cannot proceed, leading to inefficiencies or stalling unless `notifyAll()` is used appropriately.
* Unlike semaphores, `notify()` does **not persist**. If no thread is waiting when `notify()` is called, the signal is lost.


While Java monitors provide a simple and safe mechanism for concurrency control, they are limited to **a single condition queue per object**. For more advanced control (e.g., multiple condition queues, fine-grained control), Java provides the `java.util.concurrent.locks.Condition` interface in conjunction with `ReentrantLock`.



### Scenario

The following class models a one-shot event between a producer and a consumer. The producer sets a flag and calls `notify()` to wake the consumer.

```java
class OneShotEvent {
    private boolean ready = false;
    private final Object lock = new Object();

    public void produce() {
        System.out.println("Producer's running")
        synchronized (lock) {
            ready = true;
            lock.notify(); // signal consumer
        }
    }

    public void consume()  {
    
        System.out.println("Consumer's running")
        while (!ready){
            synchronized(lock){
              lock.wait(); // wait for signal
            }
        }
        System.out.println("Consumed");

    }
}
```

This test is run:

```java
OneShotEvent event = new OneShotEvent();
new Thread(() -> {
    try {
        event.consume();
    } catch (InterruptedException e) {}
}).start();


System.out.println("Producer Begins")
event.produce();
```

**Observation**: Sometimes the program **hangs** and the message `"Consumed"` is never printed.


**Answer the following questions:**
1. Why does the `consume()` method sometimes block forever?
2. What happens if `produce()` calls `notify()` before `consume()` reaches `wait()`?
3. How does this differ from a semaphore-based approach?
4. Modify the code **minimally** so that no missed signal occurs, using only Java synchronization.

{:.highlight}
> **Hints**:
> * `notify()` only works if a thread is already waiting.
> * Always use `while`, not `if`, with `wait()`.
> * Semaphores **store** permits, condition variables do not.


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">

<p>The <code>consume()</code> method may block forever because it checks <code>ready</code>, sees it is false, and calls <code>wait()</code>. But if the <code>produce()</code> method is in the middle of execution and was about to set the <code>ready</code> flag, <span class="orange-bold">that signal is lost</span> as the consumer was not waiting yet. The consumer then goes to **sleep** with no one left to wake it up.</p>

<p>So in this scenario: a **race condition** that occur on the `ready` flag, interleaved with producer's <code>produce()</code>: consumer checks for `ready` and it's falls, then producer sets `ready` to true and calls <code>notify()</code>, then switched back to consumer that now enters <code>wait()</code>. The `signal` does not get stored or queued. The monitor simply ignores it if no threads are waiting. In short, when consumer does call <code>wait()</code>, it sleeps forever because the one chance to wake it was already missed.</p>

<p>Semaphores behave differently. If a thread calls <code>release()</code> (or <code>signal()</code>) before another thread calls <code>acquire()</code> (or <code>wait()</code>), the signal is **saved** by incrementing a counter. When another thread eventually calls <code>acquire()</code>, it will proceed immediately. This persistent state <span class="orange-bold">prevents</span> missed signals.</p>

<p><strong>4.</strong> The minimal fix is to protect the entire code inside a `syncrhonized` method. This ensures that there's no race condition with the flag `ready`. When `notify()` is called, it will wake up a consumer thread **or** prevent a consumer thread from waiting using `ready` flag.</p>

<pre><code>
  public synchronized void consume() {
        System.out.println("Consumer's running")
        while (!ready) 
            lock.wait();
        System.out.println("Consumed");

  }
</code></pre>

<p>This guarantees correctness even if <code>notify()</code> is called before <code>wait()</code>, because the condition <code>ready == true</code> will skip the wait. The <code>while</code> also handles spurious wakeups correctly.</p>

</div><br>





## Can This Bounded Buffer Deadlock?

### Background

A classic bounded buffer allows multiple producers and consumers to share access to a fixed-size queue. In this Java implementation, threads use `wait()` and `notify()` to coordinate access.

Important properties:

* `wait()` suspends the current thread until it is notified.
* `notify()` wakes one thread waiting on the same monitor.
* All threads synchronize on a single shared `lock` object.
* There is no explicit condition variable: all threads wait on the same monitor.


### Scenario

The following implementation runs with multiple producer and consumer threads:

```java
class BoundedBuffer {
    private final Queue<Integer> buffer = new LinkedList<>();
    private final int CAPACITY = 5;
    private final Object lock = new Object();

    public void produce(int item) throws InterruptedException {
        synchronized (lock) {
            while (buffer.size() == CAPACITY)
                lock.wait();

            buffer.add(item);
            lock.notify(); // wake up one thread
        }
    }

    public int consume() throws InterruptedException {
        synchronized (lock) {
            while (buffer.isEmpty())
                lock.wait();

            int item = buffer.remove();
            lock.notify(); // wake up one thread
            return item;
        }
    }
}
```

At first glance, using `notify()` (not `notifyAll()`) seems risky: it wakes an arbitrary thread, even if that thread can’t make progress. **What if** producer mistakenly wake up *another producer thread* instead of a consumer thread? 




**Answer the following question:**
Is it possible for this program to enter a **deadlock** where no thread makes progress, despite threads being alive and correctly synchronized?

{:.note}
> To make a clear answer, either:
> * Provide a concrete example of a buffer state and thread state that causes deadlock, **or**
> * Give a convincing **proof** that no such deadlock is possible, explaining why progress is always guaranteed.

{:.highlight}
> **Hint**:
> * What condition causes a thread to block?
> * Who can unblock it?
> * Can `notify()` ever wake the “wrong” thread?
> * Does that matter if only one kind of thread is waiting at a time?



<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">

<p><strong>No, this program cannot deadlock.</strong></p>

<p><strong>Proof by invariant and exclusion:</strong></p>

<ul>
  <li>A <strong>producer</strong> only blocks if the buffer is <strong>full</strong> (size == CAPACITY).</li>
  <li>A <strong>consumer</strong> only blocks if the buffer is <strong>empty</strong> (size == 0).</li>
  <li>Therefore, both types of threads <strong>cannot be blocked at the same time</strong> — the buffer cannot be both full and empty.</li>
  <li>When a consumer removes an item, it calls <code>notify()</code>. Since at that point, only producers can be waiting (because buffer was full), the <code>notify()</code> wakes one producer, which can now proceed (buffer has space).</li>
  <li>Likewise, when a producer adds an item and calls <code>notify()</code>, only consumers may be waiting (buffer was empty), and the consumer can now proceed.</li>
</ul>

<p>Thus, even though <code>notify()</code> wakes an arbitrary thread, the only threads that will be waiting are those that <strong>can make progress</strong> once notified.</p>

<p><strong>Therefore, no combination of buffer state and thread state leads to deadlock.</strong> The system always makes progress, and deadlock is impossible under this design.</p>

</div><br>



## Circular Wait in Print Spooler and Scanner

### Background

**Deadlock** is a state where a group of processes is unable to proceed because each is waiting for a resource held by another. Four conditions we learned in class (mutex, circular wait, no-preemption, hold-and-wait) must hold simultaneously for a deadlock to occur. If all four conditions are present, the system can reach a deadlocked state.

{:.note}
These are formally known as the **Coffman conditions**. 

### Scenario

A user launches two applications at the same time:

* **App A** locks the **scanner** first, then tries to lock the **printer**.
* **App B** locks the **printer** first, then tries to lock the **scanner**.

Each device is protected by a mutex. Sometimes, both applications stop responding and remain blocked forever.

The system has only one printer and one scanner. The simplified pseudocode for both apps is shown below.

```c
// App A
lock(scanner);
sleep(1);  // simulate time gap
lock(printer);
... // scan and print
unlock(printer);
unlock(scanner);

// App B
lock(printer);
sleep(1);  // simulate time gap
lock(scanner);
... // print and scan
unlock(scanner);
unlock(printer);
```


**Answer the following questions:**
1. Identify how each of the Coffman conditions is satisfied in this system.
2. Provide a step-by-step explanation of how the two applications could enter a deadlock.
3. Suggest two alternative designs that could prevent this deadlock and explain why they work.
4. Suppose the printer driver is updated to include a log message before allowing access, which adds a short delay. How could this change affect the likelihood of deadlock?


{:.highlight}
> **Hints**:
> * Consider how timing affects the acquisition order.
> * Draw a resource allocation graph if needed.
> * Use the Coffman conditions to reason through both cause and prevention.


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>All four Coffman conditions are satisfied:</p>
<ul>
  <li><strong>Mutual exclusion:</strong> The printer and scanner are mutex-protected, so only one application can hold each at a time.</li>
  <li><strong>Hold and wait:</strong> App A holds the scanner and waits for the printer. App B holds the printer and waits for the scanner.</li>
  <li><strong>No preemption:</strong> Once an app holds a device, it will not release it unless it finishes its task. The OS does not forcibly take resources away.</li>
  <li><strong>Circular wait:</strong> App A is waiting for the printer held by App B, and App B is waiting for the scanner held by App A. This forms a cycle.</li>
</ul>

<p>Deadlock sequence:</p>
<ol>
  <li>App A locks the scanner.</li>
  <li>App B locks the printer.</li>
  <li>App A attempts to lock the printer but is blocked since App B holds it.</li>
  <li>App B attempts to lock the scanner but is blocked since App A holds it.</li>
  <li>Neither app can proceed. A deadlock occurs.</li>
</ol>

<p>Two alternative designs:</p>
<ul>
  <li><strong>Global resource ordering:</strong> Require all applications to acquire resources in a fixed order. For example, always lock the printer before locking the scanner. This prevents circular wait.</li>
  <li><strong>Try-lock and backoff:</strong> Use a non-blocking attempt to lock the second device. If the second lock fails, release the first and retry later after sleeping. This prevents hold and wait.</li>
</ul>

<p>If the printer driver introduces a delay due to logging, App B may take longer to acquire the printer. During that delay, App A is more likely to acquire the scanner and reach the point of attempting to acquire the printer. This increases the window in which both applications hold one device each and wait for the other, raising the chance of deadlock.</p>


</p></div><br>


## Banker's Algorithm: Approve or Deny?

### Background

In operating systems, **Banker's Algorithm** is used to avoid deadlocks by checking whether a resource request will leave the system in a **safe state**. A safe state is one in which all processes can finish in some order, even if they do not finish immediately.

The algorithm works with four matrices or vectors:

* **Available** — the number of free instances of each resource.
* **Max** — the maximum demand each process may eventually request.
* **Allocation** — the number of resources currently held by each process.
* **Need** — the remaining resources needed, calculated as `Max - Allocation`.

Before granting a request, the system simulates the allocation and checks whether a **safe sequence** of completion exists for all processes. If so, the request is granted. If not, it is denied to avoid the risk of entering an **unsafe** state that could lead to deadlock.


### Scenario

A system has three resource types: A, B, and C. The total number of instances for each resource is:

* A: 10
* B: 5
* C: 7

There are five processes (P0 to P4). The current state of the system is as follows:

#### Allocation Matrix

| Process | A | B | C |
| ------- | - | - | - |
| P0      | 0 | 1 | 0 |
| P1      | 2 | 0 | 0 |
| P2      | 3 | 0 | 2 |
| P3      | 2 | 1 | 1 |
| P4      | 0 | 0 | 2 |

#### Max Matrix

| Process | A | B | C |
| ------- | - | - | - |
| P0      | 7 | 5 | 3 |
| P1      | 3 | 2 | 2 |
| P2      | 9 | 0 | 2 |
| P3      | 2 | 2 | 2 |
| P4      | 4 | 3 | 3 |

The **Available** vector is:

* A: 3
* B: 3
* C: 2

Now, process **P1** makes a request for resources: **(1, 0, 2)**.


**Answer the following questions:**
1. Compute the **Need** matrix for each process.
2. Should the system grant P1's request? Show your reasoning using the Banker's safety check.
3. If the request is granted and P1 finishes and releases all its resources, what will the new Available vector be?
4. Explain why entering an unsafe state does not immediately cause deadlock but is still dangerous.

{:.highlight}
> **Hints**:
> * A safe state guarantees at least one execution order where all processes can finish.
> * An unsafe state does *not* guarantee deadlock but makes it *possible*.
> * Holding resources without a guaranteed finish path creates future risk.
> * The number of requested units doesn't determine safety — the global state does.


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p> Need matrix is computed as Max - Allocation:</p>
<table border="1">
<tr><th>Process</th><th>A</th><th>B</th><th>C</th></tr>
<tr><td>P0</td><td>7</td><td>4</td><td>3</td></tr>
<tr><td>P1</td><td>1</td><td>2</td><td>2</td></tr>
<tr><td>P2</td><td>6</td><td>0</td><td>0</td></tr>
<tr><td>P3</td><td>0</td><td>1</td><td>1</td></tr>
<tr><td>P4</td><td>4</td><td>3</td><td>1</td></tr>
</table>

<p>First check if P1's request is valid:</p>
<ul>
  <li>Request ≤ Need: (1 ≤ 1, 0 ≤ 2, 2 ≤ 2) — valid</li>
  <li>Request ≤ Available: (1 ≤ 3, 0 ≤ 3, 2 ≤ 2) — valid</li>
</ul>

<p>Simulate granting the request:</p>
<ul>
  <li>New Available: (3 - 1, 3 - 0, 2 - 2) = (2, 3, 0)</li>
  <li>New Allocation for P1: (2 + 1, 0 + 0, 0 + 2) = (3, 0, 2)</li>
  <li>New Need for P1: (1 - 1, 2 - 0, 2 - 2) = (0, 2, 0)</li>
</ul>

<p>Now check for a **safe** sequence:</p>
<ul>
  <li>P3: Need (0, 1, 1) - **cannot** proceed, needs 1 unit of C but C = 0</li>
  <li>P1: Need (0, 2, 0) - fits, gets resources and finishes</li>
  <li>**Updated** Available: (2 + 3, 3 + 0, 0 + 2) = (5, 3, 2)</li>
  <li>P3: Now has enough, finishes and adds (2, 1, 1) → (7, 4, 3)</li>
  <li>P0: Need (7, 4, 3) - fits, finishes → Available becomes (7, 5, 3)</li>
  <li>P2: Need (6, 0, 0) - fits, finishes → (10, 5, 5)</li>
  <li>P4: Need (4, 3, 1) - fits, finishes → all processes done</li>
</ul>

<p>The **safe** sequence is therefore: P1, P3, P0, P2, P4.</p>

<p><strong>Conclusion:</strong> Yes, the system remains in a safe state. The request should be granted.</p>

<p>If P1 finishes, it releases (3, 0, 2), so Available becomes:<br>
(3 + 3, 3 + 0, 2 + 2) = <strong>(6, 3, 4)</strong></p>

<p>Entering an unsafe state means the system cannot guarantee that all processes will finish. A deadlock may not happen immediately, but if future requests are made and the wrong process proceeds, the system may become stuck with no way to satisfy some processes. Banker's Algorithm avoids this by rejecting requests that lead to this uncertainty.</p>


</p></div><br>

## The Unsafe Shortcut

### Background 
Deadlock avoidance techniques prevent the system from entering states where deadlock *could* happen. One such approach is the Banker's Algorithm, which checks whether a resource request leaves the system in a "safe state". 

{:.note}
**Safe state**: a state where all processes can eventually finish.

### Scenario
A system uses the Banker's Algorithm to approve resource requests. To optimize performance under load, a developer modifies the algorithm to skip the safe-state check for requests that appear "small" (e.g., requesting only one unit). At first, the system continues to run fine. But under heavy load, it freezes entirely. Post-mortem analysis shows multiple unfinished processes holding resources and waiting ***indefinitely***.

**Example**
Suppose the system has 2 identical units of a resource and 3 processes:

* P1 may request up to 2
* P2 may request up to 1
* P3 may request up to 1

The current allocation is:

* P1 has 0
* P2 has 1
* P3 has 0

If P3 now requests 1, the system has only 1 unit left. Granting it would leave 0 free units, and no process can finish.
This is **unsafe**, but <span class="orange-bold">not yet deadlocked</span>. If P3 finishes quickly and releases its resource, the system can recover.

{:.highlight}
However, if P1 then requests 2, and P2 is still holding its 1 unit, **no one can proceed**.
This is a **deadlock**.

**Answer the following questions**
1. What is the difference between an *unsafe state* and a *deadlocked state*?
2. Explain how entering an unsafe state can eventually lead to deadlock.
3. In the scenario above, why is skipping the safety check dangerous even if the request seems small?
4. Suppose the system grants a request and enters an unsafe state. Describe one possible sequence of events that leads to deadlock.
5. Suggest one modification to improve performance *without* removing the safety check.

{:.highlight}
> **Hints**:
> * A safe state guarantees at least one execution order where all processes can finish.
> * An unsafe state does *not* guarantee deadlock but makes it *possible*.
> * Holding resources without a guaranteed finish path creates future risk.
> * The number of requested units doesn't determine safety — the global state does.


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
An <strong>unsafe state</strong> means that the system cannot guarantee that all processes can complete; however, it may still be possible to avoid deadlock through careful scheduling. A <strong>deadlocked state</strong> means that a set of processes are waiting on each other in a cycle and none can proceed.
</p>

<p>
If the system enters an unsafe state and future resource requests do not arrive in a favorable order, it can lead to a situation where some processes hold resources and wait indefinitely, completing the conditions for deadlock.
</p>

<p>
Even small requests can cause the system to enter an unsafe state. The Banker's Algorithm considers the entire system state, not just the size of an individual request. Skipping the check removes the guarantee of safety.
</p>

<p>
Suppose Process A is granted a request without a safety check. The system enters an unsafe state. Later, Process B requests resources, but they are held by A. A is now waiting on resources held by Process C. If this forms a wait cycle and no process can finish, a deadlock occurs.
</p>

<p>
One approach is to cache safe-state results for common request patterns or to parallelize the safety check itself. Another strategy is to batch requests and analyze them in bulk while still performing safety checks before granting.
</p>


</p></div><br>



## Hidden Cycles in Resource Allocation Graph

### Background

A **Resource Allocation Graph (RAG)** is a <span class="orange-bold">directed</span> graph used to visualize how processes and resources interact in a system. It consists of:

* **Processes** (circles)
* **Resources** (squares)
* **Request edges** from a process to a resource
* **Assignment edges** from a resource to a process

In a system where each resource has exactly one instance, a **cycle** in the graph implies a deadlock. However, if a resource has multiple instances, a cycle does <span class="orange-bold">**not necessarily**</span> mean deadlock. 

{:.important}
The graph must be analyzed more carefully to distinguish between **true deadlocks** and **safe but cyclic** states.


### Scenario

You are given the following snapshot of a system. It includes three processes (P1, P2, P3) and two resource types:
* **R1** has 2 instances
* **R2** has 1 instance

Current graph state:
* P1 holds 1 instance of R1 and is requesting R2
* P2 holds R2 and is requesting 1 instance of R1
* P3 holds 1 instance of R1 and is not requesting anything

Graphically, this forms a cycle:

* P1 → R2 → P2 → R1 → P1

> But R1 has 2 instances, and one is held by P3.


**Answer the following questions:**
1. Draw the resource allocation graph based on the description. Label request and assignment edges clearly.
2. Does this graph represent a deadlock? Justify your answer using resource instance counts.
3. What is the key difference in reasoning when resources have multiple instances compared to when each has only one?
4. How would adding another instance of R2 affect the potential for deadlock in this graph?

{:.highlight}
> **Hints**:
>
> * A cycle in a resource allocation graph is only sufficient to prove deadlock when all resources have one instance.
> * If a resource has multiple instances, you must check whether the processes can eventually proceed.
> * Track how many units are held and whether a process’s request can be fulfilled with the remaining instances.
> * Just because a cycle exists does not mean the system is stuck.

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>The resource allocation graph has the following structure:</p>
<ul>
  <li>P1 holds 1 instance of R1 → R1 has an assignment edge to P1</li>
  <li>P1 is requesting R2 → request edge from P1 to R2</li>
  <li>P2 holds R2 → R2 has an assignment edge to P2</li>
  <li>P2 is requesting R1 → request edge from P2 to R1</li>
  <li>P3 holds 1 instance of R1 → R1 has an assignment edge to P3</li>
</ul>
<p>This forms a cycle: P1 → R2 → P2 → R1 → P1</p>

<p>This graph does <span class="orange-bold">not</span> represent a deadlock. Although a cycle exists, R1 has two instances. One is held by P1 and one is held by P3. P2 is requesting one instance of R1. If P3 releases its R1 instance, then P2 can proceed, finish, and release R2, allowing P1 to continue. Therefore, progress is still possible, and the system is not deadlocked.</p>

<p>When each resource has only one instance, any cycle in the graph means a deadlock. With multiple instances, a cycle is not sufficient. You must analyze whether resource demands can be satisfied. The system may still be able to progress if at least one process can complete and release its resources.</p>

<p>If R2 had one more instance, then the request by P1 for R2 could be satisfied immediately. P1 would proceed, finish, and release R1. This would allow P2’s request for R1 to be granted next. Adding another instance of R2 therefore reduces the risk of deadlock and may eliminate the cycle altogether depending on timing.</p>


</p></div><br>

## Dining Savages with Broken Pot Semaphore

### Background

The **Dining Savages Problem** is a *classic* concurrency scenario involving a group of threads (savages) sharing a common pot of food. A **cook** thread refills the pot when it is empty. To coordinate correctly, synchronization primitives such as **mutexes** and **semaphores** are used.

Each savage must:

1. Acquire a lock to check the pot.
2. If food is available, take one serving.
3. If the pot is empty, signal the cook to refill it.
4. Wait until food becomes available again.

{:.note}
If the synchronization is incorrect; for example, if a **semaphore is missing or misused** then the system can enter a **deadlock** where savages wait indefinitely and the cook is never notified.


### Scenario

Consider the following buggy implementation:

```c
semaphore serving_available = 0; 
semaphore empty = 1; // used to wake up cook 
mutex mtx = 1; // used by savages to protect servings_eaten 
int servings_eaten = 0; // shared resource among savages

// savage code
while (true) {
    wait(serving_available);
    eat();

    wait(mtx);
    servings_eaten++;
    if (servings_eaten == N) {
        servings_eaten = 0;
        signal(empty);
    }
    signal(mtx);
}

// cook code
while (true) {
    wait(empty);
    cook();
    signal(serving_available);
}
```


However, after some time, the program <span class="orange-bold">deadlocks</span>. All savage threads are **blocked**, and the cook is **idle**.



**Answer the following questions:**
1. Explain the purpose of each semaphore and **how** they coordinate between the savages and the cook.
2. Describe the condition (scenario) that leads the system into a deadlock. 
3. Why is it incorrect to simply call `signal(serving_available)` once?
4. Modify the synchronization so that deadlock is mitigated. The modification should be as minimal as possible. 

{:.highlight}
> **Hints**:
>
> * Only **one** savage should signal the cook when the pot is empty.
> * Semaphores do not queue extra signals if no one is waiting.
> * The cook only wakes one savage after refilling and the others remain blocked with the current implementation.
> * Think about whether `serving_available` should count the number of available servings.


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>The <code>empty</code> semaphore is used by savages to notify the cook when the pot is empty. The <code>full</code> semaphore is used by the cook to signal back to the waiting savage that the pot has been refilled.</p>

<p>The deadlock occurs because only one savage is woken after the pot is refilled. That savage proceeds and takes one serving, but no other savage is signaled. If that savage consumes the only <code>serving_available</code> signal and more savages find the pot empty again, they each signal <code>empty</code>, but the cook is already asleep. Since semaphores do not count signals when no thread is waiting, the cook remains idle and all savages are blocked.</p>

<p>It is incorrect to call <code>signal(serving_available)</code> only once because there are **multiple** savages. Semaphores are counters. If only one <code>full</code> signal is given, only one savage can proceed. The others will block indefinitely on <code>wait(serving_available)</code>. 

<p>To fix this, <code>full</code> should represent the number of servings in the pot. The cook should <code>signal(serving_available)</code> **N** times after refilling. Each savage then <code>wait(serving_available)</code> before taking a serving. This ensures that N servings can be consumed by N savages, one per signal. Also, only one savage should be allowed to signal <code>empty</code> to prevent duplicate cook notifications. This can be done with an additional `mtx` semaphore to indicate that a refill is already in progress.</p>


</p></div><br>

### Epilogue 

The above solution to the dining savages problem *works*, but it requires you to `signal` many times as the semaphore `servings_available` represents the number of servings available for each savage. Below is an alternative solution that uses `int servings` variable instead of using the semaphore to **count** the servings. Either solution works. 


```c
// Shared variables
int servings = 0;
mutex pot_mutex;
semaphore empty = 0;     // used to wake up cook
semaphore full = 0;      // used to wake up savages

// Savage thread
while (true) {
    wait(pot_mutex);
    if (servings == 0) {
        signal(empty);   // notify cook
        wait(full);      // wait for refill
    }
    servings--;
    signal(pot_mutex);
    eat();
}

// Cook thread
while (true) {
    wait(empty);         // wait until pot is empty
    cook();
    servings = N;        // refill pot
    signal(full);        // notify savage
}
```

## Preemption and Priority Inversion in Lock Acquisition



### Background

In preemptive operating systems with **priority scheduling**, a high-priority process can interrupt a low-priority one to ensure timely execution. However, if a **low-priority thread holds a lock** needed by a **high-priority thread**, and medium-priority threads keep preempting the low-priority one, the high-priority thread gets blocked. This scenario is called **priority inversion**.

<span class="orange-bold">Priority inversion violates scheduling expectations</span>. It may not cause a classical deadlock, but it leads to **indefinite blocking** of higher-priority threads. In real-time systems, this can cause serious issues. Some systems implement **priority inheritance** to solve the problem.


### Scenario

Three threads are running with the following priorities:
* **Low-priority T1** holds a lock `L` and is in its critical section.
* **High-priority T2** tries to acquire `L` and is blocked.
* **Medium-priority T3** does not need `L` but frequently runs and preempts T1.

{:.important}
Because T1 never gets CPU time, it <span class="orange-bold">cannot</span> release the lock. T2 remains blocked even though it has higher priority. This behavior is observed in some real-time systems like [Mars Pathfinder](https://en.wikipedia.org/wiki/Mars_Pathfinder).


**Answer the following questions:**
1. Why does T2 not proceed, even though it has higher priority than T1 and T3?
2. Does this situation meet the four Coffman conditions for deadlock? Why or why not?
3. How does **priority inheritance** help resolve this issue?
4. Suppose priority inheritance is not available. Suggest an alternate way to prevent such inversion scenarios.

{:.highlight}
> **Hints**:
> * Deadlock requires circular wait. Is that happening here?
> * T1 is not waiting for a resource, but still blocks T2 indirectly.
> * Inheritance temporarily boosts T1’s priority to match T2’s.
> * Consider how resource access policies can be redesigned to avoid this.

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>T2 cannot proceed because it is blocked waiting for lock <code>L</code>, which is held by T1. T1 is unable to run because it has the lowest priority and gets preempted by T3. Although T2 has the highest priority, it depends on T1 to release the lock, and T1 never gets CPU time to do so.</p>

<p>This situation does not satisfy all Coffman conditions. Specifically:</p>
<ul>
  <li><strong>Mutual exclusion:</strong> The lock is non-sharable — satisfied.</li>
  <li><strong>Hold and wait:</strong> T2 is waiting for the lock — satisfied.</li>
  <li><strong>No preemption:</strong> Locks are only released voluntarily — satisfied.</li>
  <li><strong>Circular wait:</strong> Not present. T1 is not waiting for anything. Therefore, this is not a classical deadlock, but it is a form of indefinite blocking.</li>
</ul>

<p>Priority inheritance allows T1 to temporarily inherit the higher priority of T2. This ensures that T1 will get CPU time and release the lock quickly. Once it does, T2 can proceed, and T1 reverts to its original priority. This eliminates the priority inversion.</p>

<p>If priority inheritance is unavailable, an alternative is to use a **priority ceiling protocol**, where any thread acquiring a lock temporarily runs at the highest priority of any thread that may need it. Another option is to design systems such that low-priority threads do not hold locks required by high-priority ones, or to avoid holding locks for extended periods. Using non-blocking data structures or message-passing models may also help avoid this situation entirely.</p>

</p></div><br>


## The Ostrich Algorithm

### Background

A **deadlock prevention strategy** often taught in operating systems is the **Ostrich Algorithm**, which simply ignores the problem (what a wonderful strategy!). The idea is that if deadlocks are *rare*, it may not be worth the overhead of avoiding or detecting them. This approach is named after the ostrich's supposed habit of "burying its head in the sand" when facing danger.

In practice, many general-purpose operating systems adopt this strategy for certain types of locks or resources. The assumption is that users or programs will avoid mistakes, or that occasional restarts are acceptable.

However, ignoring deadlocks is only safe when the cost of failure is low and the system can recover *gracefully*. In long-running systems or mission-critical applications, even a rare deadlock can be catastrophic.


### Scenario

A file server uses a multi-threaded model where threads acquire read and write locks on files. Occasionally, two threads acquire locks in different orders:

* **Thread A** locks File X and then requests File Y.
* **Thread B** locks File Y and then requests File X.

The system does not enforce lock ordering and does not use any deadlock detection. After a long uptime, the server becomes unresponsive. Investigation reveals that Thread A and Thread B are both blocked, each waiting on the other's lock.

The server does not crash but <span class="orange-bold">stops</span> processing requests. *This problem only happens once every few weeks*.


**Answer the following questions:**
1. Why is the Ostrich Algorithm sometimes chosen as a design strategy in operating systems?
2. What are the risks of using the Ostrich Algorithm in the file server scenario described?
3. Suggest one runtime strategy and one design-time strategy that could help reduce the likelihood of such deadlocks.
4. In what types of systems is the Ostrich Algorithm a reasonable tradeoff, and when is it irresponsible?

{:.highlight}
> **Hints**:
> * Think about the cost of prevention versus the cost of failure.
> * Is the system interactive, recoverable, or mission-critical?
> * Not all deadlocks are equally harmful — some are tolerable, others are fatal.
> * Runtime fixes involve detection or recovery, while design-time fixes involve resource usage rules.



<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>The Ostrich Algorithm is chosen when the probability of deadlock is low and the cost of preventing it is high. It avoids the complexity and overhead of prevention or detection mechanisms. This is common in systems where restarts are acceptable, such as desktop applications or non-critical services.</p>

<p>In the file server scenario, ignoring deadlock leads to a full system halt. Since threads can block each other indefinitely, the server becomes unresponsive without crashing. This is dangerous for a service expected to run continuously, as recovery requires manual intervention or automated restarts, which can lead to data inconsistency or service downtime.</p>

<p>Here's one runtime strategy suggestion:/p>
<ul>
  <li><strong>Runtime strategy:</strong> Implement a watchdog timer or deadlock detector that checks for threads blocked beyond a timeout and kills or resets them.</li>
  <li><strong>Design-time strategy:</strong> Enforce a strict lock ordering policy where all threads must acquire resources in a predefined sequence, such as by file ID order. This eliminates circular wait.</li>
</ul>

<p>The Ostrich Algorithm is reasonable in systems where failure is rare, recovery is easy, and data is not critical — for example, user interfaces or non-essential background processes. It is irresponsible in real-time systems, financial systems, or infrastructure services where reliability, uptime, and consistency are essential. In such systems, even rare deadlocks are unacceptable.</p>



</p></div><br>




## The Waiting Ring

### Background
One of the four necessary conditions for deadlock is *circular* wait: a set of processes each waiting for a resource held by the next. When this occurs and no process can proceed, the system enters a deadlocked state.

### Scenario
A messaging server uses five worker threads (T0 to T4) to handle tasks requiring two locks: one for input and one for output buffers. Due to legacy design, each thread acquires the input lock for its assigned buffer, processes some data, and then requests the output lock of the next thread in sequence.

Under certain timing conditions, all five threads hold their input **locks** and are **blocked** waiting for the output lock held by the next thread. **The system halts with no progress**.

**Answer the following questions**
1. Which deadlock condition(s) are clearly satisfied in this scenario?
2. Draw the wait-for graph involving the five threads and the two types of locks.
3. How do we know this situation is a deadlock, and not merely contention?
4. Describe a strategy to prevent this deadlock using lock acquisition ordering.

{: .highlight}
> **Hints**:
> * Always remember that deadlock requires four conditions to hold <span class="orange-bold">simultaneously</span>.
> * A wait-for graph shows who is waiting on whom.
> * A cycle in the wait-for graph implies deadlock.
> * Lock ordering is a common prevention strategy.



<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>

<p>
The system satisfies all four necessary conditions for deadlock:
<ul>
  <li><strong>Mutual exclusion</strong>: Locks are held exclusively by one thread.</li>
  <li><strong>Hold and wait</strong>: Each thread holds an input lock while requesting an output lock.</li>
  <li><strong>No preemption</strong>: Locks are not forcibly taken away.</li>
  <li><strong>Circular wait</strong>: T0 waits on T1, T1 on T2, ..., T4 on T0, forming a cycle.</li>
</ul>
</p>

<p>
Each thread Ti holds Lock Li (input) and waits for Lock L(i+1 mod 5) (output). The graph is a directed cycle:
<ul>
  <li>T0 → T1</li>
  <li>T1 → T2</li>
  <li>T2 → T3</li>
  <li>T3 → T4</li>
  <li>T4 → T0</li>
</ul>
This confirms a circular wait among all five threads.
</p>

<p>
Contention *becomes* deadlock when there is no possibility for any thread to proceed. In this case, each thread is waiting for a resource held by another, and none can make progress, which defines deadlock. The presence of a cycle in the wait-for graph confirms this.
</p>


<p>
To prevent deadlock, enforce a global lock acquisition <span class="orange-bold">order</span>. For example, assign a consistent numeric ordering to all locks and require threads to always acquire locks in increasing order. If all threads acquire input and output locks in that global order, circular wait is impossible.
</p>

</p></div><br>



## The Greedy Request

### Background
Deadlock can occur when processes hold resources while waiting for others. This is the *hold and wait* condition. Systems that allow incremental resource acquisition without constraints are particularly vulnerable under high contention.

### Scenario
A simulation engine uses thread pools to run batches of physics computations. Each thread requests resources for memory buffers, GPU time, and disk access — but requests them one by one, as needed.

To improve throughput, the engine starts aggressively launching more threads. Each thread acquires whatever resource it needs immediately and continues execution. Under pressure, threads begin holding onto one resource while waiting for another. Eventually, the system locks up completely.

**Answer the following questions**
1. Which of the four necessary deadlock conditions are present in this scenario?
2. Why does allowing threads to acquire resources incrementally make the system more prone to deadlock?
3. Propose a method to eliminate the hold-and-wait condition. What are the tradeoffs?
4. Suppose the system cannot eliminate hold-and-wait. What other mitigation options are available?

{: .highlight}
> **Hints**:
> * Deadlock arises from mutual exclusion, hold and wait, no preemption, and circular wait.
> * Acquiring resources one by one creates dependency chains.
> * Forcing threads to request all resources upfront eliminates hold-and-wait.
> * Timeout, ordering, or rollback may help reduce risk.


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
This system satisfies:
<ul>
  <li><strong>Mutual exclusion</strong>: Resources like GPU and disk cannot be shared.</li>
  <li><strong>Hold and wait</strong>: Threads hold resources while requesting more.</li>
  <li><strong>No preemption</strong>: Resources are only released voluntarily.</li>
  <li><strong>Circular wait</strong>: With enough threads, cycles of dependency can form.</li>
</ul>
</p>

<p>
When threads request resources one at a time, they may end up waiting for resources held by others who are also waiting. This forms chains of dependencies, which can close into cycles if not controlled.
</p>

<p>
A common method is to require all resources to be requested <em>at once</em>. If any resource is unavailable, the thread releases all it has and retries. This prevents hold-and-wait entirely.
<br><br>
The tradeoff is reduced concurrency and potential starvation: threads may repeatedly fail to acquire all needed resources, even if some are free.
</p>

<p>
If hold-and-wait cannot be removed:
<ul>
  <li>Use <strong>lock ordering</strong> to prevent circular wait.</li>
  <li>Apply <strong>timeouts</strong> or <strong>request denial</strong> policies.</li>
  <li>Use <strong>deadlock detection</strong> and recover by killing or rolling back threads.</li>
</ul>
</p>

</p></div><br>



## The Banker’s Mistake
### Background

The Banker's Algorithm is used to avoid deadlock by simulating allocations and only approving those that leave the system in a safe state — a state where all processes can eventually finish. A mistake in the algorithm's logic can admit requests that appear harmless but eventually trap the system in a deadlock.

### Scenario
A system is managing three types of resources (A, B, C) and four running processes. To optimize speed, a developer modifies the safety check to only verify if **at least one process** can finish after each allocation. The system continues to run, but under a specific sequence of requests, all processes become blocked and hold some resources. The Banker's Algorithm did not catch this unsafe path because it didn’t simulate all possible finishing orders.

### Answer the following questions
1. What is the purpose of simulating the safety check in the Banker's Algorithm?
2. Why is it incorrect to only check if one process can finish when evaluating safety?
3. Give a small example (2–3 processes, 2–3 resources) where granting a request appears safe at first glance but results in an unsafe state.
4. How can the Banker's Algorithm be modified to correctly detect unsafe allocations?

{: .highlight}
> **Hints**:
> * A safe state guarantees <em>all</em> processes can eventually finish.
> * Just one process finishing doesn't ensure overall safety.
> * The algorithm simulates a possible sequence where every process can finish in some order.
> * You must evaluate the system state <em>after</em> the hypothetical allocation before committing it.


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
The Banker's Algorithm simulates whether granting a resource request would leave the system in a <strong>safe state</strong> — meaning there exists at least one sequence of process completions where every process can finish. If no such sequence exists, the request is denied to avoid entering an unsafe (potentially deadlocked) state.
</p>

<p>
Just because one process can finish does not mean the remaining system will remain safe. It may be that only that process can finish, and once it releases resources, the rest are still stuck — leading to an unsafe or deadlocked state.
</p>

<p>
Suppose we have:
<ul>
  <li><strong>Resources available:</strong> A: 3, B: 2</li>
  <li><strong>P1</strong> – Max: A: 2 B:1, Allocated: A: 1 B:0 → Needs: A:1 B:1</li>
  <li><strong>P2</strong> – Max: A: 1 B:1, Allocated: A: 1 B:1 → Needs: A:0 B:0</li>
  <li><strong>P3</strong> – Max: A: 2 B:1, Allocated: A: 0 B:0 → Needs: A:2 B:1</li>
</ul>

Available = A:1, B:1

If we grant P3’s request of A:1, B:1 → Available becomes A:0, B:0.  
P2 can finish (needs 0), but after that:
<ul>
  <li>P1 still needs A:1 B:1 (unavailable)</li>
  <li>P3 still needs A:1 B:0 (A is still unavailable)</li>
</ul>
Now the system is stuck. Granting the request led to an <strong>unsafe state</strong>.
</p>

<p>
The Banker's Algorithm must simulate the *full* system:
<ul>
  <li>Start from the available vector after the hypothetical allocation.</li>
  <li>Find a process that can finish given current availability.</li>
  <li>Release its resources and repeat until all processes finish.</li>
</ul>
If no such sequence exists, the request must be denied.
</p>


</p></div><br>



## The Forgotten Detector

### Background

Not all systems prevent deadlock. Some detect it after the fact and try to recover. This works **well** if detection runs frequently enough. But if detection is delayed or misconfigured, the system can enter a deadlocked state and remain frozen <span class="orange-bold">until manual intervention</span>.

### Scenario

A database engine manages read/write locks using a resource allocation graph. It relies on a background thread to detect cycles every 60 seconds.

During a high-load batch import, several transactions begin holding read locks while requesting write locks. A cycle forms among four transactions, but the detection thread is delayed due to CPU starvation. No queries complete for several minutes, and the monitoring system marks the service as unhealthy.

### Answer the following questions

1. Why does the system deadlock in this scenario even though detection is enabled?
2. How does the delay in detection affect system reliability?
3. Suppose the detection thread uses the following code. What flaw might cause it to miss cycles under load?

```c
// simplified detector logic
while (1) {
    sleep(60);
    if (load_average() < 5.0) {
        check_for_cycles();  // scans graph for wait cycles
    }
}
```

4. Propose **two** improvements to make deadlock detection more robust in this kind of system.

{: .highlight}
> **Hints**:
> * Deadlock detection requires timely and accurate cycle checking.
> * Sleeping too long or skipping detection under high load makes it worse.
> * A cycle in the wait-for graph means no process in the cycle can proceed.
> * Starvation of the detector thread can delay resolution.


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
Detection only works if it runs frequently enough. In this case, a deadlock forms between transactions, but the detector thread is delayed or suppressed due to high CPU usage. While detection is configured, it is not timely, so the system remains deadlocked.
</p>

<p>
Delayed detection increases downtime and affects system availability. During this period, all transactions involved in the cycle are stuck. If the detector doesn’t run promptly, the system may appear unresponsive to users or monitoring tools.
</p>

<p>
The condition <code>if (load_average() &lt; 5.0)</code> causes the detector to skip checks under high load — which is <em>precisely</em> when deadlocks are more likely to occur. This creates a blind spot in the system during peak usage, defeating the purpose of detection.
</p>

<p>
Two possible improvements:
<ul>
  <li><strong>Run detection at fixed intervals regardless of load</strong>. Instead of checking load average, always perform the cycle detection every fixed time (e.g. every 10s).</li>
  <li><strong>Use event-based triggers</strong>. Instead of polling every interval, detect when transactions block for unusually long durations and trigger a graph analysis immediately.</li>
</ul>
</p>

</p></div><br>




## Deadlock in Bounded Buffer with Split Locks

### Background

A **deadlock** occurs when threads acquire locks in a <span class="orange-bold">circular dependency</span>, where each thread is holding a lock that another thread needs. This happens when:

* Two or more locks are used.
* Threads acquire the locks in different orders.
* There is no enforced lock acquisition protocol.

The safest way to avoid deadlocks is to **impose a consistent global lock order**, so all threads acquire multiple locks in the same sequence.


### Scenario

The following implementation splits the synchronization responsibilities across **two lock objects**, intending to separate producer and consumer paths:

```java
class BoundedBuffer {
    private final Queue<Integer> buffer = new LinkedList<>();
    private final int CAPACITY = 5;

    private final Object producerLock = new Object();
    private final Object consumerLock = new Object();

    public void produce(int item) throws InterruptedException {
        synchronized (producerLock) {
            while (true) {
                synchronized (consumerLock) {
                    if (buffer.size() < CAPACITY) {
                        buffer.add(item);
                        consumerLock.notify(); // wake a consumer
                        return;
                    }
                }
                producerLock.wait(); // wait until space available
            }
        }
    }

    public int consume() throws InterruptedException {
        synchronized (consumerLock) {
            while (true) {
                synchronized (producerLock) {
                    if (!buffer.isEmpty()) {
                        int item = buffer.remove();
                        producerLock.notify(); // wake a producer
                        return item;
                    }
                }
                consumerLock.wait(); // wait until item available
            }
        }
    }
}
```

This runs with **multiple** producer and consumer threads **concurrently**.


**Answer the following questions:**
1. Can this program deadlock? If yes, describe a concrete interleaving and lock state that leads to a deadlock.
2. Why does this happen, even though both `wait()` and `notify()` are used correctly?
3. What is a **lock ordering protocol**, and how does it prevent this deadlock?
4. Modify the code **minimally** to implement a lock ordering protocol. Assume both `producerLock` and `consumerLock` must still be used.

{:.highlight}
> **Hints**:
> * Focus on the order in which each method acquires the two locks.
> * Consider always locking one before the other, regardless of role.
> * You may refactor each method to acquire both locks together before any critical section.


<div cursor="pointer" class="collapsible">Show Answer</div>
<div class="content_answer">

<p>Yes, the program can deadlock.<strong>Example interleaving:</strong></p>
<ul>
  <li>Producer thread enters <code>produce()</code> and acquires <code>producerLock</code>.</li>
  <li>At the same time, consumer thread enters <code>consume()</code> and acquires <code>consumerLock</code>.</li>
  <li>Producer tries to acquire <code>consumerLock</code> but it’s held by the consumer.</li>
  <li>Consumer tries to acquire <code>producerLock</code> but it’s held by the producer.</li>
</ul>
<p>Now each thread waits for the other to release a lock, resulting in a deadlock.</p>

<p>This happens because the locks are acquired in **opposite orders**:  
Producers acquire <code>producerLock</code> → <code>consumerLock</code>,  
Consumers acquire <code>consumerLock</code> → <code>producerLock</code>.  
This creates the potential for circular wait.</p>

<p>A <strong>lock ordering protocol</strong> means enforcing that all threads always acquire locks in the same order.  
For example, always lock <code>consumerLock</code> before <code>producerLock</code> (or vice versa) in <em>both</em> methods.  
This prevents circular wait and guarantees deadlock freedom.</p>

<p><strong>4.</strong> Minimal fix using a lock ordering protocol (always acquire <code>consumerLock</code> before <code>producerLock</code>):</p>

<pre><code>
class BoundedBuffer {
    private final Queue<Integer> buffer = new LinkedList<>();
    private final int CAPACITY = 5;

    private final Object producerLock = new Object();
    private final Object consumerLock = new Object();

    public void produce(int item) throws InterruptedException {
        // Always acquire consumerLock before producerLock
        while (true) {
            synchronized (consumerLock) {
                synchronized (producerLock) {
                    if (buffer.size() < CAPACITY) {
                        buffer.add(item);
                        consumerLock.notify(); // signal a waiting consumer
                        return;
                    }
                }
                // Only wait while holding consumerLock
                consumerLock.wait(); // wait until consumer removes an item
            }
        }
    }

    public int consume() throws InterruptedException {
        while (true) {
            synchronized (consumerLock) {
                synchronized (producerLock) {
                    if (!buffer.isEmpty()) {
                        int item = buffer.remove();
                        producerLock.notify(); // signal a waiting producer
                        return item;
                    }
                }
                // Only wait while holding consumerLock
                consumerLock.wait(); // wait until producer adds an item
            }
        }
    }
}
</code></pre>

<p>This version:
<ul>
  <li>Always locks <code>consumerLock</code> before <code>producerLock</code>.</li>
  <li>Avoids holding both locks during <code>wait()</code>.</li>
  <li>Eliminates circular wait: hence prevents deadlock.</li>
</ul></p>

<p>Note that using two locks like this is <span class="orange-bold">inefficient</span>, but the question is just made for exercise purposes.</p>
</div><br>


