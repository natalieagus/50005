---
layout: default
permalink: /labs/02-toctou
title: Time of Check, Time of Use Attack
description:  Investigate how TOCTOU attack is launched
parent: Labs
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
**Natalie Agus (Summer 2025)**

# Time of Check, Time of Use Attack 
{:.no_toc}
In this lab, you are tasked to investigate a program with TOCTOU (Time of Check, Time of Use) race-condition vulnerability.

A "Time of Check to Time of Use" (**TOCTOU**) attack is a type of cybersecurity vulnerability that arises when a system checks the state of a resource or condition at one point in time, but uses that resource or relies on that condition at a later time. During the gap between the "check" and the "use," an attacker can manipulate the resource or condition, leading to unauthorized access, data corruption, or other security breaches.
{:.info}

The lab is written <span style="color:#f77729;"><b>entirely in C</b></span>, and it is more of an <span style="color:#f77729;"><b>investigative</b></span> lab with fewer coding components as opposed to our previous lab. At the end of this lab, you should be able to:

- Understand what is a <span style="color:#f77729;"><b>TOUTOU bug</b></span> and why is it prone to attacks
- Detect <span style="color:#f77729;"><b>race-condition</b></span> caused by the TOCTOU bug
- Provide a <span style="color:#f77729;"><b>fix</b></span> to this TOCTOU vulnerability
- <span style="color:#f77729;"><b>Examine</b></span> file permissions and modify them
- Understand the concept of <span style="color:#f77729;"><b>privileged programs</b></span>: user level vs root level
- <span style="color:#f77729;"><b>Compile</b></span> programs and make documents with different privilege level
- Understand how <span style="color:#f77729;"><b>sudo</b></span> works
- Understand the difference between <span style="color:#f77729;"><b>symbolic</b></span> and <span style="color:#f77729;"><b>hard</b></span> links

# Submission 
You are to complete this lab's questionnaire on eDimension as you complete the tasks.

Schedule a checkoff as a Team with your Lab TA by next week Friday 6PM. 

{:.task-title}
> ✅ Checkoff
> 
> Demonstrate a successful TOCTOU Attack (Task 11) to gain marks. 


# Background

## Race Condition

A <span style="color:#f77729;"><b>race condition</b></span> occurs when two or more threads (or processes) access and perform <span style="color:#f77729;"><b>non-atomic operations</b></span> on a shared variable (be it across threads or processes) value at the same time. Since we cannot control the <span style="color:#f77729;"><b>order</b></span> of execution, we can say that the threads / processes race to modify the value of the shared variable.

The <span style="color:#f77729;"><b>final</b></span> value of the shared variable therefore can be <span style="color:#f7007f;"><b>non deterministic</b></span>, depending on the particular <span style="color:#f77729;"><b>order</b></span> in which the access takes place. In other words, the cause of race condition is due to the fact that the function performed on the shared variable is non-atomic.

In this lab we are going to <span style="color:#f77729;"><b>exploit</b></span> a program that is <span style="color:#f7007f;"><b>vulnerable to race-condition</b></span>.

- The program alone is single-threaded, so in the absence of an attacker, there’s nothing wrong with jthe program.
- The program however is vulnerable because an attacker can exploit the fact that the program can be subjected to race-condition.

You will learn about race condition properly in future lectures. This lab is just a preview.
{:.note}

# Setup

## Installation

Clone the files for this lab using the command:

```
git clone https://github.com/natalieagus/lab_toctou
```

You should find that the following files are given to you:
<img src="{{ site.baseurl }}//docs/Labs/images/03-Lab3-toctou/2025-05-22-13-51-24.png"  class="center_seventy no-invert"/>

Now go to the `User/` directory and call `make`. You should find these files in the end.
<img src="{{ site.baseurl }}/assets/images/lab2/2.png"  class="center_full no-invert"/>

Do <span class="orange-bold">NOT</span> used a shared drive with your Host machine, or a shared external drive with other OS. You will NOT be able to create root files in this case. Clone the above file in `/home/<user>` directory instead.
{:.error}

## Login as Root User

### Task 1

`TASK 1:` To switch user and login as root, you must first set the <span style="color:#f77729;"><b>password</b></span> for the root account.
{:.task}

Type the following command:
```sh
sudo passwd root
```

You will be prompted for your <span style="color:#f77729;"><b>current user</b></span> password, then set a new password for `root`.

Remember this `root` <span style="color:#f77729;"><b>password</b></span>! You need this later to switch to `root` user. If you have a brainblock, use a simple phrase like: `rotiprata`.
{:.error}

Follow the instructions, and then switch to the root user using the command and the password you just created above for `root`:

```sh
su root
```

You will see the following <span style="color:#f7007f;"><b>new prompt</b></span>, indicating now you're logged in as `root`:
<img src="{{ site.baseurl }}/assets/images/lab2/3.png"  class="center_full no-invert"/>

Root user is the user with the highest (administrative) privilege. It has <span style="color:#f7007f;"><b>nothing to do with Kernel Mode</b></span>. Processes spawned while logged in as Root still runs on <span style="color:#f77729;"><b>User Mode</b></span>.
{:.warning}

### Task 2

`TASK 2:` Create files while logged in as `root`.
{:.task}

While logged in as `root`, navigate to `/FilesForRoot`, and type `make`. You should see a new directory called `Root/` created with the following contents:

<img src="{{ site.baseurl }}/assets/images/lab2/4.png"  class="center_full no-invert"/>

It is <span style="color:#f7007f;"><b>important</b></span> to check that the newly created files belong to the user root as shown in yellow above (beside the date is the `owner` of the file).

## Add other Users

### Task 3

`TASK 3:` Create 2-3 other users.
{:.task}

In order to proceed with this lab, you need to <span style="color:#f77729;"><b>login</b></span> as `root` and create a few <span style="color:#f77729;"><b>new</b></span> users before proceeding.

For the first user, do the following (the username must be `test-user-0`):

```sh
adduser test-user-0
```

Give it any password you like (preferably a good one, like `LDcwzD&#6JKr`), and then add it to the `sudo` group:

```sh
adduser test-user-0 sudo
```

<img src="{{ site.baseurl }}/assets/images/lab2/14.png"  class="center_full no-invert"/>

Then add a few more with different names: e.g `test-user-0`, `test-user-1`, with any appropriate password of your choice.

## Switch User 

### Task 4

`TASK 4:` Switch account as any of these new users to ensure that you've created new users successfully.
{:.task}

You can switch to your new user by using the command `su <username>`:

<img src="{{ site.baseurl }}/assets/images/lab2/15.png"  class="center_full no-invert"/>

Once you're done, <span style="color:#f77729;"><b>switch back</b></span> to `root` again using your `root` password you've set above.

### Task 5

`TASK 5:` Check `/etc/shadow`.
{:.task}

Once you're done, you should enter the command `cat /etc/shadow` and see that your newly created users are at the bottom of the file, with some hash values (actual values are different depending on the password you set for these users).

<img src="{{ site.baseurl }}/assets/images/lab2/16.png"  class="center_full no-invert"/>

### Task 6

`TASK 6:` Return to your original user account.
{:.task}

You can switch back to your original normal user account by using the same `su <username>` command:
<img src="{{ site.baseurl }}/assets/images/lab2/5.png"  class="center_full no-invert"/>

Now go up one level by typing `cd ..` and attempt to <span style="color:#f7007f;"><b>delete</b></span> `Root/` directory while logged in as your original <span style="color:#f77729;"><b>username</b></span>. You will find such permission denied message:

<img src="{{ site.baseurl }}/assets/images/lab2/6.png"  class="center_full no-invert"/>

🔎 What happened?
{:.highlight}

## File Permission

The reason you faced the <span style="color:#f7007f;"><b>permission denied</b></span> error is because your username doesn't have the correct <span style="color:#f7007f;"><b>permission</b></span> to edit the `Root/` directory.

<img src="{{ site.baseurl }}/assets/images/lab2/7.png"  class="center_full no-invert"/>

Notice that a <span style="color:#f77729;"><b>directory</b></span> is an <span style="color:#f77729;"><b>executable</b></span>, indicated by the ‘x’ symbol.

## The SUID bit

SUID stand for Set Owner User ID. A file with <span style="color:#f77729;"><b>SUID</b></span> set always <span style="color:#f77729;"><b>executes</b></span> as the user who <span style="color:#f7007f;"><b>owns</b></span> the file, regardless of the user passing the command.
{:.note}

List the file permission for all files inside `Root/` directory:
<img src="{{ site.baseurl }}/assets/images/lab2/8.png"  class="center_full no-invert"/>

Notice that instead of an `x`, we have an `s` type of permission listed on the two progs: `vulnerable_root_prog` and `rootdo`. **This is called the SUID bit.**

The <span style="color:#f7007f;"><b>SUID</b></span> bit allows <span style="color:#f77729;"><b>normal</b></span> user to gain <span style="color:#f77729;"><b>elevated privilege</b></span> (again this does <span style="color:#f7007f;"><b>NOT</b></span> mean Kernel Mode, just privilege levels among regular users) when executing this program.

If a normal user executes this program, this program runs in root <span style="color:#f77729;"><b>privileges</b></span> (basically, the creator of the program)
{:.info}

### Task 7

`TASK 7:` Reading protected file using regular user account.
{:.task}

While logged in as your original user account, try to read the file `/etc/shadow`:

```sh
cat /etc/shadow
```

You will be met with <span style="color:#f77729;"><b>permission denied</b></span> because this file can only be read by `root` user, and other users in the <span style="color:#f77729;"><b>same group</b></span>, as shown in the file details below;
<img src="{{ site.baseurl }}/assets/images/lab2/10.png"  class="center_full no-invert"/>

What group does `root` belong to? What about the user account in question (ubuntu in example above)? You can find out using the command `groups`:

<img src="{{ site.baseurl }}/assets/images/lab2/22.png"  class="center_seventy no-invert"/>

### Task 8

`TASK 8:` Gain privilege elevation.
{:.task}

Now, run the following command. We assume that your <span style="color:#f77729;"><b>current working directory</b></span> is at `/lab_toctou` directory. If not, please adjust accordingly.

```
./Root/rootdo cat /etc/shadow
```

When prompted, type the word `password`, and then press enter.

You will find that you can now read this file:
<img src="{{ site.baseurl }}/assets/images/lab2/9.png"  class="center_full no-invert"/>

The <span style="color:#f77729;"><b>reason</b></span> you can now successfully read the file `/etc/shadow` is because `rootdo` <span style="color:#f77729;"><b>has the SUID bit</b></span>. Any other program that is <span style="color:#f77729;"><b>executed</b></span> by `rootdo` will run with `root` (`rootdo` creator) privileges and <span style="color:#f77729;"><b>not</b></span> the regular user.

You can open `rootdo.c` inside `/lab_toctou/FilesForRoot/` to examine how it works, especially this part where it just simply checks that you have keyed in `password` and proceed to `execvp` (execute) the input command:

```java
    if (!strcmp(password, "password"))
    { // on success, returns 0
        printf("Login granted\n");
        int pid = fork();
        if (pid == 0)
        {
            printf("Fork success\n");
            wait(NULL);
            printf("Children returned\n");
        }
        else
        {
            if (execvp(execName, argv_new) == -1)
            {
                perror("Executable not found\n");
            }
        }
    }
```

In short, as the <span style="color:#f7007f;"><b>SUID</b></span> bit of rootdo program is set, it <span style="color:#f7007f;"><b>always runs with root privileges</b></span> <span style="color:#f77729;"><b>regardless</b></span> of which user executes the program.

While rootdo seems like a <span style="color:#f77729;"><b>dangerous</b></span> program, don’t forget that the <span style="color:#f77729;"><b>root</b></span> itself was the one who made it and set the SUID bit in the first place, so yes it is indeed<span style="color:#f77729;"><b> meant to run that way</b></span>.


## The `sudo` Command

`sudo` is a command in Unix and Linux-based operating systems. It stands for "superuser do" and allows a permitted user to execute a command as the superuser or another user, as specified in the sudoers file (`/etc/sudoers`). This provides a mechanism for granting <span class="orange-bold">administrator</span> privileges, ordinarily reserved for the root user, to normal users in a controlled manner.
{:.info}


When you typo `sudo <command>`, it prompts you for your <span style="color:#f77729;"><b>password</b></span>, then the program checks whether the user is verified, before executing with root privileges. In our little `rootdo` example, we just use hardcoded `password` to proceed.


Setting of `rootdo`'s <span style="color:#f77729;"><b>SUID</b></span> bit is done in the `makefile` inside `FilesForRoot` that you execute earlier while logged in as root,

```makefile
# refresh the root files and root log file
# Set UID for rootprog and rootdo
setup:
	chmod u+s ../Root/vulnerable_root_prog
	chmod u+s ../Root/rootdo
```

The command `chmod u+s filename` sets the `SUID` bit of that `filename`.

{:.new-title}
> About `sudo`
> 
> `sudo` is <span class="orange-bold">not</span> a built-in shell command. It is an external command provided by a package that is installed on most Unix and Linux-based systems. Built-in commands are part of the shell itself, like `cd` (change directory), `echo`, or `exit`, and they are executed directly by the shell without the need for calling an external program. You will be tasked to implement both built-in command and external command in **Programming Assignment 1**.

Running a command with `sudo` has no direct relationship with switching the CPU to Kernel/Machine mode. 
{:.warning}

# The Vulnerable Program
Our vulnerable program can be found in `/Root/vulnerable_root_prog`. Open `/FilesForRoot/vulnerable_root_prog.c` to find out what it does.

The program expects <span style="color:#f77729;"><b>two</b></span> arguments: to be stored at `char *fileName`, and `char *match`. It is a _supposedly secure_ program that will allow `root` to replace `*match` string inside `*fileName` with an SHA-512 hashed password `00000`.

```java
    if (argc < 3)
    {
        printf("ERROR, no file supplied, and no username supplied. Exiting now.\n");
        return 0;
    }

    char *fileName = argv[1];
    char *match = argv[2];
    FILE *fileHandler;

    ....
```

It will then <span style="color:#f77729;"><b>check</b></span> with the `access` <span style="color:#f7007f;"><b>system call</b></span> on whether or not the caller of this program (it's REAL ID) has <span style="color:#f7007f;"><b>permission</b></span> to access the target `fileName`:

```java
    if (!access(fileName, W_OK))
    {
        printf("Access Granted \n");
        ....

    }
```

Below we paste the documentation for `access()` from Linux man page:

> *`access()` checks whether the calling process can access the file pathname. If pathname is a symbolic link, it is dereferenced.*
>
> *The check is done using the calling process's <span style="color:#f77729;"><b>real</b></span> `UID` and `GID`, rather than the <span style="color:#f77729;"><b>effective IDs</b></span> as is done when actually attempting an operation (e.g., open, fopen, execvp, etc) on the file. Similarly, for the root user, the check uses the set of permitted capabilities rather than the set of effective capabilities; and for non-root users, the check uses an empty set of capabilities.*
>
> *This allows set-user-ID programs and capability-endowed programs to easily determine the invoking user's authority. In other words, `access()` does not answer the "can I read/write/execute this file?" question. It answers a slightly different question: "(assuming I'm a setuid binary) <span style="color:#f77729;"><b>can the ACTUAL user who invoked me</b></span> read/write/execute this file?", which gives set-user-ID programs the possibility to prevent malicious users from causing them to read files which users shouldn't be able to read.”*

Other system calls: `execvp`, `open` that we used in `rootdo` or standard `sudo` only checks the <span style="color:#f77729;"><b>effective</b></span> ID of the calling process, <span style="color:#f7007f;"><b>not the real ID</b></span>.
{:.warning}

## Details
`rootdo` runs with effective <span style="color:#f77729;"><b>root</b></span> privileges (<span style="color:#f77729;"><b>effective</b></span>, not real, since the caller to `rootdo` is only normal user), and that’s enough to run the `cat /etc/shadow` program since `cat` doesn’t utilise `access()` to check for the calling process.

On the other hand, `vulnerable_root_prog` <span style="color:#f7007f;"><b>tries</b></span> to be more secure by using the `access` system call to <span style="color:#f7007f;"><b>prevent users with elevated privileges to modify files that do not belong to them</b></span>.

`vulnerable_root_prog` "security" attempt fails miserably and ends up being susceptible to a particular race condition attack due this weakness called TOCTOU (time-of-check time-of-update)!
{:.error}

# The TOCTOU Bug

The time-of-check to time-of-use (often abbreviated as`TOCTOU`, `TOCTTOU` or `TOC/TOU`) is a class of software bug caused by a <span class="orange-bold">race condition</span> involving:

- The <span style="color:#f77729;"><b>checking</b></span> of the state of a part of a system (such as this check in `vulnerable_root_prog` using `access`),
- And the <span style="color:#f77729;"><b>actual use</b></span> of the results of that check

We <span style="color:#f77729;"><b>exaggerate</b></span> the `DELAY` between:

1. The time of <span style="color:#f77729;"><b>CHECK</b></span> of the file using `access` and
2. The time of <span style="color:#f77729;"><b>USE</b></span> (actual usage of the file) using `fopen`
   by setting `sleep(DELAY)` in between the two instructions, where `DELAY` is specified as 1 to simulate 1 second delay.

```java
...
    if (!access(fileName, W_OK))
    {
        printf("Access Granted \n");
        /*Simulating the Delay*/
        sleep(DELAY); // sleep for 1 sec, exaggerate delay
        ...

        int byte_index = 0;
        int previous_byte_index = 0;
        int byte_match = -1;

        FILE *fp = fopen(fileName, "r+");
        ...
    }
```

Consider the `vulnerable_root_prog` being called by a user to modify a text file <span style="color:#f77729;"><b>belonging</b></span> to the user account as such:
<img src="{{ site.baseurl }}/assets/images/lab2/11.png"  class="center_full no-invert"/>

The `access()` check of course grants the normal user caller to modify `userfile.txt` because indeed it <span style="color:#f77729;"><b>belongs</b></span> to the normal user (<span style="color:#f77729;"><b>ubuntu</b></span> in the screenshot above).

The output doesn't make sense as of now, but we will explain what those `hash` values are about really soon. The `vulnerable_root_prog` does two things:

1. Accepts a `fileName` and a string inside that file to `match`.
2. It will <span style="color:#f77729;"><b>replace</b></span> that string inside the file with the  following hash value if `access()` system call grants permission.

```sh
$6$jPSpZ3iS84semtGU$DLwyTleAM2Of8NzDrwwNTnuSamJlnTx6NlMgbhPT5L8POT/J1MSCPucOAp1Qt3zRClS2NWT.RksROF9R1XLrn0
``` 

## Symbolic Link

We will soon exploit this bug with <span style="color:#f77729;"><b>symbolic link</b></span>.

{:.new-title}
> Symbolic Link
> 
> A <span style="color:#f77729;"><b>symbolic</b></span> link is a special kind of file that points to (reference) another file, much like a <span style="color:#f77729;"><b>shortcut</b></span> in Windows or a Macintosh alias. It contains a text string that is <span style="color:#f7007f;"><b>automatically interpreted</b></span> and followed by the operating system as a path to another file or directory. You will learn more about this in the later weeks of lecture.

### Task 9

`TASK 9:` Creating symbolic link.
{:.task}

We can create a text file with the following command and output redirection: 

```sh
echo "good morning" > goodmorning.txt
```

Then we can create a <span style="color:#f77729;"><b>symbolic link</b></span> using the command:

```sh
ln -s <source> <symlink>
```

In this example below, we created a `goodmorning_symlink.txt` that <span style="color:#f77729;"><b>points</b></span> to the actual file `goodmorning.txt`:
<img src="{{ site.baseurl }}/assets/images/lab2/12.png"  class="center_full no-invert"/>

There are different terminologies for `ln`. POSIX states `target --> source` while GNU states `link_name --> target`, and hence the word _target_ alone can mean either the symlink or the actual file depending on which manual you read. <span class="orange-bold">Be careful!</span>
{:.warning}

## Exploiting the TOCTOU Bug with SymLink

During this <span style="color:#f77729;"><b>delay</b></span> between <span style="color:#f77729;"><b>checking</b></span> (with `access`) and <span style="color:#f77729;"><b>usage</b></span> (with `fopen`):

1. A <span style="color:#f7007f;"><b>malicious</b></span> attacker can <span style="color:#f7007f;"><b>replace</b></span> the actual file `text.txt` into a <span style="color:#f77729;"><b>symbolic link</b></span> pointing to a protected file, e.g: `/etc/shadow`
2. Since `fopen` only checks <span style="color:#f77729;"><b>effective</b></span> user ID, and `vulnerable_root_prog` has its `SUID` bit <span style="color:#f77729;"><b>set</b></span> (runs <span style="color:#f77729;"><b>effectively</b></span> as `root` despite being called by only normal user), the “supposedly secure” <span style="color:#f77729;"><b>rootprog</b></span> can end up allowing normal user to gain elevated privileges to <span style="color:#f77729;"><b>MODIFY</b></span> protected file like `/etc/shadow`.

In the screenshot below, we created a <span style="color:#f77729;"><b>symbolic link</b></span> `userfile.txt` to point to `/etc/shadow`, resulting in the regular user being unable to `cat userfile.txt`.
<img src="{{ site.baseurl }}/assets/images/lab2/13.png"  class="center_full no-invert"/>

We have written the **command** to create the symbolic link for you. It is inside `/User/exploit.sh`.

## Race Condition

The malicious attacker has to <span style="color:#f77729;"><b>attack</b></span> and can only <span style="color:#f77729;"><b>successfully</b></span> launch the attack (modifying `userfile.txt -> /etc/shadow`) during that <span style="color:#f77729;"><b>time window</b></span> between <span style="color:#f77729;"><b>time-of-check</b></span> (`access`) and <span style="color:#f7007f;"><b>time-of-use</b></span> (`fopen`), hence the term “race condition vulnerability attack” or “a bug caused by race condition”.

The attacker has to <span style="color:#f7007f;"><b>RACE</b></span> with the `vulnerable_root_prog` to <span style="color:#f77729;"><b>quickly</b></span> change the `userfile.txt` into a symbolic link pointing to `/etc/shadow`. This can happen <span style="color:#f77729;"><b>ONLY</b></span> on this very specific time window of <span style="color:#f77729;"><b>AFTER</b></span> the `access()` check and <span style="color:#f77729;"><b>BEFORE</b></span> the `fopen()`.
{:.important}

# The Attack

## Modify `/etc/shadow` 

We will use this TOCTOU bug in `vulnerable_root_prog` to <span style="color:#f77729;"><b>modify</b></span> the content of the file: `/etc/shadow`.

`/etc/shadow` is <span style="color:#f77729;"><b>shadow</b></span> password file; a system file in <span style="color:#f77729;"><b>Linux</b></span> that stores <span style="color:#f77729;"><b>hashed</b></span> user passwords and is accessible only to the root user, preventing unauthorized users or malicious actors from breaking into the system.
{:.info}

Try reading it using `sudo cat /etc/shadow`. You will find that the lines have the following format:

```sh
usernm:$y$Apj1GQn.U$ZWWEtt19A8:17736:0:99999:7:::
[----] [---------------------] [---] - [---] ----
|                 |              |   |   |   |||+-----------> 9. Unused
|                 |              |   |   |   ||+------------> 8. Expiration date
|                 |              |   |   |   |+-------------> 7. Inactivity period
|                 |              |   |   |   +--------------> 6. Warning period
|                 |              |   |   +------------------> 5. Maximum password age
|                 |              |   +----------------------> 4. Minimum password age
|                 |              +--------------------------> 3. Last password change
|                 +-----------------------------------------> 2. Hashed Password
+-----------------------------------------------------------> 1. Username
```

Pay attention to segment <span style="color:#f77729;"><b>2</b></span>
{:.important}

It contains the password of the `usernm` using the `$type$salt$hashed` format. 1.`$type` is the method cryptographic hash <span style="color:#f77729;"><b>algorithm</b></span> and can have the following values:

- `$1$` – MD5
- `$2a$` – Blowfish
- `$y$` – yescrypt
- `$5$` – SHA-256
- `$6$` – SHA-512

If the hashed password field contains an asterisk (\*) or exclamation point (!), the user will not be able to login to the system using password authentication. Other login methods like key-based authentication or switching to the user are still allowed (out of scope). You can read more about the file [here](https://www.cyberciti.biz/faq/understanding-etcshadow-file/) but it's not crucial. Let's move on.

## Replacing Hashed Password in `/etc/shadow`

We <span style="color:#f7007f;"><b>aim</b></span> to:

1. Replace the targeted `username` entry in `/etc/shadow` with a <span style="color:#f7007f;"><b>new Hashed Password</b></span> section
2. So that we can `login` to this targeted `username` using <span style="color:#f7007f;"><b>preset password</b></span> `00000` (five zeroes), effectively <span style="color:#f77729;"><b>overriding</b></span> the password you set earlier.
3. "We" being <span style="color:#f77729;"><b>regular user account</b></span> (NOT root!) but with the help of this TOCTOU bug from `vulnerable_root_prog`.

These targeted `username` is none other than `test-user-0` that you have created earlier.
{:.highlight}

### Task 10

`TASK 10:` Launch attack that does the 3 things above by running `exploit.sh`.
{:.task}

While logged in as a regular user (your original username), change your directory to `/User/` and run the script `exploit.sh`:

```sh
cd User
./exploit.sh
```

<img src="{{ site.baseurl }}/assets/images/lab2/17.png"  class="center_full no-invert"/>

The program will run for <span style="color:#f77729;"><b>awhile</b></span> (20-30 secs) and eventually <span style="color:#f7007f;"><b>stop</b></span> with such message:

<img src="{{ site.baseurl }}/assets/images/lab2/18.png"  class="center_full no-invert"/>

If you do a `cat /etc/shadow` right now, notice how `test-user-0` account hashed password section has been changed to match that `replacement_text` in `vulnerable_root_prog`:

<img src="{{ site.baseurl }}/assets/images/lab2/19.png"  class="center_full no-invert"/>

## Login to target user account

### Task 11

`TASK :` Login to user account `test-user-0` with password `00000`
{:.task}

Now you can login to `test-user-0` account with <span style="color:#f77729;"><b>password</b></span>: `00000` instead of your originally set password:

<img src="{{ site.baseurl }}/assets/images/lab2/20.png"  class="center_full no-invert"/>

We have <span style="color:#f77729;"><b>successfully</b></span> changed a supposedly <span style="color:#f7007f;"><b>protected</b></span> `/etc/shadow` file while logged in as a <span style="color:#f77729;"><b>regular user</b></span> (ubuntu in the example above).

## What Happened?

Open `exploit.sh` file inside `/User/` directory:

```bash
#!/bin/sh
# exploit.sh

# note the backtick ` means assigning a command to a variable
OLDFILE=`ls -l /etc/shadow`
NEWFILE=`ls -l /etc/shadow`

# continue until THE ROOT_FILE.txt is changed
while [ "$OLDFILE" = "$NEWFILE" ]
do
    rm -f userfile.txt
    # create userfile again
    cp userfile_original.txt userfile.txt

    # the following is done simultanously
    # if a command is terminated by the control operator &, the shell executes the command in the background in a subshell.
    # the shell does not wait for the command to finish, and the return status is 0.
    # on the other hand, commands separated by a ; are executed sequentially; the shell waits for each command to terminate in turn.
    # the return status is the exit status of the last command executed.
    ../Root/vulnerable_root_prog userfile.txt test-user-0 & ln -sf /etc/shadow userfile.txt & NEWFILE=`ls -l /etc/shadow`

done

echo "SUCCESS! The root file has been changed"
```

The first two lines create a variable called `OLDFILE` and `NEWFILE`, containing the file information of `/etc/shadow`.

```sh
OLDFILE=`ls -l /etc/shadow`
NEWFILE=`ls -l /etc/shadow`
```

Then, the main loop is repeated <span style="color:#f77729;"><b>until</b></span> `/etc/shadow` is successfully changed (different <span style="color:#f77729;"><b>timestamp</b></span> and <span style="color:#f77729;"><b>size</b></span>):

1. Remove existing `userfile.txt` if any (from previous loop)
2. Then, create `userfile.txt` (this line can be replaced by any other instructions that create this `userfile.txt`)
3. Then, runs 3 commands in <span style="color:#f77729;"><b>succession</b></span>:
   1. `../Root/vulnerable_root_prog userfile.txt test-user-0`: runs the vulnerable program with `userfile.txt`, belonging to currently logged in user account and the <span style="color:#f77729;"><b>targeted</b></span> username.
   2. `ln -sf /etc/shadow userfile.txt`: immediately, `fork` and execute the `ln` program to change `userfile.txt` to <span style="color:#f77729;"><b>point</b></span> to `/etc/shadow`. <span class="orange-bold">Remember</span> from Lab 1 that a shell spawns a **new** child process upon execution of commands. 
   3. `NEWFILE=ls -l /etc/shadow`: check the file **info** (not <span class="orange-bold">content</span>!) of `/etc/shadow`and store it into variable `NEWFILE`; to be used in the <span style="color:#f77729;"><b>next</b></span> loop check
4. It terminates the moment the metadata of the new file `NEWFILE` differs from `OLDFILE`: this means `/etc/shadow` has been **recently** modified which indicates that our attack is successful (assuming no other process is currently changing `/etc/shadow`)

Two processes from step (3a) and (3b) above are <span class="orange-bold">racing</span>, and the script terminates when `/etc/shadow` has been successfully changed.

# Preventing the TOCTOU Bug
Preventing TOCTOU vulnerabilities involves strategies that either <span class="orange-bold">minimize</span> the window between the "check" and the "use" or <span class="orange-bold">avoid</span> the need for separate check and use steps altogether. Here are some general strategies for mitigating TOCTOU bugs:
1. **Using Atomic operations**: Atomicity ensures that a sequence of operations is completed in a single step from the perspective of other tasks. This approach is effective because it inherently eliminates the gap between the "check" (verification) and the "use" (action) phases of an operation, which is where the vulnerability would typically be exploited. 
2. **File Locking**: Use file locking mechanisms where appropriate. When a file is locked, other processes are prevented from modifying it until the lock is released. However, file locking *can introduce its own complexities and is not universally supported across all filesystems and platforms*.
3. **Avoid unnecessary checks**: Sometimes, the best way to avoid TOCTOU vulnerabilities is to avoid unnecessary checks. If you can perform an operation directly without checking first, especially if the operation is atomic or fails safely if preconditions are not met, you can often sidestep TOCTOU issues.
4. **Privilege management strategies**: These strategies focus on minimizing the risk and potential impact of TOCTOU vulnerabilities by controlling the privilege levels at which operations are performed, rather than directly eliminating the timing window between checking and using a resource.

Here we look at two trivial variations of approach (4) above: 

## `seteuid()`

One of the ways to <span style="color:#f77729;"><b>patch</b></span> this TOCTOU bug is to add just <span style="color:#f77729;"><b>one line of instruction</b></span> after `access()` (before `fopen()` is called) to manually set the effective UID of the process as the actual UID of the process.

This approach is more about privilege management rather than avoiding checks per se.
{:.info}

You can do this using the following system call: `seteuid(getuid());`

### Task 12

`TASK 12:` Modify `vulnerable_root_prog.c` to add that <span style="color:#f77729;"><b>one line</b></span> of code above right under the following if-clause. 
{:.task}

```cpp
if (!access(fileName, W_OK))
{
   ...
```

Then follow these steps:

1. <span style="color:#f77729;"><b>Login as root</b></span> and <span style="color:#f77729;"><b>recompile</b></span> with `make` inside `/FilesForRoot/`.
2. Login back as your original user account, and cd to `/User/` again, and run `exploit.sh`.
3. <span style="color:#f77729;"><b>Observe</b></span> the result
4. Press `ctrl+c` to cancel the script (yes, this change will cause step 2 to run in infinite loop).

## Disable `SUID`

Of course another way is to <span style="color:#f77729;"><b>disable</b></span> the SUID bit of `vulnerable_root_prog` altogether, however in practice sometimes this might not be ideal since there might be other parts of the program that temporarily requires execution with elevated privilege.

This is more of a preventative measure to limit the potential impact of vulnerabilities rather than a method to avoid unnecessary checks in the context of TOCTOU vulnerabilities.
{:.info}

### Task 13

`TASK 13:` Run `exploit.sh` with `root_prog_nosuid`.
{:.task}

Open <span style="color:#f77729;"><b>exploit.sh</b></span> and replace `vulnerable_root_prog` with `root_prog_nosuid`, and run the script again (while logged in as user account).


# Summary

By the end of this lab, we hope that you have learned:

- What <span style="color:#f77729;"><b>SUID</b></span> bit does, and how can it be utilised to gain elevated privileges to access protected files
- The differences between <span style="color:#f77729;"><b>root</b></span> and <span style="color:#f77729;"><b>normal</b></span> user
- The meaning of file <span style="color:#f77729;"><b>permission</b></span>. Although we do not go through explicitly on how it is set, you can read about it [here](https://kb.iu.edu/d/abdb) and experiment how to do it using the `chmod` command.
- How <span style="color:#f77729;"><b>race condition</b></span> happens and how it can be used as an <span style="color:#f77729;"><b>attack</b></span>
- How to <span style="color:#f77729;"><b>fix</b></span> (patch) the TOCTOU bug
