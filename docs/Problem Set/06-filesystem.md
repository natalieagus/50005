
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
<p>
The <code>write()</code> system call places data into the kernel’s page cache (a memory buffer), but this does not immediately persist the data to disk. To ensure durability, the application must call <code>fsync()</code> or <code>fdatasync()</code>, which flushes the buffered data and metadata to the physical storage device.
</p>

<p>
The file size appears correct because the inode metadata—such as size and timestamps—may have already been flushed to disk before the crash. However, the actual file data might still have been sitting in volatile memory buffers and never committed to disk, resulting in empty or corrupted content.
</p>

<p>
Journaling filesystems like ext4 and XFS help protect against corruption by recording metadata changes in a journal before applying them. However, unless data journaling is explicitly enabled, only the metadata is protected. The actual file content can still be lost if not flushed before a crash.
</p>

<p>
Calling <code>close()</code> does not guarantee that data is flushed to disk. While it may trigger a flush indirectly, only <code>fsync()</code> provides a strict guarantee that the data has reached persistent storage. Relying on <code>close()</code> alone is insufficient for applications that require durability.
</p>

<p>
To reduce risk, applications should explicitly call <code>fsync()</code> after writing critical data. For atomic updates, a safer pattern is to write to a temporary file and use <code>rename()</code> after flushing. In cases where performance is less critical, files can be opened with <code>O_SYNC</code> to force synchronous writes at the cost of speed.
</p>

</p></div><br>



## The Phantom File

### Background

In UNIX-like systems, deleting a file via `unlink()` removes the **name** from its directory, but the file’s data <span class="orange-bold">remains on disk</span> until its inode’s reference count drops to zero. If a process still holds an open file descriptor, the file persists invisibly and continues to consume disk space.

{:.note-title}
> Recap
>
> In UNIX-like systems, each file is associated with an inode, which stores metadata and pointers to data blocks. The inode reference count tracks how many directory entries (i.e., filenames) or open file descriptors refer to that inode. When a file is deleted via `unlink()`, the reference count is decremented. The inode and its data blocks are only reclaimed when the count reaches zero—meaning no directory entry and no open file descriptor still points to it. This mechanism ensures safe deletion only when no process can access the file anymore.

### Scenario

A data processing service writes output to a temporary log file. An administrator, attempting to free up space, deletes the file mid-execution using `rm /tmp/output.log`. However, the file seems to keep growing, and disk usage remains high. After the process finishes, the space is finally reclaimed.

**Answer the following questions**:
* Why does deleting a file not immediately free disk space?
* How can a deleted file continue to grow?
* What happens to the file’s data while the process is still running?
* How can an administrator identify these "phantom" files?
* What is the proper way to handle temporary files for long-running processes?

{: .highlight}
> **Hints**:
> * File deletion removes a name, not the data.
> * Open file descriptors keep inodes alive.
> * Use tools like <code>lsof</code> to find unlinked but open files.
> * Space is reclaimed only after all file descriptors are closed.



<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
In UNIX, deleting a file using <code>rm</code> unlinks its name from the directory structure, but the file’s inode and data blocks remain allocated as long as any process holds an open file descriptor to it. The kernel keeps the file alive until all such references are closed.
</p>

<p>
If a process continues writing to an open file descriptor after the file is deleted, the data is still appended to the same inode. Although the file has no visible path in the filesystem, it remains fully functional and continues consuming disk space.
</p>

<p>
The data written after deletion is stored normally and remains accessible to the writing process. It becomes orphaned in the filesystem namespace but not in memory; the file persists in the background and behaves like any regular file until it is closed.
</p>

<p>
Administrators can identify such orphaned files using tools like <code>lsof | grep deleted</code>, which list open file descriptors referencing deleted inodes. These entries are typically shown with a file path marked "(deleted)" to indicate that they no longer exist in any directory.
</p>

<p>
A safer approach is to let the program manage its own temporary files, deleting them only after explicitly closing all file descriptors. Alternatively, using <code>O_TMPFILE</code> (on supported systems) or the standard <code>tmpfile()</code> interface ensures the file has no name from the start and is automatically cleaned up once closed.
</p>

</p></div><br>



## The Alias Trap

### Background

In UNIX-like systems, hard links allow multiple directory entries to point to the same inode. While useful for redundancy or organization, hard links can confuse users who mistakenly assume they’re working with independent copies of a file when they are in fact modifying shared data.

### Scenario

A student copies an important configuration file using `ln original.conf backup.conf`, intending to experiment safely on the backup. After changing a few values in `backup.conf`, they realize that `original.conf` has changed too. Panicking, they realize their only "backup" has overwritten the original.

**Answer the following questions**:
* What exactly does a hard link do in UNIX filesystems?
* Why did changes to `backup.conf` also affect `original.conf`?
* How can users confirm whether two files are hard-linked?
* What are the risks of using hard links for backups?
* What is a safer alternative when a user intends to duplicate a file?

{: .highlight}
> **Hints**:
> * Hard links reference the same inode.
> * Modifications affect shared data blocks.
> * Use <code>ls -li</code> or <code>stat</code> to inspect inode numbers.
> * Hard links are not copies; they are aliases.
> * <code>cp</code> creates an actual copy with a new inode.



<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
A hard link creates an additional directory entry that points to the same inode as the original file. This means both filenames refer to the exact same data and metadata on disk, including content, permissions, and timestamps.
</p>

<p>
Changes made to <code>backup.conf</code> were reflected in <code>original.conf</code> because both filenames point to the same underlying file structure. Any edits modify the shared data blocks since there is no duplication of content—just an alias at the directory level.
</p>

<p>
Users can inspect whether two files are hard-linked by running <code>ls -li</code> or <code>stat</code> on both files. If the inode numbers are identical, the files are hard links to the same inode. The link count field in <code>stat</code> also reveals how many directory entries point to that inode.
</p>

<p>
Using hard links for backups is risky because users often assume each filename refers to a separate file. Any accidental change or deletion affects all hard links. There is no isolation between "copies," and recovery is not possible if changes are undesired.
</p>

<p>
The safer alternative is to use <code>cp</code> to create a true copy of the file. This generates a new inode with its own data blocks and metadata, so modifications to one file do not affect the other. For added safety, users may also consider versioning or backup tools with snapshot capabilities.
</p>


</p></div><br>




## The Shadow Directory

### Background

In UNIX-like systems, access to files is governed not only by file-level permissions, but also by the permissions on each directory in the path. A user may be denied access to a file even if the file itself is world-readable, simply because one of the parent directories restricts traversal (`x` permission).

### Scenario

A system contains a folder `/secure/data/reports.txt` which is readable by everyone. However, the administrator changes the permissions on `/secure` to `700` to restrict access. After this change, non-root users report that `/secure/data/reports.txt` seems to have “disappeared,” even though the file still exists and has not been modified.

**Answer the following questions**:
* Why can’t users access a readable file if a parent directory lacks execute permission?
* What is the effect of removing read or execute permissions from a directory?
* How is directory traversal different from reading a directory?
* How can a file be effectively “invisible” despite existing and having open permissions?
* What are safe ways to enforce directory-level restrictions without breaking valid access?

{: .highlight}
> **Hints**:
> * `x` permission controls traversal into directories.
> * `r` permission allows listing contents of a directory.
> * Each component of the file path must be accessible.
> * File permissions do not override parent directory restrictions.

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
Even if a file like <code>reports.txt</code> is world-readable, users must be able to traverse every parent directory to reach it. The <code>x</code> (execute) permission on a directory allows traversal—i.e., the ability to access files within it. If a user lacks execute permission on <code>/secure</code>, they cannot access anything inside it, regardless of file-level permissions.
</p>

<p>
Removing read permission (<code>r</code>) from a directory prevents users from listing its contents (e.g., with <code>ls</code>), while removing execute permission (<code>x</code>) blocks access to entries by name. Without <code>x</code>, even if a user knows the full filename, the system denies access when trying to open the file.
</p>

<p>
Directory traversal is distinct from directory reading. Traversal checks whether the process is allowed to follow a path through a directory (controlled by <code>x</code>), while reading involves listing all entries in the directory (controlled by <code>r</code>). A user may be able to access a file by name but not see it in listings, or vice versa.
</p>

<p>
A file can appear “invisible” when directory traversal is blocked. This can happen when an intermediate directory in the path removes execute permission. Although the file remains on disk with unchanged permissions, users who cannot traverse the path see errors like “Permission denied” or “No such file,” even if the file technically exists.
</p>

<p>
To enforce secure access, administrators should apply directory-level restrictions carefully. Rather than blocking entire directories, consider using group ownership and applying the right permission bits (e.g., <code>750</code> for group-readable access). This way, necessary users can still traverse the directory while outsiders are restricted appropriately.
</p>


</p></div><br>



## The Descriptor Leak

### Background

In UNIX-like operating systems, file descriptors are small integers used by a process to access open files, sockets, or other I/O resources. Each process has a per-process file descriptor table, but the system also enforces a global limit. Failure to properly close descriptors can exhaust these limits, causing system calls like `open()` or `socket()` to fail.

### Scenario

A long-running server application begins failing sporadically after several days, with errors like “Too many open files.” Debugging reveals that each time a client connects, the server opens a file to log the session—but never closes it. Eventually, the server becomes unable to accept new clients, despite having enough memory and CPU.

**Answer the following questions**:
* What are file descriptors, and how are they managed?
* What is the difference between per-process and system-wide descriptor limits?
* Why does forgetting to close descriptors lead to resource exhaustion?
* How can a developer detect or debug file descriptor leaks?
* What coding practices can help avoid such leaks in long-running applications?

{: .highlight}
> **Hints**:
> * Descriptors are reused only after <code>close()</code>.
> * Use <code>lsof</code> or check <code>/proc/\[pid]/fd</code>.
> * <code>ulimit -n</code> shows per-process soft limit.
> * File descriptors cover more than just regular files.
> * Consider <code>defer close()</code> or RAII-style patterns.

<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
File descriptors are indexes into a per-process table that point to kernel-managed file objects. When a process opens a file, the kernel assigns the lowest available descriptor number and increments reference counts on internal structures. These descriptors must be explicitly released using <code>close()</code> when no longer needed.
</p>

<p>
Each process has its own descriptor table, typically limited by a soft per-process limit (e.g., 1024 entries). The system as a whole also imposes a global limit on open file objects, shared across all processes. Reaching either limit can prevent new files, sockets, or pipes from being opened.
</p>

<p>
When an application fails to close descriptors, each new open call consumes a slot in the table. Over time, this leads to resource exhaustion, and operations like <code>open()</code> or <code>accept()</code> return errors such as “EMFILE” or “ENFILE.” The problem persists until the leaking process is killed or corrected.
</p>

<p>
Descriptor leaks can be detected using <code>lsof -p &lt;pid&gt;</code> or by inspecting <code>/proc/&lt;pid&gt;/fd/</code> to see all open handles. A continuously growing number of descriptors—especially pointing to the same file or socket type—is a clear sign of a leak. Some languages or runtimes provide leak detection utilities or hooks.
</p>

<p>
To avoid leaks, developers should always close descriptors in error paths and normal operation. In higher-level languages, using constructs like Python’s <code>with</code> block or Go’s <code>defer f.Close()</code> helps ensure proper cleanup. In C or systems code, disciplined use of <code>close()</code> and centralized resource management are key.
</p>



</p></div><br>



## The Dangling Shortcut

### Background

In UNIX-like systems, symbolic links (symlinks) are special files that store a textual reference to another path. Created with `ln -s <target> <linkname>`, symlinks redirect file operations to the target path. Access to the **target** is governed by its own permissions — not the symlink’s. Symlink permissions (e.g., `lrwxrwxrwx`) are generally ignored during access. Importantly, modifying a symlink’s **target path** requires replacing the symlink, which involves writing to the directory where the symlink resides.

### Scenario

A symbolic link has been created as follows:

```bash
ln -s /home/user-x/secret.txt /public/link.txt
```

Setup:
* The symlink `/public/link.txt` has permissions `lrwxrwxrwx` and is owned by `root`
* The directory `/public/` is owned by `root` and has permissions `drwxr-xr-x`
* The target file `/home/user-x/secret.txt` is owned by `user-x` and has permissions `rwx------`
* A different user, `user-y`, exists on the system

**Answer the following questions**:
* Can `user-y` change the name of the symlink? Why or why not?
* Can `user-y` change the target (i.e., content) of the symlink? Why or why not?
* Can `user-y` read the contents of `/home/user-x/secret.txt` by accessing `/public/link.txt`? Why or why not?

{: .highlight}
> **Hints**:
> * Changing file names requires write access to the containing directory.
> * Symlinks are <span class="orange-bold">not</span> editable like normal files.
> * A symlink's permissions do not override the target's permissions.
> * Modifying a symlink’s target means replacing it.
> * Reading the target file must still pass permission checks.


<div cursor="pointer" class="collapsible">Show Answer</div><div class="content_answer"><p>
<p>
<code>user-y</code> cannot rename or delete <code>/public/link.txt</code> because that requires write access to the <code>/public/</code> directory. The directory is owned by <code>root</code> and only grants read and execute permissions to others. Without write access to the directory, <code>user-y</code> cannot modify its contents, including renaming files.
</p>

<p>
<code>user-y</code> also cannot change the target path of the symlink. Although the symlink itself has permissions <code>lrwxrwxrwx</code>, modifying what the symlink points to is not done by editing the symlink in place. It requires **deleting** and **recreating** it, or overwriting it with a new symlink. Both actions are write operations on the <code>/public/</code> directory. Since <code>user-y</code> lacks write permission on <code>/public/</code>, they cannot modify or replace the symlink.
</p>

<p>
<code>user-y</code> cannot read the contents of <code>/home/user-x/secret.txt</code> by following the symlink. Even though the symlink is world-readable and accessible, dereferencing it leads to a file that has permissions <code>rwx------</code> and is owned by <code>user-x</code>. The kernel performs a permission check on the target file, and since <code>user-y</code> is not the owner and lacks permission, access is denied.
</p>


</p></div><br>
