---
layout: default
permalink: /pa1/part4
title: The .rc File
description: Implement shell .rc file processing
parent: Programming Assignment 1
grand_parent: Programming Assignment
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

# Shell `.rc` file
{: .no_toc}

{:.new-title}
> The "rc" file 
> 
> A shell "rc" file, where "rc" stands for "run commands," is a script that runs every time a shell starts. 


These files are used to <span class="orange-bold">configure</span> the shell **environment**, including setting environment variables, aliases, and functions, as well as<span class="orange-bold"> running commands and scripts</span> that should execute with every new shell instance. The specific rc file and its location can vary depending on the shell and the operating system. 

Here are some of the common shell rc files:

1. `~/.bashrc`: The bashrc file is executed for interactive non-login shells. On Linux systems, when you open a new terminal window, the commands in this file run. For macOS, bash was the default shell until Catalina, after which zsh became the default.
2. `~/.zshrc`: This is the configuration file for Zsh, executed for interactive shells. It's similar to `~/.bashrc` but for Zsh.
3. `~/.bash_profile` or `~/.profile` or `~/.bash_login`: These files are executed for login shells (e.g., when you log in from the console or remotely). Typically, `~/.bash_profile` will source `~/.bashrc` to ensure consistent behavior between login and non-login shells.
4. `~/.vimrc`: Allows users to customize Vim, a highly configurable CLI text editor to their liking, automating many aspects, setting global preferences, and including custom commands and key mappings.

{:.task}
For this part of the assignment,  we're tasked with implementing functionality to read and interpret a configuration file named `.cseshellrc `. This file should be placed in the root project directory.


This functionality <span class="orange-bold">enhances</span> the shell by providing a simple but powerful way to <span class="orange-bold">automate</span> startup tasks and configure the shell environment automatically upon launch, making it more flexible and personalized to the user's needs.


# `.cseshellrc` 

This `.cseshellrc` file allows for <span class="orange-bold">initial</span> setup actions to be performed <span class="orange-bold">automatically</span> each time your custom shell starts. The requirements for processing `.cseshellrc` are straightforward and involve two main actions based on the content of each line in the file: setting `PATH`, and executing commands. 


### Setting the PATH Environment Variable

If a line in .cseshellrc starts with the keyword PATH, the rest of the line should be interpreted as the value for the PATH environment variable. This involves <span class="orange-bold">setting</span> the PATH environment variable of your shell process to the specified value in `.cseshellrc`. The PATH variable is crucial as it tells the shell where to look for the executables corresponding to the command names that are entered. 

With this, we are able to execute any other system program in the system, and not only those at `[PROJECT_DIR]/bin`. 
{:.note}

### Executing Commands
For all other lines that do not start with "PATH", the shell should treat these lines as commands to be executed. Each line represents a separate command that should be run by the shell, just as if it were typed in by the user at the command prompt. You will still need to execute this command in the child process. This allows for automatic execution of a series of commands at the startup of your shell, enabling tasks such as setting aliases, defining functions, or performing initial setup actions.



{:.new-title}
> Tips 
>
> The processing of `.cseshellrc` should be carried out before the user is prompted for the <span class="orange-bold">first</span> time. Additionally, it should be done only once at the very beginning of the cseshell process.

# Sample Output and Implementation Notes

Given the following `.cseshellrc`:

```sh
PATH=/Users/natalie_agus/Desktop/pa1-2024/bin:/usr/bin
clear
cal
```

Your shell must be able to process it the first time it launches: 

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/04-rc-file/2024-04-11-10-04-33.png"  class="center_full no-invert"/>

Notice how the console is **cleared**, and calendar is printed out before the first prompt is presented. We are also able to access system program `clear` residing at `/usr/bin`, which indicates that the `PATH` is registered correctly. We can also check that using the `env` command implemented earlier and inspect the value of the PATH variable. 
<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/04-rc-file/2024-04-11-11-15-00.png"  class="center_seventy no-invert"/>

If `.cseshellrc` contains invalid commands or empty spaces:

```sh 
PATH=/Users/natalie_agus/Desktop/pa1-2024/bin:/usr/bin
clear
test
something
else 


cal
```

It should still be able to execute the valid commands, while indicating that the previous commands don't exist. 

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/04-rc-file/2024-04-11-10-21-01.png"  class="center_full no-invert"/>

{:.important}
Afterwards, your shell **must** also be able to execute any system program set in `PATH` using `.cseshellrc` at all times, even when we `cd`. Failure to do this results in -1% <span class="orange-bold">penalty</span>.

## Use `execvp` instead of `execv` 

`execv` and `execvp` are two of the exec family functions in Unix-like operating systems, which are used to **replace** the current process image with a new process image. These functions are typically used in system programming to execute external programs from within a C program. However,   `execv` requires a full path, while `execvp` can work with just the program name or a relative path, <span class="orange-bold">automatically searching the PATH environment variable</span>. 

```c
// using execvp 
#include <stdio.h>
#include <unistd.h>

int main() {
    // Arguments array
    char *args[] = {"ls", "-l", NULL}; // Lists in long format

    // Execute command
    execvp(args[0], args);

    // If execvp returns, it must have failed
    perror("execvp failed");
    return 1;
}
```

```c
// using execv
#include <stdio.h>
#include <unistd.h>

int main() {
    // Arguments array
    char *args[] = {"ls", "-l", NULL}; // Lists in long format

    // Execute command - assuming the full path to 'ls' is known
    execv("/bin/ls", args);

    // If execv returns, it must have failed
    perror("execv failed");
    return 1;
}

```

Therefore, all you need to do while processing the .rc file is to set the PATH value as the environment variable for the process. You don't have to *append* anything, simply replace the existing `PATH` with the new `PATH` specified in the .rc file. 

## Use `_exit` instead of `exit` in Child Process

{:.important}
In the child process, if `execvp` or `execv` fails, you <span class="orange-bold">should</span> use `_exit(0)` instead of `exit(0)` to terminate the child process so that it does not interfere with parent process' resources.

`_exit()` is a system call that terminates the process immediately, <span class="orange-bold">without performing any clean-up</span> operations mentioned above. It ensures that the child exits quickly, without interfering with any shared resources (like standard I/O buffers) that the parent process might be using. 

It is likely that you would open the `.rc` file with `fopen` and then read its content line by line using `fgets` or `getline`, and then immediately process each line by `fork` and executing the line as a command. However, it is likely that you will meet an infinite loop as you combine `fgets` or `getline` with `fork`.  [Read this post here if you're interested](https://stackoverflow.com/questions/50110992/why-does-forking-my-process-cause-the-file-to-be-read-infinitely). You can also try this compiling and running this program: 

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/wait.h>
#include <unistd.h>

#define MAX_LINE_LENGTH 1024

int main()
{
    const char *filePath = "[PATH TO YOUR TEXT FILE]"; // Specify the path to your text file

    FILE *file = fopen(filePath, "r");
    if (file == NULL)
    {
        perror("Failed to open file");
        return EXIT_FAILURE;
    }

    char line[MAX_LINE_LENGTH];
    while (fgets(line, sizeof(line), file))
    {
        printf("Line [%s]\n", line); // Print the line read from the file
                                     // fflush(0);
        pid_t pid = fork();
        if (pid == -1)
        {
            // If fork() returns -1, an error occurred
            perror("fork failed");
            exit(EXIT_FAILURE);
        }
        else if (pid == 0)
        {
            // Child process
            printf("Hello from child\n");
            exit(EXIT_SUCCESS); // Exit child process, should've used _exit(EXIT_SUCCESS) instead
        }
        else
        {
            // Parent process
            int status;
            waitpid(pid, &status, 0); // Wait for child process to finish
        }
    }

    fclose(file); // Close the file
    return EXIT_SUCCESS;
}

```

Text file data, e.g `./test.txt`:
```
hello
from
the 
other 
side


```

It's likely that this is your output (you'd have to terminate the program with ctrl+c):

<img src="{{ site.baseurl }}/docs/Programming Assignment/pa1/images/04-rc-file/2024-04-11 11.08.26.jpg"  class="center_fifty no-invert"/>

{:.highlight-title}
> The Cause: Shared file descriptor and resources between parent and child
>
> Open file descriptors, along with other resources are <span class="orange-bold">inherited</span> (shared) between parent and child. To this end, file descriptors need very careful handling around `fork()`. If you use `exit()` in the child process when `execvp` fails to execute (and therefore the child process hasn't been replaced by a new process), `exit`  will perform clean-up operations before terminating the process. *These operations include flushing buffered data to output streams, closing all open file descriptors, and executing functions registered with `atexit`.*

{:.important}
You <span class="orange-bold">should use `_exit`</span> (or its synonym `_Exit`) to <span class="orange-bold">abort</span> the child program when the `execv` fails, because in this situation, the child process may interfere with the parent process' external data (files) by calling its atexit handlers, calling its signal handlers, and/or flushing buffers.

For the same reason, you should also use `_exit` in any child process that does not do an exec, but those are rare. 

{:.note-title}
> When to use `exit()`?
> 
> Typically, `exit()` is used in the parent process because the clean-up operations are necessary to ensure a clean and orderly shutdown of the program.
