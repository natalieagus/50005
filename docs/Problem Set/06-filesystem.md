
---
layout: default
permalink: /ps/6-filesystem
title: Filesystem and Directories
parent: Problem Set 
nav_order:  6
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

# Filesystem and Directories
{: .no_toc}




## The Missing Inode

### Background

In UNIX-like systems, files are represented by inodes, which store metadata and pointers to actual data blocks on disk. Hard links increase an inode’s reference count, allowing multiple directory entries to point to the same underlying file.

### Scenario

A user creates a directory `/data/project` and fills it with several large files. To back up the most important one, they create a hard link at `/backup/file1` pointing to `/data/project/file1`. Later, a cleanup script deletes the entire `/data/project` directory. Although `/backup/file1` still works, the disk usage reported for `/data` no longer includes `file1`, and the user worries about data loss.

### Answer the following questions

* What information does an inode store about a file?
* Why does `file1` still exist even after `/data/project` is deleted?
* Why does the disk usage of `/data` no longer reflect the size of `file1`?
* What would have happened if a **symbolic link** had been used instead of a hard link?
* What are the risks of relying solely on hard links for backup?

{: .highlight}
> **Hints**:
>
> * Inodes exist independently of directory paths.
> * Deleting a directory entry decreases the inode’s link count.
> * Disk usage tools often traverse only accessible paths.
> * A symbolic link does not preserve file data if the target is deleted.

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
An inode stores metadata about a file, including ownership, permissions, timestamps, size, and the disk block pointers for the actual file data. It does not store the filename or the directory path, which are maintained separately in directory entries.
</p>
<p>
When a file is hard-linked, multiple directory entries point to the same inode. Deleting one path merely decrements the link count on the inode. As long as at least one directory entry (like `/backup/file1`) still references the inode, the file remains accessible and its data persists on disk.
</p>
<p>
Disk usage tools like <code>du</code> typically perform recursive traversal from a given directory. Since `/data/project` was deleted, <code>du /data</code> no longer encounters `file1` during its walk, even though the data blocks still exist. The file is no longer logically reachable from `/data`, even if it physically consumes space.
</p>
<p>
If the user had created a symbolic link instead, it would have stored the pathname `/data/project/file1` as its target. Once the original file was deleted, the symlink would become broken, resulting in a “No such file or directory” error when accessed.
</p>
<p>
Hard links can lead to confusion when users assume files in different locations are independent, even though they modify the same underlying data. Relying solely on hard links for backup is risky because it doesn't create a true copy — it simply adds another pointer. A mistaken edit or deletion could affect the data across all links.
</p>

</p></div><br>


Here is **Handout #2: *The Silent Cycle***, in full 50.005 OS handout format:

---

## The Silent Cycle

### Background

Symbolic links in UNIX-like systems are flexible pointers to file or directory paths, but unlike hard links, they can point across filesystems and create arbitrary chains. Improper use can lead to infinite traversal cycles.

### Scenario

A developer is organizing a complex codebase split into multiple directories. To simplify navigation, they create a symbolic link `lib/alias → ../common/lib`, and within `common`, they add another symbolic link `lib_back → ../../lib`. Running `ls -R` inside `lib` causes the terminal to freeze. After forcibly killing the process, they realize the command had entered an infinite loop.

### Answer the following questions

* How do symbolic links behave during directory traversal?
* Why did `ls -R` enter an infinite loop in this case?
* Why can’t the filesystem detect and stop such cycles automatically?
* How could this situation be prevented or mitigated?
* Would using hard links instead have caused the same problem? Why or why not?

{: .highlight}

> **Hints**:
>
> * Symbolic links are path-based and can point to any location.
> * Tools like `ls -R` follow symlinks recursively unless explicitly configured.
> * The kernel does not track visited paths during user-space traversal.
> * Hard links cannot create cycles between directories.

---

### ✅ HTML Answer Key

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
Symbolic links act as redirectors to other filesystem paths. When a program accesses a symlink, the system resolves it to its target path, potentially traversing into other directories. If the link points to another directory, tools that perform recursive traversal, like <code>ls -R</code>, may continue following it as part of the hierarchy.
</p>
<p>
In this case, the symbolic link <code>lib/alias → ../common/lib</code> and <code>common/lib_back → ../../lib</code> form a cycle. When <code>ls -R</code> enters <code>lib/alias</code>, it gets <span class="orange-bold">redirected</span> back to <code>lib</code> through the symlink in <code>common</code>. This causes it to recursively loop through the same directories indefinitely, consuming system resources until killed.
</p>
<p>
*The filesystem doesn't track symbolic link resolution history during traversal*. Since each symbolic link is just a **path** string, the kernel does not retain a visited set or loop detection mechanism. Infinite loops from symlinks are therefore a user-space responsibility to avoid or detect.
</p>
<p>
To prevent this, developers can avoid creating mutual symlinks between directories. Tools like <code>find</code> and <code>ls</code> offer options such as <code>-P</code> to prevent symlink following, or <code>-L</code> with depth limits. Scripts that traverse directories should maintain a set of visited real paths to detect cycles.
</p>
<p>
Hard links cannot be used to create directory loops in most modern UNIX systems. This is because hard links to directories are either forbidden or heavily restricted to prevent cycle formation in the directory tree. Even when allowed, hard links refer to inodes directly and do not cause redirection across paths.
</p>

</p></div><br>
