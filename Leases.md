# Lease-Based Serialization
- Locking does not feasibly work with distributed systems since:
    - Obtaining the lock through message exchange takes a long amount of time
    - The locking operation itself could get lost due to networking issues
    - A node that crashes while holding a lock can result in other nodes waiting on that lock to hang
        - There typically no manager to determine if a node has failed and clean it up
- It is common for locks to be replaced with **leases**, which grants the owner of a resource exclusive access until they release it *or* until the lease duration expires
    - Otherwise, leases work akin to locks, though they are often enforced - a request to a remote server will typically contain a copy of the requestor's lease, which is used to verify the request
- Leases are often economical, especially if many operations are performed under a single lease-grant