---
layout: default
permalink: /os/directories
title: UNIX Directories
description: How UNIX directories are managed 
parent: Operating System
nav_order: 12
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

# UNIX Directories

{:.highlight-title}
> Detailed Learning Objectives
> 
> 1. **Explain Directories and Namespace Organization**
>   - Define what a directory is in the context of file systems, as well as its **differences** vs a regular file.
>   - Explain how directories help **organize** files and **subdirectories** within a **structured namespace**.
> - **Explain Directory Permissions and Operations**
>   - Examine the significance of `x`, `r`, and `w` permissions in directories.
>   - Identify the effects of directory permissions on file access and operations within the directory.
> - **Explain Directory Structures and Their Types**
>   - Compare and contrast different directory structures such as single-level, two-level, tree-structured, acyclic graph, and general graph directories.
>   - Discuss the benefits and limitations of **each** directory structure type.
> - **Explain Directory and File Linking Concepts**
>   - Differentiate between **hard** links and **symbolic** links.
>   - Discuss the use **cases** for each type of link and how they affect file management and system organization.
> - **Describe File Searching and Directory Traversal Process**
>   - Explain the general **process** of searching for files within directories.
>   - Describe how directory traversal works (utilising directories and inode tables) and its importance in file system **navigation**.
> - **Explain Advanced Directory Concepts**
>   - Justify each graph directory structures and the implications of **cycles** within directories.
>   - Explain how **reference counting** and **garbage collection protocols** are used in managing directory **lifecycles**.
>
> These learning objectives aim to provide a comprehensive understanding of how directories function as essential components of file systems, facilitating the organization, access, and management of files.

<span style="color:#f77729;"><b>Directories</b></span> are the file system cataloguing structure that contains metadata (references) to <span style="color:#f7007f;"><b>organize</b></span> files in a <span style="color:#f7007f;"><b>structured name space</b></span> (path). In computing, a namespace is a <span style="color:#f77729;"><b>set of symbols</b></span> that are used to organize objects of various kinds, so that these objects may be referred to by <span style="color:#f77729;"><b>name</b></span>.
{:.info}

In UNIX-based system, a directory is implemented as a file of a special type; a <span style="color:#f77729;"><b>special file</b></span> whose <span style="color:#f77729;"><b>content</b></span> is a named collection of other file(s) *plus* its corresponding inode id. It is no different than a regular file, but its contents are controlled by the OS. The content of a directory consists of list of names assigned to <span style="color:#f77729;"><b>inodes</b></span> that can be <span style="color:#f77729;"><b>interpreted</b></span> by the OS.

You can think of a directory as some kind of dictionary; a _symbol_ table that <span style="color:#f77729;"><b>translates</b></span> file names or as lists of association structures, each of which contains one filename and one inode number.

- They form lists of names assigned to <span class="orange-bold">inodes</span>.
  - Remember, inodes have numbers associated for each file but not filenames. <span style="color:#f77729;"><b>File names to inode mapping is done by the directory.</b></span>
- Each of the entries in the directory contains a <span style="color:#f77729;"><b>mapping</b></span> between <span style="color:#f77729;"><b>filename</b></span> and its associated <span style="color:#f77729;"><b>inode</b></span> number, plus _just enough_ other information to translate from a filename to an inode to get to the actual content.

Most of the metadata about the file is stored within the inode associated with the file, not the directory entry.
{:.note}
## Directory `x` Permission

Unlike regular files, the `x` bit on a directory permission is very different compared to regular files. It is often referred to as the _search bit_ and it stands for permission to **enter** the directory (e.g: via `cd`) and to actually access any of its files.
{:.info}

`Execute` or `x` is needed on a directory to **access** the inode information of the files within. Users need this to search a directory to read the inodes of the files within. For this reason the execute permission on a directory is often called search permission instead.

If we know the inode of the file beforehand within a directory which we don't have the `x` permission, can we still acess it? Well, from this little experiment, it doesn't seem so because `find` still returns the path of the target file having that inode and `cat` still needs the path of that file to be resolved:

<img src="{{ site.baseurl }}//assets/images/week6-2_directories/2023-06-02-21-47-40.png"  class="center_seventy no-invert"/>

If you know a way to `open` a file directly given the inode number in Linux, let us know!
{:.info}

This is **different** from `r` permission on directories: which is simply to **view** the directory's contents (e.g: via `ls`) and `w` permission which allows you to modify its content (add or remove files within it). Here's what `ls` output will look like on a directory without `x` permission. As you can observe, you can still obtain the file names and that's about it, no additional access to the files containing the usual metadata (date created, size, inode, etc).

<img src="{{ site.baseurl }}//assets/images/week6-2_directories/2023-06-02-21-44-52.png"  class="center_seventy no-invert"/>

## Searching for a File

Since a directory is also a file, it also has its own inode numbers, and its contents are other filenames and their inode numbers. One of the most famous directory is the inode 2 which is / (`root`) directory.

**Fun fact:** The inode with number 1 is typically reserved for a special purpose and is used by the filesystem for its internal purposes. In many Unix-like systems, inode 1 is used for the "bad blocks inode." This inode is used to keep track of bad blocks on the disk - blocks that have been determined to be unreliable for data storage. The filesystem can maintain a list of these blocks in this inode so that they are not used in the future, helping to prevent data loss or corruption.
{: .new}


The file system driver (part of Kernel) must <span style="color:#f77729;"><b>use</b></span> the directory when looking for a particular filename and then convert the filename to the correct corresponding inode number. It will start from the `root` node and <span style="color:#f77729;"><b>recursively</b></span> search for the file name.

# Directory Structure

A directory can also contain the name of other directories. We know this as a <span style="color:#f77729;"><b>subdirectory</b></span>.
{:.note}
Typically, both the directory structure and the files themselves reside on disk (and cached in memory). The figure below taken from the SGG book shows a common file system organization:

<img src="{{ site.baseurl }}/assets/images/week6/6.png"  class="center_fifty"/>

Each <span style="color:#f77729;"><b>volume/partition</b></span> that contains a file system must also contain information about the files in the system. This information is kept as <span style="color:#f77729;"><b>entries</b></span> in the device directory (a.k.a volume table of contents). The device directory (more commonly known simply as the directory) records some <span style="color:#f77729;"><b>important</b></span> information such as name, location, size, and type—for all files _in that volume_.

## Purpose

The <span style="color:#f77729;"><b>purpose</b></span> of having a directory structure is to <span style="color:#f77729;"><b>improve</b></span> user experience. 
{:.info }


1. <span style="color:#f77729;"><b>Efficiency</b></span>: locating a file or group of files quickly
2. <span style="color:#f77729;"><b>File naming</b></span> in a user-friendly manner
   - Users can pick the same name for their (different) files without conflicts
   - Same name for files of _different_ types (different extensions)
   - <span style="color:#f7007f;"><b>Same file can have different names</b></span> (multiple logical purposes; reference of same file/folder from different points in the name space)
3. <span style="color:#f77729;"><b>Organization</b></span> – logical grouping of files by various properties for ease of usage:
   - Group files with the same user, project, purpose, type, etc.

> Think about it as **IKEA** having a *directory* (named spaces, e.g: living room, kitchen things, and named items) to **improve** user experience: you know where to find the things that you wanted to browser or buy and you can remember the items by name more easily than remembering its serial number. 

## Folder

Note that notion of a directory is very _similar_ to the definition of a folder.
{:.note}

However folder is a <span style="color:#f77729;"><b>GUI</b></span> concept, associated with the common folder icons to represent collection of files. If you are referring to a _container of documents_, then the term folder makes more sense, related to the GUI. The term directory refers more <span style="color:#f77729;"><b>broadly</b></span> to the way <span style="color:#f77729;"><b>a structured list of document</b></span> is stored in the computer system.

## Directory Traversal

File manager and GUI shell like Finder for macOS and File Explorer for Windows <span style="color:#f77729;"><b>traverse</b></span> the directories (depending on our clicks, which is the same as typing cd <path> on the terminal) and read its contents.

In fact, they are all simply opening directories and rendering its entries. Our files are not stored in the GUI (_if that makes sense_). The GUI is just a <span style="color:#f77729;"><b>logical</b></span> representation of the contents of our secondary storage. These programs render different <span style="color:#f77729;"><b>icons</b></span> for different filenames (inclusive of its extension), and subdirectories of the current working directory, so that the content of the directories are more user friendly. 

Our disk does _not_ actually have places called `Desktop` or `Downloads`. These are just _logical representation_ of where our files are stored. The directory <span style="color:#f77729;"><b>maps</b></span> the file names (the way we are addressing our files) to its corresponding <span style="color:#f77729;"><b>inode</b></span>, and then from there (inode table), the system finally finds the <span style="color:#f77729;"><b>physical address</b></span> where the file is actually stored in the secondary storage.
{:.note}

Without the file system and the directory, the bytes that reside in our secondary storage are just a <span class="orange-bold">huge chunk</span> of bytes with <span class="orange-bold">no</span> discernible boundary separating the content of each file to another.
{:.error}

## Directory operations

All of the following file operations involves <span style="color:#f77729;"><b>modifying</b></span> and/or accessing the directory:

- <span style="color:#f77729;"><b>Create</b></span> a file or folder (subdirectory) n Delete a file or folder
- <span style="color:#f77729;"><b>List</b></span> a directory
- <span style="color:#f77729;"><b>Rename</b></span> a file
- <span style="color:#f77729;"><b>Search</b></span> for a file
- <span style="color:#f77729;"><b>Traverse</b></span> the file system with `cd` or via the GUI shell


## Single-level Directory

A single-level directory is best illustrated as follows:

<img src="{{ site.baseurl }}/assets/images/week6/7.png"  class="center_seventy"/>

In the above example, there exist a directory with five entries in it (inside the rectangular border), each pointing to a separate file (the circles).
- This means that all files are contained in the <span style="color:#f77729;"><b>same</b></span> directory, which is easy to support and understand. 
- The square nodes are the file names (inside the directory), and the circle nodes are files.

This is the simplest method to manage all of our files in the system is by having just one _giant_ list of all the files on the disk. The entire system will contain only <span style="color:#f77729;"><b>one</b></span> directory which is supposed to mention all the files present in the file system. The directory contains one entry per each file present on the file system.

However, since all files are in the same directory (same **name space**), when the number of files increases or when the system has more than one user, they must have unique <span style="color:#f77729;"><b>names</b></span>. This makes the system hardly usable.

Note that the file names (`readme.md`, `ls`, `myprog.c`, etc) that we see here illustrates how a directory entry points to an inode (not drawn) which finally leads us to the file itself.
{:.note}

If we were to use a single-level directory in our system and the GUI file-manager shell, we will see simply all directory entries (or as we will normally say, ‘all filenames’) in our entire mounted secondary storage.

## Two-level Directory

<img src="{{ site.baseurl }}/assets/images/week6/8.png"  class="center_seventy "/>

A two-level directory allows for a separate directory for <span style="color:#f77729;"><b>each user</b></span>. Notions of subdirectory and paths become clearer, e.g: `/User1/readme.md`, or `/Guest/readme.md`

Each user can have a <span style="color:#f77729;"><b>separate name space</b></span>, but still limited in logical grouping. To delete a file, the operating system confines its search to the _local_ user file directory thus, it cannot accidentally delete another user’s file that has the same name.

## Tree-Structured Directory

<img src="{{ site.baseurl }}/assets/images/week6/9.png"  class="center_full "/>

In a tree structure directory, there’s only <span class="orange-bold">one path</span> to reach each file, as shown above. This is <span class="orange-bold">not the same</span> as a [Graph-structured Directory](#graph-directory-structure). 

Multiple levels of hierarchy like this allow more <span style="color:#f77729;"><b>elaborate</b></span> organization of files, however:

- Full paths can become really <span style="color:#f7007f;"><b>long</b></span>
- Notions of <span style="color:#f7007f;"><b>current working directory</b></span> can shorten the path, e.g you can `cd` to a particular location and use files from your current directory without giving the full path.

Each <span style="color:#f77729;"><b>process</b></span> has its own current working directory, typically from the process who created it.

In this kind of structure, path names can be of two types: <span style="color:#f77729;"><b>absolute</b></span> and <span style="color:#f77729;"><b>relative</b></span>.

1. An <span style="color:#f7007f;"><b>absolute</b></span> path name begins at the root and follows a path down to the specified file, giving the directory names on the path.
2. A <span style="color:#f7007f;"><b>relative</b></span> path name defines a path from the current working directory.

# File Links

We can have different names for the same file by adding more than one entries pointing to the same inode in the directory. These directory entries are formally called <span style="color:#f7007f;"><b>links</b></span>.
{:.info}

## Recap about Inode

In UNIX-like file systems:

- An inode is a data structure containing <span style="color:#f77729;"><b>attributes</b></span> about all files in the system.
- These are the <span style="color:#f77729;"><b>references</b></span> to the actual file content: anonymous chunks of data, each given an inode number.
- A file in its entirety is _not_ an inode, a file has an associated inode with it. Inode contains only the file attributes (metadata).

It is pretty difficult for users to find their files based on inode numbers. Therefore, we have <span style="color:#f77729;"><b>directories</b></span> which allows us to traverse the file system using named paths.

## Recap about Directories

A directory is:

- A <span style="color:#f77729;"><b>special</b></span> file whose content is some kind of <span style="color:#f77729;"><b>mapping</b></span> of names to files (more specifically inodes).
- A directory can have another directory as its entry (so we can search for files via recursion).
- We can have <span style="color:#f77729;"><b>different</b></span> names for the same file; in this case, they have no difference (in content) at all, like a person with different nicknames.

These “extra names” mapped to the same inode entry are called <span class="orange-bold">hard links</span>.
{:.note}

## Hard Links

A hard link is a feature in many file systems that allows multiple filenames to refer to the same physical location on disk. In other words, a hard link is essentially another name for an existing file. 
{:.info}

Let's begin with an example. A containing `helloworld` characters is created, with inode `54346159`. We name it `input`. At this point, a directory entry named `input` pointing to this file containing `helloworld` is created within the directory named `Links`. We call this entry the original filename.

<img src="{{ site.baseurl }}/assets/images/week6/14.png"  class="center_full no-invert"/>

All files in a directory-based file system must have at least one hard link (mapping) giving what we call the original name for each file.
{:.note}

### Reference Count

Each hard link <span style="color:#f77729;"><b>increases</b></span> the <span class="orange-bold">reference count</span> to the inode table entry it is pointing to by 1. Notice how after `hard_link` is created then the reference to inode `54346159` is 2 in the screenshot above.

### Subdirectories

In some operating systems, such as Linux, when you create new subdirectories, you’re creating a new entry to the existing directory.

<img src="{{ site.baseurl }}/assets/images/week6/15.png"  class="center_full no-invert"/>

The reference count for `subdir` is 2 by default, shown in the screenshot above. UNIX-like file system will create two entries in every directory:

- `.` pointing to the directory itself (thus increasing the reference count of itself by 1 -- resulting in 2 that we see in the screenshot above)
- `..` pointing to its <span style="color:#f77729;"><b>parent</b></span> (thus increasing the reference count of its parent: `Links`)

Note that the above are _special links_, created by the OS for easy of traversal. For the sake of simplicity in this course, we _ignore_ the presence of `.` and `..`.

This provided an <span style="color:#f77729;"><b>easy</b></span> way to traverse the filesystem, both for applications and for the OS itself.

### Hard Links to Link Directories

In UNIX-like systems, you <span style="color:#f7007f;"><b>cannot</b></span> create more hard links to link directories, only files. <span style="color:#f7007f;"><b>This is to prevent cycles using hard links</b></span>. By effectively prohibiting multiple references to directories, UNIX-like OS like Linux maintains an <span class="orange-bold">acyclic</span>-graph structure (more about this structure later). However this is not always true for all OS.

<img src="{{ site.baseurl }}/assets/images/week6/16.png"  class="center_full no-invert"/>

Note that `..` and `.` can be regarded as _special_ hard links, so although they technically points to directories (itself and the parent) we don't treat it the same as other regular user-created hard links. Again, for the same of simplicity in this course we <span style="color:#f77729;"><b>ignore</b></span> the presence of these special links. All sections below assume that `.` and `..` are not implemented.
{:.note}

### Path Names

Links have path names, e.g: `/Users/natalie_agus/Desktop/Links/input_hardlink`.

<img src="{{ site.baseurl }}/assets/images/week6/17.png"  class="center_full no-invert"/>

## Symbolic Links

A symbolic link is simply a file whose content is a text string (i.e: reference to another file or directory) that is <span style="color:#f77729;"><b>automatically interpreted</b></span> and followed by the operating system as a path to another file or directory.
{:.info}

Symbolic links are also known as "shortcuts" or "soft links".

<img src="{{ site.baseurl }}/assets/images/week6/18.png"  class="center_full no-invert"/>

### Symbolic Links to Link Directories

You <span style="color:#f77729;"><b>can</b></span> link directories with symbolic links, but <span class="orange-bold">not</span> with hard links. 
{:.warning}

Unlike hard links, a symbolic link is <span style="color:#f7007f;"><b>NOT</b></span> just a directory entry. <span style="color:#f7007f;"><b>It is a file</b></span>. Of course as a consequence, this (symbolic link) file has a name and this name is a directory entry somewhere. As shown in the screenshot above, it has its <span style="color:#f77729;"><b>own</b></span> inode number: `54354162`, indicating it is a <span style="color:#f77729;"><b>new file</b></span>.

### File Permission

The file permission value of a symlink differs between operating systems. For instance, `chmod` command in Linux states that:

> *`chmod` never changes the permissions of symbolic links; the `chmod` system call cannot change their permissions. This is **not** a problem since the permissions of symbolic links are **never** used. However, for each symbolic link listed on the command line, `chmod` changes the permissions of the **pointed-to** file. In contrast, `chmod` **ignores** symbolic links encountered during recursive directory traversals.*

However on macOS, `chmod` does things differently. See the example below:

<img src="{{ site.baseurl }}//assets/images/week6-2_directories/2023-06-03-12-36-20.png"  class="center_seventy no-invert"/>

If the file is a symbolic link, `chmod` with a `-h` option changes the mode of the link itself rather than the file that the link points to. However most utilities like `echo` and `cat` follows the symlink and ignored the permission. `ls` however is affected. If a symlink is set to have `000` permission, `ls` will not print out where it's pointing to.

On the contrary, here's what happens on Ubuntu:

<img src="{{ site.baseurl }}//assets/images/week6-2_directories/2023-06-02-20-16-47.png"  class="center_seventy no-invert"/>

Note that these are just for illustration purposes only, there's **no need to memorise**.
{:info}

### Link Properties

#### Broken Symlink

If you delete the <span style="color:#f77729;"><b>reference</b></span> hard link (file name) where the symbolic link is pointing to, the symbolic link will be <span style="color:#f7007f;"><b>broken</b></span>.

<img src="{{ site.baseurl }}/assets/images/week6/19.png"  class="center_full no-invert"/>

If we recreate `input` (file with the same name), then `input_softlink` will work again.

<img src="{{ site.baseurl }}/assets/images/week6/20.png"  class="center_full no-invert"/>

Therefore from this we can know that `input_softlink` is a FILE with id `54354162`, which <span class="orange-bold">content</span> is path to input, e.g: `~/Desktop/Links/input`. It is automatically interpreted by the OS such that when you open `input_softlink` it will resolve` ~/Desktop/Links/input` and open the referred file.
{:.note}

#### Reference Count

In the example above, although `input` was the original filename, since there’s one more hardlink (`input_hardlink`) pointing to the same inode `54346159`, then this file containing `helloworld` is not yet deleted. We can still access the file via this other hard link pointing to the same file.

Likewise, if you delete all hard links to the file, its entry in the inode table will be removed (file completely deleted, space is freed).
{:.warning}

## Why do we need links?

### Hard links use case
Imagine having photos in your system, thousands of them. You want to <span style="color:#f77729;"><b>categorize</b></span> them for easier access (create different albums):

- You can categorize them as: photos of red things, photos of blue things, photos of green things.
- You can also categorize them as: photos of vehicles, photos of trees, photos of flowers

Suppose you want to categorize them both ways. What you can do:
<span style="color:#f77729;"><b>Method 1</b></span>: You could create a <span style="color:#f77729;"><b>copy</b></span> of the photos and categorize it multiple times in the way you want it.

- This means you have two copies of the same file taking up two times the space.
- It is not ideal to do this, especially with large amounts of data.

<span style="color:#f77729;"><b>Method 2</b></span>: Create hard links.

- A hard link takes up almost no space at all.
- You could, therefore, store the same photo in various different categories (i.e. by color, by type, etc).

### Symbolic links use  case
A symbolic link is much like a desktop <span style="color:#f77729;"><b>shortcut</b></span> within Windows. The symbolic link merely points to the location of a file. We have all been using it.


## Links Summary

The figure below summarizes the concept of links:
<img src="{{ site.baseurl }}/assets/images/week6/21.png"  class="center_seventy"/>

# Graph Directory Structure

## Acyclic Graph Directory

In acyclic graph directories, we can have the <span style="color:#f77729;"><b>same</b></span> file, but with <span style="color:#f77729;"><b>multiple</b></span> names (through symbolic/hard links).
{:.info}


In the graphical representation of a directory, we draw edges to represent any links, be it symbolic or hard link. This diagram is just an example. Don’t try to make much sense about the file names, they're just arbitrary examples.

<img src="{{ site.baseurl }}/assets/images/week6/22.png"  class="center_full"/>

Note that Acyclic Graph Directory structure is <span class="orange-bold">not the same</span> Tree Directory Structure. In <span style="color:#f77729;"><b>Tree</b></span> directory structure, we can’t have more than 1 path to reach the same file entry denoted in pink circles.
{:.warning}

From the example above, notice that `/Users/Guest/readme.md` is a _hard_ link to` /Users/Guest/readme.txt` and `/bin/readme` is a _symbolic_ link to the <span style="color:#f77729;"><b>same</b></span> file pointed by `/Users/Guest/readme.txt`.

- If we do `rm /Users/Guest/readme.md` AND `rm /Users/Guest/readme.txt`, the underlying file is <span style="color:#f77729;"><b>removed</b></span>. The inode for this file is removed, and we can say that our file is _erased_. The symbolic link becomes a _dangling_ pointer, references to a non-existent file.
- If we do `rm /bin/readme`, the underlying file is <span style="color:#f7007f;"><b>not</b></span> removed, because there’s still _another_ hard-link reference to the file.
- The inode table entry (for the file) is completely removed only when its reference count is zero.

An inode entry <span class="orange-bold">cannot</span> be deleted as long as the reference count to it is more than 0, meaning that there’s 1 or more directory entry that points to the inode entry.
{:.warning}

## General Graph Directory

The primary advantage of an acyclic graph is the relative simplicity of the algorithms to <span style="color:#f77729;"><b>traverse</b></span> the graph and to determine when there are no more references to a file. We want to avoid traversing shared sections of an acyclic graph twice, mainly for performance reasons. The figure below illustrates a general graph directory, where **loops** (self referencing) are allowed for usage convenience. 

<img src="{{ site.baseurl }}/assets/images/week6/23.png"  class="center_full "/>

**Caveat**: if cycles are allowed to exist in the directory (red arrow above -- can be either soft/symbolic link or hard link), we need to _avoid_ searching any component twice or more for reasons of <span class="orange-bold">correctness</span> as well as <span class="orange-bold">performance</span>.

A poorly designed algorithm might result in an infinite loop continually searching through the cycle and never terminating. One solution is to limit the number of directories traversed when searching.
{:.note}

### Self referencing

In a cyclic directory, using <span style="color:#f77729;"><b>reference count</b></span> to determine whether we need to delete a node might not work due to self referencing. You’d need other <span class="orange-bold">garbage-collection</span> protocols.
{:.warning}

For example, suppose we `delete` the reference edge as shown below.

This `delete` action will <span style="color:#f77729;"><b>reduce</b></span> the reference count of all the children of that edge (whatever node -- file -- that is pointed to by the edge). However due to <span style="color:#f77729;"><b>self referencing nodes</b></span>, the directory containing {`User1`, `Guest`} has their reference count set to `2` (for simplicity, we ignore reference count due to “.” and “..”).

Note that symbolic links <span style="color:#f77729;"><b>do not</b></span> increase reference count
{:.note}

As a result, the `delete` action <span style="color:#f7007f;"><b>does not reduce </b></span>the reference count of this directory to _zero_. We are left with three inaccessible directories as shown in the image below.

<img src="{{ site.baseurl }}/assets/images/week6/24.png"  class="center_full"/>

# Summary

Directories, also known as folders, play a crucial role in organizing and managing files within an operating system. 

This chapter delves into the **concept** of directories, explaining their **purpose**, structure, and **functionality**. It explores how directories are used to hierarchically organize files, providing a systematic way to navigate and access data stored on storage devices. We also discuss **various** directory operations, such as creating, renaming, moving, and deleting directories, elucidating their impact on file organization and system management. Additionally, we examine directory **structures**, including tree-based and graph-based representations, and elucidates their advantages and limitations. 


Key learning points include:
- **Directory Structure and Permissions**: Covers the significance of directory permissions (read, write, execute) and the structure of directories including single-level, two-level, and tree-structured directories.
- **File and Directory Operations**: Explores operations such as creating, searching, and deleting files and directories, and explains how directories serve as a namespace to manage file organization.
- **Links and Inodes**: Discusses the concepts of hard and symbolic links, their use cases, and how inodes play a critical role in file system structure.


Understanding directories is essential for effectively managing file systems and ensuring efficient data organization within an operating system.

