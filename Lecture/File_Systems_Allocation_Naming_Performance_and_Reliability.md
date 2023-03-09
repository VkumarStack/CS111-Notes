# File Systems - Allocation, Naming, Performance, and Reliability
## Free Space and Allocation Issues
- File systems are not usually static, as files are frequently created, updated, and destroyed
    - This means that unused blocks may be converted to used blocks or vice-versa
- To deal with this, file systems typically need to maintain some sort of free list structure of unused disk blocks 
### Creating a New File
- Creating a file requires allocating a free file control block
    - In a UNIX system, this requires searching the superblock bitmap for a free inode and taking the first free inode found
    - In the Microsoft DOS system, this requires searching the parent directory for an unused directory entry 
- After the file control block is allocated, it is initialized and the file itself is given a name
### Extending a File
- If an application requests new data to be assigned to a file, then the file system must find a free block of data to extend the file
    - This may require traversing the free list to find an appropriate chunk and remove it from the list
    - This also requires associating the found chunk with the appropriate address in the file (update the appropriate file or extend descriptor to point to the newly allocated chunk)
### Deleting a File
- Deleting a file requires releasing all the space that is allocated to the file
    - In a UNIX system, this just means returning each block to the block free list (though this may take a while for a large file with many indirect blocks)
    - The DOS file system does not free space - the directory entry associated with that file is marked as "free"
        - Periodically, the file system utilizes a garbage collection mechanism to search out deallocated blocks and then add them to the free list
- Deleting a file also requires deallocating the file control block
    - In a UNIX system, the inode is zeroed out and returned to the free list
    - In the DOS system, the first byte of the name in the parent directory is zeroed out, to indicate that the directory entry is no longer in use
### Free Space Maintenance
- The file system manager manages free space, requiring that acquiring or releasing blocks should be *fast* operations (doing as little I/O as possible) since they are extremely frequent
- Which blocks that are acquired does matter in terms of performance, as contiguous data is often favorable
## Performance Improvement Strategies
- Large transfers of data are more efficient for storage devices, as they ammortize fixed overhead costs 
- Reads are often cached to reduce the amount of needed I/O operations
    - Read blocks are stored in an in-memory cache, which can be consulted before scheduling I/O
    - The file block cache may be structured using the least-recently used (LRU) principle
- It may also be common for **read-aheads** to be utilized, where blocks are requested from the I/O device before any process asks for them, which could potentially reduce process wait times
    - This is useful for when clients sequentially access a file, but there is the risk of device access time being wasted if unwanted blocks are read in
- Device writes also usually go to a **write-back** cache, which are flushed out to the device later
    - This improves performance by aggregating many small writes into a large batch of writes, which could also eliminate any redundant writes (i.e. if an application subsequently rewrites the same data or deletes the file)
- Types of Disk Caching
    - General blocks can be cached, including popular files that are read frequently, files that are written and then promptly re-read, etc.
    - There also may be caches for special purpose blocks, such as directories (which speed up directory accesses) or inodes (which speed up the re-uses of the same file)
        - Special purpose caches are more complex
## Naming in File Systems
- Files can be referred to by the file system via *inodes*, but these low level names are not user friendly
- Thus, the file system must handle name-to-file mapping - associating names with new files (binding), finding the underlying representation (inode) for a given name, changing names associated with existing files, and allowing users to organize files using names
### Name Space Structure
- A flat name space structure, where all file names exist at a single level, yields poor organization
    - This approach implies that there can only be one name per file system
- Graph-based name spaces are more common, with hierachrical trees or acyclic directed graphs 
    - In this structure, directories are utilized for organization, which are files that contain references to other files
        - Directories act as a *naming context*, where each process has a current directory and file names are interpreted relative to that current directory
        - Directories cannot be arbritarily written to - they are written to only by the operating system, which users can access through system calls
    - This approach implies that there can only be one name per directory 
- In DOS systems, a file only has a single "true name"
- In UNIX systems files can have multiple names (hard links)
    - All hard links point to the same inode, which is the true descriptor of a file
    - A file exists as long as at least one hard link (name) to that file exists
        - This requires keeping a reference count of links in the file's *inode*, which is decremented every time a hard link to that file is deleted and incremented every time a hard link to that file is created
## File System Reliability
- There are a multitude of reliability issues associated with file systems, such as system crashes that result in an incorrect file state, data loss, and potentially even file system corruption (key structures being corrupted can be very detrimental)
- The main issue with reliability comes from the fact that file system writes typically involve multiple operations (writing to the data block as well as potentially many metadata blocks)
- One approach can involve blocking writes until all writes have *actually occurred* (instead of telling the user that they have occurred when in reality they are being buffered)
    - This, however, incurs a performance overhead for the *rare* case of a crash
- Another approach can be to utilize *ordered writes*
    - This involves writing the data out first and then pointers (metadata)
        - If there is a crash, the unreferenced data can be cleaned later during a garbage colection process
        - This is meant to minimize incorrect metadata, which is a serious issue
    - This also involves writing out deallocations (fixing pointers) before allocations (correcting the free list)
    - Even so, doing this still reduces I/O performance as now there is an imposed ordering in I/O operations
        - Moreso, ordered writes may not be possible since many modern storage devices may re-order requests
- Yet another approach can be to design file system structures to be auditable and repairable
    - This often involves placing redundant information in multiple, distinct places
    - Periodically, the file system would be audited for correctness and the redundant information could be used to enable automatic repair 
        - This was typically done in Unix using the `fsck` tool
    - This approach is no longer practical (though it used to be) since the audit and repair process takes long
- The most modern approach is to make use of **journaling**, which utilizes a circular buffer journaling device to store all intended file system updates (transactions)
    - The file system then consults the journal entries to undergo all of the updates (as the actual writes are stored in journal entries) and then *when the updates have been completed*, the journal entry is cleared
        - If there is a crash during a write (or even right before a write), then the journal entry would not have been cleared and the updates could be replayed
    - Journal entries can be batched, since as soon as an update is written into the journal it is safe
    - Journaling writes are much faster than data writes since journal writes are sequential
        - Scanning the journal on restart is alaso very fast 
- **Metadata journaling**, unlike normal journaling, only stores metadata 
    - The data is written first and then the metadata is written in the journal, which ensures that there is no metadata pointing to garbage
    - This is more efficient than normal journaling becauase writing data blocks to the journal would otherwise take up much of its bandwidth
## Log Structured File System
- In a log-structured file system, the journal *is* the file system
    - All inodes and data updates are written to the log, meaning that updates are redirect-on-write (point to the new version instead of the old version)
    - Typically, an in-memory index will cache inode locations
- Log-structured file systems may be slow with regards to recovery time (to reconstruct index/cache) and may incur log garbage that must be periodically defragmented and garbage collected
- In a log-structured file system, inodes point at data segments in the log, and since these inodes themselves may be updated there is a need for a inode map that indicates the location of the most recent inode
- When a block is written to, its blocks and inodes are *immutable*