### ECE353 Midterm

#### Topics
1. Blitz
2. CPU Scheduling
3. Concurrency 
4. Context Switching 

#### Research Topics
- GNU Pthreads 


# Threads
**Thread** A state of executing code, including the PC, stack, and register values

**Program** The code and data sections indicating behaviour, and constants. As well it includes open files? Tbh that should be in the thread space as it is runtime specific 

Thread states: Running, Blocked, Ready, Exited 

**Yield** Current thread halts executing and gives CPU to other waiting threads

**Sleep** Current thread blocks execution, and waits for a resource eg: waiting for a buffer to empty a little

**Wakeup** Another thread can wakeup a sleeping thread because the resource is now available 

## Multithreading
The use of multiple threads with the *same address space*. Thread creation, termination, and switching are lighter than process equivalents because less information needs to be changed. 

### User Threads (N:1)
#### Advantages 
Context switching done at the userprocess level. This gives *cheap* switch, in the time it takes to do a procedure call. Lets you have custom scheduling algorithms per process space. *Eg: GNU Portable Threads*

#### Disadvantages 
System calls cannot be *blocking* lest all threads stop executing. 

### Kernel Threads (1:1)
Instead of running in user mode, kernel switches for you. Syscals are more *expensive* but can be *blocking*.

## Asynchronous Model
- Maintain a *single* thread and have *non-blocking* io.
- Maintain an event queue for responding to requests and io changes

**Callback** The mechanism by which the os triggers repsonses to i/o events 

# IPCommunication
***Assumption:*** each thread gets dedicated CPU

**Producer Consumer Problem** Given a multithreaded system with a producer thread sending data into a buffer and a consumer thread removing data, how do you ensure both can get equal access without corruption? 

**atomic** an operation that is all or nothing. It can either succeed for fail totally. 

**TSL** a hardware instruction that ensures no other devices can access memory bus. It lets you implement mutexes even if there are other CPU's active.

```KPL
acquire(lock L)
    R1 = TSL(L.state)
    while R1 == LOCKED
        R1 = TSL(L.state)
release (lock L)
    L.state = UNLOCKED
```

TSL(L.state) will set L.state = LOCKED, and return UNLOCKED to the one thread that executes it at the right time. The next thread that tries will get a R1 = LOCKED. 

# Context Switching

## Thread Manager
1. Allocates memory for stack
2. Select CPU and set PC to *starting function*
3. Set SP to stack 

**Q: Yield** must occur at the CPU level, but how?

1. 