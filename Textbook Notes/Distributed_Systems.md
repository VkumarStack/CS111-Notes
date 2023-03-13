# Distributed Systems
- Distributed systems are designed with **failure** in mind
    - In a well-implemented distributed system, a single machine should be able to fail without the entire system failing
- Distributed systems are also designed to account for **performance** overheads, especially when it considering messages between devices (which are relatively slow since they must be done over network)
- Distributed systems must also deal with **security**, since remote connections are obviously not as secure as usage on a single device
## Communication Basics
- Communication via networks is inherently unreliable, as packets are often regularly lost
    - A common cause of packet loss is due to a lack of buffering within a network device, as too many packets arriving at a router may force the router to **drop** packets (since its memory is too overloaded)
### Unreliable Communication Layers
- Some transportation layers, such as **UDP/IP** allow for communication via an unreliable messaging layer, often with the expectation that the application can deal with the packet loss (rather than the networking protocol)
    - Processes using UDP typically use the **sockets** API in order to create a **communication endpoint**
    - Other processes send UDP **datagrams** to the original process
- UDP does some amount of guarding for packet corruption by including a **checksum**
### Reliable Communication Layers
- Reliable transportation layers, such as **TCP/IP**, utilize **acknowledgement** - when the sender sends a message to the receiver, the receiver sends a short message back to the sender to acknowledge the receipt
    - If the sender receives the acknowledgement, then it knows that the message has been sent properly
    - If the sender does not receive acknowledgement within a set period of time, a **timeout** occurs, and the sender **retries** the send
- It may be the case, though, that the original message does send but the acknowledgement gets lost, which results in the receiver getting the same message *twice*
    - To account for this, receivers must have some mechanism to identify duplicate messages so that, when they are received, an acknowledgement is sent back but the message itself is not passed up to the application (since it is a duplicate)
- Duplicate messages are identified using a **sequence counter** - the sender and receiver agree on a start value and each maintain a counter
    - Whenever a message is sent, the current value of the counter is also sent along with the message, and the sender increments their counter on each send (it does not increment on resends)
    - The receiver uses its counter value as the expected value for the counter sent with the message and increments their counter on each receive (it does not increment on a duplicate, though it does still acknowledge to the sender)
### Remote Procedure Call
- Most distributed systems use **remote procedure calls (RPC)** as abstractions for communication, with the goal of code execution on a remote machine acting akin to calling a local function
- The **stub generator** handles packing the arguments to and unpacking the return values from remote procedure calls
    - The **client stub** for a function handles remote procedure calls by creating a message buffer, packing the needed information into this message buffer (**serialization**), sending the message to the remote procedure call server, waiting for a reply, unpacking the return code (**deserialization**), and finally returning to the caller
        - Special care must be taken towards complex arguments, such as data structures involving pointers (since they point to memory on the client device, which the remote server cannot access)
    - The **server stub** similarly unpacks the received message, calls the actual function, packages the results, and sends the reply back to the caller
        - Servers are typically set up in a concurrent fashion, with a **thread pool** of worker threads to handle remote procedure calls
- The **run-time library** handles much of the work in a remote procedure call system
    - In building the run-time layer, there is the issue of **naming**, though simple approaches build upon existing naming systems
        - The client must know the hostname or IP address of the machine running the service as well as its port number, which act as identifiers for different communication channels on a particular machine
    - The run-time layer is usually built on UDP rather than TCP, since the latter incurs performance overhead due to the time needed for acknowledgement