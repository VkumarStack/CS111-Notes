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