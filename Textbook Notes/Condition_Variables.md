# Condition Variables
- In cases where a thread wishes to check whether a **condition** is true before execution (such as a parent checking if all its children threads have completed), **condition variables** are used
- When the condition is not as desired, the checking thread **waits** until some other thread changes the state of the condition and **signals** the waiting thread(s) to wake
- Example:
    - ![Condition Variable Example](../Images/Condition_Variable_Example.png)
    - Mutex locks are necessary when dealing with condition variables in order to address any possible race conditions
        - The parent could check that `done` is zero but get *interrupted before actually trying to `sleep`*, switching to the child that then changes `done` to one and signals the parent to wake up - but since the parent was interrupted before sleeping, the wake up signal will do nothing and when the parent returns it will sleep forever
        - The lock prevents interrupts from ruining any important functionality since, in the above example, if the parent were interrupted before actually sleeping, the child would not be able to continue since the lock is set by the parent; when the parent does wait (which releases the lock), the child can access the lock as necessary
    - The state variable `done` is necessary, as otherwise, in the case where the child runs first, the child could signal *before* the parent runs `thr_join()`, which would then result in it waiting forever when it does actually call `thr_join()` since it does not know the condition is already desirable