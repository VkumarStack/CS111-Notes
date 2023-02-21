# Common Concurrency Problems
## Non-Deadlock Bugs
- **Atomicity violation** bugs result from an *assumption* that a code section is atomic when in actuality it is not
    - These issues are easily fixed by adding locks around the code section in question
- **Order violation** bugs result from the desired order of two actions not being enforced, resulting in (often rare) cases where the actions execute in the wrong sequence
    - These issues are also easily fixed by using condition variables to enforce proper ordering
## Deadlock Bugs
- **Deadlock** arises from complex locking protocols, often characterized by a thread T1 holding a lock waiting for another thread T2; T2 is holding the lock that T1 is waiting on but itself is waiting on lock held by T1, resulting in neither thread being able to run
    - Deadlock arises from circular dependencies, or **cycles**, among resources
- There are four conditions necessary for deadlock to occur, and eliminating any of the four conditions prevents deadlock
    - **Mutual exclusion** - threads exclusively control the resource (locks) that they require
        - This can be implemented by designing data structures to be **wait-free**, in that they are designed to be atomic
    - **Hold-and-wait** - threads hold resources allocated to them (locks) while waiting for additional resources (other locks)
        - This can be implemented by having a thread acquire all locks at once, atomically, possibly through a *global lock*
            -     lock(prevention);
                  lock(L1);
                  lock(L2);
                  ...
                  unlock(prevention);
    - **No preemption** - resources (locks) cannot be forcibly removed from threads holding them
        - This can be implemented by utilizing operations to *test* a lock instead of attempting to acquire the lock - if the lock is found to be taken, then the current lock being held should be released and tried again later
            -     top:
                    lock(L1);
                    if (trylock(L2) == -1) {
                        unlock(L1);
                        goto top;
                    }
            - This approach, however, could result in **livelock**, where is a *chance* that both threads could repeatedly attempt to acquire the lock but fail again and again - though they are still actually running (unlike deadlock)
                - One potential solution to this is to add a random delay before each loop to decrease the odds of livelock
    - **Circular wait** - there exists a circular chain of threads such that each thread holds one or more resources (locks) that are being requested by the next thread in the chain
        - This can be feasibly implemented by introducing **total ordering** on lock acquisition - it should be ensured that, if multiple locks must be acquired, they must always be acquired in the same order for any thread (i.e. always acquiring L1 and *then* L2)
- Deadlock **avoidance** can be achieved through smart scheduling, as if it is known which threads will try to grab which locks, scheduling can be done to ensure that threads that do not attempt to grab the same locks do not run together
    - This is limited, though, as it requires full knowledge of the tasks being done
    - It can also limit concurrency by preventing some threads from being scheduled
- Another strategy is to employ a deadlock detection system and restart the system whenever an instance of deadlock is detected