# Virtual Machines
- A **virtual machine** is a software illusion meant to appear to be a real machine
    - Software is utilized to make it appear that multiple computers (often different kinds of computers) are running on a single, real computer
- The virtual machine must appear to apps and users as a real machine
    - In virtual machines, *real hardware* is utilized to implement *virtual hardware*
- Virtual machines are utilized for several reasons:
    - Fault isolation
        - A virtual machine's operating system can crash and not take down the entire (real) machine in the process
    - Security
        - A virtual machine does not see real shared resources, so processes in other virtual machines are protected from each other
    - Use of different operating systems
        - If an application only works on one operating system (i.e. it makes use of a different system call interface), then it can be run on a virtual machine, even though the underlying real machine may not be the compatible operating system
    - Controlled sharing
        - It is easier to guarantee an entire virtual machine gets a set allocation of resources, which ensures that processes running in it will not steal resources for other virtual machines
            - This is vital for cloud computing
## Running Virtual Machines
- Virtual machines are easiest to implement if the virtual and real machine use the same instruction set architecture
- In this case, limited direct execution is utilized so that the virtual machine activity is run directly on the CPU
- The controller that manages all virtual machines is known as the **hypervisor**, and whenever the virtual machine traps, it traps into the hypervisor (rather than the virtual machine's operating system)
    - This trapping occurs whenever the virtual machine does anything privileged - the visor will then forward the request to the virtual machine's operating system
        - Subsequent privileged operations will then trap back into the visor