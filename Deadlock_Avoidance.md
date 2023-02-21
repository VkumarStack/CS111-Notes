# Deadlock Avoidance
- In many cases, it may be difficult to *prevent* deadlock, especially in cases where deadlock results from a critical resource being exhausted (i.e. a process will free up a resource when it completes, but it needs more resources to actually complete)
- Rather, techniques for deadlock **avoidance** are commonly implemented for such cases
    - One common action involves asking processes to *reserve* their resources before actually needing them, ensuring that the system does not enter a state of exhausted resource that may result in the aforementioned type of deadlock
        - i.e. The system could refuse an `sbrk` call if the amount of memory asked for would be too much
        - i.e. The system could refuse to create new files if disk space gets low
        - i.e. The system could refuse to create new processes if it is already thrashing due to not having enough physical memory