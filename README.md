# Concurrency, multi-threading and parallel programming concepts

I wanted to understand concurrency, multi-threading, parallel programming and such concepts so I did some digging. **You are welcome to contribute to this document.**

This document gives you the explanations for all types of concurrency, multi-threading or parallel programming concepts.

*Note: This document mainly discuss modern OS and hardware.*

## [Processes](http://en.wikipedia.org/wiki/Process_(computing\))

Processes are [operating system](http://en.wikipedia.org/wiki/Operating_system) (OS) managed and in case of a modern OS they are truly concurrent in the presence of suitable hardware support ([multiprocessor](http://en.wikipedia.org/wiki/Multiprocessor) and [multicore](http://en.wikipedia.org/wiki/Multi-core_processor) systems).

Processes exist within their own [address space](http://en.wikipedia.org/wiki/Address_space). No other process can read or write to another one's memory, because the OS secures this with [process isolation](http://en.wikipedia.org/wiki/Process_isolation) as the OS is the one who manages the processes.

Processes are heavy and spawning many of them (in contrast to other concurrency models) is not recommended. Creating a process requires creating an entirely new [virtual address space](http://en.wikipedia.org/wiki/Virtual_address_space). 

[Context switching](http://en.wikipedia.org/wiki/Context_switch) between processes requires a lot of work. This means saving of the entire CPU state (all [processor registers](http://en.wikipedia.org/wiki/Processor_register) that were in use, [program counter](http://en.wikipedia.org/wiki/Program_counter), etc.) into [PCB](http://en.wikipedia.org/wiki/Process_control_block) (usually). Context switching also involves switching the [MMU](http://en.wikipedia.org/wiki/Memory_management_unit) context and keeping OS related data.

Processes can't talk to each other directly (in modern OS). Instead the OS provides facilities for [inter-process communication](http://en.wikipedia.org/wiki/Inter-process_communication).

Typical ways of inter-process communication involve [files](http://en.wikipedia.org/wiki/Computer_file), [signals](http://en.wikipedia.org/wiki/Signal_(computing\)), [sockets](http://en.wikipedia.org/wiki/Berkeley_sockets), [message queues](http://en.wikipedia.org/wiki/Message_queue), ([named](http://en.wikipedia.org/wiki/Named_pipe)) [pipes](http://en.wikipedia.org/wiki/Pipeline_(Unix\)), [semaphores](http://en.wikipedia.org/wiki/Semaphore_(programming\)), [shared memory](http://en.wikipedia.org/wiki/Shared_memory) and even [memory-mapped files](http://en.wikipedia.org/wiki/Memory-mapped_file).

Processes are all about [preemptive multitasking](http://en.wikipedia.org/wiki/Computer_multitasking#Preemptive_multitasking.2Ftime-sharing) (today) meaning that the OS decides when a process is preempted ("goes to sleep") and which process goes "alive" next.

## Threads

Threads are also OS managed. Threads share the same address space of their parent process. 

Much like processes, threads are about preemptive multitasking and the OS decides when they are preempted.

Generally threads are considered more lightweight than processes and spawning them requires less effort since they share the same address space and context switching is easy as the address space remains the same.

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


Sources:

- http://en.wikipedia.org/wiki/Process_(computing)
- http://en.wikipedia.org/wiki/Inter-process_communication
- http://en.wikipedia.org/wiki/Thread_(computer_science)
- http://en.wikipedia.org/wiki/Computer_multitasking