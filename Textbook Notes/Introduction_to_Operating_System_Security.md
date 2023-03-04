# Introduction to Operating System Security
- Security is especially important for applications such as operating systems, which provide a foundation that other applications depend on
    - If an operating system were insecure, then all other applications at higher levels would also, by extension, be insecure
## Security Goals and Policies
- One major security goal that systems try to achieve is that of **confidentiality**, as information that is meant to be hidden from others (users, processes, etc.) should stay hidden
- Another security goal is that of **integrity**, as a component that is expected to have a certain state (possibly by other applications) should not be maliciously changed to some other state
- One last major security goal is that of **availability**, as a malicious party should not be able to prevent the usage of a resource or service
- These goals are kept in mind with the idea of *controlled sharing*, as some systems may need to be made available to a set of preferred users and not available to others
- Security goals are implemented via **security policies**, which leverage (typically) existing operating system mechanisms that ensure that such goals are met properly
## Designing Secure Systems
- **Economy of mechanism**: The system should be kept as small and simple as possible, as this ensures there are fewer bugs and better understanding of underlying behavior
- **Fail-safe defaults**: Any default behavior for the system should be *secure*
- **Complete mediation**: Actions should be checked to determine if they meet security policies
- **Open design**: Systems should be designed as if their inner workings are already known; they should be secure regardless if their inner workings (not to be confused with inner data such as passwords, etc.) are exposed
- **Separation of privilege**: Separate parties or credentials should perform critical actions (i.e. two-factor authentication)
- **Least privilege**: A user or process should be given the *minimumm* privileges requried to perform a certain action
- **Least common mechanism**: Separate mechanisms or data structures should be used to handle different users or processes (i.e. own page table in virtual memory)
- **Acceptability**: The system should still be usable despite any security policies
## Basics of Operating System Security
- Most security goals related to operating systems involves ensuring that *access to hardware* is controlled, as all processes share hardware and therefore exploiting a hardware vulnerability can result in exploitation of other processes
    - Other goals involve securing built-in operating system services, such as file systems, memory management, and interprocess communications
- System calls enable security goals to be implemented well, the process identifier can be used by the operating system to determine the identity of the calling process 
    - It can then determine via **access control mechanisms** if the process is **authorized** to perform the given action