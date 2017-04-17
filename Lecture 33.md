# Lecture 33
March 31, 2017

## I/O System Arch

Hierarchal structure

```
CPU    Memory
_|_______|___  memory bus (proprietary) 
    |
    |
____|________________ graphics io bus (PCI)
    |        |
    |        GPU
    |    
____|______________     peripheral I/O bus 
    |    |       |
   USB  SATA    SCSI
```

### Canonical Device 
**Interface**:  Registers(Status, Command, Data)  
**Internals**:  MCU, DRAM/SRAM Memory, Other Hardware Chips  
A a very common example is RAID Controllers. 

**Device** actual hardware eg HDD  
**Device Controller** aka adapter - electrical hardware that connects to computer. It contains a chip/PCB in PCI slot with data, status, command registers. Serves as an interface between CPU/device. May use SATA, SCSI, USB, Thunderbolt.  
**Device Driver** device-specific software in O/S that controls the I/O device. 

#### Canonical Protocol: Polling 
Overall structure is to Wait -> data -> command -> wait  

```
while (STATUS == BUSY) 
     // wait until device is not busy
write data to DATA register 
write command to COMMAND reg
     // (this will start the device and execute the command)
while (STATUS == BUSY)
     // wait until device is done with your request
```
In this case CPU does the work so it's known as **Programmed IO**  
This can make computer slow as CPU wastes time waiting.

##### Should we use interrupts?
On one hand, interrupts let us do other things.  
But if a device is *really fast*, then it induces overhead and can cause **Livelock**. A huge number of packets arrive, each generating an interrupt. The OS will service all of these, starving user level process.  
The solution is to take a *hybrid* approach, and poll for a bit. If its not done yet then use interrupts. 

### DirectMemoryAccess
Issue: CPU still involved in all transfers when using interrupts.  
We use another device that can access memory on our behalf.  

Given data location, how many bytes, and destination the DMA will do the transfer. On completion, the device will raise an *interrupt*.  

### OS <> Device 
**I/O instructions** in/out instructions on x86 are privileged. specify port per device. Can't use C to write device drivers :C    
**Memory Mapped IO** hardware control reg mapped as memory locations.  
+No new instructions (just use load/store).  
+Can use C:  
+To protect just don't give user addrspace  
-command or data might be cached in the buffer cache, which needs to be disabled. This caching must be *selectively disabled*

### Hardware Abstraction 
```
Application Layer                                   USER
    -POSIX API(open,read,write,close,etc...)    -----------------    
File System                                         Kernel
    - Generic Block Interface (blockread/write)
Generic Block Layer                             
    - Specific Block Interface (protocol spec. r/w)
Device Driver (SCSI, ATA)
```

SCSI gives very rich error codes, which are not passed through abstraction layer :<  

characteristics that an os considers when abstracting i/o

- data transfer
- access method 
- transfer schedule
- sharing
- device speed
- i/o direction 

UNIX originally considered three kinds of devices:

- block device
- character device 
- network socket (new) 

#### Block Devices
- Store info in fixed size block, each with address.  
- read(), write(), seek() syscalls  
- Raw I/O still allowed (if you rly want to). 

#### Character Device
- Datastreams. Not addressable, nor seek()-able. 
- User get() and put() syscalls handle individual characters 

 
#### Network Devices
File descriptors also represent network sockets. In general have 2^16 entires in openfile table. 

Network i/o works on a nonblocking protocol. `select()` followed by `read() or write()`. 

`select()` checks to see if the buffer has room or data before doing anything. If it is unavailable, it returns immediately. You can stick it in an event loop. 

Asynchronous system calls have callbacks (notifications of i/o completion). eg: `epoll, kqueue, iocp` 

### Buffering
Writing from the modem (slow) to hard disk) means we need a buffer to _reconcile speed mismatch_. 

**Double Buffering** when a buffer is full, it is flushed to disk. At this point, switch to the other buffer which continues filling. 
