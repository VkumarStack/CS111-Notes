# Cryptography
- Cryptography is relevant for securing resources that are not solely controlled by the operating system
- Cryptography essentially involves an algorithm, known as a **cipher**, that, provided a **key**, converts data (**plaintext**) into a new form (**cipher-text**)
    - *C = E(P, K)*
- The key can be used to convert the cipher-text back into plaintext
    - *P = D(C, K)*
- The encryption and decryption algorithms should yield a result that is computationally infeasible to reverse (decrypt) without knowing the key
    - As such, security via cryptography relies entirely on the **secrecy of the key**, as it is common for the encryption and decryption algorithms to be known to the public 
- **Symmetric cryptography** involves utilizing the *same key* to encrypt and decrypt data, thus requiring both parties to have the *same key* if they are sending encrypted messages to each other
    - One party encrypts the data with the key and then sends it to the other party, which then decrypts the data with their copy of the key
## Public Key Cryptography
- In **public key cryptography**, two different keys are used - one to encrypt and one to decrypt
    - *C = E(P, K<sub>encrypt</sub> )*
    - *P = D(C, K<sub>decrypt</sub> )*
- In this scheme, a **public key** is given out to any party while only one party (or a few parties) retains the **private key**
    - The public key can be used for encryption and the private key for decryption or the other way around
        - In one case, a private encryption key could be kept by one party and this key could be used to distribute messages that can be deciphered using the public decryption key by other parties
            - Since there is only *one* private key, the decrypted message is understood to be from only the one party (holding the encryption key)
        - In another case, a private decryption key could be kept by one party and this key could be used to receive messages sent by others holding a public encryption key
            - Anyone can send an encrypted message, but only the party holding the decryption key can actually decrypt and view those messages
        - If both cases want to utilized, then *two different key pairs* are necessary
- Public key cryptography is more computationally expensive than traditional cryptography (which itself is somewhat expensive), so it should only be used when necessary
- Public key cryptography also requires a mechanism to properly distribute the public key, as it is possible for a malicious party to provide a false public key if not done properly
## Cryptographic Hashes
- **Cryptographic hashes** are hash functions that yield a *very low chance of findable collisions* (computationally infeasible to find two colliding inputs), provide *unpredictable results given a change to the initial input*, and are *difficult to infer properties of the input from based only on the hash value*
    - *S = H(P)*
    - The resulting hash, *S*, is typically shorter than *P*
- These cryptographic hashes can be used to verify data integrity: 
    - A piece of data can be cryptographically hashed and then the resulting hash itself can be encrypted and sent alongside the original, unencrypted message (the point here is data integrity, not secrecy)
    - The receiver can decrypt the hash and repeat the hasing operation on the data to check for any mismatches (tampering)
## Cryptography and Operating Systems
- Cryptography cannot necessarily be used to *hide* data from an operating system, or any system that has control over data resources, since it can likely access the secret key
    - Cryptography *can* be useful in the case of an attacker accessing the system if that attacker does not have access to the private key (or access to wherever it is stored) and sensitive data is encrypted (*not stored in plaintext in storage, memory, etc.*)
- Cryptography can be useful in the case of distributed environments, where data is encrypted on one machine and sent to another across a network; here, the local machine is trustworthy but other machines may not be, hence the need for encryption
## At-Rest Data Encryption
- **At-rest data encryption** involves storing *encrypted data* on persistent storage
    - Using this data requires decrypting it first (which may incur some overhead), and generally the key to do so is provided at boot time via request from the user (i.e. using a password)
    - This approach does not protect the data against users who are able to access the system via standard access control mechanisms or users who leverage flaws in the applications or operating system using the data (after they unencrypt it)
    - Rather, at-rest data encryption is relevant in cases where the hardware device storing the data is moved from one machine to another, which may not have the access key
        - This prevents potential sensitive data stored on a drive from being stolen as the other machine cannot access it
    - This may also be useful in cases where the data is provided to another, untrustworthy medium (such as a cloud storage facility) - an encrypted version of the data can be sent for storage and then when it is requested back it can be decrypted using a key only available to the host machine
## Cryptographic Capabilities
- Capabilities can be implemented using cryptography techniques, ensuring that capabilities are not forged
    - This can be done either via symmetric cryptography or public key cryptography, with the latter being common in cases where the resource manager and the requester may not be as close
    - The capabilities themselves could contain information such as expiration times, identification information, and so forth, and cryptography ensures that such information is valid