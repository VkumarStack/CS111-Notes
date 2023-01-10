# Operating Systems Principles
- Operating systems are examples of very complex software, and dealing with complex software often requires project management techniques and design principles in order to ensure that one does not get overwhelmed
    - Make use of *hierarchical decomposition*, which involves decomposing a complex system into smaller components that can understood individually
    - Ensure that your software is *modular*, meaning that the aforementioned components have a coherent purpose and its functions can be performed entirely within the component; combining the various components together allows for the broader system's purpose to be performed
        - Modules should be *encapsulated*, meaning that their inner details should not be necessary to understand by external users and can change without affecting its outer interface whatsoever
        - A good rule of thumb is to keep closely related components in a single module, as frequent communication between differing components often results in performance overhead
    - Appropriately *abstract your interfaces* - a programming interface should provide meaningful parameters (flags, arguments, etcs.) that can easily get the desired results and *hide information* regarding the inner complexity of the interface, as this should be unnecessary for the client to understand
        - On the topic of interfaces, understand that interfaces are *contracts* with users - the interfaces themselves should not be changed (though more options can be added) as many users depened on their functionality; changing the inner workings (the implementation) is completely fine so long as the interface itself is untouched
    - Incrementally develop your projects - functionality should be added in smaller steps, possibly even one feature at a time
- Operating systems make use of a variety of architectural paradigms for easier development
    - One such paradigm is the *mechanism* and *policy* separation
        - A mechanism keeps track of resources and manages access to such resources (i.e. keeping track of processes and switching/killing processes)
        - A policy controls which clients to get the resources when (i.e. scheduling process switching)
        - The two should be separated; changing the mechanism should not affect the policy and vice-versa
    - Another paradigm deals with accomodating multiple implementations of similar functionality (i.e. various types of file systems)
        - This is addressed via plug-in modules, which can be added to the system as needed - these modules must adhere to a common interface and are indirectly accessed (not built in to the operating system)
            - The indirect access may be bound only when necessary, in which case it is dynamically discovered/loaded
    - When adjusting a system for a particular load, designing dynamic equillibria can be useful - that is, a system that can continuously adapt to a wide range of conditions
    - Choose the correct data structure, considering their speed, complexity, error recovery, and potential for parallelism
    