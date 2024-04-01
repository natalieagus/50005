---
layout: default
permalink: /labs/part1
title: Lab 1 - Command Line Interface
description: Introduction to the CLI 
parent: Labs
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

# Introduction to the Command Line Interface 
{: .no_toc}

An operating system Kernel’s job is to provide **services** so that softwares can communicate with hardware. It manages computer system hardware, memory, and processes (among all others).  Its services are exposed via the system call interface, which in itself is wrapped in C standard Library to provide an API that user applications can interact with. Various user applications are made so that the OS services are usable to humans. These services are those you encounter daily when using a computer, for example: 
* File management (rename, create, list files, delete, etc), 
* Process management (run and terminate), 
* System diagnostics (RAM used, CPU %, disk space), 
* I/O operations like printing, reading from disk, communication via network, resource management (overclock speed, VM size), 
* Protection (security and permission settings), etc. 

# The Shell

A **shell** is one of these user applications that acts as an interface to allow users to **access** OS services. It is named shell because it is seen as an *“outer”* layer around the OS kernel. OS shells are made either in a form of command-line interface (**CLI**, also known as) where users can **provide commands** via **text**, or graphical user interface (**GUI**) where users can provide commands via mouse clicks. In this lab, we are going to learn a little bit about the command-line interface, bash scripting, and makefiles. 

You would need to install any POSIX-compliant OS before coming to this lab. The guide is provided in our course handout. 
{:.warning}

In order for us to be able to use CLI, we need to be familiar with their **commands** and their calling **syntax**. In particular, we are concerned with **UNIX-type shells** (POSIX is an IEEE standard that acts as a standard UNIX version) in this course. 
* **Open** your terminal/command line window. 
* The terminal window in front of you contains a `shell`, which enables you to use commands to access OS services.

**Fun fact:** the term "terminal" in the context of operating systems and computing has its origins in the early days of computers. Initially, computers were large, centralized machines that were accessed by multiple users through individual terminal devices. These terminals were called "terminal" because they served as the endpoint (or the terminus) of a communication line between the user and the mainframe or minicomputer. Today, when we refer to the "terminal" on a computer, we're usually talking about a <span class="orange-bold">terminal emulator</span> program that provides a text-based interface to the operating system. Users can enter commands through a shell (like bash, zsh, or PowerShell), which are then executed by the operating system. The term "terminal" has thus evolved from its original meaning but retains the core concept of being the point of interaction between a user and the computer system.
{:.new}

# Submission
The total marks for this lab is **20**. Please answer the questionnaire provided on eDimension Week 1. You are to score any **20 points** for this lab.  


### Task 1
`TASK 1:`{:.info} To find your current shell, type the command: `ps -p $$`

```java
bash-3.2$ ps -p  $$
  PID TTY           TIME CMD
70846 ttys003    0:00.01 bash
bash-3.2$
```
Bash shell ([bash](https://en.m.wikipedia.org/wiki/Bash_(Unix_shell)) is used in the screenshot above. There are other shells as well: [z-shell](https://en.m.wikipedia.org/wiki/Z_shell)) or [fish](https://en.m.wikipedia.org/wiki/Fish_(Unix_shell)). Which one to choose? It is entirely up to you. 

The CLI accepts **commands** (the **first** word, e.g. ps, that you type into the CLI is the command), **entered line by line** and it will be executed **sequentially**. There are two types of commands in general, commands **with** and **without** options or arguments.

# Basic Commands

## Without Options

### Task 2

`TASK 2:`{:.info} Try the following basic commands in sequence `date`, `cal`, `pwd`, `who`, `clear`:

1. E.g: date and press enter. You should see today’s date given to you, for example:

```java
bash-3.2$ date
Thu May  5 14:08:51 +08 2022
bash-3.2$
```

2. Do the same thing with `cal, pwd, who, clear`.
   - Figure out what each command does using `man`
   - You can use `man <command>` and press `q` to quit anytime.

## With Options

The commands you have typed above are those that **do not** require options or more arguments (although they do accept these too). Some commands require more input arguments.

For example, the command ps shows the list of processes for the current shell, while `ps -x` shows all processes that are owned by the current user even when it doesn’t have a **controlling terminal**.

```java
bash-3.2$ ps
  PID TTY           TIME CMD
61581 ttys000    0:01.80 /bin/zsh -l
61605 ttys000    0:00.00 /bin/zsh -l
61652 ttys000    0:00.00 /bin/zsh -l
61653 ttys000    0:00.04 /bin/zsh -l

bash-3.2$ ps -x
  PID TTY           TIME CMD
  393 ??         0:02.43 /System/Library/Frameworks/LocalAuthentication.framework/Support/coreauthd
  394 ??         1:29.34 /usr/sbin/cfprefsd agent
  398 ??         0:00.05 /System/Library/Frameworks/ColorSync.framework/Versions/A/XPCServices/com.app
  399 ??        25:02.92 /usr/libexec/UserEventAgent (Aqua)
  401 ??         2:06.64 /usr/sbin/distnoted agent
  403 ??         0:12.27 /System/Library/PrivateFrameworks/CloudServices.framework/Helpers/com.apple.s
  404 ??         2:02.15 /usr/libexec/knowledge-agent
  405 ??         0:06.55 /usr/libexec/lsd
  406 ??         3:05.55 /usr/libexec/trustd --agent
  407 ??        15:44.58 /usr/libexec/secd
  409 ??         0:02.05 /System/Library/CoreServices/sharedfilelistd
  410 ??        40:41.61 /System/Library/PrivateFrameworks/CloudKitDaemon.framework/Support/cloudd
  411 ??         0:00.19 /System/Library/CoreServices/backgroundtaskmanagementagent
  412 ??         0:05.30 /System/Library/PrivateFrameworks/TCC.framework/Resources/tccd
  413 ??        12:36.92 /usr/libexec/nsurlsessiond
```

Some commands accept single hyphen (-) as options, and some other accepts double hyphens (--), or no hyphen at all. It really depends on **convention**, so be sure to **read the manual properly**.
{:.info}

For example, the command: `git commit -v -a --amend` stands for:

- Use git to **stage** new changes (`commit`)
- Automatically stage **all** files that have been modified/deleted (`-a`)
- Do it in a **verbose** way, with detailed message highlighting all diff (`-v`)
- **Replace** the tip of the current branch by creating a new commit (`--amend`)

# Path

## Basics of Directory

The file system has many directories, starting from the **ROOT** (symbolised as a single forward slash /) and you can have directories within a directory thus forming a **hierarchy** of directories. Each “level” is separated by the forward slash symbol.
{:.info}

For instance,

`/Users/natalie_agus/Downloads` simply look like this in the window:

<img src="{{ site.baseurl }}/assets/images/lab1/2.png"  class="center_seventy no-invert"/>

If your directory has spaces in its name, you need to use the **backslash**) to indicate that it is part of the string, eg:
`/Users/natalie_agus/Google\ Drive`

### Task 3

`TASK 3:`{:.info} Find your starting context by running the command `pwd`, followed by `ls`:

- The command `ls` lists **all** files that exist in this **context**, which is the Desktop directory in the example above (`/Users/natalie_agus/Desktop`)
- When you run the command **ls** by itself, it uses your **current** directory as the context, and lists the files that are in the directory you are in.
- You can use the `ls` command to list the files in a directory that's **not** your current directory, e.g: `ls /Users/natalie_agus/Downloads`

Now try another command called `cd` to **change** the current working directory:

- Change the current working directory into any directory that’s accessible from the current context
- The command cd is analogous to **double clicking** a folder in the GUI
- You can go “back” to one previous directory level using the command `cd ..` (or you can chain it to go two levels up for instance, `cd ../..`

<img src="{{ site.baseurl }}/assets/images/lab1/3.png"  class="center_seventy no-invert"/>

## Environment Variables

Another way of providing context is through something called **environment** **variables**. Tryout the command: `cd $HOME`

- The `$HOME` part is a reference to the HOME variable, and is replaced by the path to your home directory when the command is run.
- In other words, running cd `$HOME` is the same as running `cd <actual path to your home>`, or `cd ~` (~ is a default **shell variable** that points to the current User’s home directory, and it also has [other usages](https://www.baeldung.com/linux/tilde-bash) that you can read if you’re free).
- To checkout what the value of your `$HOME` variable is, type the command: echo `$HOME`

To find out about your current environment variables, you can enter the command `env`. Look at the values of common variables such as `HOME` and `PATH`. You can also create your own environment variables using the command `export`.
{:.info}

For example:

1. Run the command: `export MESSAGE1="This is message 1"`
2. You can now execute `echo $MESSAGE1` and observe get the string output

In the example below, the environment variable `$MESSAGE1` initially did not exist. After we `export` it, we can now print the environment variable `$MESSAGE1`.

<img src="{{ site.baseurl }}/assets/images/lab1/4.png"  class="center_fifty no-invert"/>

## `$PATH` Environment Variable

One of the most important environment variables you’ll work with on the command line is <span style="color:#f7007f;"><b>$PATH</b></span>.

- This is the key to how our shell **knows** which file to execute for commands like cd or echo or other built-in or installed programs.
- The PATH variable provides the <span style="color:#f7007f;"><b>additional context</b></span> that the command line needs to figure out where that particular file is in the system.
- Hence, if you have installed an app (e.g: Telegram) and tried to execute the binary from the command line and met with the error `command not found`, it simply means you haven’t added the path where that binary is to the `$PATH` environment variable.

For example, you can add the **binary** of the **Telegram** app onto the `$PATH` using the command `export`, and now you can simply execute it from anywhere (a new Telegram window is opened on the second Telegram command):

<img src="{{ site.baseurl }}/assets/images/lab1/5.png"  class="center_seventy no-invert"/>

### Task 4

`TASK 4:`{:.info} Examine the value for `$PATH` in your system.

Open that folder (from your Desktop GUI) and navigate to that path. **You may need to enable viewing of hidden files.** If you are using WSL, you need to cd to this path as there’s no GUI. The screenshot below shows the content of `/bin/` directory where some default system programs like `ps, date, pwd, echo,` etc reside:

<img src="{{ site.baseurl }}/assets/images/lab1/6.png"  class="center_fifty no-invert"/>

# Configuring a Terminal Session Using `.rc` Files

### Motivation 
The command `export` that modifies the environment variable is only valid for <span style="color:#f7007f;"><b>this</b></span> session. For instance, the Telegram command above will not work anymore if a user opens a new Terminal session (need to `export` again).

### The `.rc` File
To avoid this hassle, there exists a setup script (unique to each shell) that is run whenever a new session starts. This script is typically placed in the user's `home` directory. For instance, `.bashrc` (yes, with this exact name) is a Bash shell script that Bash runs whenever it is started interactively. For Z-shell, this script is called `.zshrc`.

In short, it initialises an interactive shell session. You can put **any command** or setup instructions that you could type at the command prompt in that file. It works by being run each time you open up a new terminal, window or pane.

The term "rc" in filenames like `.bashrc`, `.zshrc`, or `.vimrc` stands for "run commands" or "runtime configuration." These "rc files" are configuration files that contain settings, variables, and scripts that are loaded and run by their respective programs upon startup. For shells like bash and zsh, their respective rc files (~/.bashrc for Bash and ~/.zshrc for Zsh) are used to configure the shell environment. This can include setting up environment variables, aliases, functions, and other shell settings that customize the behavior or appearance of the shell.
{:.info}

### Task 5

`TASK 5:`{:.info} Adding _Desktop_ to your `$PATH` environment variable **permanently**.

- Go to your home directory: `cd $HOME`
- Create a new file called .bashrc: `touch .bashrc`
- Open the file with any text editor, eg: `nano .bashrc`
- Type: `PATH="$HOME/Desktop:$PATH" `
- Save the file by pressing <span style="color:#f7007f;"><b>CTRL+X</b></span>, and then follow the instruction and press `Enter`
- Restart your session by typing `exec bash`
- Print your `$PATH` using `echo $PATH` command and notice how **Desktop** is now part of your environment variable

```java
bash-3.2$ cd $HOME
bash-3.2$ touch .bashrc
bash-3.2$ nano .bashrc
bash-3.2$ echo $PATH
/Users/natalie_agus/Desktop:...
bash-3.2$
```

# Common Commands

### Shortcut: alias

An `alias` lets you create a **shortcut** name for a command, file name, or any shell text. By using aliases, you save a lot of time when doing tasks you do frequently. You can see current aliases using the `alias` command:

<img src="{{ site.baseurl }}/assets/images/lab1/7.png"  class="center_seventy no-invert"/>

Or **create** an alias:

- `alias name=’command’`
- Example: `alias gst=’git status’`

<img src="{{ site.baseurl }}/assets/images/lab1/8.png"  class="center_seventy no-invert"/>

`alias` is particularly useful when you define them in your shell’s setup script.
{:.info}

### General system usage and statistics

Note that each of the commands below accept **OPTIONS**. Read their manuals for more information using the `man` command.

<hr>
`man <command>`
{:.highlight}
Shows **documentation** of the `<command>` (press q to exit the window after you’re done reading).
<hr>
`ls <options>`
{:.highlight}
Shows the **list of files** in the current directory.
<hr>
`ps <options>`
{:.highlight}
Shows the **list of processes** in the system.
<hr>
`sudo <command>, sudo apt install <packagename>`
{:.highlight}
`sudo` **Executes** the command with **administrative** privileges. `apt` is a **package manager**, it installs packages for **Debian-based Linux distributions**. We can use it to install anything, eg install `node` and `npm`. Using package manager is recommended since you can simply `update` or `remove` (uninstall) it too (`sudo apt remove <packagename>`, etc).

- `sudo apt update`
- `sudo apt install nodejs npm`
- For mac users, you can use [brew](https://brew.sh) as your package manager instead.
<hr>
`chmod +x <path/to/filename>`
{:.highlight}

Make a file **executable**. However, firstly, you need to declare in your script **which interpreter** to use.

- You state the path to this interpreter as the **first line** in your file
- This is called a **shebang**.
  - In Unix-like operating systems, the shebang line provides the **path** to an executable program (e.g. `bash`, `python`) that can interpret the following lines as executable instructions, allowing the user to **run the text file as an executable program** by typing the name of the file directly in the shell **provided the execute permission bit is set**.
- For instance, if this file is a shell script, it should be `#!/bin/sh` or `#!/bin/bash`. If it is a python script, the interpreter should be something like `#!/usr/bin/env python`
<hr>
`df <options>...`
{:.highlight}
Shows the **available disk space** in each partition.
<hr>
`top`
{:.highlight}
**Monitors** processes and system resource usage on Linux.
<hr>
`ifconfig <options>`
{:.highlight}
Displays **information** about all network interfaces currently in operation.
<hr>
`kill -9 <pid>, pkill <process_name>`
{:.highlight}
**Kills** (ends) the process matching the given pid, the same thing happens when you click the close (x) button on the window of a running app.
<hr>
`ping <servername> <options>`
{:.highlight}
**Checks connection** to a server. For example, `ping google.com` tells you whether your connection is active or not.
<hr>
### File creation and manipulation

Note that each of the commands below accept OPTIONS. Read their manuals for more information using the man command.
<hr>
`mkdir <dirname>`
{:.highlight}
Creates a directory (folder).
<hr>
`rmdir <dirname>, rm -r <dirname>`
{:.highlight}
Deletes an **empty** directory, and the latter removes a directory that contains files. <span style="color:#f7007f;"><b>Be careful!</b></span> Deleting things from the command line doesn’t allow you to retrieve it back. <span style="color:#f7007f;"><b>Unlike deleting from the GUI, it won't be found in the trash</b></span>.
<hr>
`touch <newfilename.format>`
{:.highlight}
**Creates** a new file with whatever name and format you want.
<hr>
`mv <source> <destination>`
{:.highlight}
**Moves** a file from the source path to destination path. Commonly used to **rename** files.
<hr>
`cp <source> <destination>`
{:.highlight}
**Copies** a file from a location to another.
<hr>
`locate <filename>`
{:.highlight}
**Locate** a particular filename in your file system, if you have set it up in the first place. Will return a path to that `<filename>`.
<hr>
`cat <path/to/filename>`
{:.highlight}
**Displays** the **contents** of a file, and
<hr>
`wc <path/to/filename>`
{:.highlight}
**Prints** a **count** of newlines, words, and bytes for each input file.
<hr>

### Command-Line Text Editor: nano, vim

Command-line text editor provides a handy way to manipulate text files in terminal without the need of installing any other apps. There are a few options, but two of the most popular ones are [nano](https://www.howtogeek.com/howto/42980/the-beginners-guide-to-nano-the-linux-command-line-text-editor/) and [vim](https://opensource.com/article/19/3/getting-started-vim). The latter has a higher learning curve, so we will stick to nano for quick editing.

You can open any created text file using the command:

- `nano <path/to/filename>`
- Then you can start typing as per normal
- Press <span style="color:#f7007f;"><b>Ctrl + X</b></span> to exit, and then **Enter** to save
- You can press **Ctrl + G** as well to bring up the shortcut menu anytime during editing
  <img src="{{ site.baseurl }}/assets/images/lab1/9.png"  class="center_seventy no-invert"/>

After saving, you can check the content of the file using the command `cat <path/to/filename>`.

## Standard Streams

Standard streams are **input** **and** output communication channels between a <span style="color:#f7007f;"><b>running process</b></span> and its <span style="color:#f7007f;"><b>environment</b></span> when it begins execution. They are **streams** of data that travel from where a program was **executed**, to the places where the program is **processed** and then back again to where the program was **executed**.

The three input/output (I/O) connections are called standard input (stdin), standard output (stdout) and standard error (stderr).
{:.info}

Streams are usually connected to the **terminal** in which they are executed. By default, `stdin` is connected to your **keyboard**, and `stdout + stderr` are directed to your **terminal**. You might be wondering how your keyboard and display then is shared among so many processes? The details require Streams in Linux are treated as though they were files (**Week 6 Material**). E.g: you can **read** text from a file, and you can **write** text into a file. <span style="color:#f7007f;"><b>Both of these actions involve streams of data</b></span>.

When you run a python script, e.g: `python3 playground.py`,

<img src="{{ site.baseurl }}/assets/images/lab1/10.png"  class="center_seventy no-invert"/>
* **Input** from your keyboard is passed from the terminal into the python process, and 
* **Output** from your python process is passed back to your terminal **display**. Here’s a simplified illustration:
<img src="{{ site.baseurl }}/assets/images/lab1/11.png"  class="center_seventy"/>

> *Where are these “files” for `stdin`, `stdout`, and `stderr` respectively?*

`stdin`, `stdout`, and `stderr` are **created** by your OS. You can witness this pretty easily if you run a python script on one terminal (and let it hang there, don’t let it terminate yet! Shown on the right side), find out its process id and then see its details on another terminal (shown on the left) using the `lsof` command.
{:.info}
<img src="{{ site.baseurl }}/assets/images/lab1/12.png"  class="center_full no-invert"/>

You can see in the last three lines that stdin (0u), stdout (1u), and stderr (2u) all point to the file `/dev/ttys004`. This is the file that is created by the OS and watched by our terminal.

### Standard output

A standard output is a **default** _place_ (it's just a file actually) for output to go, also known as `stdout`. Your shell is constantly **watching** that output file, and whenever there’s something there, it will automatically print to your screen. For instance, `echo "hello"` is a command that means ”output the string hello to standard output”.

- The process echo prints to `stdout` (probably `/dev/ttyx`),
- ...and your terminal in turn shows it in its GUI display.

### Standard input

The standard input (`stdin`) is a default place where processes listen for information. For example, all the commands above with no other arguments listen for input on `stdin`.
Try typing `cat` on the command line and press enter:

- Notice you can type any character from your keyboard, because it watches for input on `stdin`,
- Then, output what you type to `stdout` (and your shell is watching that output place so it is being printed on your screen), until you type an EOF (end of line) character: `CTRL + d`.

### Standard error

The standard error (`stderr`) is the place where error messages go. Try this command that will prompt an error such as: `cat <inexistent_path/to/filename>`.

What is the output that you see? Similar to stdout, stderr is printed directly to your screen by your shell.
<img src="{{ site.baseurl }}/assets/images/lab1/13.png"  class="center_fourty no-invert"/>


### Stream Redirection

`stdin`, `stdout`, and `stderr` for every process is symbolized with [file descriptor](https://en.wikipedia.org/wiki/File_descriptor) `0`, `1`, and `2` respectively. Each file associated with a process is allocated a unique number to identify it, this number is called the **file descriptor**.

We can <span style="color:#f7007f;"><b>redirect</b></span> stdin using the `< `operator, stdout using the `>` operator, and stderr using the `2>` operator.

- `<command> > <filename>`:  we are **redirecting** the `stdout` of `<command>` to the file `<filename>`. That means, we will write whatever that was printed out by the process `<command>` to the file with `<filename>`.
- `<command> < <filename>`: we are **redirecting** the `stdin` of `<command>`, e.g: use the **content** of filename as an **input** to command. This is particularly useful for commands that only take in input streams, and are unable to read the content of a file given a filename.

Note that `stdout` redirection will **truncate** (<span class="orange-bold">erase</span>) the original content of the `<filename>`. If you want to append to the **existing** content of file instead, you can use the `>>` operator.
{:.warning}

### Task 6

`TASK 6:`{:.info} One example of a program that benefits from stream redirection is `tr`. It is a command line utility for **translating** or **deleting** characters. Do the following:

1. Create a text file with the following content: ”Hello, have a good day today!”, name it `test.txt` and **save** it to your current directory.
2. Then, type the following in the command line (don't forget to navigate to your current directory first using `cd`!): `tr "[a-z]" "[A-Z]" test.txt`
3. You will see such usage suggestion instead:

```java
usage: 	tr [-Ccsu] string1 string2
        tr [-Ccu] -d string1
        tr [-Ccu] -s string1
        tr [-Ccu] -ds string1 string2
```

This is because tr <span class="orange-bold">cannot</span> get input directly from a file. It only reads from `stdin`. 4. Now try` tr "[a-z]" "[A-Z]" < test.txt`. Your console should print ”HELLO, HAVE A GOOD DAY TODAY!”, capitalizing the content of the `test.txt` file (but not changing its content). 5. So based on your observation, can you deduce what is the difference between these two commands:

1.  `tr "[a-z]" "[A-Z]" < test.txt`
2.  `tr "[a-z]" "[A-Z]" test.txt`
3.  Now what if we want to store the capitalized content to another file? Try: `tr "[a-z]" "[A-Z]" < test.txt > new_test.txt`. You should find that “HELLO, HAVE A GOOD DAY TODAY!” exists within `new_test.txt`, since we **redirect** `stdout` to create this new file.
4.  What if we want to write back to `test.txt`? What can you deduce from the output?

Perhaps the screenshot below might help you for steps 1-7 above.
<img src="{{ site.baseurl }}/assets/images/lab1/14.png"  class="center_seventy no-invert"/>

## Pipe `|`

The Pipe `|` is a *special* feature in Linux that lets you use **two** or **more** commands such that <span style="color:#f7007f;"><b>output</b></span> of one command serves as <span style="color:#f7007f;"><b>input</b></span> to the next. Since pipes are a feature of the shell (and not a command), they are available in **any** shell environment, including bash, zsh, and others found on Unix-like systems such as Linux and macOS

It may sound similar to stream redirection, but the general rule of thumb is that if you're connecting the output of a command to the input of another command, use a pipe, denoted as `|` symbol. If you are outputting to or from a file use the redirect.
{:.info}

### Task 7

`TASK 7:`{:.info} Try out how to pipe:

1. Suppose we want to pass the output of `cat` command as the input of `sort` command. We can’t do this with redirection:
   <img src="{{ site.baseurl }}/assets/images/lab1/15.png"  class="center_fifty no-invert"/>

2. However, using **pipe** works. It serves as a way to allow **interprocess** communication (Week 3 materials):
   <img src="{{ site.baseurl }}/assets/images/lab1/16.png"  class="center_thirty no-invert"/>

## Download Files using `curl`

The curl command allows us to transfer data (download, upload) online. For instance, we can download the GNU general public license text file using the command:

`curl -o GPL-3 https://www.gnu.org/licenses/gpl-3.0.txt`

- The `-o` option: Write output to `<filename>` instead of `stdout`

If you don't have any command (e.g: there's an error saying `command [command] not found`), install it using `sudo apt install curl`.
{:.note}

## Output Filtering

One last handy tool to learn is output **filtering**. The [`grep` or `awk`](https://techviewleo.com/awk-vs-grep-vs-sed-commands-in-linux/) command will scan the document for the desired information and present the result in a format you want. The command grep also accepts [regex](https://regexr.com) to search for more complex patterns.

A filter takes input from one command, does some processing, and gives output.
{:.info}

### Task 8

`TASK 8:`{:.info} Suppose we have a long document, that GNU license we downloaded from the task above.

1. To download the license, type the command: `curl -o GPL-3 https://www.gnu.org/licenses/gpl-3.0.txt`
2. Now search for a string inside a file using the commands:

- `grep "<string>” <path/to/file>`
- For example: `grep “GNU” GPL-3` prints every line containing “GNU” word.
- Some shells don't require the quotation marks for single-word search, so you can try `grep GNU GPL-3` as well.
  <img src="{{ site.baseurl }}/assets/images/lab1/17.png"  class="center_seventy no-invert"/>

Here are the common grep options to try:

- `-i`: both lower and upper case
- `-v`: search everything that does not contain the `<string>`
- `-n`: prints the line number too

The `<string> `argument of the grep command can accept **regex** too, for instance:

- `grep "^[A-Z]" GPL-3 -n`
- This search for every line starting with **capital** letters
  <img src="{{ site.baseurl }}/assets/images/lab1/18.png"  class="center_seventy no-invert"/>

You can also **pipe** the output of a command to the `grep` command so that you can filter it out. For example, `ps -ax` will report all running processes in your system (by all users, including your own). If we want to filter some processes by name, we can use grep and pipe to filter the Telegram process: `ps -ax | grep -i Telegram`

<img src="{{ site.baseurl }}/assets/images/lab1/19.png"  class="center_seventy no-invert"/>

## Why do we use the CLI? 
Now that we have learned _how_ to execute some commands, it is natural to **question** _why_ we need to use the command line. What can you do here with CLI that you cannot do through your common graphical user interface?

Well, it **depends** on your <span style="color:#f7007f;"><b>use case</b></span>. There’s a lot of debate on that, some might say that CLI allows you to do tasks **fast**, but that depends: depends on how **well versed you are** in using the CLI.
- If you are just a **basic** user, i.e: browse, watch your favorite tv series, edit photos, or text your friends then chances are you don’t need to use the command line.
- _If you’re a computer science graduate who intends to work in the field then CLI is probably your new best friend._

The most common use of the command line is ”system administration” or, basically, **managing** computers and servers. Some servers don't even have a GUI because they're not intended for everyday usage.
{:.note}

This includes **installing** and **configuring** software, **monitoring** computer resources (manage logs, setup cron jobs, daemons), **setting** up web servers (renaming or modifying thousands of files), and **automating** processes (setup databases / servers) on many hosts. Obviously these tasks are <span style="color:#f7007f;"><b>repetitive</b></span> and <span style="color:#f7007f;"><b>tedious</b></span> such that it is impossible to be done manually or one by one via the GUI.

## Shell Scripting

In this section, we briefly overview how shell scripts work, eg: `.sh`, `.zsh` scripts. A shell script is simply lines of code for the shell to interpret (if you use z/bash shell, you can use z/bash shell script. Since both bash and z are derived from the Bourne shell family, both scripts have similar syntax). 

Why do we need to write a shell script? 
- Imagine you want to create thousands of text files (using touch). 
- You wouldn’t want to type the command one by one, but rather just run a script that does this task in one shot.

Writing shell scripts help us automate repetitive and tedious tasks. 
{:.info}

### Task 9

`TASK 9:`{:.info} Open `nano` and type the following:

```bash
#!/bin/bash

start_time=$(date +%s)

# the following substitute the arguments as a list, without re-splitting them on whitespace
"$@"

end_time=$(date +%s)

echo "Time elapsed: $(($end_time - $start_time)) seconds"
```

Then, save it with a name `run.sh`, and change its mode to be executable: `chmod +x run.sh`. Now you can run any script and **time** it. For instance, suppose we have the following python script:

```python
import time
print("Loading.....")
time.sleep(2.5)
print("Done")
```

Running it with our script will **time** the execution of the python script above:

```java
bash-3.2$ ./run.sh python3 hello.py
Loading.....
Done
Time elapsed: 3 seconds
bash-3.2$
```

From the task above, you have just created and run a super simple bash script. Note that the first line is the [shebang](<https://en.wikipedia.org/wiki/Shebang_(Unix)>). Similar to coding in any other language, you can use variables, functions, conditional statements, loops, comparisons, etc in your bash script. You can learn more in your own time, and see [examples of awesome bash scripts](https://github.com/awesome-lists/awesome-bash).

For example, you can [customize your prompt](https://github.com/arialdomartini/oh-my-git) in a git repo:
<img src="{{ site.baseurl }}/assets/images/lab1/20.png"  class="center_seventy no-invert"/>

## Compiling Programs

We can also compile and run programs from the **command** line, provided that you have the **compiler** or **interpreter**, e.g. `python3`, `gcc`, or `javac`.

### Task 10

`TASK 10:`{:.info} To demonstrate this idea, download this starter code: `git clone https://github.com/natalieagus/makeFileDemo.git`

We require you to have `gcc` for this task. If your OS doesn't have it, you can install it with (Ubuntu):

```
sudo apt update
sudo apt install build-essential
```

Read all the `.c` and `.h` files and get an understanding of what each file is supposed to do. To **compile** the files and **run** the executable:

1. Navigate to this directory and type the command: `gcc -o prog.o main.c hello.c factorial.c binary.c `
2. And then execute by typing `./prog.o`

Experiment with the program a little bit. You should see a prompt for you to key in a number. Simply type something and press enter.

<img src="{{ site.baseurl }}/assets/images/lab1/21.png"  class="center_seventy no-invert"/>

In case you haven't connected the dots, `gcc` compiles all the input argument files: `main.c, hello.c, factorial.c, binary.c` and produces a binary **output** (this is what `-o` means) named `prog.o` which you can execute using `./prog.o `.

## Makefile

In this simple context, it is feasible to type out the source file one by one each time you want to compile your program. However in a large scale project with thousands of files, it is very tedious to type the compilation command all the time. Hence, the make command allows us to compile these files more easily. It requires a special file called the `makefile`.

1. Now instead of typing gcc and all that above, type `make` instead
2. After executing `make`, realize that prog.o is made. You can run the executable in the terminal by typing `./prog.o` or by simply clicking that executable in your shell GUI (your desktop).

Lets now examine how makefile is made.

```makefile
# Define required macros here
REMOVE = rm
CC = gcc
DEPENDENCIES = main.c hello.c factorial.c binary.c
OUT = prog.o

# Explicit rules, all the commands you can call with make
# (note: the <tab> in the command line is necessary for make to work)
# target:  dependency1 dependency2 ...
#       <tab> command

#Called by: make prg
#also executed when you just called make. This calls the first target.
prog: main.c hello.c factorial.c binary.c
        gcc -o prog.o main.c hello.c factorial.c binary.c

prog1: $(DEPENDENCIES)
        $(CC) $(DEPENDENCIES) -o $(OUT)

prog2: main.o hello.o factorial.o binary.o
        gcc -o prog2 main.o hello.o factorial.o binary.o


##make clean will remove myexecoutprog.o from the directory
clean:
        rm prog.o

clean1:
        $(REMOVE) $(OUT)

clean2:
        rm prog2 *.o

#Implicit rules
main.o : main.c functions.h
factorial.o: factorial.c functions.h
hello.o: hello.c functions.h
binary.o: binary.c functions.h
```

1. The first four lines are the MACROS, which are convenient shorthands you can make to make your life easier when typing these codes.
2. Afterwards, there’s a bunch of explicit rules that you can call using make. So in this makefile, you can try calling these in sequence and observe what each rule do:
   - `make prog`
   - `make prog1`
   - `make clean`
   - `make clean1`

## Recompilation

Run this command consecutively:

- `make clean`
- `make prog2`

You should see the following output on your terminal as a result of `make prog2`:
<img src="{{ site.baseurl }}/assets/images/lab1/22.png"  class="center_fifty no-invert"/>

Now open `binary.c` in the `nano` and add another instruction in it, eg a `printf` function at the end:
<img src="{{ site.baseurl }}/assets/images/lab1/23.png"  class="center_fifty no-invert"/>

When you try to `make prog2` again, the output shows that we only compile files concerning `binary.c` and <span style="color:#f7007f;"><b>not all files are recompiled</b></span>. Scroll down to the end of the makefile, and notice there’s **implicit** rules there to determine dependencies.

<img src="{{ site.baseurl }}/assets/images/lab1/24.png"  class="center_fifty no-invert"/>

This gives more **efficient** compilation. It only recompiles parts that are changed. The Figure below shows the data dependency between files that are specified in the makefile.
Note: `File_1` → `File_2` means that `File_2` depends on `File_1`.

<img src="{{ site.baseurl }}/assets/images/lab1/25.png"  class="center_seventy "/>

# Summary 
In this lab, you are exposed to various commands and how to navigate your system using the CLI. It needs time to develop the habit. Perhaps one of the more fun things to do is to decorate and beautify your shell. You may [check this out](https://ohmyz.sh) for a beginner-friendly start. 

# Appendix 

## Simple `.zshrc` setup 
Oh My Zsh is a popular framework for managing Zsh configurations, offering a wide array of themes, plugins, and features that enhance the Zsh experience. However, due to its **extensive** functionality and the sheer number of plugins and themes it provides, some users may find it somewhat **bloated**, especially if they only need a subset of its capabilities or are using older hardware where performance is a concern.

If you're looking for alternatives that are lighter or wish to customize Zsh in a more minimalistic manner, here's a few simple steps to take.  

```sh 
# run these commands at ~ directory 
# update apt 
sudo apt update 
# install zsh 
sudo apt install zsh -y 
# set  sudo and user password 
sudo passwd 
sudo passwd [YOUR_USERNAME]
# change to zsh as your default shell 
chsh -s $(which zsh) 

# zsh setup at ~
# install useful plugins like syntax highlighting, completions, auto-suggestions, fzf and fzf tab 
# install basic theme: spaceship 
# please give each git repo a read to understand how to use them 
sudo apt install git
mkdir -p ~/.zsh/plugins ~/.zsh/themes
touch ~/.zsh/.zshrc
touch ~/.zsh/.zsh_history
git clone https://github.com/spaceship-prompt/spaceship-prompt.git ~/.zsh/themes/spaceship-prompt
git clone https://github.com/zdharma-zmirror/fast-syntax-highlighting.git ~/.zsh/plugins/fast-syntax-highlighting
git clone https://github.com/zsh-users/zsh-completions.git ~/.zsh/plugins/zsh-completions
git clone https://github.com/zsh-users/zsh-autosuggestions.git ~/.zsh/plugins/zsh-autosuggestions
git clone https://github.com/unixorn/fzf-zsh-plugin.git ~/.zsh/plugins/fzf-zsh-plugin
git clone https://github.com/Aloxaf/fzf-tab.git ~/.zsh/plugins/fzf-tab

# create symbolic link
ln -s -T ~/.zsh/.zshrc ~/.zshrc
# open ~/.zshrc using nano
nano ~/.zshrc 
```

Now paste the content below: 

```sh
### ZSH HOME
export ZSH=$HOME/.zsh

### ---- history config -------------------------------------
export HISTFILE=$ZSH/.zsh_history

# How many commands zsh will load to memory.
export HISTSIZE=10000

# How many commands history will save on file.
export SAVEHIST=10000

# History won't save duplicates.
setopt HIST_IGNORE_ALL_DUPS

# History won't show duplicates on search.
setopt HIST_FIND_NO_DUPS

# Immediate Append
setopt INC_APPEND_HISTORY

# Add timestamp
export HISTTIMEFORMAT="[%F %T] "
setopt EXTENDED_HISTORY

### PATH
export PATH=$HOME/bin:/usr/local/bin:/snap/bin:/opt/bin:$PATH

### ---- PLUGINS & THEMES -----------------------------------
source $ZSH/themes/spaceship-prompt/spaceship.zsh-theme
source $ZSH/plugins/fast-syntax-highlighting/fast-syntax-highlighting.plugin.zsh
source $ZSH/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
source $ZSH/plugins/fzf-zsh-plugin/fzf-zsh-plugin.plugin.zsh
source $ZSH/plugins/fzf-tab/fzf-tab.plugin.zsh
fpath=($ZSH/plugins/zsh-completions/src $fpath)


### --- Spaceship Config ------------------------------------
SPACESHIP_PROMPT_ORDER=(
  user          # Username section
  dir           # Current directory section
  host          # Hostname section
  git           # Git section (git_branch + git_status)
  hg            # Mercurial section (hg_branch  + hg_status)
  exec_time     # Execution time
  line_sep      # Line break
  jobs          # Background jobs indicator
  exit_code     # Exit code section
  char          # Prompt character
)
SPACESHIP_USER_SHOW=always
SPACESHIP_PROMPT_ADD_NEWLINE=false
SPACESHIP_CHAR_SYMBOL="❯"
SPACESHIP_CHAR_SUFFIX=" "
```

Afterwards, install a few more useful tools:

```sh
# exa
sudo apt install exa -y
echo "alias ls=\"exa --long\"" >> ~/.zshrc
echo "alias zr=\"source ~/.zshrc\"" >> ~/.zshrc

# bat
sudo apt-get install bat -y
echo "alias cat=\"batcat\"" >> ~/.zshrc

# jump
wget https://github.com/gsamokovarov/jump/releases/download/v0.51.0/jump_0.51.0_amd64.deb && sudo dpkg -i jump_0.51.0_amd64.deb
echo 'eval "$(jump shell)"' >> ~/.zshrc
```

Exit your current shell and reopen. You should use `zsh` now and have your prompt look as such. Press `TAB` after a command and you will be faced with nicely rendered choices as shown. Take your time to explore and install new tools as necessary to elevate your CLI usage and navigation experience. 
<img src="{{ site.baseurl }}//docs/Labs/images/01-Lab1-CLI/2024-04-02-01-41-22.png"  class="center_full no-invert"/>
