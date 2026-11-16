---
title: "How does your phone know a website is really who it claims to be?"
section: "10_SecurityAndCryptography"
language: "en"
status: "published"
order: 2
depth: "intermediate"
summary: "The server presents a certificate: a public document that binds its domain name to a public key and is signed by a certificate authority. The phone follows the chain of signatures up to a root it already has installed, checks the name, dates and revocation, and makes the server produce a fresh signature with the private key. Only whoever holds that key can produce it."
prerequisites:
  - "Diffie-Hellman key exchange (article 10/01)"
  - "What encrypting data with a key means, in general terms"
translation: "../../ES/10_SeguridadYCriptografia/02-ComoSabeElMovilQueUnaWebEsAutentica.md"
---

# How does your phone know a website is really who it claims to be?

## Question and framing

You type your bank's name into the browser. Between your phone and the server sit a WiFi network, a router and several carriers. Any of them could answer in the bank's place. The [article on Diffie-Hellman](01-HowTwoDevicesAgreeOnASecretKey.md) explained how two devices agree on a secret key. It also warned that the exchange does not tell you who you are talking to. This article covers that missing part: how the phone checks that the other end really is the owner of the domain.

It covers digital signatures at a conceptual level, what a certificate contains, the chain of trust up to the roots shipped with the system, and the checks a client performs in TLS 1.3. It also explains how the authorities that issue certificates are kept in check: revocation and Certificate Transparency. It does not cover the internal mathematics of each signature algorithm, or how to configure a server. It does not describe attacks against specific systems.

The depth is intermediate. Reading the Diffie-Hellman article first helps, although the concepts needed are recapped here.

## Short answer

The server shows a certificate signed by an authority the phone already trusts and, at the same moment, signs the conversation with the private key that certificate names; the phone checks both signatures, the name and the dates, and only then carries on.

### 60-second summary

A digital signature uses two keys. The private key is held only by its owner and is used to sign. The public key can be held by anyone and is used to check signatures. Nobody can forge a valid signature without the private key.

A certificate is a document that says "public key X belongs to `bank.example`". A certificate authority (CA) signs it after checking that whoever requested it controls that domain. Often the signature comes from an intermediate CA, whose own certificate is in turn signed by a root CA.

The phone ships with a list of roots it trusts. When the server sends its certificate, the phone follows the chain of signatures up to one of those roots. It checks that the name matches what you typed, that the certificate has not expired and that it has not been revoked.

The certificate is public and anyone can copy it. That is why there is one last step. The server signs a digest of the whole conversation with its private key. An impostor can replay the certificate, but cannot produce that signature.

## Mental model

Think of a door check that asks for an ID card. The card carries your name and photo. It was issued by an agency the guard recognises. The guard checks the agency's seal, the expiry date and that the photo looks like you. If everything matches, you get in.

The analogy breaks in three places. First, the digital "card" is public: anyone can hold an exact copy. What plays the role of the photo is a live proof, a signature only the holder of the private key can make. Second, the CA does not check who you are or whether you are honest. Usually it only checks that you control the domain. A fraudulent site can hold a perfectly valid certificate for its own domain. Third, the guard judges by eye, but the phone runs exact mathematical checks: a signature is either valid or not, with nothing in between.

## Technical mechanism

### 1. What happens in the system

It all starts well before the connection, when the domain owner requests a certificate:

1. **They generate a key pair.** The private key stays on the server. The public key goes into the request.
2. **They prove control of the domain.** The CA asks them, for example, to publish a specific value in a DNS record of the domain or at a path on its website. The ACME protocol (RFC 8555) automates these challenges, called `dns-01` and `http-01` (sections 8.4 and 8.3). The permitted methods are listed in section 3.2.2.4 of the CA/Browser Forum Baseline Requirements. Since September 2017 the CA must also check the domain's CAA record, where the owner can state which CAs are allowed to issue.
3. **The CA signs the certificate** and logs it in several public transparency logs (explained below).

Then, on every TLS 1.3 connection:

1. The phone and the server run the Diffie-Hellman exchange. From then on, the rest of the handshake is encrypted.
2. The server sends its certificate and the intermediates in the `Certificate` message. According to RFC 9846 (section 4.5.1), its own certificate must come first and each following one should certify the one before. The root can be left out, since the client already has it.
3. The server sends `CertificateVerify`: a signature, made with its private key, over the hash of every message exchanged so far.
4. The phone validates the chain, checks the name and verifies the `CertificateVerify` signature. If anything fails, it drops the connection.

### 2. What each component does

**The digital signature.** The signer computes a digest of the data and transforms it with the private key. The verifier uses the public key to check that the signature matches exactly that data. If a single bit changes, the check fails. Its security rests on mathematical problems similar to those behind key exchange: the elliptic-curve discrete logarithm (ECDSA, Ed25519) or factoring (RSA). An Ed25519 signature is 64 bytes long (RFC 8032, section 5.1.6).

**The certificate.** Its format is X.509 version 3, profiled in RFC 5280 (section 4.1). The signed part is called `tbsCertificate` and includes, among other fields:

- the serial number;
- the issuer, meaning the CA that signs it;
- the validity: start and end dates;
- the subject and its public key (`subjectPublicKeyInfo`);
- the extensions, such as the domain names and whether the certificate may sign other certificates.

After it come the signature algorithm and the CA's signature over everything above.

**The domain names.** They go in the `subjectAltName` extension, as entries of type `dNSName` (RFC 5280, section 4.2.1.6). RFC 9525 defines how they are compared. The client uses the name it meant to visit, not one supplied by the server. It compares label by label, ignoring case. The old *Common Name* field may no longer be used to identify a service (RFC 9525, section 2). A wildcard is allowed, but only as the complete left-most label and covering a single label (section 6.3). So `*.bank.example` covers `web.bank.example`, but not `bank.example` nor `a.web.bank.example`.

**The intermediate CA and the root.** The `basicConstraints` extension states whether a certificate belongs to a CA. If it does not, its key must not be used to verify signatures on other certificates (RFC 5280, section 4.2.1.9). This stops the owner of an ordinary certificate from issuing certificates for other domains. Roots are usually kept offline. Intermediates do the daily work, so a compromised intermediate can be withdrawn without replacing the root.

**The root store.** This is the list of roots the operating system or browser trusts. Each vendor maintains its own through a dedicated program. Chrome, for example, relies solely on its Chrome Root Store to authenticate TLS servers. Version 1.8 of its policy (February 2026) removes any root whose key was generated more than 15 years ago (section 1.3.1.2). Information about CAs is shared through the common CCADB database.

**The `CertificateVerify` signature.** This is the proof of possession. According to RFC 9846 (section 4.5.2), the server signs the concatenation of four pieces: 64 bytes of value 0x20, the context string `TLS 1.3, server CertificateVerify`, a single 0 byte and the digest of the conversation. The padding and the context string prevent the signature from being reused in another protocol or another role. The digest includes both sides' Diffie-Hellman values. So if someone in the middle had changed them, the signature would not match. If verification fails, the client must abort with a `decrypt_error` alert.

### 3. Key parameters

- **Checks on every certificate in the chain** (RFC 5280, section 6.1.3). The signature verifies with the public key of the certificate above. The validity period includes the current time. The certificate is not revoked. The issuer matches the subject of the certificate above.
- **Maximum validity of a server certificate.** According to version 2.3.1 of the CA/Browser Forum Baseline Requirements (October 2026, section 1.2.2): 200 days from 15 March 2026, 100 days from 15 March 2027 and 47 days from 15 March 2029.
- **Reuse of domain validation.** Same schedule: 200 days, then 100 and, from March 2029, 10 days. After that, the CA must check again that the requester controls the domain.
- **Transparency in Chrome.** A certificate must carry stamps from public logs (SCTs): from at least 2 distinct logs if its lifetime is 180 days or less, and from 3 if longer. At least two must come from different operators. Without them, Chrome rejects it.
- **Transparency at Apple.** At least two SCTs are required. With embedded stamps: 2 from distinct logs for 180 days or less, and 3 for 181 to 398 days.
- **Signature schemes in TLS 1.3** (RFC 9846, section 4.3.3). Among others, `ecdsa_secp256r1_sha256` (0x0403), `rsa_pss_rsae_sha256` (0x0804) and `ed25519` (0x0807). The client advertises which ones it accepts in the `signature_algorithms` extension.

### 4. Diagram or flow

Validation in a typical TLS 1.3 connection with one intermediate CA:

```text
PHONE'S ROOT STORE (installed before the connection)
  [Root R]  public key of R
       |
       | signs
       v
  [Intermediate I]  subject: I · issuer: R · cA = true
       |
       | signs
       v
  [Leaf L]  subject: bank.example · issuer: I · cA = false
            server's public key · valid 200 days · 3 SCTs

Phone                                              Server (bank.example)
-----                                              ---------------------
ClientHello  [key_share, signature_algorithms] ------>
             <------ ServerHello  [key_share]
                     (from here on, everything encrypted)
             <------ Certificate  [L, I]
             <------ CertificateVerify  [signature with L's private key
                                         over the conversation digest]
             <------ Finished

1. Does R's key, from the store, verify I's signature?
2. Does I's key verify L's signature?   Is I a CA?
3. Is today within the validity of L and I?   Neither revoked?
4. Is "bank.example" in L's subjectAltName?
5. Does L carry the required SCTs?
6. Does CertificateVerify verify with L's public key?

All yes -> phone's Finished and application data.
Any no  -> alert and connection dropped.
```

## Limits, exceptions, and real-world cases

**A certificate proves control of the domain, not honesty.** A fraudulent site at `bank-secure.example` can obtain a valid certificate for that name. The padlock only guarantees that you are talking to whoever controls the domain shown in the address bar. Paying attention to the name is still up to the user.

**Any trusted CA can issue for any domain.** If a single CA in the store makes a mistake or is compromised, it could issue a certificate for someone else's domain. There are three defences. The CAA record narrows which CAs may issue. Certificate Transparency puts every issuance in plain sight. And root programs remove CAs that break the rules.

**Certificate Transparency.** A transparency log is an append-only Merkle tree (RFC 9162, section 4). When it receives a certificate, the log returns an SCT: a promise to include it within a fixed maximum time, the *Maximum Merge Delay*. Browsers require those stamps. A certificate issued in secret would therefore not work, and one issued in public can be spotted by the domain owner by monitoring the logs. Transparency does not block wrongful issuance; it makes it visible.

**Revocation is the weakest piece.** If a private key is stolen, the CA can revoke the certificate. Telling every client is hard. Certificate revocation lists (CRLs, RFC 5280, section 5) are signed lists of revoked serial numbers. They can be large. Online OCSP queries, by contrast, reveal to the CA which site you are visiting. Let's Encrypt switched off its OCSP service on 6 August 2025 for that privacy reason and now publishes only CRLs. Some browsers compress revocation data for the whole web into filters they download periodically. In 2020 Mozilla described how CRLite reduced about 300 MB of revocation data to around 1 MB. Short-lived certificates are another answer: a 47-day certificate stops being valid soon even if nobody revokes it.

**The device's clock matters.** The validity check uses the phone's clock. If the date is badly off, correct certificates look expired or not yet valid.

**Roots added by an employer or the user.** A managed device may have company roots installed. With them, equipment on the corporate network can present certificates the phone accepts and inspect the traffic. This is legitimate on a managed device, but it changes whom you are trusting. Apps can apply their own rules, such as accepting only certain keys (unverified as a general rule).

**Quantum computers affect signatures too.** Shor's algorithm would break ECDSA, Ed25519 and RSA just as it breaks Diffie-Hellman. In August 2024 NIST published a resistant signature standard, ML-DSA (FIPS 204). The urgency differs. A weak key exchange lets traffic recorded today be decrypted later. A signature only has to hold up while the connection lasts. That is why migrating key exchange is ahead of migrating certificates.

## Practical consequences

The invalid-certificate warning is the only visible sign that this chain has failed. It may be an innocent error: a clock that is off, an expired certificate or a public WiFi captive portal. But the phone cannot tell it apart from an impersonation attempt. Accepting it switches off exactly the part that protects you from intermediaries.

For anyone running a service, the 200-day maximum validity, dropping to 47 days in 2029, makes automatic renewal close to mandatory. A forgotten certificate makes the service unreachable. It also pays to publish a CAA record and to monitor the transparency logs to spot certificates issued in your name without permission.

For device designers, the root store is a living component. A device that is never updated keeps roots that have been withdrawn and lacks the new ones. Over time it stops being able to validate legitimate sites.

## Frequently asked questions

### Does the certificate encrypt the connection?

No. The certificate authenticates. Encryption uses keys derived from the Diffie-Hellman exchange. In TLS 1.3 the certificate's private key is only used for signing.

### Is a paid certificate more secure than a free one?

Not cryptographically. The checks the browser performs are the same. What differs between certificate types is how much the CA verifies about the organisation. The browser does not use that to decide whether the connection is secure.

### If the certificate is public, why can't an impostor use it?

Because showing it is not enough. The server has to sign a digest of the conversation with the private key, right then. The digest includes fresh random values from that connection, so an old signature is useless.

### What happens if a server's private key is stolen?

The thief could impersonate that server until the certificate is revoked and clients find out, or until it expires. They could not decrypt past connections, because in TLS 1.3 session keys come from Diffie-Hellman secrets that are erased when the connection ends.

## Sources

- [RFC 5280 — Internet X.509 Public Key Infrastructure Certificate and CRL Profile (IETF, May 2008)](https://www.rfc-editor.org/rfc/rfc5280.html). Certificate fields (section 4.1), `subjectAltName` (4.2.1.6), `basicConstraints` (4.2.1.9), revocation lists (5) and per-certificate checks in path validation (6.1.3).
- [RFC 9525 — Service Identity in TLS (IETF, November 2023)](https://www.rfc-editor.org/rfc/rfc9525.html). Ban on using the *Common Name* (section 2), reference identifiers (6.1), matching rules and wildcards (6.3).
- [RFC 9846 — The Transport Layer Security (TLS) Protocol Version 1.3 (IETF, July 2026)](https://www.rfc-editor.org/rfc/rfc9846.html). Signature schemes (section 4.3.3), `Certificate` message and chain order (4.5.1), construction of `CertificateVerify` and the `decrypt_error` alert (4.5.2).
- [RFC 8555 — Automatic Certificate Management Environment (ACME) (IETF, March 2019)](https://www.rfc-editor.org/rfc/rfc8555.html). `http-01` (section 8.3) and `dns-01` (8.4) challenges for proving control of a domain.
- [RFC 8032 — Edwards-Curve Digital Signature Algorithm (EdDSA) (IRTF, January 2017)](https://www.rfc-editor.org/rfc/rfc8032.html). 64-byte Ed25519 signature (section 5.1.6).
- [RFC 9162 — Certificate Transparency Version 2.0 (IETF, December 2021)](https://www.rfc-editor.org/rfc/rfc9162.html). Log as an append-only Merkle tree, SCT and *Maximum Merge Delay* (sections 4 and 4.1).
- [CA/Browser Forum — Baseline Requirements for TLS Server Certificates, version 2.3.1 (4 October 2026)](https://cabforum.org/working-groups/server/baseline-requirements/requirements/). Schedule for maximum validity and validation reuse (section 1.2.2), domain validation methods (3.2.2.4) and CAA mandatory since version 1.4.3 (September 2017).
- [Chrome Root Program Policy, version 1.8 (5 February 2026)](https://googlechrome.github.io/chromerootprogram/crp/policy/). Sole reliance on the Chrome Root Store and removal of roots with keys older than 15 years (section 1.3.1.2).
- [Chrome — Certificate Transparency Policy](https://googlechrome.github.io/CertificateTransparency/ct_policy.html). Number of SCTs by certificate lifetime and rejection of non-compliant certificates.
- [Apple — Apple's Certificate Transparency policy (21 April 2025)](https://support.apple.com/en-us/103214). Number of SCTs required by certificate lifetime.
- [CCADB — Common CA Database](https://www.ccadb.org/). Database shared by root programs.
- [Let's Encrypt — Ending OCSP Support in 2025 (5 December 2024)](https://letsencrypt.org/2024/12/05/ending-ocsp/) and [OCSP Service Has Reached End of Life (6 August 2025)](https://letsencrypt.org/2025/08/06/ocsp-service-has-reached-end-of-life). Privacy rationale and move to CRLs.
- [Mozilla Security Blog — CRLite Part 1: All Web PKI Revocations Compressed (9 January 2020)](https://blog.mozilla.org/security/2020/01/09/crlite-part-1-all-web-pki-revocations-compressed/). Compression of about 300 MB of revocations into around 1 MB.
- [NIST FIPS 204 — Module-Lattice-Based Digital Signature Standard (13 August 2024)](https://csrc.nist.gov/pubs/fips/204/final). ML-DSA signature standard.
