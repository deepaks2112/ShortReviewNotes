# Operating Systems
## Inventory of a process
**Components of a process state -**
1. Process Identifier
2. Memory Contents
	1. Code and data (static)
	2. Stack and heap (dynamic)
3. CPU context (registers)
	1. Program Counter
	2. Stack pointer
	3. Current operands
4. File descriptors

## Creation of a process
* Allocation of memory (both static and dynamic)
	* Static (code and data) is loaded from disk onto the memory
	* Dynamic (stack and heap) are created and associated with the process
* Basic files like _stdin, stdout_ and _stderr_ are opened
* Program counter points at the first instruction

## Process states (broad)
1. New
2. Running
3. Ready (waiting to be scheduled)
4. Blocked (can not be run currently)
5. Dead (terminated)

## Data structures required for running processes
* Process list
* Each process represented using a ```struct```
* The ```struct``` is called **Process Control Block**
### Typical Contents of a PCB
* PID
* Process state
* Pointers to related processes
	* Parent process
* CPU context of the process
* Pointers to memory locations
* Pointers to open files

## Modes of execution
### User mode
A restricted mode where critical operations are not directly accessible, this ensures security to other processes and the OS itself.
### Kernel mode
An unrestricted mode where any code can be executed. This is the OS mode. All the OS code is executed in this mode. User programs can also request for services through _system calls_ which is served by the OS.

## A simple function call explained

## A kernel mode execution
* The **trap**  instruction to switch to kernel mode from user mode
* A separate **kernel stack** is used to protect from executing arbitrary code through the user stack
* At the end **return-from-trap** instruction to switch back to user mode

A **trap table** is set at **boot time** by the kernel - containing the reference to code of each of the system calls/routines.

A **system-call number** is assigned to each system call. The user program should place the desired number in a register. The OS has a service called trap handler to execute the corresponding code after examining validity of the system-call number.

## How OS regains control
* A process starts execution
* How can OS interfere and regain control?
	* **Approach 1:** Trust the process and wait until it itself places a system call or an exception is encountered, further provide a special system call to programmers to provide control to OS.
	* **Approach 2:** A mechanism to provide OS with control due to some exception (or interrupt) - the timer interrupt. This is done using hardware which periodically interrupts and the OS gets to regain control.

## Context switching
* Saving the context (state) of process so it can resume execution when the chance is provided again
* The OS saves the state of the process on the kernel stack
* Retrieves the state of the process to be executed next
* Involves saving the register contents and loading on to the registers

### A special case:
* How to tackle interrupts when handling another interrupt
* When handling an interrupt, interrupts are to be rejected

## Scheduling
* We have a **workload** (some property about the jobs that are to be scheduled)
* We have **policies** (to be applied based on the workload and the results we want)
* We have **metrics** (to evaluate how good or bad the policies perform under the given workload)
## Workload
* CPU only
* CPU and I/O both
* CPU intensive
* I/O intensive
* etc.
## Scheduling metrics
* **Turnaround time** = Completion time - Arrival time
* **Response time** = Time of first run - Arrival time
* **CPU utilisation**
* **Throughput** = Number of processes completed per unit of time
* **Waiting time** = Time spent by a process in the waiting queue

## Policies
### Non-preemptive scheduling
1. **First In First Out/First Come First Served**
	> Process gets to execute in the order they join the queue. The execution is not interrupted in normal conditions.
	
	> **The Convoy Effect**  - shorter jobs have to wait behind longer jobs
	
2. **Shortest Job First**
	> Sort the jobs in ascending order based on their running time and execution follows the same order
	
	> It is optimal when all the jobs arrive at the same time and are CPU only

### Preemptive scheduling
Stopping a running process before its completion and run another process in order to optimise some of the metrics

1. **Shortest Time to Completion First/Preemptive Shortest Job First/Shortest Remaining Time First**
	> The process which can complete earliest is allowed to run first
	
	> It can not capture **interactivity**

2. **Round Robin**
	> Run a process for a **time slice** and switch to the next job in the queue. Repetition of the same until all are finished

	> Sensitive to **interactivity** - minimises the average response time
	
	> **Time slice** - can not be made very small - switching overhead - a trade off

#### If I/O burst is also there
I/O of one job can coincide with CPU use of another process
## Smarter Scheduling - most generic (gets rid of many assumptions)
### Multi-level feedback queue
Two types of jobs:
* Interactive jobs (response time)
* Batch jobs (turnaround time)

Goal is to minimise both based on the job that is running without any prior knowledge about the job

**Adaptive scheduling** - scheduler learns while the job is running
### MLFQ uses multiple levels of round-robin
* Each of the round-robin queue is assigned with some priority
* The priority is used to determine which job to run next

Some basic rules -
* A job whose priority is higher is allowed to run
* Two job with same priority run in round-robin

Two approaches to assign priorities:
* User supplies it
* Scheduler learns from observed behaviour

Rules to change the priority:
* Fresh job is placed at highest priority
* Reduce priority if job uses up an entire time slice while running (CPU intensive jobs)
* Job giving up CPU before time slice is up stays at same priority level (I/O intensive jobs)

**Starvation** of CPU intensive jobs if there are too many I/O intensive jobs.
**Gaming** a CPU intensive job can present itself interactive job
**Change in behaviour** not handled if a CPU intensive job turns interactive after some time

Addition of new rule:
* After some time S, move every job to the topmost queue (solves starvation and change in behaviour)

S is a Voo-Doo constant:
> too high value - long running jobs could starve
> too low value - interactive jobs may not get proper share

To tackle gaming the scheduler issue:
Add new rule and the final set of rules are now:
1. Job with higher priority gets to run first
2. Jobs with same priority run in RR
3. Fresh job has the topmost priority
4. **[NEW]** Jobs are provided a time slice at a given level, reduce the priority when all of the time slice is used up
5. Move all the jobs to topmost priority after some time S

## Other scheduling policies
* **Lottery scheduling** - whoever wins runs and higher chance to win implies higher priority

## Inter Process Communication
The design goals for OS involves isolation of processes to ensure protection. Thus
* Processes have isolated memory image (not affected by other process)

Processes can communicate among them using 
* Shared memory
* Message passing
### Shared Memory
```int shmget(key_t key, int size, int shmflg)```
> Key parameter used to allocate same chunk of memory to the processes
> Prone to inconsistency, attacks, etc.

### Message Passing
* Using signals
* Using sockets
* Using pipes - the system call returns two file descriptors - written on one read from the other

## Virtualisation of memory
* Private and potentially infinitely large address space (as viewed by the processes)
* Single and finite memory (physically)

## Early systems
Separated the physical memory into two:
* For operating system
* For user programs (only one can run at a time or multiple)
### Design goals
* Effective utilisation
* Efficiency

One solution could be time sharing but this is very inefficient because memory is a lot slower than processor

## Virtual address space
Three parts
* Program code
* Heap (Downward growth)
	> dynamically allocated data
	
* Stack (Upward growth)
	> local variables, arguments to routines, return values, etc.

## Memory Management Unit (MMU)
* CPU 'knows' only *virtual addresses*
* Hardware 'knows' only *physical addresses*
* MMU translates virtual address to physical address and vice-versa

## Process Relocation
Approaches:
1. **Static relocation**- rewrite the addresses of instructions itself before loading it as a process (obvious protection issues)
2. **Dynamic relocation (Base)**- address translation by adding a fixed offset - the offset stored in ```Base``` register
	> PA = VA + Base
	Base can be changed to access OS or other processes

3. **Base+Bounds**- bounds limit the address space addressable and ensure physical address lies within address space
	> Internal fragmentation - memory between stack and heap not usable
	> Address space out of physical memory

## Free list
A linked list used by OS to track free parts of memory

## Segmentation
Pair of Base and Bounds register for each of the logical segment of address space (code, heap and stack).
A segment is continuous portion of address space of a particular length.
Need of segment identifier as stack grows upward so its base-bounds is its top.
Addresses consist of segment identifier and protection bits apart from the virtual address.
Granularity of segmentation:
1. **Coarse grained**-relatively large segments
2. **Fine-grained**-relatively smaller fragments (base bounds replaced with segment table and stored in memory)
 > **External fragmentation** due to variable size segments
 

## Paging
* Division of address space into fixed-size units called **pages**
* Similar division of physical memory into **page-frames**
* No external fragmentation

## Page table
* Records which **virtual page** in which **physical frame**
* Per process information
* Virtual Address = Virtual Page Number + Offset
* Physical Address = PT[VPN] + Offset
* Page tables stored in memory

## Page table contents
* Valid bit (accessible by process or not)
* Protection bit (permissions)
* Present bit (page frame in use or not)
* Dirty bit (updated page frame)
* Reference bit (accessed or not)
* PFN

### There is a factor 2 slow down as the memory is accessed twice

## Faster Paging
### Translation-Lookaside Buffer
* Cache of page table entries
* First check whether TLB contains the required data
* Workflow:
	1. Check the TLB if the entry exists or not
	2. If found, retrieve the corresponding PA 
	3. Access the memory
	4. If not found, access the page table to get the desired entry
	5. Update it onto the TLB
	6. Retry the instruction (different return from trap code) (PC should be the previous instruction)
## TLB contents

 * VPN
 * PFN
 * Valid
 * Protection
 * Address Space Identifier (indicates entry belongs to which process)

## Replacement Policy
* LRU
* Random
 
## Sharing of Pages
* Code and data part can be shared (same function in two files)

## Smaller page tables

* Increase the page size
> Wastage of useful memory - internal fragmentation

* Paging and segments
> Different page tables for different segments

* Multi-level page tables
> 1. Chop up the page table into page-size units
> 2. If all the page-table entries of a page are invalid do not allocate that page to page table
> 3. Extend it to multiple levels

* Hashed page tables
> Chaining of colliding entries reduce the page table size

* Inverted page tables
> Single page table containing entry about current owner of each of the physical pages
>  Search linearly for each of the pages

* Swapping page tables onto disk

## Persistence
## I/O devices
* Connect to CPU and memory via a bus
* Device interface (to abstract out the internal details) has the following parameters:
	* Status
	* Command
	* Data
* Read/write to I/O devices
	* Explicit instructions - ```in``` and ```out```
	* Memory mapped I/O - Registers or certain part of memory act as the device, OS reads/writes on that locations, memory hardware does the remaining work

## Polling Vs Interrupts
How can OS check device status?
Two approaches - 
1. **Polling**-wastes CPU cycles, can be optimised by executing some other job but can cause **livelock**
2. **Inerrupts**-use of programmed I/O, the job of polling is done by a controller, interrupts as and when required, some CPU cycles still wasted due to copying of data onto controller, can be solved by DMA (Direct Memory Access) - copies the data from the memory onto the PIO by accessing memory directly

## Device driver
* A piece of software in OS - knows in detail about the device

## Files and directories
### File
* **inode number** - identifier for OS
* inode number is the low level name
### Directory
* List of directories and inode numbers

### Directory Structure
* Tree-like
## File System Interface
* **create** using open and appropriate flag
* **read/write** using ```read()/write()```
*  writes are buffered in memory temporarily

## Hard links vs Soft links
* **Soft link** is like a pointer - nothing remains if the pointed file is deleted
* **Hard link** is a copy of the file - the original file remains after deletion of the link

## File Systems Implementation
**inode** or index-node is a ```struct``` containing -
* which data blocks comprise a file
* size of the file
* owner and access rights
* access and modification times
* etc.

**data blocks** contains the actual data of the file

**bitmap** to indicate which inode/data block is in use or free. Two types of bitmaps -
* Data bitmap
* inode bitmap

**superblock** contains the metadata about the file system like -
* number of inodes and data blocks
* address of inode table
* magic number - a bit sequence to identify the format of the file in the file system

## Inode
* **Direct pointers** store a few data blocks in the inode itself
*  **Indirect block** stored in inode for larger files
* **Multi-level index** two or three levels of indirect pointers

## File Allocation Table
* Linked list based approach
* Store next block pointer for each block

**[Fact]** The amount of I/O generated by open is proportional to the length of the pathname

## Open file table
* Global open file table
* Per process open file table

## Reducing cost I/O operations
* Caching
	* Static vs dynamic partitioning
* Buffering
	* Batch updation by delaying writes
	* Schedule of I/Os
	* Avoiding writes by delaying
