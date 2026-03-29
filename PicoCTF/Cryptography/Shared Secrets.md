# 🚩 CTF Writeup: Shared Secrets

**Author:** Yahaya Meddy  
**Category:** Cryptography  

---

## 📜 Challenge Description
> A message was encrypted using a shared secret... but it looks like one side of the exchange leaked something. Can you piece together the secret and get the flag?

---

## 🕵️‍♂️ The Recon

The target was using a classic Diffie-Hellman Key Exchange (DHKE) to establish a secure communications line. In a standard DH handshake, the server and client agree on public parameters (`g` and `p`). They each generate a private secret (`a` and `b`), and compute public keys (`A` and `B`) to swap over the network. 

The shared secret is then calculated using the other party's public key and your own private key. For the client, the math is `A^b mod p`.

But the target got incredibly sloppy. When I intercepted their `message(1).txt` file, I didn't just find the public parameters. They left the client's private secret (`b`) sitting right out in the open. 

## 🐛 The Vulnerability

Diffie-Hellman relies entirely on the absolute secrecy of the private variables (`a` and `b`). If an attacker compromises either one of those keys, the underlying math offers zero protection, and the cryptographic fortress collapses. 

Because the target leaked `b`, we didn't have to break the discrete logarithm problem. We could just walk right through the front door and calculate the exact same shared secret the client did.

## 💥 The Exploit

With the private key `b` secured, I dropped into Python to calculate the shared secret, derive the XOR key, and decrypt the intercepted message.

**The Decryption Script 🗝️**
According to `encryption.py`, the cipher only uses the lowest byte of the shared secret for its XOR operation. I wrote this script to automate the extraction:

```python
# 1. Load the leaked intel (Truncated for brevity, use full numbers from message.txt)
p = 2945889223405899... 
A = 1925392662772808... 
b = 2348305787882664... 
enc_hex = "6178727e5245576a75794e6222726322654e23727028267372276c"

# 2. Reconstruct the shared secret (A^b mod p)
shared = pow(A, b, p)

# 3. Isolate the XOR key (lowest byte)
key = shared % 256

# 4. Decrypt the payload
enc_bytes = bytes.fromhex(enc_hex)
flag = bytes([x ^ key for x in enc_bytes])

print(flag.decode('utf-8'))
```

Executing the script instantly shattered the XOR cipher and dumped the plain text flag. Contract closed. 🏴‍☠️

## 🛡️ The Fix (How to not get hacked like this)

To keep your encrypted comms secure:
* **Never** hardcode, log, or transmit private keys (`a` or `b`) in plain text or debug files.
* Use established, modern libraries for key exchanges (like ECDHE) rather than rolling your own cryptographic implementations.
* Ensure proper operational security (OpSec) so diagnostic files and secrets don't leak into production environments.