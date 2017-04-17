# Lecture 28
March 20, 2017

Page Replacement Algos Covered thus far:

- Belady
- FIFO (subject to belady's anomaly)
- Enhanced SC
- SC/Clock
- LRU
- ARB 
- Working set 

## Local vs Global Replacement 
###### Ch 8.5.3

Processes require different numbers of frames.  
If the # page faults too high, need to allocate more frames. If too low, allocated too many frames - starving another process. 

**Page Fault Freq** provides estimate of how well a program's working set is being satisfied. 

```
f = f + 1 // increment on fault
// every second, 
ff =  (1 - a) * ff + a * f,      f = 0 
(0 < a < 1, when a -> 1, history is ignored)
```
**goal** is to have page fault frequency to be equal for all processes. 

### Paging Daemon
Run the replacment algo when pool is running low. Write out dirty pages and free them. Pages not reallocated can still be used. 

### Windows 
Operate on local page replacement (variation of Clock algorithm). 
If amount of free pages  below threshold, eval # pages allocated to a process:

- if it is more than the minimum, evict until it reaches the minimum.
