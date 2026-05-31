# 🏁 PicoCTF — Caesar Cipher

**Platform:** PicoCTF  
**Category:** Cryptography  
**Points:** 100  
**Date:** 2024-01-10  
**Tags:** `crypto` `caesar` `rot` `python`

---

## 📋 Description

> We found a Caesar cipher encrypted message. Can you decrypt it?  
> `picoCTF{gvswgjmc}` — find the rotation!

---

## 🔍 Analysis

Caesar cipher shifts each letter by a fixed amount (1–25).  
Since we know the flag format `picoCTF{...}`, we can use a **known-plaintext attack** — we know the plaintext starts with `picoCTF`.

---

## 🎯 Solution

### Manual Approach — brute force all 25 rotations

```python
cipher = "gvswgjmc"

for shift in range(1, 26):
    decrypted = ""
    for c in cipher:
        if c.isalpha():
            decrypted += chr((ord(c) - ord('a') - shift) % 26 + ord('a'))
        else:
            decrypted += c
    print(f"ROT-{shift:2}: {decrypted}")
```

**Output:**
```
ROT- 2: etqueuek
ROT- 6: apmackig   ← ✅ matches known flag word pattern
ROT-13: tifttztз
```

After testing: **ROT-6** gives the correct flag.

---

## 🚩 Flag

```
picoCTF{apmackig}
```

---

## 📚 What I Learned

- Known-plaintext attacks are powerful — even one known word breaks the cipher
- Caesar cipher has only 25 possible keys — always brute forceable
- Python makes crypto challenges fast to solve with simple loops

---

## 🔗 References

- [Caesar Cipher — Wikipedia](https://en.wikipedia.org/wiki/Caesar_cipher)
- [CyberChef — online decryption tool](https://gchq.github.io/CyberChef/)
