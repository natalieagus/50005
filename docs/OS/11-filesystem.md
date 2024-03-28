---
layout: default
permalink: /os/filesystem
title: File System
description: How UNIX file system is managed 
parent: Operating System
nav_order: 11
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

# File System

## File
A file is a <span style="color:#f77729;"><b>named</b></span> collection of related information that is stored on the secondary storage. It acts as a <span style="color:#f77729;"><b>logical</b></span> storage unit in the computer, defined by the operating system. In layman terms, they are a <span style="color:#f77729;"><b>group of data bytes</b></span> which are stored neatly in a known location with a unique name (<span style="color:#f77729;"><b>path</b></span>).
{:.info}

Files are mapped by the operating system onto physical devices that are non-volatile. Sometimes they are memory-cached for performance.

<span style="color:#f77729;"><b>Volatile</b></span> storage devices are unable to store any data after the power source is removed.

- Physical memory (RAM) is generally volatile.
- SSDs, HDDs, and thumbdrives are non-volatile.

In this chapter, we will specifically learn about <span class="orange-bold">UNIX</span> File system.
{:.note}

There are <span style="color:#f77729;"><b>two</b></span> types of file in the UNIX file system: regular files and directories. More information can be found in the later parts.

- <span style="color:#f77729;"><b>Directories</b></span>:
  - Has a structured namespace, e.g: /users/Desktop
- <span style="color:#f77729;"><b>Regular files</b></span>:
  - Can represent programs, or data
  - Can take any format: .txt, .bin, etc
  - Can contain any information defined by its creator

## File Example

Consider the file with name: `Banker.c` below.

1. The file consists of file <span style="color:#f77729;"><b>attributes</b></span> and file <span style="color:#f77729;"><b>content</b></span>.
2. File attributes (stored in _inode_) contain <span style="color:#f77729;"><b>important</b></span> metadata such as name, size, datetime of creation, user ID, etc.
3. Its <span style="color:#f77729;"><b>content</b></span> is a group of data bytes (~12KB) containing instructions to the Banker's algorithm written in C.

<img src="{{ site.baseurl }}/assets/images/week6/1.png"  class="center_fifty"/>

When we use this file, we _don’t care_ about its physical address (where it is actually stored on disk). We only care about its <span style="color:#f77729;"><b>path</b></span>. The path is a “logical” storage unit.
{:.info}

## Content Structure

File content structure that can exist in your computer varies greatly:

- None: **uninterpreted** sequence of bytes
- Simple:
  - **Lines** (text files)
  - Fixed Length Records (structured database, e.g: NRIC)
  - Variable Length Records (unstructured database, e.g: audio, video, books, are all of variable length)
- Complex Structure:
  - **Executable** files (`ELF` files)

## File Format and Type

File format is the actual data structure of a certain file, defining the <span style="color:#f77729;"><b>syntax</b></span> (permitted values, formal structure or grammar) and <span style="color:#f77729;"><b>semantics</b></span> (meaning and interpretation) of data within a file.

File _type_ is a group of file formats that serve similar functions. For instance: arc, zip, and tar are all the format of archive file type. The image below illustrates the common examples (don't have to memorise them, image taken from SGG book):

<img src="{{ site.baseurl }}/assets/images/week6/2.png"  class="center_fifty"/>

You can tell that different files have different formats from its _extension_. Some OS replaces the icons of the files depending on its extension to give us a <span style="color:#f77729;"><b>visual representation</b></span> of the file format.

### File Extension vs File Format

A file extension is the <span style="color:#f77729;"><b>character</b></span> or <span style="color:#f77729;"><b>group</b></span> of characters after the period that makes up an entire <span style="color:#f77729;"><b>file name</b></span>. It often indicate the file type, or file format, of the file, but <span style="color:#f7007f;"><b>not always</b></span>. We usually add it with a dot in the file name, e.g: `grades.xls`

- The file extension helps an OS to determine which program on your computer the file is <span style="color:#f77729;"><b>associated</b></span> with.
- For example, files ending with .xls will be associated with Excel -- opening it will open Excel and _load_ that file into Excel.

Any file's extension can be renamed, but that <span style="color:#f7007f;"><b>won't convert</b></span> the file to another <span style="color:#f7007f;"><b>format</b></span> or change anything about the file <span style="color:#f7007f;"><b>other than this portion of its name</b></span>.
{:.error}

Hence, a file with a PDF <span style="color:#f7007f;"><b>format</b></span> will <span class="orange-bold">not</span> be converted into a ZIP format by simply changing it's name from `submission.pdf` to `submission.zip`. You will need a _tool_ (e.g: the `zip` system program) to **convert** its format into an actual `ZIP` formatted file.

**Another example**: a file named `grades.csv` contains an extension that is appropriately related to its actual format: comma separated values. A computer user could rename that file to `grades.mp3`, however that wouldn't mean you could play the file as some sort of audio on a media player. The file itself is still rows of text (a CSV file), not a compressed musical recording (an MP3 file).

## File Interpreters

Who can read and understand (interpret) the content of the file?

- **The OS Kernel**:
  - UNIX-based OS implements <span style="color:#f77729;"><b>directories</b></span> as special files
  - It knows the <span style="color:#f77729;"><b>difference</b></span> between directory & regular files;
  - It supports <span style="color:#f77729;"><b>executable</b></span> files
  - Doesn’t otherwise require/interpret any formats of other regular files
- **System Programs**:
  - <span style="color:#f77729;"><b>Linker</b></span> or <span style="color:#f77729;"><b>loader</b></span> that knows `ELF` formatted files
  - <span style="color:#f77729;"><b>Compilers</b></span> that know certain file script formats (e.g: `.c` for `gcc`)
- **User/Application Programs**:
  - Web browser can interpret HTML, web-assembly format
  - Video players can interpret .mp4 format
  - PDF readers can interpret .pdf format
  - Excel can interpret .xls format, along with .csv

# File Abstract Data Type

A file can be seen as an <span style="color:#f77729;"><b>abstract data type</b></span>, that will be implemented differently depending on the OS supporting it. Some file structures are universal (can be identified by any OS and the OS can find a suitable interpreter app in the system), such as .txt and .pdf files. However, some others are system specific.

Regardless of the actual structural detail, we can abstract the concept of file as having file <span style="color:#f7007f;"><b>attributes</b></span> and file <span style="color:#f7007f;"><b>interface</b></span>.
{:.note}

## File Attributes

A file has <span style="color:#f77729;"><b>attributes</b></span> (metadata), analogous to how a class/object has states. These include:

- <span style="color:#f77729;"><b>Name</b></span>:
  - The symbolic file name is the only information kept in human readable form (stored in directory)
- <span style="color:#f77729;"><b>Identifier</b></span>:
  - This unique tag,usually a number, identifies the file within the file system; it is the non-human-readable name for the file (stored in inode, also known as inode id)
- <span style="color:#f77729;"><b>Type/Format</b></span>:
  - This information is needed for systems that support different types of files.
- <span style="color:#f77729;"><b>Location</b></span>:
  - This information is a pointer to a device and to the location of the file on that device.
- <span style="color:#f77729;"><b>Size</b></span>:
  - The current size of the file (in bytes, words, or blocks) and possibly the maximum allowed size are included in this attribute.
- <span style="color:#f77729;"><b>Protection</b></span>:
  - Access-control information determines who can do reading, writing, executing, and so on.
- <span style="color:#f77729;"><b>Time, date, and user identification</b></span>:
  - This information may be kept for creation, last modification, and last use. These data can be useful for protection, security, and usage monitoring.

You can use the command `ls -ali` to list all files in the current directory, together with its identifier (inode number, will be explained in the later section):

<img src="{{ site.baseurl }}/assets/images/week6/3.png"  class="center_fifty no-invert"/>

You should know how to interpret the above file permission output from Lab 1.
{:.info}

## File Interface

We can perform <span style="color:#f77729;"><b>operations</b></span> on files, and these operations are possible through the file interface or methods, analogous to the interface/methods we implement for classes in OOP. Operations that can be performed on files are: create, read, write, delete, reposition, and truncate among many others.

### File Creation

To create a file, we need to ensure two things are done:

- A free <span style="color:#f77729;"><b>space</b></span> in the system storage must be found for the file.
- An <span style="color:#f77729;"><b>entry</b></span> for the new file must be made in the directory (details provided later).

### File Read and Write

In order to R/W to a file, you must first have the <span style="color:#f77729;"><b>permission</b></span> to `open` the file. From Lab 1: The creator of the file can modify such permissions using `chmod` command.

If permitted, you can then:
1. <span style="color:#f77729;"><b>Open</b></span> the file.
   - This can be done using system call `open()`. There are other methods of opening the file depending on your programming language.
2. Perform the read/write operation using its respective <span style="color:#f77729;"><b>file pointer</b></span>, to indicate where exactly in the file we want to write or read, _byte by byte_.
3. <span style="color:#f77729;"><b>Close</b></span> the file.
   - This can be done using system call `close()`.

### File Deletion

To delete the file, we need to search the <span style="color:#f77729;"><b>directory</b></span> to find the file given the file's path, and also remove it from the file system (more details later). This action also frees the space initially occupied by the file's content

### File Truncation

In file truncation, we want to keep all the file attributes but **remove** its content. We search the directory given the file's path. Once found, the file length is reset to be zero, and its file space for the content is released.

### File Reposition

This requires us to first have permission to `open` the file and gain a file pointer. And then we reposition the value of the file pointer to be pointing to other portion of the file (e.g: byte N of the file). This can be done in C using `lseek()`

### Combining File Operations

These basic operations can then be combined to perform more <span style="color:#f77729;"><b>complex</b></span> file operations. For instance, we can create a <span style="color:#f77729;"><b>copy</b></span> of a file by <span style="color:#f77729;"><b>creating</b></span> a new file and then <span style="color:#f77729;"><b>opening</b></span> and <span style="color:#f77729;"><b>reading</b></span> from the old file and <span style="color:#f77729;"><b>writing</b></span> its content to the new one.

# The File System

A file system controls how data is <span style="color:#f77729;"><b>stored</b></span> and <span style="color:#f77729;"><b>retrieved</b></span> in a system.
{:.info}

It is a <span style="color:#f77729;"><b>set of rules (and features)</b></span> applicable to determine the way the data is stored. A physical disc can be separated into multiple physical partitions, where each partition can have different file system

<span style="color:#f7007f;"><b>The purpose of a file system is to maintain and organize secondary storage.</b></span>
Without a file system, data placed in a storage medium would be one large body of data with no way to tell where one piece of data stops and the next begins.

The file system is able to <span style="color:#f77729;"><b>logically</b></span> separate the segments of data on disk and give each piece a unique name. Each group of data is the content of a file, and the file attributes may be stored <span style="color:#f77729;"><b>elsewhere</b></span> (details provided later). The structure and logic rules used to manage these groups of data and their names is called a "file system".

The File System operates using specific data structure and has a specific format. Its <span style="color:#f77729;"><b>interface</b></span> is part of the OS, so they vary between operating systems.

Examples of common file system includes (there's no need to memorise):

1. File allocation table (<span style="color:#f77729;"><b>FAT</b></span>) is supported by the Microsoft Windows OS
2. New Technology File System (<span style="color:#f77729;"><b>NTFS</b></span>) is the default file system for Windows products from Windows NT 3.1 OS onward
3. <span style="color:#f77729;"><b>ext4</b></span> is a file system for many Linux Distributions
4. Universal Disk Format (<span style="color:#f77729;"><b>UDF</b></span>) is a vendor-neutral file system used on optical media and DVDs
5. Hierarchical file system (<span style="color:#f77729;"><b>HFS</b></span>) was developed for use with Mac operating systems. HFS is succeeded by HFS+
6. Apple File System (<span style="color:#f77729;"><b>APFS</b></span>) for macOS

In 50.005, we shall focus on UNIX-specific file system and its data structure.

# UNIX File System Data Structure

The figure below illustrates a simplified UNIX file system data structure in-memory. All these data structures are implemented in the <span class="orange-bold">kernel</span> space:

<img src="{{ site.baseurl }}/assets/images/week6/4.png"  class="center_full"/>

## File Descriptor Table

The fd table exists per-process.
{:.important}

Whenever a process `open()` or `dup()` a file, the opened file is associated with a <span style="color:#f77729;"><b>file descriptor</b></span>.

- A file descriptor is a number that <span style="color:#f77729;"><b>uniquely</b></span> (unique in that process) identifies an open file in a computer's operating system
- A file descriptor has a file `pointer` field, that is a pointer to the <span style="color:#f7007f;"><b>system wide open-file-table</b></span>.

## System-Wide Open File Table (SWOFT)

The SWOFT is managed by the Kernel, contains a list of opened files and its entries created by `open()`.
{:.important}

Opened files in the system may include disk files, named pipes, network sockets and devices opened by <span style="color:#f77729;"><b>all</b></span> processes.

The fields of each opened files contains:

1. `cp`:current pointer offset, pointing to a specific byte in the file
2. <span style="color:#f77729;"><b>Access status</b></span>: such as read, write, append, execute, etc (not drawn)
3. <span style="color:#f77729;"><b>Open count</b></span>: how many fd table entries point to it. We cannot remove open file table entry if reference count is more than 0.
4. <span style="color:#f77729;"><b>Inode pointer</b></span>: a pointer to UNIX inode table.

## Inode Table

An Inode table is also known as an Index Node table or File Control Block (FCB): a <span style="color:#f7007f;"><b>database</b></span> of all file attributes and <span style="color:#f77729;"><b>location</b></span> of file contents.
{:.important}

It does <span style="color:#f77729;"><b>not</b></span> contain the content of any file, and it also does <span style="color:#f7007f;"><b>not</b></span> contain any file name. Each file is associated with an <span style="color:#f77729;"><b>inode</b></span> and has a <span style="color:#f77729;"><b>unique</b></span> inode number. You may view inode number of each file in the current directory using the command `ls -i`. 

File name (string) is stored in another file system cataloguing structure called <span style="color:#f77729;"><b>directory</b></span> (more details later).
{:.note}

## Example

This example is created for you to observe per-process <span style="color:#f77729;"><b>file descriptor table</b></span>. Suppose there’s a <span style="color:#f77729;"><b>blocking</b></span> C-code as follows, compiled, and its binary output called out is stored at `/Users/natalie_agus/Desktop`:

```cpp
// example.c
#include <stdio.h>
#include <stdlib.h>

int main(){
   char str1[20];

  printf("Enter name: ");
  scanf("%s", str1);

  printf("Name entered : %s \n", str1);
  fprintf(stdout, "EXIT! \n");
  return 0;
}
```

It is then saved as `example.c` and compiled as follows:

```
gcc -o out example.c
```

The blocking instruction `scanf` is there to make the process not terminate yet, so that we can have enough time to observe it's file descriptor table. Running this process <span style="color:#f77729;"><b>twice</b></span>, and then running the command `ps | grep ./out | grep -v grep` in the third terminal results in:

<img src="{{ site.baseurl }}/assets/images/week6/5.png"  class="no-invert center_full"/>

As you can see, there’s two processes `./out` at two terminals: `ttys009` and `ttys011`. We can examine its file descriptor table content with the command `lsof -p [pid]`. This is an example of how the <span style="color:#f77729;"><b>same</b></span> program can be used to create <span style="color:#f77729;"><b>multiple</b></span> processes,

There are 3 standard fd per process:

- <span style="color:#f77729;"><b>0u</b></span>: This is `stdin` (standard input stream), obtained from the terminal (eg: `/dev/ttys011`)
- <span style="color:#f77729;"><b>1u</b></span>: This is `stdout` (standard output stream), directed to the terminal
- <span style="color:#f77729;"><b>2u</b></span>: This is `stderr` (standard error stream), directed to the terminal as well

Currently, `scanf` is blocking as it is “listening” from `stdin` (fd 0). Whatever we typed at the terminal window can be passed to the process, and then passed back out to `stdout`. The function `printf()` automatically outputs to `stdout`. Otherwise, we can explicitly “ask” the program to _print_ to `stdout` using `fprintf(stdout, … )`.

Another observation that we can notice is that both processes are pointing to file with inode id `54133720`, that is the inode id of the binary `out` (the program code).

# File Usage Information

It is not a good idea to pass file name to every file system operation (e.g., `read()`, `open()`, `write()`, etc) because:

- Name can be really <span style="color:#f77729;"><b>long</b></span> and of variable length
- <span style="color:#f77729;"><b>Mapping</b></span> of name to internal data structures takes time

A process <span style="color:#f77729;"><b>uses</b></span> file descriptor (“handle” to the file) instead of file name to manipulate that file. Using system call, it can translate file name to its corresponding file descriptor at the beginning of a usage session.
{: .note}


File descriptor `fd` is an <span style="color:#f77729;"><b>index</b></span> (`int`) into a per-process file-descriptor (fd) table. Its numerical index has meaning only in the context of its process. You witness this in the above example.

```cpp
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/stat.h>
int main() {
     int fd_a, fd_b, n;
     char str[100];
     /* 1. open a file in read mode */
     fd_a = open("input.txt", O_RDONLY, 0);
     /* create an output file with read write permissions, 0666 is the file permission created, the 0 in front means OCTAL notation --- 666 is the octal notation

        if you want to create a directory, the same permission logic can be made also, but directories ARE EXECUTABLES:
            if (mkdir("myFolder", 0777) == -1)
                cerr << "Error :  " << strerror(errno) << endl;

            else
                cout << "Directory created!";
    */

    /* then, open a  file in RW mode */
     fd_b = open("output.txt", O_CREAT | O_RDWR, 0666);
     /* 2. read the data from input file and write it to output file */
     while ((n = read(fd_a, str, 10)) > 0) {
             write(fd_b, str, n);
     }
     /* 3. move the cursor to the 13th byte of the input file */
     lseek(fd_a, 12, 0);
     /* write the given text in output file */
     write(fd_b, "\nMoved cursor to 13th byte:\n", 28);
     /* writes the contents of input file from 12 byte to EOF */
     while ((n = read(fd_a, str, 10)) > 0) {
             write(fd_b, str, n);
     }
     /* 4. close both input and output files */
     close(fd_a);
     close(fd_b);
     return 0;
}
```

Details:
1. The system call `open()` with different arguments: one to _read_, the other to _write_.
   - Two file descriptors, <span style="color:#f77729;"><b>fd_a</b></span> and <span style="color:#f77729;"><b>fd_b</b></span> are returned from the system call. Now the program can read the content of `input.txt`.
   - `input.txt` is a text file with content “abcdefghijklmnopqrstuvwxyz” (the entire 26 alphabets, 26 bytes in content size).
2. We can perform `read` system call and `write` system call with opened files through `fd_a` and `fd_b`.
   - We no longer have to associate it with the name input.txt and output.txt
3. Afterwards, we <span style="color:#f77729;"><b>change</b></span> the current pointer (`cp`) of the file using system call `lseek`.
   - This updates the entry for this file in the system-wide opened file table. We move the pointer to point to the 13th byte, which is the letter ‘m’
4. Do not forget to <span style="color:#f77729;"><b>close</b></span> the file descriptor using the `close()` system call.

Upon execution, we found that the output.txt contains:

```bash
abcdefghijklmnopqrstuvwxyz

Moved cursor to 13th byte:
mnopqrstuvwxyz
```

# File System Mapping
There are two cases of file system mapping. 

<span style="color:#f77729;"><b>Case 1:</b></span> Multiple file descriptor table entries can point to the same system-wide open file table entry.
{:.note}

For example:

1. Two or more processes read the same file via:
   - File descriptors are created by one of them and then passed through socket, or
   - A child inheriting the parent’s file descriptor after `fork()`. Note that child and parent processes have a separated file descriptor table (note that they are two _different_, separate processes)
2. A single process that has two or more file descriptors referencing to the same file. In fact, you can do this by doing the system call `dup()` or `dup2()`

<span style="color:#f77729;"><b>Case 2:</b></span> Multiple system-wide open file table entries can point to the same file in the inode table.
{:.note}

We can do this by calling `open` to the <span style="color:#f77729;"><b>same file</b></span> from multiple different processes. The system call `open()` creates a <span style="color:#f77729;"><b>new entry</b></span> in the swoft. Both cases are considered as <span style="color:#f7007f;"><b>many-to-one</b></span> mapping.

## Example with `dup` and `dup2`

Study the following example. It illustrates <span style="color:#f77729;"><b>Case 1</b></span> mapping: 

```cpp
  int fd_a, fd_b, fd_c, n;
  char str;

  /* open an input file in read mode */
  fd_a = open("input.txt", O_RDONLY, 0);
  // fd_b will be the LOWEST available fd, which is 4
  fd_b = dup(fd_a);
  fd_c = dup2(fd_a, 9); // custom fd 9
```

At first, we `open(input.txt)`, and then <span class="orange-bold">duplicate</span> the returned file descriptor as `fd_b` and `fd_c`.

The difference between the two system calls is that `dup()` will return the <span style="color:#f77729;"><b>lowest</b></span> available `fd`, which is 4 (since 0, 1, and 2 are _reserved_ (set by default due to convention) as stdin, stdout, and stderr, and 3 is already used for open), while `dup2(old fd, new fd)` allows us to explicitly set the `new fd`.

If the new fd is <span style="color:#f77729;"><b>already in use</b></span> then the existing one will be closed first before being reused again. Note that fd 0, 1, and 2 can be closed or changed to point to another file as per other fds. The only difference is that upon process creation, these 3 fds are _already_ set as convenience to stdin, stdout, and stderr.

To observe a running process' fd table, we can make the process suspend (block) itself (as shown in previous section, use `scanf` or some blocking operation that waits for user input) and then print its file descriptor table content (use `ps` to get its `pid`, and then use `lsof -p pid`).

Since `fd_a`, `fd_b`, and `fd_c` are all sharing the <span style="color:#f77729;"><b>same pointer</b></span> to the system-wide open file table, their actions (read/write) <span style="color:#f7007f;"><b>affect each other</b></span>. The subsequent below illustrates this phenomenon:

```cpp
  /* read the data from fd_a */
  printf("From fd_a: ");
  int counter = 0;
  while ((n = read(fd_a, &str, 1)) > 0) {
          printf("%c", str);
          counter++;
          if (counter > 9) break;
  }
  printf("\n");

  /* read the data from fd_b */
  printf("From fd_b: ");
  counter = 0;
  while ((n = read(fd_b, &str, 1)) > 0) {
          printf("%c", str);
          counter++;
          if (counter > 9) break;
  }
  printf("\n");

  int fd_d = open("input.txt", O_RDONLY, 0);
  /* read back the data from fd_a */
  printf("From fd_d: ");
  counter = 0;
  while ((n = read(fd_d, &str, 1)) > 0) {
          printf("%c", str);
          counter ++;
          if (counter > 9) break;
  }
  printf("\n");
```

If we read the first 10 char from `fd_a`, `cp` in the swoft would've been advanced by 10 bytes. Subsequent 10 bytes of read by `fd_b` will read off from byte 10 to 19. Assuming that `input.txt` contains all 26 alphabets as mentioned above, the output of the program above will be:

```cpp
From fd_a: abcdefghij
From fd_b: klmnopqrst
From fd_d: abcdefghij
```

Note that fd_d is a <span style="color:#f77729;"><b>different entry</b></span> in the system-wide open file table because a read/write operation using `fd_a`, `fd_b`, and `fd_c` do not affect the read operation using fd_d. `fd_d` does not share the same **pointer** cp as both `fd_a`, `fd_b`, and `fd_c.
{: .note}


## File Descriptor Table Inherted During `fork`

A child process will inherit the parent's file descriptor table after `fork`. They will have a separate file descriptor table but pointing to the same entry(ies) in the system wide file descriptor table. This can be used as a way for parent and child processes to **communicate**.
{:.info}

For instance, suppose we have 3 text files: `input.txt`, `output.txt`, and `foo.txt` in the current working directory, and a `c` program as shown below. To check whether the two processes share a kernel resource, [we use `kcmp`](https://man7.org/linux/man-pages/man2/kcmp.2.html). You can read the `main` function straightaway to quickly understand what the sample code below does:

```cpp
#define _GNU_SOURCE
#include <sys/syscall.h>
#include <sys/wait.h>
#include <sys/stat.h>
#include <stdint.h>
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <linux/kcmp.h> // this code will only run on Linux, do not try it on macOS!


#define errExit(msg)    do { perror(msg); exit(EXIT_FAILURE); \
                        } while (0)

static int
kcmp(pid_t pid1, pid_t pid2, int type,
    unsigned long idx1, unsigned long idx2)
{
    return syscall(SYS_kcmp, pid1, pid2, type, idx1, idx2);
}

static void
test_kcmp(char *msg, pid_t pid1, pid_t pid2, int fd_a, int fd_b)
{
    printf("\t%s\n", msg);
    printf("\t\tkcmp(%jd, %jd, KCMP_FILE, %d, %d) ==> %s\n",
            (intmax_t) pid1, (intmax_t) pid2, fd_a, fd_b,
            (kcmp(pid1, pid2, KCMP_FILE, fd_a, fd_b) == 0) ?
                        "same" : "different");
}


int main(int argc, char const *argv[])
{
   // for blocking
   char str1[20];

   int fd_a, fd_b, fd_c;

   fd_a = open("input.txt", O_RDONLY, 0666);
   fd_b = open("output.txt", O_RDONLY, 0666);

   printf("Parent's pid is %d\n", getpid());
   printf("Originally we have fd_a: %d, fd_b: %d\n", fd_a, fd_b);

   pid_t pid;

   pid = fork();
   printf("Child pid is: %d\n", pid);

   if (pid < 0)
   {
       fprintf(stderr, "Fork has failed. Exiting now");
       return 1; // exit error
   }
   else if (pid == 0)
   {
      // child
      printf("This is child process with pid %d\n", getpid());
      printf("Originally fd_a: %d, fd_b: %d in child\n", fd_a, fd_b);
      close(fd_a);
      fd_c = open("foo.txt", O_RDWR, 0666);
      printf("After closing fd_a and opening foo.txt, we have fd_b: %d, fd_c: %d in child\n", fd_b, fd_c);
      printf("Blocking, this is when you might want to do lsof in another shell. Type something to exit: ");
      scanf("%19s", str1);
      printf("\nBye from child.");
      fflush(stdout);
      exit(0);
   }
   else
   {
      // parent
      sleep(1); // introduce artificial delay to increase the chance that child process reaches scanf first
      test_kcmp("\nCompare duplicate fd_b from different processes:",
                  getpid(),pid,  fd_b, fd_b);
      test_kcmp("\nCompare duplicate fd_a from different processes:",
                  getpid(),pid,  fd_a, fd_a);
      wait(NULL); // wait for any child to return
   }
   return 0;
}
```

Before both processes exit, we deliberately **block** both child and parent process with `scanf` and `wait` so that we have enough time to investigate the file descriptors of both processes using `lsof -p [pid]` as follows:
<img src="{{ site.baseurl }}//assets/images/week6-1_filesystem/2023-06-21-11-01-47.png"  class="center_full no-invert"/>

Upon successful `fork`, both parent and child will have `fd_a` and `fd_b` pointing to the <span class="orange-bold">same</span> open file table entry (which eventually points to `input.txt` and `output.txt` respectively). That means both parent and child `fd_a` will share the same `cp` (current pointer) in the open file table, and similarly for `fd_b`. However, the child process eventually `close(fd_a)` and open another file `foo.txt`. Since `open` assigns the lowest available `fd`, the value of `fd_c` is `3` (same as `fd_a`) but now it's pointing to `foo.txt` instead of `input.txt`.

The printout from `kcmp` confirms that `fd_b` (fd 4) in both processes point to the same resources (`output.txt`), but fd 3 (`fd_c` in child process, and `fd_a` in parent process) points to **different** resources (both different open file table entry and different resources). In fact, as long as two processes are pointing to a different open file table entry, `kcmp` will report it as **different** resources. You can confirm this by modifying the child process to `open` `input.txt` instead of `foo.txt`, `kcmp` will still report that they are pointing to different resources:

<img src="{{ site.baseurl }}//assets/images/week6-1_filesystem/2023-06-21-11-31-21.png"  class="center_full no-invert"/>

We have provided you with enough examples above, please try them out and do your own investigation.
{: .highlight}

<hr>

# Appendix

## UNIX inode

The inode (index node) is a data structure in a Unix-style file system that <span style="color:#f77729;"><b>describes a file-system object such as a file or a directory</b></span>, as shown in the image, taken from the SGG book. You can think of it as Unix kernel’s internal data structure to manage its file system. The kernel maintains the inode table (a collection of inodes). Each inode stores the attributes and disk block locations of the object's data.

<span style="color:#f7007f;"><b>The inode is the file, excluding its content</b></span>, which is identified by a unique <span style="color:#f7007f;"><b>inode number</b></span>. A filename on the other hand, is just metadata in the file system that refers to a file. A single file/inode can have multiple filenames referring to it (called links, which we will learn later).

<img src="{{ site.baseurl }}/assets/images/week6/10.png"  class="center_seventy"/>

## The Logical and Physical File system

The <span style="color:#f77729;"><b>logical</b></span> file system is the level of the file system at which users can request file operations by system call. It is responsible for interaction with the user application. This is typically what we see.

The <span style="color:#f77729;"><b>physical</b></span> file system contains the <span style="color:#f7007f;"><b>actual</b></span> data of the file system and the actual location. E.g: a huge file that is logically represented as one file is not necessarily physically stored near one another (it can be fragmented in the disk).

On disk, the physical file system contain information about how to:

- Boot an operating system stored there,
- The total number of blocks (recall blocksize from 50002)
- The number and location of free blocks,
- The directory structure, and
- Individual files location

The four main components of the entire file system is:

1. The <span style="color:#f77729;"><b>Boot</b></span> Control Block (per volume): contains information needed by the system to boot an operating system from that volume.

2. The <span style="color:#f77729;"><b>Volume</b></span> Control Block (per volume): contains volume (or partition) details, such as the number of blocks in the partition, the size of the blocks, a free-block count and free-block pointers, and a free-FCB count and FCB pointers.

3. The <span style="color:#f77729;"><b>Directory</b></span> Structure (per file system, it can span several volumes): used to organise the files.

4. The <span style="color:#f77729;"><b>File Control Block</b></span> (per file, also known as <span style="color:#f77729;"><b>inode</b></span> in UNIX): contains file attributes, has a unique id, and is associated with directory entry.

_Side note about partition vs volume: _

- Partition is always created on a single physical disk, however a volume can span multiple disks and have many partitions.
- Partitions only identified by numbers, but volumes have names.
- Finally, partitions are more suitable for individual devices, while volumes (especially logical volumes) are more flexible and suited for network attached storage.

<span style="color:#f77729;"><b>Example</b></span> on how user interacts with the logical file system to create a file (simplified):

1. An application program calls the <span style="color:#f77729;"><b>logical</b></span> file system to <span style="color:#f77729;"><b>create</b></span> a file, e.g: by right click >> create.
2. The <span style="color:#f77729;"><b>logical</b></span> file system traverse the directory, and check the _requested_ location.
3. To create a new file, it allocates a new <span style="color:#f77729;"><b>inode</b></span> or FCB.
4. The system then update the directory it with the new file name and FCB, and writes it back to the disk.

## Mounting File System during Boot

All systems have a <span style="color:#f77729;"><b>root</b></span> partition, which contains the operating-system kernel and sometimes other system files, which are mounted <span style="color:#f77729;"><b>at boot time</b></span>. The system has an <span style="color:#f77729;"><b>in-memory mount table</b></span> that contains information about each mounted volume. Other volumes can be automatically mounted at boot or manually mounted later, depending on the operating system.

<span style="color:#f77729;"><b>The mount procedure:</b></span>

1. The operating system is given the name of the <span style="color:#f77729;"><b>device</b></span> and the <span style="color:#f77729;"><b>mount point</b></span> — the location within the file structure where the file system is to be attached.

2. Typically, a mount point is an _empty directory._ For instance, on a UNIX system, a file system containing a user’s home directories might be mounted as `/home`; then, to access the directory structure within that file system, we could precede the directory names with `/home`,for example, `/home/alice`.

3. Mounting that file system under `/users` instead would result in the path name `/users/alice`, which we could use to reach the same content under `alice`.

The in-memory (means these are in RAM, as long as the computer is alive) information about file system that is loaded at mount time are the three data structures we show above and a few more:

1. <span style="color:#f77729;"><b>Mount-table</b></span>: contains information about each mounted volume. Notice the list of all mounted volumes in the screenshot below. Disk image, USB drives, all those are also mounted devices.
   <img src="{{ site.baseurl }}/assets/images/week6/11.png"  class="center_seventy"/>

2. <span style="color:#f77729;"><b>inode table</b></span>: Contains directory structure and file <span style="color:#f77729;"><b>pointers</b></span> to the actual data on secondary storage or cached memory

   - You can list the disk file directory and see how many inodes are present using `df -i`. You should be able to figure out by yourself what each column means, the leftmost one being the filesystem name.  
     <img src="{{ site.baseurl }}/assets/images/week6/12.png"  class="center_seventy"/>

3. <span style="color:#f77729;"><b>System-wide open file table</b></span>

   - You might not want to print this out even if you can, as there’s hundreds, even thousands of files that are probably opened right now.

4. Per-process <span style="color:#f77729;"><b>file descriptor table</b></span> (for currently alive processes)
   - You can check opened files for your current terminal process opened files using `lsof -p $$` command.
     <img src="{{ site.baseurl }}/assets/images/week6/13.png"  class="center_seventy"/>

_Details:_

```
FD –  the file descriptor. Some of the values of FDs are,
    cwd – current Working Directory
    txt – text file
    mem – memory mapped file
    mmap – memory mapped device
    <number> – the actual file descriptor.
      the character after the number i.e ‘1u’ the MODE in which the file is opened:
      r: read,
      w: write,
      u: read and write
TYPE – specifies the type of the file. Some of the values of TYPEs are,
REG – regular File
DIR – directory
FIFO – first in first out policy
CHR – character special file
NODE - the inode id that it is pointing to.
```