# Operating System Services
## Abstractions
- Directly using hardware functionality is very difficult, as they are often detail-oriented (i.e. directly setting a pixel on the screen)
- The operating system offers abstract versions of these resources, allowing for easier use of hardware functionality since the operating system deals with the difficult, often platform-specific details while the user deals with the higher-level functionality
    - The operating system must actually implement the abstraction of these physical resources (i.e. the processes abstraction requires hiding away the CPU and RAM, the file abstraction requires hiding away the disk); physical resource -> abstract resource
- In addition to being easier to use than the original resources, abstractions allow for complexity to be encapsulated since there is no longer a need to worry about lower-level details (i.e. no need to understand cache behavior when dealing with memory, etc.). Additionally, behavior irrelevant to the user is eliminated and instead focus on more convenient behavior 
- There are many variations in hardware and software, requiring their abstractions to be generalized, often through common cases (i.e. all printers print a page of paper) in a unifying model (i.e. all printers understand pfs)
## Resources
- **Serially reusable resources** are resources that are used by multiple clients, but only *one at a time* (i.e. a core on a CPU - multiple programs cannot simultaneously run on a *single* core)
    - These resources require access control to ensure exclusive use as well as *graceful transitions* from one user to the next
        - These transitions should only occur after the first user is finished with the resource (there are no incomplete operations) and should ensure that the subsequent user does not see any traces of data or state from the previous user ("like new")
    - i.e. Context switching for CPU cores
- **Partionable resources** are resources divided into disjoint pieces for multiple clients
    - These resources require access control to ensure containment - an application cannot access resource outside of its partition - and privacy - other applications cannot access an applications resources
    - These resources may still need graceful transitions, as these resources aren't permanently allocated (i.e. memory) - once the resource is freed, a transition is necessary
    - i.e. Memory, hard disk
- **Shareable resources** are resources usable simultaneously by clients - they don't have to wait for the resource or have it allocated to them
    - Shareable resources tend to involve effectively limitless resources, such as the operating system, shared by processes (i.e. when making system calls)
    - Shareable resources don't require a graceful transition since shareable resources tend to not change state (read-only)
    - It is a good thing to build systems to maximize shareable resources
## General Operating Systems Trends
- Over time, operating systems have grown larger and more sophisticated, with a fundamentally changed role
    - They have changed from shepherding the use of hardware to shielding applications from the hardware, providing power application computing platforms, and mediating application "traffic"
- These changes are a result of the evolving complexity of applications, requiring the operating system to provide more complex core services to such applications
- As a result of the growing scale of operating systems, there are only a handful of widely used operating systems - Windows, MacOS, Linux
    - Operating systems in the same family are used for vastly different purposes, which presents difficulties for the designer because they must accomodate not only for different machines, but also for different purposes
    - These operating systems are based on older design philosophies
    - New operating systems have not emerged because they're expensive to build and maintain and can only succeed if they have some sort of clear advantage over contemporary operating systems
    - *Windows* is popular for personal computers
    - *MacOS* is exclusively in Apple products
    - *Linux* is the choice for industrial servers and embedded systems (it is highly configurable and free, so it is easy to fit into embedded devices)
# Operating System Services
- Basic Abstractions:
    - CPU (processes, threads, virtual machines) and memory abstractions (virtual address spaces)
    - Persistent storage abstractions (files and file systems)
    - I/O abstractions (virtual terminal sessions, windows, sockets, pipes, etc.)
- Higher Level Abstractions:
    - Cooperating parallel processes (requires locks, condition variables, distributed transactions, leases, etc.)
    - Security (user authentication, secure sessions, at-rest encryption)
    - User interface (GUI)
- Background Services (Invisible):
    - Enclosure management (power, fan, fault handling)
    - Software updates
    - Dynamic resource allocation and scheduling (CPU, memory, bus resources, disk, network)
    - Networks, protocols, and domain services (USB, BlueTooth, TCP/IP, etc.)