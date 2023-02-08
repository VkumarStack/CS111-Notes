# Threads, IPC, and Synchronization
## Threads
- Processes are very expensive because they require *their own* resources - such as their own address spaces
    - Processes, by nature, are distinct from each other, so they are not necessarily *meant* and usually *cannot* share resources
- Threads are more relevant in programs where this *resource sharing is necessary*, typically in order to work for a single goal
- **Threads** are units of execution and scheduling, each with their own stack, program counter, and registers
    - Multiple threads can run in a process, sharing the same code and data space - since they access the same resources, they are cheaper to create a run
- The CPU is shared between multiple threads, and this can be implemented either via user-level threads (yielding) or with scheduled system threads (preemption)
## Threads vs. Processes
- Processes should be used:
    - When there are multiple, distinct programs needing to be run
    - When there are only a small handful of programs needing to be run (the creation/destruction of processes are minimal)
    - When running agents have distinct privileges
    - When it is necessary to prevent interference between executing interpreters
    - When it is necessary to firewall one process from the failures of the other - one process failing may not necessarily cause another process to fail
- Threads should be used:
    - When there are parallel activities *in a single program* (i.e. a web server)
    - When there is frequent creation and destruction of running agents
    - When all running agents can run with the same privileges 
    - When it is necessary to share resources
    - When it is necessary to exchange many messages/signals between running agents
    - When it is not necessary to protect running agents from each other 
- Tradeoffs:
    - If multiple processes are used, the application may run much more slowly and there may be difficulty in sharing resources
    - If multiple threads are used, the creation and management of them is up to the user (not the operating system) - resulting in more complexity, not only in thread management but also in ensuring that resources can be serialized and threads are protected from each other (if necessary)
## Thread State and Thread Stacks
- Each thread has its own registers, processor status word, program counter, and its own stack area
    - The maximum stack size is specified when a thread is created since if it were otherwise able to grow indefinitely it could collide with other threads' stacks
## User Level Threads vs. Kernel Threads
- Kernel threads are an abstraction provided by the kernel that still involve one address space being shared but the *scheduling of threads being done by the kernel*
    - This can allow for multiple threads to use multiple cores at once
- User level threads are independent from the kernel and are provided and maanged via a user-level library - these threads are scheduled *by the library, not by the kernel*
## Communications Between Processes
- Distinct processes may occasionally need to exchange information - the mechanisms for **inter-process communications** is provided by the operating system, since processes cannot directly interact with each other
    - IPC requires activity from *both* communicating processes (both must agree to communicate), and this communication is mediated at each step by the operating system in order to protect both processes while also ensuring correct behavior
- For *local processes*, the data in the memory space of the sender needs to sent to the memory space of the receiver
    - In one approach, the operating system can copy the data between the two memory spaces
        - This is conceptually simpler, but results in potentially high overhead
    - In another approach, the operating system could use virtual memory techniques to *switch ownership* of the memory from the sender to the receiver 
        - This is much cheaper than copying the data, but requires changing page tables 
        - Additionally, only one of the two processes can see the data at a time (so the sender must no longer need the data after it is gone)
- **Synchronous IPC** involves the writer being *blocked* until the message is sent/delivered/received and the reader being *blocked* until a new message is available
    - This is very easy to implement but is obviously slow due to frequent blocking
- **Asynchronous IPC** involves the write operation returning when the system accepts the message and *not when it is confirmed to be delivered/received* and the read operations returning promptly if there is no message available
    - This requires an auxiliary mechanism to learn of errors in regards to the delivery of writes and an auxiliary mechanism to learn of new messages received in regards to reads
- Typical IPC Operations:
    - Create/Destroy a persistent IPC channel
    - Write/Send/Put data into the channel
    - Read/Receive/Get data from the channel
    - Query content from the channel (how much data is in the channel)
- **Streams** involve a continuous "flow" of bytes - few or many bytes may be read or written at a time
    - The write and read buffer sizes are unrelated, and the stream is delimited via program-specific conventions (like newlines, commas, etc.)
- **Messages (datagrams)** are a sequence of distinct messages, each with its own length (subject to limits)
    - Each message is typically read/written as a unit, and message delivery is usually all-or-nothing