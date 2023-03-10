# Deadlock: Problems and Solutions
- **Deadlock** is a situation in which two entities have each locked some resource, but each needs the other's locked resource to continue
    - Neither entity will unlock unless they acquire both locks, hence no progress will be made by either entity
- Deadlock is a relevant issue for cooperating parallel processes, and thus are fairly common in complex applications, possibly resulting in catastrophic system failures (since deadlock *halts* progress)
    - Deadlock is difficult to find through debugging since whether it occurs is non-deterministic
    - The best way to combat deadlock is to *prevent* it from even occurring at design time
- The possibility of deadlock occurring may not be obvious since the resources that processes need are ever-changing, dependent on a variety of factors
    - Modern software depends on many services, each of which encapsulate complexity - meaning that what resources they require (and therefore lock) are unknown
## Deadlock and Resource Types
- Commodity resources, such as memory, can result in deadlock from over-commitment 
    - This can be avoided not handing out *too many resources*, which is usually handled by a resource manager
- General resources, such as a particular file or semaphore, commonly result in deadlock due to *dependency relationships*
    - This can be prevented at *design time*
## Conditions for Deadlock
- Mutual exclusion
    - The resources in question can each only be used by one entity at a time 
    - There would be no deadlock if the resources could be *shared*
- Incremental allocation
    - Entities are allowed to *ask* for resources whenever they want
    - There would be no deadlock if entities must pre-allocate all resources since they either get all the resources they need and run to completion or do not get all the resources they need and abort
- No preemption
    - If an entity has reserved a resource, it *cannot* be taken away from it - not even temporarily
    - There would be no deadlock if a resource could simply be taken away from an entity and given to another one 
- Circular waiting
    - If there is a cycle in a graph of resource requests
    - There would be no deadlock if this cycle dependency is eliminated
## Deadlock Avoidance
- Resource-based deadlock can be avoided through **advanced reservations** 
    - A resource manager could  *avoid* overbooking a resource (i.e. memory) as otherwise deadlock could occur (there is no more RAM, but a process needs more RAM before being able to free its already allocated RAM)
    - However, a resource manager *could* overbook (in order to avoid under-utilization of resources) but ensure that any deadlock conditions are avoided by *safely* overbooking
        - This involves resource management mechanisms that determines which process to give a resource to once another process is done using it - in other words, the system determines if processes will *complete* so that their resources can be given to other processes that had reserved that resource  
            - If a reservation *cannot* be made, the resource manager should deny the resource request, as making the request would otherwise result in potential deadlock
            - This requires applications to deal with reservation failures themselves - ideally, they should continue to run (maybe in a reduced mode) instead of crashing
                - It is better to deny a reservation ahead of time instead of letting the application fail later on when it is found that the resource cannot actually be given
- General-resource-based deadlock can be avoided by ensuring that a particular lock does not cause deadlock, such as by eliminating any of the four aforementioned deadlock conditions 
    - Mutual exclusion:
        - Shareable resources can be maintained - either through atomic instructions or throguh reader/writer locking
        - Processes can be given private resources instead of shareable resources so that there is no need for mutual exclusion in the first place
    - Incremental allocation:
        - All resources can be allocated in a single operation
        - A lock can be *tested* instead of being acquired - if the lock is found to be taken, then the current locks can be released to potentially allow for other processes waiting on that lock to acquire it
        - Blocking can be disallowed while resources are being held - a process must release all held locks prior to blocking, reacquiring them when they return 
            - Although this may cause other issues, this *does* prevent deadlock since locks are acquired in an all-or-none fashion
    - No preemption:
        - Resource confiscation can be allowed - resource "leases" have an associated time before a time-out occurs and the lock is taken away (and reallocated to a new client)
            - This reallocation must be enforced, usually by the operating system's resource manager
                - It is usually difficult for the operating system to revoke a process' access to a resource if the process has *direct access* to that object (it is part of the object's address space) - as otherwise the process would need to be killed
            - Some resources may be damaged by lock breaking, so there should also be mechanisms to audit or repair a resource  
    - Circular waiting:
        - Total resource ordering can be used, when applicable, where all requesters allocate resources *in the same order* (i.e. always R1 first and then R2)
            - This requires a convention for ordering resources, such as by type or by relationship
        - This may result in *lock dances*, where a later-ordered lock is released in order to acquire a different lock of lower order (as otherwise this would break the total ordering)
            - i.e. If the enforced order is R1 and then R2 and R2 is held, in order to acquire R1, R2 must *first be released* and then R1 must be attempted to be held
        - This approach is usually best applicable to a system where all locks are known (i.e. own code)
            - This does not work well in a general system where the locks are not all known 
- In some cases, deadlocks can be allowed to happen and recovered via deadlock detection systems
    - These systems require identifying all resources that can be locked
        - Once all resources have been identified, a dependency graph can be created and checked for a cycle
            - This graph can be maintained on each resource request or built whenever a deadlock is suspected to have occurred 
    - This approach makes more sense at the application level (i.e. database systems), since applications can keep track of their locks, compared to the operating system level (since the operating system cannot keep track of application locks)
- Rather than a pure deadlock detection system, complex applications tend to use a health monitoring approach to determine if a service is hung
    - Health monitoring systems are able to deal with a wide range of problems beyond just deadlocks
    - Health can be monitored through:
        - Checks for obvious failures (process exits/core dumps), usually by asking the operating system
        - Observation to detect hangs (check if it is using CPU time/is doing network requests/is doing disk requests)
        - External health monitoring (heartbeat pings or standard test requests)
        - Internal instrumentation (a process determines itself if it is healthy)
    - Unhealthy processes can *killed and restarted* to fix the issues at hand
        - Only as many processes are necessary should be killed, and not ones that are not broken
        - The intrusion that killing a process may cause to other clients or processes should be minimized 
            - Modern systems tend to define restart groups, where processes are started and killed as a group, with inter-group dependencies in mind 
    - Typically, the failure recovery methodology involves first trying to *retry the request*, and if this doesn't work, *roll-back the failed operations* and try to continue with reduced capacity or functionality
        - If there are still issues, then *restarts* should be used
        - If there are *still* issues, then more mechanisms should be used
        - The principle is to start from low-cost recovery and escalate to high-cost if necessary
## Making Synchronization Easier
- One approach to making synchronization easier for programmers is to have some sort of compiler mechanism that automatically generates locks and releases for identified shared resources - the programmer writes the code serially
    - Some object-oriented languages implement this through **monitors**, where an object is designated as a monitor that is automatically given a semaphore
        - This semaphore is automatically acquired on any method invocation and automatically released on method return
        - This enables good encapsulation, since developers do not need to explicitly identify critical sections, while ensuring adequate protection
        - However, monitors are very *conservative*, as the entire object is locked on *any* method and for the entire duration, resulting in possible performance issues since parallelism is reduced
    - Another approach is through **synchronized methods** (as in Java), where each object has an associated mutex and it is specified which methods acquire this mutex - not all objects need to be specified
        - This mutex is automatically released upon returns from the method
        - This allows for a finer lock granularity and a reduced deadlock risk, but now it requires which methods requiring a mutex to be identified explicitly 