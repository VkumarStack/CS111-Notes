# Event-Based Concurrency
- Concurrency can be achieved without using threads, as with **event-based concurrency**
- This form of concurrency relies on what is known as an **event loop** in which a program *waits for some event to occur* and *executes the appropriate instructions once it has occurred*
    -     while(1) {
            events = getEvents();
            for (e in events)
                processEvent(e);
          }
- The code that processes each event is known as an **event handler**, and event handlers themselves take place one at a time - event-based concurrency is inherently single threaded and relies on *scheduling* for the ordering of event processing
## Receiving Events
- Events are typically received through system calls, such as `select()` or `poll()`
    - The `select()` system call, as an example, takes in I/O descriptors and marks them as set if they are able to be read or written to - allowing for event processing to occur
## Blocking
- An issue arising with event-based concurrency is the potential of *blocking*
    - For event-based systems, calls that result in blocking - such as read or write operations to disk - would result in the *entire event loop being blocked* until the call in question completes
    - To account for this, event-based systems make use of **asynchronous I/O**, which are I/O requests that, once issued, immediately return to the caller (instead of waiting for completion)]
        - Asynchronous I/O may require another mechanism to **poll** for the completion of the operation
        - An alternative to polling is an **interrupt** based approach, where the operating system **signals** applications whenever an asynchronous I/O operation completes
## State Management
- An issue with event-based concurrency is that of state management, as an asynchronous operation must find some way to keep the program state valid for the next event handler to use
    - i.e. 
        -     int rc = read(fd, buffer, size);
              rc = write(sd, buffer, size);
        - Since the `read` and `write` operations are done asynchronously, the program must determine some way to ensure that the result of the `read` (stored in `buffer`) is properly available to the next `write` call
        - This is typically done through **continuations**, where the necessary information to finish processing an event is stored in some data structure and looked up when needed
            - In the example, the socket descriptor `sd` would be stored in some data structure indexed by the `fd` so that when the `read` completes, the associated file descriptor could be used to look up the socket descriptor and continue with the `write` call 