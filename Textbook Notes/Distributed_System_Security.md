# Distributed System Security
- In dealing with authentication with respect to distributed systems, most approaches utilize passwords and private keys
    - Passwords are typically utilized when multiple parties (i.e. users) need to authenticate themselves to a single entity (i.e. the service)
    - Private keys are typically utilized when a single entity (i.e. the service) needs to authenticate itself to multiple parties (i.e. users)
- If a user provides a password, the password itself must be encrypted before being transported across the network
    - This requires that a symmetric key is shared with the service, or the service's public key has been already provided
## Public Key Authentication for Distributed Systems
- For public key authentication to work, the public keys must be distributed - the actual process of distributing public keys rely on third party **certificates**, each of which contains information about the party that owns the key and the public key itself (and other info)
    - All of the information is cryptographically hashed and the result is encrypted using the third party's private key
        - When a copy of a certificate is provided, its signature can be checked and the public key can be obtained
- **Certificate authorities** are companies that create certificates that are trusted by many parties
    - When a party wishes to get a certificate, it (very carefully) provides its public key to the authority, which then runs a cryptographic hashing algorithm (i.e. SHA-3) on the party's name, public key, and other information
        - The resulting hash is then encrypted by the authority's own private key, which produces a digital signature
        - All of the information used to produce the hash, the authority's identity, and the digital signature are all combined to form the certificate
    - When an entity needs to obtain the public key (and it does not have it already), it is sent the certificate by the party, which is examined to verify the key
        - The entity examines the certificate, identifies the authority associated with the certificate, and uses the information (in the certificate) to produce a hash in the same manner that the authority had initially created the certificate with
        - The digital signature in the certificate is the decrypted using the authority's (already known) public key, which results in the hash produced by the authority; this hash is compared with the hash produced by the entity, and if there is a match, then the public key is legitimate
        - After verifying the key, the entity can simply store the key so that the process does not need to be repeated in the future
- The certificate approach requires the certificate authorities' public keys to already be known - many software applications often come with keys to hundreds of trusted authorities
- After acquiring the public key, messages between the entity and party are encrypted - this means that a symmetric key can be sent between the parties without any worry of the key being exposed
    - Although the messages sent are encrypted, it is important to ensure that each encrypted message contains unique information not in any other message, as otherwise a **replay attack** could occur where an attacker makes a copy of a legitimate message (that they do not know since it is encrypted) and sends it later
## SSL/TLS
- Machines typically communicate over a network using sockets, and so it is necessary to add cryptographic features to such sockets
    - This was initially done through **SSL (Secure Socket Layer)**, but its implementation was faulty and instead the newer **Transport Security Layer (TSL)** is used (though TSL is often used interchangably with SSL)
- SSL functions to move encrypted data through an ordinary socket by setting up a structure to encrypt data and send its output to the input of the socket, which sends a message that is decrypted on the other end
- A SSL connection typically bootstraps a secure connection based on asymmetric cryptography 
    - Typically, the party being communicated with via a SSL connection is verified using a certificate (in the same manner as aforementioned), though other methods exist
        - Servers will typically have certificates, but clients usually do not - so the client is sure about the server's identity but the server likely does not know the client's identity 
            - This is fine as the client can verify themselves to the server using a password (after the SSL connection is established)
- The two parties negotiate on which ciphers, hashes, key distribution strategies, and authentication schemes should be used
    - After the negotiations have been decided, both sides utilize the appropriate techniques and strategies to generate a symmetric key that can be used for subsequent communications
- SSL/TLS is a *protocol*, and thus there may be different software packages that *implement the protocol*
    - There may be a flaw in a package, but this does not necessarily imply that there is a flaw in the protocol itself (though if there is a flaw in the protocol, then all packages are likely flawed too)
## Web Cookies
- Another common authentication method involves the usage of **web cookes**, which are pieces of data that a website sends to a client, with the intention of that data being stored and sent back whenever the client next communicates with the server
    - When a user provides their password, the server can create a web cookie that securely stores the client's identity and send it to them
    - The client's web browser, then automatically includes the cookie in the body of future requests, which verifies the client's identity without asking them for the password again
- Cookies involve a security tradeoff, as they are more convenient for users (who do not need to type in their password every single time) but may be more susceptible to attacks, such in the case where the attacker is able to directly access where the cookies are stored (and therefore have access to the client's account)
## HTTPS
- **HTTPS** takes the existing HTTP definition and connects it with SSL/TLS, ensuring that interactions between the client and server are *secure*
    - There has been a push for servers insisting on HTTPS and refusing to communicate with clients that use HTTP
## SSH
- **SSH** or **Secure Shell** is a competitor to SSL used to set up secure sessions with remote computers
- SSH utilizes similar techniques as in SSL, in that it utilizes public keys and certificates to authenticate servers, and typically passwords (though other methods are available) to authenticate clients
- SSH may be susceptible to **man-in-the-middle** attacks, where two parties believe that they are communicating directly when in actuality they are unknowingly communicating through a malicious third party, which can see all messages passed between the two and even inject their own messages
    - These attacks stem from a fishy public key used to start the SSH session; if the remote site simply sends the public key to the client (rather than rely on certificates), then a middleman can intercept the sent public key and replace it with their own, allowing them to pose as the server