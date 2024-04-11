---
layout: default
permalink: /pa1/part3
title: Builtin Commands 
description: Implement more shell builtin commands
parent: Programming Assignment 1
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

# Shell Builtin Commands 
{: .no_toc}

Your first task is to implement more shell builtin commands. Right now, only `exit` is considered a builtin command:

```c
    // If the command is "exit", break out of the loop to terminate the shell
    if (strcmp(cmd[0], "exit") == 0)
      break;
```

All other commands should trigger a `fork` and `execvp` of a system program in `[PROJECT_DIR]/bin` whose name matches the first word of the command. 
{:.important}

# Expand the Builtin Commands 

{:.task}
The shell must be expanded support the following 7 builtin commands: `cd, help, exit, usage, env, setenv, unsetenv`.  

## List the builtin commands 

We suggest that you declare them as an array of `char*` in `shell.h`: 

```c 
const char *builtin_commands[] = {
    "cd",    // Changes the current directory of the shell to the specified path. If no path is given, it defaults to the user's home directory.
    "help",  //  List all builtin commands in the shell
    "exit",  // Exits the shell
    "usage", // Provides a brief usage guide for the shell and its built-in command
    "env", // Lists all the environment variables currently set in the shell
    "setenv", // Sets or modifies an environment variable for this shell session
    "unsetenv" // Removes an environment variable from the shell
    };
```

{:.note-title}
> An array of character pointers
> 
> `const char *builtin_commands[]` declares an array of character pointers where each pointer is pointing to a constant character string. 


## Builtin command handler
It is recommended that you refactor the shell code such that each builtin command will call a specific handler function, instead of writing a gigantic `if-else` string matching clause in the `main`. You can do this elegantly in c by first declaring each handler in the header file:


```c
/*
Handler of each shell builtin function
*/
int shell_cd(char **args);
int shell_help(char **args);
int shell_exit(char **args);
int shell_usage(char **args);
int list_env(char **args);
int set_env_var(char **args);
int unset_env_var(char **args);
``` 

{:.important}
Ensure they have the same signature. 

Then declare an array of functions as follows. 

```c
/*** This is array of functions, with argument char ***/
int (*builtin_command_func[])(char **) = {
    &shell_cd,     // builtin_command_func[0]: cd 
    &shell_help,   // builtin_command_func[1]: help
    &shell_exit,   // builtin_command_func[2]: exit
    &shell_usage,  // builtin_command_func[3]: usage
    &list_env,     // builtin_command_func[4]: env
    &set_env_var,  // builtin_command_func[5]: setenv
    &unset_env_var // builtin_command_func[6]: unsetenv
};
```

{:.note-title}
> An array of function pointers
> 
> This declaration creates an array of function pointers named builtin_command_func. Each element in the array is a pointer to a function that takes a single argument (char **) and returns an int. The functions pointed to by this array are intended to implement the functionality for each of the built-in commands supported by your shell.


## Using the function pointers
The idea is that each builtin command has an <span class="orange-bold">index</span>. When we process command from the user, we shall match whether any of the commands given matches any element in `builtin_commands`. If so, we will call the corresponding builtin command handler, here's a rough idea:

```c   
    // Helper function to figure out how many builtin commands are supported by the shell
    int num_builtin_functions()
    {
        return sizeof(builtin_commands) / sizeof(char *);
    };

    // Loop through our command list and check if the commands exist in the builtin command list
    for (int command_index = 0; command_index < num_builtin_functions(); command_index++)
    {
        if (strcmp(args[0], builtin_commands[command_index]) == 0) // Assume args[0] contains the first word of the command
        {
        // We will create new process to run the function with the specific command except for builtin commands.
        // These have to be done by the shell process. 
        return (*builtin_command_func[command_index])(args);
        }
    }

```

{:.note-title}
> About `strcmp`
> 
> In C, the `strcmp` function compares two strings and returns an integer value based on the outcome of the comparison. <span class="orange-bold">If the strings are equal, strcmp returns `0`.</span>

The expression `(*builtin_command_func[command_index])(args)` is a way to call a function through a pointer obtained from an array of function pointers, using a specific index (`command_index`) to select which function to call, and passing it an array of strings (`args`) as arguments. 

Here's a breakdown of the expression.

#### Array of function pointers
`builtin_command_func` is an <span class="orange-bold">array of pointers</span> to functions. Each element of this array is a pointer to a function that matches a specific prototype: the functions take a single parameter of type `char **` and return an `int`.

#### Selecting a function pointer
`builtin_command_func[command_index]` accesses the element at <span class="orange-bold">position</span> `command_index` within the array `builtin_command_func`. This element is a <span class="orange-bold">pointer</span> to a function. The value of `command_index` determines which function pointer is selected, based on the command the user has input. 

#### Dereferencing the function pointer

`*builtin_command_func[command_index]` <span class="orange-bold">dereferences</span> the function pointer obtained in the previous step. 

Actually, in C, when you have a function pointer, *you don't actually need to explicitly dereference it* with `*` to call the function it points to. Both `(*func)(args)` and `func(args)` are valid and equivalent when `func` is a function pointer. The dereferencing here is more about clarity and emphasis, showing that we are indeed dealing with a pointer.
{:.note}

#### Calling the handler function

`(*builtin_command_func[command_index])(args)` <span class="orange-bold">calls</span> the function pointed to by the selected element in the `builtin_command_func` array, passing `args` as its argument. `args` is expected to be an array of strings (char pointers), which aligns with the expected parameter type of the functions being pointed to. 

The whole expression invokes the function just like a normal function call, but the function being called is selected at runtime based on the value of `command_index`.

{:.highlight-title}
> Summary
> 
> In summary, this mechanism allows for a <span class="orange-bold">flexible</span> and <span class="orange-bold">dynamic</span> dispatch of function calls based on user input or other runtime conditions. 
> 
> It's a common pattern in C for implementing simple <span class="orange-bold">polymorphism</span> or <span class="orange-bold">callback</span> functions, enabling the selection and invocation of different functions without using if-else or switch-case statements directly on the `commandIndex`. This approach is especially useful in command line interpreters or shells, where the set of commands and their corresponding functions can be neatly organized in arrays.

# Sample Output and Implementation Notes

{:.note}
You may decorate your shell prompt in any way you like. 

### cd
Change the current working directory of the shell. You can print out your current working directory as part of the shell's prompt, or you can use the `ld` command to list the current directory. 

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/03-builtin-commands/2024-04-10-18-43-05.png"  class="center_full no-invert"/>

### help 

Print out all builtin commands in the shell. 

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/03-builtin-commands/2024-04-10-18-33-59.png"  class="center_full no-invert"/>

### usage 

Print a brief description on how to use each builtin command. Also prints a useful message if the command given is not part of CSEShell's builtin command.

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/03-builtin-commands/2024-04-10-18-38-36.png"  class="center_full no-invert"/>

### env 

Print all environment variables of this shell. This should inherit your system's environment variables too. 

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/03-builtin-commands/2024-04-10-18-54-50.png"  class="center_full no-invert"/>


In C programming, environment variables can be accessed using the external variable `environ`. You may refer to this code snippet to get started: 

```c 
// This program will list all the environment variables available to it at runtime
#include <stdio.h>

// Declaration of the external variable 'environ'
// environ is actually defined in the C standard library
extern char **environ; 

int main() {
    char **env = environ; // Pointer to the array of environment strings
    
    while (*env) { // Loop until NULL pointer is encountered
        printf("%s\n", *env); // Print the current environment variable
        env++; // Move to the next environment variable
    }
    
    return 0;
}

```

{:.new-title}
> `environ` 
>
> The `environ` variable in C is a pointer to an array of strings that represent the environment variables for the current process

### setenv 

The command `setenv KEY=VALUE` simply adds to the list of this process' environment variables. It should not print anything:

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/03-builtin-commands/2024-04-10-18-59-38.png"  class="center_full no-invert"/>

However, you can check that it's indeed registered to this process' (the shell) environment variables by typing `env`. Your newly set env variable should be visible at the bottom of the list:

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/03-builtin-commands/2024-04-10-19-00-56.png"  class="center_full no-invert"/>

{:.important}
It's important <span class="orange-bold">not</span> to attempt to remove an environment variable by directly modifying the `environ` global variable or the strings to which it points. The correct way to modify the environment is through the use of functions like `setenv()`, `unsetenv()`, and `putenv()`.

### unsetenv 

This command `unset KEY` should delete any environment variable whose `KEY` matches any existing environment variables. You don't have to do anything if you attempt to unset an environment variable that doesn't exist. 

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/03-builtin-commands/2024-04-10-19-02-30.png"  class="center_full no-invert"/>

{:.note}
If you attempt to unset an environment variable that does not exist using `unsetenv()`, the function is considered to **succeed**, and it returns `0`. The absence of the specified environment variable means there's nothing to remove, which aligns with the desired outcome of `unsetenv()`: ensuring that the environment variable is not present in the environment of the current process.

### exit 

As it originally was, `exit` should gracefully quit the shell. 

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/03-builtin-commands/2024-04-10-19-05-56.png"  class="center_full no-invert"/>

