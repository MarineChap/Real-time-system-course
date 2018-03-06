# Real Time System

## Introduction : Definition for Real Time System
There are many explanations but my favorite is resuming like this in the book, "...."
> Real-time computing may be defined as that type of computing in which the correctness of the system depends not only on the logical results of the computation but also on the time at which the results are produced.  <br>

So, to resume, it means that we need an operating system optimized for predictability while general purpose is optimized for speed and throughput. <br>

For this kind of system, there are some specific characteristics for embedded systems and real-time system. <br>
For embedded systems, we need to have : <br>
- Ability to run on embedded hardware with the available drivers 
- A system scalable and reliable for the long time
- Development tools, possibly using cross-compilation.
- Communication mechanisms for synchronization and shared resources between processes

For real-time system, we need to have : <br>
- a predictable worst case execution time
- reaction time to interrupts
- enough priority levels
- Preemptive scheduling. 

A real-time system has tasks, which represents some work that it should perform, either **periodically**, **sporadically** or **aperiodically**. <br>
Definition can vary depending on the source and context. 
- **Periodic**: Task occurs at a regular time interval
- **Aperiodic**: Task can occur at any time, no limitations
- **Sporadic**: Task can occur at any time, but not more often than a specified limit. <br>

For each task, there are some constraints of deadline which can divide in 3 categories in function of the influence on the result : <br>
- **Hard real time** : Missing a deadline can fail the system
- **Soft real time** : Missing a deadline means the result is less valuable
- **Firm real time** : Missing a deadline means the result are no value

Some examples of real-time system are reactive systems, time-aware systems, time-triggered and event-triggered systems.

This article will be divided in 3 parts. The first talk about Hardware and OS. The second talk about multiprogramming with thread and process and memory management with in particular the virtual memory divided in some pages or segments. The last develops the problematic of real time scheduling. 

We also make a parallel with some OS:
- *QNX, VxWorks* which are proprietary OS for embedded and real-time systems, 
- *FreeRTOS*, which is minimalistic OS for microcontrollers
- *Windows, Android and Real time variant of Linux (RT-Linux, Xenomay, preempt-RT)*, which improve the real time capabilities of Linux

## II. Hardware and OS 

### Definition of OS : <br>
- Program that controls the execution of application programs and acts as an interface between applications and the computer hardware. It has three objectives :
    - **Convenience** – an OS makes a computer more convenient to use
    - **Efficiency** – An OS allows the computer system resources to be used in an efficient manner
    - **Ability to evolve** – An OS should be constructed in such a way as to permit the effective development, testing, and introduction of new system functions without interfering with service <br>

### Definition of some vocabulary related to the hardware : <br>
- **A translation lookaside buffet** is a small cache for page table entries 
- **A MMU** is a unit aiding the CPU for controlling access to memory
- *BusyBox* is a collection of useful command line program 
- *YOCTO and BUILDROOT* automate the build process for both the target file system and for development tools, i.e. crosscompile toolchain. 

## III. Multiprogramming

### Introduction of multiprogramming : 

**Objective** : <br>
- Maximize processor utilization
- Providing reasonable response times
- Let different tasks use different resources
- Allow tasks to communicate with each other

**Presentation of different types of kernel** : <br> 
- OS kernel 
  - Usual functions 
      - Process management – creation and termination, scheduling and dispatching, switching, synchronization and support for IPC, management of process control blocks
      - Memory management – allocation of address space to processes, swapping, page and segment management
      - I/O management – buffer management, allocation of I/O channels and devices to processes
      - Support functions – interrupt handling, accounting, monitoring

- Monolitic kernel 
  - implemented as a single process, with all elements sharing the same address space
  - Large kernel which allows all possibilities of complete operating systems including scheduling, file system, device drivers, and memory management.
  - The monolithic kernel is efficiency with direct communication between task . 
  - Con : if a kernel process fail, all the system fails but, in reality, it’s not a problem because, the system is become extremely stable and it rarely fails. 

- Microkernel
  - Small kernels with only important feature like memory management, process scheduling and communication service. 
  - The microkernel is smaller in the memory, modular, easy to add/remove functionality by loading dynamically module but his application is limited. 

### Definition of a process : <br>
-  An instance of a computer program, which contains the code of the program, the memory and other resources it uses. A process typically consists of one or more threads of execution. 
    - 5 state process model:
      - Running – process currently executing in the CPU
      - Ready – Process is in a queue waiting to be executed
      - Blocked – running process has to wait for an event, remain blocked until the event occurs, a separate queue for the blocked processes
      - New – Process has been created but not yet added to the pool of executable processes
      - Exit – Process that is removed from the pool of executable processes, but still exists in memory
    - Swapping – when there is no room for a process in main memory, it is swapped to disk -Seven-state process model includes suspend states for ready and blocked
    - Sleeping – the process is blocked while waiting on another process

- Process image
    - User data – the modifiable part of the user space. May include program data, a user stack area, and programs that may be modified.
    - User program – the program to be executed
    - Stack – Each process has one or more LIFO stacks associated with it. A stack is used to store parameters and calling addresses for procedures and systems calls.
    - Process control block – data needed by the OS to control the process

### Definition of a thread
Multiple threads in the same process share the same memory. Switching between threads within the same process is typically faster than switching between processes. A running thread can consist of one or many tasks of execution <br>


|        |Description|Pro    |Con   |
| :---:  | :---:     |:---: | :---: |
| User-level threads  |The application performs the thread management and scheduling. The kernel does not know that the thread exists | Fast thread switch, portability, custom scheduling for task | In a typical OS, many system calls are blocking. When a ULT executes a system call, not only is that thread blocked, but also all of the threads within the process are blocked. A multithreaded application cannot take advantage of multiprocessing. A kernel assigns one proves to only one processor at a time.  |
| Kernel-level threads  | The kernel schedule each of the threads in the process| The context switch of thread is less faster than ULT|  If one thread of a process is blocked, the kernel can schedule another to run, different threads of a process can run on different CPU cores |
| User and kernel combined  | All threads made in user space, multiple user level threads are mapped into a number of kernel threads| | |
    
To conclude, this part we can say *process relate to resource ownership, thread relates to program execution.*

### Definition of memory management 

- Protection and sharing resources
- Placement algorithm
    - Best fit
    - First fit
    - Next fit
- Paging and segmentation : 
    - Both divide the memory of a program into smaller parts. 
    - Both introduce some variant of a table that keep track of where the different parts of a process is stored

**Virtual memory**
- Pro: “unlimited” memory, only data that is actually used are placed in main memory
- Con: thrashing, meaning that a system spends most of its time moving pages/segments between main and secondary memory
- Problems for real time – if a part of a real-time process is suddenly in secondary memory, it can give an unexpected delay

**Fragmentation problem**
- Internal fragmentation – wasted internal space to a partition due to the fact that the block of data loaded is smaller than the partition
- External fragmentation – memory that is external to all partitions becomes increasingly fragmented. Lots of small holes in memory and memory utilization decline

**Trashing problem**
- throwing out a piece of memory just before needing it and having to go get it back almost immediately – swapping pieces rather than executing instructions

## IV. Problematic of Real-Time Scheduling 

Type of deadline scheduling :
- **Long-term scheduling** – which processes that are allowed to be added to the pool of processes being executed
- **Medium-term scheduling** – decide which processes are fully or partially swapped into main memory
- **Short-term scheduling** – decide which of the currently ready processes the CPU should execute – called the dispatcher – main objective is to allocate processor time to optimize certain aspects of system behavior

Criteria and definition :
- **Turnaround time** – interval between submission of a process and its completion
- **Throughput** – maximize number of processes completed per unit of time
- **Response time** – time from the submission if a request until the start of response
- **Deadlines** – when process completion deadlines can be specified, as many deadlines as possible should be met
- **Predictability** – a given job should run in about the same amount of time and at about the same cost regardless of the load on the system
- **Processor utilization** – Percentage of time the processor is busy
- **Fairness** – processes thread the same and no process suffer starvation
- **Enforcing priorities** – favor higher-priority processes
- **Balancing resources** – if a resource is overused, processes not needing it should be favored

### Scheduling deadline tasks

- **First come first serve** – dispatcher chooses the process that has been in the ready queue the longest
- **Round robin** – each process is given a time slice
    - Preemptive multitasking 
    -  a clock interrupted 
    - There aren’t priorities among tasks.
- **Shortest process next** – shortest executing time
- **Shortest remaining time** – preemptive version of SPN
- **Highest response ratio next** – dispatcher chooses process from execution time and time waited in the queue *R = (w+s)/s*
    - *R = response ratio
    - w = time waited in queue
    - s = service time*
- **Feedback** – decrease priority each time process is preempted, does not require knowledge of execution time

### Scheduling periodic tasks

**Simple task model**
- Fixed set of tasks (No sporadic tasks. Not optimal, but can be worked around)
- Periodic tasks with known periods (Realistic in many systems)
- The tasks are independent (Completely realistic in an embedded system)
- Overheads, switching times can be ignored (Depends)
- Deadline == Period (Inflexible, but fair enough)
- Fixed Worst Case Execution Time : Not realistic to know a tight (not overly conservative) estimate here
- And in addition: Rate-Monotonic Priority ordering

**Fixed priority Scheduling (FPS)**
- If preemptive, then the highest priority available task will always run
- Optimal for periodic, independent tasks 
  - The shorter the period, the higher priority
- *Pro* : more predictable when the system gets overloaded

**Earliest deadline first scheduling (EDF)**
- Ability to dynamically change the priority of the task based on the current deadlines
- Allow higher CPU utilization (test d’utilisation <1 au lieu de maximum ln2)  
- *Con* : more difficult to implement than FPS and difficult to incorporate task without deadline

**Comparison between FPS / EDF** 
- FPS is easier to implement as priorities are static
- EDF is dynamic and requires a more complex runtime system which will have higher overhead
- It is easier to incorporate tasks without deadlines into FPS; arbitrary deadline is more artificial.
- Easier to incorporate other factors into the notion of priority rather than deadline
- FPS more predictable during overload situations, but EDF gets more out of the processor

**Rate Monotonic scheduling**
- Deadline monotonic priority 
  - Rate monotonic is not suitable when deadline is shorter than the period (D < T). Not the same as EDF, as priorities are still static. Same as rate monotonic when D = T.
  - Adding block time to response time analysis
– the usage function: 1 for shared resources that are used by lower and higher (than i) priority tasks. 0 for others. Ri = Ci + Bi + Ii

### Scheduling aperiodic task : 

In a rate/deadline monotonic scheme, an aperiodic task could be given the lowest priority – suffer from starvation. <br>
For better response for aperiodic tasks, we use Execution-time servers who maintain a budget for clients <br>

**Periodic/Polling server**
- each period the computation time budget is available for any pending clients 
- If a client needs more, it will continue next period. If one or more clients are pending, they will be scheduled within the server. 
- The budget has to be used when the server is running or it is wasted. <br>

**Deferrable server**
- Only lose budget when used. 
- Better response time for aperiodic tasks. 
- Back-to-back, where one budget is used just after another. <br>

**Sporadic server**
- Budget is replenished one-time period after it has started to be used.

### Microprocessor scheduling

Multicore computers are computers with multiple CPU cores on the same physical CPU chip which share memory and usually operating system.

We have two kinds of architecture for the assignment:  
- **Master/slave architecture**: Kernel scheduling always run on the same master core
- **Peer architecture** : Kernel scheduling function can run on any core.

**Assignment of processes to cores** :
- Static assignment to a core
- Global queue
- Dynamic load balancing

To conclude, we talk about **Real-time on multicore** :
- Difficult since another potential uncertainty is introduced
- Moving a thread from one core to another will cause a longer and more unpredictable delay than normal scheduling 

### Inter-process communication and deadlock

**Race condition** : multiple threads or processes read and write a shared data item and the final result depends on the relative timing of their execution <br>

**4 conditions for deadlocks and how avoid this**: <br>
- Mutual exclusion
- No preemption 
- Hold and wait 
- Circular wait

**Deadlock prevention**:
- *No preemption* : allow the OS to force a thread to release its resource
    - Not always possible, resources can be left in an unpredictable state
- *Hold and wait* : can be removed by forcing a thread to request all resources at the same time
    - Unnecessary locking of resource
    - Not always possible to know which resources are needed in the future
- *Circular wait* : define an order in which the resources must be requested
    - If all process requests resources in the same order, a circular wait is impossible
    - Unnecessary locking of resources

**Deadlock detection**: 
- The system will instead check if it has any deadlock periodically
- If it’s detected, the system has to be recovered : 
  - *Abort all deadlock* processes
  - *Return each deadlock* process to a previous checkpoint but it is not always possible and it is also possible that the deadlock will recur. 
  - *Abort deadlock thread one by one* until the deadlock no longer exists. The order of aborting can be decided by what is considered the smallest cost.
  - *Preempt resources* until the deadlock no longer exists. 

**Deadlock avoidance** : 
- More permitting than deadlock prevention
- Doesn’t allow for deadlock to occur as deadlock prevention
- *Process initiation denial* : Not optimal as is assumed the worst
    - Prevent a process to start if it is likely to cause a deadlock
    - Check that the maximum use of resources from all existing processes and the process that wants to start are less than the available resources
- *Ressource allocation denial*
    - Only allow a process to request a resource if that does not lead to a unsafe state that could cause deadlock

### Problem of Priorities Inversion  

This problem arrives when a process with less priority blocks a resource used by a process with an important priority. <br>

- **Priority inheritance** – When a process with an important priority ask for a resource blocked by another process, the priority of the first process is to add temporary at the priority of the process which used the resource. It reduces the effect of priority inversions but does not prevent deadlocks and makes it easier to calculate the worst case blocking time.

- **Immediate ceiling priority protocol (ICPP)** – each shared resource is assigned the priority of the highest priority task that uses it. When a task is allowed to access a shared resource, it will inherit its priority. This means that when a task starts using a resource, it is guaranteed to be uninterrupted until it is done with the resource. It also minimizes the time a resource is locked.

- **Original ceiling priority protocol (OCPP)** – a task inherits the priority of a resource ONLY if it blocks a higher priority task. A task can only access a shared resource if its priority is higher than the priority of all locked resources. Does not need to know the priority of the tasks in advance.

**Comparison between ICPP and OCPP** : 
- ICCP is easier to implement than OCPP, which has to keep track of what is blocking what. ICCP leads to less context switches as blocking is prior to first execution. 
- ICPP requires more priority movements as this happens with all resource usage. 
- OCPP changes priority only if an actual block has occurred.

## V. Presenting of each OS 

### Android 
Same as Linux kernel 
- Applications
    - Activities
    - Services
    - Content provider
    - Broadcast receivers
- Interprocess communication

### Linux
- Free, open-source, scalable and not native support for the real-time hardware
- Worst case isn’t deterministic and interrupt can be interfered 
- Support many architecture adaptability 
- Modular monolithic kernel module Ex: device drivers (interaction with hardware and make one or more files in the /dev folder)
- Linux page replacement : Least frequently used policy 
- Kernel memory allocation: Buddy algorithm
- Concurrency 
  - spinlock =>  user-system:  mutual exclusion mechanism in which a process executes in an infinite loop waiting for the value of a lock variable to indicate availability
    - fast because without switch context but not protected by threads in the user space 
  - mutex => kernel system : protected from all threads but more weight 
  - Semaphore
  - Barriers : Enforce order in which instruction is executed

Add a real-time support hardware :
- **Patch PREMPT_RT** 
  - Reducing the size of critical section in the kernel
  - Critical sections : Parts of the code in the Linux kernel that has to execute atomically. This means that when entering this critical section, interrupts is disabled and the kernel code will not be preempted. When a critical section has an unbound duration, then it means that we cannot know how long these critical sections, when it is not possible to preempt the kernel, lasts. This is a concern for real time as a real-time task might be prevented from starting up if it is triggered during this time.

- **Dual kernel approach (Xenomai)** : 
  - Introduce a small (often called micro or nano) kernel between the hardware and the Linux kernel. This real-time kernel intercept all interrupts and communication with the hardware, and can handle these before deciding whether they should be forwarded to Linux.
  - The Linux kernel is scheduled as a process with the lowest priority, meaning that the Linux kernel only runs when there are no real-time tasks that need to run. This could cause starvation of the Linux kernel if the real-time tasks are using most of the CPU capacity. 
  - Cons: Tasks that runs in the real-time context does not have access to all of the resources (drivers, services, etc.) of the Linux system. In order to access some of these, it will lose its real-time priority and there will be a context switch penalty. Programming real-time tasks can therefore be challenging, and will often require some special experience. Drivers might have to be re-implemented for the real-time system in order to access hardware without the Linux kernel. 

### Windows

It is written in C but has object-based design and really using for the real time but it is **proprietary and expansive**

**Kernel-Level**  
- Kernel controls how the processor is used - schedule threads, exception and interrupt handling (*Similar to micro-kernels*)
- Executive provide - the core OS services/run has a kernel-level thread. 
  - Memory management
  - Create, manage and delete processes and threads
  - Manage I/O, IPC
  - Security
  - Hardware abstraction layer (between hardware and kernels) - Make each computer’s system bus, DMA controller, interrupt controller, system timers, and memory controller look the same to the Executive and Kernel components.
  - Device drivers
  - Windowing and graphics system 
  
 **User-mode process**
- Special system processes : manage the system, session manager and authentification subsystem
- Service processes : « Server » program (run in background)
- Environment subsystems : difference « os personalities » (Win32, POSIX)
- User applications

**Window thread** - similar to other systems
- Fibers are low-weight threads that are run and scheduled within a thread. 
- 5-states models

**Windows Scheduling**
- RT fixed priority class (16 to 31), Round robin
- Variable priority class (0 to 15) 

**Multiprocessor scheduling**
- Any thread can run on any processor
- Window dispatcher objects are used for synchronization

**Concurrency** 
- Critical section similar to mutex
- Slim reader-writer - like critical section
- Condition variable
- Lock-free synchronization - atomic access

### VxWorks 

Most real-time OS used 
Support POSIX / Synchronization
Microkernel but without task communicate directly 
Expansive system - Originally no protection memory 

**New version** : 
- Memory management
  - Real-time process : task similar to thread. All tasks are scheduled independently by the kernel (like QNX)
  - Non-overlapped memory model
- Task model - Task can be in more than one state at a time
  - Ready
  - Suspended
  - Delayed
  - Pended

- Scheduling
  - Single-core : preemptive/ highest priority / round-robin (same as QNX)
  - Multi-core: 
    - All tasks can run on any CPU core by default
    - A task can be given a CPU affinity – a task will only run on that CPU core (same memory cache – fewer data moved)
    - A task can reserve a CPU core – only this task can run on that CPU core 

### FreeRTOS

Minimalists OS for micro controllers and can run on many platforms
**Co-Routines** – intended for use on small processors that have severe RAM constraints, share a single stack
**Task model**: 
- running
- ready
- blocked
- suspended
  
**Scheduling**
- Either cooperative or preemptive
- Scheduler activated by a timer interrupt every clock tick
- Equal priority tasks are scheduled with Round-Robin

**message passing**
- semaphores mutexs and queues 
*Cons* - For better or worse, a different type of system than the other we have covered


## Conclusion : Comparison of OS

|        |When to use|Pro    |Con   |
| :---:  | :---:     |:---: | :---: |
| No OS |Simple applications or system which only doing one (or very few) things simultaneously   |Full control can be deterministic, No CPU and memory is spent on OS, on low-powered microcontrollers, Some low-level operations are much simpler than an OS | Tricky to do multiple things, often need to read CPU datasheet to use features, Often not scalable |
| Free RTOS | Simple applications and when you need multitasking, etc. on a simple microcontroller   |The advantages of bare metal at the same time as simple operating system capabilities, operating system for hardware that can’t run other operating systems| For better or worse, a different type of system than the other we have covered|
| Android  | On hand-held devices and graphical systems in ARM CPU    | Wide adoption, many apps, libraries, etc. available, Linux kernel customized and minimized for hardware, every app run within its own virtual machine| Not intended for real-time system or control systems |
|Linux | Soft real time, when developers have Linux experience and many different programs and libraries are needed/useful   |Support for many platforms and system types, large user community, scalable and configurable, many free and open-source tools, programs, libraries | Not hard real time, unpredictable periods when the kernel is non-preemptive, costly to spend time on setting up the system instead of buying something that just works |
| Real time Linux | Linux with hard real time and Mixed critically systems   |A fully functional Linux OS with addition of Hard RT, Separation between RT task and regular Linux processes| RT task cannot fully use the Linux system, Difficult to set up experience with Linux in not directly applicable for dev with RT task |
| Window  |  Developers have windows experience, Existing Windows code   |Embedded compact has suitable capabilities for hard real time, many tools, prog, libraries| Cost, Consumer version is not suitable for embedded/RT|
| QNX  | When a proven, commercial real-time system is required, distributed control systems.    |Hard real time, Specialized development tools, Microkernel, Customizable|Cost, Environment is unfamiliar for many |

VxWorks is similar to QNX but arguably lower weight





