#Lecture 24
March 10, 2017  

## How blitz works

#### Linear page table 
Kept in memory, has up to 2K entries. Each entry is 

Frame # (19) | Valid (1)|Write(1)|Dirty(1)|Referenced(1)|unusued(9)
---|---|---|---|---|---

referenced bit set when page was modified or referenced. 

Each address space includes its own page table, pointed to be **PTBR** and **PTLR**. These are set in kernel by privileged instructions **ldptbr and ldptlr** 


