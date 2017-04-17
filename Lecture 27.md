# Lecture 27
###### March 17, 2017

### Optimal Page Replacement
Making an optimal page replacement policy is hard because we can't predict the future. 

### FIFO
Maintain list of order when all the pages came into memory.  
On page fault replace the oldest page  
**Issues**  
Oldest page might be constantly used.  
##### Belady's Anomaly
```
1,2,3,4,2,4,1,2,3,4,5
Increasing size from 3 to 4 frames causes more page faults!
```

Instead, learn from previous usage patterns. We need to store more information with help from hardware:  
**Referenced** bit set by CPU when page is read or written, and cleared by software.  
**Dirty** Set by CPU when page written to. Cleared by OS software.  
**Q:** can we determine usage patterns from the page status bits?

### FIFO with Referenced Bit
Still want to use FIFO, but look at referenced bit.  
If it's 0 (not used) then select for replacement.  
If it's 1, clear referenced bit and move to end as if new page.  
If every page was referenced, eventually all will be _dereferenced_ and available for replacement. It **degenerates to FIFO**. 
Performance


### Clock Algorithm
Maintain circular list of pages. Rotate through the list and and look for a page that doesn't have a referenced bit set (instead of moving pages around the list). It's a simpler implementation of FIFO with Referenced Bit. 

### Enhanced Second Chance (Not Freq. Used)
Replace page not recently used. Pages have referenced and dirty bit.  Periodically (eg. during timer interrupt) clear the referenced bit for all pages, so we know which pages were recently used. 

Class|Refernced|Dirty|Note
---  | ---     | --- |---
1|0|0|Use this for replacement!
2|0|1
3|1|0
4|1|1

### Least Recently Used
Keep track of *when* we last used the page, not just whether or not it was recent. 

c | a | d | b | e | b | a | b | c | d
---|---|---|---|---|---|---|---|---|---
  |   |   |   | R |  |   |   |   |   | 
  
#####   Implementation 1
Keep a stack of all the pages. But this is **expensive**. Must move elements around for every memory access. Now each memory read has incredible overhead. 
##### Implementation 2
MMU maintains counter, incremented with each memory read. When page entry used, update its timestamp. On page fault, look for oldest timestamp in the page table for replacement.  
**Problem** this is still expensive! Still double memory overhead. 
##### Implementation 3: Additional Reference Bits 
Maintain _8-bit_ counter in software for each page, initialized to 0.  
At each timer interrupt, shift the reference bit for the page into the high order bit of the counter. Discard lower bit.  
On age fault, evict page with lowest counter.  This can be done in hardware, so it's very fast?  

**Issues**  
limited time precision with which thread was least recently used. That and you can only maintain so much history. 

### Working Set Model
The set of pages a process needs *currently*. If working set is in memory, then no page faults will happen. 

**thrashing** when not enough frames to accommodate WS, page faults constantly happen. The user program gets nothing done. 

**demand paging** load pages as needed. good for large exe  
**prepaging** load the working set before letting it run, minimizing page faults. Much faster since there is _less page faults_. Gr8 for HDD! 

#### Determining the Working Set 
**prepaging** load the first X Pages for an executable, assuming spatial locality will ensure sequential access. 

Approximate with interval timer + reference bit:

```
T = 10,000 time units
Timer interrupts every 5000 time units 
Keep 2 bits for each page 
On timer int, copy and set the value of the ref bit to 0 
If one of the bits is 1, it's in the WS. It was used in the past time horizon. 
Evict pages with 00. 
```

#### *The* Working Set Algorithm
1. Hardware sets the R bit when page referenced. The virtual time is the time that the process has run on the CPU.
2. On a timer interrupt
    1. If R== 1, clear bit. Time of last use = current virtual time
3. On a page fault: 
    1. if R == 1, Time of last use  = current virtual time.  
    2. If R == 0 && age = (current virt time - time of last use) > T, *evict*
    3. If R == 0 && age <= T, record the page with greatest age. 
    4. If entire table scanned, *evict oldest page*.
    
