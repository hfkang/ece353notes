# Security
April 5, 2017 

## Overall Principles
We want to restrict access to **individuals** who have been **authorized**. 


### Secure Channel
Enables secure commmunication between parties without compromising info to an **adversary**. The adversary may gain access to the communication channel and can 

1. Read/copy messages (eavsedropping)
2. Inject arbitrary data (tampering / replay)
3. Masquerade as another principle (mitm attack)

DDOS are not interesting as they don't have countermeasure.  
Security criteria: 

1. **Secrecy** data can't be read by unintended recipients
2. **Integrity** data tampering must be detectable
3. **Authentication** data attributed to correct author

#### Symmetric Algorithms

Both parties share a _secret key **K**_. 
Common name: **block cipher**. 

##### Requirements
1. **Correctness**: D(K, E(K,M) ) = M 
2. **Security**: D(? , E(K,M) ) = ??? 
    Without Key, it must be _computationally infeasible_ to determine the original message. 

DES uses 64b blocks 56b keys.  
AES uses 128b blocks 128b,192b,256b keys.  

Secrecy satisfied by assumption  
Integrity satisfied by adding msg checksum  
Authentication satisfied when key is secret  


How do you transmit the secret key to begin the session?
 
## Asymmetric Crypto Algorithms

Encrypt with public key, decrypt with private key. 

Secrecy, and integrity maintained with checksum, but no authorization. 

Given a public key, it is extremely hard to determine the private key. Likewise, a public key cannot be used to determine the original message. 

### Authentication via Digital Sig
When sending a message, Alice can encrypt it with their *private* key, and sends it to Bob. Bob decrypts it with her *public* key, which returns the original message. However, anyone can decrypt it so *secrecy is lost*.

**Idea** Combine the two! Sign a message with private key, then encrypt with public key. E_pub,B ( E_priv,A({M,C(M)}) ) = C. This is decrypted by B, verified with A's public key, and checksum verified. This satisfies all requirements of a secure channel. 

#### Disadvantages
Since the keys are 1024 - 2048 bits long, RSA is _expensive_! We don't want to use it for every single message that we send across the channel. 

### Signing with Secure Hash Functions 
H(M) -/> M  and M->H(M)  
Given H(M) it should be computationally infeasible to find another M' that returns the same hash value. 

Instead of signing the whole message, we sign the message digest, and can substantially reduce computational requirements when sending a messsage. 

E_pub,B({M,E_Priv,A(H(M))})     encrypt Msg, with signed hash 

### Combine Asym/Symm 
Instead of using RSA for everything, use it to establish a *session unique* shared secret key. Since the shared key K is short, it is computationally efficient. In general, SSL(TLS), SHTTP use this idea, as well as SSH. 

### Trusting Public Key
We ask a trusted entity S to give us the public key of B.
But how do we know K_pubS is legit? We can ask more trusted entities, and so on. 
Eventually this chain of certificates stops at the operating system. 

**Root Certificate Authorities** are well known authorities such as verisign.com who are trusted by default. 
