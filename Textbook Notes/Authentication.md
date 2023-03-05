# Authentication
- In the context of security, the party asking for something (i.e. a resource or some other kind of **object**) is known as the **principal** and the entity that performs this request is known as the **agent**
    - The agent determines if access to the object is permitted, typically by referring to a **credential**
## Attaching Identities to Processes
- Processes are usually created by other processes, and this mechanism can be used to attach identities to new processes (as child processes inherit the identity of the parent process)
- Most systems base identities based on the *human user* they belong to
    - These users work with a main process (such as the shell process in a terminal system or a window manager in a windowed system) that can spawn other processes inheriting their user ID
        - The main process gets it identity at startup, via a login
## Authentication By What You Know
- Authentication via password is fairly common, though it is imperative that this password does not leak - whether it be from the person who created the password or the system that accepts the password
- The system does not actually store the password itself but rather stores a **hash** of the password
    - Whenever a user enters a password, that entered password is hashed and compared to the store hash 
    - This is secure because it is very difficult to reverse a hash to find the original password (and inputting the hash itself as a password does not work) 
- It is common for passwords to contain common names or phrases (since they are easy to remember), and as a result attackers may attempt **dictionary attacks**, where they guess passwords using lists of names or words
    - Most systems secure against these types of attacks by limiting logins to a certain number of guesses, beyond which either the system shuts off access to the account or drastically slow down the process of password checking (to discourage attackers since the attack will now take a very long amount of time)
- Passwords are also **salted**, meaning that a random number is generated and concatenated to the password and the hashed result is stored
    - This random number is typically stored in the password file next to the hashed password
    - Whenever the user attempts to log in, the random number is concatenated to the inputted password and the result is hashed and compared with the stored hash
    - This approach is more secure as it accounts for the case where a password file is stolen and a dictionary attack is run by running many different phrases through the hashing algorithm until the one that matches the stored password is found
        - Each dictionary attack phrase must have a translation for every possible salt (which can be a lot of the salt is 32 or 64 bits), which ultimately requires more computing time and therefore makes it harder to actually crack the password
## Authentication By What You Have
- Another form of authentication involves providing some kind of identity device or object, such as a security token that can be plugged in via USB 
    - These devices typically rely on frequent changes of whatever information the device passes to the operating system verifying authenticity, as otherwise keeping hte information static would simply turn the authentication mechanism into a password-based one rather than an object-based one
- This approach, of course, has issues with what would occur if the authentication object were *lost* or even *stolen*
## Authentication By What You Are
- Biometric-based authentication, such as via fingerprint or face scan, has become increasingly prevalent, though they work very differently from other authentication mechanisms
- These mechanisms cannot *perfectly* determine whether a match (in face, fingerprint, etc.) is correct and are therefore susceptible to **false negatives** (where the real user is not matched) as well as **false positives** (where a fake user is matched)
    - Ultimately, the **sensitivity** should be adjusted to account for the situation at hand - i.e. smartphones may want to minimize false negatives as it would be inconvenient for users whereas banks may want to minimize false positives as they do not want to get robbed easily
- Biometrics are not common due to the technology requiring such forms of authentication not being present on every device - i.e. embedded devices do not have ready access to cameras or fingerprint scanners
- It is also important to be wary of biometric authentication that is *remote*, as it could be possible for biometric readings to be picked up by attackers during the communication process and then used to hack into the device being protected
    - This is not the case if the biometric layer is physically connected to the machine, as there is less opportunity for the information to slip out
## Authenticating Non-Humans
- When considering authentication for entities such as web servers, which are not really associated with a human user, it is common for a human user to *start* the entity but then change the ownership of the entity (in some manner) to not inherit the user ID but instead its own 
- It also common for systems to authenticate via *group membership* - an ID can be indexed into a list of groups that the user belongs to, and privilege can be based on group when necessary
    - This could be useful for processes that are authenticated via group membership rather than individual identification