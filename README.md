# Concurrency, multi-threading and parallel programming concepts

This document describes various concurrency, multi-threading, parallel programming methods and concepts. The purpose is to write a single document/wiki that contains everything you need to know about concurrency. Things should be explained clearly yet in as great detail as possible. **You are welcome to contribute to this document.**

*If you are interested in contributing to this document, you can ask me to add you as a contributor or you can just fork this.*

*Note 1: This document mainly discusses modern OS and hardware.*

*Note 2: While attention is paid to what is written here, there is no guarantee that this document is correct at its entirety.*

# Table of Contents
#### Approaches to concurrency
* [Processes](#Processes)
* [Threads](#Threads)
* [Green Threads](#GreenThreads)
* [Green Processes](#GreenProcesses)
* [Isolates](#Isolates)

#### Synchronization primitives
* [Locks](#Locks)
* [Mutex](#Mutex)
* [Semaphores](#Semaphores)
* [Monitors](#Monitors)
* [Message Passing](#MessagePassing)
 
#### Higher-Level Frameworks and Languages
* [Microsoft's TPL and other asynchronous features] (#TPL)
* [Apple's GCD and blocks] (#GCD)
* [Clojure] (#Clojure)
* [Erlang] (#Erlang)
* [Node.js] (#Node.js)
* [Python gevent] (#gevent)
* [HTML5 Web Workers] (#WebWorkers)


<a name="Processes"></a>
## Processes

Processes are [operating system](http://en.wikipedia.org/wiki/Operating_system) (OS) managed and in case of a modern OS they are truly concurrent in the presence of suitable hardware support ([multiprocessor](http://en.wikipedia.org/wiki/Multiprocessor) and [multicore](http://en.wikipedia.org/wiki/Multi-core_processor) systems).

Processes are scheduled by the operating system's [scheduler](http://en.wikipedia.org/wiki/Scheduling_(computing\)). They may be made up of multiple [threads of execution](http://en.wikipedia.org/wiki/Thread_(computer_science\)) that execute instructions concurrently.

Processes exist within their own [address space](http://en.wikipedia.org/wiki/Address_space). No other process can read or write to another one's memory, because the OS secures this with [process isolation](http://en.wikipedia.org/wiki/Process_isolation) as the OS is the one who manages the processes.

Processes are heavy and spawning many of them (in contrast to other concurrency models) is not recommended. Creating a process requires creating an entirely new [virtual address space](http://en.wikipedia.org/wiki/Virtual_address_space). 

[Context switching](http://en.wikipedia.org/wiki/Context_switch) between processes requires a lot of work. This means saving of the entire CPU state (all [processor registers](http://en.wikipedia.org/wiki/Processor_register) that were in use, [program counter](http://en.wikipedia.org/wiki/Program_counter), etc.) into [PCB](http://en.wikipedia.org/wiki/Process_control_block) (usually). Context switching also involves switching the [MMU](http://en.wikipedia.org/wiki/Memory_management_unit) context and keeping OS related data.

Processes can't talk to each other directly (in modern OS). Instead the OS provides facilities for [inter-process communication](http://en.wikipedia.org/wiki/Inter-process_communication).

Typical ways of inter-process communication involve [files](http://en.wikipedia.org/wiki/Computer_file), [signals](http://en.wikipedia.org/wiki/Signal_(computing\)), [sockets](http://en.wikipedia.org/wiki/Berkeley_sockets), [message queues](http://en.wikipedia.org/wiki/Message_queue), ([named](http://en.wikipedia.org/wiki/Named_pipe)) [pipes](http://en.wikipedia.org/wiki/Pipeline_(Unix\)), [semaphores](http://en.wikipedia.org/wiki/Semaphore_(programming\)), [shared memory](http://en.wikipedia.org/wiki/Shared_memory) and even [memory-mapped files](http://en.wikipedia.org/wiki/Memory-mapped_file).

Processes are all about [preemptive multitasking](http://en.wikipedia.org/wiki/Computer_multitasking#Preemptive_multitasking.2Ftime-sharing) (today) meaning that the OS decides when a process is preempted ("goes to sleep") and which process goes "alive" next.

<a name="Threads"></a>
## Threads

Threads, like processes, are also OS managed. Threads share the same address space of their [parent process](http://en.wikipedia.org/wiki/Parent_process). This means that processes can spawn threads indirectly using OS provided functionality (e.g. [CreateThread](http://msdn.microsoft.com/en-us/library/ms885186.aspx) or [pthread_create](http://www.kernel.org/doc/man-pages/online/pages/man3/pthread_create.3.html)).

On a single processor, multithreading generally occurs by [time-division multiplexing](http://en.wikipedia.org/wiki/Time-division_multiplexing) (as in [multitasking](http://en.wikipedia.org/wiki/Computer_multitasking)): the processor switches between different threads. This [context switching](http://en.wikipedia.org/wiki/Context_switch) generally happens frequently enough that the user perceives the threads or tasks as running at the same time. On a [multiprocessor](http://en.wikipedia.org/wiki/Multiprocessor) (including [multi-core](http://en.wikipedia.org/wiki/Multi-core) system), the threads or tasks will actually run at the same time, with each processor or core running a particular thread or task.

Many modern operating systems directly support both [time-sliced](http://en.wikipedia.org/wiki/Preemption_(computing\)#Time_slice) and multiprocessor threading with a [process scheduler](http://en.wikipedia.org/wiki/Scheduling_(computing\)).

Like processes, threads are about [preemptive multitasking](http://en.wikipedia.org/wiki/Computer_multitasking#Preemptive_multitasking.2Ftime-sharing) and the OS decides when they are preempted. To avoid preemption with threads, [mutex](http://en.wikipedia.org/wiki/Mutual_exclusion) can be used. Windows also offers support for [Critical Sections](http://en.wikipedia.org/wiki/Critical_section) with two functions [EnterCriticalSection](http://msdn.microsoft.com/en-us/library/ms885212.aspx) and [LeaveCriticalSection](http://msdn.microsoft.com/en-us/library/bb202768.aspx).

Communication between threads is far simpler than [inter-process communication](http://en.wikipedia.org/wiki/Inter-process_communication) between processes. This is mainly due to the shared memory, but also due to the fact that the [strict security constraints](http://en.wikipedia.org/wiki/Process_isolation) the OS puts on processes do not exist with threads.

#### Memory usage

Threads are generally said to be "lightweight", but that is relative. Threads have to support the execution of [native code](http://en.wikipedia.org/wiki/Machine_code) so the OS has to provide a decent-sized [stack](http://en.wikipedia.org/wiki/Stack-based_memory_allocation), usually measured in megabytes. In Windows, the [default stack reservation size](http://msdn.microsoft.com/en-us/library/windows/desktop/ms686774(v=vs.85\).aspx) used by the linker is 1 MB. In Linux, the typical [thread stack size](http://www.kernel.org/doc/man-pages/online/pages/man3/pthread_create.3.html) is between 2 MB and 10 MB. This means that in Linux, creating 1000 threads would equal to memory usage from ~2 GB to ~10 GB, without even beginning to do any actual work with the threads.

##### Determining stack size on Linux
You can rather easily determine the default stack size for Linux OS by running the following:
```
$ ulimit -a | grep stack
stack size    (kbytes, -s) 8192
```
The above output was from Ubuntu 11 x86. We can also test this with some code:

```c
void makeTenThreads()
{
    std::vector<pthread_t> threads;
 
    for (int i = 0; i < 10; i++)
    {
        threads.push_back(pthread_t(0));
        pthread_create(&threads.back(), 0, &doNothing2, 0);
    }
 
    std::vector<pthread_t>::iterator itr = threads.begin();
    while (itr != threads.end())
    {
        pthread_join(*itr, 0);
        ++itr;
    }
 
    sleep(11);
 
    threads.clear();
}
 
int main()
{
    makeTenThreads();
    sleep(10);
}
```
Running `pmap -x 1234` where `1234` is the PID will give us 10 x 8192K blocks allocated, because we created 10 threads and each of them got 8 MB allocated.

##### Setting the stack size

Thread default stack size varies depending on the OS and you can actually set it on your own. On Linux, you call [pthread_attr_setstacksize](http://www.kernel.org/doc/man-pages/online/pages/man3/pthread_attr_setstacksize.3.html) and on Windows it can be specified as a parameter to [CreateThread](http://msdn.microsoft.com/en-us/library/ms885186.aspx).

The number of threads a process can create is limited by the available virtual memory and depends on the default stack size. On Windows, if every thread has 1 MB of stack space, you can create a maximum of 32 threads. If you reduce the default stack size, you can create more threads. These details vary greatly depending on the platform and libraries.

Reducing the thread stack size will not reduce overhead in terms of CPU or performance. Your only limit in this respect is the total available virtual address space given to threads on your platform. Generally you should not change the stack size, because you can't really compute how much you need for an arbitrary thread, as it totally depends on the code run. You would have to analyze the entire code and the resulting disassembly executed by the thread to know how much stack size to use. This is non-trivial.

#### "Threads are lightweight"
Threads are "lightweight processes", not "lightweight" themselves as some may claim. They require less resources to create and to do context switching as opposed to processes, but it still is not cheap to have many of them running at the same time.

While thread context switching still involves restoring of [the program counter](http://en.wikipedia.org/wiki/Program_counter), CPU [registers](http://en.wikipedia.org/wiki/Processor_register), and other potential OS data, they do not need the context switch of an [MMU](http://en.wikipedia.org/wiki/Memory_management_unit) unlike processes do, because threads share the same memory. Context switching with threads is less of a problem unless you have many threads.

#### Race conditions

Since memory/data is shared among threads in the same process, applications frequently need to deal with [race conditions](http://en.wikipedia.org/wiki/Race_conditions). [Thread Synchronization](http://en.wikipedia.org/wiki/Synchronization_(computer_science\)#Thread_or_process_synchronization) is needed. Typical synchronization mechanisms include [Locks](Locks), [Mutex](Mutex), [Monitors](Monitors) and [Semaphores](Semaphores). These are concurrency constructs used to ensure two threads won't access the same shared data at the same time, thus achieving correctness.

Programming with threads involve hazardous race conditions, deadlocks and livelocks. This is often said to be one of the bad things about threads along with the overhead they bring.

#### Summary
Threads are good for concurrent CPU heavy work across 1-n processors and cores within a single application/process. This is, because they scale to cores and CPUs thanks to the OS scheduler. For IO heavy work, threads are a nightmare, because that usually involves spawning of multiple threads for shorter periods of time.

##### When to use

* You need plenty of CPU.
* You keep the threads running for a longer time.
* You do not spawn and kill and spawn and kill threads too often.
* You do not need many threads.

An example of a good use for a thread could be a game. Running e.g. AI logic on a separate thread makes sense, because the thread is spawned once and kept alive for a long time. AI logic also requires plenty of CPU making threads very good for such purposes.

Short-lived, frequently spawned threads make little sense. Building a chat web application that involves 1000 concurrent active chatters are an example when not to use threads. Memory usage would be high, context switching would take too much time relative to the actual application. Creating threads and killing them that often has an unacceptable high overhead. A chat requires more IO than CPU work, thus, threads do not suit that situation.

<a name="GreenThreads"></a>
## Green Threads

Green threads are [threads](Threads) that are scheduled by a virtual machine (VM). In contrast, typical threads are scheduled by the underlying operating system. Green threads emulate multithreaded environments without relying on any native OS capabilities, and they are managed in user space instead of kernel space, enabling them to work in environments that do not have native thread support.

Native threads can switch between threads pre-emptively, switching control from a running thread to a non-running thread at any time. Green threads only switch when control is explicitly given up by a thread (Thread.yield(), Object.wait(), etc.) or a thread performs a blocking operation (read(), etc.). Whether a blocking call causes yielding depends on the virtual machine implementation.

On multi-CPU machines, native threads can run more than one thread simultaneously by assigning different threads to different CPUs. Green threads run on only one CPU, because every green thread runs on the same operating system thread. This means that a green thread that blocks will block every other green thread as well.

Green threads can be started much faster on some VMs. They significantly outperform Linux native threads on thread activation and synchronization. Native threads have better performance on I/O and context switching operations, however.

Green threads are a good choice over native threads if the platform does not provide support for threading. Another reason is that if your code is not thread-safe or if you need to spawn many threads and often since this is cheaper with green threads.

The Erlang virtual machine has what might be called ['green processes'](Green Processes) - they are like operating system processes (they do not share state like threads do) but are implemented within the Erlang Run Time System (erts). These are sometimes erroneously cited as green threads.

#### Summary
Green threads are rarely used nowadays. They provide little use and have their drawbacks.

##### When to use
* The underlying platform does not provide support for threads.
* We need to spawn many threads or often.
* Our code is not thread-safe.

<a name="GreenProcesses"></a>
## Green Processes

Green processes are like operating system processes except that they are implemented in a virtual machine on the user space. This means that green processes do not share state like threads do, freeing us from the race condition hazards that threads pose.

The Erlang Run Time System (erts) implements green processes. These are sometimes erroneously cited as green threads.

Green processes are neither operating system processes nor threads, but lightweight processes. It has been estimated that [Erlang's green processes take around 300 WORDs](http://www.erlang.org/doc/efficiency_guide/processes.html) to create, thus many of them can be created without degrading the performance. It has also been proven to be possible to create even [20 million processes](http://groups.google.com/group/comp.lang.functional/msg/33b7a62afb727a4f?dmode=source). These figures are highly implementation dependent, but they show the potential of green processes.

One reason why green processes can be so lightweight as opposed to operating system processes is that green processes totally lack of any type of security restrictions or other unnecessary overhead that the OS has to put on its processes. Another reason for their lightness is that the virtual machine is fully aware of the application that uses these processes, unlike in case of an OS that has less of an idea of what an application might do.

Inter-process communication works via shared-nothing asynchronous message passing style. The use of message passing eliminates hazards such as deadlocks and livelocks that threads have and allow for clean communication.

<a name="Isolates"></a>
## Isolates

Isolates are a concurrency mechanism used in [Google Dart](http://dartlang.org). Isolates are spawned and use [message passing](Message Passing). They are inspired by Erlang's [green processes](Green Processes).

There are two types of isolates, heavy and light. These two differ dramatically, and the programmer has to know which one to use in which scenario. Luckily isolates offer an intuitive API that works the same way for both light and heavy isolates so it is easy to change the type of an isolate at any time.

Isolates are independent of each other. This means they do not share state as one might guess from the use of [message passing](Message Passing).

#### Heavy isolates
Heavy isolates use [threads](Threads) behind the scenes. Typical synchronization mechanisms such as [locks](Locks) and [monitors](Monitors) are not needed nor do they exist in the programmers eyes. Heavy isolates use [message passing](Message Passing) and internally rely on operating system threads. It's up to the implementation to decide what kind of synchronization mechanism to use (e.g. [monitors](Monitors)) and that mechanism may even vary depending on scenarios.

Due to use of [message passing](Message Passing) you are free from [thread related problems](Threads) such as deadlocks and spinlocks. Heavy isolates are essentially a nice implementation on top of native [threads](Threads).

#### Light isolates
Light isolates are different from heavy isolates in the sense that the underlying implementation does not use [threads](Threads). Light isolates are single-threaded, they live on the thread that spawned the light isolate. For example, one may spawn two heavy isolates which both spawn a light isolate and in this case, you would have two native [threads](Threads) both running one light isolate.

Light isolates are very efficient in context switching and to create/kill. And in fact, it is very well possible to create millions of light isolates. This allows for concurrent, real-time, web applications for instance.

Beware that blocking in a light isolate blocks the rest of the isolates on that thread.

##### Heavy vs light isolates
Heavy isolates come with some of the benefits and drawbacks of threads. The same applies to light isolates, they come with the problems and benefits of single-threaded evented systems.

##### When to use heavy isolates
* You need CPU heavy work.
* You do not need multiple isolates.
* You are not spawning and killing isolates too often.

##### When to use light isolates
* You are IO bound rather than CPU bound.
* You need several isolates or you spawn/kill them frequently.

If you were to program a game, you should use heavy isolates for the AI. If you were programming a web server, you would use light isolates for handling requests.

There is no reason not to use both and in fact, using both at times makes sense.

#### Communication
As said earlier, isolates rely on [message passing](Message Passing). There is no shared data and communication is done through ports. This communication allows you to also restart parts of the program if something goes wrong and an isolate lives as long as its port(s) are open. The [message passing](Message Passing) is asynchronous.

Here's an example code sample written in Dart:
```java
class Worker extends Isolate {
    Worker() : super.heavy();
 
    main() {
        this.port.receive(
            void _(var message, SendPort replyTo) {
                print("Message received: ${message}");
                replyTo.send("sup");
                this.port.close();
            }
        );
    }
}
```
The Worker class extends the Isolate class, and when it's instantiated, it calls the Isolate class' heavy constructor which means that this Worker class uses a heavy isolate.

We are supplying an anonymous callback function to the `receive`-method. The function receives `message` and `replyTo` arguments. Since the `message` is specified as `var`, the sender can send anything from integers to strings to complex objects. The same API works with light isolates as well.

#### Other benefits of isolates
Isolates have their own heap. This means that garbage collection can happen per isolate. This solves the big problem of today's garbage collectors which need to pause the entire program to clean the garbage. With isolates, only one isolate needs to be paused while being cleaned allowing the rest of the program to continue. This has huge benefits for UI-centric software where a garbage collector pause would freeze the UI.

It is also a possibility to send messages between isolates on different hosts (called remote isolates), although Dart does not implement this yet.

Isolates may also be used to enforce security rules on ports and to even replace the [Same Origin Policy](http://en.wikipedia.org/wiki/Same_origin_policy).

<a name="Locks"></a>
## Locks

[Locks](http://en.wikipedia.org/wiki/Lock_(computer_science\)) are usually advisory locks, where each thread cooperates by acquiring the lock before accessing the corresponding data. Most locking designs block the execution of the thread requesting the lock until it is allowed to access the locked resource. A [spinlock](http://en.wikipedia.org/wiki/Spinlock) is a lock where the thread simply waits ("spins") until the lock becomes available. It is very efficient if threads are only likely to be blocked for a short period of time, as it avoids the overhead of operating system process re-scheduling. It is wasteful if the lock is held for a long period of time. Careless use of locks can result in [deadlock](http://en.wikipedia.org/wiki/Deadlock) or [livelock](http://en.wikipedia.org/wiki/Livelock). In many programming languages Locks are the easiest synchronization mechanism to use.

<a name="Mutex"></a>
## Mutex

A [mutex](http://en.wikipedia.org/wiki/Mutex) is similar to a monitor; it prevents the simultaneous execution of a block of code by more than one thread at a time. In fact, the name "mutex" is a shortened form of the term "mutually exclusive." Unlike monitors, however, a mutex can be used to synchronize threads across processes. When used for inter-process synchronization, a mutex is called a named mutex because it is to be used in another application, and therefore it cannot be shared by means of a global or static variable. It must be given a name so that both applications can access the same mutex object. Mutexes unlike Locks or Monitors, can be used for inter-process synchronization.

<a name="Semaphores"></a>
## Semaphores

[Semaphores](http://en.wikipedia.org/wiki/Semaphore_(programming\)) is a variable or an abstract data type that provides a simple, but useful abstraction for controlling access by multiple processes to a common resource in a parallel programming environment. Thus, Semaphores are not used for **intra-process** synchronization, but only **inter-process** synchronization. A Semaphore is essentially a record of how many units of a particular resource are available, coupled with operations to safely (i.e. without [race conditions](http://en.wikipedia.org/wiki/Race_conditions)) adjust that record as units are required or become free, and if necessary wait until a unit of the resource becomes available.

Semaphores which allow an arbitrary resource count are called **counting semaphores**, while semaphores which are restricted to the values 0 and 1 (or locked/unlocked, unavailable/available) are called **binary semaphores**. A mutex is essentially the same thing as a **binary** semaphore, and sometimes uses the same basic implementation. However, the term "mutex" is used to describe a construct which prevents two processes from executing the same piece of code, or accessing the same data, at the same time. The term "binary semaphore" is used to describe a construct which limits access to a single resource on the system.

<a name="Monitors"></a>
## Monitors

[Monitors](http://en.wikipedia.org/wiki/Monitor_(synchronization\)) prevent blocks of code from simultaneous execution by multiple threads just like Locks. In fact, Locks are sometimes based on Monitors, like in case of C#. Monitors provide a mechanism for threads to temporarily give up exclusive access, in order to wait for some condition to be met, before regaining exclusive access and resuming their task.

<a name="MessagePassing"></a>
## Message Passing

Message passing is a form of communication used in parallel computing, object-oriented programming, and interprocess communication. In this model, processes or objects can send and receive messages -- consisting of zero or more bytes, complex data structures, or even segments of code -- to other processes. By waiting for messages, processes can also synchronize.

Message passing has several forms and may be implemented differently:

* Communication can be either synchronous or asynchronous.
* Messages may be passed one-to-one (unicast), one-to-many (multicast or broadcast), many-to-one (client-server) or even many-to-many (AllToAll).
* Messages may or may not be received in the original order they were sent.
* Messages may or may not be sent reliably.

Implementations of concurrent systems that use message passing can either have message passing as an integral part of the language, or as a series of library calls from the language. Message passing systems have been called "shared nothing" systems because the message passing abstraction hides underlying state changes that may be used in the implementation of sending messages.

Messages are also commonly used in the same sense as a means of inter-process communication; the other common technique being streams or pipes, in which data are sent as a sequence of elementary data items instead.

#### Synchronous vs asynchronous message passing
Synchronous message passing systems require the sender and receiver to wait for each other to transfer the message. That is, the sender will not continue until the receiver has received the message.

Synchronous communication has two advantages. The first advantage is that reasoning about the program can be simplified in that there is a synchronisation point between sender and receiver on message transfer. The second advantage is that no buffering is required. The message can always be stored on the receiving side, because the sender will not continue until the receiver is ready.

Asynchronous message passing systems deliver a message from sender to receiver, without waiting for the receiver to be ready. The advantage of asynchronous communication is that the sender and receiver can overlap their computation because they do not wait for each other.

Synchronous communication can be built on top of asynchronous communication by ensuring that the sender always wait for an acknowledgement message from the receiver before continuing.
The buffer required in asynchronous communication can cause problems when it is full. A decision has to be made whether to block the sender or whether to discard future messages. If the sender is blocked, it may lead to an unexpected deadlock. If messages are dropped, then communication is no longer reliable.

<a name="TPL"></a>
##Microsoft's TPL and other asynchronous framework features
The Task Parallel Library (TPL) is a set of public types and APIs in the System.Threading and System.Threading.Tasks namespaces in the .NET Framework 4. The purpose of the TPL is to make developers more productive by simplifying the process of adding parallelism and concurrency to applications. The TPL scales the degree of concurrency dynamically to most efficiently use all the processors that are available. In addition, the TPL handles the partitioning of the work, the scheduling of threads on the ThreadPool, cancellation support, state management, and other low-level details. By using TPL, you can maximize the performance of your code while focusing on the work that your program is designed to accomplish. 

Starting with the .NET Framework 4, the TPL is the preferred way to write multithreaded and parallel code. However, not all code is suitable for parallelization; for example, if a loop performs only a small amount of work on each iteration, or it doesn't run for many iterations, then the overhead of parallelization can cause the code to run more slowly. Furthermore, parallelization like any multithreaded code adds complexity to your program execution. Although the TPL simplifies multithreaded scenarios, we recommend that you have a basic understanding of threading concepts, for example, locks, deadlocks, and race conditions, so that you can use the TPL effectively.

[TPL](http://msdn.microsoft.com/en-us/library/dd460717.aspx)

[Microsoft's TPL and Traditional .NET Framework Asynchronous Programming](http://msdn.microsoft.com/en-us/library/dd997423.aspx)

<a name="GCD"></a>
##Apple's GCD and blocks

Grand Central Dispatch (GCD) is an approach to multicore computing that is woven throughout the fabric of OS X version 10.6 Snow Leopard. GCD combines an easy-to-use programming model with highly-efficient system services to simplify the code needed to make best use of multiple processors and improve performance.

The central insight of GCD is shifting the responsibility for managing threads and their execution from applications to the operating system. As a result, programmers can write less code to deal with concurrent operations in their applications, and the system can perform more efficiently.

GCD is implemented as a set of extensions to the C language as well as a new API and runtime engine. While initially inspired by the challenge of multicore computing, these actually solve a more general problem: how to efficiently schedule multiple independent chunks of work. GCD does this using four primary abstractions:

* block objects

* dispatch queues

* synchronization

* event sources

[Apple's GCD and blocks](http://developer.apple.com/library/mac/#featuredarticles/BlocksGCD/_index.html)

<a name="Clojure"></a>

##Clojure

Clojure is a dynamic programming language that targets the Java Virtual Machine (and the CLR, and JavaScript). It is designed to be a general-purpose language, combining the approachability and interactive development of a scripting language with an efficient and robust infrastructure for multithreaded programming. Clojure is a compiled language - it compiles directly to JVM bytecode, yet remains completely dynamic. Every feature supported by Clojure is supported at runtime. Clojure provides easy access to the Java frameworks, with optional type hints and type inference, to ensure that calls to Java can avoid reflection.

Clojure is a dialect of Lisp, and shares with Lisp the code-as-data philosophy and a macro system. Clojure is predominantly a functional programming language, and features a rich set of immutable, persistent data structures. When mutable state is needed, Clojure offers a software transactional memory system and reactive Agent system that ensure clean, correct, multithreaded designs.

[Clojure](http://clojure.org/)


<a name="Erlang"></a>

##Erlang

Erlang is a programming language used to build massively scalable soft real-time systems with requirements on high availability. Some of its uses are in telecoms, banking, e-commerce, computer telephony and instant messaging. Erlang's runtime system has built-in support for concurrency, distribution and fault tolerance.

[Erlang](http://www.erlang.org/)

<a name="Node.js"></a>
##Node.js

Node.js is a platform built on Chrome's JavaScript runtime for easily building fast, scalable network applications. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient, perfect for data-intensive real-time applications that run across distributed devices.

[Node.js](http://nodejs.org)

<a name="gevent"></a>
##Python's gevent

gevent is a coroutine-based Python networking library that uses greenlet to provide a high-level synchronous API on top of the libevent event loop.

Features include:

* Fast event loop based on libevent (epoll on Linux, kqueue on FreeBSD).
* Lightweight execution units based on greenlet.
* API that re-uses concepts from the Python standard library (for example there are Events and Queues).
* Cooperative sockets with SSL support »
* DNS queries performed through libevent-dns.
* Monkey patching utility to get 3rd party modules to become cooperative »
* Fast WSGI server based on libevent-http »

[Python's gevent](http://www.gevent.org/)

<a name="WebWorkers"></a>
##HTML5 Web Workers

Web Workers provide a simple means for web content to run scripts in background threads.  Once created, a worker can send messages to the spawning task by posting messages to an event handler specified by the creator.

The worker thread can perform tasks without interfering with the user interface.  In addition, they can perform I/O using XMLHttpRequest.

The Worker interface spawns real OS-level threads, and concurrency can cause interesting effects in your code if you aren't careful. However, in the case of web workers, the carefully controlled communication points with other threads means that it's actually very hard to cause concurrency problems.  There's no access to non-thread safe components or the DOM and you have to pass specific data in and out of a thread through serialized objects.  So you have to work really hard to cause problems in your code.

Web Workers are part of the HTML5 specification, and exposed by a HTML5 compliant engine. Thus, Web Workers are accessible in client-side JavaScript and other client-side compatible languages such as [Google Dart](http://api.dartlang.org/docs/continuous/dart_html/Worker.html).

[HTML5 Web Workers](https://developer.mozilla.org/en-US/docs/DOM/Using_web_workers)
