# Lecture 26
March 15, 2017

Page miss is 1000x slower than accessing from ram. Avoid this with a _smart page replacement policy_

Paging strategies depend on reference locality:  
**spatial locality** programs tend to access small, contiguous segments of their memory in an approximately sequential order.  
**temporal locality** memory access recently will be used again.

##### Random Page Replacement 
Very simple. Too simple. 

##### Belady (optimal) 
Evict the page not needed for the longest time in the future. Assume fortune telling capabilities.
 
**cold start** the interval during which you fill up an empty cache

##### FIFO
How do you know which page is the oldest?  
Maintain a linked list.  
On a page fault, swap the page at the front of the list. 

**Issues** Oldest page may be used again, causing another page fault. Pages used consistently will suffer. 
