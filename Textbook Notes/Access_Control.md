# Access Control
- Determining whether a request to a resource matches security policy is known as **access control**
    - A **subject** is the entity (user, process, etc.) that wants to perform the access on a given resource (file, device, etc.), which is commonly referred to as an **object**; the particular mode (read, write, etc.) in which the subject deals with the object is known as the **access**
- Access control decisions are usually implemented by **reference monitors**
    - How frequently access control is checked is ultimately a tradeoff consideration, balancing security (frequently checking access control) and overhead (computing cost of checking for proper access)
- A simple, low cost way of access control is to leverage virtualization, as there is no need to enforce access control for the resources that a process already owns (i.e. virtual memory)
    - This, however, still requires that the physical resources (which lie under the illusion of virtualization) are still protected
    - Moreso, not all resources can be virtualized - consider files, for example
## Using Access Control Lists for Access Control Lists
- One approach is to utilize **access control lists**, which are lists specifying which entities can access a particular resource
    - This is typically done on a per-file basis, so each file (for example) has its own access control list that is consulted whenever it is opened on a particular mode (read, write, execute)
- Access control lists are implemented via *permission bits*, which specify read, write, and execute permissions for the owner of the file, members of a particular group, and all other users
    - Permission bits are stored in a file's inode, so they are readily accessible whenever the file is opened
- Access control lists in general are useful for determining all users who are allowed to access a particular resource, as that resource's access control list merely needs to be consulted
    - Changing the set of users that can access a resource is also easy to do for similar reasons
- This approach still requires searching through access control information (such as lists of groups) and is also inefficient in determining all the resources a specified user can access
    - Additionally, this approach may suffer from namespacing problems, which could arise in distributed environments where one user may have the same name or ID as another user on a different device (they could potentially access a resource that they were not meant to access)
## Using Capabilities for Access Control
- Another approach is to utilize **capabilities**, which can be thought of as "keys" to a resource that specify access permissions
- In a capability-based system, a process is given the capability to access a certain resource (i.e. read/write a file) by the operating system (via system call)
    - This capability is then checked whenever an operation is performed on the given resource
    - It is important that capabilities are *managed by the operating system* as otherwise they would not be secure (if a process had complete control over a capability, it could just copy the data associated with the capability and access the given resource forever)
        - This is not necessary, though, if capabilities are cryptographically protected
- Capabilities are typically stored in some protected data structure, such as a process control block (stored *in kernel memory*)
    - Active capabilities may be kept on a persistent storage device and then brought into memory when needed
- Capabilities make it easy to determine which resources a given entity can access and likewise makes it easy to revoke access to a certain resource from a user
- However, capabilities may lack in determining all entities that can access a particular resource (as this would require checking every single user's capability list)
- Linux utilizes access control lists whenever the `open` system call is invoked, though subsequent reads and writes rely on capabilities that indicate what permissions are associated with the file (or device), allowing for more efficient access control since the operating system can readily consult the process control block on each file access (instead of searching through the access control list)
    - Whenever the `close` system call is invoked, the capability is removed 
## Role-Based Access Control
- It is common in many organizations for different roles to require different types of access permissions (think manager vs developer)
- In these cases, **role-based access control** is relevant; typically, they involve assigning roles only the necessary privileges
    - Users assigned multiple roles can *switch* from their role to another when encountering an access operation that requires the different role
        - This is ultimately more secure as it prevents potential breaches associated with a given role from occurring