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
- Passwords are regarded as an outdated technology but are still extremely used
    - Passwords have to be unguessable but still easy enough to remember for users
    - If networks connect remote devices to computers, they may be susceptible to password sniffers
    - Passwords, unless quite long, are susceptible to bruce force attacks
- If passwords *are* being used, they should be long, should be unguessable, should never be written down, and should never be shared 