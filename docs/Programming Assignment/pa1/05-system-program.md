---
layout: default
permalink: /pa1/part5
title: System Programs
description: Implement more system programs
parent: Programming Assignment 1
grand_parent: Programming Assignment
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

# System Programs 
{: .no_toc}

{:.task}
In this part, you are tasked to create 4 **more** system programs:  **sys** (simple system information), **dspawn** (daemon spawn), **dcheck** (daemon check), and **backup** (zip certain dir, and move the zipped dir to `./archive` as backup). 

# `sys` 

The `sys` system program prints out basic information about your operating system as follows: 

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/05-system-program/2024-04-11-11-51-34.png"  class="center_full no-invert"/>

You are free to print out the information in any format. It should include some information about the Operating System, kernel, total memory size, user currently logged-in, and CPU. You are free to add any additional information you deem fit. This system program is intended to mimic the functionality of [`neofetch`, a command-line system information tool](https://github.com/dylanaraps/neofetch?tab=readme-ov-file):

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/05-system-program/2024-04-11-11-53-35.png"  class="center_full no-invert"/>

{:.important}
You should place `sys` source file inside `[PROJECT_DIR]/source/system_programs/`. The `makefile` will <span class="orange-bold">automatically</span> detect this new script and **compile** it into `[PROJECT_DIR]/bin` 

# `dspawn` 

The `dspawn` system program <span class="orange-bold">summons</span> a daemon process and then terminates so that the shell may continue to display the next prompt. This is unlike other programs where the shell waits for it to finish before printing the next prompt.

{:.note-title}
> Daemon Processes
> 
> **Daemons** are processes that are typically started when the system is bootstrapped and terminate only when the system is shut down. They <span style="color:#f77729;"><b>don’t have a controlling terminal</b></span> and they run in the <span style="color:#f77729;"><b>background</b></span>. In other words, a daemon is a computer program that runs as a background process, rather than being under the direct control of an interactive user.

Your system should be running multiple daemon processes right now. As mentioned, these daemon processes are background services that start at boot time or when needed, running <span class="orange-bold">continuously</span> to perform or manage system tasks and services without direct user interaction. 

Examples of such daemons include:
1. `sshd`, which handles Secure Shell (SSH) connections allowing secure remote access to the system; 
2. `httpd` or `nginx`, which serve web pages and manage HTTP traffic; 
3. `crond`, responsible for executing scheduled tasks;  
4. `syslogd`, which collects and manages system logs. 

These daemons are essential for the regular operation, maintenance, and security of a Linux system, often starting with system initialization through scripts managed by the system's init system (such as systemd, SysVinit, or Upstart). They typically run with <span class="orange-bold">elevated</span> privileges (e.g: root privileges) to perform tasks that regular users cannot, ensuring smooth and secure operation of both servers and desktops.

To list daemon processes on a Linux system using the ps command, you can look for processes that do not have a controlling terminal (`tty` or `pts`). Traditionally, the process names of a daemon end with the letter `d`, for clarification that the process is in fact a daemon, and for differentiation between a daemon and a normal computer program. You can use the command `ps axo tty,pid,ppid,comm | grep '[d]$ | sort -k 4` to try listing out some daemons in your system: 

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/05-system-program/2024-04-11-12-08-47.png"  class="center_full no-invert"/>

For the sake of our lab and our machine’s health, our daemon <span style="color:#f77729;"><b>terminates</b></span> after a certain period of time and _violates the traditional daemon definition_, but we’re sure you get the idea.
{:.info}

### Overgoogling

The basic information about daemons presented in this handout is <span style="color:#f7007f;"><b>sufficient</b></span>. Do not over-Google about Daemons unless you are really interested in it. The concept of daemons alone is very complex and large, and is out of our scope.


## Characteristics of a Daemon Process

A daemon process is still a normal process, running in <span style="color:#f7007f;"><b>user mode</b></span> with certain characteristics which distinguish it from a normal process.

The characteristics of a daemon process are listed below.

### No controlling terminal

By definition, a daemon process <span style="color:#f7007f;"><b>does not require direct user interaction</b></span> and therefore must detach itself from any controlling terminal.
{:.important}

In the `ps -ef` output, if the `TTY` or `TT` column is listed as a `?` meaning it does not have a controlling terminal.

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/05-system-program/2024-04-11-12-10-52.png"  class="center_full no-invert"/>

{:.info}
You might have heard about `pts` as well.  PTS (Pseudo Terminal Slave) and TTY (Teletype) are both types of terminal interfaces in Unix-like operating systems, but they serve different purposes and operate in slightly different ways. 

You can inspect which process uses `pts` and which uses `tty` in your current shell session with the following command `ps a -o pid,ppid,tty,comm`:
<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/05-system-program/2024-04-11-12-18-11.png"  class="center_full no-invert"/>

### PPID is Typically 1 or 2

The PPID of a daemon process is either 1 or 2, meaning that whoever was creating the daemon process must <span style="color:#f7007f;"><b>terminate</b></span> to let the daemon process be <span style="color:#f77729;"><b>adopted</b></span> by the `init` process (or equivalent).

Although it is common, not all systems assign the `initd` process to adopt orphaned process. More modern linux distros uses `systemd`, `kthreadd`, or `launchd` (or other designated descendant processes or equivalent) to <span class="orange-bold">adopt</span> all orphaned process. The pid of `initd` or `systemd` or equivalent processes is also **not always 1**. As long as your daemon process' ppid is the same as the pid of `init` _or_ pid of other special instances of `init` _or_ pid of `systemd` and equivalent, it is **acceptable** as long as it is **consistent** (that is the same designated system process is always adopting your daemon processes). When we test it in our system, we _know_ exactly what process will adopt your daemon process, so you don't need to worry.
{:.note}

To check the details of the processes with PID 1 and PID 2 on a Linux system, you can use the following command `ps -f -p 1,2` (note that your output might differ):
<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/05-system-program/2024-04-11-12-12-47.png"  class="center_full no-invert"/>

### Working directory: root

The working directory of the daemon process is typically the `root` (/) directory.

### Closes all uneeded file descriptors

It closes all unneeded file descriptors. In <span style="color:#f77729;"><b>UNIX</b></span> and related computer operating systems, a file descriptor (FD, less frequently fildes) is an abstract indicator (handle) used to access a file or other input/output resource, such as a pipe or network socket.

Also, it <span style="color:#f77729;"><b>closes</b></span> and <span style="color:#f77729;"><b>redirect</b></span> fd 0, 1, and 2 to `/dev/null`.
{:.important}

### Logging

It logs important messages through a <span style="color:#f77729;"><b>central</b></span> logging facilities, such as the BSD `syslog`. 

{:.info}
For instance, Ubuntu uses the systemd system and service manager, which includes systemd-journald, a service that collects and manages journal entries from all parts of the system. However, rsyslog is still present and integrates with systemd-journald to provide traditional log file management.

You can view the system log using the command `journalctl -o short-precise` in Linux distros: 
<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/05-system-program/2024-04-11-12-31-09.png"  class="center_full no-invert"/>

In macOS, you can use the following command instead `log show --last 1s`:
<img src="{{ site.baseurl }}/docs/Programming Assignment/pa1/images/05-system-program/2024-04-11-12-29-49.png"  class="center_full no-invert"/>

{:.note}
For our toy daemon, we should log to a designated log file, such as `[PROJECT_DIR]/dspawn.log`.

### Privileges
Some daemons are running with elevated privileges depending on its task. We are <span class="orange-bold">not</span> going to do this for security reasons. We let our daemon run with regular user privileges.

## Implementation Notes 

### Incantation
The general steps to summon a daemon process inside `dspawn` main function is as follows:
 1. `fork()` from the parent process (`dspawn`)
 2. Close parent with `exit(1)`
 3. On child process (this is intermediate process), call `setsid()` so that the child becomes **session** leader to **lose** the controlling TTY
 4. Ignore `SIGCHLD`, `SIGHUP`
 5. `fork()` again, then parent (the intermediate) process terminates
 6. Child process (the daemon) set new file permissions using `umask(0)`. Daemon's PPID at this point should be 1 or 2 (the initd or equivalent)
 7. Change working directory to `root`
 8. Close all open file descriptors using `sysconf(_SC_OPEN_MAX)` and redirect fd `0`,`1`,`2` to `/dev/null`
 9. Execute `daemon_work()`, this function supposedly never returns, but we will modify it to terminate after some time 

#### Step 1: first `fork`

The `fork()` splits this process into <span style="color:#f77729;"><b>two</b></span>: the <span style="color:#f77729;"><b>parent</b></span> (process group leader) and the <span style="color:#f77729;"><b>child</b></span> process (that we will call <span style="color:#f7007f;"><b>intermediate</b></span> process for the next few sections).

The reason for this `fork()` is so that `dspawn` returns immediately to `cseshell` and our shell does not wait for the daemon to exit. Daemons are background processes that do not exit until the system is shut down. We don’t want our shell to wait forever.

#### Step 2: exit `dspawn`

At this point, the shell will return while the <span style="color:#f77729;"><b>intermediate process</b></span> proceed to spawn the daemon.

#### Step 3: `setsid`

The child (<span style="color:#f77729;"><b>intermediate</b></span> process) process is by default not a process group leader. It calls `setsid()` to be the <span style="color:#f77729;"><b>session leader</b></span> and <span style="color:#f77729;"><b>loses controlling</b></span> TTY (terminal).

`setsid()` is only <span style="color:#f77729;"><b>effective</b></span> when called by a process that is not a process group leader. The `fork()` in step 1 ensures this. The system call `setsid()` is used to create a new session containing a single (new) process group, with the current process as both the session leader and the process group leader of that single process group. You can read more about setsid() [here](http://man7.org/linux/man-pages/man2/setsid.2.html).

#### Group and Session leader

Compile and try this code:

```cpp
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>


int main(){
   pid_t pid = fork();
   if (pid == 0){
       printf("Child process with pid %d, pgid %d, session id: %d\n", getpid(), getpgid(getpid()), getsid(getpid()));
       setsid(); // child tries setsid
       printf("Child process has setsid with pid %d, pgid %d, session id: %d\n", getpid(), getpgid(getpid()), getsid(getpid()));

   }
   else{
     printf("Parent process with pid %d, pgid %d, session id :%d\n", getpid(), getpgid(getpid()), getsid(getpid()));
       setsid(); // parent tries setsid
       printf("Parent process has setsid with pid %d, pgid %d, session id: %d\n", getpid(), getpgid(getpid()), getsid(getpid()));
       wait(NULL);

   }
   return 0;
}
```

It results in such output:

<img src="{{ site.baseurl }}/docs/Programming Assignment/pa1/images/05-system-program/10.png"  class="center_seventy no-invert"/>

Let's analyse them <span style="color:#f77729;"><b>line by line</b></span>.

<span style="color:#f77729;"><b>The first line:</b></span> The parent process has pid == pgid, that is `75614`.

- This tells us that this process ‘iddemo’ is the process group leader, but <span style="color:#f77729;"><b>not</b></span> a session leader since the session id `27063` is not equal to the pid `75614`.

<span style="color:#f77729;"><b>The third line:</b></span> When process `75614` <span style="color:#f77729;"><b>forks</b></span>, it has a child process with pid `75615`. It is clear that since child `pid != pgid`, then the child process is <span style="color:#f7007f;"><b>not</b></span> a session leader and is not a group leader either.

So who is 27063? We can type the command ps -a -j and find a process with pid 27063. Apparently, it's the `zsh`, the shell itself, connected to the controlling terminal `s002`.

<img src="{{ site.baseurl }}/docs/Programming Assignment/pa1/images/05-system-program/11.png"  class="center_seventy"/>

<span style="color:#f77729;"><b>The third and fourth lines:</b></span> When <span style="color:#f7007f;"><b>both</b></span> the child and parent process attempt to call `setsid`,

- In the child process, `setsid` effectively makes the `pgid` and session `id` to be equal to its pid, `75615`.
- In the parent process, setsid has <span style="color:#f7007f;"><b>no effect</b></span> on the session id, since the manual states that setsid only sets the process to be the session and process group leader if it is called by a process that is <span style="color:#f7007f;"><b>not</b></span> a process group leader.

Thanks to Step 1, the effect of `setsid` in Step 3 works as intended, and our <span style="color:#f77729;"><b>intermediate process</b></span> now lose the controlling terminal (part of a requirement to be a daemon process).
{:.info}

#### Step 4: Ignore `SIGCHLD` and `SIGHUP`

This intermediate process is going to `fork()` one more time in Step 5 to create the <span style="color:#f7007f;"><b>daemon</b></span> process.

By ignoring `SIGCHLD`, our daemon process -- the child of this intermediate process <span style="color:#f7007f;"><b>will NOT be a zombie process</b></span> when it terminates. Normally, a child process will be a zombie process if the parent does not `wait` for it.

- Since `SIGCHLD` is ignored, when the daemon (child of this intermediate process) exits, it is reaped immediately.
- However, the daemon will outlive the parent process anyway, so it does not really matter. This step is just for "in case".

Also, this intermediate process is a <span style="color:#f77729;"><b>session leader</b></span> (from step 3, since we need to <span style="color:#f77729;"><b>lose</b></span> the controlling terminal).

- If we terminate a session leader, a `SIGHUP` signal will be received and the children of the session leader will be <span style="color:#f77729;"><b>killed</b></span>.
- We do not want our daemon (child of this process) to be killed, therefore we need to call `signal(SIGHUP, SIG_IGN)` first <span style="color:#f77729;"><b>before forking</b></span> in Step 5.

#### Step 5: second `fork`

The child of this intermediate process is the <span style="color:#f7007f;"><b>daemon</b></span> process.
{:.note}

The second fork, is useful for allowing the parent process (intermediate process) to terminate. This ensures that the child process is <span style="color:#f77729;"><b>not a session leader</b></span>.

{:.new-title}
> Think!
> 
> Why must you ensure that the daemon is not a session leader? Since a daemon has no controlling terminal, if a daemon is a session leader, an act of opening a terminal device will make that device the controlling terminal.

We do not want this to happen with your daemon, so this second `fork()` handles this issue. As mentioned above, before forking it is necessary to ignore `SIGHUP`. This prevents the child from being killed when the parent (which is the session leader) dies.

#### Step 6: umask(0)

The daemon process must set all new files created by it to have `0777` permission using umask(0). This is that the file it has created can be globally readable, writeable, and executable by any other processes.
{:.warning}

Setting the `umask` to `0` means that newly created files or directories created will have all permissions set, so any file created by this daemon can be accessed by any other processes because we can’t directly control the daemon anymore.

A umask of zero will cause all files to be created as permission `0777` or <span style="color:#f7007f;"><b>world-RW & executable</b></span>. Some system sets the permission as `0666` by default instead of `0777` for security reasons, and we don't want this!

How does setting `umask(0)` lands you with `0777` file permission?

`0777` actually stands for <span style="color:#f77729;"><b>octal</b></span> `777`. In C, the first 0 indicates octal notation, and you can translate the rest in binary: `111 111 111`, which means we will have `- rwx rwx rwx`, equivalent to global RW and executable for the file. If we want to restrict permission of <span style="color:#f77729;"><b>write</b></span> to only the <span style="color:#f77729;"><b>owner</b></span>, we can set `umask(022)` -- equivalent to having permission `0755`, with the binary: 111 101 101, which translates to `- rwx r-x r-x`.
{:.info}

The manual for `umask` can be found [here](http://man7.org/linux/man-pages/man2/umask.2.html).

#### Step 7: `chdir` to root directory

Change the <span style="color:#f77729;"><b>current</b></span> working directory to root using `chdir("/")`. If a daemon were to leave its current working directory unchanged then this would prevent the filesystem containing that directory from being <span style="color:#f7007f;"><b>unmounted</b></span> while the daemon was running. It is therefore good practice for daemons to change their working directory to a <span style="color:#f7007f;"><b>safe</b></span> location that will never be umounted, like root.

#### Step 8: handle fd 0, 1, 2 and close all unused fds

Close all open file descriptors and redirect `stdin`, `stdout`, and `stderr` (fd 0, 1, and 2 by default in UNIX systems) to `/dev/null` so that it won’t <span style="color:#f7007f;"><b>reacquire</b></span> them again if you mistakenly attempt to output to `stdout` or read from `stdin`.

Once it is running a daemon should <span style="color:#f77729;"><b>NOT</b></span> read from or write to the terminal from which it was launched. The simplest and most effective way to ensure this is to <span style="color:#f77729;"><b>close</b></span> the file descriptors corresponding to `stdin`, `stdout` and `stderr`. These should then be reopened, either to `/dev/null`, or if preferred to some other location.

There are two reasons for not leaving them closed:

1. To prevent code that refers to these file descriptors from failing
2. To prevent the descriptors from being reused when we call open() from the daemon’s code.

To <span style="color:#f77729;"><b>close</b></span> all opened file descriptors, you need to <span style="color:#f77729;"><b>loop through existing file descriptors</b></span>, and re-attach the first 3 fd’s using `dup(0)`. Note that `open()` and `dup()` will assign the <span style="color:#f77729;"><b>smallest</b></span> available file descriptor, in this case that is 0, 1, and 2 in sequence. You will learn more about these stuffs in the last OS chapter.

```cpp
   /* Close all open file descriptors */
   int x;
   for (x = sysconf(_SC_OPEN_MAX); x>=0; x--)
   {
       close (x);
   }

   /*
   * Attach file descriptors 0, 1, and 2 to /dev/null. */
   fd0 = open("/dev/null", O_RDWR);
   fd1 = dup(0);
   fd2 = dup(0);
```

#### Step 9: execute `daemon_work()` 

You are free to implement `daemon_work()` however way you like. Here's one example:

```c
char output_file_path[PATH_MAX];

static int daemon_work()
{
    // put your full PROJECT_DIR path here  
    strcpy(output_file_path, "[PROJECT_DIR]/dspawn.log"); 

    int num = 0;
    FILE *fptr;
    char *cwd;
    char buffer[1024];

    // write PID of daemon in the beginning
    fptr = fopen(output_file_path, "a");
    if (fptr == NULL)
    {
        return EXIT_FAILURE;
    }

    fprintf(fptr, "Daemon process running with PID: %d, PPID: %d, opening logfile with FD %d\n", getpid(), getppid(), fileno(fptr));

    // then write cwd
    cwd = getcwd(buffer, sizeof(buffer));
    if (cwd == NULL)
    {
        perror("getcwd() error");
        return 1;
    }

    fprintf(fptr, "Current working directory: %s\n", cwd);
    fclose(fptr);

    while (1)
    {

        // use appropriate location if you are using MacOS or Linux
        fptr = fopen(output_file_path, "a");

        if (fptr == NULL)
        {
            return EXIT_FAILURE;
        }

        fprintf(fptr, "PID %d Daemon writing line %d to the file.  \n", getpid(), num);
        num++;

        fclose(fptr);

        sleep(10);

        if (num == 10) // we just let this process terminate after 10 counts
            break;
    }

    return EXIT_SUCCESS;
}
```

<span class="orange-bold">Alternatively</span>, you can store the current working directory of `dspawn` in its main function first, before the first `fork()` so that you don't have to hardcode the `[PROJECT_DIR]` in `daemon_work()`. 

```c
// in main() of dspawn.c 

    // Setup path
    if (getcwd(output_file_path, sizeof(output_file_path)) == NULL)
    {
        perror("getcwd() error, exiting now.");
        return 1;
    }
    strcat(output_file_path, "/dspawn.log"); 
```

{:.important}
Note that you won't be able to simply do `fptr = fopen("./dspawn.log", "a")` in `daemon_work` because at this point, the daemon's current working directory is **root** (`/`). Since we did not elevate the privilege of the daemon process, we won't be able to create any new file at root directory. 

### Observed Output
You should be able to spawn a daemon process using `dspawn` command. The shell should almost immediately return with a new prompt:

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/05-system-program/2024-04-11-14-45-14.png"  class="center_full no-invert"/>

{:.note}
You should be able to run multiple `dspawn` command to spawn multiple daemon processes. All daemons should write the "log" to `[PROJECT_DIR]/dspawn.log`.

A log file called `dspawn.log` should appear at `[PROJECT_DIR]` with the following content:
<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/05-system-program/2024-04-11-14-47-56.png"  class="center_full no-invert"/>

You can see clearly that the daemon process is adopted by the `init` (or equivalent) process with PID of 1, and that it's current working directory is root `/`. The file `dspawn.log` is opened using fd 3. To check whether fd 0,1,2, are all pointed to `/dev/null` correctly, you can use the `lsof -p [DAEMON_PID]` command:

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/05-system-program/2024-04-11-14-51-25.png"  class="center_full no-invert"/>

{:.info}
In Linux distros, everything is a *file*. Hence, you can read any process' file descriptor directly using the command: `ls -l /proc/[PID]/fd`. `lsof` works on both Linux-based OS and macOS. 

If you spawn multiple daemon processes at the same time, the log file should show an interleaved printout of messages from each daemon process:

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/05-system-program/2024-04-11-14-58-07.png"  class="center_full no-invert"/>

# `dcheck` 

This system program simply checks how many live daemons (spawned from `dspawn`) are alive right now:

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/05-system-program/2024-04-11-14-56-19.png"  class="center_full no-invert"/>

## Implementation Notes

The easiest way to find out how many daemon processes spawned from `dspawn` are running right now is by using the `ps` command, and filtering its output with the name `dspawn` that doesn't have an output terminal:

{:.info}
The daemon process' name inherits `dspawn` command name, hence they both have the same name. 

```sh
ps -efj | grep dspawn  | grep -Ev 'tty|pts' 
```

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/05-system-program/2024-04-11-15-00-02.png"  class="center_full no-invert"/>

From the screenshot example above, we have two daemon processes that are alive. This corresponsds to `ps` output, with two lines (entries) reported. You can execute any command from a C program using the [system call API `system`](https://man7.org/linux/man-pages/man3/system.3.html). 

{:.new-title}
> hint    
>
> You might need to use some kind of output redirection to read and analyse the output of `ps` executed from your C script with `system`. 

# `backup`

The `backup` system program must be able to automatically zip a directory whose name matches environment variable `BACKUP_DIR`, and move this zipped directory to `[PROJECT_DIR]/archive/`. `BACKUP_DIR` must be set up by the shell before calling `backup`. The filename of the zipped file must include the correct datetime indicating when the `backup` command was executed. 

Here's an example of running `backup` before and after `BACKUP_DIR` environment variable is set:

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/05-system-program/2024-04-11-15-09-58.png"  class="center_full no-invert"/>

You are free to decorate the output of `backup` command, as long as the bare minimum information as shown above is included. 

It should also be able to zip a single file if `BACKUP_DIR` points to a file instead of a directory:

<img src="{{ site.baseurl }}//docs/Programming%20Assignment/pa1/images/05-system-program/2024-04-11-15-12-06.png"  class="center_full no-invert"/>

# Summary 

By the end of this task, your shell will have access to these 4 new system program at `[PROJECT_DIR]/bin`. 