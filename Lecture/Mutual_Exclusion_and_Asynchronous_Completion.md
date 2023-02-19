# Mutual Exclusion and Asynchronous Completion
## Mutual Exclusion
- **Mutual exclusion** of a critical section ensures that, if one thread is running a critical section, other threads are *not*
    - This is important for multithreaded applications which frequently share data structures
    - This can also occur with processes with share operating system resources - i.e. files
- Critical sections can be recognized as code sections involving updates to *object state* - whether it be a single object or multiple objects
    - Critical sections tend to involve multi-step operations that lead to an inconsistent state until *all such steps are completed* (so preempting into another thread to run into that section when all steps are not completed is bad)
- Mutual exclusion allows for *atomicity* of a critical section
    - **Before or after atomicity** ensures that thread A enters the critical section before B starts and B enters the critical section after A completes - no *overlap*
    - **All or none atomicity** requires that an update that starts will *complete fully* or will be *undone* (if it fails) 
### Locking
- Critical sections are protected using a data structure known as a **lock**
    - The party holding a lock can access the critical section while parties not holding the lock cannot access the critical section until the lock is no longer held
    - The party holding the lock *must* release the lock once it is done with the critical section, as otherwise the other parties waiting on the lock would never be able to access it
- Since the operating of locking and unlocking itself is a critical section, actually implementing locks requires *hardware assistance*, which can be done specialized atomic instructions
    - Example: Test and Set
        -     bool TS(char *p) {
                bool rc; 
                rc = *p; 
                *p = TRUE;
                return rc;
              } // all of this is done in a SINGLE instruction
            - If the lock was FALSE when this instruction is run, then the instruction would return FALSE (to indicate that the lock was not held) while simultaneously setting the lock to TRUE (so other threads attempting to get it at the same time do not acquire it)
    - Example: Compare and Swap
        -     bool compare_and_swap( int *p, int old, int new) {
                if (*p == old) {
                    *p = new;
                    return (TRUE);
                } else
                    return (FALSE);
              }
              if (compare_and_swap(flag, UNUSED, IN_USE))
                // critical section acquired
              else
                // critical section not acquired
    - Example Lock Code:
        -     bool getlock(lock *lockp) {
                if (TS(lockp) == 0)
                    return (TRUE);
                else
                    return (FALSE);
              }
              void freelock( lock *lockp) {
                *lockp = 0;
              }
- Locks are enforced by ensuring that it is not possible to use the locked resource without actually holding the lock
- If a thread does not get the lock, it can keep trying to get it again - this is **spin waiting** or **spin locking**
    - Spin locks are able to properly enforce access to critical sections and are fairly simple to implement
    - However, spin locks are *wasteful* since spinning uses up processor cycles
        - There is even wastage with the thread holding the lock, as it could be preempted for another thread that is spin waiting, and since the waiting thread is doing nothing but spinning, this delays the thread holding the lock from actually finishing what it is doing and releasing the lock
## The Asynchronous Completion Problem
- Often, it is necessary for activities to *wait* for other activities (i.e. I/O activities, network request, etc.) to complete
- This could be implemented via spinning *if the wait is relatively short* - that is, it is guaranteed to happen soon
    - This way, the spinning does not waste too many CPU cycles or delay the awaited operation by a significant amount
    - Thus, a **yield and spin** approach could be utilized: the event completion is checked for and, if it is not done, the thread could yield - it would then check again when it becomes rescheduled
        - This adds extra context switches and still wastes some cycles if the thread spins each time it is scheduled
        - If there are multiple threads waiting, this also introduces issues with fairness
- This could also be implemented via **completion events**, where a thread that cannot get a lock blocks itself with the expectation of the operating system waking it when the lock is available
    - Completion events are implemented using **condition variables**, which is a synchronization object associated with a resource or request
        - A thread will *wait* on the condition variable, and when the associated event is completed, the waiting thread will *wake*
        - Condition variables are generally provided by the operating system (or library code)
            - A blocked process waiting on a condition variable is moved out of the ready queue and is put back in the ready queue when the desired event occurs (and when it is unblocked)
        - If multiple threads are waiting on a single event (condition variable), an implementation needs some associated *waiting list* to determine which thread to wake up
            - Threads could be woken up one-at-a-time using `pthread_cond_wait` or all-at-once using `pthread_cond_broadcast`
            - Waiting lists may deal with the issue of the **sleep/wakeup race condition**
                - If thread B has locked a resource and thread A needs to get that lock, thread A will call `sleep()` to wait for the lock to be free
                - Thread B will call `wakeup()` to release the lock
                - The issue occurs when B tries to *wake up* A before A has even tried to *sleep* - thus, when A does try to sleep, it will sleep forever
                - This issue can be solved by *locking* the waiting list