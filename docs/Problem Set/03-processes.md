---
layout: default
permalink: /ps/3-processes-and-threads
title: Processes and Threads 
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

# Processes and Threads 
{: .no_toc}

## The Clone Conundrum


A student writes a C program that opens a file and then calls `fork()`. Both the parent and child continue to write to the file. However, the resulting file contents appear **garbled**, with overlapping or duplicated lines. The student is surprised. They expected the two processes to behave independently and becomes unsure about what `fork()` actually <span class="orange-bold">duplicates</span> and whether file descriptors or file offsets are shared. The unexpected corruption makes them question how resources are managed between parent and child.

### Background

In Unix-like operating systems, the `fork()` system call is used to create a new process by duplicating the calling process. The newly created process, known as the *child*, receives a copy of the parent’s memory space, register state, and file descriptor table. This allows the child to begin execution as a near-exact replica of the parent, resuming at the instruction immediately following the `fork()` call.

A subtle but important aspect of `fork()` is how it handles open file descriptors. While the file descriptor table is copied, the underlying file **descriptions** (which include the file offset and access mode) are **shared**. This means that if both parent and child write to the same file descriptor, they are writing through a shared offset, and their operations can interfere with one another unless properly synchronized.

Understanding how `fork()` interacts with file descriptors and other process resources is critical for writing correct concurrent programs. Misunderstandings in this area often lead to race conditions, data corruption, or unpredictable behavior, especially when multiple processes interact with shared I/O resources.

### Task
**Answer the following questions:**
1. What parts of a process are **copied** and what are **shared** between parent and child after a `fork()`?
2. Why can writing to the same file descriptor in both parent and child lead to **race conditions** or **data corruption**?
3. If the parent wants to **avoid interference**, what should they do after `fork()`?


{:.highlight}
> **Hints:**
> * `fork()` duplicates the **entire process memory space**, including file descriptors.
> * Open file descriptors refer to the same **underlying file offset** unless explicitly changed.
> * The child continues from the **same program counter** as the parent, immediately after the `fork()` call.
> * Use `close()` or `dup2()` strategically to manage file descriptor interference.


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
When a process calls `fork()`, the operating system creates a nearly identical copy of the parent process. This includes a copy of its memory space, register values, and open file descriptor table. However, the file descriptors in the child and parent point to the **same underlying file description**, which includes a shared file offset. As a result, if both parent and child write to the same file descriptor without coordination, their writes may overlap or interleave, leading to <span class="orange-bold">corrupted output</span>. This also known as race condition (next lesson's material)
<br><br>
The program counter is also duplicated, meaning both processes resume execution immediately after the `fork()` call. However, since `fork()` returns different values (0 in the child, PID in the parent), conditional logic can be used to differentiate behavior. To avoid unintended effects, such as duplicated file writes, either the parent or the child should close the file descriptor after `fork()`, or the code should implement synchronization mechanisms (e.g., `wait()`, pipes, or locks) to coordinate access.
</p></div><br>



## Mystery of the Missing Command

### Background


The `exec` family of system calls in Unix-like operating systems allows a running process to **replace its current memory image** with a new program. Unlike `fork()`, which creates a <span class="orange-bold">separate</span> child process, `exec()` transforms the current process into a new one without changing its process ID or creating a new execution thread. All code, data, and stack segments of the original program are replaced, and the process begins execution from the entry point of the new program.

This behavior has important implications in scripting contexts. When a shell script invokes `exec`, the shell process itself is <span class="orange-bold">replaced</span> by the specified program. As a result, control does not return to the script after the `exec` call. The script effectively **terminates** at that point. If `exec` is used at the top level of an interactive shell session, it can even terminate the session itself.

Understanding the semantics of `exec()` is critical for correctly managing control flow in scripts and for designing process lifecycles in larger systems. Misusing `exec()` often leads to confusion, as the process replacement is irreversible and does not allow the original program to resume execution.

### Scenario

While experimenting with shell scripts, a student writes a line that uses `exec ls`. Instead of listing the directory and continuing with the rest of the script, the shell seems to vanish, the script ends <span class="orange-bold">abruptly</span>, and control doesn’t return as expected. In some cases, the terminal session itself closes. The student is caught off guard and begins to suspect a bug, not realizing that `exec` fundamentally changes how the process behaves.



**Answer the following questions:**
1. What is the effect of calling `exec()` in a process that is already running?
2. How does `exec()` differ from `fork()` in terms of control flow and process replacement?
3. Why does using `exec ls` in a shell script cause the script to terminate afterward?

{:.highlight}
> **Hints:**
> * `exec()` replaces the **current process image** with a new one.
> * After a successful `exec()`, there is **no return** to the original code.
> * `exec` does **not create a new process**, it transforms the current one.
> * In a shell script, `exec ls` means the shell process is overwritten by `ls`.



<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
The `exec()` system call replaces the currently running process image with a new one. This means the memory, code, and execution context of the process are replaced entirely by the new program specified in the `exec` call. Unlike `fork()`, which creates a new process and allows both parent and child to run concurrently, `exec()` completely **replaces** the current process.
<br><br>
In the case of `exec ls` within a shell script, the script’s shell process is replaced by the `ls` program. Once `ls` completes, there is no original shell script process to return to, it was overwritten. This is not a bug, but rather expected behavior. If the student wants to **run `ls` and continue the script afterward**, they should omit `exec` and just call `ls`, which runs as a child process and returns control to the shell script when done.
</p></div><br>



## The Case of the Cloned Counter

You’re helping your teammate debug a strange issue in a C program that involves process creation. The program uses `fork()` to spawn a child process. Both parent and child increment a shared counter variable in a loop, and print it out to a file.

Here's a simplified version of the code:

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int counter = 0;

int main() {
    FILE *f = fopen("log.txt", "w");
    if (f == NULL) return 1;

    pid_t pid = fork();

    for (int i = 0; i < 5; i++) {
        counter++;
        fprintf(f, "PID %d: counter = %d\n", getpid(), counter);
    }

    if (pid > 0) wait(NULL);
    fclose(f);
    return 0;
}
```

When you inspect the contents of `log.txt`, you find that both processes wrote five lines, but the `counter` values overlap. For example, both processes print `counter = 1`, `counter = 2`, etc. You expected the values to continue incrementing from 6 to 10 in the child. This isn’t what happens.



**Answer the following questions:**
1. Explain why the `counter` variable is not shared between parent and child after the `fork()`.
2. Describe what happens to the **standard memory**, **open file descriptor**, and **file offset** in this program after `fork()`.
3. Suppose you want both processes to share and update the same `counter` variable. What approaches could you use? Compare at least two.
4. If the file writes had also interleaved or corrupted each other, what might be the cause? How would you fix it?


{:.highlight}
> **Hints:**
> * `fork()` duplicates memory: the parent and child get **separate copies** of global variables.
> * The `counter` in each process is an independent integer stored in separate address spaces.
> * `fopen()` returns a file descriptor that refers to the **same open file description** across `fork()`, meaning they share the **file offset**.
> * To truly share memory, consider using `mmap()` or a shared memory segment (e.g., `shm_open()`, `mmap()`), or use threads instead of processes.
> * File I/O issues could arise from **buffered output** or lack of flushing. Use `fflush(f)` or unbuffered I/O to ensure correctness.



<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
After a `fork()`, the child process receives a **copy of the parent’s entire memory**, including the `counter` variable. However, this is a *deep copy*, meaning any changes to `counter` in the parent are not seen by the child and vice versa, they operate on entirely separate copies. This is why both processes start incrementing `counter` from the same base value, producing overlapping output.
<br><br>
In contrast, `fopen()` returns a file stream based on a file descriptor, and `fork()` does **not duplicate the underlying open file**. It simply gives both processes file descriptors that refer to the same open file description. That means the **file offset** is shared, and without coordination, both processes can write to the same file position, leading to potential corruption.
<br><br>
To share a single `counter` between parent and child, you need to place it in a **shared memory region**. This could be done using `mmap()` with the `MAP_SHARED` flag, or via POSIX shared memory (`shm_open`). Alternatively, if shared state is crucial and isolation is not, using **threads** (via `pthread_create`) would allow shared access to all memory by default.
<br><br>
Lastly, even though both processes are using the same file descriptor internally, **standard C I/O (`fprintf`) is buffered**. This means writes may not be flushed to disk in the order they are issued. To ensure predictable file output, each `fprintf` should be followed by `fflush(f)`, or the file should be opened in unbuffered mode.
</p></div><br>


## The Case of the Shared Counter (mmap edition)

You now refactor the previous program to try **actual shared memory**. You use `mmap()` to create a `counter` that both parent and child can access and modify. Here's the updated code:

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/mman.h>
#include <sys/types.h>
#include <sys/wait.h>

int main() {
    int *counter = mmap(NULL, sizeof(int),
                        PROT_READ | PROT_WRITE,
                        MAP_SHARED | MAP_ANONYMOUS,
                        -1, 0);
    if (counter == MAP_FAILED) return 1;

    *counter = 0;

    pid_t pid = fork();
    if (pid < 0) return 1;

    for (int i = 0; i < 5; i++) {
        (*counter)++;
        printf("PID %d: counter = %d\n", getpid(), *counter);
        usleep(100000); // simulate work
    }

    if (pid > 0) wait(NULL);
    munmap(counter, sizeof(int));
    return 0;
}
```

Now both processes increment the same memory, but <span class="orange-bold">new issues</span> emerge: sometimes you see the same number printed twice, or values skip.


**Answer the following questions:**
1. Why does `mmap()` work here while a global variable did not?
2. Why do some counter values appear duplicated or skipped?
3. Propose a fix to ensure both processes safely update the counter without data races.
4. Could this still fail if printing were done to a shared file? Why?

{:.highlight}
> **Hints:**
> * `mmap(... MAP_SHARED ...)` creates shared memory visible across `fork()`.
> * The race occurs because `(*counter)++` is **not atomic**, it's a read-modify-write.
> * Use `fcntl` locks, `pthread_mutex_t` in shared memory, or Linux `futex`es for synchronization.
> * `printf()` is buffered per process and may reorder output unpredictably.



<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
Unlike regular variables, the memory returned by `mmap()` with `MAP_SHARED` is visible to both parent and child after `fork()`. That’s why both can see and update the same `counter` in this example. However, because both processes execute `(*counter)++` concurrently, they may read the same value before either has a chance to write back, resulting in lost updates (i.e., one increment overwrites another).
<br><br>
This is a classic **race condition**. To fix it, you need synchronization. For instance, you could use a **POSIX semaphore** or place a `pthread_mutex_t` in the shared memory region (initialized with `PTHREAD_PROCESS_SHARED`). This would ensure only one process updates the counter at a time.
<br><br>
Even if the counter is correct, the printed output can still be disordered because `printf()` is **line-buffered** and local to each process. So flushing or synchronizing output matters too.

</p></div><br>



## The Case of the Shared Counter (pipe edition)


This time, you try a **pipe-based approach** to solve the problem differently. Instead of sharing memory, the child sends its `counter` values to the parent via a pipe. Here's the revised code:

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main() {
    int pipefd[2];
    if (pipe(pipefd) == -1) return 1;

    pid_t pid = fork();
    if (pid < 0) return 1;

    if (pid == 0) {
        // Child: close read end, write counter values
        close(pipefd[0]);
        for (int i = 1; i <= 5; i++) {
            dprintf(pipefd[1], "Child PID %d: counter = %d\n", getpid(), i);
            usleep(100000);
        }
        close(pipefd[1]);
    } else {
        // Parent: close write end, read from child
        close(pipefd[1]);
        char buffer[128];
        while (read(pipefd[0], buffer, sizeof(buffer)-1) > 0) {
            buffer[sizeof(buffer)-1] = '\0'; // ensure null-terminated
            printf("Parent received: %s", buffer);
        }
        close(pipefd[0]);
        wait(NULL);
    }

    return 0;
}
```

{:.note}
In this version, only the child counts, and the parent prints what it receives.


**Answer the following questions:**
1. What problem does this approach avoid, compared to shared memory?
2. Why is this version free from data races on the counter value?
3. Suppose you wanted both parent and child to send messages, how would you modify the pipe(s)?
4. What are some downsides of this design compared to the `mmap()` approach?


{:.highlight}
> **Hints:**
> * Pipes are **unidirectional**, they serialize output through the kernel buffer.
> * Only the child modifies the counter, so no race occurs.
> * To make communication two-way, you need a **second pipe** or use `socketpair()`.
> * You lose **shared state**, so the parent cannot directly modify the counter.

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
Using a pipe avoids the need for shared memory or synchronization primitives. Since only the child modifies the `counter`, there's no risk of conflicting updates. The parent only reads what the child writes, and the pipe ensures message order and atomicity (if message sizes are under `PIPE_BUF`).
<br><br>
However, communication is **one-way** unless a second pipe is added. Unlike `mmap()`, there's no shared memory, just message passing. This limits how much state can be shared and modified across processes. That said, pipes are simpler to reason about and safer when you only need to send occasional values or logs, especially if you want clear separation between process responsibilities.

</p></div><br>

### Further Notes 

IPC is essential when separate processes need to coordinate or exchange data. Two primary IPC models are **shared memory** and **message passing**. In the shared memory model, multiple processes map a common memory region (e.g., using `mmap()`), allowing direct access to shared variables. This model is efficient but requires careful synchronization (e.g., mutexes or semaphores) to avoid race conditions.

In contrast, the message passing model enforces stricter isolation between processes. Communication occurs through explicit system calls such as `write()` and `read()`, and the kernel acts as an <span class="orange-bold">intermediary</span>. **Pipes** are a classic example of message passing IPC in Unix-like systems. A pipe provides a <span class="orange-bold">unidirectional</span> channel through which one process can send data to another. This helps serialize communication and reduce the risk of data races, especially when only one process writes and the other reads.

The code in this problem demonstrates a simple parent-child interaction using a pipe. Rather than sharing a `counter` variable directly, the child sends its updates to the parent through a pipe. The parent reads and prints the received messages, illustrating a safe and structured approach to communication without shared state.

This problem contrasts with shared memory designs and raises useful questions about synchronization, data integrity, and bidirectional communication in IPC.



## Ghosts of the Dead


A student writes a C program that spawns several child processes using fork(). Each child completes its task and exits, but the student notices that their process IDs remain in the system with a "Z" status, indicating zombie processes. As more children are created, the number of zombies grows **steadily**. Although the children have clearly terminated, they still *linger in the process table*, leading the student to suspect a resource leak or a flaw in their process management.


**Answer the following questions:**
1. What is a **zombie process**, and how is it different from a running or stopped process?
2. Why do zombie processes appear when the parent does **not call `wait()`**?
3. How can the parent properly clean up after its children to avoid this problem?


{:.highlight}
> **Hints:**
> * A zombie process has **terminated**, but its **exit status has not been collected**.
> * The OS retains minimal information about the process (PID, exit code) for the parent to retrieve.
> * If the parent doesn’t call `wait()`, the process table entry cannot be freed.
> * Use `wait()` or `waitpid()` to **reap** child processes.
> * If the parent dies first, `init` or `systemd` adopts the child and reaps it.



<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
A zombie process is one that has completed execution (i.e., exited), but still has an entry in the process table because its parent has not yet read its exit status. When a process terminates, the operating system retains a small amount of information, such as its PID and exit code, so that the parent process can retrieve it using `wait()` or `waitpid()`.
<br><br>
If the parent does not perform this cleanup, the child remains in a "zombie" state. These zombies do not consume CPU or memory in the usual sense, but they occupy slots in the limited process table. Over time, an accumulation of zombies can exhaust system resources and prevent new processes from being created.
<br><br>
To avoid this, the parent must call `wait()` after `fork()` to reap each child. Alternatively, the parent can set a signal handler for `SIGCHLD` to automatically handle terminated children, or explicitly call `waitpid()` in a loop. If the parent exits before the child, the `init` process (PID 1) adopts the orphaned child and takes care of reaping it.

</p></div><br>


## Race Against the Shell



While running a long-running simulation in the terminal, a student tries two different approaches: launching it normally with `./simulator` and then again using `./simulator &` to run it in the background. They observe that the foreground job **blocks** the shell, preventing them from typing further commands until it finishes, while the backgrounded version immediately returns control to the prompt.

Curious, the student begins experimenting with multiple background jobs and notices differences in responsiveness, I/O behavior, and CPU usage. In some cases, background jobs seem to be throttled or behave inconsistently. This leads the student to question how the shell manages job execution, whether the OS treats foreground and background processes differently, and why performance varies depending on how the job was launched.


**Answer the following questions:**
1. What’s the key difference between **foreground** and **background** processes in terms of shell behavior?
2. Who controls scheduling: the shell, or the OS?
3. Why might background jobs behave differently in terms of **I/O or CPU usage**?

{:.highlight}
> **Hints:**
> * A foreground job is directly connected to the shell’s **standard input/output**.
> * Background jobs still run but cannot **interact with the terminal** by default.
> * The shell regains control immediately after launching a background job.
> * The OS handles scheduling, but shells **manage job control signals** (e.g. `SIGTSTP`, `SIGCONT`).
> * CPU usage can depend on I/O blocking or whether the process yields control.

### Job Control

In Unix-like systems, **job control signals** allow the shell and users to manage how processes run within a terminal session. These signals enable suspending (`SIGTSTP`), resuming (`SIGCONT`), or terminating (`SIGINT`) jobs. For example, when a user presses `Ctrl+Z`, the foreground process is sent `SIGTSTP`, which causes it to pause (or "stop") execution. The user can then run `bg` to resume the process in the background, or `fg` to bring it back to the foreground. Both commands send `SIGCONT`, which tells the stopped process to continue running.

The shell plays a key role in **managing these signals**. It tracks process groups and forwards appropriate signals to foreground or background jobs. It also uses system calls like `waitpid()` to detect when jobs stop or exit, enabling the shell to update job status and provide interactive control. This system enables a single terminal session to juggle multiple processes efficiently and safely.

### Common UNIX Job Control Signals


| **Signal** | **Name**             | **Description**                                    | **Typically Sent By**      |
| ---------- | -------------------- | -------------------------------------------------- | -------------------------- |
| `SIGINT`   | Interrupt            | Interrupts the process (e.g., `Ctrl+C`)            | Terminal / shell           |
| `SIGTSTP`  | Terminal Stop        | Suspends the process (e.g., `Ctrl+Z`)              | Terminal / shell           |
| `SIGCONT`  | Continue             | Resumes a stopped process                          | Shell (`fg`, `bg`) or user |
| `SIGHUP`   | Hangup               | Sent when terminal closes; can terminate or reload | Kernel / shell             |
| `SIGSTOP`  | Stop (non-catchable) | Forcefully stops a process (cannot be ignored)     | User via `kill -STOP PID`  |


Below is a short example showing how a program might **handle `SIGTSTP`** (triggered by `Ctrl+Z`) using `signal()` or `sigaction()`:

```c
#include <signal.h>
#include <stdio.h>
#include <unistd.h>

void handle_tstp(int sig) {
    printf("Caught SIGTSTP (Ctrl+Z), ignoring suspend...\n");
}

int main() {
    // Set up custom handler for SIGTSTP
    signal(SIGTSTP, handle_tstp);

    while (1) {
        printf("Running... press Ctrl+Z\n");
        sleep(1);
    }

    return 0;
}
```

{:.important}
In a real terminal, `SIGTSTP` normally stops the process regardless of handlers. To truly override this behavior, you need terminal control and job control logic, which is typically managed by the shell. This example only **prints a message** when the signal is caught (and may not work depending on terminal settings).


### Task 

**Answer the following questions:**
1. What is the difference between a foreground and background job in the shell?
2. Who is responsible for scheduling background jobs: the shell or the operating system?
3. Why do background jobs sometimes get stopped when they try to read from the terminal?
4. What role does the shell play in job control, especially in managing job-related signals?

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
Foreground and background jobs differ mainly in how the shell manages them. A **foreground process** is given access to the terminal’s input and output. The shell waits for it to finish before accepting new commands. A **background process**, invoked with `&`, runs concurrently, and the shell <span class="orange-bold">immediately</span> returns control to the user, allowing them to type further commands.
<br><br>
While both types are scheduled by the operating system’s scheduler (not the shell), background processes are often **disallowed from reading input** from the terminal. If they attempt to, they may be stopped by the shell. This distinction can affect how responsive or CPU-heavy the process appears, especially if one blocks on I/O while the other does not.
<br><br>
The shell does not schedule CPU time, that's the kernel’s job. However, the shell **does manage job control**, including signals like `SIGINT`, `SIGTSTP`, and `SIGCONT`, which affect whether jobs run or pause based on user actions like `Ctrl+Z` or `fg`. As a result, background jobs may appear to behave differently depending on how they're managed, even though from the OS perspective, they’re all just processes.
</p></div><br>



## MS-DOS vs FreeBSD: A Process Story

Your teammate Alice is exploring how different operating systems handle program execution for a class project. While reading about legacy systems, she notices that MS-DOS replaces the running program entirely when launching a new one, with no way to resume or manage the previous process. Meanwhile, FreeBSD and other Unix-like systems follow a different approach: they first create a duplicate process with `fork()`, and then selectively replace it using `exec()`.

This design difference puzzles her. She wonders why modern systems bother with two steps instead of directly loading the new program, and whether the extra complexity actually serves a purpose. To make sense of it, Alice starts comparing how both systems handle multitasking, error handling, and control over the execution flow.


**Answer the following questions:**

1. What happens when MS-DOS loads a new `.exe` file? What is lost?
2. In FreeBSD, what are the distinct roles of `fork()` and `exec()` during program execution?
3. Compare the **flexibility** and **robustness** of these two approaches in multi-tasking environments.

{:.highlight}
> **Hints:**
> * MS-DOS is **single-tasking**: it replaces the current program in memory.
> * FreeBSD allows **multi-tasking** using process creation (`fork`) and replacement (`exec`).
> * `fork()` creates a **child process**, which can then choose to run a new program using `exec()`.
> * The separation allows **pre-processing**, such as setting up file descriptors or memory mappings before running the new code.
> * MS-DOS programs cannot spawn or manage other programs, control returns only when the new program exits.


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
In MS-DOS, loading a new program means the currently running program is completely overwritten in memory. There’s no concept of multi-tasking or child processes, once you load another `.exe`, the previous program is gone, and control only returns after the new program finishes. This model is simple but rigid, and unsuitable for systems that need concurrency or background processes.
<br><br>
In contrast, FreeBSD (like other Unix-based systems) separates the steps of **creating a new process** (`fork()`) and **replacing its code** (`exec()`). This allows the parent to retain control, manage multiple children, and customize each child before it begins execution, such as redirecting output or setting environment variables. This model supports flexible and robust multi-tasking, inter-process communication, and better error handling.
<br><br>
While MS-DOS’s model might be lighter-weight, FreeBSD’s approach enables **complex and controlled process management**, making it more suitable for modern multi-user, networked, and interactive systems.
</p></div><br>



## State of Confusion


While debugging a custom scheduler, you notice that some processes remain stuck in the "Ready" or "Waiting" state far **longer** than expected. <span class="orange-bold">These processes aren’t progressing</span>, some never seem to get scheduled, while others wait indefinitely for events that never occur.

This unexpected behavior raises concerns about possible bugs in the scheduler logic or the way certain blocking operations are handled. To diagnose the issue, it becomes necessary to revisit the expected transitions between process states and explore how processes might become trapped due to scheduling flaws, missed signals, or incorrect assumptions about resource availability.

**Answer the following questions:**
1. Describe the purpose of the **Ready**, **Running**, and **Waiting** states in a typical process state model.
2. What are valid transitions between these states? What causes them?
3. Why might a process remain in **Ready** or **Waiting** indefinitely? Give at least one explanation for each case.

{:.highlight}
> **Hints:**
> * A process in **Ready** is waiting for CPU time, it is not blocked.
> * A process in **Waiting** is blocked, it’s waiting for some event or I/O to complete.
> * The scheduler picks from the Ready queue, not the Waiting list.
> * A bug in I/O handling or signaling could cause a process to **never exit Waiting**.
> * A process may starve in the **Ready queue** if the scheduler is unfair or buggy.

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
In a typical OS process state model, a process in the **Ready** state is prepared to run but is waiting for CPU time. A **Running** process is currently being executed by the CPU. A **Waiting** (or Blocked) process is paused until some external event completes, such as I/O or receiving a signal.
<br><br>
<p><strong>Valid transitions include:</strong></p>
<ul>
  <li><strong>Ready → Running</strong>: when the scheduler picks the process.</li>
  <li><strong>Running → Waiting</strong>: when the process performs a blocking operation (e.g., <code>read()</code> on an empty pipe).</li>
  <li><strong>Waiting → Ready</strong>: when the awaited event occurs (e.g., data arrives).</li>
  <li><strong>Running → Ready</strong>: due to preemption (e.g., time slice expired).</li>
</ul>
<br><br>
A process can get stuck in **Ready** if the scheduler never selects it, possibly due to starvation (e.g., priority inversion or a faulty round-robin implementation). A process can remain in **Waiting** indefinitely if the event it’s waiting for never happens, for example, if a signal or I/O completion is never delivered due to a driver bug, missed wake-up, or incorrect synchronization logic.
</p></div><br>



## The Phantom Thread

### Background

Certainly. Here is the revised **background section** without hyphens:

---

### Background

In multithreaded Java programs, memory visibility refers to whether one thread's changes to shared variables become observable by other threads. This issue is governed by the Java Memory Model (JMM), which specifies the rules for how and when reads and writes to variables are propagated between threads and main memory.

By default, the JVM and modern processors may cache variables in registers or processor-local caches. This can lead to a situation where one thread updates a variable, but another thread continues reading a stale value. Such behavior is <span class="orange-bold">not</span> a compiler bug. It is legal under the JMM unless a happens before relationship is explicitly established.

Two common ways to enforce visibility are:

* Declaring a variable as `volatile`, which ensures that reads and writes go directly to main memory rather than being cached
* Using `synchronized` blocks or methods, which enforce visibility and mutual exclusion by acquiring and releasing locks

Failing to establish proper memory visibility leads to *nondeterministic* bugs where a thread may never observe an update even though it logically should. These bugs are subtle and often arise in busy wait loops, flags, and concurrent data structures. Understanding and applying the Java Memory Model is essential for writing correct and predictable concurrent programs.


### Scenario
You're reviewing a teammate’s Java program that uses two threads: one sets a shared boolean flag `done`, while the other loops until the flag becomes `true`. Despite `done = true` being executed in the setter thread, the loop sometimes **never exits**, it gets stuck as if the flag was never set.

Here’s a simplified version of the code:

```java
public class FlagExample {
    static boolean done = false;

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            while (!done) {
                // Busy-wait
            }
            System.out.println("Done!");
        });

        Thread t2 = new Thread(() -> {
            done = true;
        });

        t1.start();
        t2.start();
    }
}
```

Sometimes, `System.out.println("Done!")` is printed. Sometimes, it isn't, even though `done = true` is always executed.


**Answer the following questions:**
1. Why does the loop sometimes never terminate even though the flag is set?
2. How does the Java Memory Model explain this behavior?
3. What is the role of the `volatile` keyword in fixing this bug?
4. Would using `synchronized` blocks also solve the problem? Why or why not?

{:.highlight}
> **Hints:**
> * Modern CPUs and JVMs may cache variables locally in registers or CPU cache.
> * Without proper memory visibility guarantees, threads may not see updates made by others.
> * `volatile` ensures that changes to a variable are **visible across threads**.
> * Busy-wait loops on non-volatile variables are prone to optimization by the compiler or JVM.

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>

<p>The issue here is a classic example of a <strong>memory visibility bug</strong>. In Java, threads can maintain <em>local copies</em> of shared variables (e.g., in CPU caches or registers). Without proper synchronization, there's no guarantee that one thread’s update (<code>done = true</code>) is ever visible to the other thread. As a result, <code>t1</code> may keep reading an outdated <code>false</code> value indefinitely, even though <code>t2</code> set it to <code>true</code>.</p>

<p>The <strong>Java Memory Model (JMM)</strong> defines rules for how and when changes to memory made by one thread become visible to others. By default, without synchronization or <code>volatile</code>, updates can be <strong>delayed</strong> or <strong>reordered</strong>.</p>

<p>Declaring <code>done</code> as <code>volatile</code>:</p>

<pre><code>static volatile boolean done = false;
</code></pre>

<p>fixes the problem. The <code>volatile</code> keyword enforces two guarantees:</p>

<ol>
  <li><strong>Visibility</strong>: Any write to a <code>volatile</code> variable by one thread is immediately visible to others.</li>
  <li><strong>Ordering</strong>: It prevents certain types of instruction reordering, ensuring <code>done = true</code> happens before <code>t1</code> can exit the loop.</li>
</ol>

<p>Alternatively, enclosing the read and write in <strong>synchronized blocks</strong> would also work, because entering and exiting a synchronized block <em>flushes thread-local caches</em> and ensures memory consistency.</p>

</p></div><br>




## The Forgotten Join

You're helping your teammate to debug a multithreaded C program using POSIX threads. The goal is to create several threads that each perform some logging and then exit. The thread creation appears to work, but sometimes not all log messages are printed, and the program exits unexpectedly early.

Here's a simplified version of his code:

```c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

void *log_function(void *arg) {
    int thread_num = *(int *)arg;
    printf("Thread %d is running\n", thread_num);
    sleep(1); // simulate work
    printf("Thread %d has finished\n", thread_num);
    return NULL;
}

int main() {
    pthread_t threads[3];
    int ids[3] = {1, 2, 3};

    for (int i = 0; i < 3; i++) {
        pthread_create(&threads[i], NULL, log_function, &ids[i]);
    }

    // forgot to join threads

    return 0;
}
```

Sometimes the program prints all thread messages, sometimes it doesn't. He is unsure whether this is a bug or expected behavior.


**Answer the following questions:**
1. What happens to a thread created with `pthread_create` if the main program finishes before it does?
2. What is the purpose of `pthread_join`, and how does it affect program correctness?
3. Could there be memory-related side effects from omitting `pthread_join` in this context?
4. Rewrite the loop to properly wait for all threads to complete.


{:.highlight}
> **Hints:**
> * `pthread_join(thread_id, NULL)` blocks the calling thread until the target thread exits.
> * Without joining, the main thread can terminate the entire process prematurely.
> * Threads that have not been joined become **detached** and may leak memory or leave output unfinished.
> * Always match `pthread_create` with `pthread_join` unless you explicitly detach the thread.


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>When a thread is created using <code>pthread_create</code>, it begins executing concurrently with the main thread. However, if the <code>main()</code> function finishes before these threads complete, the process may terminate entirely, killing all still-running threads. This results in lost output or partially executed logic, as observed in the inconsistent logging behavior.</p>

<p>The function <code>pthread_join</code> ensures that the main thread <em>waits</em> for the specified thread to finish before proceeding. Without this, the main thread doesn’t know or care whether the child threads have completed. If it returns from <code>main()</code> first, the entire process ends, and any threads still running are terminated prematurely.</p>

<p>In the provided code, the absence of <code>pthread_join</code> leads to a race condition between thread completion and program exit. To fix this, each thread should be joined explicitly:</p>

<pre><code>for (int i = 0; i < 3; i++) {
    pthread_join(threads[i], NULL);
}
</code></pre>

<p>Not joining threads can also lead to <strong>resource leaks</strong>, since the operating system may still reserve memory for thread metadata until it's reclaimed. Using <code>pthread_detach</code> is an alternative when joining is not required, but in most cases, joining ensures predictable and correct multithreaded behavior.</p>


</p></div><br>


## Stack vs Heap: The Return Trap


In a pthread-based C program, your teammate  writes a thread function that returns a pointer to a local character variable. The main thread retrieves this pointer via `pthread_join` and tries to dereference it. Sometimes it works, but sometimes it results in garbage or even a segmentation fault.

Here’s a simplified version of the code:

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

void* thread_func(void* arg) {
    char letter = 'X';         // local stack variable
    return (void*)&letter;     // return address of stack variable
}

int main() {
    pthread_t tid;
    char* result;

    pthread_create(&tid, NULL, thread_func, NULL);
    pthread_join(tid, (void**)&result);

    printf("Returned: %c\n", *result); // sometimes garbage, sometimes segfault
    return 0;
}
```

He wants to know why the behavior is inconsistent, and whether there's a better way to return values from threads.

### Background

In C programs, memory is typically managed using two primary regions: the **stack** and the **heap**. These regions serve different purposes and have distinct lifetimes and allocation behaviors.

The **stack** is used for automatic storage. When a function is called, its local variables are allocated on the stack. This memory is automatically reclaimed when the function returns. In multithreaded programs, each thread receives its own stack. Once a thread terminates, its stack memory is deallocated. Accessing this memory after the thread has exited leads to undefined behavior, such as segmentation faults or stale data. This a common source of <span class="orange-bold">dangling pointer bugs</span>.

The **heap**, by contrast, is a region of memory used for dynamic allocation. Memory allocated on the heap (e.g., via `malloc`) persists until it is explicitly freed by the program. Heap memory is shared across threads and is not tied to the lifetime of any particular function or thread. This makes it the appropriate choice for returning data from a thread after it has terminated.

### Task


**Answer the following questions:**
1. Why is returning a pointer to a stack variable from a thread unsafe?
2. What happens to the thread’s stack after it exits?
3. How does returning a heap-allocated pointer fix the problem?
4. Rewrite the thread function to correctly return a character pointer.


{:.highlight}
> **Hints:**
> * Stack memory is valid only while the thread is alive.
> * After a thread exits, its stack frame is deallocated.
> * Heap memory persists beyond the lifetime of the thread that allocated it.
> * Always ensure data returned via `pthread_join` points to valid, live memory.



<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>The problem arises because the thread returns the address of a <strong>stack-allocated</strong> variable. Once the thread function ends, its stack frame is deallocated. This means the pointer returned by <code>pthread_join</code> may refer to invalid memory, leading to undefined behavior, including garbage values or segmentation faults when dereferenced.</p>

<p>Stack memory is automatically managed and only exists for the lifetime of the thread. After the thread exits, the address previously returned points to memory that has been <em>freed or repurposed</em>. Using it after that point is a classic case of a <strong>dangling pointer</strong>.</p>

<p>To safely return data from a thread, the thread should <em>allocate memory on the heap</em>, which is valid until explicitly freed. For example:</p>

<pre><code>void* thread_func(void* arg) {
    char* result = malloc(sizeof(char));
    *result = 'X';
    return (void*)result;
}
</code></pre>

<p>Then in the main function, the returned pointer can be safely dereferenced and later <code>free()</code>d:</p>

<pre><code>char* result;
pthread_join(tid, (void**)&result);
printf("Returned: %c\n", *result);
free(result);
</code></pre>

<p>This ensures that the memory returned is valid regardless of the thread’s termination, avoiding crashes and ensuring predictable behavior.</p>


</p></div><br>



## Mapping Mayhem: One-to-One or Many-to-Many?

You're reviewing a performance-critical application that uses a large number of threads to handle concurrent work. It runs fine on your local machine, but performs surprisingly poorly on a production multicore server. CPU usage stays low despite many threads being active.

After profiling, you discover that the thread library in use implements a **many-to-one** mapping model, where all user threads are managed in user space and mapped onto a single kernel thread. Despite having multiple cores available, the threads do not run in parallel.

**Answer the following questions:**
1. What are the differences between **many-to-one**, **one-to-one**, and **many-to-many** thread mapping models?
2. Why does a many-to-one model prevent true parallelism on multicore systems?
3. What are the trade-offs between these models in terms of system calls, scalability, and overhead?
4. Which mapping model is most suitable for performance-intensive multithreaded applications on modern systems, and why?

{:.highlight}
> **Hints:**
> * Many-to-one threads are all scheduled by a user-level thread library, not the kernel.
> * Only one thread can execute at a time, even on multicore CPUs.
> * One-to-one provides true concurrency but incurs system call overhead for every thread.
> * Many-to-many offers flexibility, but is harder to implement and less common in mainstream systems.


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>The difference between the thread mapping models lies in how user-level threads are associated with kernel-level threads:</p>

<ul>
  <li><strong>Many-to-One (M:1)</strong>: All user threads are mapped onto a single kernel thread. The OS kernel is unaware of the individual threads and schedules only the single backing kernel thread.</li>
  <li><strong>One-to-One (1:1)</strong>: Each user thread maps to its own kernel thread. These are scheduled independently by the OS, allowing true parallelism.</li>
  <li><strong>Many-to-Many (M:N)</strong>: Multiple user threads are multiplexed over a smaller or equal number of kernel threads. The OS sees only the kernel threads, but the thread library manages user thread scheduling.</li>
</ul>

<p>In a many-to-one model, only one user thread can be executed at a time because the kernel sees them as a single thread. Even on a multicore machine, threads cannot run in parallel, they’re context-switched in user space. This severely limits performance in CPU-bound or highly parallel workloads.</p>

<p>One-to-one mapping allows multiple threads to run in parallel on different cores, since the kernel is aware of each thread and can schedule them independently. However, it has higher overhead: creating, managing, and context-switching threads requires system calls, making it more expensive in terms of performance and system resources.</p>

<p>Many-to-many models offer a middle ground, user threads are mapped dynamically to a limited pool of kernel threads. This model enables better scalability and customization, but is rarely used in practice due to its complexity. Most modern systems (e.g., Linux pthreads) adopt the one-to-one model.</p>

<p>For performance-intensive applications on multicore systems, the <strong>one-to-one model</strong> is typically the best choice. It allows for true concurrency and leverages the OS scheduler to balance threads across available CPUs. While it may introduce more overhead than many-to-one, the gain in parallelism far outweighs the cost in most real-world scenarios.</p>
</p></div><br>



## Fault Isolation

You're comparing two server implementations: one uses multiple **threads**, and the other uses multiple **processes**. Both handle client connections concurrently.

In the threaded version, a bug in one handler causes a buffer overflow and crashes the entire server. In the process-based version, the same bug only crashes a single client handler,  the server remains alive and continues serving other clients.

This prompts a *deeper* investigation into how threads and processes isolate faults differently, and what trade-offs come with each approach.

**Answer the following questions:**
1. Why does a crash in one thread typically bring down the entire process?
2. How do protection boundaries differ between threads and processes?
3. What are the trade-offs between using threads and processes in terms of fault isolation, performance, and resource overhead?
4. In what scenarios might processes be preferred over threads despite their higher overhead?

{:.highlight}
> **Hints:**
> * Threads share the same address space and memory.
> * A fault like a buffer overflow in one thread corrupts the memory of the whole process.
> * Processes have separate address spaces, preventing one from corrupting another.
> * Context switching between processes is more expensive but offers better isolation.


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>Threads within the same process share a single address space. This includes shared access to code, global variables, heap memory, and file descriptors. As a result, a crash in one thread,  for example, due to a buffer overflow,  can corrupt shared memory and crash the entire process. The operating system treats all threads of a process as one failure domain.</p>

<p>In contrast, processes have <strong>separate address spaces</strong>. Each process is isolated by the operating system’s memory management mechanisms. A memory violation in one process (e.g., writing outside allocated memory) will not affect other processes. Instead, the faulty process is terminated, and others continue running.</p>

<p>The trade-off is between <strong>performance</strong> and <strong>safety</strong>:</p>

<ul>
  <li><strong>Threads</strong> offer faster communication and lower overhead. They’re ideal for lightweight parallelism where tasks must share state and communicate frequently.</li>
  <li><strong>Processes</strong> offer stronger fault isolation and protection. However, they require inter-process communication (IPC) mechanisms, which are more complex and slower.</li>
</ul>

<p>In systems where robustness and fault tolerance are critical,  such as servers handling untrusted or independent clients,  using separate processes may be preferred, even with added resource costs. This model ensures that a single handler crash doesn’t compromise the entire service.</p>
</p></div><br>






## Heavyweight Messaging


You're benchmarking two parallel programs that perform the same task: pass one million messages between worker units. One version uses **multiple processes**, and the other uses **multiple threads**.

Despite having similar logic, the process-based implementation performs significantly worse. CPU usage is high, but throughput is low. After profiling, you notice that the delays mostly come from inter-process communication and frequent context switching.

You decide to dig into the architectural differences that affect performance.


**Answer the following questions:**
1. Why is switching between processes more expensive than switching between threads?
2. How do threads achieve faster communication compared to processes?
3. What happens during a process context switch that doesn't occur with thread switching?
4. How do these differences impact system design choices for parallel workloads?


{:.highlight}
> **Hints:**
> * Thread switching is lighter,  no need to flush TLB or change address space.
> * Process switching involves MMU updates, cache invalidation, and system calls.
> * Threads share memory; processes require IPC mechanisms like pipes or shared memory.
> * Thread APIs may run in user space, while process control always involves the kernel.



<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>Process switching is <span class="orange-bold">significantly more expensive</span> than thread switching due to the isolation enforced by the operating system. When switching between processes, the CPU must:</p>

<ul>
  <li>Flush or update the TLB (Translation Lookaside Buffer)</li>
  <li>Change the memory mapping (virtual address space)</li>
  <li>Save and restore a larger context (including page tables and open file descriptors)</li>
  <li>Execute a system call, involving mode switch between user and kernel space</li>
</ul>

<p>In contrast, thread switching is lighter because threads within the same process share the same memory space. Only the CPU registers, program counter, and stack pointer need to be saved and restored. No virtual memory remapping is required, and in many implementations, thread context switching can happen entirely in user space without a system call.</p>

<p>When it comes to communication, <strong>threads share memory</strong> by default, enabling direct access to shared variables or buffers. In contrast, <strong>processes must use IPC mechanisms</strong> like pipes, message queues, or shared memory segments, which require coordination and system calls,  adding latency and overhead.</p>

<p>These differences make <strong>threads ideal for high-throughput, fine-grained parallelism</strong>, while processes are better suited for fault isolation and secure compartmentalization. System designers must weigh these trade-offs when building scalable, concurrent applications.</p>
</p></div><br>




