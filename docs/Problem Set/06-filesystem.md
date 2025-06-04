
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

**Answer the following questions:**
* What information does an inode store about a file?
* Why does `file1` still exist even after `/data/project` is deleted?
* Why does the disk usage of `/data` no longer reflect the size of `file1`?
* What would have happened if a **symbolic link** had been used instead of a hard link?
* What are the risks of relying solely on hard links for backup?

{: .highlight}
> **Hints**:
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



## The Silent Cycle

### Background

Symbolic links in UNIX-like systems are flexible pointers to file or directory paths, but unlike hard links, they can point across filesystems and create arbitrary chains. Improper use can lead to infinite traversal cycles.

### Scenario

A developer is organizing a complex codebase split into multiple directories. To simplify navigation, they create a symbolic link `lib/alias → ../common/lib`, and within `common`, they add another symbolic link `lib_back → ../../lib`. Running `ls -R` inside `lib` causes the terminal to freeze. After forcibly killing the process, they realize the command had entered an infinite loop.

**Answer the following questions**:
* How do symbolic links behave during directory traversal?
* Why did `ls -R` enter an infinite loop in this case?
* Why can’t the filesystem detect and stop such cycles automatically?
* How could this situation be prevented or mitigated?
* Would using hard links instead have caused the same problem? Why or why not?

{: .highlight}
> **Hints**:
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




## The Forgotten Mount



### Background

In UNIX and Linux systems, a **volume** refers to a logical storage unit, often represented by a disk partition, device, or remote filesystem. The **mount** operation binds such a volume to a specific directory path—called a **mount point**—so its contents become accessible as part of the global filesystem tree.

When a volume is mounted on a directory (e.g. `mount /dev/sdb1 /data`), the original contents of `/data` are hidden and replaced by the new volume’s namespace. This **mount namespace** remaps the visible contents of `/data` to those of the mounted volume, without deleting the underlying files. These hidden files remain on disk, consuming space, and can reappear after unmounting the volume.

Understanding this remapping behavior is critical when diagnosing **discrepancies** in disk usage, especially in systems that dynamically mount and unmount storage volumes.


### Scenario

An engineer configures a persistent storage volume and mounts it at `/data` to store logs. Later, they notice that disk usage in `/data` is very low despite writing large files. Upon unmounting the volume to debug the issue, they find an old set of forgotten `.log` files taking up several gigabytes that were previously invisible.

**Answer the following questions:**
* What happens to the original directory contents when a new filesystem is mounted over it?
* Why do tools like `du` report misleadingly low usage in `/data`?
* After unmounting, why did the old `.log` files reappear?
* How can this situation lead to wasted disk space or confusion?
* What are good practices when choosing or preparing a mount point?

{: .highlight}
> **Hints**:
> * Mounting does not delete original data.
> * Tools scan the mounted view, not what's hidden beneath.
> * Check for leftover data before and after mounting.
> * Reserved mount points should be empty or cleared.

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
When a filesystem is mounted onto a directory like <code>/data</code>, the mount point acts as an entry into the new filesystem, and the original contents of <code>/data</code> are hidden—not removed. They continue to exist on the underlying disk but are no longer visible in the current namespace until the mount is unmounted.
</p>
<p>
Disk usage tools such as <code>du</code> operate by walking through the visible directory structure. If a mount overlays a directory, these tools only see the contents of the mounted filesystem, not the underlying files. This can lead to an illusion that <code>/data</code> has very little usage when, in reality, hidden files beneath the mount point still occupy disk space.
</p>
<p>
Once the mount is removed with <code>umount /data</code>, the kernel stops redirecting access to the mounted filesystem and re-exposes the original directory contents. In this case, the engineer discovers lingering <code>.log</code> files from before the mount operation, which had continued to consume space without being visible.
</p>
<p>
This can cause disk usage to silently increase, especially if large files were left behind under mount points. These files won't be seen or managed during normal usage, yet they remain on the root filesystem, potentially leading to disk full errors or confusion during debugging.
</p>
<p>
To avoid this, it's good practice to ensure that any directory used as a mount point is either <strong>empty</strong> or <em>explicitly cleared</em> before mounting over it. Scripts can use <code>ls -A /mountpoint</code> before mounting to detect leftover files. Additionally, use fixed-purpose directories (e.g., <code>/mnt/volume1</code>, <code>/var/lib/data</code>) rather than general-purpose paths to avoid accidental overlay of critical data.
</p>



</p></div><br>



## The Incomplete Flush

### Background

In UNIX-like operating systems, file writes are buffered in memory to improve performance. These buffers are managed by the kernel and are only written to disk either periodically or when explicitly flushed using system calls like `fsync()`. Crashes or power loss before flushing can cause inconsistencies where files appear to exist but contain incomplete or missing data.

### Scenario

A monitoring service writes logs to `/var/log/monitor.log` continuously. After a sudden system crash, the administrator reboots the machine and checks the log file. The file exists and has the expected size, but its contents are mostly zeroed out or garbage characters. This occurs despite the application calling `write()` regularly.

**Answer the following questions**:
* What is the difference between writing to a file and flushing it?
* Why might the file size appear correct despite its contents being invalid?
* How do journaling filesystems help in this situation?
* Could using `close()` instead of `fsync()` have helped?
* What practices can developers adopt to reduce this risk?

{: .highlight}
> **Hints**:
> * `write()` only affects kernel buffers.
> * Inodes and data blocks can be updated independently.
> * File content loss may occur even if the file "exists."
> * Journaling often protects metadata, not content.
> * Only `fsync()` guarantees persistence.

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p><strong>What is the difference between writing to a file and flushing it?</strong><br>
The <code>write()</code> system call places data into the kernel’s page cache (a memory buffer), but this does not immediately write data to disk. To ensure persistence, the application must call <code>fsync()</code> or <code>fdatasync()</code> to flush the file’s data and metadata to the storage device. Without flushing, data can be lost if the system crashes before the kernel gets a chance to write it out.
</p>

<p><strong>Why might the file size appear correct despite its contents being invalid?</strong><br>
The metadata updates, such as file size and timestamps, may have been flushed to disk before the crash, while the actual file contents remained in memory buffers. As a result, the inode reflects the intended size, but the data blocks on disk are uninitialized or stale, leading to mismatch between structure and content.
</p>

<p><strong>How do journaling filesystems help in this situation?</strong><br>
Journaling filesystems like ext4 or XFS can reduce corruption risks by recording intended metadata changes in a log before applying them. However, most only guarantee metadata consistency unless explicitly configured to journal data too. Without full data journaling or proper flushing by the application, file contents can still be lost.
</p>

<p><strong>Could using <code>close()</code> instead of <code>fsync()</code> have helped?</strong><br>
Not necessarily. While <code>close()</code> may trigger an implicit flush in some cases, it does not guarantee that data reaches the physical disk before the call returns. Only <code>fsync()</code> provides a strict durability guarantee. Applications that need crash-resilient writes must flush data explicitly.
</p>

<p><strong>What practices can developers adopt to reduce this risk?</strong><br>
Developers should flush critical data explicitly using <code>fsync()</code> after key writes, especially for logs, databases, or checkpoint files. Using temporary files followed by <code>rename()</code> ensures atomic updates. For embedded systems, disabling write buffering or using <code>O_SYNC</code> may also help, though at a performance cost.
</p>

</p></div><br>
