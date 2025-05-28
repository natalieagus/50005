---
layout: default
permalink: /ps/2-os-services-design-structure
title: OS Services and Design Structure
parent: Problem Set 
nav_order:  1
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

# OS Services and Design Structure
{: .no_toc}

## The Mystery of the Missing System Program


### Background: Shell Builtin Functions vs System Programs
When we type a command in a shell, it may either trigger a **built-in function** or invoke an **external system program**.

* **Built-in Shell Functions**
  These are commands implemented *within the shell itself*. They run **without creating a new process**. Examples include:

  * `cd` – changes the current directory
  * `export` – sets environment variables
  * `echo` – prints text to the terminal
    Since they're handled internally, they're **faster** and essential for changing the shell’s own state.

* **System Programs (External Commands)**
  These are **separate executables** located in directories like `/bin` or `/usr/bin`. When called, the shell **creates a child process** to run them. Examples:

  * `ls` – lists directory contents
  * `cat` – reads file contents
  * `grep`, `python`, `gcc`, etc.
    These programs use **system calls** to request kernel services (e.g., read files, print output).

In summary, builtins are fast and internal; system programs are external and involve process creation and system calls.

### Scenario

A student implemented a minimalistic shell in C. The student notices that built-in commands like `cd` work perfectly, but external commands such as `ls` or `rm` fail mysteriously. The student <span class="orange-bold">mistakenly</span> refers to these external commands as "system calls" and is unsure why they don't behave the same way as the built-in commands.

**Answer the following questions:**
1. Explain clearly the difference between a system call and a system program, giving examples of each in the context of a shell.
2. Identify whether commands such as ls and rm are system calls or system programs. Provide reasoning to support your identification.
3. Describe clearly why built-in shell commands (like `cd`) behave differently from external system programs (like `ls`, `rm`) within a custom shell.

{:.highlight}
> **Hints**:
> * Recall what you've learned about **how user programs access OS services**. Are system calls directly executable commands from the terminal, or are they something lower-level?
> * Think about where commands like `ls` or `rm` are located and how your shell executes them. Do these commands directly interact with kernel space, or do they rely on something else? Refer to Command Line Lab for clues.
> * Consider how built-in commands are executed within the shell compared to how external programs are launched and executed. What is fundamentally different in their implementation within a shell?
 
<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
System calls are <span class="orange-bold">programming interfaces</span> provided by the operating system kernel that allow user-level programs to request low-level OS services, such as file I/O operations, memory allocation, or process management. They represent controlled entry points into kernel space, typically invoked indirectly via programming APIs (e.g., `write()`, `open()` in POSIX). In contrast, system programs are user-level utility programs provided by the operating system to perform common tasks, such as managing files or processes, and running in user mode. Examples of system calls include `open()`, `read()`, and `write()`, while examples of system programs are terminal commands like `ls`, `rm`, and `mkdir`.
<br><br>
The commands `ls` and `rm` are system programs, <span class="orange-bold">not</span> system calls. They are independent executable programs residing typically in directories such as `/bin` or `/usr/bin`, executed by a shell upon request. These programs internally utilize system calls, such as `open()`, `read()`, and `write()`, to perform their tasks. However, they themselves are not directly executable system calls.
<br><br>
Built-in commands (such as `cd`) behave differently from external system programs (`ls`, `rm`) because built-in commands are executed directly within the shell's own process. **The shell itself contains the code for built-in commands** (you will experience this in Programming Assignment 1), and these do <span class="orange-bold">not</span> require starting new processes or loading external executable files. On the other hand, external commands like `ls` or `rm` exist as separate executable programs stored in specific system directories. When the shell executes external commands, it must locate, load, and run these executable files, typically through mechanisms like `fork()` and `exec()`. Hence, the difference lies primarily in how they are executed: built-ins run internally, whereas external system programs require external execution procedures.
</p></div><br>

## The Case of the Unresponsive UI

{:.note}
This question is similar to [another](#the-case-of-the-hanging-logger) question below, but is fundamentally easier to start with. 

You built a GUI application that downloads files from a remote server. After initiating a download operation, users noticed that the application's user interface (UI) becomes completely unresponsive. Clicking buttons or interacting with the UI does nothing until the file download completes, at which point the UI suddenly becomes responsive again.

**Answer the following questions:**
1. **Explain** clearly what it means for a system call to be <span class="orange-bold">blocking</span> and provide an example related to this GUI scenario.
2. **Describe** the impact of blocking system calls on the responsiveness of GUI-based applications.
3. Briefly **describe** what a non-blocking or asynchronous system call is, and how using this approach could improve the application's responsiveness.
4. Suggest a **practical** approach to fix this issue and clearly explain why your solution would resolve the problem.

{:.highlight}
> **Hints**: 
> * Blocking vs non-blocking: *what's the key difference?*
> * Consider what happens when the UI thread is waiting for an operation to finish.
> * Is there a way to let the download happen "in the background"?
> * Think about threads or asynchronous mechanisms you might know. We will learn more about this in the weeks to come. 

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
A **blocking** system call is one that causes the calling process or thread to **pause** execution until the requested operation fully completes. In the given scenario, an example is the *file download operation*. When the GUI program initiates a blocking network or file-read operation, the application's main thread is halted and waits for the operation to finish before continuing execution.
<br><br>
Blocking system calls <span class="orange-bold">severely impact GUI responsiveness</span> because GUI frameworks typically handle UI interactions and rendering in a single main thread. If a blocking system call pauses this main UI thread, the application becomes **unresponsive**, unable to process user events (like button clicks, mouse movements) or repaint the UI until the blocking call completes.
<br><br>
A non-blocking or <span class="orange-bold">asynchronous</span> system call initiates the requested operation but *immediately returns control* to the calling thread or process, allowing other tasks to continue concurrently. The result of the operation can be retrieved later once it completes, usually through **callbacks**, **event-driven programming**, or **polling** mechanisms. By adopting non-blocking or asynchronous calls, the GUI thread remains responsive, allowing interaction and continuous UI updates even while long-running tasks execute in the background.
<br><br>
A practical solution to this problem is to **implement file downloads in a separate worker thread** or use **asynchronous** I/O mechanisms. By moving the potentially blocking network or disk I/O operations off the main GUI thread, the UI thread remains responsive to user input and can refresh the display. This design resolves the issue, as the main thread no longer pauses to wait for completion of the download operation; instead, the download proceeds in parallel, updating the GUI only once complete or incrementally, using callbacks or thread-safe communication methods. 
<br><br>
We will learn more about this (multithreading and synchronization) in the weeks to come. 
</p></div><br>


## The Hidden Cost of Layered OS Structure

### Background: Layered OS Architecture
A layered OS architecture organizes the operating system into a hierarchy of layers, each built upon the one below. This structure promotes modularity and simplifies debugging, though it may introduce performance overhead if strictly enforced. Real-world systems often adopt a loosely layered or hybrid approach. They have the following key points: 

* Each layer only interacts with adjacent layers (above and below).
* The lowest layer handles hardware; the top layer interacts with user applications.
* Early example: THE OS – 6 strict layers from hardware to user interface.
* Modern example: Windows NT – HAL → Kernel → Executive → Subsystems → Applications.
* Linux shows layered traits: Hardware → Kernel → System Libraries → User Apps.

### Scenario
A team of students decides to **reorganize** their operating system project into a strictly layered structure. They reason that this *will simplify debugging* and *improve modularity*, as each layer clearly defines its responsibilities and only communicates directly with the layer immediately below it. However, after restructuring their OS, they observe a <span class="orange-bold">noticeable drop</span> in overall system **performance**. Tasks that were previously fast now take significantly longer to execute.

**Answer the following questions:**
1. **Explain** the main concept behind the layered approach to operating system design.
2. **Describe** clearly one significant advantage of organizing an OS using a layered structure.
3. **Identify** and **explain** why implementing a strictly layered OS structure might negatively affect system performance.
4. **Suggest** a way to balance the advantages of a layered approach while mitigating its performance drawbacks.

{:.highlight}
> **Hints**:
> * Think about how layers communicate and rely on each other.
> * Consider what happens to data or calls as they travel through multiple layers.
> * Is there overhead involved with layering, and where might it come from?
> * Can a hybrid approach provide a better compromise?

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
The **layered** approach to operating system design involves breaking down the OS into clearly defined, hierarchical layers. Each layer is **built on top** of the layer immediately below it, providing services only to the layer above it. The lowest layer interacts **directly** with hardware, while the **highest** layer provides an interface to the users or application programs. Each layer has a **specific**, **isolated** responsibility, simplifying implementation and debugging.
<br><br>
One significant advantage of the layered approach is **improved modularity**. Because each layer only relies on the services and interfaces provided by the layer directly below, it simplifies debugging and error handling. When errors occur, developers can systematically test each layer independently, isolating faults and ensuring correctness before moving to higher layers.
<br><br>
A strictly layered OS structure can **negatively** affect system performance due to additional overhead introduced by multiple layer transitions. Every time a higher-level layer requests a service provided by a lower layer, the request must pass **sequentially** through intermediate layers. Each intermediate layer may process, validate, or modify data, adding latency. This overhead accumulates significantly when operations require frequent communication across multiple layers, causing noticeable degradation in performance compared to less strictly layered or monolithic designs.
<br><br>
A practical approach to balance the advantages of layered design with performance considerations is to adopt a <span class="orange-bold">hybrid</span> OS structure. A hybrid OS leverages layering selectively, combining layers only where beneficial for modularity and simplicity, while retaining monolithic-like structures in performance-critical areas. For example, critical kernel services like **process scheduling**, **interrupt handling**, and **memory management** can remain closely integrated (part of the microkernel) to minimize overhead, while less performance-sensitive functionalities can still benefit from the modularity provided by layering. This approach achieves a balance between maintainability, modularity, and efficiency.
</p></div><br>

## The Layered Linux Dilemma

Imagine the developers of Linux decided to restructure their operating system kernel into a strictly layered approach. They reorganize the kernel into multiple clearly-defined layers: **hardware** interaction at the bottom (layer 0), basic resource management above that, followed by advanced **scheduling**, memory management, and finally **system-call interfaces** at higher layers. The motivation was to improve modularity and ease of debugging, inspired by earlier operating systems like **[MULTICS](https://en.wikipedia.org/wiki/Multics)**.

{:.note}
Initially, this structure allowed kernel developers to debug each layer separately, significantly reducing development complexity. However, users soon began complaining about slower performance, especially for operations that previously required direct interactions with hardware (such as disk I/O and interrupt handling).

**Answer the following questions:**
1. Using the given scenario, **explain** how a layered kernel design helps developers simplify debugging and maintenance.
2. Based on the scenario, clearly **identify** and **explain** the main reason why users experienced slower performance.
3. **Discuss** a specific operation (e.g., handling disk I/O or an interrupt) and explain how its execution would differ in a strictly layered kernel compared to a monolithic kernel, focusing on performance.
4. **Suggest** a practical structural compromise or alternative approach that Linux developers could use to maintain some benefits of layering while improving performance.

{:.highlight}
> **Hints**:
> * How does isolating responsibilities simplify error tracking?
> * Consider how many steps it takes to reach hardware in a layered system.
> * Think concretely about I/O requests or interrupts: what must happen in each kernel architecture?
> * Could selectively collapsing some layers or using a hybrid model help?

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
A layered kernel design significantly **simplifies** debugging and maintenance because each layer handles a distinct, isolated responsibility and communicates directly only with adjacent layers. Developers can test each layer **independently**, ensuring it works correctly before proceeding to the next higher layer. When errors or bugs arise, developers can quickly narrow down the potential sources of problems to a single layer, greatly reducing complexity and improving efficiency in debugging.
<br><br>
Users experienced slower performance primarily due to increased overhead from multiple layer transitions. In a strictly layered structure, every user request, system call, or hardware interrupt must sequentially traverse several layers before reaching its destination. Each layer introduces processing delays and extra data handling, cumulatively resulting in increased latency compared to monolithic or less strictly layered designs.
<br><br>
<span class="orange-bold">Consider disk I/O handling</span>. In a strictly layered kernel, a user's disk-read request moves from the system-call interface (topmost layer), downward sequentially through memory management, advanced scheduling, basic resource management, and finally to hardware interaction at the bottom layer. Each layer potentially **inspects**, **modifies**, or **verifies** the request, adding latency. Conversely, in a monolithic kernel, the same disk-read request directly invokes kernel functions that interact closely with hardware drivers, resulting in fewer transitions and significantly faster execution.
<br><br>
Linux developers could adopt a **hybrid** or **selectively layered approach** to address this issue. Critical performance-sensitive components: interrupt handling, disk I/O, and basic memory management, could remain tightly integrated in a monolithic structure. Meanwhile, *less* performance-critical components (such as higher-level system-call interfaces or logging services) could be structured into clearly-defined layers. This *practical compromise* maintains most modularity and debugging benefits while significantly reducing performance overhead, combining advantages from both layered and monolithic designs.
</p></div><br>

## The Policy-Mechanism Paradox

### Background: Policy vs Mechanism

In operating systems, **mechanism** refers to the low-level implementation that enables certain operations, while **policy** determines the strategy or rules that govern how those operations are used. 

{:.highlight}
This separation allows flexibility: the OS provides general tools (mechanisms), and the system or user defines how to use them (policies).

For example:
1. **In CPU scheduling**, the **mechanism** is the context switch (the ability to save and restore process states). The **policy** is the scheduling algorithm, such as round-robin, shortest job first, or multilevel feedback queues, which decides which process to run next.
2. **In memory management**, the **mechanism** includes paging and virtual memory mapping. The **policy** determines which page to evict on a page fault. For instance, using Least Recently Used (LRU) or First-In-First-Out (FIFO).
3. **In file systems**, the **mechanism** is the ability to set permissions (e.g., read, write, execute). The **policy** decides who gets what access. For example, based on user roles or security levels.


By separating mechanisms from policies, OSes like Linux and Windows allow different strategies to be applied without changing the core implementation. This makes the system more adaptable to diverse use cases.

### Scenario
You are reviewing a simple memory allocator written by another student. The original design aims to separate policy (how to choose a memory block) from mechanism (how to perform allocation). The student's code is shown below:


```c 
// Mechanism: marks a block as used
void allocate(Block* block) {
    block->used = 1;
    // update system records
}

// Policy: finds a suitable block
Block* first_fit(Block blocks[], int n) {
    for (int i = 0; i < n; i++) {
        if (!blocks[i].used) return &blocks[i];
    }
    return NULL;
}

Block* best_fit(Block blocks[], int n) {
    Block* best = NULL;
    for (int i = 0; i < n; i++) {
        if (!blocks[i].used && (best == NULL || blocks[i].size < best->size)) {
            best = &blocks[i];
        }
    }
    return best;
}

// Main allocator: delegates policy
void malloc_sim(Block blocks[], int n, Block* (*policy)(Block[], int)) {
    Block* selected = policy(blocks, n);
    if (selected != NULL) {
        allocate(selected);
    }
}

```

This design lets the user *switch memory allocation strategies* by changing the policy function passed into `malloc_sim`.

**Answer the following questions:**
1. **Explain** in your own words what part of the code represents the mechanism, and what part represents the policy.
2. **Describe** how this design demonstrates good separation between policy and mechanism.
3. Suppose the developer had **hardcoded** the best-fit logic directly inside `malloc_sim`. What <span class="orange-bold">negative</span> consequence would that have on future flexibility or maintainability?
4. Suggest **one** specific advantage of this approach for experimentation or system tuning in real-world OS memory allocation.

{:.highlight}
> **Hints**:
> * Mechanism: performs the actual effect or system-level action.
> * Policy: decides which block to act on.
> * Think about what happens if you want to try a new policy.
> * What does this modularity allow system developers to do?

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
In this code, the function `allocate(`) represents the **mechanism**. It performs the actual memory allocation by marking a block as used and updating relevant system state. The functions `first_fit()` and `best_fit()` are **policies**; they define different *strategies* for selecting which memory block to allocate but do not modify system state themselves.
<br><br>
This design demonstrates good separation between policy and mechanism because the decision-making logic (**policy**) is completely decoupled from the execution logic (**mechanism**). The `malloc_sim()` function <span class="orange-bold">doesn’t care</span> how the block was chosen. It only invokes the chosen policy to get a block, then delegates allocation to a fixed mechanism. This modularity improves clarity and prevents accidental entanglement of concerns.
<br><br>
If the developer had hardcoded the best-fit logic directly into `malloc_sim()`, any future attempt to use a different strategy (such as first-fit or worst-fit) would require modifying the core allocator itself. This tightly couples the decision logic with the core mechanism, reducing flexibility and increasing the risk of introducing bugs during changes. It also makes testing and experimentation with different policies much harder.
<br><br>
This approach is highly beneficial for experimentation and tuning, especially in real-world OS memory allocation, where the best allocation strategy may vary by workload or system configuration. Developers or researchers can easily **swap** or **test** new allocation policies by providing alternative functions, without touching or risking the integrity of the core allocator mechanism. This allows for rapid prototyping and cleaner optimization workflows.
</p></div><br>

## The Flexible Scheduler

### Background: Priority Based Scheduling

**Priority-based scheduling** is a CPU scheduling method where each process is assigned a priority value, and the CPU is always allocated to the process with the highest priority. Priorities can be assigned **statically** (fixed at process creation) or **dynamically** (adjusted during execution based on behavior or resource usage).

This approach ensures that important tasks are served *quickly* but can lead to **starvation**, where low-priority processes wait *indefinitely* and ultimately might not meet the deadline. To address this, many systems use **aging**, a technique that gradually increases the priority of waiting processes over time to ensure fairness.

{:.highlight}
Priority scheduling is used in both **preemptive** and **non-preemptive** forms. In preemptive systems, a higher-priority process can interrupt a running lower-priority one, while in non-preemptive systems, the current process runs to completion before the scheduler checks priorities again.

#### Real-Life Example: I/O Scheduling in Linux

Linux applies scheduling not just to CPUs but also to I/O operations (e.g., disk reads/writes).
* **The Deadline Scheduler** assigns deadlines to I/O requests and serves them in order to **prevent starvation**.
* It uses FIFO queues internally and checks **deadlines** to ensure timely completion of I/O.

Other Linux I/O schedulers include:
* **CFQ** (Completely Fair Queuing): balances fairness among processes.
* **BFQ** (Budget Fair Queuing): ideal for desktop responsiveness.
* **noop**: a simple FIFO queue, best for SSDs with low seek times.

Deadline scheduling is especially helpful on spinning disks to avoid long delays from deep I/O queues.

#### Deadline Setting

The deadline for each I/O request is determined using a **fixed** timeout value set for read and write operations. They are set by the system but **configurable** by the user.

We can view or modify them using the `sysfs` interface. For example, on a system using the deadline scheduler:

```bash
cat /sys/block/sda/queue/iosched/read_expire
echo 250 > /sys/block/sda/queue/iosched/read_expire
```

This level of control allows you to prioritize latency vs throughput depending on your system’s behavior (e.g., tuning read responsiveness for databases or write buffering for batch logging).

### Scenario

You are reviewing the scheduler design for a toy operating system. The system initially used a simple round-robin policy to decide which process runs next. Over time, new requirements emerged: support for **priority-based scheduling**, and the option to plug in experimental policies like <span class="orange-bold">shortest job first (SJF)</span>. The initial implementation had the scheduling decision logic mixed into the same function that performed context switching, which made policy changes risky and error-prone.

To address this, the developers refactored the code as shown:

```c
// Mechanism: performs the context switch
void context_switch(Process* p) {
    // low-level code to save current state and load p's state
    // update CPU control structures
}

// Policy: selects the next process to run
Process* round_robin(ProcessQueue* q) {
    Process* p = q->head;
    q->head = q->head->next;
    return p;
}

Process* priority_based(ProcessQueue* q) {
    Process* highest = q->head;
    for (Process* p = q->head; p != NULL; p = p->next) {
        if (p->priority > highest->priority) {
            highest = p;
        }
    }
    return highest;
}

// Scheduler entry point
void schedule(ProcessQueue* q, Process* (*policy)(ProcessQueue*)) {
    Process* next = policy(q);
    if (next != NULL) {
        context_switch(next);
    }
}

```


**Answer the following questions:**
1. **Identify** which functions in the above code represent the mechanism and which represent policy.
2. **Explain** clearly how this refactoring improves the maintainability and flexibility of the scheduler.
3. Suppose a new scheduling policy needs to be tested under high-load conditions. How would this design make that *easier* compared to a non-separated design?
4. What would be a <span class="orange-bold">risk</span> or <span class="orange-bold">drawback</span> if policy and mechanism were not separated in this context?

{:.highlight}
> **Hints**:
> * Look for the function that actually performs the "switching."
> * What changes when you try different strategies? What stays the same?
> * Can you reuse the mechanism code when trying new policies?
> * What happens if you want to test **SJF** without modifying existing low-level code?

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
In this code, the `context_switch()` function <span class="orange-bold">is the mechanism</span>. It carries out the low-level task of switching from the currently running process to the next one, handling CPU state and control structures. The functions `round_robin()` and `priority_based()` are <span class="orange-bold">policies</span>; they implement strategies for selecting which process to run next based on different scheduling criteria.
<br><br>
This refactoring improves maintainability and flexibility because it isolates the decision-making logic from the execution mechanism. The scheduling policy can be changed simply by passing a different function to `schedule()` without modifying the underlying context-switching logic. This reduces code duplication, minimizes the risk of introducing errors, and allows developers to update or optimize scheduling strategies independently from core system operations.
<br><br>
If a new scheduling policy like shortest job first (**SJF**) needs to be tested, this design allows developers to implement the new policy as a separate function and pass it into the `schedule()` function. Since the mechanism is reused unchanged, the test can focus entirely on the policy logic, making experimentation and benchmarking under load much easier and safer.
<br><br>
If the policy and mechanism were not separated, every change to scheduling logic would require editing the core scheduler function that also performs context switches. This increases the risk of breaking critical functionality, slows down development, and makes it <span class="orange-bold">harder</span> to test or switch between policies. It also creates <span class="orange-bold">tightly coupled code</span> that is <span class="orange-bold">harder</span> to read, debug, and maintain over time.
</p></div><br>


## The Case of the Hanging Logger

### Background: The Need for Supervisor Call

When user-space programs need to perform actions that require **access** to hardware or protected system resources like r*eading input or writing to a file*, they cannot interact with the hardware (disk) directly. 

Instead, they must **request** these services from the operating system through well-defined interfaces known as <span class="orange-bold">system call APIs</span>. These APIs are typically provided by system libraries (like `libc` in Linux), which act as a **bridge** between the application and the OS kernel. 

Under the hood, the API triggers a software interrupt or trap instruction that switches the CPU from user mode to kernel mode, allowing the kernel to safely perform the requested operation before returning control. 

{:.highlight}
Understanding this interface is critical, especially in cases where responsiveness and efficient resource handling are essential, such as in a logging program that waits for user input.

### Scenario 
You wrote a simple C application to log user activities in real-time. It reads user inputs from the terminal and records the input along with timestamps into a log file. However, you notice the application hangs whenever users stop providing input for extended periods.

Your logging function snippet looks something like this:

```c
#include <unistd.h>
#include <fcntl.h>
#include <string.h>

int main() {
    int fd = open("log.txt", O_WRONLY | O_CREAT | O_APPEND, 0644);
    char buffer[100];
    
    while (1) {
        int n = read(STDIN_FILENO, buffer, sizeof(buffer)); // blocking call
        if (n > 0) {
            write(fd, buffer, n); // write log entry
        }
    }

    close(fd);
    return 0;
}

```

**Answer the following questions:**
1. Clearly explain how your user-space logging program can invoke OS kernel functionalities via APIs. Use your `write()` function call as a concrete example.
2. Parameter passing for system calls can be done in several ways. Briefly describe these three methods with simple conceptual examples (no detailed code necessary):
   * Registers
   * Stack
   * Memory Block / Table
3. Explain what it means for a system call to be "blocking." Using the code snippet above, identify precisely which system call is blocking, and why this causes your logger to hang.
4. Propose a practical modification to the above code snippet to make the program **responsive** even if the user stops typing. Clearly explain your solution, addressing how non-blocking system calls or related system calls (such as `select()` or `poll()`) can be utilized.


{:.highlight}
> **Hints**:
> * How does API connect to OS kernel? Think about how user programs typically request services from the OS kernel. Do they call the kernel directly, or is there something in between?
> * Parameter passing strategies:
>   * **Registers**: CPU registers are very fast but limited in number.
>   * **Stack**: Remember function calls: how do you usually pass parameters to regular functions in C?
>   * **Block or Table**: Useful when you have many parameters. Can you think of a structure that could hold these parameters in memory?
> * Blocking system call: Look carefully at your provided code snippet: where exactly could the code become stuck waiting indefinitely for input?
> * Making it non-blocking: Consider system calls or methods in POSIX C that help you check if input is ready before trying to read it. Is there a way to ask the OS, "Hey, is there anything to read yet?" before actually reading?

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
A user-space logging program <span class="orange-bold">cannot</span> directly execute kernel instructions because it runs in user mode. To request services provided by the OS kernel, the program instead relies on system call APIs provided by the OS libraries. For example, when calling the `write()` function in POSIX C, this API internally issues a system call via a software-generated interrupt (also known as a trap/system call/supervisor call). The OS kernel then temporarily switches to kernel mode to execute the requested task (e.g., writing data to a file) before returning control back to the user program in user mode.
<br><br>
Parameters for system calls can be passed through various methods. Passing parameters using registers involves directly placing the arguments into CPU registers, which is very **fast** but **limits** the number of parameters due to limited registers available. Alternatively, parameters can be passed using the **stack**, where arguments are <span class="orange-bold">pushed</span> onto the calling process's stack, similar to ordinary function calls in C. Finally, a parameter block or table can be used, where all parameters are stored **consecutively** in a memory block, and the address of this block is passed via a single register to the kernel, allowing for an efficient way of **handling multiple or large parameters**.
<br><br>
A system call is termed "blocking" if it pauses the execution of the calling process until the requested action is completed. In the given logging program snippet, the call `read(STDIN_FILENO, ...)` is a <span class="orange-bold">blocking</span> call. This means the program waits indefinitely until user input arrives. This behavior explains why the logging program hangs whenever users stop typing for an extended period.
<br><br>
To avoid this hanging behavior, the logging program can be modified to utilize non-blocking system calls or mechanisms such as `select()` or `poll()`. You might need to look online for its documentation. Specifically, before calling `read()`, the program could use `select()` to *check* whether input is available on the file descriptor (`STDIN_FILENO`). If input is not available, `select()` immediately **returns control**, allowing the program to continue executing other tasks rather than hanging indefinitely. Another option is configuring the file descriptor to non-blocking mode using the `fcntl()` system call (e.g., `fcntl(STDIN_FILENO, F_SETFL, O_NONBLOCK)`), ensuring the program remains responsive even when input is unavailable. 
<br><br>
You will learn more about file systems terminologies (file descriptor, etc) later on.
</p></div><br>


## `PATH` of Least Resistance

### Background: Where is the path to `PATH`? 

The `PATH` environment variable in Unix-like systems (Linux, macOS) defines a list of directories where the shell looks for executable programs when a command is entered. Instead of typing full paths (like `/usr/bin/ls`), users can just type `ls` because it's located in one of the directories listed in `PATH`. This variable is critical for convenience, scripting, and correct system behavior.

Different users and system contexts can define or modify `PATH` in various ways:

* **System-wide default PATH**: Usually defined in files like `/etc/profile`, `/etc/environment`, or shell-specific global files like `/etc/zsh/zshenv` (for Zsh) or `/etc/bash.bashrc` (for Bash).
* **Root user PATH**: Often set in `/root/.bashrc`, `/root/.profile`, or `/root/.zshrc`. Root may also inherit from `/etc/profile`.
* **Regular user PATH**: Can be customized in `~/.bashrc`, `~/.bash_profile`, `~/.profile`, or `~/.zshrc`, depending on the shell in use. These override or extend the system-wide defaults.
* **Login vs non-login shells**: Login shells read `~/.profile` or `~/.bash_profile`; interactive non-login shells read `~/.bashrc` or `~/.zshrc`.
* **Changes via `export PATH=...`** only affect the **current** session unless added to one of the config files above.

Understanding where and how `PATH` is set ensures that commands behave as expected and that custom tools or scripts can be accessed without typing their full paths. Revisit the Command Line lab if you're still unsure. 

### Scenario

You install a new version of Python using a third-party installer (e.g., from python.org or `pyenv`), but your terminal still runs the system-installed Python when you type `python`. You’ve heard that environment variables like `PATH` control how commands are resolved, but you want to understand exactly how the shell finds and executes system programs.

**Answer the following questions:**
1. Run `echo $PATH` and `which python`. Then temporarily prepend a fake directory with a dummy Python script to the front of your `PATH`. What happens when you type `python` now?
2. Create a file called `python` in a new directory and make it executable. Inside it, echo "You’ve been hijacked!". Add that directory to the front of your `PATH`. Try running `python`. What does this demonstrate about system program resolution?
3. Now run `env -i bash --noprofile --norc` to start a clean shell with no environment. Run `python`. What do you notice about `$PATH` and which Python runs? Compare with your original shell.
4. Explain how the OS design distinguishes **system program location and resolution** (which is user-mode behavior) from **kernel system calls** (which are accessed via fixed interfaces like syscall tables).


{:.highlight}
> **Hints:**
> * `$PATH` is searched left to right for the first matching executable name.
> * `env -i` wipes the environment (`-i` stands for ignore environment). Consult the `man` for further details.
> * Try using `ls`, `echo`, or `pwd` inside the clean shell to observe limitations.
> * Consider how `exec()` family functions use `$PATH`.

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
When you run `echo $PATH`, you should see a **list** of directories separated by colons, such as `/usr/local/bin:/usr/bin:/bin`. Running `which python` shows the full path to the first `python` executable found in that list. If you <span class="orange-bold">prepend</span> a custom directory to the `PATH` using `export PATH=/my/custom/bin:$PATH` and put a dummy `python` script in there, running `python` executes your script instead. This shows that the shell uses the `PATH` variable to resolve commands.
<br><br>
After creating a fake `python` script that prints “You’ve been hijacked!” and placing it in a directory at the front of your `PATH`, typing `python` **runs** the dummy script. This confirms that shell command resolution is purely based on the *ordering* of directories in `PATH`, and not tied directly to system calls or kernel services. It's a user-mode behavior controlled entirely by environment variables and shell logic.
<br><br>
When you run `env -i bash --noprofile --norc`, you enter a **clean** shell with no environment variables, including `PATH`. Running `python` now fails or uses a fallback path (e.g., `/bin/python`), showing that the shell cannot resolve usual commands without a valid `PATH`. Compared to your original shell, <span class="orange-bold">this highlights how critical user-level configuration is for system program behavior</span>.
<br><br>
The process of locating and executing system programs is a user-mode activity involving the shell, environment variables, and standard conventions like the `PATH`. In contrast, **kernel system calls are fixed entry points defined by the OS and accessed via software interrupts or CPU instructions** like `syscall` (x86) or `ecall` (RISC-V) . The shell may use system calls like `execve()` to launch a program, but it decides *what* to execute using logic that depends on environment settings like `PATH`. This **separation** between program resolution (policy/user-space) and execution (mechanism/kernel-space) reflects a core OS design principle.
</p></div><br>

















