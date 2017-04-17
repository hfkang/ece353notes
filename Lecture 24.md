#Lecture 24
March 10, 2017  

## How blitz works




#### Linear page table 
Kept in memory, has up to 2K entries. Each entry is 

Frame # (19) |unused(9)| Valid (1)|Writable(1)|Dirty(1)|Referenced (has been modified, or read) (1)
---|---|---|---|---|---

referenced bit set when page was modified or referenced. 

Each address space includes its own page table, pointed to be **PTBR** base and **PTLR** length. These are set in kernel by privileged instructions **ldptbr and ldptlr** 


The PTBR and PTLR registers are set on every context switch. 


## Blitz Managers 
The blitz OS relies on allocation of a fixed pool of resources, as opposed to dynamically allocating them during runtime. This is because we have no garbage collection, and thus no heap. 

### 1) Thread Manager
#### Interface 
`GetANewThread()`  
`FreeThread()`

### 2) Process Manager
#### Interface
`GetANewProcess()` : assign PCB 

### 3) Frame Manager 
#### Interface
`GetNewFrames()` allocate frames to the pcb's address space's page table. 


## Blitz Objects
### PCB
```c
fields
    pid: int
    parentsPid: int
    status: int 
    myThread: ptr to Thread 
    exitStatus: int
    addrSpace: AddrSpace 
```
### Class Thread
```c
fields
    regs: array[13] of int
    stackTop: ptr to *
    name: String
    status: enum
    initialFunction: ptr to function(int)
    initialArgument: int
    systemStack: array[SYS_STACK_SIZE] of int
    isUserThread: bool
    userRegs: array[15] of int
    myProcess: ptr to PCB
```

## Context Switchings 
```c
func Run(nextThread: ptr to Thread)
    var prevThread, th: ptr to Thread
    
    1) prevThread <- currentThread
    2) if prevThread.isUserThread
        saveUserRegs(prevThread.userRegs[0])
    3) currentThread = nextThread 
    nextThread.status = RUNNING
    4) Perform context switch Switch(prevThread,nextThread)
    5) Check if threadManager needs to destroy threads
    6) restore user regs of current thread
    7) set current thread page table if user thread

```

## Syscalls

Triggered by calling the DoSysCall routine, which puts the arguments in user registers r1-r5, indicates which syscall in r5. 

`syscall r5` will trap the CPU and indicate the syscall type is in register r5. 

`SyscallTrapHandler` transfers args from user regs to kernel stack, then calls `_P_Kernel_SyscallTrapHandler`

### A note on zombies
Zombie exist because a child process returns an exit status code for the parent process. This is saved in the child's PCB, so it must be kept around until the parent can process it. This us why when a child exits it turns into a zombie, and isn't cleared immediately. 

