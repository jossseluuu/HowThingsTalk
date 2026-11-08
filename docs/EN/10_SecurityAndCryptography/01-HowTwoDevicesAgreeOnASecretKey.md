---
title: "How do two devices agree on a secret key over a channel anyone can listen to?"
section: "10_SecurityAndCryptography"
language: "en"
status: "published"
order: 1
depth: "intermediate"
summary: "With Diffie-Hellman key exchange, each device picks a secret number, publishes a value derived from it, and combines the other side's value with its own secret. Both reach the same key without ever sending it, because undoing the operation is infeasible for today's computers. To stop anyone posing as the other end, the exchange must be signed."
prerequisites:
  - "What it means to encrypt data with a key, at a general level"
  - "Layers of a protocol stack (glossary)"
translation: "../../ES/10_SeguridadYCriptografia/01-ComoAcuerdanDosAparatosUnaClaveSecreta.md"
---

# How do two devices agree on a secret key over a channel anyone can listen to?

## Question and framing

When a phone opens an HTTPS website, the traffic crosses a café's WiFi, an operator's router and several data centres. Any of them can copy every byte. Even so, within a few milliseconds the phone and the server share a key that only they know. They had never talked before, and neither has sent the key to the other. The question is how that is possible.

This article explains Diffie-Hellman key exchange: the mathematical idea, the elliptic-curve version that almost every connection uses today (X25519), and how TLS 1.3 uses it. It also explains why the exchange alone is not enough and a signature is needed, and what changes in the face of a quantum computer. It does not cover how data is encrypted afterwards with that key, or how certificates work internally. It does not describe attacks on specific systems.

The depth is intermediate. You only need to know that encryption transforms data using a key, and have a rough idea of the layers of a protocol stack, which are defined in the [glossary](../../../guides/TaxonomyAndGlossary.md). The arithmetic needed is explained along the way.

## Short answer

Each device mixes a secret number of its own with a public value, sends the result, and repeats the mix with what it receives from the other side; both arrive at the same value, and an eavesdropper cannot undo the mixing to get it.

### 60-second summary

The trick relies on an operation that is easy to do and very hard to undo. Both ends share a public starting point. Each picks a random secret number and applies the operation to the starting point as many times as its secret says. It sends the result to the other side, unprotected.

Next, each side applies its own secret to the result it received. Because the order does not matter, both get the same final value. That value has never crossed the network.

An eavesdropper sees the starting point and both public results. To reach the final value they would need one of the secrets. Recovering it from the public results is a problem that, at today's sizes, nobody knows how to solve in reasonable time on classical computers.

On its own, the exchange does not tell you who you are talking to. Someone in the middle could run a separate exchange with each end. That is why TLS 1.3 adds a server signature over the whole negotiation. And because the secrets are discarded at the end of each connection, stealing the server's key later does not decrypt traffic that was already recorded.

## Mental model

The classic picture uses paint. Two people publicly agree on a base colour, say yellow. Each privately adds a secret colour: Alice adds red and Bob adds blue. They swap their mixtures in plain sight. Then each adds their own secret colour to the mixture they received. Both end up with the same shade: yellow, red and blue. An onlooker has only seen yellow, orange and green, and separating a paint mixture back into its colours is practically impossible.

The analogy breaks in three places. First, the difficulty of un-mixing paint is physical and obvious; the difficulty of undoing the mathematical operation is an assumption. Nobody has proved that no fast method exists, although decades of research have not found one. And a large quantum computer would have one. Second, with paint the final result is a colour; in the protocol it is a number that still has to be turned into encryption keys. Third, paint offers no protection against someone who sits in the middle and pretends to be Bob to Alice. That protection has to come from another component.

## Technical mechanism

### 1. What happens in the system

Whitfield Diffie and Martin Hellman published the idea in 1976. Their version uses modular arithmetic: always working with the remainder after dividing by a prime number `p`. A public number `g` is chosen. Each party picks a secret and computes `g` raised to that secret, modulo `p`.

A toy example with `p = 23` and `g = 5`:

1. Alice secretly picks `a = 6` and computes `A = 5^6 mod 23 = 8`. She sends 8.
2. Bob secretly picks `b = 15` and computes `B = 5^15 mod 23 = 19`. He sends 19.
3. Alice computes `19^6 mod 23 = 2`.
4. Bob computes `8^15 mod 23 = 2`.

Both reach 2 because `(5^15)^6` and `(5^6)^15` are the same number: `5^(6·15)`. The eavesdropper sees 23, 5, 8 and 19. To get to 2 they need to know that 8 is 5 raised to the power 6. That problem is called the **discrete logarithm**. With 23 it can be solved by trial, but for numbers hundreds of digits long there is no known practical way to solve it.

Today the same idea is almost always used on an **elliptic curve**. Instead of raising a number to a power, a point on the curve is "added" to itself as many times as the secret says. That operation is called scalar multiplication. Undoing it is the discrete logarithm on the curve. Curves reach the same security with much shorter numbers.

### 2. What each component does

**The public group.** This is the playing field: a prime and a generator in the classic version, or a curve and a base point in the elliptic one. It is fixed by a standard and is the same for everyone. It is not secret.

**Each party's secret.** A random number that never leaves the device. In X25519 it is 32 random bytes, according to RFC 7748 (Section 6.1). Its quality depends on the random number generator: a predictable secret defeats the whole scheme.

**The public value.** The result of applying the secret to the base point. It travels unencrypted. In TLS 1.3 it goes inside the `key_share` extension.

**The shared secret.** The final result, identical at both ends. It is not used directly as a key. TLS 1.3 feeds it into its key schedule, built on the HKDF derivation function. That produces separate keys for each direction and each phase.

**The signature.** The server signs the negotiation with the private key that matches its certificate. This is the `CertificateVerify` message, which RFC 9846 describes as a signature over the entire handshake. If someone in the middle changes a public value, the signature does not match and the connection is dropped.

### 3. Key parameters

- **X25519 curve.** Defined in RFC 7748. It works modulo the prime `2^255 − 19`, with base point `u = 9`. The secret, the public value and the shared secret are 32 bytes each. The RFC places it at roughly the 128-bit security level, "slightly under" the standard 128-bit level (Section 7).
- **X448 curve.** Also in RFC 7748, for a level of about 224 bits. Its public value is 56 bytes (RFC 9846, Section 4.3.8.2).
- **Named groups in TLS 1.3.** Among others, `x25519` (code 0x001D) and `secp256r1` (0x0017), according to RFC 9846 (Section 4.3.7).
- **Checks on the received value.** In the classic version, each party must check that the other's value satisfies `1 < Y < p − 1`. On NIST curves, that the point really lies on the curve (RFC 9846, Section 4.3.8). For X25519, RFC 7748 allows checking that the shared secret is not all zeros; RFC 10024 makes the check mandatory in hybrid mode.
- **Single use.** RFC 9846 says clients and servers must not reuse a `key_share` value across connections (Section 4.3.8).
- **Size in post-quantum hybrid mode.** With `X25519MLKEM768` (code 0x11EC, RFC 10024), the client's value is 1216 bytes (1184 for ML-KEM and 32 for X25519) and the server's is 1120 bytes. Compared with 32 bytes for X25519 alone, that is about 38 times more.

### 4. Diagram or flow

Exchange in a full TLS 1.3 handshake with X25519, without pre-shared keys:

```text
Client (phone)                                       Server (website)
--------------                                       ----------------
secret c  (32 random bytes)                          secret s  (32 random bytes)
C = c · P   (P = base point, u = 9)                  S = s · P

ClientHello  [key_share: C] ----------------------->
                                                     K = s · C
             <----------------------- ServerHello  [key_share: S]
             <----------------------- Certificate  (server's public key)
             <----------------------- CertificateVerify  (signature over the whole handshake)
             <----------------------- Finished
K = c · S
checks certificate and signature
Finished  ------------------------------------------>

K = c·s·P at both ends  ->  HKDF  ->  traffic keys
The eavesdropper sees P, C and S, but not c, s or K.
c and s are erased when the connection ends.
```

## Limits, exceptions, and real-world cases

**Without a signature there is no protection against someone in the middle.** An active intermediary can run one exchange with the client and another with the server. Each end would believe it shares a key with the other, and the intermediary would read everything. Diffie-Hellman protects against listeners, not impersonators. Authentication comes from the certificate and the `CertificateVerify` signature. If the client does not check the certificate, the exchange loses its value.

**Forward secrecy depends on discarding the secrets.** RFC 8446 already stated that TLS 1.3 removed the static RSA and static Diffie-Hellman modes, and that all its public-key based key exchanges provide forward secrecy (Section 1.2). This means that stealing the certificate's key later does not decrypt recorded sessions. The guarantee only holds if the ephemeral secrets are actually erased. That is why RFC 9846 forbids reusing `key_share` values. Even so, to avoid breaking compatibility, it requires accepting a peer that does reuse them.

**Received values are checked.** A malformed public value can force a weak or predictable shared secret. That is why the standards require the checks listed above. It is one reason curves and formats are fixed by standards, instead of parameters chosen by each application.

**Randomness is the foundation.** If a device's random number generator produces repeated or guessable secrets, nothing else matters. The scheme assumes `c` and `s` are unpredictable.

**The quantum computer.** Shor's algorithm, published in 1997, solves the discrete logarithm in polynomial time on a large enough quantum computer. Proos and Zalka adapted the algorithm to elliptic curves in 2003. No machine able to do this at X25519 sizes is publicly known. The risk is that someone records encrypted traffic today and decrypts it once such a machine exists. The IETF's answer is the hybrid exchange in RFC 10024 (August 2026). It combines X25519 with ML-KEM-768, a key-encapsulation mechanism standardised by NIST in FIPS 203 (August 2024). According to RFC 10024, the combination stays secure as long as at least one of the two components does.

**ML-KEM is not Diffie-Hellman.** In key encapsulation, the client publishes a key and the server sends back a secret "encapsulated" with it. The practical result is the same, a shared secret that is never sent in the clear, but the structure is different. The price is size: more than 1 kB in each direction instead of 32 bytes.

**One more value to guess.** The client sends its `key_share` before knowing which groups the server accepts. If the server supports none of those offered, it asks for another with a `HelloRetryRequest`. RFC 9846 (Section 4.3.8) notes that this costs an additional round trip.

## Practical consequences

For the person using the device, an open WiFi network or a curious operator can see which server the phone connects to and how much data it exchanges. They cannot see the content of a properly authenticated TLS connection. The browser padlock means exactly that: a key exchange took place and a signature could be verified.

An invalid-certificate warning is not a minor detail. It is the sign that the part preventing impersonation has failed. Accepting it leaves the key exchange without its protection against intermediaries.

For people designing devices or services, the lesson is not to invent. Curves, formats and checks are fixed in standards because failures show up in the details: unvalidated values, reused secrets or poor randomness. It is also worth planning room for hybrid mode: a `ClientHello` carrying 1216 extra bytes of key material may no longer fit in a single network packet.

## Frequently asked questions

### Is this the same as encrypting with the server's public key?

No. In the old static RSA modes, the client chose the key and sent it encrypted with the certificate's public key. Anyone who later stole the private key could decrypt everything recorded. In ephemeral Diffie-Hellman, the key comes from two secrets that are discarded at the end. The certificate's key is only used to sign, not to protect the session key.

### Does Diffie-Hellman encrypt the data?

No. It only produces a shared secret. The data is then encrypted with a symmetric algorithm, such as AES-GCM or ChaCha20-Poly1305, using keys derived from that secret.

### Why use elliptic curves instead of the prime-number version?

Size and speed. With curves, the same security needs much shorter numbers: X25519 works with 32-byte values for about 128 bits of security. The classic version still exists in TLS 1.3 as finite-field groups, but with much larger values.

### Would a quantum computer also break the AES encryption of the data?

Not in the same way. Shor's algorithm attacks the discrete logarithm and factoring, which underpin the key exchange and signatures. Symmetric encryption does not rely on those problems. That is why the urgent change is happening in key exchange.

## Sources

- [Diffie, W. and Hellman, M. E. — New Directions in Cryptography. IEEE Transactions on Information Theory, vol. IT-22, no. 6, November 1976, p. 644 ff.](https://ee.stanford.edu/~hellman/publications/24.pdf) Original proposal of the exchange with `Y = α^X mod q` and common key `α^(X_i·X_j) mod q`, based on the difficulty of logarithms over a finite field (Section III).
- [RFC 7748 — Elliptic Curves for Security (IRTF, January 2016)](https://www.rfc-editor.org/rfc/rfc7748.html). Prime `2^255 − 19` and base point `u = 9` (Section 4.1), curve448 (4.2), 32-byte inputs and outputs of X25519 (5), exchange protocol and optional all-zero check (6.1), security level (7).
- [RFC 8446 — The Transport Layer Security (TLS) Protocol Version 1.3 (IETF, August 2018)](https://www.rfc-editor.org/rfc/rfc8446.html). Removal of static RSA and Diffie-Hellman, and forward secrecy in all public-key based exchanges (Section 1.2).
- [RFC 9846 — The Transport Layer Security (TLS) Protocol Version 1.3 (IETF, July 2026)](https://www.rfc-editor.org/rfc/rfc9846.html). Obsoletes RFC 8446. Description of the handshake and of `CertificateVerify` (Section 2), named groups (4.3.7), the `key_share` extension, the ban on reusing it, the cost of an extra round trip and checks on received values (4.3.8, 4.3.8.1 and 4.3.8.2).
- [RFC 10024 — Post-Quantum Traditional (PQ/T) Hybrid Key Agreement Mechanisms for TLS 1.3 (IETF, August 2026)](https://datatracker.ietf.org/doc/draft-ietf-tls-ecdhe-mlkem/). Definition of hybrid exchange (Section 3), client and server `key_share` sizes (4.1 and 4.2), mandatory all-zero check (4.3) and the `X25519MLKEM768` code point (7.1).
- [NIST FIPS 203 — Module-Lattice-Based Key-Encapsulation Mechanism Standard (13 August 2024)](https://csrc.nist.gov/pubs/fips/203/final). The ML-KEM standard.
- [Shor, P. W. — Polynomial-Time Algorithms for Prime Factorization and Discrete Logarithms on a Quantum Computer. SIAM Journal on Computing, 26(5), 1484-1509, 1997](https://doi.org/10.1137/S0097539795293172). Quantum algorithm for the discrete logarithm.
- [Proos, J. and Zalka, C. — Shor's discrete logarithm quantum algorithm for elliptic curves. Quantum Information and Computation, 3(4), 317-344, 2003](https://arxiv.org/abs/quant-ph/0301141). Adaptation of Shor's algorithm to elliptic curves.
