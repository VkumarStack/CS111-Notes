# Security and Privacy
## Introduction
- The operating system provides the lowest level of software visible to users and is very close to the hardware
    - It is imperative, then, that the operating system is secure as otherwise flaws in the operating system can compromise all security at higher levels
- **Security** is a policy (i.e. no unauthorized user may access this file) whereas **protection** is the mechanism (i.e. protection bits determine who can access the file)
- A **vulnerability** is a weakness that can allow an attacker to cause problems while an **exploit** is the actual incident of taking advantage of a vulnerability
    - Not all vulnerabilities can cause problems, and most vulnerabilities are never exploited (i.e. vulnerabilities on uncommon software/hardware)
- **Trust** is an important security concept as actions are performed for entities that are *trusted*
    - This trust must be expressed in terms of systems design 
    - The operating system *must be trusted*, as not trusting the operating system is futile (it controls hardware, processes, memory, etc.)
## Authentication and Authorization
- **Authentication** involves determining what party is requesting something and **authorization** involves determining if that party is allowed to actually receive that request
# Identities in Operating Systems
- Operating systems usually rely on user IDs to uniquely identify users
    - Processes run on behalf of users, so they inherit the user ID
        - Forked processes have the same user associated to it as the parent
    - This implies a model where any process belonging to a user has all their privileges
- The initial identity that is inherited is done at login time
    - The system must check that the password is correct, and this is typically done by comparing it to a stored hash
    - If the password is correct, then the user ID is tied to a new command shell or window management process
## Passwords
- Passwords are regarded as an outdated technology but are still extremely used
    - Passwords have to be unguessable but still easy enough to remember for users
    - If networks connect remote devices to computers, they may be susceptible to password sniffers
    - Passwords, unless quite long, are susceptible to bruce force attacks
- If passwords *are* being used, they should be long, should be unguessable, should never be written down, and should never be shared 
## Challenge/Response Systems
- Challenge/response systems are a sort of alternative to passwords in that they authenticate users by what questions they can answer correctly
    - Unlike passwords, there may be *multiple* questions, though this is not very practical
- Hardware-based challenge/response systems send the challenge to the hardware device rather than the user themselves
    - The device may perform a secret function on the challenge
        - i.e. A challenge sends a number, and the device adds a value to that number; the authentication mechanism expects the number they sent plus the value to be inputted
    - An example of this is a text challenge sent to a smart phone to be typed into a web request
- Utilizing a user challenge/response system is impractical  because there are usually too few unique and secret challenge/response pairs
    - Often, the response can be found by attackers (i.e. childhood pet)
    - Some forms of challenge/response authentication are susceptible to network sniffing, though this can be mitigated via encryption
## Biometric Authentication
- Biometric authentication is based on physical attributes of the user being authenticated, such as fingerprints, voice patterns, retinal patterns, etc.
    - These attributes are converted into a binary representation, which are checked against stored value representations of those attributes
    - If there is a *close* (not necessarily perfect) match, then the user is authenticated
- Biometric authentication requires *special hardware* in most cases (i.e. fingerprint reader)
- Additionally, many physical characteristics often vary too often for practical use (i.e. one's voice)
- Biometric authentication is not very useful for authenticating programs or roles, as biometrics are *for people*
- Biometric authentication requires special care when done across a network, as it is possible for the biometric information to be sniffed across a network if not careful
- If **false positives** frequently occur in a biometric system, it is likely that the system is *too generous in making matches* - another user can pretend to be the system's owner
- If **false negatives** frequently occur in a biometric system, it is likely that the system is *too strict in making matches* - the system's owner cannot log in to their account
## Multi-factor Authentication
- The current preferred approach in authentication is to utilize *multiple*, separate authentication methods - i.e. a password and a text message to a cell phone
    - This enables each method to compensate for the others' drawbacks, *if done correctly*
# Access Control
- The operating system controls which processes access which resources, which provides it the opportunity to enforce its security policies
    - The mechanisms to enforce policies on which parties can access resources is known as **access control**
## Access Control Lists
- One approach to access control is to use **access control lists**, which are maintained by the operating system for each protected object
    - Each entry in the list specifies which parties can access the object, as well as allowable modes of access (read, write, execute etc.)
    - Whenever a request is made to the object, its access control list is checked to determine if the requester can gain access
- Unix-based systems utilize access control lists per file (or anything represented by a file, such as a device or IPC channel), with three subjects on the list for each file (owner, group, and other) and three modes (read, write, execute)
- Access control lists enable ease in figuring out who can access a specific resource as well as ease in revoking or changing access permissions
- However, access control lists make it difficult to determine all resources a particular subject can access
    - This means that changing access rights for an object requires actually getting to that object (where it is stored)
## Capabilities
- Another approach to access control is to use **capabilities**, which are data objects that specify allowable access to objects
    - Thus, to access an object, the requesting party must present the proper capability
- Capabilities are essentially just a data structure, meaning they are a collection of bits
    - Since possessing the capability grants access, it must be ensured that they are not *forgeable*
    - One solution to ensuring that they are not forgeable is to store them in the *operating system* (i.e. in a process control block) so that users cannot change capabilities (and thus potentially forge them)
- Capabilities enable ease in determining what objects a particular subject can access as well as ease in transferring privilege (simply transfer the capability)
    - Utilizing capabilities allow for only the minimum required privilege to be assigned to parties (i.e. only display instead of potentially be able to delete files)
- Capabilities make it difficult to determine who can access a particular object and they may also require an extra mechanism to allow revocation
    - Capabilities also require extra care in distributed systems, where a malicious operating system could present a forged capability - in this case cryptography is likely necessary
## Operating System Use of Access Control
- Operating systems often use both access control lists and capabilities
- In Unix, access control lists are used for file opens but the resulting file descriptor (with a particular set of access rights) acts as a *capability*
## Enforcing Access in an Operating System
- Access control can only be enforced in an operating system if protected resources are inaccessible - usually through hardware protection so that only the operating system can make them accessible to a process
    - To get access to a resource, an application must make a system call such that the operating system can be trapped into, enabling it to consult the access control policy data
# Cryptography
- Cryptography essentially boils down to transforming bit patterns in controlled ways in order to obtain security advantages
- It is common for both symmetric and asymmetric cryptography to be used in tandem
    - Asymmetric cryptography is essentially used to bootstrap the symmetric cryptography
        - A public key algorithm is used to authenticate and establish a session key, which is used as the key for symmetric cryptography that is used for the rest of the transmission
    - This approach is favored because asymmetric cryptography is more expensive than symmetric cryptography, so it is best to use symmetric cryptography whenever possible
        - Though, the issue with symmetric cryptography is with key distribution, hence why asymmetric cryptography is utilized to securely distribute the symmetric key
- The security of asymmetric cryptography algorithms often relies on an underlying mathematical problem that requires a lot of work, such as factoring large numbers
    - This means that, if a mathematician "solves" that mathematical problem, then the underlying cryptography algorithms break