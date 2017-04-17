April 10, 2017
## Message Auth Code
Used for **integrity and auth**, assumes secret key already established. 

## HMAC HashMAC

h1 = Hash(Innerpad + message)  
h2 = Hash(Outpad + h1) 

two layers of salted hash, with predefined salt pattern.  
Innerhash = 5c5c5c...  
Outerhash = 363636...

```
function hmac(key, msg)
    if (len(key) > blocksize)
        key = hash(key)
    if (len(key) < blocksize)
        key = key || 0x00 * blocksize - len(key)
    o_key_pad = [0x5c * blocksize] XOR key
    i_key_pad = [0x36 * blocksize] XOR key
        return hash(o_key_pad || hash(i_key_pad || msg)) 
```

## TLS 

### Layer presence 
1. Application
2. TLS
3. Transport (TCP)


###Handshake Protocol
Asymmetric is best! Use it to establish a shared secret key. 

1. Client Sends `ClientHello` (TLS Ver, nonce) 
2. Server Sends `ServerHello` (Chosen protocol ver, nonce) 
3. Server sends `Certificate` (PubKey of Server) 
4. Server sends `ServerHelloDone`
5. Client sends `ClientKeyExchange` (`PreMasterSecret` encrypted with server public key) 
6. `PreMasterSecret` + 2 nonces are used to encrypt future communications. 
7. Client sends `ChangeCipherSpec` to initiate encrypted comm. 
8. Client sends `Finished` which is encrypted. 
9. Server sends `ChangeCipherSpec` then `Finished` likewise. 

### MAC Choice
In TLS, we use HMAC with SHA-256 for MAC. 

# User Auth
- Must indicate one of the three:
- Something the user **knows**
- Something the user **has**
- Something the user **is**

### Passwords
If the user demonstrates knowledge of password, then give access. 

#### Disadvantages
People are lazy and you can guess/steal passwords from elsewhere.  
Opens door for brute force attack.  
Can bypass with SQL injection sometimes. 

#### Storing Passwords 
Use file permissions and hash algo to save passwords. 

### Dictionary Attack
If hash viewable, then people can consult rainbow tables to see what password is. 

### Salting
Hash random string with password. Change string when password changes. 
