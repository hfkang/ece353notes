# Lecture 32

## Data Integrity 
### Fail Stop model
 all or nothing, prevented via RAID. Aggregate multiple disks into one. Offers better performance, capacity, and reliability.  

RAID can only handle whole disk failure, not block level corruption.

Raid Level| Desc| Note
---|---|---
0| Striping| split data across phys drive 
1|Mirroring| redundancy (can be made striped)|
4| Parity| disk 5/5 stores parity bits for corresponding bit on remaining disks. Sometimes we'll overload parity disks with writes from all the other disks. 
5| Rotating Parity| Stripe parity segments across all disks. each disk responsible for only one parity chunk. 

### Some blocks fail
**Latent Sector Error** can be detected (maybe corrected) by disks using error correction codes. 

**Block Corruption** occurs when block contents bad but disk cannot detect/repair it. Caused by 

1. Buggy disk firmware writing to wrong loc
2. Transferred to disk across faulty bus. 

To resolve an LSE, reconstruct the block with Raid4/5 or get copy from RAID1.  

#### Block Corruption
4 or 8b summary of block contents. Computed when reading data and compared with checksum on disk. Either disk mfg makes 520b sectors (so u have 8b reserved for checksum), or keep all checksums in reserved area near front. 

#### Misdirected Writes
Store disk# and block offset with checksum. This is used to show where data was supposed to go. 

#### Lost Writes
If everything but datablock was written, we dont know what do. THus, include a checksum in the inode (metadata) so that if data was not written, a discrepancy will arise. 
