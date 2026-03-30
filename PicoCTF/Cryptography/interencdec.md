# 🚩 CTF Writeup: interencdec

**Author:** NGIRIMANA Schadrack  
**Category:** Cryptography / Encoding  

---

## 📜 Challenge Description
> Can you get the real meaning from this file.

---

## 🕵️‍♂️ The Recon

The challenge name `interencdec` stands for Intermediate Encoding/Decoding. This is a classic case of security by obscurity. The target didn't use actual encryption to lock down their secrets; they just wrapped the flag in multiple layers of different encodings, hoping to throw investigators off the scent. 

We intercepted a single string of data:
`YidkM0JxZGtwQlRYdHFhR3g2YUhsZmF6TnFlVGwzWVROclh6ZzVNR3N5TXpjNWZRPT0nCg==`

Any string of alphanumeric characters ending in `==` padding is a dead giveaway. The outer shell is Base64. 

## 💥 The Exploit

This was a triple-wrapped package. To extract the payload, we had to peel back three distinct layers.

**Step 1: The Outer Shell (Base64) 🧅**
I dropped the initial payload into the terminal to strip away the first layer of Base64 encoding:
```bash
echo "YidkM0JxZGtwQlRYdHFhR3g2YUhsZmF6TnFlVGwzWVROclh6ZzVNR3N5TXpjNWZRPT0nCg==" | base64 -d
```
The output: `b'd3BqdkpBTXtqaGx6aHlfazNqeTl3YTNrXzg5MGsyMzc5fQ=='`

**Step 2: The Inner Shell (Base64... again) 🧅**
Notice the Python bytes string syntax (`b'...'`)? The target tried to be clever by double-wrapping the payload. I stripped the `b'` and `'` away and decoded the string *inside* the quotes:
```bash
echo "d3BqdkpBTXtqaGx6aHlfazNqeTl3YTNrXzg5MGsyMzc5fQ==" | base64 -d
```
The output: `wpjvJAM{jhlzhy_k3jy9wa3k_890k2379}`

**Step 3: The Core Cipher (Caesar) 🗡️**
The string looked like gibberish, but the formatting (`wpjvJAM{...}`) perfectly mirrored standard flag formats. The target got lazy. Instead of encrypting the final layer, they just used a basic substitution cipher (Caesar Cipher).

Knowing the flag had to start with `picoCTF{`, I looked at the first letter: `w`. The letter `w` is exactly 7 letters ahead of `p` in the alphabet. 

I ran the string through a Caesar decoder, shifting the entire alphabet backward by 7 (or forward by 19) to snap the text back into place. 

The encryption shattered, revealing the plain text:
**Flag:** `picoCTF{caesar_d3cr9pt3d_890d2379}`

Contract closed. 🏴‍☠️

## 🛡️ The Fix (How to not get hacked like this)

* **Encoding is not Encryption:** Base64, URL encoding, and Hex are designed to format data for transport, not to hide secrets. 
* **Avoid weak ciphers:** The Caesar cipher was broken by mathematicians centuries ago. Always use modern, cryptographically secure algorithms (like AES) to protect sensitive data.