---
layout: default
permalink: /pa1/part4
title: The .rc File
description: Implement more shell .rc file 
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
**Natalie Agus (Summer 2024)**

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

This `.cseshellrc` file allows for <span class="orange-bold">initial</span> setup actions to be performed <span class="orange-bold">automatically</span> each time your custom shell starts. The requirements for processing `.cseshellrc` are straightforward and involve two main actions based on the content of each line in the file:
1. <span class="orange-bold">Setting the PATH Environment Variable</span>: If a line in .cseshellrc starts with the keyword `PATH`, the rest of the line should be interpreted as the value for the PATH environment variable. This involves <span class="orange-bold">setting</span> the PATH environment variable of your shell process to the specified value in `.cseshellrc`. The PATH variable is crucial as it tells the shell where to look for the executables corresponding to the command names that are entered.
2. <span class="orange-bold">Executing Commands</span>: For all other lines that do not start with "PATH", the shell should treat these lines as commands to be executed. Each line represents a separate command that should be run by the shell, just as if it were typed in by the user at the command prompt. You will still need to execute this command in the child process. This allows for automatic execution of a series of commands at the startup of your shell, enabling tasks such as setting aliases, defining functions, or performing initial setup actions.



{:.new-title}
> Tips 
>
> The processing of `.cseshellrc` should be carried out before the user is prompted for the <span class="orange-bold">first</span> time. Additionally, it should be done only once at the very beginning of the cseshell process.



