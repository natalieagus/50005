---
layout: default
permalink: /pa1/part2
title: Shell Loop and fork()
description: Implement basic shell loop functionality using `fork()`
parent: Programming Assignment 1
grand_parent: Programming Assignment
nav_order:  2
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

# Shell Loop and `fork()`
{: .no_toc}

{:.task}
<span class="orange-bold">Expand</span> the provided basic shell starter code into a more fully-featured shell that can handle multiple commands in a session and execute commands in <span class="orange-bold">separate processes</span> without terminating after a single command. This feature <span class="orange-bold">must be present</span> for us to test other functionalities. Without this functionality, you will obtain <span class="orange-bold">zero marks</span> for your PA1.

You need to implement a few key features: a continuous loop to read commands, forking to create child processes for command execution, and proper process management. 

## Modify `main()` to Implement Shell Functionality

The `main()` function should have the following **functionality**:
- Enters an <span class="orange-bold">infinite</span> loop where it calls `type_prompt` to display the shell prompt and `read_command` to read and parse a command from the user.
- Skips execution if the command is empty.
- Exits the loop (and the shell) if the command is "exit".
- For other commands, it attempts to execute them by first forking the shell process using `fork()` then using `execv` in the child process to replace its image with the one specified by the command path. 
- The parent process (shell) should **wait** (block) for the child to complete before continuing using [`waitpid`](https://linux.die.net/man/2/waitpid), and inspect child process' exit status
- Handles process creation errors and command not found errors.
- Cleans up by freeing allocated memory for the command arguments.

{.error}
Your shell should <span class="orange-bold">never</span> abruptly terminate, even when we give commands that don't exist or simply pressing enter multiple times. Your shell should also be able to execute commands as-given, it should <span class="orange-bold">not</span> accidentally access garbage input values from uncleared buffers in any way. Failure to do this results in 1% grade penalty. 


### Tips for Cleaning up 

`strdup` in `read_command` will **allocate** a new memory location for your new string. This location is stored at `char** cmd`. It is essential to **free** this for the next loop of prompt. At the end of your shell `main()` function you should do the following before looping to get another prompt from the user:

```cpp 
  // Free the allocated memory for the command arguments before exiting
  for (int i = 0; cmd[i] != NULL; i++)
  {
    free(cmd[i]);
  }
```




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

If you implement the shell's loop properly, you should be able to enter various commands (including commands that don't exist) in succession without having to spawn the shell again:

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/02-shell-loop/2024-04-10-18-07-51.png"  class="center_full no-invert"/>

Your shell will only exit if you type `exit`:

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/02-shell-loop/2024-04-10-18-08-30.png"  class="center_full no-invert"/>

