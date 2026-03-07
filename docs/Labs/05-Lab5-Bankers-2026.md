---
layout: default
permalink: /labs/05-bankers-algorithm-c
title: Banker's Algorithm
description:  Implement Banker's Algorithm, a deadlock prevention method
parent: Labs
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
**Natalie Agus (Summer 2026)**

# Banker's Algorithm
{:.no_toc}

{:.note}
> Recap
> 
> In <span class="pink-bold">deadlock avoidance</span>, before granting a resource request (even if the request is valid and the requested resources are now available), we need to <span style="color:#f77729;"><b>check</b></span> that the request will not cause the system to enter a deadlock state in the future or immediately.

The Banker's Algorithm is a resource allocation and deadlock avoidance algorithm. It can be used to predict and compute whether or not the current request will lead to a <span style="color:#f77729;"><b>deadlock</b></span>.

- If yes, the request for the resources will be <span style="color:#f77729;"><b>rejected</b></span>,
- Otherwise, it will be <span style="color:#f77729;"><b>granted</b></span>.

There are two parts of the Banker's Algorithm:

1. <span style="color:#f77729;"><b>Resource Allocation</b></span> Algorithm
2. <span style="color:#f77729;"><b>Safety</b></span> Algorithm

You will be implementing the <span class="orange-bold">Safety Algorithm</b></span> and then calling it in <span class="orange-bold">Resource Allocation Algorithm</span> in this lab.

# Submission

You are to complete this lab's questionnaire on eDimension and complete all the sections labeled `TODO` in the starter code. There are 4 `TODO` tasks in total.

Once you have completed all the `TODO` in `banker.c`, schedule a checkoff as a Team with your Lab TA by next week Friday 6PM. 

{:.task-title}
> ✅ Checkoff
> 
> You are to demonstrate your code passing all 6 test cases using `make test`. 
 

# Starter Code

### System Requirements

This project requires a C11 compiler, `make`, and POSIX threads (`pthread`). 

```bash
# Ubuntu / Debian
sudo apt install build-essential

# macOS
xcode-select --install
```

To verify installation, do the following:

```bash
gcc --version
make --version
```


<img src="{{ site.baseurl }}//docs/Labs/images/05-Lab5-Bankers-2026/2026-03-06-15-31-05.png"  class="center_seventy no-invert"/>

### Clone

Clone the starter code:

```
git clone https://github.com/natalieagus/cse-lab-banker-2026-starter.git
```

You should have the following files:

```cpp
.[ROOT_PROJECT_DIR]
├── banker.c
├── banker.h
├── main.c
├── Makefile
├── README.md
├── test_banker.c
└── test_files
    ├── q0_answer.txt
    ├── q0.txt
    ├── q1_answer.txt
    ├── q1.txt
    ├── q2_answer.txt
    ├── q2.txt
    ├── q3_answer.txt
    ├── q3.txt
    ├── q4_answer.txt
    ├── q4.txt
    ├── q5_answer.txt
    ├── q5.txt
    └── q6.txt
```

You will be required to modify only certain sections in `banker.c`.

It is important that you follow these rules to ensure that the tester works as intended:
1. Leave *ALL* other files untouched
2. <span class="orange-bold">DO NOT</span> change any public API interfaces
3. <span style="color:#f7007f;"><b>DO NOT</b></span> `print` anything else in `banker.c`. Only type your answers in the given space labeled in the starter code as `TODO X`
4. <span style="color:#f7007f;"><b>DO NOT</b></span> import any other modules

### Test run

You can try compile the code using `make`. It should produce the binary `banker` upon successful compile. You can run `banker` with any of the `test_files/[TEST_FILE].txt. See `makefile` and the `readme.md` for further information.

<img src="{{ site.baseurl }}//docs/Labs/images/05-Lab5-Bankers-2026/2026-03-06-15-34-52.png"  class="center-full no-invert"/>

You can ignore any other warnings for now like unused parameter/variable/function. You will implement those later.

### C Basics

The lab requires you to know a little bit about C language. Since you already know how to code in Python and Java, it shouldn't be really hard to pick up some C syntax on the go. Consult the [Appendix](#appendix) when in doubt. 
# Banker's Algorithm

# Introduction

## System State and Initial Values

{:.info}
In order for Banker's Algorithm to work, we need to know `N` (max process competing for resources) and `M` (max types of resources) beforehand, and how much each process `i` requires each resource type `j` at maximum.

Suppose we have `N` processes competing for `M` different types of resources. We call these processes "<span style="color:#f7007f;"><b>customers</b></span>" (to these available resources). The <span style="color:#f7007f;"><b>maximum</b></span> demand of process `i` for resource `j` can be illustrated using the max matrix:

Process | Resource `j_1` | Resource `j_2` | ...
---------|----------|--------- |---------
 `i_1` | ... | ... | ...
 `i_2` | ... | ... | ...
 `i_3` | ... | ... | ...

We also need to represent the four <span style="color:#f7007f;"><b>system state</b></span>:

1. available: 1 by M vector
   - `available[i]`: the available instances of resource `i`
2. max: N by M matrix
   - `max[i][j]`: maximum demand of process `i` for resource `j` instances (shown above)
3. allocation: N by M matrix
   - `allocation[i][j]`: current allocation of resource `j` instances for process `i`
4. need: N by M matrix
   - `need[i][j]`: how much more of resource `j` instances might be needed by process `i`

For example, the following arrays display the `available`, `max`, `allocation`, and `need` values of a current state of a system with 5 processes: `P0`, `P1`, `P2`, `P3`, and `P4`, and 3 types of resources: `A`, `B`, `C`;

<img src="{{ site.baseurl }}/assets/images/lab3/1.png"  class="center_seventy"/>

## `banker.c`

Open `banker.c` and give it a quick read. You will fill up the `#TODO` sections in this file for this lab. There are several public APIs as well as internal helpers.


### The `Banker` Struct

All the state for the Banker's Algorithm lives in a single struct:

```c
typedef struct {
    int N;                                         // number of customers
    int M;                                         // number of resource types
    int available[MAX_RESOURCES];                   // what is free right now
    int maximum[MAX_CUSTOMERS][MAX_RESOURCES];      // declared max demand per customer
    int allocation[MAX_CUSTOMERS][MAX_RESOURCES];   // currently held by each customer
    int need[MAX_CUSTOMERS][MAX_RESOURCES];          // max - allocation
    pthread_mutex_t lock;                           // protects everything above
} Banker;
```

There is no class, no constructor, no `new` in C. You declare a `Banker` variable, then call `banker_init` to set it up. From there, the typical flow looks like this:

```c
Banker b;

// 1. Initialise with available resources
banker_init(&b, available_resources, n, m);

// 2. Register each customer's maximum demand
banker_set_maximum_demand(&b, 0, max_demand_0);
banker_set_maximum_demand(&b, 1, max_demand_1);

// 3. Customers request and release resources
banker_request_resources(&b, 0, request);   // returns true if safe, false if denied
banker_release_resources(&b, 0, release);

// 4. Inspect current state at any point
banker_print_state(&b);

// 5. Clean up when done
banker_destroy(&b);
```

Notice that every function takes `&b` (the address of the struct) as its first argument. This is how C passes "the object" to operate on. The `&` gives the function a pointer so it can read and modify the actual struct, not a copy of it.

Head to [appendix](#appendix) to know more about these if you're interested.

### Public APIs

These methods are also defined in `banker.h`. They are called by the `main` or `tester` function. 

{:.task}
You will complete the implementation of Resource Request algorithm in `banker_request_resource` (`TODO 2` and `TODO 3`) in this lab. 

The description provided in `banker.h` is self-explanatory:

```cpp

/* PUBLIC API */
/* Note: only public API typically get declared in the header files */

/**
 * Initialise a Banker instance
 *
 * @param b                    pointer to Banker struct (caller-owned)
 * @param available_resources  array of length @p m with initial counts
 * @param n                    number of customers
 * @param m                    number of resource types
 */
void banker_init(Banker *b, const int *available_resources, int n, int m);

/** Clean up (destroy mutex) */
void banker_destroy(Banker *b);

/**
 * Register the maximum demand vector for customer @p cid
 * Must be called before any request/release for that customer
 */
void banker_set_maximum_demand(Banker *b, int cid, const int *max_demand);

/**
 * Customer @p cid requests the resources described in @p request
 *
 * @return true  if the request is granted (state remains safe)
 * @return false if the request is denied  (would be unsafe or invalid)
 */
bool banker_request_resources(Banker *b, int cid, const int *request);

/**
 * Customer @p cid releases the resources described in @p release
 * For simplicity the release is assumed valid (no bounds checking)
 */
void banker_release_resources(Banker *b, int cid, const int *release);

/**
 * Print the current state (Available, Maximum, Allocation, Need)
 * to stdout in the same format as the reference Python implementation
 */
void banker_print_state(const Banker *b);

/**
 * Parse and execute a command file (same format as the Python version)
 *
 * @return 0 on success, non-zero on error
 */
int banker_run_file(const char *filename);
```

### Internal Helpers

These functions will only be utilised inside `banker.c`. 

{:.task}
You will complete the implementation of `banker_snapshot` and `check_safe` (`TODO 1 and TODO 4`) in this lab.

```cpp
/*  INTERNAL HELPERS */
/**
 * array_le — "array less-than-or-equal" (element-wise).
 *
 * Returns true iff a[i] <= b[i] for all 0 <= i < len.
 * This is the vector comparison  Need[i] <= Work  used inside the
 * safety algorithm.
 */
static bool array_le(const int *a, const int *b, int len)v


/** Print a 1-D int array in Python-list style: [1, 2, 3] */
static void print_vector(const int *v, int len)

/** Print a 1-D int array to stderr for logging */
static void log_vector(const int *v, int len)

/* Takes a snapshot of the current bankers state */
/**
 * @p work: a pointer to single-dimensional array of integers
 * @p alloc: a pointer to rows that are each MAX_RESOURCES wide (alloc matrix)
 * @p need: a pointer to rows that are each MAX_RESOURCES wide (need matrix)
 *
 * This function copies the content of current Banker's state: b->available, b->allocation, b->need into temp arrays: @p work, @p alloc, and @p need matrices
 * @return none, arguments are all passed by reference
 */
static void banker_snapshot(const Banker *b, int *work, int alloc[][MAX_RESOURCES], int need[][MAX_RESOURCES])

/*  SAFETY ALGORITHM */
static bool check_safe(int cid, const int *request, int *work, int alloc[][MAX_RESOURCES], int need[][MAX_RESOURCES], int N, int M)

```

## Input Format

The inputs to `banker.c` are inside `test_files/`. All `.txt` files named as `qx.txt` are various input files.

The format are as follows:

1. The first three lines contain values of `N` (number of processes), `M` (number of resource type), and the contents of the `available` vector.
2. For lines starting with `c`, it signifies the `max` demand of a process for ALL resource types.
   - The format is `c,pid,rid_1 rid_2 rid_3 ...`
3. For lines starting with `r`, it signifies a request by a process.
   - The format is: `r,pid,rid_1 rid_2 rid_3 ...`
4. For lines starting with `f`, it signifies a release of resources by a process.
   - The format is: `f,pid,rid_1 rid_2 rid_3 ...`
5. For lines with just `p`, we print the state of the system at that time

The lines in the input are read from top to bottom, and are treated as incoming requests or releases by each process <span style="color:#f77729;"><b>as time progresses</b></span>.

For instance, open `test_files/q0.txt`. It contains 10 lines of data:

```c
1   n,3
2   m,3
3   a,0 0 0
4   c,0,2 2 4
5   c,1,2 1 3
6   c,2,3 4 1
7   r,0,1 2 1
8   r,1,2 0 1
9   r,2,2 2 1
10  p
```

Line 1 and 2 reports `N` and `M`: 3 distinct processes and 3 different types of resources 

- Let's name them as P0, P1, and P2 so that it is easier for us to address them.
- We will also label the 3 resources as A, B, and C.

From line 3, we know that NONE of the resources are available: `available = [0,0,0]`.

From line 4-6, we can initialise the `max` matrix as follows:

```
max = [[2,2,4]
        [2,1,3]
        [3,4,1]]
```

This means `P0` will need <span style="color:#f77729;"><b>at most</b></span>:
- 2 instances of Resource A, 
- 2 instances of Resource B, and 
- 4 instances of Resource C during its lifetime of execution.

*Similar logic applies for P1 and P2.*


Line 7: `r,0,1 2 1` indicates that P0 then start requesting for 1 instance of Resource A, 2 instances of Resource B, and 1 instance of Resource C at this instance. 

{:.note}
Each line that starts with `r` indicates the sequence (of time) of incoming requests. Banker's algorithm MUST be run each time we process lines starting with `r`.

We will need to run the Banker's algorithm now to determine whether this request is `safe`/`not safe` and therefore granted/not granted. The same is done for all lines starting with `r`. We can then print the state of the system at any point in time with `p`.

In this example, no resources are available. So obviously we are met with the *same* state of `available`, `allocation` and `need` after running the file because none of the resource requests can be fulfilled.

You can run the algorithm using the command:

```
./banker test_files/q0.txt
```

The output will report `allocation` matrix remaining at `0`.

```
Customer 0 requesting
[1, 2, 1]
Customer 1 requesting
[2, 0, 1]
Customer 2 requesting
[2, 2, 1]

Current state:
Available:
[0, 0, 0]

Maximum:
[2, 2, 4]
[2, 1, 3]
[3, 4, 1]

Allocation:
[0, 0, 0]
[0, 0, 0]
[0, 0, 0]

Need:
[2, 2, 4]
[2, 1, 3]
[3, 4, 1]
```

Experiment with the other 5 files as well. Try them one by one and study the result. Compare it with `qx_answer.txt`.

```
./banker test_files/q1.txt
./banker test_files/q2.txt
./banker test_files/q3.txt
./banker test_files/q4.txt
./banker test_files/q5.txt
```

You will notice that the starter code will NOT produce the right answer for q3-q5. Your job is to fix that.

## Resource Request Algorithm

This algorithm is already implemented for you in this function:

```c
bool banker_request_resources(Banker *b, int cid, const int *request)
```

The resource allocation algorithm goes as follows.

### Validity Checks

1. If `request[i] <= need[customer_index][i]` for all `i < M`, go to step 2. Otherwise, return `False` <span style="color:#f7007f;"><b>(request rejected)</b></span> since the process has exceeded its maximum claim.

2. If `request[i] <= available[i]` for all `i < M`, go to step 3. Otherwise, return `False`.
   - Process i must <span style="color:#f77729;"><b>wait</b></span> since its requested resources are not immediately available <span style="color:#f7007f;"><b>(request rejected)</b></span>.


This is already implemented for you:

```c
    /* Validity checks */
    for (int j = 0; j < b->M; j++)
    {
        if (request[j] > b->need[cid][j])
        {
            LOG_INFO("DENIED: Customer %d request[%d]=%d exceeds need[%d]=%d",
                     cid, j, request[j], j, b->need[cid][j]);
            LOG_EDUCATIONAL("A process must never request more than its declared maximum need.");
            pthread_mutex_unlock(&b->lock);
            return false;
        }
        if (request[j] > b->available[j])
        {
            LOG_INFO("DENIED: Customer %d request[%d]=%d exceeds available[%d]=%d",
                     cid, j, request[j], j, b->available[j]);
            LOG_EDUCATIONAL("Not enough resources available right now — customer must wait.");
            pthread_mutex_unlock(&b->lock);
            return false;
        }
    }
```



### Demo Example: `REJECT`, request > need + alloc

{:.highlight}
This example demonstrate the case where a request is rejected because a particular process is requesting more than its maximum.

You may run: `./banker test_files/q1.txt` and study the output.

Initially, we know that `allocation` was initialised as 0 and that `available=[5,5,5]` from the third line in `test_files/q1.txt`. The first 3 requests are <span style="color:#f77729;"><b>granted</b></span>:

```
Customer 0 requesting
[1, 2, 1]
Customer 1 requesting
[2, 0, 1]
Customer 2 requesting
[2, 2, 1]
```

After granting the three requests above, we have the system state:

```
Current state:
Available:
[0, 1, 2]

Maximum:
[2, 2, 4]
[2, 1, 3]
[3, 4, 1]

Allocation:
[1, 2, 1]
[2, 0, 1]
[2, 2, 1]

Need:
[1, 0, 3]
[0, 1, 2]
[1, 2, 0]
```

Further request by Process 0: `request=[0,1,0]` is <span class="orange-bold">rejected</span> because Process 0 requests <span style="color:#f77729;"><b>MORE</b></span> than what's been declared at `maximum`.

- It declared that it needs at maximum of 2 resources B (the second type of resource)
- It already held 2 types of resources B --> `Allocation[0][1]: 2`

```
Customer 0 requesting
[0, 1, 0]

Current state: <--- request above is rejected, hence state remains the same.
Available:
[0, 1, 2]

Maximum:
[2, 2, 4]
[2, 1, 3]
[3, 4, 1]

Allocation:
[1, 2, 1]
[2, 0, 1]
[2, 2, 1]

Need:
[1, 0, 3]
[0, 1, 2]
[1, 2, 0]
```


### Preparation to call Safety Algorithm

At this point, the requested resources are available, but we need to <span style="color:#f77729;"><b>check</b></span> if granting the request is <span style="color:#f7007f;"><b>safe</b></span> (will not lead to deadlock) by calling the safety check algorithm.

Before that, we need to do some prep work:
- Allocate a memory region to make temporary `available`, `allocation`, and `need`
- Copy the `available`, `allocation`, and `need` values to these region


We already declare the temp variables for you.
```c
    /* Declare temp variables */
    int work[MAX_RESOURCES];
    int temp_alloc[MAX_CUSTOMERS][MAX_RESOURCES];
    int temp_need[MAX_CUSTOMERS][MAX_RESOURCES];
```

#### Task 1 

Complete the function `banker_snapshot` which should copy the current Banker's state into these new temp vars:

```cpp
/* Takes a snapshot of the current bankers state */
/**
 * @p work: a pointer to single-dimensional array of integers
 * @p alloc: a pointer to rows that are each MAX_RESOURCES wide (alloc matrix)
 * @p need: a pointer to rows that are each MAX_RESOURCES wide (need matrix)
 *
 * This function copies the content of current Banker's state: b->available, b->allocation, b->need into temp arrays: @p work, @p alloc, and @p need matrices
 * @return none, arguments are all passed by reference
 */
static void banker_snapshot(const Banker *b, int *work, int alloc[][MAX_RESOURCES], int need[][MAX_RESOURCES])
{
    /*
    TODO 1: Implement memcpy here
    */

    /* END OF TODO 1 */
}
```

#### Task 2

Once you have completed `banker_snapshot`, call it in `banker_request_resource`:

```cpp

bool banker_request_resources(Banker *b, int cid, const int *request)
{
    // other code

    /* Safety check */
    LOG_EDUCATIONAL("Request passes basic checks, running safety algorithm...");

    /* Declare temp variables */
    int work[MAX_RESOURCES];
    int temp_alloc[MAX_CUSTOMERS][MAX_RESOURCES];
    int temp_need[MAX_CUSTOMERS][MAX_RESOURCES];

    /* Snapshot current state */
    /*
    TODO 2: Call banker_snapshot here
    */

    // other code
}
```

### Calling the Safety Algorithm

We then pass these new data structures (`work`, `alloc`, `need`) to the `safety_check` algorithm, which will return `True` (safe) or `False` (unsafe).

If the outcome of the `safety_check` algorithm is `True`: <span class="orange-bold">UPDATE</span> all system states concerning Process i <span style="color:#f7007f;"><b>(request granted)</b></span>:
 - `available[i] = available[i] - request[i]` for all `i<M`
 - `need[customer_index][i] = need[customer_index][i] - request[i]` for all `i<M`
 - `allocation[customer_index][i] = allocation[customer_index][i] + request[i]` for all `i<M`

Otherwise, if it returns `False`: This means <span style="color:#f7007f;"><b>request rejected</b></span>, and process i has to try again in the future. This is because granting the request results in deadlock in the future.

#### Task 3

Call the safety algorithm with appropriate parameters inside `banker_request_resources` function:

```c
    /*
    TODO 3: Call check_safe algorithm here
    */
    bool safe = true; // modify this line

    /* END OF TODO */

    if (safe)
    {
        /* Apply the allocation for real */
        for (int j = 0; j < b->M; j++)
        {
            b->allocation[cid][j] += request[j];
            b->need[cid][j] -= request[j];
            b->available[j] -= request[j];
        }
        LOG_INFO("GRANTED: Customer %d request.", cid);
    }
    else
    {
        LOG_INFO("DENIED (unsafe): Customer %d request would lead to unsafe state.", cid);
        LOG_EDUCATIONAL("The request is denied because no safe sequence exists after the"
                        " hypothetical allocation. Granting it could lead to DEADLOCK.");
    }
```


The variable `safe` is always set to `True` in the starter code. This means we <span style="color:#f77729;"><b>always grant</b></span> the request as long as the resources are available and the processes do not violate their `max`. For input `q0, q1, q2`, we are <span style="color:#f77729;"><b>lucky</b></span> because we never had any requests that might result in deadlock.

However, if you try running the program with `q3`, you will notice that the printed output is not the same as the answer: `test_files/q3_answer.txt`. We need to <span style="color:#f77729;"><b>implement and call</b></span> the `check_safe` algorithm to ensure that we get the right answer.

## Task 4: The Safety Algorithm

This algorithm is called each time we encounter a resource request (line starting with `r`), but only if we managed to pass the validity checks of the Resource Request Algorithm. It is implemented in the function `check_safe`:

```cpp
/*  SAFETY ALGORITHM 
 * @return true if safe, false otherwise
 */
static bool check_safe(int cid, const int *request, int *work, int alloc[][MAX_RESOURCES], int need[][MAX_RESOURCES], int N, int M)
{
    /* Finish array to determine termination */
    bool finish[MAX_CUSTOMERS];

    /* Variables to track safe sequence */
    int safe_sequence[MAX_CUSTOMERS];
    int seq_len = 0;

    /* Return value */
    bool safe = true;

    /*
    TODO 4: Implement Safety Algorithm here
    */

    /* END OF TODO 4 */

    // other logging code

    return safe;
}
```

You can implement Task 4 by following the steps below.

### Step 1: Create Helper Variables

Four helper variables are created for you:

```cpp
/* Finish array to determine termination */
bool finish[MAX_CUSTOMERS];

/* Variables to track safe sequence */
int safe_sequence[MAX_CUSTOMERS];
int seq_len = 0;

/* Return value */
bool safe = true;
```

### Step 2: Prepare `finish` vector

You need to initialise `finish`: set all elements to `False`. This indicates that none of the processes has finished yet.

### Step 3: Hypothetically grant the current `request`
<span style="color:#f77729;"><b>Hypothetically</b></span> grant the current request by customer `customer_index` by updating:
- `work[i] = work[i] - request[i]` for all `i<M`
- `need[customer_index][i] = need[customer_index][i] - request[i]` for all `i<M`
- `allocation[customer_index][i] = allocation[customer_index][] + request[i]` for all `i<M`
  
Note that this request granting is _hypothetical_ because we are modifying a copy of the Banker's state via `work`, `temp_alloc` and `temp_need`. In reality, we haven't granted the request yet, we simply compute this hypothetical situation and decide whether it will be `safe` or `unsafe`.

### Step 4: Try to find process `i` that can finish

Find an index `i` (which is a _customer_) such that:
- `finish[i] == False` <span style="color:#f7007f;"><b>and</b></span>
- `need[i][j] <= work[j]` for <span style="color:#f7007f;"><b>all</b></span> `j<M`.
- The two above condition signifies that an incomplete Customer `i` can _complete_ even after this request by `customer_index` is granted

If such `i` from exists do the following:
- **Set**: `work[j] = work[j] + allocation[i][j]` for <span style="color:#f7007f;"><b>all</b></span> `j<M`.
    - This signifies that a Customer `i` that can _complete_ will free its _currently_ allocated resources.
- **Set**: `finish[i] = True`

If such `i` does <span class="orange-bold">not</span> exist, go to Step 5. The algorithm should terminate soon.

<span style="color:#f7007f;"><b>Then, repeat this Step 4</b></span>

#### Tracking Safe Sequence

You might want to store the values of `i` each time you execute this Step 4 in `safe_sequence` array to store a possible **safe execution sequence**. It will greatly help your debugging process. Head to the [lecture notes](https://natalieagus.github.io/50005/os/deadlock#part-2-the-safety-algorithm) for a summary.  

This is not required to pass the tester for this lab.


### Step 5: Termination

If you reach here, that means there's no more process `i` that can finish. Check the `finish` array: 
- If `finish[i] == True` for <span style="color:#f7007f;"><b>all</b></span> `i<N`, then it means the system is in a <span style="color:#f77729;"><b>safe state</b></span>. Return `True`.
- Else, the system is <span style="color:#f77729;"><b>not in a safe state</b></span>. Return `False`.


## Potential Careless Mistake

Step 4 contains a `while` loop. 
- We need to find Process `i` that can finish.
- If we do, we need to update `work` matrix. 
- When `work` matrix is updated, we can find more Process `i` that can finish.

## Checking Implementation

Once you have completed all the tasks, run the Banker algorithm with `q3`:

```
./banker test_files/q3.txt
```

In this test file, P0 is making the request `[1,0,3]` and it is <span style="color:#f77729;"><b>granted</b></span> as shown in the `allocation` matrix:

```
Customer 0 requesting
[1, 0, 3]

Current state:
Available:
[4, 2, 1]

Maximum:
[2, 2, 4]
[2, 1, 3]
[3, 1, 1]

Allocation:
[1, 0, 3]
[0, 0, 0]
[0, 0, 0]

Need:
[1, 2, 1]
[2, 1, 3]
[3, 1, 1]
```

> You can <span style="color:#f77729;"><b>verify</b></span> whether any requests so far is granted by checking that the `allocation` matrix value adds up to the requests made.

Then, P1 requests for `[1,1,1]`. This request will pass resource allocation algorithm's validity checks, but it should NOT pass the safety check since it leads to an `unsafe` state.

Hence the request is rejected and the state of the system should remain the same:

```
Customer 1 requesting
[1, 1, 1]

Current state:
Available:
[4, 2, 1]

Maximum:
[2, 2, 4]
[2, 1, 3]
[3, 1, 1]

Allocation:
[1, 0, 3]
[0, 0, 0]
[0, 0, 0]

Need:
[1, 2, 1]
[2, 1, 3]
[3, 1, 1]
```

The same logic applies to samples in `q4` and `q5` as well:

- In q4, the last request by P0: `[0,2,0]` leads to an `unsafe` state and therefore <span style="color:#f77729;"><b>rejected</b></span>.
- In q5, the request by P1: `[2,0,1]` is rejected because it is `unsafe`. However, after P0 release the resources it held: `[1,3,4]`, a <span style="color:#f7007f;"><b>repeated</b></span> request by P1 for the same set of resources `[2,0,1]` is granted (`safe`).


# Further Notes

## Rejecting Requests

If a resource request is <span style="color:#f77729;"><b>rejected</b></span>, dont panic, it's not the end of the world. The process/consumer just need to <span style="color:#f77729;"><b>try</b></span> to request it again in the future.

How can this be implemented? Usually schedulers are programmed to tackle this kind of cases; e.g: they can be placed to a special queue and will periodically prompt for resource request until granted, or there exists some kind of event-driven solution -- it's free-for-all to implement.

## Resource Release 

On the contrary, resource **release** request is **always granted**. This is implemented by `banker_release_resource` function:

```cpp
void banker_release_resources(Banker *b, int cid, const int *release)
{
    /* Print the release */
    printf("Customer %d releasing\n", cid);
    print_vector(release, b->M);

    pthread_mutex_lock(&b->lock);
    for (int j = 0; j < b->M; j++)
    {
        b->allocation[cid][j] -= release[j];
        b->need[cid][j] += release[j];
        b->available[j] += release[j];
    }
    LOG_INFO("Customer %d released resources.", cid);
    pthread_mutex_unlock(&b->lock);
}
```

For the simplicity of this lab, we assume that a process will <span style="color:#f77729;"><b>not</b></span> release more than what has been allocated to them. Each test files follows this assumption.

## Synchronisation using `mutex`

Since `max, allocation, need` and `available` are shared data structures among all these methods, we use a `mutex` lock to protect it. The mutex is created under `banker_init`:

```cpp
void banker_init(Banker *b, const int *available_resources, int n, int m)
{
    memset(b, 0, sizeof(Banker));
    b->N = n;
    b->M = m;
    for (int j = 0; j < m; j++)
        b->available[j] = available_resources[j];

    /* Create a mutex to protect the thread executing Banker's algorithm */
    pthread_mutex_init(&b->lock, NULL);

    LOG_INFO("Banker initialised: %d customers, %d resource types", n, m);
    if (LOG_LEVEL >= 1)
    {
        fprintf(stderr, "[INFO]  Initial available = ");
        log_vector(b->available, m);
        fprintf(stderr, "\n");
    }
}
```


We guard the <span style="color:#f77729;"><b>critical sections</b></span> of the Resource Request algorithm with `pthread_mutex_lock` and `pthread_mutex_unlock` to make it <span style="color:#f77729;"><b>thread safe</b></span>:


```cpp

bool banker_request_resources(Banker *b, int cid, const int *request)
{
    /* Attempt to obtain lock */
    pthread_mutex_lock(&b->lock);

    // Implementation

    /* Release lock */
    pthread_mutex_unlock(&b->lock);
    return safe;
}
```

# Running the test file

You can run all test files at one go using the command:

```
make test
```

This will run your compiled `banker` against all 6 test cases: q0 to q5. If all goes well, the following message will be printed:

<img src="{{ site.baseurl }}//docs/Labs/images/05-Lab5-Bankers-2026/2026-03-06-17-50-17.png"  class="center_full no-invert"/>



Note that the tester performs a simple output string matching. Therefore it is <span style="color:#f7007f;"><b>crucial</b></span> for you not to print anything else (comment it out) before running the tester.

## Debugging 

You should use a proper debugger to debug the files. One way is to use [VSCode C/C++ debugger](https://code.visualstudio.com/docs/cpp/cpp-debug). 

<img src="{{ site.baseurl }}//docs/Labs/images/05-Lab5-Bankers-2026/2026-03-06-17-56-46.png"  class="center_seventy no-invert"/>

You can use the following `launch.json` and `tasks.json` files. Create it under `.vscode/` folder in your project directory.

`launch.json`:
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug banker",
      "type": "lldb",
      "request": "launch",
      "program": "${workspaceFolder}/banker",
      "args": ["test_files/q3.txt"],
      "cwd": "${workspaceFolder}",
      "preLaunchTask": "make"
    }
  ]
}
```

`tasks.json`:
```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "make",
      "type": "shell",
      "command": "make",
      "options": {
        "cwd": "${workspaceFolder}"
      },
      "problemMatcher": ["$gcc"],
      "group": {
        "kind": "build",
        "isDefault": true
      }
    }
  ]
}
```


# Summary 

In this lab, you have implemented the Banker's Algorithm using C. There are 4 Tasks in total (`TODO 1` - `TODO 4`) and you must complete them all for the checkoffs with our TAs.

The Banker's Algorithm is used in operating systems to <span class="orange-bold">avoid</span> deadlocks by safely allocating resources to processes. It's a deadlock **avoidance** technique that checks whether granting a resource request will keep the system in a safe state: i.e., whether all processes can still eventually complete. 

It’s mainly taught as a *theoretical model and rarely used in modern OS kernels* due to its high runtime complexity and need for advanced knowledge of future resource requests. However, it serves as a **foundation** for understanding resource allocation, particularly in systems where resource usage patterns are predictable, such as embedded or real-time systems with strict guarantees.

## Usage in Modern System

**The Banker's Algorithm is not used in modern general-purpose operating systems** like Linux, Windows, or macOS. It's mostly a teaching tool or used in **very controlled environments** like embedded or real-time systems where all resource requirements are known in advance. In practice, mainstream OS avoid deadlocks using a combination of:

1. **Deadlock prevention**: Structuring the system to make deadlocks impossible (e.g., by enforcing **resource ordering**).
2. **Deadlock avoidance (rare)**: Like Banker's, but impractical due to overhead.
3. **Deadlock detection and recovery**: Allowing deadlocks to happen, then detecting and breaking them (e.g., killing a process).
4. **Ostrich algorithm**: Ignoring deadlocks entirely if they are rare and impact is low (common in general OSes).

So in modern OS, deadlocks are often just **detected (not prevented)** using techniques like wait-for graphs, especially in database engines or advanced resource managers.

# Appendix

## C for Java/Python Developers

A quick reference for the C concepts used in this project. This is not a full C tutorial. It covers the things that will look unfamiliar if you are coming from Java or Python.

### Structs (there are no classes)

C has no classes, no methods, no inheritance. A `struct` is just a block of named fields packed together in memory. It has the following syntax and we use it to hold the states of our Bank:

```c
typedef struct {
    int N;
    int M;
    int available[MAX_RESOURCES];
    int allocation[MAX_CUSTOMERS][MAX_RESOURCES];
    pthread_mutex_t lock;
} Banker;
```

In Java you would write `banker.requestResources(...)` as a method call on the object. In C, you write a free function and pass the struct as the first argument:

```c
bool banker_request_resources(Banker *b, int cid, const int *request);
```

Every function that operates on a `Banker` takes a `Banker *b` as its **first** parameter. This is the C equivalent of `this` or `self`. You access fields with the arrow operator: `b->available[j]`, `b->N`, etc.

The arrow `->` is shorthand for _dereferencing_ then accessing: `b->N` means `(*b).N`.

### Pointers

A pointer holds a memory address. That is all it is. We use the syntax `*` for a pointer. We can dereference a pointer by putting another `*` symbol in front of it.

```c
int x = 42;
int *p = &x;   // p holds the address of x
*p = 10;       // dereference p, now x == 10
```

| Syntax     | Meaning                                          |
| ---------- | ------------------------------------------------ |
| `int *p`   | Declare p as a pointer to int                    |
| `&x`       | "Address of x"                                   |
| `*p`       | "The value at the address p holds" (dereference) |
| `p->field` | Shorthand for `(*p).field`                       |

In Java and Python, objects are always passed by reference. In C, everything is passed **by value** by default. If you write:

```c
void foo(Banker b) { ... }   // receives a COPY of the entire struct
```

The function gets its own copy. Any changes are lost when it returns. To modify the original, you pass a pointer:

```c
void foo(Banker *b) { ... }  // receives the ADDRESS of the struct
```

This is why every public API function takes `Banker *b`, not `Banker b` because we need to modify the banker's state.

### Arrays decay to pointers

When you pass an array to a function, C does <span class="orange-bold">not</span> copy it. The array name "decays" into a pointer to its first element:

```c
void print_vector(const int *v, int len);

int arr[3] = {1, 2, 3};
print_vector(arr, 3);   // arr decays to &arr[0]
```

This is also why functions cannot know the length of an array from the pointer alone. You always pass the length separately.

### 2D Arrays vs Double Pointers

A 2D array like `int alloc[64][64]` is a single **contiguous** block of `64 * 64` integers in memory. It is **not** an array of pointers. An array of pointers is basically an array of addresses where each element is an address that can point anywhere else and not necessarily contiguous.

When passing a 2D array to a function, the first dimension decays to a pointer but the compiler still needs the second dimension to compute offsets:

```c
// These two signatures are equivalent:
void foo(int alloc[][MAX_RESOURCES], int rows);
void foo(int (*alloc)[MAX_RESOURCES], int rows);
```

Both mean: _a pointer to rows, where each row is `MAX_RESOURCES` ints wide._

On the other hand, an `int **` (double pointers) is a completely different memory layout (an array of pointers to separately allocated rows). You cannot interchange them no matter what AI says.

| Declaration   | Memory layout                    | Access `[i][j]`                                          |
| ------------- | -------------------------------- | -------------------------------------------------------- |
| `int a[N][M]` | One flat block of N\*M ints      | `base + i*M + j` (one offset)                            |
| `int **a`     | Pointer to array of row pointers | Follow outer ptr, then follow row ptr (two dereferences) |

### Header Files and Include Guards

C splits declarations (what exists) from definitions (how it works). Here we provide you with `banker.h` and `banker.c`:

| File       | Contains                                       | Analogy                     |
| ---------- | ---------------------------------------------- | --------------------------- |
| `banker.h` | Struct definition, function signatures, macros | Java interface / Python ABC |
| `banker.c` | Function bodies (the actual code)              | Java class implementation   |

Any `.c` file that wants to use `Banker` writes `#include "banker.h"`. The preprocessor literally copy-pastes the header content into that file before compilation.

The include guard prevents double-inclusion:

```c
#ifndef BANKER_H       // if BANKER_H is not yet defined...
#define BANKER_H       // define it now

// ... all declarations ...

#endif                 // end of guard
```

If a file includes `banker.h` twice (directly or indirectly), the second time `BANKER_H` is already defined, so the `#ifndef` is false and the entire block is skipped. Without this you get "duplicate definition" errors during compilation.

### `static` Functions

In Java, `private` hides a method from other classes. In C, the keyword `static` on a function means "visible only within this `.c` file" so it kinda works the same way as Java's `private:

```c
static bool check_safe(...)  { ... }   // only banker.c can call this
static void print_vector(...) { ... }  // only banker.c can call this

bool banker_request_resources(...) { ... }  // anyone who includes banker.h can call this
```

If it is not in the header and it is marked `static`, it is internal. If it is in the header without `static`, it is part of the public API. This is C's version of access control.

### `const` Correctness

`const` tells the compiler (and the reader) that a value will not be modified through this pointer:

```c
void banker_init(Banker *b, const int *available_resources, int n, int m);
```

`Banker *b` means the function **will** modify the struct (initialising it).
`const int *available_resources` means the function **will not** modify the array it reads from.

If you accidentally try to write through a `const` pointer, the compiler gives you an error.

| Declaration          | What is const                                     |
| -------------------- | ------------------------------------------------- |
| `const int *p`       | Cannot modify the int through p (the data)        |
| `int *const p`       | Cannot change where p points (the pointer itself) |
| `const int *const p` | Both                                              |

In this project you will mostly see `const int *`, meaning "read-only view of this data."

### `memcpy` and `memset`

C has <span class="orange-bold">no built-in array copy or array fill</span>. You use these two library functions from `<string.h>`:

```c
// Copy n bytes from src to dst
memcpy(dst, src, num_bytes);

// Set n bytes of dst to value (usually 0)
memset(dst, 0, num_bytes);
```

In the project, `banker_snapshot` should use `memcpy` to deep-copy the banker's matrices into local buffers:

```c
memcpy(work, b->available, sizeof(int) * (size_t)b->M);
```

This copies `b->M` integers (each `sizeof(int)` bytes) from `b->available` into `work`. It is the C equivalent of Python's `copy.deepcopy()` for flat arrays.

`memset` is used to zero-initialise:

```c
memset(finish, false, sizeof(bool) * (size_t)N);
```

Alternatively, you can just loop through the array and set each value to zero, but `memset` is just a neater way to do it.

### Preprocessor Macros

Lines starting with `#` are handled by the **preprocessor** **before** compilation. They do text substitution to make it easier for programmers to code using symbols instead of plain numbers. Here we list the use cases.

#### Simple constants

```c
#define MAX_CUSTOMERS 64
```

Every occurrence of `MAX_CUSTOMERS` in the code is replaced with `64` before the compiler sees it. Similar to `static final int` in Java or a module-level constant in Python, but it is pure text replacement.

#### Parameterised macros

```c
#define LOG_INFO(...)                                \
    do {                                             \
        if (LOG_LEVEL >= 1) {                        \
            fprintf(stderr, "[INFO]  ");             \
            fprintf(stderr, __VA_ARGS__);            \
            fprintf(stderr, "\n");                   \
        }                                            \
    } while (0)
```

`__VA_ARGS__` captures whatever arguments you pass. The `do { ... } while (0)` wrapper is a standard C trick so the macro behaves like a single statement (safe to use after `if` without braces).

#### Conditional compilation

```c
#ifndef LOG_LEVEL
#define LOG_LEVEL 0
#endif
```

This sets a default. But if you compile with `-DLOG_LEVEL=2`, the compiler defines it before the file is processed, so the `#ifndef` is false and the default is skipped. This is how `make verbose` changes behaviour without editing code.

### `pthread_mutex_t` (Locking)

{:.note}
Consult the lecture notes about [mutex]({{ site.baseurl }}/os/synchronization#mutex-lock) if you forgot what it does.

In Python you write `lock.acquire()` / `lock.release()`. In Java you write `synchronized`. In C with POSIX threads we do:

```c
pthread_mutex_t lock;

pthread_mutex_init(&lock, NULL);    // create the lock (once)
pthread_mutex_lock(&lock);          // acquire (blocks if held)
pthread_mutex_unlock(&lock);        // release
pthread_mutex_destroy(&lock);       // clean up (once, when done)
```

In this project the mutex is a field inside the `Banker` struct. Every function that reads or writes the shared matrices locks it first and unlocks before returning. This is the same concept as Python's `threading.Lock()` or Java's `synchronized` block, just explicit.

### Compilation Model

Java compiles each `.java` into a `.class` and the JVM links them at runtime. Python just **interprets**. C has a multi-step build:

```
  banker.c  -->  [compiler]  -->  banker.o  (object file)
  main.c    -->  [compiler]  -->  main.o    (object file)
                                     |
                              [linker]
                                     |
                                  banker   (executable)
```

**Step 1 (compile):** Each `.c` file is compiled independently into a `.o` (object file) containing machine code. It only needs the `.h` headers to know what functions exist, not how they work.

**Step 2 (link):** The linker combines all `.o` files and resolves function calls across files. If `main.o` calls `banker_request_resources`, the linker finds the actual code in `banker.o` and connects them. Linker is a huge topic by itself, you don't know what it means (big picture), it might be worth searching online.

This is why you can change `main.c` _without_ recompiling `banker.c`. The Makefile tracks these dependencies.

### Makefile Basics

A Makefile describes build rules, we have seen this in our first lab. Each rule has a **target**, **dependencies**, and **commands**:

```makefile
banker.o: banker.c banker.h
	gcc -std=c11 -c banker.c -o banker.o
```

This says: "to produce `banker.o`, you need `banker.c` and `banker.h`. Run this gcc command." If neither dependency has changed since the last build, `make` skips the rule.

| Flag            | Meaning                                           |
| --------------- | ------------------------------------------------- |
| `-c`            | Compile only, do not link (produce `.o`)          |
| `-o name`       | Name the output file                              |
| `-Wall -Wextra` | Enable warnings (catch bugs early)                |
| `-std=c11`      | Use the C11 standard                              |
| `-g`            | Include debug info (for gdb)                      |
| `-pthread`      | Link the POSIX threads library                    |
| `-DLOG_LEVEL=2` | Define a preprocessor macro from the command line |

{:.note}
It is very common to ask AI to write a makefile for you, and we think it is a suitable usage of AI.

### Quick Syntax C Cheat Sheet

You can refer to [this site](https://learnxinyminutes.com/c/) for more stuffs but these are sufficient for this lab:

| Java / Python                        | C                                                         |
| ------------------------------------ | --------------------------------------------------------- |
| `System.out.println(x)` / `print(x)` | `printf("%d\n", x);`                                      |
| `new int[n]` / `[0]*n`               | `int arr[n];` (stack) or `malloc(n * sizeof(int))` (heap) |
| `arr.length` / `len(arr)`            | No equivalent. Track length yourself.                     |
| `true` / `false` / `True` / `False`  | `true` / `false` (with `#include <stdbool.h>`)            |
| `null` / `None`                      | `NULL`                                                    |
| `String`                             | `char *` or `char arr[N]` (null-terminated)               |
| `try/catch`                          | No exceptions. Check return values.                       |
| Garbage collection                   | Manual: `malloc()` to allocate, `free()` to deallocate    |
| `import`                             | `#include "file.h"` or `#include <stdlib.h>`              |
| `this` / `self`                      | Pass struct pointer explicitly as first argument          |

### Memory Management (malloc and free)

In Java and Python, you create objects freely and the garbage collector cleans them up. C has no garbage collector. If you allocate memory on the heap, you are responsible for freeing it.

```c
// Allocate an array of 10 ints on the heap
int *arr = malloc(10 * sizeof(int));
if (arr == NULL) {
    // malloc returns NULL if it fails
    fprintf(stderr, "Out of memory\n");
    return 1;
}

// Use it...
arr[0] = 42;

// When done, free it. This is YOUR job.
free(arr);
arr = NULL;  // good habit: avoid dangling pointer
```

There are **two** places memory can live:

| Location  | How                      | Lifetime                                   | Cleanup                              |
| --------- | ------------------------ | ------------------------------------------ | ------------------------------------ |
| **Stack** | `int arr[64];`           | Automatic. Dies when the function returns. | Nothing to do (remember your 50.002) |
| **Heap**  | `int *arr = malloc(...)` | Lives until you call `free()`.             | You must call `free()` or it leaks.  |

If you `malloc` but never `free`, that memory is **leaked**. It stays allocated until the process exits. In a long-running program this adds up and eventually you run out of memory. In a short lab program you might not notice, but it is still a bug.

If you `free` and then access the pointer again, that is a **use-after-free**. The memory might have been reused for something else. This causes mysterious crashes or corrupted data. Setting the pointer to `NULL` after freeing helps catch this earlier.

In this lab we mostly avoid heap allocation entirely. The `Banker` struct is declared on the stack in `banker_run_file`, and the temporary arrays in `check_safe` are also stack-allocated with fixed sizes like `int work[MAX_RESOURCES]`. This is a deliberate design choice: by using fixed-size arrays with `MAX_CUSTOMERS` and `MAX_RESOURCES`, we avoid `malloc`/`free` altogether and eliminate an entire category of bugs. The tradeoff is that we waste some memory if the actual number of customers is much smaller than 64, but for a teaching codebase that simplicity is worth it.

You will play more with memory management later on during Programming Assignment 1.

### Summary

C is not an easy language to use if you're used to Python and Java because it is the lowest-level language most programmers will ever use. It beats assembly any day though. Since you survived Beta Assembly, C language shouldn't be too hard to learn. It just takes way more instructions to implement the same thing in modern languages like Python and Java.

Just keep in mind these few things for now:

1. **No objects.** Structs hold data, free functions operate on them. Pass the struct pointer as the first argument.
2. **No automatic memory.** You manage lifetimes. In this project we mostly use stack allocation (local arrays), which is automatically cleaned up when the function returns.
3. **Pointers are explicit.** You choose whether to pass by value or by pointer. You dereference with `*` and take addresses with `&`.
4. **Headers are manual.** You split declarations and definitions yourself and use include guards.
5. **The preprocessor is powerful.** Macros, conditional compilation, and build-time configuration all happen before the compiler runs.



