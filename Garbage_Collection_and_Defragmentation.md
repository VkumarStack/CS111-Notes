# Garbage Collection
- In a garbage collected system, resources are allocated but never *explicitly* freed
    - Rather, when available resources become significantly small, garbage collection is initiated 
        - This pool of resources is first scanned to find all resources that are still *reachable* - if it is, it is removed from the original resource list
        - At the end of the scan, anything still in the original list of resources can be freed
- Garbage collection requires all active resource references to be identifiable, requiring data structures associated with resource references
- Garbage collection often comes at a performance cost; a naive garbage collector may temporarily halt the program for garbage collection to occur
    - More modern garbage collectors use background garbage collectors, running continously although at the cost of added complexity and still overall performance overhead
# Defragmentation
- Defragmentation actually *organizes* resources such that they are *contiguous* and *compacted*
- The defragmentation process often involves identifying an open space to where the resource can be copied to in order to free up space and then coalescing it into a continous chunk