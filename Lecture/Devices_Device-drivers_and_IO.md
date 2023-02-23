# Devices, Device Drivers, and I/O
# Devices
- Computers typically have many **peripheral devices** attached to them, and each of these devices need to have some code to ensure that it can perform whatever operation it does and to integrate it with the rest of the system
    - Much of the codebases of modern operating systems comprise of device driver code
## Devices and Performance
- Compared to CPU and RAM, most devices are *very* slow, which may often lead to challenge in managing these devices since systems often want to operate at the *CPU speed* instead of the *device speed* but still want to maintain interaction between the CPU and such devices 
## Peripheral Device Code and the Operating System
- Peripheral devices are managed at an operating system level rather than a user level because many such peripheral devices are critical for system correctness - i.e. the disk
- Moreso, these peripheral devices may be shared among multiple processes (i.e. the monitor), this requiring operating system management to handle
- It is also more secure to deal with these devices at an operating system level
# Device Drivers
- **Device drivers** refer to the code for a *specific device*, ensuring that the device performs the operations it is designed for
    - These drivers are highly specific to a particular device - so a system only needs drivers for the devices it hosts 
- Device drivers are typically *modular* and interact with the rest of the system in limited, *well-defined* ways
- These drivers *must* be correct since they run in privileged mode - if they are not correct, the device will not be able to work correctly (and if this device is critical to the rest of the system, then the whole system may not work correctly)
- These drivers are generally written by programmers who understand the device well (i.e. programmers from companies who worked on the hardware) instead of operating system developers 
## Abstractions and Device Drivers
- To account for the sheer amount of variation within devices, the operating system defines idealized **device classes** - i.e. classes for flash drives, display, printers, networks, serial ports, etc.
    - These classes define an expected interface that all drivers in a certain class should be able to support - i.e. flash drives should be able to read a block of data and write a block of data
- Thus, device drivers that follow these classes are able to implement standard behavior that enables diverse devices to fit into a common operating system
    - This encapsulates the complexity associated with the specifics of each device from user applications
- Driver abstractions are able to encapsulate:
    - Knowledge of how to use the device 
        - Standard operations on the device are mapped to standard operations able to be used by users
        - Device states are mapped into standard object behavior able to be accessed by users
        - Irrelevant behavior is hidden from users
        - Device and application behavior are coordinated without the user doing too much work
    - Knowledge of optimization
        - Optimizations are done within the driver implementation itself, minimizing the amount of optimization that the user must do
    - Fault handling
        - Recoverable faults are handled by the driver itself rather than the user 
## Fitting Device Drivers 
- Device drivers are typically independent pieces of code and may need to be added on to the operating system later, so they often follow a *pluggable model*
    - The operating system provides capabilities to plug in a particular driver in well-defined ways, as well as to unplug them when they are not longer needed
## Layering Device Drivers
- At the lower level, devices are attracted to a bus, which effectively just moves data between different components
- At the higher level, devices are treated as *files*, referred to using some sort of descriptor
- In between these levels are device-specific drivers
## Device Drivers vs. Core OS Code
- Common functionality belongs in the core operating system code, often dealing with frequent issues such as caching (regardless of device), device-specific file system code, higher level network protocols, etc.
- Specialized functionality belongs in drivers - that is, functionality that may differ from hardware to hardware or may pertain only to a specific type of hardware
## Devices and Interrupts
- Since devices are often slower than CPU/RAM, they are primarily interrupt-driven
    - These devices do their *own work* while the CPU does something else, and when they are done, they raise an interrupt to signal the CPU that completion has occurred 
- Devices are not directly connected to the CPU, but are rather connected to a bus, which it uses to communicate with the CPU (which is also on a bus)
    - Modern devices have multiple busses (i.e. PCI bus vs. USB bus)
    - The bus is used to send and receive interrupts and to transfer data and commands 
        - When a device is finished doing its work, it signals a controller, which puts an interrupt on the bus that then transfers the interrupt to the CPU - after this, the actual data transfer associated with the work done by the device may occur
- Interrupts are similar to traps in terms of handling, but are different in that they are caused externally to the CPU (traps are internal)
    - Interrupts can be enabled/disabled by special, privileged CPU instructions 
        - In this case, interrupts associated with devices may be held as *pending* until interrupts are turned back on
## Device Performance
- Certain, key system devices, limit overall system performance - i.e. disk devices are particularly important since they deal with the file system as well as page swapping
- If such devices sit idle, its throughput drops, which in turn results in a lower overall system throughput
    - Thus, it is important to keep devices busy - a device should start the next request immediately after the current one finishes rather than wait for a CPU response for each request
- Since devices operate independently of the CPU, they both can operate in parallel
    - However, both the device and the CPU may need to access RAM, which could be a bottleneck
        - Modern CPU's typically avoid going to RAM by working entirely within its registers or within the CPU cache - if it does need to use RAM, it will not use that much
        - Thus, **Direct Memory Access** can be used allow any two devices attached to the memory bus to move data *directly*, without having to go through the CPU first, enabling devices to move data directly to RAM 
            - If Direct Memory Access is occurring, though, it is not serving CPU requests - but this fine since the CPU often does not directly need the RAM in the first place
- Devices can be constantly kept busy by letting multiple requests be pending at a time (in a queue) and using DMA to perform the actual data transfers
    - When a request completes, then, a completion interrupt can be generated and the operating system can call an appropriate handler to start the *next* data transfer 
- It is better to perform *large* transfers with devices rather than multiple *smaller transfers* since transfers themselves have overhead - related to Direct Memory Access, the device, and the operating system itself having to set up the operations, start the new operation, and so forth
    - Larger transfers, therefore, have less overhead per byte transferred
## I/O and Buffering
- The data associated with I/O requests are placed into some in-memory location, known as a **buffer**
    - Buffers can be used to send data *to* a device or can be used to receive data *from* a device
- Although larger data transfers are more efficient with devices, they are typically not convenient for applications
    - Thus, the operating system often consolidates I/O requests to still try to transfer data in large quantities
        - i.e. For writes, the operating system accumulates a cache of recently used disk blocks and accumulates writes to those blocks, and when there are enough writes, it is actually written out to disk as one large I/O operation
        - i.e. For reads, the operating system reads entire blocks and stores them in cache, even if the requested amount is less, and delivers data from such blocks if more reads are performed 
- Operating systems may also perform read-aheads - that is, they read blocks before they are even requested
## Scatter/Gather I/O
- Direct Memory Access requires an entire transfer to be in *contiguous* physical memory, which may not work for virtual memory
- User buffers are paged in virtual memory, so these buffers are spread all over physical memory
- This requires a *scatter* to read from device to multiple page frames and a *gather* to write from multiple page frames to a device
    - One approach to this is to copy all user data into a physically contiguous buffer
    - Another approach is to split the logical request into chain-scheduled page requests
    - Yet another approach is to have the I/O MMU deal with the scattering and gathering
## Memory Mapped I/O
- Since DMA is not the best way to do I/O, especially for small transfers, an alternative is to use **memory mapped I/O**, where registers or memory in a *device* is treated as part of the regular memory space for a *program*, but under the hood it is translated by the MMU and routed appropriately between the device and the program
    - This directly moves data between the device and the CPU, and as such should be used for small, frequently transfers 