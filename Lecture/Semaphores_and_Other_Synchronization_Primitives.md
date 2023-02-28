# Semaphores and Other Synchronization Primitives
## Semaphores
- **Semaphores** are a theoretically sound way to implement locks
    - They have been thoroughly studies at the theoretical level, but there are of course gaps between thereotical and practical implementations 
- Semaphores are more powerful than simple locks in that they incorporate a FIFO waiting queue and make use of a counter rather than a binary flag 
    - `P() / Wait()` decrements the counter and, if the counter is greater than or equal to zero the method returns; otherwise, if the counter is less than zero (negative), the process is added to the waiting queue (and it sleeps)
    - `V() / Post()` increments the counter and, if the queue is non-empty, wake up one of the waiting processes
### Semaphores for Exclusion
- The semaphore count can be initialized to *one* - the count reflects the number of threads allowed to hold the lock (and in the case of a mutual exclusion lock, this should be one)
    - The `Wait()` operation takes the lock and the `Post()` operation releases the lock
        - The first wait will succeed and subsequent waits will block
        - Releasing the lock will increment the semaphore count to indicate one less waiting request; the first waiting thread will be unblocked
### Semaphores for Notifications
- The semaphore count can be initialized to *zero* - the count reflects the number of completed events
    - The `Wait()` operation can be used to await an event completion and the `Post()` operation can be used to signal completion
        - If the event is already completed, the wait will return immediately - otherwise, all callers will block until a post occurs
        - The post operation will increment the count and wake up the *first in line* - there are no broadcasts
### Counting Semaphores
- The semaphore count is initialized to the number of available resources
    - The `Wait()` operation is used to consume a resource and the `Post()` operation is used to produce a resource
        - The wait will return immediately if the resource is available or block otherwise
        - The post will unblock the first thread waiting on the resource
### Limitation to Semaphores
- Semaphores are a very basic mechanism - they are simple and have few features
    - Semaphores lack many practical synchronization features, as one cannot *check* the lock without blocking (no testing the lock), they do not support reader/writer shared access, and so forth
## Mutexes
- **Mutexes** are a Linux/Unix locking mechanism intended to lock sections of code (expected to be locked *briefly*) with fairly low overhead
### File Descriptor Locking
- Mutexes typically protect *code* critical sections for brief durations - locking *objects* (i.e. files) are fairly more difficult
    - Critical sections associated with objects are likely to last much longer and may involve many different programs trying to access the object (as opposed to threads which share the same address space of the process)
- `int flock(fd, operation)` locks a *file descriptor*
    - This system calls supports *shared locks* (multiple allowed) and *exclusive locks* (one at a time)
    - The lock applies to open instances of the *same* file descriptor
    - This is an advisory lock
- `int flock(fd, cmd, offset, len)`
    - This system call supports getting/waiting for an exclusive lock, releasing a lock, testing a lock
    - The lock applies to the file itself and not the instance of the file 
    - This is typically an enforced lock, though it depends on the underlying file system
### Advisory vs. Enforced Locking
- **Enforced locking** guarantees that the locking occurs and is done within the implementation of the locked object's methods
- **Advisory locking** is a *convention* expected to be followed - users are expected to lock before calling methods
    - Mutexes and `flocks` are advisory locks
## Performance of Locking
- Locking is often performed by a system call, which incurs overhead - especially if called frequently
    - Even library-based locks still involve extra instructions to lock and unlock, though not as bad as system calls
- Locking should be called when a critical section needs to be protected to ensure correctness
    - Critical sections are often brief, so the overhead of the locking operation may even be much higher than the time spent in the critical section itself
- If the lock is not acquired, then the thread must block - which is expensive due to the overhead associated with the yield and context switch as well as waiting through the scheduling queue to get to the thread that actually does hold the lock 
- If all processes need a single resource, then a *convoy* may occur, where processes are in a *line* for a resource, waiting in a sequential (instead of parallel) order 
    - This resource becomes a *bottleneck*
- Locking can be optimized at the *code-level* by:
    - Eliminating the critical section entirely
        - Threads could be given their own copy of the data
        - Only atomic instructions could be used to modify the critical section, though this is often not feasible
    - Eliminating preemption in the critical section
        - This could involve disabling interrupts, which is not always an option
    - Reducing time in the critical section
        - i.e. The required memory could be allocated before taking the lock
        - i.e. An I/O instruction could be done before or after releasing the lock
        - The code inside the critical section could be minimized - that is, lock only code that is subjective to destructive races 
    - Reduce frequency of entering the critical section
        - i.e. Approximate counters
    - Remove requirement for full exclusivity
        - i.e. Read/Write locks
            - Allow many readers to share a resource and enforce exclusivity only when a writer is active
    - Spread requests over more resources
        - Change lock granularity - use a fine grained lock where there is one lock per object 
            - This spreads activity over many locks, which reduces overall contention
            - However, this adds more overhead associated with lock creations and locking/unlocking operations
## Priority Inversion
- **Priority inversion** can occur in priority scheduling systems that use locks
    - If a low priority process P1 that holds a mutex is preempted and a high priority process P2 blocks for that mutex, then P2 is effectively reduced to the priority of P1
- A lower priority task effectively runs because of a lock held elsewhere, preventing a higher priority task from running
- To deal with priority inversion, one solution is to *temporarily increase the priority of the task holding the lock* and then *drop the priority back to normal when the lock is released* - this is **priority inheritance**