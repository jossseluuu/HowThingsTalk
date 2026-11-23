---
title: "How does an authenticator app produce a valid code without connecting to the internet?"
section: "10_SecurityAndCryptography"
language: "en"
status: "published"
order: 3
depth: "intermediate"
summary: "When you turn on two-step verification, the server and the phone store the same secret. Every 30 seconds, each side computes an HMAC of the secret and the current time on its own, and cuts it down to 6 digits. No network is needed: the two clocks only have to agree. The server accepts the code if it matches its own result."
prerequisites:
  - "What a hash function is, at a general level"
  - "What a shared secret key is (article 10/01, optional)"
translation: "../../ES/10_SeguridadYCriptografia/03-ComoGeneraLaAppDeAutenticacionSusCodigos.md"
---

# How does an authenticator app produce a valid code without connecting to the internet?

## Question and framing

You turn on two-step verification on a website. You scan a QR code with an authenticator app. From then on, the app shows a 6-digit number that changes every 30 seconds. It works in airplane mode, and the website accepts it. This article explains how that is possible: what the phone and the server share, what calculation both perform, and why the results match without the two ever talking.

It covers the TOTP algorithm (RFC 6238) and what it is built on: HOTP (RFC 4226) and HMAC (RFC 2104). It explains how the secret is handed over when you scan the QR code and what the server checks. It does not cover codes sent by SMS, which work differently, or how to set up any particular app or website. It does not describe how to attack specific accounts.

The depth is intermediate. It helps to know what a hash function is. The [article on Diffie-Hellman](01-HowTwoDevicesAgreeOnASecretKey.md) explains another way of sharing a secret, but it is not needed to follow this one.

## Short answer

The phone and the server each keep a copy of the same secret and look at the same clock; every 30 seconds, both apply an identical cryptographic calculation to the secret and the interval number, and the result, cut down to 6 digits, is the code.

### 60-second summary

The QR code you scan contains a random secret. The server keeps another copy. This is the only time the secret travels.

To produce a code, the app counts how many 30-second intervals have passed since 1 January 1970. That number is the same on any device whose clock is right. It then computes an HMAC: a kind of fingerprint that mixes the secret with that number. Without the secret, the fingerprint cannot be predicted, even by someone who has seen many earlier codes.

The fingerprint is 20 bytes long. The app picks 4 of them by a fixed rule and turns them into a 6-digit number.

The server does the same calculation with its copy of the secret. If the result matches, the code is valid. It usually also accepts the previous interval, to allow for typing delay or a slightly drifting clock.

## Mental model

Picture two spies who part ways, each with a copy of the same codebook. Each page corresponds to a time slot. To identify themselves, one says the word on the page for the current hour. The other opens their book at the same page and checks. They never need to call each other: the book and a watch are enough.

The analogy breaks down in two places. First, there is no book with pages written in advance. Each "page" is computed on the spot from the secret, so billions of time slots fit without taking up any space. Second, in a real codebook, seeing some pages tells you nothing about the others. The same holds here, but not by chance: it is guaranteed by the HMAC function, which does not let anyone work out the secret from its outputs.

## Technical mechanism

### 1. What happens in the system

**Enrolment (once).**

1. The server generates a random secret `K` for your account and stores it.
2. It shows it to you inside a QR code. The content is usually an `otpauth://` address, a format first formalized by Google and recognized by almost every app. It carries the secret in Base32 and, optionally, the algorithm, the number of digits and the interval. It looks like `otpauth://totp/Issuer:account?secret=…&issuer=Issuer`.
3. The app stores `K`. To confirm enrolment, the website usually asks you for a first code.

**Every sign-in.**

1. The app reads the phone's time in Unix seconds.
2. It computes the interval number `T` (RFC 6238, section 4.2):

   `T = floor((Unix time − T0) / X)`, with `T0 = 0` and `X = 30 s` by default.

3. It computes `HOTP(K, T)`: an HMAC of the secret with `T` as the message, then a reduction to decimal digits (RFC 4226, section 5).
4. You type the code. The server repeats steps 1 to 3 with its copy of `K` and compares.

### 2. What each component does

**The shared secret `K`.** It is the only piece that is not public. RFC 4226 requires at least 128 bits and recommends 160 (section 4, requirement R6). RFC 6238 advises making it as long as the HMAC output: 160 bits with SHA-1 (section 5.1). NIST SP 800-63B-4 requires a minimum security strength of 112 bits (section 3.1.4). Each Base32 character carries 5 bits, so a 160-bit secret takes 32 characters.

**The time counter `T`.** It is an 8-byte integer (RFC 4226, section 5.1). With `X = 30 s`, it equals 1 at 59 seconds after 1 January 1970 and 2 at 60 seconds (RFC 6238, section 4.2). The standard requires support for values larger than 32 bits, so the calculation keeps working after 2038.

**HMAC.** It is a message authentication code built from a hash function (RFC 2104, section 2). It runs the key and the message through the hash function twice, mixed with two fixed constants. The result depends on every bit of the key and the message. Without the key, nobody can compute it. Nobody can work out the key from earlier results either. TOTP uses HMAC-SHA-1 by default and allows HMAC-SHA-256 and HMAC-SHA-512 (RFC 6238, section 1.2).

**Dynamic truncation.** HMAC-SHA-1 produces 20 bytes. They have to become a few digits (RFC 4226, section 5.3):

1. Take the low 4 bits of the last byte. They give an offset from 0 to 15.
2. Read 4 bytes starting at that position.
3. Clear the top bit. This leaves a 31-bit number and removes any doubt between signed and unsigned arithmetic.
4. Take the remainder of dividing by 10^digits.

The standard requires at least 6 digits and allows 7 or 8.

**The verifier.** Besides comparing, it has three duties. It must reject a code that has already been used: RFC 6238 forbids accepting a second attempt with the same code after a successful validation (section 5.2). It must limit failed attempts (RFC 4226, section 7.3). And it must tolerate some clock drift without opening the window too wide.

### 3. Key parameters

- **Interval `X`.** 30 s by default and recommended (RFC 6238, sections 4.1 and 5.2). The `otpauth://` format allows 15, 30 or 60 s, according to Yubico's documentation.
- **Digits.** 6 at minimum, 7 or 8 optional (RFC 4226, section 5.3). NIST allows truncation down to 6 decimal digits.
- **Acceptance window.** RFC 6238 recommends allowing at most one interval backwards for network delay (section 5.2). For clock drift, section 6 allows several intervals. With two backwards, the maximum tolerated drift is around 89 s.
- **Code lifetime according to NIST.** There is no fixed figure. It must cover the expected clock drift in both directions, plus network delay and typing time (SP 800-63B-4, section 3.1.4).
- **Attempt limit.** NIST sets a maximum of 100 consecutive failures per authenticator and account before disabling it. It is a ceiling, and a lower one may be set (section 3.2.2).
- **Probability of a blind guess.** RFC 4226 estimates it as `s · v / 10^digits`, where `s` is the number of codes accepted at once and `v` the number of attempts (section 6). With 6 digits, 3 accepted codes and 100 attempts, it comes to 3 × 10⁻⁴: 0.03%. The calculation is ours, applying that formula to those values.

### 4. Diagram or flow

Example using the standards' test values: the secret is the ASCII text `12345678901234567890` (20 bytes) and the Unix time is 59 s. The intermediate bytes were recomputed from those inputs and match the codes published in RFC 6238 (Appendix B) and RFC 4226 (Appendix D).

```text
ENROLMENT (once)
Server --- QR: otpauth://totp/...?secret=<K in Base32> ---> App
  stores K                                                   stores K

EVERY CODE (on both sides, without communicating)
Unix time = 59 s
T = floor(59 / 30) = 1   ->  00 00 00 00 00 00 00 01   (8 bytes)

HMAC-SHA-1(K, T) =
  75 a4 8a 19 d4 cb e1 00 64 4e 8a [c1 39 7e ea] 74 7a 2d 33 ab
  byte:  0 ...                   11  12 13 14            ...  19

last byte 0xab -> low 4 bits = 0xb = 11 -> offset 11
bytes 11-14     = 0xc1397eea
top bit cleared = 0x41397eea = 1,094,287,082
mod 10^6        = 287082        (6 digits)
mod 10^8        = 94287082      (8 digits, RFC 6238 value)

App shows 287082 ---- you type it ----> Server
                                        computes the same with its K
                                        for T = 1 (and maybe T = 0)
                                        match and not used before?  -> access
```

## Limits, exceptions, and real-world cases

**Everything depends on the clock.** If the phone's time drifts by several intervals, the codes stop matching even though the secret is correct. RFC 6238 suggests that the server record the drift it detects for each device and compensate for it next time (section 6). If the drift grows too large, the standard calls for another form of authentication.

**It does not resist phishing.** The code does not know which website it is meant for. If a fake page asks for it, the page can forward it instantly to the real site and sign in within the 30-second window. NIST says so explicitly: authenticators whose output the person types in are not considered phishing-resistant, because that output is not bound to the specific session (SP 800-63B-4, section 3.2.5). Security keys and passkeys are, because they sign a challenge that includes the site's name.

**The server keeps the same secret.** Unlike a properly stored password, which is kept as an irreversible digest, the server needs `K` in the clear to compute the code. Anyone who steals the database of secrets can generate valid codes. That is why the secret must be protected like any other key.

**The QR code is the sensitive moment.** The secret travels only once, but it travels whole. Anyone who sees or photographs that QR code has a copy of the generator. The same goes for app backups: if the secrets are stored in the cloud, protection now depends on that account.

**Losing the phone means losing the generator.** Without the secret there is no way to recompute the codes. That is why websites offer single-use recovery codes when you turn on the second factor.

**It is not the same as an SMS code.** With SMS, the server generates the code and sends it to you over the telephone network. NIST classifies the use of the telephone network for this purpose as a "restricted" authenticator (SP 800-63B-4, section 3.2.9). With TOTP, the code does not travel until you type it.

**The QR format is not an IETF standard.** The `otpauth://` scheme is a de facto format. There is an individual IETF draft (version 03, August 2026) that tries to describe it, but it has no standards status.

## Practical consequences

For the user, the app works without coverage because it does not need to send anything: it only needs the secret and the time. If one day every code fails, the first thing to check is that the phone sets its time automatically. And the code should be treated as what it is, a password that expires in seconds: do not read it out over the phone or type it into a page you reached from a link.

For someone designing a service, three pieces are critical: generating `K` with enough randomness (128 bits or more), storing it securely on the server, and enforcing the single-use rule with an attempt limit. The acceptance window is a trade-off. Each extra interval makes life easier for a user with a drifting clock, but adds one more valid code for an attacker to hit.

## Frequently asked questions

### Is TOTP the same as HOTP?

Almost. HOTP (RFC 4226) uses a counter that advances each time you ask for a code. TOTP (RFC 6238) replaces that counter with the number of time intervals. With HOTP, the two sides can lose count if you press the button without using the code. With TOTP, they fall out of sync if the clock drifts.

### Can the secret be worked out from many codes?

Not in practice. Each code comes from an HMAC, and the security of HMAC is precisely that its outputs do not reveal the key. On top of that, the code keeps only about 20 of the HMAC's 160 bits. RFC 4226 concludes that the best possible attack is guessing codes blindly (section 6).

### Why does the code last 30 seconds and not 5?

For usability. You have to read the code, switch apps and type it, and clocks are never perfectly synchronized. RFC 6238 recommends 30 s as a balance between security and usability (section 5.2).

### Is it the same as a passkey?

No. A passkey uses a public/private key pair. The server stores only the public key, and the device signs a challenge bound to the site's name. With TOTP, both sides store the same secret, and the code works on any site that receives it while it is valid.

## Sources

- [RFC 6238 — TOTP: Time-Based One-Time Password Algorithm (IETF, May 2011)](https://www.rfc-editor.org/rfc/rfc6238.html). Use of HMAC-SHA-256 and SHA-512 (section 1.2), parameters `T0` and `X` (4.1), computing `T` and support beyond 2038 (4.2), key length (5.1), recommended interval, window and single use (5.2), resynchronization (6) and test values (Appendix B).
- [RFC 6238 errata, no. 2866 (verified November 2011)](https://www.rfc-editor.org/errata/rfc6238). Test secrets of 20, 32 and 64 bytes depending on the hash function.
- [RFC 4226 — HOTP: An HMAC-Based One-Time Password Algorithm (IETF, December 2005)](https://www.rfc-editor.org/rfc/rfc4226). Secret length (section 4, R6), 8-byte counter (5.1), dynamic truncation and number of digits (5.3), worked example (5.4), security analysis (6), attempt throttling (7.3) and test values (Appendix D).
- [RFC 2104 — HMAC: Keyed-Hashing for Message Authentication (IETF, February 1997)](https://www.rfc-editor.org/rfc/rfc2104.html). Definition of HMAC (section 2).
- [NIST SP 800-63B-4 — Digital Identity Guidelines: Authentication and Authenticator Management (July 2025)](https://pages.nist.gov/800-63-4/sp800-63b/authenticators). OTP authenticators (sections 3.1.4 and 3.1.5), attempt limit (3.2.2), phishing resistance (3.2.5) and restricted authenticators (3.2.9). [Publication record](https://csrc.nist.gov/pubs/sp/800/63/B/4/final).
- [Yubico — URI string format (SDK documentation)](https://docs.yubico.com/yesdk/users-manual/application-oath/uri-string-format.html). Structure of `otpauth://` addresses, parameters and allowed values.
- [IETF — draft-linuxgemini-otpauth-uri, version 03 (individual draft, August 2026)](https://datatracker.ietf.org/doc/draft-linuxgemini-otpauth-uri/). Attempt to describe the `otpauth://` scheme; no standards status.
