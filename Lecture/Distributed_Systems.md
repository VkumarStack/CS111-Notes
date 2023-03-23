# Distributed Systems
- Distributed systems are relevant for their scalability and performance capabilities
    - Often, applications may require more resources than a single computer has
- Distributed systems are useful for reliability and availability
    - If one node fails, the other nodes can take its place
- Distributed systems are easier and cheaper to use for certain cases, as services and systems can be centralized in a distributed environment
- Ideally, distributed systems should be **transparent**, meaning that they look as much like single machine systems as possible
    - It is technically impossible for distributed systems to be completely transparent
## Issues with Distributed Computing
- Since different machines do not share memory or peripheral devices, one machine cannot easily know the state of another
- The only way for machines to communicate remotely is to use a network, which are often slow and error prone
- Failures of one machine are not so easily visible to other machines
- Seven (Eight) Fallacies of Network Computing
    - The network is reliable - **it is not**
    - There is no latency (instant response time) - **it is not**
    - The available bandwidth is infinite - **it is not**
    - The network is secure - **it is not**
    - The topology (what nodes are connected to what) of the network does not change - **it does**
    - There is one administrator for the whole network - **there is no central authority**
    - The cost of transporting additional data is zero - **it is not**
    - All locations on the network are equivalent - **they are not**
## Distributed System Paradigms
- Parallel processing
    - Relies on tightly coupled special hardware
    - Is not very popular
- Single system images
    - Makes all nodes look like one computer
    - Somewhere between hard and impossible to implement, and therefore not popular
### Loosely Coupled Systems
- This paradigm involves a parallel group of independent computers connected by a high speed local access network (LAN), serving similar but independent requests
    - Ideally, minimal coordination and cooperation is required between the computers
- This approach allows for good scalability and price performance, availability (if the protocol permits stateless servers), and ease of management/reconfigurability
- Web servers, application servers, cloud computing, and other services often rely on the loosely coupled systems paradigm 
- The horizontal scalability potential of the system may be bottlenecked by the network speed or a load balancer 
- Architecture:
    - A farm of independent servers, which run the same software but serve different requests
        - These servers may share a common backend database
    - A front-end switch (load balancer) that distributes incoming requests among available servers
    - A service protocol that specifies stateless servers and *idempotent* operations
        - If there is failure with a machine, then the operation can be repeated on a *different* machine
### Loosely Coupled Horizontally Scaled Performance
- Individual servers are often very inexpensive (i.e. $100-200 each) and scalability and availability are excellent
- The challenge is with managing thousands of servers, as this requires automated installations, global configuration services, automated monitoring and healing systems, and so forth
### Cloud Computing
- Cloud computing is another variation of distributed computing that sets up a large number of machines all identically configured and connected to a high speed LAN
    - These machines accept arbitrary jobs from remote users, which are run on one or more nodes
- In principle, anything can run in a cloud environment; in reality, however, general distributed computing in the cloud is difficult
    - Much of the work is done using particular kinds of parallel/distributed processing tools which are provided to users
#### MapReduce
- MapReduce is a common cloud computing software tool to deal with a large amount of data
    - It divides a large problem into compartmentalized pieces, each of which can be performed on a separate node and then combined into a set of results
- There is a single function that is to be performed on a large set of data (i.e. searching for a particular string)
    - The data is divided into disjoint pieces, and the function is performed on each piece on a separate node (computer); **map**
    - The results are combined to obtain the ouput; **reduce**
        - Two nodes might be assigned to doing a reduce operation, each receiving a share of data from a map node
        - Each reduce node combines their results, and then another MapReduce may be run to combine each reduce node's individual outputs, this time with one reduce node that combines everything
- Each map node produces an output file for each roduce node, and this is done atomically
    - The reduce node cannot work on the data until the whole file (data from all map nodes) is written, which forces a synchronization point between the map and reduce phases
        - If either a map or reduce node fails, its work can just be assigned to a different node
### Cloud Computing and Horizontal Scaling
- Cloud computing works well for horizontal scaling, as if the load gets heavy, the system can ask the cloud provider for more nodes (though at a price)
## Remote Procedure Calls
- Remote procedure calls (RPC) are one way of building a distributed program
    - They are based on procedure calls, which are a fundamental paradigm in programming, providing an easily usable interface
    - Remote procedure calls act as a natural boundary between the client and server
- There are limitations with remote procedure calls as there cannot be implicit parameters or returns (i.e. global variables) or call by reference parameters/pointers (the server cannot access the client's memory)
    - Remote procedure calls are much slower than traditional procedure calls
- Remote procedure calls specify their interface, indicating the methods available, their parameter types, and their return types
    - To account for calls from different machine architectures (i.e. 32-bit vs. 64-bit), data is passed into remote procedure calls in an eXternal Data Representation (XDR) which are machine *independent*
    - The client stub will take the information used for a remote procedure call and put it into the appropriate format (XDR) and send it to the server
    - The server stub will receive the API invocations and serve them 
- Remote procedure calls typically involve the client application not worrying about the actual specifics of the remote procedure call - the work of formatting the information, communicating with the server, and so forth, is done by RPC tools
- Remote procedure call is not a complete solution, as it still requires a live connection between the client and server - there must be binding at some point
    - The approach also may require a threading model implementation on the server-side, which each thread servicing one request at a time
    - In terms of failure handling, it is often the client that must arrange for timeout and recovery rather than the server
    - RPC has limited consistency support, as procedure calls are only guaranteed consistent between just the calling client and the server - managing consistency is more complex with multiple clients and servers working together 
## Distributed Synchronization
- Synchronization in the context of distributed systems is difficult to achieve because of the inherent spatial and temporal separation between machines
    - Different processes run on different systems, so there is no shared memory for atomic locks or a single operating system controlling all machines
    - There is an inherent difficulty in being able to "totally order" spatially separated events, so ensuring that operations occur "before", "simultaneously", or "after" relative to other actions is impossible
- Distributed systems make use of **leases**, which are more robust locks that are only valid for a limited period of time
    - Revoking an expired lease is fairly easy, as the time period of the lease can be compared to the *server's clock* (the one that issued the lease)
    - Thus, it is generally safe to issue a new lease, but there is a potential issue of a lease expiring while a request is being performed
        - An object that is being updated may be left in an incomplete state, so it is necessary to account for such a situation either by rolling back state prior to the aborted lease or to implement all-or-none transactions
## Security for Distributed Systems
- Security is much more difficult with distributed systems because network activities happen *outside* of the operating system
    - The wire connecting to the user to the distributed system is inherently insecure, as it could be susceptible to eavesdropping, replays, man-in-the-middle attacks, and so forth
- Network security seeks to ensure that there are *secure conversations*
    - There is a need for **privacy** such that only the client and their partner know the messages they send
    - There is a need for **integrity** such that messages cannot be tampered with (unknowingly)
- Network security also seeks the identification of both parties
    - The authentication of the identity of the message sender is necessary
    - Assurance that a message is not a replay or forgery is necessary
    - Assurance that messages cannot be repudiated 
- Network security relies heavily on cryptography
    - Public key cryptography is used primarily for authentication, and after this authentication is performed, symmetric cryptography is used for the bulk of transport of data
    - Cryptographic hashes are used to detect message alterations
- Network security also makes use of filtering technologies such as firewalls to keep harmful connections from reaching host machines
### Tamper Detection: Cryptographic Hashes
- Cryptographic hashes can act as very strong checksums, which are often used to detect data corruption
    - It is incredibly unlikely for two messages to produce the same hash - it is computationally hard to find two messages with the hash
    - Cryptographic hashes are designed in a way such that the original message cannot be inferred from the hash, and any changes to the input will significantly change the output 
- In typical use, a cryptographic hash computed for the message being protected, often using a secure algorithm (such as SHA-3)
    - The hash is transmitted securely (encrypt the hash and the receiver unencrypts it), alongside the data
        - Unless the data needs to be secret, it is generally not encrypted - only the hash is encrypted
    - The recipient does the same computation on the received message and compares it with the hash (that they have unencrypted) to check for tampering
### A Principle of Key Use
- Both symmetric and public key cryptography rely on a secret key for their properties
- The more a key is used, however, the less secure it becomes
    - The key stays around in various places longer, which provides more opportunities for attackers to get ahold of it
    - It may even be the case that brute force attacks could eventually succeed in solving the key
- It is good practice, therefore, to use a given key as little as possible and to change keys often
### Secure Socket Layer (SSL)
- SSL/TLS is a general solution for securing network communication
- It is built upon the existing socket IPC, but establishes a *secure* link between two parties, ensuring that no other party can snoop on conversations and that no party can generate fake messages
- SSL/TLS relies on certificate-based authentication, usually to authenticate the server
    - With this certificate-based authentication, the client is able to obtain the server's public key, and this is used to distribute a symmetric session key that is used for the remainder of the conversation between the two parties