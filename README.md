# Concurrency, multi-threading and parallel programming concepts

This document describes various concurrency, multi-threading, parallel programming methods and concepts.

**This document is meant to be read from beginning to end. You are welcome to contribute to this document.**

*Note: This document mainly discuss modern OS and hardware.*

## [Processes](http://en.wikipedia.org/wiki/Process_(computing\))

Processes are [operating system](http://en.wikipedia.org/wiki/Operating_system) (OS) managed and in case of a modern OS they are truly concurrent in the presence of suitable hardware support ([multiprocessor](http://en.wikipedia.org/wiki/Multiprocessor) and [multicore](http://en.wikipedia.org/wiki/Multi-core_processor) systems).

Processes are scheduled by the operating system's [scheduler](http://en.wikipedia.org/wiki/Scheduling_(computing\)). They may be made up of multiple [threads of execution](http://en.wikipedia.org/wiki/Thread_(computer_science\)) that execute instructions concurrently.

Processes exist within their own [address space](http://en.wikipedia.org/wiki/Address_space). No other process can read or write to another one's memory, because the OS secures this with [process isolation](http://en.wikipedia.org/wiki/Process_isolation) as the OS is the one who manages the processes.

Processes are heavy and spawning many of them (in contrast to other concurrency models) is not recommended. Creating a process requires creating an entirely new [virtual address space](http://en.wikipedia.org/wiki/Virtual_address_space). 

[Context switching](http://en.wikipedia.org/wiki/Context_switch) between processes requires a lot of work. This means saving of the entire CPU state (all [processor registers](http://en.wikipedia.org/wiki/Processor_register) that were in use, [program counter](http://en.wikipedia.org/wiki/Program_counter), etc.) into [PCB](http://en.wikipedia.org/wiki/Process_control_block) (usually). Context switching also involves switching the [MMU](http://en.wikipedia.org/wiki/Memory_management_unit) context and keeping OS related data.

Processes can't talk to each other directly (in modern OS). Instead the OS provides facilities for [inter-process communication](http://en.wikipedia.org/wiki/Inter-process_communication).

Typical ways of inter-process communication involve [files](http://en.wikipedia.org/wiki/Computer_file), [signals](http://en.wikipedia.org/wiki/Signal_(computing\)), [sockets](http://en.wikipedia.org/wiki/Berkeley_sockets), [message queues](http://en.wikipedia.org/wiki/Message_queue), ([named](http://en.wikipedia.org/wiki/Named_pipe)) [pipes](http://en.wikipedia.org/wiki/Pipeline_(Unix\)), [semaphores](http://en.wikipedia.org/wiki/Semaphore_(programming\)), [shared memory](http://en.wikipedia.org/wiki/Shared_memory) and even [memory-mapped files](http://en.wikipedia.org/wiki/Memory-mapped_file).

Processes are all about [preemptive multitasking](http://en.wikipedia.org/wiki/Computer_multitasking#Preemptive_multitasking.2Ftime-sharing) (today) meaning that the OS decides when a process is preempted ("goes to sleep") and which process goes "alive" next.

## [Threads](http://en.wikipedia.org/wiki/Thread_(computing\))

Threads, like processes, are also OS managed. Threads share the same address space of their [parent process](http://en.wikipedia.org/wiki/Parent_process). This means that processes can spawn threads indirectly using OS provided functionality (e.g. [CreateThread](http://msdn.microsoft.com/en-us/library/ms885186.aspx) or [pthread_create](http://www.kernel.org/doc/man-pages/online/pages/man3/pthread_create.3.html)).

On a single processor, multithreading generally occurs by [time-division multiplexing](http://en.wikipedia.org/wiki/Time-division_multiplexing) (as in [multitasking](http://en.wikipedia.org/wiki/Computer_multitasking)): the processor switches between different threads. This [context switching](http://en.wikipedia.org/wiki/Context_switch) generally happens frequently enough that the user perceives the threads or tasks as running at the same time. On a [multiprocessor](http://en.wikipedia.org/wiki/Multiprocessor) (including [multi-core](http://en.wikipedia.org/wiki/Multi-core) system), the threads or tasks will actually run at the same time, with each processor or core running a particular thread or task.

Many modern operating systems directly support both [time-sliced](http://en.wikipedia.org/wiki/Preemption_(computing\)#Time_slice) and multiprocessor threading with a [process scheduler](http://en.wikipedia.org/wiki/Scheduling_(computing\)).

Communication between threads is far simpler than [inter-process communication](http://en.wikipedia.org/wiki/Inter-process_communication) between processes. This is mainly due to the shared memory, but also due to the fact that the [strict security constraints](http://en.wikipedia.org/wiki/Process_isolation) the OS puts on processes do not exist with threads.

Threads are generally said to be "lightweight", but that is relative. Threads have to support the execution of [native code](http://en.wikipedia.org/wiki/Machine_code) so the OS has to provide a decent-sized [stack](http://en.wikipedia.org/wiki/Stack-based_memory_allocation), usually measured in megabytes. In Windows, the [default stack reservation size](http://msdn.microsoft.com/en-us/library/windows/desktop/ms686774(v=vs.85\).aspx) used by the linker is 1 MB. In Linux, the typical [thread stack size](http://www.kernel.org/doc/man-pages/online/pages/man3/pthread_create.3.html) is 2 MB. This means that in Linux, creating 1000 threads would equal to memory usage of ~2 GB, without even beginning to do any actual work with the threads.

Threads are "lightweight processes", not "lightweight" themselves. They require less resources to create and to do context switching as opposed to processes, but it still is not cheap to have many of them running at the same time.

While thread context switching still involves restoring of [the program counter](http://en.wikipedia.org/wiki/Program_counter), CPU [registers](http://en.wikipedia.org/wiki/Processor_register), and other potential OS data, they do not need the context switch of an [MMU](http://en.wikipedia.org/wiki/Memory_management_unit) unlike processes do, because threads share the same memory.

Like processes, threads are about [preemptive multitasking](http://en.wikipedia.org/wiki/Computer_multitasking#Preemptive_multitasking.2Ftime-sharing) and the OS decides when they are preempted. To avoid preemption with threads, [mutex](http://en.wikipedia.org/wiki/Mutual_exclusion) can be used. Windows also offers support for [Critical Sections](http://en.wikipedia.org/wiki/Critical_section) with two functions [EnterCriticalSection](http://msdn.microsoft.com/en-us/library/ms885212.aspx) and [LeaveCriticalSection](http://msdn.microsoft.com/en-us/library/bb202768.aspx).

Since memory/data is shared among threads in the same process, applications frequently need to deal with [race conditions](http://en.wikipedia.org/wiki/Race_conditions). [Thread Synchronization](http://en.wikipedia.org/wiki/Synchronization_(computer_science\)#Thread_or_process_synchronization) is needed. Typical synchronization mechanisms include [Locks](http://en.wikipedia.org/wiki/Lock_(computer_science\)), [Mutex](http://en.wikipedia.org/wiki/Mutex), [Monitors](http://en.wikipedia.org/wiki/Monitor_(synchronization\)) and [Semaphores](http://en.wikipedia.org/wiki/Semaphore_(programming\)). These are concurrency constructs used to ensure two threads won't access the same shared data at the same time, thus achieving correctness.

[Locks](http://en.wikipedia.org/wiki/Lock_(computer_science\)) are usually advisory locks, where each thread cooperates by acquiring the lock before accessing the corresponding data. Most locking designs block the execution of the thread requesting the lock until it is allowed to access the locked resource. A [spinlock](http://en.wikipedia.org/wiki/Spinlock) is a lock where the thread simply waits ("spins") until the lock becomes available. It is very efficient if threads are only likely to be blocked for a short period of time, as it avoids the overhead of operating system process re-scheduling. It is wasteful if the lock is held for a long period of time. Careless use of locks can result in [deadlock](http://en.wikipedia.org/wiki/Deadlock) or [livelock](http://en.wikipedia.org/wiki/Livelock). In many programming languages Locks are the easiest synchronization mechanism to use.

[Monitors](http://en.wikipedia.org/wiki/Monitor_(synchronization\)) prevent blocks of code from simultaneous execution by multiple threads just like Locks. In fact, Locks are sometimes based on Monitors, like in case of C#. Monitors provide a mechanism for threads to temporarily give up exclusive access, in order to wait for some condition to be met, before regaining exclusive access and resuming their task.

A [mutex](http://en.wikipedia.org/wiki/Mutex) is similar to a monitor; it prevents the simultaneous execution of a block of code by more than one thread at a time. In fact, the name "mutex" is a shortened form of the term "mutually exclusive." Unlike monitors, however, a mutex can be used to synchronize threads across processes. When used for inter-process synchronization, a mutex is called a named mutex because it is to be used in another application, and therefore it cannot be shared by means of a global or static variable. It must be given a name so that both applications can access the same mutex object. Mutexes unlike Locks or Monitors, can be used for inter-process synchronization.

[Semaphores](http://en.wikipedia.org/wiki/Semaphore_(programming\)) is a variable or an abstract data type that provides a simple, but useful abstraction for controlling access by multiple processes to a common resource in a parallel programming environment. Thus, Semaphores are not used for **intra-process** synchronization, but only **inter-process** synchronization. A Semaphore is essentially a record of how many units of a particular resource are available, coupled with operations to safely (i.e. without [race conditions](http://en.wikipedia.org/wiki/Race_conditions)) adjust that record as units are required or become free, and if necessary wait until a unit of the resource becomes available.

Semaphores which allow an arbitrary resource count are called **counting semaphores**, while semaphores which are restricted to the values 0 and 1 (or locked/unlocked, unavailable/available) are called **binary semaphores**. A mutex is essentially the same thing as a **binary** semaphore, and sometimes uses the same basic implementation. However, the term "mutex" is used to describe a construct which prevents two processes from executing the same piece of code, or accessing the same data, at the same time. The term "binary semaphore" is used to describe a construct which limits access to a single resource on the system.

## Green Threads


## Protothreads


## Fibers


## Coroutines


## Goroutines


## Actors


## Isolates


## Callbacks


## Generators


## Events

