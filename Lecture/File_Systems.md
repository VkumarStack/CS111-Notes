# File Systems
- Systems need to store data persistently, as vital data to the system (including the operating system itself) must able to be accessed even after the system reboots or powers down
- It is possible to use raw storage blocks to store this persistent data, but this is not convenient to work with
- Rather, a **file system**, which is an organized way of structuring persistent data, is utilized
    - This file system should be able to work regardless of the storage medium - whether it be a hard disk drive, a flash drive, a RAID, etc. 
## Basic File System Concept
- Data is organized into coherent, easily-accessible units, known as **files**
    - Each file is stored in a way that enables efficient access, thus requiring a powerful organizing principle
- These file systems are stored on hardware providing persistent memory (i.e. hard disk drives or flash drives) with the expectation that a file placed in one location will be there later on
    - A user-visible file system should be able to work on any reasonable hardware, despite any potential performance distances
## Flash Drives
- Flash drives are solid state persistent storage devices, meaning they have no moving parts
    - Compared to hard disks, flash drives have fairly fast reads and writes
    - However, a given block of a flash drive can only be written once - overwriting it requires erasing the *entire block* and then writing to it
## Data and Metadata
- File systems deal with both **data**, which is the information that the file is actually supposed to store, and **metadata**, which is the information about the information the file stores (i.e. creation time)
## Desirable File System Properties
- Persistence
- Ease of use (for accessing one file as well as organizing a collection of files)
- Flexibility (no limit on number of files or file size/type/contents)
- Portability across hardware device types
- Performance
- Reliability
- Security
## Basic File System Design
- At a higher level, applications believe they are accessing files and at the bottom level, various block devices are reading and writing blocks
    - There are multiple layers of abstraction in between to ensure that the file system is reliable
## File System API
- A single API for file systems is present, regardless of the actual type of file system implemented underneath
    - **File container operations** are the standard file management system calls, manipulating files as objects and ignoring the actual contents of the file
    - **Directory operations** are the system calls pertaining to manipulating directories - i.e. find a file by name (name -> inode), create a new name or file mapping, or list a set of known names
    - **File I/O operations** are the system calls involving the actual reads and writes of the data on files
## Virtual File System (VFS) Layer
- The **Virtual File System Layer** generalizes file systems, permitting the rest of the operating system to treat all file systems as the same
- This acts as a plug-in interface for file system implementations (DOS FAT, Unix, EXT3, etc.)
    - All of these implementations support the same basic file methods 
        - Create, delete, open, close, link, unlink, etc.
        - Get/put block, get/set attributes, read directory, etc.
## File Systems Layer
- All file systems are implemented on top of block I/O, which is independent of the underlying devices
- All file systems perform the same basic functions:
    - Mapping names to files
    - Mapping (file, offset) to (device, block)
    - Manage free space and allocate to files
    - Create and destroy files
    - Get and set file attributes
    - Manipulate the file name space
- There are multiple file systems because some file systems work better on certain storage devices 
    - Different file systems also provide different services - i.e. some are more reliability-oriented (LFS), some are more performance-oriented (FFS), etc.
## File Systems and Block I/O Devices
- This layer implements standard operations on each block device
    - Asynchronous reads (physical block #, buffer, bytecount)
    - Asynchronous write (physical block #, buffer, bytecount)
- This layer maps logical block numbers to device addresses 
    - i.e. Logical block number to (cylinder, head, sector)
## Device Independent Block I/O
- A buffer cache can be utilized for drive data, which can hold frequently used data until it is needed again and hold pre-fetched read-ahead data until it is requested
- It may also provide buffers for data re-bllocking, which adapts a file system block size to device block size and a file system block size to user request sizes
- This layer may also perform buffering when it comes to device writes
- Having this cache eliminates many disk accesses, which are slow by nature
    - It is better to have a single cache rather than multiple since there is a better hit ratio
## File System Control Structures
- It is important for there to be an on-device description of the important attributes of a file, particularly where its data is located
    - This is usually paired with some kind of in-memory representation of the same information
        - When a file is opened, an in-memory structure is created, but it is not an exact copy of the device version 
            - The device version points to device blocks and the in-memory version should point to RAM pages
            - The in-memory version may also indicate which on-device pages have been written to (are dirty)
- A file typically consists of multiple data blocks, and the control structure must be able to find them, and do so quickly
    - Moreso, this structure should be able to account for file blocks being changed or data being added/deleted to a file 
## Basic File System Structure
- File systems live on block-oriented devices, which are divided into fixed-size blocks
    - Most of these blocks will store user data, but some will also be used to store organizing metadata
- The **boot block** is a reserved block on the device (0th block typically) that contains the code allowing the machine to boot the operating system
    - The boot block is not usually under the control of a file system - it is ignored entirely
    - Not all storage devices are bootable, but the 0th block is usually reserved just in case 
- Allocated space is managed in *blocks*, which may not necessarily be contiguous for a given file
    - This allocated space is managed by a file control data structure that may contain pointers to blocks
## DOS File System
- The Microsoft DOS system utilizes **linked extents**, which involves a file control block that contains exactly *one pointer* to the first chunk of the file
    - Each chunk contains a pointer to the next chunk, which allows for an arbitrary amount of chunks to be added to each file
    - These pointers can be in an auxiliary "chunk linkage" table, which can be kept in memory - enabling faster searches
- The file system divides space into "clusters", which are a fixed size
    - The file control structure, as mentioned earlier, points to the first cluster of a file 
- The File Allocation Table (FAT) contains the number of the next cluster in a file, with an entry of 0 indicating that the cluster is not allocated and an entry of -1 indicating that the end of the file has been reached
    - The FAT table can be kept in memory, thus allowing for the nth cluster of a file to be found without reading each cluster before it
- Thus, to find a particular block of a file
    - The number of the first cluster must be found from the directory entry, then a chain of pointers can be followed through the File Allocation Table (which is kept in memory)
        - Although the File Allocation Table is kept in memory, the in-memory search can still be long for very large files
- The DOS file system has no support for sparse files - if a file has a block *n*, it must have all blocks less than *n* 
## File Index Blocks
- **File index blocks** act as a different way to keep track of a where a file's data blocks are on the device  
    - The file control block points to all blocks in the file, enabling a very fast access to any desired block
        - For large files, some of the pointers do not point directly to data blocks but rather another block of pointers which contain the actual pointers to data blocks
            - The extent of this pointer hierarchy can be increased as needed
    - Although going through many indirect blocks may be slow, it is common for a majority of files to be only a *few thousand bytes long* so usually extra I/O is unnecessary for small files (though this is not the same for large files)
    - Assuming 4K bytes per block and 4 bytes per pointer, then:
        - 10 direct blocks = 10 * 4K bytes = 40K bytes
        - Indirect block = 1K * 4K = 4M bytes
        - Double indirect block = 1K * 4M = 4G bytes
        - Triple indirect block = 1K * 4G = 4T bytes