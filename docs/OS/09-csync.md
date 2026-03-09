---
layout: default
permalink: /os/c-sync
title: Synchronization in C
description: Methods on how processes and threads can be made cooperative in C
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
**Natalie Agus (Summer 2026)**

# Synchronization in C (POSIX Threads)

{.:highlight}
> Detailed Learning Objectives
>
> 1. **Examine the Monitor Pattern in C**
>    * Explain how a **monitor** is manually constructed in C by pairing a `pthread_mutex_t` with one or more `pthread_cond_t` variables.
>    * Contrast this with languages where monitors are built-in (e.g: Java's `synchronized`).
>    * Distinguish between protecting a critical section with a **single shared mutex** versus **per-resource (named) mutexes**.
>    * Discuss the use of **static** mutex variables and their role as class-level locks.
> 2. **Examine Condition Synchronization Patterns in C**
>    * Show the turn-based `doWork` pattern using `pthread_cond_wait` and `pthread_cond_broadcast`.
>    * Defend why **broadcast** (`pthread_cond_broadcast`) is sometimes necessary over **signal** (`pthread_cond_signal`).
>    * Reinforce why the condition must be checked in a **while loop**.
> 3. **Explain Reentrant Locks in C**
>    * Describe the difference between a default (non-reentrant) mutex and a **recursive** mutex (`PTHREAD_MUTEX_RECURSIVE`).
>    * Analyze scenarios where recursive mutexes are necessary and demonstrate **reentrance lockout** with default mutexes.
>    * Explain the lock **hold count** and why a recursive mutex must be unlocked the same number of times it was locked.
> 4. **Implement Fine-Grained Condition Synchronisation in C**
>    * Show how to create **multiple** `pthread_cond_t` variables associated with a **single** mutex for precise thread signalling.
>    * Apply these concepts to the N-thread turn-based example.
> 5. **Examine Proper Lock Release Patterns in C**
>    * Explain why C has **no** `finally` block and discuss idiomatic cleanup strategies (`goto cleanup`, single-return).
>    * Demonstrate how improper cleanup leads to **lock leaks** and **deadlocks**.
>
> These learning objectives build directly on the mutex, semaphore, and condition variable foundations from the [previous chapter](/50005/os/synchronization). The focus here is on **higher-level patterns**: monitors, reentrancy, fine-grained conditions, and robust cleanup implemented *entirely* in C with POSIX threads.



## The Monitor Pattern in C

> Monitor
>
> A **monitor** is a synchronization construct that bundles **mutual exclusion** (a lock) and **condition synchronization** (one or more condition variables) into a single logical unit. A thread <span class="orange-bold">must</span> hold the monitor's lock before it can wait on or signal any of its condition variables.

In some languages, every object **is** a monitor *automatically*: the language runtime manages the lock and condition variable behind the scenes. In C, there is no such magic. You build a monitor yourself by grouping a `pthread_mutex_t` with one or more `pthread_cond_t` variables, and you enforce the discipline manually.

This is a **good thing** pedagogically as every piece of the mechanism is visible.

A minimal monitor in C looks like this:

```c
typedef struct {
    pthread_mutex_t lock;
    pthread_cond_t  cond;
    /* shared state guarded by this monitor */
    int value;
} monitor_t;
```

Any function that accesses the shared state must first acquire `lock`, and any function that needs to sleep on a condition must call `pthread_cond_wait(&mon.cond, &mon.lock)`, which atomically releases the lock and puts the thread to sleep, just as we discussed in the previous chapter.

### Single Shared Mutex (Anonymous Style)

The simplest approach is to protect **all** shared state with one mutex. This is analogous to having every method in a class be `synchronized` on the same implicit lock.

```c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

void deposit(int amount) {
    pthread_mutex_lock(&mutex);
    /* critical section */
    balance += amount;
    pthread_mutex_unlock(&mutex);
}

void withdraw(int amount) {
    pthread_mutex_lock(&mutex);
    /* critical section */
    balance -= amount;
    pthread_mutex_unlock(&mutex);
}
```

Both `deposit` and `withdraw` share **one lock**. At most one thread can be inside either function at any time. This is simple but coarse-grained: even two threads calling `deposit` on **different** accounts would block each other.

### Per-Resource Mutex (Named Style)

For finer granularity, associate a **dedicated** mutex with each resource. This is analogous to using a named lock object rather than `this`.

```c
typedef struct {
    pthread_mutex_t lock;
    int balance;
} account_t;

void deposit(account_t *acct, int amount) {
    pthread_mutex_lock(&acct->lock);
    acct->balance += amount;
    pthread_mutex_unlock(&acct->lock);
}

void withdraw(account_t *acct, int amount) {
    pthread_mutex_lock(&acct->lock);
    acct->balance -= amount;
    pthread_mutex_unlock(&acct->lock);
}
```

Now two threads operating on **different** `account_t` instances proceed in parallel as they acquire different locks. Threads operating on the **same** account still serialise correctly.


## Static (Global) Locks

In some languages, a class can have a **class-level lock** separate from any instance lock, typically used by `static synchronized` methods. In C, the equivalent is a `static` or file-scope mutex:

```c
/* file scope: one lock shared by all calls, regardless of which "instance" */
static pthread_mutex_t class_lock = PTHREAD_MUTEX_INITIALIZER;

void class_wide_operation(void) {
    pthread_mutex_lock(&class_lock);
    /* critical section shared across ALL threads/instances */
    pthread_mutex_unlock(&class_lock);
}
```

This lock exists **once** per compilation unit (analogous to once per class). It is <span class="orange-bold">independent</span> of any per-instance lock you may also have. This distinction matters when you need to protect **shared static data** that does not belong to any single instance.

# Condition Synchronization with the `doWork` Example

Recall from the previous chapter that condition variables allow a thread to sleep until some **condition** becomes true, without busy waiting. Here we demonstrate a classic pattern: N threads share a `turn` variable, and only the thread whose `id == turn` may proceed. 

This is called condition synchronisation.


```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

#define N 5
#define ROUNDS 3

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t  cond  = PTHREAD_COND_INITIALIZER;
int turn = 0;

void *do_work(void *arg) {
    int id = *(int *)arg;

    for (int r = 0; r < ROUNDS; r++) {
        pthread_mutex_lock(&mutex);

        while (turn != id) {
            pthread_cond_wait(&cond, &mutex);
        }

        /* === CRITICAL SECTION === */
        printf("Thread %d: round %d\n", id, r);
        turn = (turn + 1) % N;

        pthread_cond_broadcast(&cond);
        pthread_mutex_unlock(&mutex);
    }

    return NULL;
}

int main(void) {
    pthread_t threads[N];
    int ids[N];

    for (int i = 0; i < N; i++) {
        ids[i] = i;
        pthread_create(&threads[i], NULL, do_work, &ids[i]);
    }
    for (int i = 0; i < N; i++) {
        pthread_join(threads[i], NULL);
    }

    pthread_mutex_destroy(&mutex);
    pthread_cond_destroy(&cond);
    return 0;
}
```

Key observations (make sure you understand the reasoning behind each point):
- The `mutex` must be **held** before calling `pthread_cond_wait`. This is a hard requirement because calling `wait` without holding the lock is **undefined behaviour**.
- `pthread_cond_wait` **atomically** releases the mutex and puts the thread to sleep. When the thread is woken up, it **re-acquires** the mutex before returning. This atomicity is critical: it prevents the "missed signal" problem described in the previous chapter.
- We use `pthread_cond_broadcast` (wake **all** waiters) instead of `pthread_cond_signal` (wake **one** waiter). The reason is discussed next.

## Broadcast vs Signal

`pthread_cond_signal` wakes up **one** arbitrary thread from the `wait` queue. There is <span class="orange-bold">no</span> guarantee that the woken thread is the one whose `id == turn`. If the wrong thread wakes up, it rechecks the `while` condition, finds `turn != id`, and goes back to sleep *but* the **correct** thread was never woken.

`pthread_cond_broadcast` wakes **all** waiting threads. Every thread rechecks its <span class="orange-bold">condition</span>; only the one with `id == turn` proceeds, the rest go back to sleep. This is less efficient but **correct**. We will show a more efficient approach using fine-grained conditions later in this chapter.

## Waiting in a Loop

{:.important}
It is **essential** to check the condition in a `while` loop, not an `if` statement. 

Two reasons behind this requirement:
1. **Spurious wakeup**: POSIX explicitly allows `pthread_cond_wait` to return even when no thread called `signal` or `broadcast`. This is permitted for performance reasons in certain implementations.
2. **Extraneous wakeup**: When using `broadcast`, multiple threads wake up but only one should proceed. The others must re-check and go back to sleep.

Using `if` instead of `while` would allow a thread to skip the condition check after a spurious or extraneous wakeup and enter the critical section incorrectly.

# Reentrant (Recursive) Mutex

A mutex is **reentrant** (or recursive) if the same thread can lock it **again** while already holding it, without deadlocking. The mutex maintains a **hold count**; each `lock` increments it, each `unlock` decrements it. The mutex is truly released only when the count reaches zero.

{:.note-title}
> Why reentrancy matters
>
> Consider a function `foo()` that acquires a lock and then calls `bar()`, which also needs to acquire the **same** lock (perhaps `bar` is also a public API that must be thread-safe on its own). With a non-reentrant (default) mutex, the second `lock` call **deadlocks** because the thread is waiting for a lock it already holds.

## Default Mutex and Reentrance Lockout

By default, `PTHREAD_MUTEX_INITIALIZER` creates a **non-recursive** mutex. Attempting to lock it twice from the same thread results in **undefined behaviour** and on most implementations, a deadlock:

```c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

void foo(int depth) {
    pthread_mutex_lock(&mutex);   /* first call: succeeds */
    printf("Locked at depth %d\n", depth);

    if (depth > 0) {
        foo(depth - 1);           /* DEADLOCK: same thread tries to lock again */
    }

    pthread_mutex_unlock(&mutex);
}
```

Calling `foo(2)` will print `"Locked at depth 2"` and then hang forever. The thread is blocked waiting on a lock **it already holds**. This is called **reentrance lockout**.

## Creating a Recursive Mutex

To allow reentrancy, initialise the mutex with the `PTHREAD_MUTEX_RECURSIVE` attribute:

```c
pthread_mutex_t mutex;
pthread_mutexattr_t attr;

void init_recursive_mutex(void) {
    pthread_mutexattr_init(&attr);
    pthread_mutexattr_settype(&attr, PTHREAD_MUTEX_RECURSIVE);
    pthread_mutex_init(&mutex, &attr);
    pthread_mutexattr_destroy(&attr);
}
```

Now the same recursive function works correctly:

```c
void foo(int depth) {
    pthread_mutex_lock(&mutex);   /* hold count incremented */
    printf("Locked at depth %d\n", depth);

    if (depth > 0) {
        foo(depth - 1);           /* same thread: hold count incremented again, no deadlock */
    }

    pthread_mutex_unlock(&mutex); /* hold count decremented */
}
```

Calling `foo(2)` will print depths 2, 1, 0 and return cleanly. The mutex is truly released after the **third** `unlock` (matching the three `lock` calls).

## Releasing Recursive Locks

An important detail: you must call `pthread_mutex_unlock` exactly **as many times** as you called `pthread_mutex_lock`. If you forget even one `unlock`, the hold count never reaches zero and **no other thread** can ever acquire the lock.

This is a common source of bugs, and is why the cleanup patterns discussed later in this chapter are so important.

# Fine-Grained Condition Synchronisation

The `doWork` example above used `pthread_cond_broadcast` to wake **all** threads, even though only one can proceed. This is wasteful when N is large. We can do better by creating **one condition variable per thread**, so that we signal only the specific thread that should proceed next.

A single mutex can be associated with **multiple** condition variables. This lets you have fine-grained control over **which** threads you wake:

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

#define N 5
#define ROUNDS 3

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t  cond_vars[N]; /* one condition per thread */
int turn = 0;

void *do_work(void *arg) {
    int id = *(int *)arg;

    for (int r = 0; r < ROUNDS; r++) {
        pthread_mutex_lock(&mutex);

        while (turn != id) {
            pthread_cond_wait(&cond_vars[id], &mutex);
        }

        /* === CRITICAL SECTION === */
        printf("Thread %d: round %d\n", id, r);
        turn = (turn + 1) % N;

        /* signal ONLY the next thread */
        pthread_cond_signal(&cond_vars[turn]);
        pthread_mutex_unlock(&mutex);
    }

    return NULL;
}

int main(void) {
    pthread_t threads[N];
    int ids[N];

    for (int i = 0; i < N; i++) {
        pthread_cond_init(&cond_vars[i], NULL);
        ids[i] = i;
        pthread_create(&threads[i], NULL, do_work, &ids[i]);
    }
    for (int i = 0; i < N; i++) {
        pthread_join(threads[i], NULL);
    }

    pthread_mutex_destroy(&mutex);
    for (int i = 0; i < N; i++) {
        pthread_cond_destroy(&cond_vars[i]);
    }
    return 0;
}
```

Observe the difference from the earlier version:

- Each thread waits on **its own** condition variable: `cond_vars[id]`.
- After updating `turn`, we signal **only** `cond_vars[turn]`. This is the exact thread that should go next.
- We use `pthread_cond_signal` (not broadcast) because exactly **one** thread is waiting on each condition variable.

This is the C equivalent of creating named `Condition` objects from a `ReentrantLock` in higher-level languages. The pattern is the same: associate multiple condition variables with a single lock for precise wakeup control.

# Proper Lock Release in C

{:.note}
> No `finally` in C
>
> Some languages provide a `finally` block that is **guaranteed** to execute regardless of exceptions. This makes it easy to ensure that a lock is always released. C has **no such mechanism**. If an error occurs between `lock` and `unlock`, and the code takes an early return or jumps past the `unlock`, the lock is **never released**. This is called a **lock leak** that will eventually deadlock the entire program.

This is an important practical concern. Let's look at idiomatic C patterns for handling this.

## The Problem: Lock Leak

```c
void dangerous_function(void) {
    pthread_mutex_lock(&mutex);

    int *data = malloc(1024);
    if (data == NULL) {
        return;  /* BUG: mutex is never unlocked! */
    }

    /* ... work with data ... */

    free(data);
    pthread_mutex_unlock(&mutex);
}
```

If `malloc` fails, the function returns early and the mutex stays locked forever.

## Pattern 1: Single Return with `goto cleanup`

The most common C idiom for ensuring cleanup is the `goto cleanup` pattern. All exit paths converge on a single cleanup label at the end of the function:

```c
void safe_function(void) {
    int *data = NULL;

    pthread_mutex_lock(&mutex);

    data = malloc(1024);
    if (data == NULL) {
        goto cleanup;
    }

    /* ... work with data ... */
    /* ... more operations that might fail ... */

cleanup:
    free(data); /* free(NULL) is safe */
    pthread_mutex_unlock(&mutex);
}
```

This guarantees the lock is **always** released, regardless of which error path is taken. It serves the same purpose as a `try-finally` block.

## Pattern 2: Minimise the Locked Region

An even better approach is to hold the lock for as **short** a time as possible. Move allocation, I/O, and other fallible operations **outside** the locked region:

```c
void better_function(void) {
    int *data = malloc(1024);
    if (data == NULL) {
        return; /* no lock held, safe to return */
    }

    pthread_mutex_lock(&mutex);
    /* only the actual shared-state manipulation is here */
    shared_value += process(data);
    pthread_mutex_unlock(&mutex);

    free(data);
}
```

This reduces the window where a lock leak is possible, and also improves concurrency by holding the lock for less time.

## Pattern 3: `pthread_cleanup_push` / `pthread_cleanup_pop`

POSIX provides a cleanup handler mechanism specifically for this purpose. It works similarly to a `finally` block, which is a registered handler runs when the thread is **cancelled** or when the cleanup scope is exited:

```c
void cleanup_unlock(void *arg) {
    pthread_mutex_unlock((pthread_mutex_t *)arg);
}

void robust_function(void) {
    pthread_mutex_lock(&mutex);
    pthread_cleanup_push(cleanup_unlock, &mutex);

    /* ... any work, including potential cancellation points ... */

    pthread_cleanup_pop(1); /* 1 = execute the handler (unlock) */
}
```

This is especially useful if your threads use **cancellation** (`pthread_cancel`), because a cancelled thread would otherwise never reach the `unlock` call.

# Atomic Wait Semantics

It is worth emphasising **why** `pthread_cond_wait` must atomically release the lock and put the thread to sleep. Consider what would happen if these were two separate operations:

```
/* HYPOTHETICAL non-atomic version — DO NOT USE */
pthread_mutex_unlock(&mutex);    /* step 1: release lock */
/* <<< WINDOW: another thread can acquire lock, change condition, and signal >>> */
sleep_on_condition(&cond);       /* step 2: go to sleep */
```

In the window between step 1 and step 2, another thread could acquire the lock, change the condition, and call `pthread_cond_signal`. The signal is **lost** because our thread has not entered the wait queue yet. The thread then sleeps forever, waiting for a signal that already happened.

POSIX guarantees that `pthread_cond_wait` performs the unlock-and-sleep as a **single atomic operation** with respect to other threads. There is no window for a missed signal. When the thread is eventually woken, it **re-acquires** the mutex before `pthread_cond_wait` returns.

This is the same guarantee that makes condition variables safe, and it is the reason you **must** hold the lock when calling `pthread_cond_wait`. If you try to sleep without the lock (for example, using `sleep()` or `usleep()`), the lock is **not** released, other threads cannot make progress, and you risk deadlock.

# Summary

This chapter translates the higher-level synchronization patterns into their C (POSIX threads) equivalents. The key concepts map as follows:

| Concept                        | C / pthreads                                               |
| ------------------------------ | ---------------------------------------------------------- |
| Monitor (lock + condvar)       | `pthread_mutex_t` + `pthread_cond_t` bundled in a struct   |
| Single shared lock             | One `pthread_mutex_t` for all functions                    |
| Per-resource lock              | A `pthread_mutex_t` inside each resource struct            |
| Class-level (static) lock      | `static pthread_mutex_t` at file scope                     |
| `wait()` / `notify()`          | `pthread_cond_wait()` / `pthread_cond_signal()`            |
| `notifyAll()`                  | `pthread_cond_broadcast()`                                 |
| Reentrant lock                 | `pthread_mutex_t` with `PTHREAD_MUTEX_RECURSIVE` attribute |
| Named condition variables      | Multiple `pthread_cond_t` sharing one `pthread_mutex_t`    |
| `try-finally` for lock release | `goto cleanup` / `pthread_cleanup_push`                    |

Key takeaways:

- **Monitors in C are explicit.** You must manually pair a mutex with condition variables and enforce the locking discipline yourself. This is more work than built-in monitors, but it makes every part of the synchronization mechanism visible.
- **Broadcast vs signal** matters. Use `pthread_cond_signal` when you know exactly which (or that only one) thread should wake. Use `pthread_cond_broadcast` when multiple threads wait on the same condition variable and you cannot target the right one. Fine-grained condition variables (one per thread) let you use `signal` precisely.
- **Recursive mutexes** allow the same thread to re-acquire a lock it already holds. Use them when you have recursive or re-entrant call patterns. Remember that every `lock` must be matched by an `unlock`.
- **C has no `finally`.** You are responsible for releasing locks on every code path, including error paths. Use `goto cleanup`, minimise locked regions, or use `pthread_cleanup_push` for robustness.


{:.note}
You are strongly recommended to try the toy code linked in Appendix below to further understand each pattern's behavior. We will ask questions with similar format in examination, where you are required to trace and debug if there exist a lock-leak or a deadlock.
