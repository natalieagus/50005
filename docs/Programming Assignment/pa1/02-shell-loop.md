---
layout: default
permalink: /pa1/part2
title: Shell Loop and fork()
description: Implement basic shell loop functionality using `fork()`
grand_parent: Programming Assignment
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
**Natalie Agus (Summer 2024)**

# Shell Loop and `fork() 
{: .no_toc}

{:.task}
<span class="orange-bold">Expand</span> the provided basic shell starter code into a more fully-featured shell that can handle multiple commands in a session and execute commands in separate processes without terminating after a single command. 

You need to implement a few key features: a continuous loop to read commands, forking to create child processes for command execution, and proper process management. 

## Modify `main*()`

The `main()` function should have the following **functionality**:
- Enters an <span class="orange-bold">infinite</span> loop where it calls `type_prompt` to display the shell prompt and `read_command` to read and parse a command from the user.
- Skips execution if the command is empty.
- Exits the loop (and the shell) if the command is "exit".
- For other commands, it attempts to execute them by first forking the shell process then using `execv` in the child process to replace its image with the one specified by the command path. 
- The parent process (shell) should **wait** (block) for the child to complete before continuing using [`waitpid`](https://linux.die.net/man/2/waitpid), and inspect child process' exit status
- Handles process creation errors and command not found errors.
- Cleans up by freeing allocated memory for the command arguments.


## Inspect Child Process Exit Status 

It is beneficial to inspect the Exit Status of the child process. We check if the child process terminated normally, and if so, retrieves the exit status of the child process.

This allows the parent process to take appropriate action based on how the child process terminated, such as reporting the exit status or handling abnormal terminations.

You can inspect the child process' exit status by passing a pointer to an integer variable `&status` which `waitpid` can use to store the exit status of the target process:

```c  
    int status, child_exit_status;

    waitpid(pid, &status, WUNTRACED);
    // if child terminates properly,
    if (WIFEXITED(status))
    {
        child_exit_status = WEXITSTATUS(status);
    }

    // checks child_exit_status and do something about it
```

{:.note}
After `waitpid()` returns, the macro `WIFEXITED(status)` can be used to check if the child process terminated normally (e.g., by calling `exit()` or returning from `main()`). If ` ` returns true, it means the child terminated **normally**.

# Expected Output

If you implement the shell's loop properly, you should be able to enter various commands in succession without having to spawn the shell again:

<img src="{{ site.baseurl }}/docs/Programming Assignment/pa1/images/01-intro/2024-04-10-10-28-40.png"  class="center_seventy no-invert"/>

Your shell will only exit if you type `exit`:

<img src="{{ site.baseurl }}/docs/Programming Assignment/pa1/images/01-intro/2024-04-10-10-29-29.png"  class="center_seventy no-invert"/>



