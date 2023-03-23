# Accessing Remote Data
- In implementing distributed file systems, there are goals of:
    - Transparency: the file system should be indistinguishable from a local file system for *all* uses
    - Performance: the file system should be at least as fast as local disk and should be scalable (unaffected by the number of clients)
    - Cost: the price of disk storage should be less than that of local storage, with ideally no administration
    - Capacity: the distributed file system should ideally be unlimited
    - Availability: the system should always be available
## Remote File Access
- The goal in a remote file access system is complete transparency
    - Normal file system calls should be able to work on remote files while maintaining performance, availability, reliability, and scalability
- In this system, the client-side file system is a local proxy that translates files operations into network requests
    - There is also a server-side daemon that receives and processes these requests, actually translating them into real file system operations
- This approach provides good application-level transparency, functional encapsulation, support for multi-client file sharing, and good performance and robustness
    - However, the approach requires part of the implementation to be in the host operating system
- Remote file access is the most common model for client/server storage
## Security for Remote File Systems
- Authentication approaches may involve:
    - Anonymous access: files are available for any user (typically used for read-only files)
    - Peer-to-peer authentication: assumes all participating nodes in the distributed system are trusted peers
        - There is client-side authentication/authorization, and there is an assumption that all users are known to all systems
        - The systems are trusted to enforce basic access control, but this hinges on each remote machine being able to be trusted
    - Server authentication: client agent authenticates to each server and is authenticated for the entire session
        - The authorization for the user is based on the credentials produced by the server
    - Domain authentication: there is independent authentication of the client and of the server
        - The authentication service issues signed "tickets" that assures each of the others' identity and rights - these tickets may be revocable or on a timed lease
        