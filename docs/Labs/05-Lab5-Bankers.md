---
layout: default
permalink: /labs/05-bankers-algorithm
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
**Natalie Agus (Summer 2024)**

# Banker's Algorithm
{:.no_toc}

You need Python 3.10.x or above to complete this lab.
{:.info}

In <span style="color:#f7007f;"><b>deadlock avoidance</b></span>, before granting a resource request (even if the request is valid and the requested resources are now available), we need to <span style="color:#f77729;"><b>check</b></span> that the request will not cause the system to enter a deadlock state in the future or immediately.

The Banker's Algorithm is a resource allocation and deadlock avoidance algorithm. It can be used to predict and compute whether or not the current request will lead to a <span style="color:#f77729;"><b>deadlock</b></span>.

- If yes, the request for the resources will be <span style="color:#f77729;"><b>rejected</b></span>,
- Otherwise, it will be <span style="color:#f77729;"><b>granted</b></span>.

There are two parts of the Banker's Algorithm:

1. <span style="color:#f77729;"><b>Resource Allocation</b></span> Algorithm
2. <span style="color:#f77729;"><b>Safety</b></span> Algorithm

You will be implementing the <span style="color:#f77729;"><b>Safety Algorithm</b></span> and then calling it in Resource Allocation Algorithm in this lab.

# Submission

You are to complete this lab's questionnaire on eDimension as you complete the tasks. 

Once you have completed `Banker.py`, schedule a checkoff as a Team with your Lab TA by next week Friday 6PM. 

{:.task-title}
> ✅ Checkoff
> 
> You are to demonstrate your code passing all 6 test cases using `banker_test.py`. 
 

# Starter Code

## Install Python 3.10.x or above

If you don't have Python 3.10.x installed already, here's a list of commands you can enter to install:

```bash
sudo apt update && sudo apt upgrade
sudo apt install software-properties-common -y
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update
sudo apt install python3.10 -y
```

When done, check that it is installed, and install pip as well:

```bash
python3.10 --version
sudo apt install python3-pip
```

## Clone

Download this repository:

```
git clone https://github.com/natalieagus/lab_banker
```

You should have the following files:

```cpp
lab_banker/
  |- test_files/
      |- q0_answer.txt
      |- q0.txt
      |- q1_answer.txt
      |- q1.txt
      |- q2_answer.txt
      |- q2.txt
      |- q3_answer.txt
      |- q3.txt
      |- q4_answer.txt
      |- q4.txt
      |- q5_answer.txt
      |- q5.txt
  |- .gitignore
  |- banker_test.py
  |- banker.py
  |- README.md
```

You will be required to modify only certain sections in `banker.py`.

Leave *ALL* other files untouched. Also, <span style="color:#f7007f;"><b>DO NOT</b></span> `print` anything else in `banker.py`. Only type your answers in the given space labeled in the starter code as `TASK 1` and `TASK 2`. Remember, <span style="color:#f7007f;"><b>DO NOT</b></span> print anything else, and <span style="color:#f7007f;"><b>DO NOT</b></span> import any other modules, and <span style="color:#f7007f;"><b>DO NOT</b></span> modify any other instructions.
{:.error}

# Banker's Algorithm

## Prerequisite

Suppose we have `N` processes competing for `M` different types of resources. We call these processes "<span style="color:#f7007f;"><b>customers</b></span>" (to these available resources).

In order for the Banker's Algorithm to work, need to know the <span style="color:#f7007f;"><b>maximum</b></span> demand of process `i` for resource `j` instances.
{:.info}

## System State

We also need to represent the <span style="color:#f7007f;"><b>system STATE</b></span>, such as the amount of resources available per-type-per-process using the following data structures:

1. available: 1 by M vector
   - `available[i]`: the available instances of resource `i`
2. max: N by M matrix
   - `max[i][j]`: maximum demand of process `i` for resource `j` instances
3. allocation: N by M matrix
   - `allocation[i][j]`: current allocation of resource `j` instances for process `i`
4. need: N by M matrix
   - `need[i][j]`: how much more of resource `j` instances might be needed by process `i`

For example, the following arrays display the `available`, `max`, `allocation`, and `need` values of a current state of a system with 5 processes: `P0`, `P1`, `P2`, `P3`, and `P4`, and 3 types of resources: `A`, `B`, `C`;

<img src="{{ site.baseurl }}/assets/images/lab3/1.png"  class="center_seventy"/>

## Banker.py

Open `Banker.py` and give it a quick read.

There are a few methods defined in `Class Banker`:

1. `set_maximum_demand`: implemented for you, this populates the corresponding row of `max` matrix
2. `request_resources`: request resource algorithm implementation. <span style="color:#f77729;"><b>Task 1 is here</b></span>.
3. `release_resources`: releases resources borrowed by a customer. Assume release is valid for <span style="color:#f77729;"><b>simplicity</b></span>
4. `check_safe`: safety check algorithm implementation. <span style="color:#f77729;"><b>Task 2 is here</b></span>.
5. `print_state` and `run_file`: functions to load the input files and printing the states of the system: `available, max, allocation, need` values.

## Input Format

The inputs to `banker.py` are inside `test_files/`. All `.txt` files named as `qi.txt` are various input files.

The format are as follows:

1. The first three lines contain values of `N` (number of processes), `M` (number of resource type), and the contents of the `available` vector.
2. For lines starting with `c`, it signifies the `max` demand of a process for ALL resource types.
   - The format is `c,pid,rid_1 rid_2 rid_3 ...`
3. For lines starting with `r`, it signifies a request by a process.
   - The format is: `r,pid,rid_1 rid_2 rid_3 ...`
4. For lines starting with `f`, it signifies a release of resources by a process.
   - The format is: `f,pid,rid_1 rid_2 rid_3 ...`
5. For lines with just `p`, we print the state of the system

The lines in the input are read from top to bottom, and are treated as incoming requests or releases by each process <span style="color:#f77729;"><b>as time progresses</b></span>.

For instance, open `test_files/q0.txt`:

```
n,3
m,3
a,0 0 0
c,0,2 2 4
c,1,2 1 3
c,2,3 4 1
r,0,1 2 1
r,1,2 0 1
r,2,2 2 1
p
```

From the first two lines, we know that there are 3 distinct processes (`N`) competing for 3 different types of resources (`M`).

- Let's name them as P0, P1, and P2 so that it is easier for us to address them.
- Also label each of the 3 resources as A, B, and C.

From the third line, we know that NONE of the resources are available: `available = [0,0,0]`.

In the fourth to sixth line, we can initialise the `max` matrix as follows:

```
max = [ [2,2,4]
        [2,1,3]
        [3,4,1] ]
```

This means `P0` will need <span style="color:#f77729;"><b>at most</b></span> 2 instances of Resource A, 2 instances of Resource B, and 4 instances of Resource C during its lifetime of execution. Similar logic for P1 and P2.

In the seventh line `r,0,1 2 1`, P0 starts requesting for 1 instance of Resource A, 2 instances of Resource B, and 1 instance of Resource C simultanously.

We will need to run the Banker's algorithm now to determine whether this request is `safe`, and to be granted.
{:.note}

This is already **done** for you inside `run_file` function. Whenever line with `r` is encountered, the function `request_resources` is called.

- If the request is granted, we need to update `available`, `allocation` and `need`. This is already done for you inside `request_resources` function.
- If the request is not granted, we simply ignore that request. The values of `available`, `allocation` and `need` remains the same. This is already done for you inside `request_resources` function.

The same is done for all lines starting with `r`. We can then print the state of the system at any point in time with `p`.

In this example, no resources are available. So obviously we are met with the *same* state of `available`, `allocation` and `need` after running the file because none of the resource requests can be fulfilled.

You can run the algorithm using the command:

```
python3.10 banker.py test_files/q0.txt
```

The output is as such as expected, with `allocation` matrix remaining at 0.

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

Experiment with the other 5 files as well. Try them one by one and study the result.

```
python3.10 banker.py test_files/q1.txt
python3.10 banker.py test_files/q2.txt
python3.10 banker.py test_files/q3.txt
python3.10 banker.py test_files/q4.txt
python3.10 banker.py test_files/q5.txt
```

## Resource Request Algorithm

This algorithm is already implemented for you inside `request_resources` function. This algorithm is called each time we encounter a resource request (line starting with `r`). The function receives two parameters and returns a bool:

```py
def request_resources(self, customer_index, request):
    """
    Request resources for a customer loan.

    Parameters
    ----------
    customer_index : int
        the customer's index (0-indexed)
    request : list[int]
        an array of the requested count for each resource

    Returns
    -------
    True : if the requested resources can be loaned
    False : otherwise
    """
```

The resource allocation algorithm goes as follows:

1. If `request[i] <= need[customer_index][i]` for all `i < M`, go to step 2. Otherwise, return `False` <span style="color:#f7007f;"><b>(request rejected)</b></span> since the process has exceeded its maximum claim.

2. If `request[i] <= available[i]` for all `i < M`, go to step 3. Otherwise, return `False`.

   - Process i must <span style="color:#f77729;"><b>wait</b></span> since its requested resources are not immediately available <span style="color:#f7007f;"><b>(request rejected)</b></span>.

3. At this point, the requested resources are available, but we <span style="color:#f77729;"><b>check</b></span> if granting the request is <span style="color:#f7007f;"><b>safe</b></span> (will not lead to deadlock).

   - Create a `deepcopy` of `available`, `allocation`, and `need`
   - Pass these new data structures to the `safety_check` algorithm, which will return `True` (safe) or `False` (unsafe)

4. If the outcome of Step(3) is:
   - `True`: <span class="orange-bold">UPDATE</span> all system states concerning Process i <span style="color:#f7007f;"><b>(request granted)</b></span>:
     - `available[i] = available[i] - request[i]` for all `i<M`
     - `need[customer_index][i] = need[customer_index][i] - request[i]` for all `i<M`
     - `allocation[customer_index][i] = allocation[customer_index][i] + request[i]` for all `i<M`
   - `False`: This means <span style="color:#f7007f;"><b>request rejected</b></span>, and process i has to try again in the future. This is because granting the request results in deadlock in the future.

### Example 1

You may run: `python3.10 banker.py test_files/q1.txt` and study the output.

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

### Example 2

You may run: `python3.10 banker.py test_files/q2.txt` and study the output.

This time round, we have `N=5` processes and `M=3` different types of resources, and `available=[10,5,7]`.

After such requests from all customers,

```
Customer 0 requesting
[0, 1, 0]
Customer 1 requesting
[2, 0, 0]
Customer 2 requesting
[3, 0, 2]
Customer 3 requesting
[2, 1, 1]
Customer 4 requesting
[0, 0, 2]
```

We should have only 3 counts of Resource A available (the first type of resource).

However here we witness Process 0 releasing 1 instance of Resource A, and hence the updated system state:

```
Customer 1 releasing
[1, 0, 0]

Current state:
Available:
[4, 3, 2]

Maximum:
[7, 5, 3]
[3, 2, 2]
[9, 0, 2]
[2, 2, 2]
[4, 3, 3]

Allocation:
[0, 1, 0]
[1, 0, 0]
[3, 0, 2]
[2, 1, 1]
[0, 0, 2]

Need:
[7, 4, 3]
[2, 2, 2]
[6, 0, 0]
[0, 1, 1]
[4, 3, 1]
```

### Task 1

`TASK 1:` Call `check_safe` algorithm inside `request_resources` in `banker.py`.
{:.task}

Right now the variable `safe` is always set to `True`, hence we <span style="color:#f77729;"><b>always grant</b></span> the request as long as the resources are available and the processes do not violate their `max`. For input `q0, q1, q2`, we are <span style="color:#f77729;"><b>lucky</b></span> because we never had any requests that might result in deadlock.

However, if you try running the program with `q3`, you will notice that the printed output is not the same as the answer: `test_files/q3_answer.txt`. We need to <span style="color:#f77729;"><b>implement and call</b></span> the `check_safe` algorithm to ensure that we get the right answer.

```
python3.10 banker.py test_files/q3.txt
```



Let's start easy. Since `check_safe` function is already defined (although not yet implemented), <span style="color:#f77729;"><b>call</b></span> `check_safe` method and assign their return value to `safe`:

```py
  # Task 1
  # TODO: Check if the state is safe or not by calling check_safe, right now it is hardcoded to True
  # 1. Perform a deepcopy of available, need, and allocation
  # 2. Call the check_safe method with new data in (1)
  # 3. Store the return value of (2) in variable safe
  # DO NOT PRINT ANYTHING ELSE

  ### BEGIN ANSWER HERE ###
  safe = True  # Change this line

  ### END OF TASK 1 ###

  if safe:
      # If it is safe, allocate the resources to customer customer_number
      for idx, req_val in enumerate(request):
          self.allocation[customer_index][idx] += req_val
          self.need[customer_index][idx] -= req_val
          self.available[idx] -= req_val

      bank_lock.release()
      return True
  else:
      bank_lock.release()
      return False
```

<span class="orange-bold">DO NOT PRINT ANYTHING ELSE</span> for submission. Please remove your debug messages.
{:.error}

## The Safety Algorithm

This algorithm is <span style="color:#f77729;"><b>also</b></span> called each time we encounter a resource request (line starting with `r`), but only if we managed to enter <span style="color:#f77729;"><b>Step 3</b></span> of the Resource Request Algorithm. The function receives five parameters and returns a `bool`:

```py
def check_safe(self, customer_index, request, work, need, allocation):
    """
    Checks if the request will leave the bank in a safe state.

    Parameters
    ----------
    work, need, allocation : list[int], list[list[int]], list[list[int]]
        deep copy of available, need, and allocation matrices
    customer_index : int
        the customer's index (0-indexed)
    request : list[int]
        an array of the requested count for each resource

    Returns
    -------
    True : if the request resources will leave the bank in a safe state
    False : otherwise
    """
```

The algorithm goes as follows:

1. Create a vector `finish: list[int]` of size `N`, initialised to be `False` for all elements in `finish`.

   - Then, <span style="color:#f77729;"><b>hypothetically</b></span> grant the current request by customer `customer_index` by updating:
     - `work[i] = work[i] - request[i]` for all `i<M`
     - `need[customer_index][i] = need[customer_index][i] - request[i]` for all `i<M`
     - `allocation[customer_index][i] = allocation[customer_index][] + request[i]` for all `i<M`
   - This request granting is _hypothetical_ because `work` is a <span style="color:#f77729;"><b>copy</b></span> of `available` (not the actual `available`). Similar argument with `need, allocation`. In reality, we haven't granted the request yet, we simply compute this hypothetical situation and decide whether it will be `safe` or `unsafe`.

2. Find an index `i` (which is a _customer_) such that:

   - `finish[i] == False` <span style="color:#f7007f;"><b>and</b></span>
   - `need[i][j] <= work[j]` for <span style="color:#f7007f;"><b>all</b></span> `j<M`.
   - The two above condition signifies that an incomplete Customer `i` can _complete_ even after this request by `customer_index` is granted

3. If such index `i` from Step 2 exists do the following, else go to Step 4.

   - Update: `work[j] = work[j] + allocation[i][j]` for <span style="color:#f7007f;"><b>all</b></span> `j<M`.
     - This signifies that a Customer `i` that can _complete_ will free its _currently_ allocated resources.
   - Update: `finish[i] = True`
   - <span style="color:#f7007f;"><b>Then, REPEAT step 2</b></span>
     > You might want to store the values of `i` each time you execute this Step 3 elsewhere to backtrack a possible **safe execution sequence**, although it is not explicitly required for this lab. Head to the [lecture notes](https://natalieagus.github.io/50005/os/deadlock#part-2-the-safety-algorithm) for a summary.  

4. If no such index `i` in Step 2 exists:
   - If `finish[i] == True` for <span style="color:#f7007f;"><b>all</b></span> `i<N`, then it means the system is in a <span style="color:#f77729;"><b>safe state</b></span>. Return `True`.
   - Else, the system is <span style="color:#f77729;"><b>not in a safe state</b></span>. Return `False`.

## Careless Mistake

<span style="color:#f77729;"><b>Common careless mistake:</b></span> A lot of people missed the “REPEAT step 2” instruction in step 3. Step 2 and 3 must be implemented in a `while` loop, as you <span style="color:#f7007f;"><b>might NOT</b></span> necessarily obtain i in <span style="color:#f77729;"><b>sequential</b></span> (increasing) order.
{:.error}

{:.new-title}
> Think!
> 
> Why is that so? Does it mean we can have a <span style="color:#f7007f;"><b>safe</b></span> execution sequence e.g: `1,0,2` for a 3-process system? Yes of course! That simply means `P1` can be executed first, then `P0`, then `P2`. Think about what `i` represents (just arbitrary naming of consumer processes), of course a safe execution sequence has nothing to do with their naming!

### Example

Run the Banker algorithm with `q3`:

```
python3.10 banker.py test_files/q3.txt
```

At first, P0 is making the request `[1,0,3]` and it is <span style="color:#f77729;"><b>granted</b></span> as shown in the `allocation` matrix:

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

Then, P1 requests for `[1,1,1]`. This is a legal request, since if the request is granted, then P1 allocation (`allocation[1]` is initially `[0,0,0]`) will still be <span style="color:#f77729;"><b>not more than</b></span> its maximum: `max[1]: [2,1,3]`.

If your implementation is <span style="color:#f77729;"><b>correct</b></span>, this should however lead to an <span style="color:#f77729;"><b>unsafe state</b></span>, and hence the request is rejected and the state of the system remain the same:

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

- In q4, the last request by P0: `[0,2,0]` leads to an unsafe state and therefore <span style="color:#f77729;"><b>rejected</b></span>.
- In q5, the request by P1: `[2,0,1]` is rejected because it leads to an <span style="color:#f77729;"><b>unsafe state</b></span>. However, after P0 release the resources it held: `[1,3,4]`, a <span style="color:#f7007f;"><b>repeated</b></span> request by P1 for the same set of resources `[2,0,1]` is granted.

### Task 2

`TASK 2:` Implement the safety check algorithm inside `check_safe` function in `banker.py`:
{:.task}

```py
def check_safe(self, customer_index, request, available, need, allocation):
  """
  Checks if the request will leave the bank in a safe state.

  Parameters
  ----------
  available, need, allocation : list[list[int]]
      deep copy of available, need, and allocation matrices
  customer_index : int
      the customer's index (0-indexed)
  request : list[int]
      an array of the requested count for each resource

  Returns
  -------
  True : if the request resources will leave the bank in a safe state
  False : otherwise
  """
  bank_lock.acquire()

  # TASK 2
  # TODO: Check if the state is safe
  # 1. Create finish list[int] of length self.N
  #    Then, hypothetically grant the current request by updating:
  #       1. work[i] = work[i] - request[i] for all i<M
  #       2. need[customer_index][i] = need[customer_index][i] - request[i] for all i<M
  #       3. allocation[customer_index][i] = allocation[customer_index][i] + request[i] for all i<M
  # 2. Find index i such that both finish[i] == False, need[i] <= work
  # 3. If such index in (2) exists, update work += allocation[i], finish[i] = True
  # 4. REPEAT step (2 & 3) until no such i exists
  # 5. If no such i exists anymore, and finish[i] == True for all i, set safe = True
  # 6. Otherwise, set safe = False
  # DO NOT PRINT ANYTHING ELSE

  ### BEGIN ANSWER HERE ###
  safe = True  # Change this line according to whether the request will be safe or not

  ### END OF TASK 2 ###

  bank_lock.release()
  return safe
```

Again, do <span class="orange-bold">NOT</span> print any other stuffs as part of your answer when submitting the files.
{:.error}

## Further Notes

### Rejecting Requests

If a resource request is <span style="color:#f77729;"><b>rejected</b></span>, dont panic, it's not the end of the world. The process/consumer just need to <span style="color:#f77729;"><b>try</b></span> to request it again in the future.

How can this be implemented? Usually schedulers are programmed to tackle this kind of cases; e.g: they can be placed to a special queue and will periodically prompt for resource request until granted, or there exists some kind of event-driven solution -- it's free-for-all to implement.

### Resource Release Caveat

We also assume that (for simplicity of this lab) a process will <span style="color:#f77729;"><b>not</b></span> release more than what has been allocated to them (that the value of `release` is valid). We wrote this detail under `release_resources` function:

```py
def release_resources(self, customer_index, release):
    """
    Releases resources borrowed by a customer. Assume release is valid for simplicity.

    Parameters
    ----------
    customer_index : int
        the customer's index (0-indexed)
    release : list[int]
        an array of the release count for each resource

    Returns
    -------
    None
    """
    print(f"Customer {customer_index} releasing\n{release}")

    bank_lock.acquire()
    # Release the resources from customer customer_number
    for idx, val in enumerate(release):
        self.allocation[customer_index][idx] -= val
        self.need[customer_index][idx] += val
        self.available[idx] += val
    bank_lock.release()
```

### Synchronisation

Finally, since `max, allocation, need` and `available` are shared data structures among all these methods, we protect each method that modifies these values using a <span style="color:#f77729;"><b>reentrant lock</b></span>: `bank_lock = threading.RLock()`.

We guard the <span style="color:#f77729;"><b>critical sections</b></span> with `bank_lock.acquire()` and `bank_lock.release()` to make it <span style="color:#f77729;"><b>thread safe</b></span>.

When you're confident that your Task 1 and Task 2 are correct, you can run the tester file:

```
python3.10 banker_test.py
```

This will run your `banker.py` against all 6 test cases: q0 to q5. If all goes well, the following message will be printed:
<img src="{{ site.baseurl }}/assets/images/lab3/2.png"  class="center_full no-invert"/>

The tester file runs on Windows as well as of 14/06/2023.
{:.info}

It performs a simple output string matching (which is how our autograder works). Therefore it is <span style="color:#f7007f;"><b>crucial</b></span> for you not to print anything else. If you need to debug, <span style="color:#f7007f;"><b>use a debugger</b></span>. Be a proper programmer, it's never too late to start today 🥳.





# Summary 

In this lab, you have implemented the Banker's Algorithm using Python. 

The Banker's Algorithm is used in operating systems to <span class="orange-bold">avoid</span> deadlocks by safely allocating resources to processes. It's a deadlock **avoidance** technique that checks whether granting a resource request will keep the system in a safe state: i.e., whether all processes can still eventually complete. It’s mainly taught as a *theoretical model and rarely used in modern OS kernels* due to its high runtime complexity and need for advanced knowledge of future resource requests. However, it serves as a **foundation** for understanding resource allocation, particularly in systems where resource usage patterns are predictable, such as embedded or real-time systems with strict guarantees.


**The Banker's Algorithm is not used in modern general-purpose operating systems** like Linux, Windows, or macOS. It's mostly a teaching tool or used in **very controlled environments** like embedded or real-time systems where all resource requirements are known in advance. In practice, OSes avoid deadlocks using a combination of:

1. **Deadlock prevention**: Structuring the system to make deadlocks impossible (e.g., by enforcing **resource ordering**).
2. **Deadlock avoidance (rare)**: Like Banker's, but impractical due to overhead.
3. **Deadlock detection and recovery**: Allowing deadlocks to happen, then detecting and breaking them (e.g., killing a process).
4. **Ostrich algorithm**: Ignoring deadlocks entirely if they are rare and impact is low (common in general OSes).

So in modern OSes, deadlocks are often just **detected (not prevented)** using techniques like wait-for graphs, especially in database engines or advanced resource managers.

# Appendix

### How to migrate .zip file between VM and Host OS?

Working on a VM and don't know how to migrate your .zip file out to your host OS? You can choose the <span style="color:#f77729;"><b>hard way</b></span> like login to your email account in your VM and then email yourself the zipfile. Or, login to Telegram in your VM.

If you're using VirtualBox, you can also enable shared clipboard:
<img src="{{ site.baseurl }}/assets/images/lab3/5.png"  class="center_seventy no-invert"/>

But why do that? Why do that if you're a CS student? Use `ssh`!
{:.info}

### ssh

You can enable `ssh` at your VM, [here](https://dev.to/developertharun/easy-way-to-ssh-into-virtualbox-machine-any-os-just-x-steps-5d9i) is a guide if you're using VirtualBox. Or, you can simply Google "ssh to your-virtual-machine" and follow simple steps there. Then, enable ssh on `VSCode`: install the `ssh` extension and connect! You can follow the guide [here](https://code.visualstudio.com/docs/remote/ssh-tutorial) just this part:

<img src="{{ site.baseurl }}/assets/images/lab3/4.png"  class="center_seventy"/>

Then, type in your VM address when prompted in this format: `ssh -p <port_number> <username>@<local_ip>`, for example:

```
ssh -p 3022 ubuntu@192.168.55.2
```

You will need to then enter your password. Then click `Open Folder` and select your Home folder. You will have a nice interface with VSCode now, connected to your VM. Drag and drop to transfer files between your host OS and your VM. No lag, nice terminal, can utilise VSCode functionalities.. Doesn't it spark joy?
<img src="{{ site.baseurl }}/assets/images/lab3/6.png"  class="center_full no-invert"/>

