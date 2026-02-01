---
title: Curve Ball
---
**Pohlig-Hellman algorithm** to solve the Elliptic Curve Discrete Logarithm Problem (ECDLP).

Here is a structured writeup of the challenge for your records or blog.

---

# Writeup: Curve Ball (PascalCTF 2026)

## 1. Challenge Overview

The challenge provides a custom implementation of a cryptographic gambling system using an elliptic curve over a finite field. We are given the curve parameters, the generator point , and a public point . The goal is to find the `secret` scalar such that .

### Parameters:

* **Prime **: 
* **Curve**: 
* **Order **: 
* **Generator **: 

## 2. Vulnerability Analysis: Smooth Order

The security of Elliptic Curve Cryptography relies on the difficulty of the Discrete Logarithm Problem. Usually, we use curves where the order  is a large prime.

However, looking at the provided order , we see it is **highly composite** (smooth). By factoring , we find:


Since the largest prime factor is only **47**, the curve is extremely vulnerable to the **Pohlig-Hellman algorithm**.

## 3. The Exploit Strategy

The Pohlig-Hellman algorithm breaks one large DLP into several smaller DLPs in subgroups of the curve.

1. **Subgroup Mapping**: For each prime power factor  of , we map the points  and  into a subgroup of order  by multiplying them by .


2. **Solve Small DLPs**: Since  is small (maximum 47 or ), we can find  using simple brute force or Baby-step Giant-step.
3. **Reconstruct with CRT**: Once we have the secret modulo every factor, we use the **Chinese Remainder Theorem** to calculate the full secret modulo .

## 4. Implementation

The Python script connects to the server, parses the public point , performs the subgroup multiplications, solves the small DLPs via brute force, and uses a CRT function to recover the 64-bit secret.

### Key Snippet (DLP & CRT):

```python
for q, e in FACTS:
    r = q ** e
    k = n // r
    Gr = k * G
    Qr = k * Q
    xr = dlog_bruteforce(Gr, Qr, r) # Brute force in small subgroup
    congr.append((xr, r))

secret, M = crt(congr)

```

## 5. Conclusion

The challenge demonstrates that the bit-length of the prime  is not the only metric for security in ECC. Even with a 64-bit prime, if the group order is smooth, the "Discrete Log" becomes trivial to compute.

**Flag**: `pascalCTF{sm00th_0rd3rs_m4k3_3cc_n0t_s0_h4rd_4ft3r_4ll}`

---

