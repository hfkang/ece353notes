# Lecture 31
March 27th, 2017

## Journalling FS 

### Crash Consistency 
Two interdependent data structures A,B need to be updated. If system crashes after one write, then the disk is left *inconsistent*. 

If a data block is written to but the block bitmap or inode are not updated, the disk is still considered *consistent*. Generally, when block bitmap and inodes disagree we have **inconsistency**

#### Solution 1 FSCK
1. scan superblock for sanity check
2. scan inode to build databitmap use this 
3. cehck reference count in each inode and check its consistent with directory. if file not found in any directories, add to lost+found 

### Solution 2 Journalling
Before overwriting the datastruct, write action in a log.  
Partition Structure:

|Super|Journal|Block Group 0| Block Group 1|...|Block Group N|
|---|---|---|---|---|---|

Journal Structure contains the inode/bitmap/datablock that we plan to write. 

|Transaction Begin|I[v2]|B[v2]|Db|TxE|
|---|---|---|---|---|

#### Crash during journalling
Suppose the journal entry was written out of order (due to caching) and our datablock was never committed. Because we have TxB and TxE, the inode/bitmap will be pushed along with *garbage data*. 

##### Sol: Write TxE at End
Modern disks support the **write barrier** mechanism, which makes sure everything is written before TxE, and TxE is written in an atomic action. TxE is made to be _512b_, or one sector (guaranteed by mfr to be atomic).

##### Sol: Incl TxE,TxB checksum
Many disks don't bother with writebarrier properly. So include a checksum in the TxE, TxB blocks. If the recorded checksum doesn't match checksum generated from the data in journal, we have corruption. 

On restart, replay transactions that were successfully recorded. If the log was never written, don't bother. 

#### Performance 
Each write has double overhead.  
To improve performance, buffer the updates in the memory cache then write them all to the journal at once. 

At some point after a journal was written to, we can write to disk at our leisure. This frees up space in our **circular buffer** in the journal. 

##### Improving Performance
First write datablocks somewhere. Then record metadata in the journal. 
