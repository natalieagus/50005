---
layout: default
permalink: /pa1/part1
title: Running the starter shell
description: Introduction about running the starter code 
parent: Programming Assignment 1
grand_parent: Programming Assignment
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
**Natalie Agus (Summer 2024)**

# CSEShell Starter Code  
{: .no_toc}

CSEShell is a simple, custom shell for Unix-based systems, designed to provide an interface for executing system programs. This project includes a basic shell framework, a set of system programs (`find`, `ld`, `ldr`), and some test files. 

It incorporates a **simple** prompt display mechanism and the ability to `exit` the shell. However, it lacks the typical continuous loop for reading and processing commands, as well as the process forking logic (`fork`) that is commonly used in shell implementations to execute commands in separate processes.

## Usage Explanation 
### Directory Structure

The project is organized as follows:

- `bin/` - Contains compiled executables for system programs.
  - `find` - Program to find files.
  - `ld` - Program for listing the contents of the current directory.
  - `ldr` - Program for listing the contents of the current directory recursively.
- `cseshell` - The main executable for the CSEShell.
- `files/` - Contains various test files used with the shell and system programs.
  - `combined.txt`, `file1.txt`, `file2.txt`, ... - Test text files.
  - `notes.pdf` - A PDF file for testing.
  - `ss.png` - An image file.
- `makefile` - Makefile for building the CSEShell and system programs.
- `source/` - Source code for the shell and system programs.
  - `shell.c` and `shell.h` - Source and header files for the shell.
  - `system_programs/` - Source code and header for the system programs.

### Building the Project

To build the CSEShell and system programs, run the following command in the root project directory:

```bash
make
```

This will compile the source code and place the executable files in the appropriate directories.

### Running CSEShell

After building, you can start the shell by running:

```bash
./cseshell
```

From there, you can execute built-in commands and any of the included system programs (e.g., `find`, `ld`, `ldr`).

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/01-intro/2024-04-10-12-39-49.png"  class="center_full no-invert"/>

You can type `exit` to exit the shell:

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/01-intro/2024-04-10-12-39-32.png"  class="center_full no-invert"/>

As the starter code only contains basic shell framework, it lacks the typical continuous loop for reading an processing commaands. The shell will terminate each time you enter a command. To type another command, you need to run `./cseshell` again. 

{:.note}
Note that only system programs at `./bin` is currently **accessible** by the shell, and hence only these three commands: `find`, `ld`, `ldr` are supported. 

If you try to type any command that's commonly available on your system's shell, such as `pwd`, you will be met with this error message: 

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/01-intro/2024-04-10-12-42-50.png"  class="center_seventy"/>

### System Programs

- `find.c` - Searches for files in a directory.
- `ld.c` - List the contents of a directory.
- `ldr.c` - List the contents of a directory recursively.

Each program can be executed from the CSEShell once it is running.

### Files Directory

The `files/` directory contains various text, PDF, and image files for testing the functionality of the CSEShell and its system programs.

## Makefile

The Makefile contains rules for compiling the shell and system programs. You can clean the build by running:

```bash
make clean
```

### Source Directory

Contains all the necessary source code for the shell and system programs. It is divided into the shell implementation (`shell.c`, `shell.h`) and system programs (`system_programs/`).

## Detailed Explanation of `shell.c`

`shell.c` contains a <span class="orange-bold">basic</span> implementation of a shell that **reads** commands from the user, **parses** them, and **executes** them as processes. It consists of three main functions (`read_command`, `type_prompt`, and `main`). Below is an expanded documentation for each part of the code, including both the existing comments and additional explanations where needed.

### Header and Global Definitions
```c
#include "shell.h"
```

{:.new-title}
> Purpose
> 
> Includes the header file `shell.h` that presumably contains necessary constants (like `MAX_LINE` and `MAX_ARGS`) and function declarations related to the shell implementation.

### Function: `read_command`
```c
void read_command(char **cmd)
```

{:.new-title}
> Purpose
> 
> Reads a single command from the standard input (stdin), parses it into arguments, and stores the result in the provided `cmd` array.

{:.highlight-title}
> Parameters
> 
> `char **cmd` - A pointer to an array of strings, where the command and its arguments will be stored.

**Functionality**:
  - Reads characters from stdin until a newline (`'\n'`) or the maximum line length (`MAX_LINE`) is reached.
  - If the command is too long, it prints an error message and exits.
  - Parses the read line into words using `strtok` and stores each word in a dynamically allocated array.
  - Copies the parsed words into the provided `cmd` array, ensuring it's NULL-terminated.

### Function: `type_prompt`
```c
void type_prompt()
```

{:.new-title}
> Purpose
> 
> Displays the shell prompt to the user.

**Functionality**:
  - If it's the first time the function is called, clears the screen.
  - Prints the prompt (`$$ `) to the standard output (stdout).

### Function: `main`
```c
int main(void)
```

{:.new-title}
> Purpose
> 
> Implements the main functionality of the shell. It displays a prompt, reads and parses a command, and then executes it.



### Error Handling and System Calls


Proper error checking is performed for critical operations like reading from stdin and executing commands using `execv` system calls. It uses `exit(1)` to terminate the program upon encountering fatal errors.
```c

    // in read_command(char** cmd)
    // If the command exceeds the maximum length, print an error and exit
    if (count >= MAX_LINE)
    {
      printf("Command is too long, unable to process\n");
      exit(1);
    }
    ...

    // in main()
    execv(fullPath, cmd);

    // If execv returns, command execution has failed
    printf("Command %s not found\n", cmd[0]);
    exit(0);

```

### Memory Management
Dynamically allocates memory for each argument parsed in `read_command` using `strdup`. It's important to free this memory at the end of `main` to prevent memory leaks.
```c
  // Free the allocated memory for the command arguments before exiting
  for (int i = 0; cmd[i] != NULL; i++)
  {
    free(cmd[i]);
  }
```

### Portability
Uses preprocessor directives to clear the screen in a way that is compatible with both Windows (`cls`) and Unix-like systems (`clear`).

```c
#ifdef _WIN32
    system("cls"); // Windows command to clear screen
#else
    system("clear"); // UNIX/Linux command to clear screen
#endif
    first_time = 0;
```



