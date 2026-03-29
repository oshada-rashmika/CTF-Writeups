# 🚩 CTF Writeup: StegoRSA

**Author:** Yahaya Meddy  
**Category:** Steganography / Cryptography  

---

## 📜 Challenge Description
> A message has been encrypted using RSA. The public key is gone… but someone might have been careless with the private key. Can you recover it and decrypt the message?

---

## 🕵️‍♂️ The Recon

The challenge name "StegoRSA" gave the entire game plan away. The target encrypted a message in a `.enc` file but got sloppy and tried to hide their RSA private key right inside a standard `.jpeg` image. 

Like a ghost on a midnight mountain pass, I bypassed the encryption entirely and went straight for the hidden cargo. I ran some basic forensics on the image and ripped out a massive, continuous block of hexadecimal data that was bleeding out of the file.

The hex string started with `2d2d2d2d2d424547494e2050524956415445204b45592d2d2d2d2d...`

If you read hex often enough, you know `2d` is the ASCII code for a hyphen `-`. It didn't take an anime villain monologue to realize this was the standard `-----BEGIN PRIVATE KEY-----` header, stripped of its formatting and converted to raw hex. 

## 🐛 The Vulnerability

The target committed two fatal errors:
1. **Steganography is not Encryption:** Hiding a private key inside an image file using basic tools or appending data does not protect it from standard forensic extraction.
2. **Poor Key Management:** Your private key is the master key to your digital life. Exposing it, even obfuscated in hex, instantly shatters any RSA encryption you've built. 

## 💥 The Exploit

I had the key in pieces. It was time to rebuild the engine and shatter the target's encryption.

**Step 1: Reconstructing the Key 🗝️**
I dropped into the terminal and used the `xxd` utility to reverse the hex encoding, forging a perfectly formatted `.pem` file out of the raw data. 

```bash
echo "2d2d2d2d2d424547494e2050524956415445204b45592d2d2d... [TRUNCATED FOR BREVITY] ...454e442050524956415445204b45592d2d2d2d2d0a" | xxd -r -p > private.pem
```

Checking the `private.pem` file confirmed the hit: a fully intact RSA Private Key.

**Step 2: Shattering the Encryption 💥**
With the target's private key secured, their `.enc` file offered zero resistance. I fired up OpenSSL to decrypt the package and extract the prize:

```bash
openssl pkeyutl -decrypt -inkey private.pem -in message.enc -out flag.txt
```

*(Note: On older rigs, `openssl rsautl -decrypt -inkey private.pem -in message.enc -out flag.txt` does the trick).*

I read the `flag.txt` file, secured the intel, and vanished. Contract closed. 🏴‍☠️

## 🛡️ The Fix (How to not get hacked like this)

To keep your encrypted comms from getting hijacked:
* **Never** store your private keys in the same location as the data they encrypt.
* **Stop relying on obscurity:** Hiding a key inside an image file (Steganography) is a parlor trick, not a security protocol. 
* Use dedicated key management systems (KMS) or secure hardware enclaves to store cryptographic keys.