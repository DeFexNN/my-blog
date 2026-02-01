---
title: Linux Penguin
---
## Writeup: Linux Penguin (PascalCTF 2026)

This challenge involves recovering hidden plaintext from a server that encrypts words using AES. By analyzing the provided source code, we can identify a critical cryptographic flaw that allows us to bypass the encryption entirely.

---

### 1. Analysis

The core of the challenge lies in how the data is encrypted. Looking at the Python source:

```python
key = os.urandom(16)
cipher = AES.new(key, AES.MODE_ECB)
...
def encrypt_words(wordst: list[str]) -> list[str]:
    ...
    padded_word = word.ljust(16)
    encrypted = cipher.encrypt(padded_word.encode()).hex()

```

#### Key Findings:

* **AES Mode:** The script uses **ECB (Electronic Codebook)** mode.
* **Deterministic:** In ECB mode, the same plaintext block always produces the same ciphertext block if the key is the same.
* **Limited Entropy:** There are only 28 possible words that the server can choose from.
* **Oracle Access:** The server allows us to encrypt up to 28 words (7 rounds of 4 words) of our choosing before asking us to guess the secret words.

---

### 2. The Vulnerability: ECB Pattern Leakage

Because the server uses a static key for the duration of the session and uses ECB mode, we can perform a **Known Plaintext Attack**. If we send the server a word we know (e.g., "unconstitutional"), it will give us the hex ciphertext. If that same hex string appears in the secret "Challenge Ciphertext," we know exactly which word it represents without ever needing to "crack" the AES key.

---

### 3. Exploitation Strategy

To solve this, we automate the following steps:

1. **Build a Codebook:** Send all 28 possible words to the server in chunks of 4.
2. **Map the Responses:** Store the resulting hex strings in a dictionary where the key is the hex and the value is the English word.
3. **Identify the Secrets:** Receive the 5 target hex strings from the server and look them up in our dictionary.
4. **Submit:** Send the translated words back to the server to claim the flag.

#### Solution Script (using `pwntools`)

```python
from pwn import *

words = ["biocompatibility", "biodegradability", "characterization", ...] # Full list from source

io = remote('penguin.ctf.pascalctf.it', 5003)
lookup = {}

# Step 1 & 2: Build the lookup table
for i in range(0, 28, 4):
    chunk = words[i:i+4]
    for word in chunk:
        io.sendlineafter(b": ", word.encode())
    
    io.recvuntil(b"Encrypted words: ")
    enc_hexes = io.recvline().decode().strip().split(" ")
    for word, hex_val in zip(chunk, enc_hexes):
        lookup[hex_val] = word

# Step 3: Parse target
io.recvuntil(b"Ciphertext: ")
target_hexes = io.recvline().decode().strip().split(" ")

# Step 4: Submit guesses
for hex_val in target_hexes:
    io.sendlineafter(b": ", lookup[hex_val].encode())

io.interactive()

```

---

### 4. Conclusion

The challenge demonstrates why AES-ECB is insecure for almost all applications. By observing the mapping between plaintext and ciphertext, an attacker can build a dictionary of values. To prevent this, developers should use modes like **AES-GCM** or **AES-CBC** with a unique **Initialization Vector (IV)** for every encryption, ensuring that even identical plaintexts result in unique, unpredictable ciphertexts.

**Flag:** `pascalCTF{why_4r3_th3_bl0ck_4lw4ys_th3_s4m3???}`

---
