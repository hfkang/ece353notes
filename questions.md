Paging

- Slides 20: Shared pages need a one-many mapping for when a frame gets updated. 
How does this one to many mapping work? Where is it stored?

- Q could you use this mechanism as a form of a shared buffer?

- Slides 20: Page 23: WHen the last CoW frame is being written to ( no other addr space is also referencing it) do we still copy or can we write to it directly? 

- Slides 21: Page 9: In the function LoadPageTableRegs Why is the ptlr given a value of numberOfPages*4? How does this let the page table grow over time?

- Slides 23: What is in a superblock?i
-
- How does RAID work tho
-
- Slides 27: How to reconcile the process->guestOS->VMM TLB entry? 
- If a TLB entry contains VPN|MFN, then how do you know which process it belongs to? Which OS it belongs to?
- Ie, does each OS have its own address space table? 
