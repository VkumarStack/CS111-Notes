# Stack Frames and Linkage Conventions
## Stack Model
- Local variables (as well as program parameters) are stored in a stack, with new call frames being pushed on to the stack whenever a procedure is called (or execution block is entered) and old frames being popped off the stack whenever a procedure returns (or a block ends)
    - The stack model for procedure calls is very common, with most processor architectures providing built-in hardware instructions for stack management
        - Though, for memory needed beyond local scope, the heap should be used instead
## Subroutine Linkage Conventions
- Subroutine linkage tends to involve some basic form of *parameter passing*, *subroutine calling* (save return address and jump to called routine), *register saving* (of nonvolatile registers), and *space allocation*
- When the routine completes, there is also some basic form of a *return value*, *popping of local storage*, *register restoration*, and *subroutine return*
- These conventions are ISA specific, though it is common for register saving to be the responsibility of the called routine (since it is the routine that knows what registers it will use and which it will leave untouched) whereas dealing with parameters is often the responsibility of the calling routine (since it knows what parameters and how many parameters it will use)
## Traps and Interrupts
- ISAs provide support for interrupts, which inform software that an external event has happened, and traps, which inform the software of an execution fault
- Dealing with traps and interrupts is similar to dealing with procedure calls in that there is a transfer of control to the appropriate handler, requiring the saving of state before and the restoration of state after the handler runs
    - There is a difference, though, in that interrupts and traps are not explicitly requested by the software - thus, the linkage conventions are under the hardware's control rather than the software
- Interrupt/Trap Mechanisms:
    - Each interrupt or exception is associated with a particular number, which is associated with the appropriate code through a trap table initialized by the operating system
    - When an interrupt or trap occurs, the CPU uses the appropriate interrupt/trap number to index to the appropriate table entry, loading the appropriate state (Program Counter and Processor Status word), and then executing from the new state
        - The code executing is typically a *first level handler*, which saves all of the general registers on the stack, gathers information from the hardware on the cause of the interrupt/trap, and then chooses and makes a procedural call to the appropriate *second level handler*, which actual deals with the interrupt/exception. Once the second level handler returns, the first level handler restores all of the saved registers, executes a return-from-interrupt/trap instruction, reloads the program counter and processor status, and resumes execution of the original program