# OS Interfaces and Abstractions
## Interfaces
- The operating system is meant to support other programs via abstractions, which are delivered via interfaces
- Application program interfaces (APIs) are generally meant for programmers and provide a source-level interface specifying include files, data types, constants, macros, routines (and their parameters), etc.
    - This serves as a basis for software portability, as the API stays the same while the program simply needs to recompiled and linked for different architectures
- Application binary interfaces (ABIs) are a binary interface meant to bind a particular API to a hardware architecture, specifying dynamically loadable libraries, data formats, calling sequences, and linkage conventions
    - This serves as a basis for compatibility, as one binary serves all customers for that hardware (i.e. x86 Linux Version, MacOS Version, Windows 64-Bit Version, etc.)
    - ABIs are primarily for users since it allows for the programs that they want to download to be able to run on their device without the need for compiling, linking, etc.
- Normal libraries are accessed through an API, with source-level definitions of how to access the library
    - Dynamically loadable libraries are also called through an API, but the dynamic loading mechanism is ABI-specific
- Interfaces require stability in order to ensure interoperability - most programs depend on the interfaces of external code (system calls, libraries, etc.), so if the interface fails, the program fails
    - When a program calls an API, the requirements (i.e. parameters) are frozen at compile time, so future upgrades must still be able to support old requirements (its fine to extend interfaces, but changing parameters definitions and orders would ruin old code)
    - Thus, complying with the API - both from the programmer and user perspective - is important
- A *side effect* occurs when an action in one object has non-obvious consequences that is not specified by the interface
    - This results in unexpected behaviors, resulting in difficulty with debugging
    - From a programmer's standpoint, APIs should be implemented without side effects
    - From a user's perspective, the side effect, if not clearly intentional, should not be purposefully used as it could easily be patched out in a future implementation
## Abstractions
- Many operating system handles are complex - life can be made easier for application programmers if they work with simple abstractions, which are created, managed, and exported by the operating system
    - While hardware is indeed fast, using it directly involves a lot of complexity - encapsulating this complexity with abstractions allows for better error handling and performance handling while eliminating irrelevant behavior to the user