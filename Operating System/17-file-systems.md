# File Systems, Inodes, Directories, and Links

## What is it?

A filesystem organizes persistent storage into named files and directories, tracks metadata and blocks, enforces permissions, and recovers from failures. Unix-like filesystems commonly use inodes to hold file metadata.

## Simple Explanation

Directories are labelled folders, file names are labels, and an inode is the record card containing the file’s metadata and where its content blocks live. A hard link adds another label to the same card.

## Hinglish Explanation

Directory filename ko inode number se map karti hai. Inode mein owner, permissions, size, timestamps aur data blocks ka reference hota hai—filename normally inode ke andar nahi hota. Hard link same inode ka another naam hai; symlink path ko point karta hai.

## Why Do We Need It?

Applications need durable named data, permissions, sharing, efficient lookup, and recovery after crashes. The OS needs a consistent mapping from path names to device blocks.

## How It Works

1. `open("/a/b/file")` resolves each directory component.
2. Kernel checks traversal/read/write permissions and finds final inode.
3. It creates a file descriptor referring to an open-file description.
4. `read`/`write` move bytes through page cache/filesystem/storage driver.
5. `close` releases descriptor; metadata/data reach durable storage according to filesystem/writeback/fsync semantics.

```text
directory entry: "report.txt" → inode 4312
inode 4312: owner, mode, size, timestamps, block pointers
block pointers → file content on storage
```

## Important Terminology

| Term | Meaning |
|---|---|
| File descriptor | Per-process integer handle for open file/socket/etc. |
| Inode | File metadata and data-location record in Unix-like FS |
| Directory | Name-to-inode mapping structure |
| Hard link | Another directory entry to same inode |
| Symbolic link | Separate file containing a target path |
| Journaling | Records metadata/data updates for crash recovery |
| Mount | Attach filesystem at a directory path |

## Example

`ls -li` shows inode numbers. `ln a b` makes hard link `b`; `ln -s a c` makes symlink `c`. Deleting `a` does not remove inode data while another hard link remains.

## Real-World Example

Log rotation can delete a log pathname while a process still has the old file open. Disk space is not released until the process closes the descriptor; `lsof +L1` can reveal this.

## Important Properties

| Property | Hard link | Symbolic link |
|---|---|---|
| Points to | Same inode | Target pathname |
| Cross filesystem | Usually no | Yes |
| Survives target rename/delete | Yes, data link remains | Can become dangling |
| Typical use | Same file name | Shortcut/reference |

### Common Misconceptions

* Filename is normally stored in directory entry, not as the inode’s one name.
* `rm` removes a directory entry; an open file’s data may remain allocated.
* A file descriptor is not the inode itself.

### What Interviewers Usually Test

Path resolution, descriptor/inode distinction, link semantics, filesystem durability, and “disk full after deleting file” debugging.

### Common Follow-up Questions

1. Hard versus symbolic link?
2. What is an inode?
3. Why can deleted file use disk?
4. What does `fsync` do?
5. What is mounting?

### Follow-up Answers

1. Hard links are additional names for the same inode; symlinks store a target path and can become dangling.
2. It is a filesystem metadata record containing properties and locations of a file’s data blocks.
3. A process may still hold an open descriptor, so the filesystem cannot reclaim the data until the final reference closes.
4. It asks the OS to flush relevant file data and metadata to stable storage according to filesystem/device guarantees.
5. Mounting attaches a filesystem’s root to a directory in the existing directory tree.
