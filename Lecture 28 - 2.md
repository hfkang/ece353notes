# Lecture 28 - 2
March 20, 2017

## File Systems
Used for virtualizing persistent storage. 

### HDD Anatomy
A hard drive has multiple platters with concentric tracks divided into sectors. Multiple platters can be read at the same time. 

For a 10,000 RPM drive,   
Seek Time: 6ms  
Settling Time: 0.5 - 2ms  

### Storage on DISK
|MBR| 
|---|
|pt1|
|pt2|
|pt3|

**MBR** Master boot record contains 1) boot code, 2) partition table. 

**partition** region of contiguous sectors. 

**sector** usually around 512 bytes. Reading *atomic*, guaranteed by manufacturer.

**blocks** are larger, around 1KB to 4KB and contain multiple contiguous sectors. Reads are not atomic 

### Multiple FS 
**Raw** partition for swap space has no file system

**Cooked** contains filesystem. 

#### Booting process 
Bootable partition starts with boot block, contains small program which reads in an OS from FS. On OS startup, BIOS reads from MBR sector and execs the boot block. 

|Boot Control Block|Volume Control Block|Directories / FCBs| Data Blocks| 
|---|---|---|---|

Q: Is C:\\ a bootable partition? 
 
## Files 
We handle files as _steams of bytes_.   
File systems view disk partition as an array of blocks 

### How to store?

#### Idea 1: Contiguous Allocation 
Put all blocks for a file in a contiguous region. 
+ Good perf on seq. reads  
+ Simple to implement  
- Suffers from **external fragmentation**. Must do periodic compaction (defragmentation).   
- Files cannot grow if placed in a gap  
As a result, this FS is used for CD/DVD and UDF.  

**extent** a portion of a file when the filesize is larger than max chunk supported by the filesystem. Each extent is stored contiguously.  
+ benefit from contiguous reads  
+ can be more flexible than cont. alloc

#### Idea 2: Linked List Allocation
Model each file as a sequence of blocks. The first word of each block points to the next block. 

|pointer|
|---|
|block contents |

*directory* indicates a filename, the first block, and last block.   
+ no external fragmentation  
+ easily extend files
- seq. reading is slow, but random access is worse!  
- the pointer makes the block size weird (4kb - 4 bytes for data) 
- damaged block can kill the pointer 

_mitigate_ by using **clusters**: a multitude of blocks as a unit. This increases **internal fragmentation** though.  

#### Idea 3: File Allocation Table 
The issues with LinkedList is that we store the pointer amongst the blocks. 
Resolve this by storing the pointers in a table at the beginning of the disk volume. _Keep this in memory_. 

|Block #| Next Block|
| ---   |   ---     |

|directory||
|---|---|
|filename|1|
A file is terminated with a -1 entry in the table. 

Characteristics:  
+ random access can be done via _in-mem search_
+ directory entries only point to first block.  
- on power outage, we lose consistency from FS to FAT as it was in memory. data corrupted. only can resolve via journaling FS.  
- entire table in mem. 20GB Disk→80MB FAT. As a result, we _cache open files_ only. This is what **open** syscall is used for. 

#### Idea 4: Index Alloc
Bring all pointers into a single _index block_. **Inode**: each file has its own bundle of indices, an array of disk-block addr. A directory contains the address of the index block for each file. 

|superblock|inode map|datablock map|inodes|datablocks|
|---|---|---| ---  |---|

Superblock stores **????**

##### Inodes
Approx 128-256bytes in size. 

- file attributes (permissions, timestamp, owner) 
- 10 pointers to blocks 
- pointer to a second-level block table

|inode|
|---|
|file attr|
|10 pointers (direct blocks)|
|pointer to block of pointers|

You can employ **double** or **triple** indirection to index a huge number of blocks. The hierarchy favours small blocks that can fit in 10 blocks. Studies indicate most files are small so a single level of indirection is usually sufficient for  our needs. 

**Strategy** only cache in RAM the inodes for currently open files. 

## Directories 
Directories need to list the **inode #** of all files in folder. 

### Searching for a file
We try looking at files in /usr/bin/blitz  

1. Locate root directory (inode 2)
2. Look up inode # for "usr" folder
3. look up inode # for "bin" folder
4. look up "blitz"

**improve perf** by caching results of previous searches. 

##### What if a file shows up in multiple directories? 
Have the directories point to the same inode. If file changes, we use new disk blocks. 

**hard link** both dir point to same inode (`ln`)  
**symlink** one dir has inode, other dir contain path to inode (`ln -s`)  

symlink is slightly _slower_ as they need to traverse the path to locate the inode. 

#### Application to Backups
Hardlinks are nice because if multiple snapshots reference the same file then we can just create more hardlinks. When no one references them anymore, we delete the inode. 

#### Handling deletions
For hardlinks, we must maintain reference count in an inode to see how many directories reference us. When count goes to zero, reclaim blocks. 

For Symbolic links, the actual file gets removed and the remaining symlinks break. 

## Opening Files via open()
1. The open() syscall passes filename to logical file system by searching the **system-wide open-file table** to see if it's already in use. 
2. If it is, then create a **per-process open-file table** entry. 
3. If not, search directory structure for the filename. (this is cached operation for performance) 
4. Once found the **File Control Block** ((inode)) is copied into the open-file-table. 

**System-wide open-file table** stores inode/FCB, and tracks # processes that are using it. 

**per-process open-file table** keeps track of which files a process has open, and it's current position, and access mode (r,w,)

### File Descriptor/File Handle
The open() call returns **index** to entry in our **ppoft**. Kernel pointers are not allowed!!!. 

We have 2^16. First three are mapped as:  
0: stdin  
1: stdout  
2: stderr   

#### Procedure
1. look in sys-wide openfile table 
2. look in storage for inode and put it in syswide openfile table
3. store in ppoft a links to sw-oft 

### Blitz File management
in blitz we dont really have files .. we have contiguous sectors. 

FCB contains 

- number of users 
- file location 


OpenFile contains

- ptr to FCB 
- numberOfUsers (parent/children)
- currentPosition 

PCB's fileDescriptor is the perprocess open file table. 

*Why do multiple processes point to same open file?*  
When we fork a process, the open files must be shared with the child, including the current position the parent was at. 

While the OpenFile object (and currpos) is shared, both processes can seek to any position they want so they can access the file range of files. 

**Q** could you use this mechanism as a form of a shared buffer? 

### Managing Free Blocks 
#### Bitmap method
Maintain bitmap, but it should be cached for good performance. 1.3GB with 512B blocks = 332KB to contain bitmap. 

#### Linked list method (?) 
Use a block to contain #'s of free blocks. Use one block number in the disk to point to the next disk block. 

#### Counter Method 
Keep track of  

1. address of first free block 
2. size of region 

RIP ZFS. 

## Performance of FS 
The unix/inode approach to hierarchial fs is very slow!  


Superblock bitmap | inode bitmap|data bitmap inodes|datablocks
---|---|---|---|---


### Creating a new file 
1. inode
2. inode bitmap
3. data block
4. data block bitmap
5. parent directory data blocks
6. parent dir inode 

This invokes a lot of random file accesses :(  
### Improving Speed by subgrouping the partition
Fast File System, ext2, ext3, ext4 address the performance issues by splitting the partition into smaller segments 

|block group 1| block group 2| block group 3|
| ---| --- | ---| 

Each block group contains an instance of the struct above.  A file is stored in one group. (huge files *not* supported) The super block is replicated across block groups for resilience. This keeps the *data pertaining to a file close together* so seek times are *reduced*. 

In FFS, these groups are called **cylinders**. 


### Improving Speed via Caching 
The **buffer cache** keeps blocks in memory for consumption by **read()** syscalls.  
Use LRU as cache replacement policy.  
In Linux, all remaining RAM is allocated to buffer cache. 

**free()** syscall in Linux shows how much ram is free, and how much is used by buffer cache. 

#### Handling Writes via Cache - Write Through 
aka **Synchronous write**, write to both cache and disk at same time. Preserves **cache consistency**.

#### Write Back 
**asynchronous writes** are done in the buffer cache, and dumped to disk later on.  
in Unix, the **update** daemon uses **sync()** syscall to force all modified blocks to disk every 30seconds. 

#### Read-ahead
Given spatial locality, we preemptively copy a block to the cache when nearby blocks have been requested. 
