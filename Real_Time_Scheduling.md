# Real Time Scheduling
- A **real-time** system is one whose correctness depends not only on functionality, but also *timing* - think of flight controllers or media players (audio or video) that have specific intervals in which they must perform processing
    - Real-time processes tend to be more predictable, meaning that it is generally known how long they run
- For real-time systems, it would make sense to use a **real-time scheduler** where the priority of a process can be explicitly set 
    - If the needs are not complex, then there might not even be a need for a scheduler - a task can just yield to another task or a form of static, batch scheduling can be performed (one process after another)
    - If it is necessary to use a dynamic scheduler, though, then it is important for real-time processes to be scheduled appropriately so that they are not mistimed
        - This means that allowing for other processes to starve is fine so long as the timing requirements of the real-time processes are met
        - This may also involve non-preemptive (time splitting) scheduling since splitting the execution of a process can result in it missing its timing deadline, and since real-time programs often have known runtimes, preemption itself is usually not necessary
- Linux provides a fairly adequate real-time scheduler; Windows should not be used for real-time scheduling, especially that of large scale and precise timing 