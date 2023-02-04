# Memory Management: Swapping and Paging
- Virtualizing memory via segmentation so that physical addresses are not referenced by programs (but instead virtual memory addresses than be translated to physical addresses) allows for relocation to be implemented for feasibly
    - This, however, does not solve the issues of memory segments needing to be contiguous and processes using more memory than physical memory available
## Swapping
- If a set of processes require more RAM than available on the system, it may be necessary to keep some of the memory elsewhere, such as on *disk*
- Programs cannot directly use data on disk - it must be in RAM - so there must be a mechanism to enable *swapping to and from disk*
- One approach is, when a process is yield or blocked (context switch), its memory can be copied to disk and then copied back when it is scheduled again
    - This allows for each currently running process to have an effective address space as large as the total amount of RAM
    - This approach, however, results in the cost of context switches being *very* high, since every single bit of RAM would need to be copied to disk and then the replacement memory must be copied from disk
        - Additionally, this does not deal with the issue that may emerge if a process needs more memory than is physically available
## Paging
- Paging eliminates the requirement of address spaces needing to be contiguous, which in turn allows for swapping to be more flexible 
- Physical memory is divided into units of a *single, fixed* size, typically one that is small (1-4k bytes) known as a **page frame**
- The same division is done for the virtual address space of each process, which now has units known as **pages**
- Each virtual address space page will store its data in one physical address page frame 
    - Both pages and page frames are the same size, so any page can be stored in any page frame
- When addressing pages, then, a per-page translation mechanism converts a virtual page to a physical page
- ![Paging Example](../Images/Paging_Example.png)
    - Notice that, with the paging approach, a process's address space is not necessarily contiguous (even the code, data, and stack segments are not contiguous)
### Paging and Fragmentation
- With the paging approach, internal fragmentation with data segments will only result in the *last* page being fragmented, and this fragmentation averages to only half a page
    - ![Paging Fragmentation](../Images/Paging_Fragmentation.png)
- Paging *does not* result in external fragmentation because there are fixed-size page units
- Overall, paging reduces fragmentation heavily
### Page Translation
- A page translation mechanism must change a virtual address to a physical address for *every* memory reference
- Since this must be done quickly, hardware support is necessary
    - This support is via the memory management unit (MMU), which is designed to perform translations quickly
    - The MMU is typically integrated into the CPU (though this was not the case before)
- Each process requires a page table containing translations for its virtual pages 
    - This page table is typically stored in memory (usually that reserved for the operating system) because the amount of page translations are too large to fit into hardware registers
    - A page table typically contains a *valid bit* indicating whether a page frame is valid (i.e. it actually belongs to the process)
- Virtual addresses are divided into two segments - the virtual page number (higher order bits) and the address offset (lower order bits)
    - The virtual page number is translated into a physical page number (using the MMU) and then the offset is added to this physical page frame to complete the translation
        - The offset is the same for the virtual and physical page, so it is not necessary to translate it
### Memory Management Unit
- Keeping a page table in memory requires performing two memory cycles for each address access (one to look up the page table entry, and another to get the actual data)
- To account for this, a very fast set of MMU registers are used as a cache, known as the **translation lookaside buffer**
    - This cache does not contain pages of memory but rather *translations* - entries from page tables (indicating if the page is valid, and if it is, its address)
- Dealing with caching, however, requires worrying about hit ratios, cache invalidation, and other problems
### Multiple Processes
- When there are several processes running, each requires its own set of pages 
- If there are more virtual pages than physical, extra pages must be stored somewhere else
    - These extra pages are stored on disk (flash drive)
- If the current process adds or removes pages, the active page table in memory must be updated and stale cache entries must be flushed
- Processes require their own page tables, so if a process is switched, a privileged instruction must load a pointer to the new page table when dealing with translations
    - A reload instruction may also be necessary to flush previous cache entries
- Pages can be shared between processes by having each process's page table point to the same physical page (which can be kept as read-only)
### Demand Paging
- Processes don't need all of its pages in memory to run - it only needs those that is actually going to use
- **On demand** paging, then, swaps pages from disk when they are actually needed versus when they are not
    - This means that the entire process does not need to be in memory to start running - only a necessary subset of pages can be loaded and can added on over time when demanded
- For this to work, though, the MMU must be able to determine if a page is on disk or not - if it is on disk, a trap can be generated so the operating system can bring it in to RAM
- Demand paging can perform porly if most memory references require disk access
    - To deal with this, *spacial locality* is exploited - the next address is likely to be one that was close to the last address demanded
    - Most programs tend to exhibit spatial locality 
        - Code segments are usually executed consecutive (or branches nearby)
        - Variable accesses are usually done in the current or previous stack frame
        - Heap references are usually to recently allocated structures
- **Page faults** occur when a page is not in RAM but rather in disk (according to the page table it is "not present"), resulting in a trap that the operating system handles
    - The kernel's page fault handler determines which page is required, and where on disk it resides
        - It then schedules I/O to fetch it, blocking the process while it is occurring
        - Once the I/O is complete, the page table is updated to point to the newly read-in page and the initial instruction is retried (back up the program counter to retry)
- Page faults do not affect a program's correctness - only its performance since processes are blocked
    - Page faults can be reduced by having enough of the requested pages in memory, which is in turn a result of the page eviction policy chosen