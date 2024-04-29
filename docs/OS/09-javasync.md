---
layout: default
permalink: /os/java-sync
title: Synchronization in Java
description: Methods on how processes and threads can be made cooperative in Java
parent: Operating System
nav_order: 9
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

# Synchronization in Java 

{:.highlight-title}
> Detailed Learning Objectives
> 
> - **Understand Java Synchronization Mechanisms**
>   - Recognize the role of synchronized methods and statements in Java.
>   - Identify the two basic synchronization idioms provided by Java: synchronized methods and synchronized statements.
> - **Grasp the Concept of Mutex Locks in Java Objects**
>   - Explain the purpose and function of mutex locks associated with Java objects.
>   - Describe how mutex locks are acquired and released through synchronized methods and blocks.
> - **Comprehend Thread Management and Locks**
>   - Understand the distinction between the entry set and wait set of threads.
>   - Analyze the conditions under which a thread can acquire an object lock and become runnable.
> - **Learn about Synchronized Methods and Blocks**
>   - Detail how to declare a synchronized method in Java and its implications.
>   - Explore the use of synchronized statements with specific named objects versus the anonymous 'this' instance.
> - **Explore Java Static and Reentrant Locks**
>   - Discuss the concept and application of static synchronized methods and their association with class-level locks.
>   - Explain the concept of reentrancy in locks and its practical application in Java programming.
> - **Understand Condition Synchronization**
>   - Examine Java's mechanisms for condition synchronization using `wait()` and `notify()` methods.
>   - Discuss the importance of managing thread states and conditions to prevent issues such as deadlocks and spurious wakeups.
> - **Differentiate Between Various Lock Types and Their Uses**
>   - Describe the differences between reentrant locks and non-reentrant locks.
>   - Analyze scenarios where reentrant locks are beneficial and situations where non-reentrant locks may cause problems.
> - **Implement Fine-Grained Synchronization Using Condition Variables**
>   - Illustrate the use of named conditional variables in conjunction with reentrant locks for more precise control over thread synchronization.
>   - Apply these concepts to practical coding scenarios to ensure correct synchronization and thread management.
> 
> These learning objectives are designed to help understand and apply Java's synchronization mechanisms effectively in concurrent programming scenarios.


## Java Object Mutex Lock

The Java programming language provides two basic synchronization idioms: synchronized <span style="color:#f77729;"><b>methods</b></span> and synchronized <span style="color:#f77729;"><b>statements</b></span>.

Each Java <span style="color:#f77729;"><b>object</b></span> has an associated binary <span style="color:#f77729;"><b>lock</b></span>:

- Lock is `acquired` by invoking a synchronized method/block
- Lock is `released` by exiting a synchronized method/block

With this lock, mutex is guaranteed for this object’s <span style="color:#f77729;"><b>method</b></span>; at most only one thread can be inside it at any time.
{:.note}

Threads <span style="color:#f77729;"><b>waiting</b></span> to acquire the object lock are waiting in the <span style="color:#f7007f;"><b>entry</b></span> set, status is still `blocked` and not runnable until it acquires the lock. Once the thread acquires the lock, it becomes `runnable`.

The wait set is NOT equal to the entry set -- it contains threads that are <span style="color:#f7007f;"><b>waiting for a certain condition</b></span>.

This "certain" condition is **not** the lock. 
{: .error}

<img src="{{ site.baseurl }}/assets/images/week4/3.png"  class="center_seventy"/>

These sets (entry and waiting) are <span style="color:#f7007f;"><b>per object</b></span>, meaning each object instance only has <span class="orange-bold">one</span> lock.

- Each object can have many synchronized methods.
- These methods share one lock.

### Synchronised Method (Anonymous)

Below is an example of how you can declare a synchronized method in a class. The mutex lock is <span style="color:#f7007f;"><b>itself</b></span> (`this`). The fact that we don't use other objects as a lock is the reason why we call this the <span style="color:#f77729;"><b>anonymous</b></span> synchronisation object.

```java
public synchronized returnType methodName(args)
{
       // Critical section here
       // ...
}
```

### Synchronised Statement using Named Object

And below is a synchronized block based on a <span style="color:#f77729;"><b>specific</b></span> named object (not `this` instance).

```cpp
Object mutexLock = new Object();
public returnType methodName(args)
{
   synchronized(mutexLock)
   {
       // Critical section here
       // ...
   }

   // Remainder section
}
```

You can also do a `synchronized(this)` if you'd like to use a synchronised statement anonymously.

## Java Static Locks

It is also possible to declare `static` methods as <span style="color:#f77729;"><b>synchronized</b></span>. This is because, along with the locks that are associated with object <span style="color:#f77729;"><b>instances</b></span>, there is a single class lock associated with <span style="color:#f77729;"><b>each</b></span> class.

The class lock has its own queue sets. Thus, for a given class, there can be several object locks, one per object instance. However, there is only one (static) class lock.
{:.warning}

# Java Condition Synchronization

Similarly, Java allows for condition synchronization using `wait()` and `notify()` method, as well as its own condition variables.

<span class="orange-bold">Example</span>: suppose a few threads are trying to execute the same method as follows, `N` threads are running this `doWork` function <span style="color:#f77729;"><b>concurrently</b></span> with argument `id` varying between `0` to `N-1`, i.e: 0 for thread 0, 1 for thread 1, and so on.


```java
public synchronized void doWork(int id)
{
   while (turn != id) // turn is a shared variable
   {
       try
       {
           wait();
       }
       catch (InterruptedException e)
       {
       }
   }
   // CRITICAL SECTION
   // ...
   turn = (turn + 1) % N;
   notify();
}
```


In this example, the condition in <span style="color:#f77729;"><b>question</b></span> is that only thread whose `id == turn` can execute the CS.
{: .highlight}

When calling `wait()`, the lock for mutual exclusion <span style="color:#f77729;"><b>must</b></span> be held by the caller (same as the conditional variable in the section above). That's why the `wait` to the conditional variable is made inside a `synchronized` method. If not, disaster might happen, for example the following execution sequence:

- At `t=0`,
  - Thread Y check that `turn != id_y`, and then Y is suspended by the asynchronous timer.
- At `t=n`,
  - Thread X <span style="color:#f77729;"><b>resumes</b></span> and <span style="color:#f77729;"><b>increments</b></span> the `turn`. This causes `turn == id_y`.
  - Suppose X then calls `notify()`, and then X is suspended
- At `t=n+m`,
  - Thread Y <span style="color:#f77729;"><b>resumes</b></span> execution and enters `wait()`.
- However at this time, the value of `turn` is ALREADY `== id_y`.

We can say that thread Y <span style="color:#f7007f;"><b>misses</b></span> the `notify()` from X and might `wait()` forever. We need to make sure that <span class="orange-bold">between</span> the check of `turn` and the call of `wait()`, <span class="orange-bold">no other thread</span> can change the value of `turn`.
{:.error}

Since we must invoke `wait()` while holding a lock, it is important for `wait()` to <span style="color:#f77729;"><b>release</b></span> the object lock eventually (when the thread is waiting).
If you call another method to `sleep` such as `Thread.yield() `instead of `wait()`, then it <span style="color:#f7007f;"><b>will not</b></span> release the lock while waiting. 

This is <span style="color:#f77729;"><b>dangerous</b></span> as it will result in indefinite waiting or <span style="color:#f7007f;"><b>deadlock</b></span>.
{:.error}

Upon return from `wait()`, the Java thread would have <span style="color:#f77729;"><b>re-acquired</b></span> the mutex lock <span style="color:#f77729;"><b>automatically</b></span>.

In summary,

- When a thread calls the `wait()` method, the following happens:

  - The thread <span style="color:#f77729;"><b>releases</b></span> the lock for the object.
  - The state of the thread is set to <span style="color:#f77729;"><b>blocked</b></span>.
  - The thread is placed in the <span style="color:#f77729;"><b>wait set</b></span> for the object.

- When it calls `notify()`:
  - Picks an <span style="color:#f77729;"><b>arbitrary</b></span> thread T from the list of threads in the wait set
  - Moves T from the wait set to the <span style="color:#f77729;"><b>entry</b></span> set
  - The state of T will be changed from blocked to <span style="color:#f77729;"><b>runnable</b></span>, so it can be scheduled to reacquire the mutex lock

## NotifyAll

Using only `notify()`, we are not completely free of another <span style="color:#f77729;"><b>potential</b></span> problem yet: `notify()` might not wake up the <span style="color:#f77729;"><b>correct</b></span> thread whose `id == turn`, and recall that `turn` is a shared variable.

We can solve the problem using another method `notifyAll()`: wakes up all threads in the wait set.  
{:.info}

## Waiting in a Loop

It is <span style="color:#f77729;"><b>important</b></span> to put the waiting of a condition variable in a `while` loop due to the possibility of:

1. <span style="color:#f77729;"><b>Spurious</b></span> wakeup: a thread might get woken up even though no thread signalled the condition (`POSIX` does this for performance reasons)
2. <span style="color:#f77729;"><b>Extraneous</b></span> wakeup: you woke up the correct amount of threads but some hasn’t been scheduled yet, so some other threads do the job first. For example:
   - There are <span style="color:#f77729;"><b>two</b></span> jobs in a queue, and there’s two threads: thread A and B that got woken up.
   - Thread A gets <span style="color:#f77729;"><b>scheduled</b></span>, and finishes the first job. It then finds the second job in the queue, and <span style="color:#f77729;"><b>finishes</b></span> it as well.
   - Thread B finally gets scheduled and, upon finding the queue empty, crashes.

# Reentrant Lock

A lock is re-entrant if it is <span style="color:#f77729;"><b>safe</b></span> to be acquired <span style="color:#f77729;"><b>again</b></span> by a caller that’s <span style="color:#f77729;"><b>already holding the lock</b></span>. You can create a `ReentrantLock()` object explicitly to allow reentrancy in your critical section.
{:.info}

Java synchronized methods and synchronized blocks with intrinsic lock <span style="color:#f7007f;"><b>are reentrant</b></span>. Recall that every object has an intrinsic lock associated with it.

For example, a thread can safely <span style="color:#f77729;"><b>recurse</b></span> on blocks guarded by reentrant locks (sync methods, sync statement):

```java
// Method 1
public synchronized void foo(int x) {
   // some condition ...
   foo(x-1); // recurse does not cause error
}

// Method 2
public void foo(int x) {
   synchronized(this) { // note that 'this' is the lock. Otherwise, non-reentrant
	// some condition ...
       foo(x-1); // recurse does not cause error
   }
}

// Method 3
Object obj1 = new Object();
synchronized(obj1){
   System.out.println("Try out object ack 1");
   synchronized(obj1){ // intrinsic lock is reentrant
       System.out.println("Try out obj ack 2"); // will be printed
   }
}
```

## Reentrance Lockout

If you use a <span style="color:#f77729;"><b>non-reentrant lock</b></span> and recurse, or try to acquire the lock again, you will suffer from <span style="color:#f7007f;"><b>reentrance lockout</b></span>. We demonstrate it with the custom lock below that's not reentrant:

```java
public class CustomLock{

 private boolean isLocked = false;

 public synchronized void lock() throws InterruptedException{
   while(isLocked){
     wait();
   }
   isLocked = true;
 }

 public synchronized void unlock(){
   isLocked = false;
   notify();
 }

}
```

And the usage:

```java
CustomLock lock = new CustomLock()

// Thread 1 code
public void doSomething(int argument){
   lock.lock();
   // recurse
   doSomething(argument-1);
   lock.unlock();
}
```

As a result, a thread that calls `lock()` for the first time will <span style="color:#f77729;"><b>succeed</b></span>, but the second call to lock() will be blocked since the variable `isLocked == true`.

## Releasing Locks

One final point to note is that some implementations require you to <span style="color:#f77729;"><b>release</b></span> the lock `N` times after <span style="color:#f77729;"><b>acquiring</b></span> it `N` times. 
{:.note}

You need to carefully <span style="color:#f77729;"><b>consult the documentation</b></span> to see if the locks are auto released or if you need to explicitly release a lock `N` times to allow others to successfully acquire it again.

## Fine-Grained Condition Synchronisation

If we want to perform fine grained condition synchronization, we can use Java's <span style="color:#f77729;"><b>named</b></span> conditional variables and a reentrant lock. Named conditional variables are created explicitly by first creating a `ReentrantLock()`. The template is as follows:

```java
Lock lock = new ReentrantLock();
Condition lockCondition = lock.newCondition(); // call this multiple times if you have more than 1 condition

// Step 1: LOCK
lock.lock(); // remember, need to lock before calling await()

// Step 2a: WAIT
// To wait for specific condition:
lockCondition.await();

// OR Step 2b: SIGNAL
// To signal specific thread waiting for this condition:
lockCondition.signal();
// ...
// ...

// Step 3: UNLOCK
lock.unlock();
```

At first, we associate a conditional variable with a lock: `lock.newCondition()`. This <span style="color:#f77729;"><b>forces</b></span> us to always hold a lock when a condition is being signaled or waited for.

We can modify the example above of N threads which can only progress if `id == turn` to use <span style="color:#f77729;"><b>conditional variables</b></span> as follows:

```java
// Create arrays of condition
Lock lock = new ReentrantLock();
ArrayList<Condition> condVars = new ArrayList<Condition>();
for(int i = 0; i<N; i++) condVars.add(lock.newCondition()); // create N conditions, one for each Thread
```

The Thread function is changed to incorporate a wait to each `condVars[id]`:

```java
// the function
public void doWork(int id)
{
   lock.lock();
   while (turn != id)
   {
       try
       {
             condVars[id].await();
	}
       catch (InterruptedException e){}
   }
   // CS
   // assume there's some work to be done here...
   turn = (turn + 1) % N;
   condVars[turn].signal();
   lock.unlock();
}
```

# Summary

This chapter delves into synchronization techniques in Java, essential for managing concurrent processes in programming. It explores mechanisms such as synchronized methods and blocks, static and reentrant locks, and condition synchronization using `wait()` and `notify()` methods. The guide covers practical aspects like mutex locks, thread management, and avoiding deadlocks, providing a comprehensive understanding of how Java handles synchronization to ensure thread safety and efficiency. 

Recall that there are a few <span style="color:#f77729;"><b>solutions</b></span> to the CS problem. The CS problem can be divided into two types:

- <span style="color:#f7007f;"><b>Mutual exclusion </b></span>
- <span style="color:#f7007f;"><b>Condition synchronization</b></span>

There are also a few solutions listed below.
1. <span style="color:#f77729;"><b>Software mutex algorithms</b></span> provide mutex via busy-wait.
2. <span style="color:#f77729;"><b>Hardware</b></span> synchronisation methods provide mutex via <span style="color:#f77729;"><b>atomic</b></span> instructions. Other software spinlocks and mutex algorithms can be derived using these special atomic assembly instructions
3. <span style="color:#f77729;"><b>Semaphores</b></span> provide <span style="color:#f77729;"><b>mutex</b></span> using binary semaphores.
4. Basic <span style="color:#f77729;"><b>condition</b></span> variables provide <span style="color:#f7007f;"><b>condition synchronization</b></span> and are used along with a mutex lock.
5. <span style="color:#f77729;"><b>Java anonymous</b></span> default synchronization object provides mutex using reentrant binary lock (`this`), and provides condition synchronisation using `wait()` and `notifyAll()` or `notify()`
6. <span style="color:#f77729;"><b>Java named synchronization</b></span> object provides mutex using named reentrant binary lock (`ReentrantLock()`) and provides condition sync using condition variables and `await()/signal()`

It is entirely up to you to figure out which one is suitable for your application.