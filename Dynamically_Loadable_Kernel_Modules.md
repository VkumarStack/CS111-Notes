# Dynamically Loadable Kernel Modules
- Device drivers can be dynamically loadable - that is, they can be loaded and unloaded at runtime by the operating system
    - Due to the sheer amount of I/O devices present in modern systems, this makes sense as otherwise building them all at *load time* would require far too much memory and take far too much time (too initialize)
    - This is also necessary if an operating system is to work on any computer, where it must *discover* devices and load the required drivers - it may not know which devices are present beforehand, especially if such devices are plugged in during usage (i.e. USBs)
## Choosing Modules to Load
- In the past, dynamically loadable modules were *probed*, meaning that the system managing the driver implementations (a **factory**) would load all modules and determine which of the modules could identify the data associated with the device and therefore handle the device
    - This was unreliable, as devices could be incorrectly adapted, as well as unsafe, sinc the probing process could touch random registers in random devices
- More commonly today, I/O busses enable *self-identifying* devices - each device has a type, model, and serial number that can be queried
    - This information can be looked up in a device driver registry to select the appropriate driver for a given device - and there could even be precedence rules to choose the best driver among multiply matching choices
## Loading a Module
- Dynamically loaded kernel modules may require external functions, such as operating system services, and so it must be linked via a *run-time loader* that can adjust such external references at the module is being loaded
    - These unresolved, direct references can only be *from the dynamically loaded module to the program (operating system)* and not the other way around since it is not guaranteed that a dynamically loaded module is even present
## Initialization and Registration
- When the run-time loader is invoked by the operating system to load a dynamically loadable module, it commonly returns a pointer to that module's initialization function
    - This function can be used by the main program loading the module to actually use the device
    - The method may perform the standard initialization that a program may need (i.e. allocate memory and other resources) as well as **register** all of the device instances it supports
        - This registration process includes providing a vector of pointers to standard device entry points that can be used by the client to call such device instances 
## Using Dynamically Loadable Modules
- After the modules have been loaded, it is the operating system that typically provides a way to use device instances
    - On Linux, this is through the `/dev` file system, which contains special files associated with registered device instances 
        - Opening any of these special files results in a reference from that open file to the registered device instance so that any system calls on the file descriptor (`write`, `read`, etc) is forwarded to the appropriate driver entry point
- The operating system maintains a table of all registered device instances and their associated driver entry points; whenever a request is made for a device, this table is indexed by device indentifer to find the address of the entry point
## Unloading
- When the open file descriptors into a device have all been closed and it is no longer needed, the operating system *unloads*, which unregisters the driver, shuts down all of its associated devices, and frees all of its resources
## Stable Interfaces
- It is imperative that drivers provide stable interfaces, as other programs may depend on them
- It is also important for the operating system to provide stable interfaces for drivers, as otherwise the drivers would fail
## Hot-Pluggable Devices and Drivers
- Hot-plugs, such as USBs, generate events whenever a device is added or removed, which can thus prompt the appropriate loading and unloading functionality, respectively