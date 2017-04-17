#Lecture 25 


### DoSysCall in UserRuntime.s

Move all arguments into register r1-r4.  
Move function code into r5.  
Perform syscall instruction pointing to r5.  
Store result contained in r1 on the stack. 

### Handling Syscall
SyscalTrapHandler prepares the syscall paramters by plaing them on the stack or use in the Kernel. 

The result is on the stack, so SyscallTrapHandler moves it from the kernel reg to the user reg. 

