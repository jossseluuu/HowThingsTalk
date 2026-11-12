---
title: "What can your internet provider see of what you do, even when the site uses HTTPS?"
section: "11_PrivacyAndFootprint"
language: "en"
status: "published"
order: 2
depth: "intermediate"
summary: "HTTPS encrypts the content: the page path, forms, cookies and whatever you download. It does not hide who you talk to, or how much and when. IP addresses, classic DNS queries and the site name in the TLS greeting travel in the clear, and packet sizes and timing also say things. Encrypted DNS and Encrypted Client Hello cover part of that, but not all of it."
prerequisites:
  - "How TCP delivers data in order (article 04·01)"
  - "How two devices agree on a secret key (article 10·01)"
translation: "../../ES/11_PrivacidadYHuella/02-QueVeTuOperadorAunqueUsesHTTPS.md"
---

# What can your internet provider see of what you do, even when the site uses HTTPS?

## Question and framing

The padlock in the browser means the connection is encrypted with TLS. Many people read it as "nobody can see what I do". The real question is narrower: which parts of a website visit are still visible to someone in the middle of the path, such as your internet provider or whoever runs a public WiFi network?

This article separates what HTTPS protects from what it leaves exposed. It walks through the four main traces: IP addresses, the DNS query, the site name in the TLS greeting, and the size and timing of the traffic. It also explains what encrypted DNS and Encrypted Client Hello (ECH) change. It is not a guide to configuring VPNs or browsers, and it does not compare services. Traffic analysis techniques are described at the level of concepts, to help you understand your own exposure.

The depth is intermediate. It helps to know what a packet, an IP address and a port are ([article 04·01](../04_DataTransfer/01-HowTCPDeliversDataInOrder.md)). It also helps to know that TLS starts with a key exchange ([article 10·01](../10_SecurityAndCryptography/01-HowTwoDevicesAgreeOnASecretKey.md)).

## Short answer

HTTPS hides what you say, but not who you say it to or how much you talk: the server's IP, the classic DNS query, the site name in the TLS greeting and the pattern of sizes and timing all remain visible to anyone on the path.

### 60-second summary

When you visit `https://example.org/health/appointment?id=7`, everything after the site name travels encrypted. That includes the path `/health/appointment`, the parameter `id=7`, cookies, forms and the page content. Your provider cannot read any of it.

But to get there, the phone does things that are visible:

1. **It asks for the name.** A classic DNS query asks "what is the IP of `example.org`?" without encryption.
2. **It opens the connection.** Every packet carries the source and destination IP in the clear, because routers need them to forward it.
3. **It greets the server.** The first TLS message carries the site name in the clear, in a field called SNI. The server needs it to know which certificate to present.
4. **It transfers data.** Encryption does not hide how many bytes go back and forth, or when.

Encrypted DNS covers the first trace. ECH covers the third if the server supports it. The other two remain, with some nuances.

## Mental model

Think of a letter inside an opaque, well-sealed envelope. Nobody at the post office can read it. But the envelope carries the recipient's address, or it would never arrive. The postal worker also sees how heavy the envelope is, which day you send it and how many you send each week.

HTTPS is the opaque envelope. The destination IP is the address on the envelope. The size and rhythm of the traffic are the weight and the frequency.

The analogy breaks in two places. First, on the internet an IP address does not always identify one recipient: many different sites share the same IP at the same provider. That is why the protocol adds a second label with the site name, the SNI, and that label was sent in the clear by design. Second, a letter does not need to ask a switchboard for the address first, with the switchboard writing everything down. On the web, the DNS query does exactly that.

## Technical mechanism

### 1. What happens in the system

A typical HTTPS website visit follows this sequence:

1. **DNS resolution.** The system asks a recursive DNS server for the IP of the name. Unencrypted, the query travels over UDP to port 53. According to RFC 9076 (Sections 1 and 5.1), anyone on the path sees the queried name and the IP of the device asking. The recursive resolver, often run by the provider itself, sees every query from its clients.
2. **Transport connection.** The phone opens a TCP connection, or QUIC over UDP, to the server's IP. The IP and TCP headers are in the clear. RFC 7624 (Section 3) notes that encryption protects little against inferences drawn from those unencrypted parts.
3. **TLS greeting.** The client sends a `ClientHello` message. It carries the `server_name` extension (SNI) with the site name. According to RFC 8744 (Section 2), that name travels in the clear. RFC 9849 (Section 1) describes it as the most sensitive item TLS 1.3 leaves unencrypted.
4. **Server response.** In TLS 1.3, everything after the `ServerHello` is encrypted, including the server certificate (RFC 8446, Section 1.2 and Section 4.4.2). In TLS 1.2, the certificate travelled in the clear and repeated the site name.
5. **Application data.** HTTP requests and responses go inside encrypted TLS records. This is where the path, parameters, headers, cookies and content live.

### 2. What each component does

**The IP address.** It identifies the machine each packet is delivered to. It cannot be encrypted end to end, because every router on the path reads it. But it does not always point to one specific site. RFC 8744 (Sections 1 and 2.2) explains that virtual hosting and content delivery networks (CDNs) serve many sites from few addresses. One IP at a large provider can correspond to thousands of domains.

**DNS.** It translates names into addresses. Classic DNS encrypts nothing. There are two main encrypted variants:

- **DNS over TLS (DoT, RFC 7858, May 2016):** uses TCP and port 853, reserved for this purpose.
- **DNS over HTTPS (DoH, RFC 8484, October 2018):** wraps each query in an HTTPS request to port 443. According to the RFC (Section 8.1), sharing a port and connection with other HTTPS traffic makes it harder to tell apart.

Encryption hides the queries from anyone on the path. Not from whoever answers them. RFC 9076 (Section 6.1) stresses that the resolver sees the queries regardless. Choosing a resolver shifts trust to whoever runs it; it does not remove it.

**SNI.** It exists because one IP can host many sites. When the server receives the `ClientHello`, it reads the name and picks the right certificate and configuration. RFC 8744 (Section 2.1) lists who else reads it: corporate firewalls, site-blocking systems, provider quality-of-service management and pervasive surveillance.

**Encrypted Client Hello (ECH, RFC 9849, March 2026).** It splits the greeting in two:

- a `ClientHelloInner`, the real greeting with the true name, encrypted with the provider's public key;
- a `ClientHelloOuter`, a visible outer envelope whose SNI carries a generic public name of the provider, the `public_name`.

The public key is published in DNS, in HTTPS records (RFC 9460), or comes preconfigured. Someone on the path sees the generic name and the provider's IP. The real site hides among all the sites that share that provider. So that ECH users do not stand out, clients without a configuration can send a fake ECH extension, called GREASE (RFC 9849, Section 6). ECH only works if the server supports it. The share of sites that support it is not measured in this article (unverified).

**QUIC and HTTP/3.** QUIC encrypts almost the whole packet. But its first packets, the *Initial* packets, are protected with keys derived from a visible connection identifier and a fixed public value (RFC 9001, Section 5.2). Anyone can derive them. The `ClientHello` is carried inside (RFC 9001, Section 4.3), so the SNI is as readable as in TLS over TCP (an inference from those two sections). ECH protects it here too.

**Size and timing.** TLS does not hide the length of what it transmits (RFC 8446, Section 1). It allows records to be padded to disguise it (Section 5.4), but padding is optional and the policy is left out of the standard. Appendix E.3 of the same RFC acknowledges that TLS is susceptible to length and timing analysis.

### 3. Key parameters

| Trace | Classic DNS + TLS 1.2 | Encrypted DNS + TLS 1.3 | Encrypted DNS + TLS 1.3 + ECH |
|---|---|---|---|
| Server IP | Visible | Visible | Visible |
| Name in DNS | Visible (UDP 53) | Hidden from the provider; seen by the resolver | Hidden from the provider; seen by the resolver |
| Name in SNI | Visible | Visible | Only the provider's public name |
| Name in the certificate | Visible | Encrypted | Encrypted |
| Path, parameters, cookies, content | Encrypted | Encrypted | Encrypted |
| Size, direction and timing | Visible | Visible | Visible |

Some reference figures on what size and timing reveal:

- **HTTPS without padding, pages within one site.** Miller and colleagues collected 463,125 page loads from 10 well-known websites between December 2013 and January 2014. Using only the sizes of packet bursts, they identified the specific page visited within a site with 90% accuracy, over 500 pages per site. A tailored padding scheme brought accuracy down to 25% at the cost of 9% more traffic.
- **Real scale.** Panchenko and colleagues tested methods of this kind on more than 300,000 pages, over Tor traffic. They concluded that no method, including their own, scaled to a realistic setting with a huge background of possible pages. Recognising an arbitrary page among all of them was like finding a needle in a haystack.

The two figures measure different scenarios. The first assumes the site is already known and only the page has to be guessed. The second tries to guess among any page on the internet.

### 4. Diagram or flow

```text
 Phone                     Provider / WiFi                    Server
   │                       (whoever is in the middle)             │
   │── Classic DNS: "IP of example.org?" ──► VISIBLE: the name    │
   │   (DoT/DoH: encrypted → only sees queries to a resolver)     │
   │                                                              │
   │── IP packet: source 10.0.0.5 → destination 203.0.113.7 ─────►│
   │                         VISIBLE: both IPs, ports, time       │
   │                                                              │
   │── ClientHello with SNI = "example.org" ─────────────────────►│
   │                         VISIBLE (with ECH: "cdn-provider.net")│
   │◄─ ServerHello ── [certificate and rest, encrypted in TLS 1.3]│
   │                                                              │
   │══ GET /health/appointment?id=7 + cookies ══ ENCRYPTED ══════►│
   │◄═ HTML page, images ═══════════════════════ ENCRYPTED ═══════│
   │                         VISIBLE: how many bytes, in which    │
   │                         bursts, when and in which direction  │
```

## Limits, exceptions, and real-world cases

**The IP alone is ambiguous, but not always.** For a site hosted on a large CDN, the IP is shared with many other domains. For a site on its own server, the IP identifies it almost completely. ECH does not change this: if the provider hosts only one site, the set to hide in has a size of one. RFC 9849 (Section 1) warns that ECH alone is not enough to protect the server's identity.

**ECH without encrypted DNS achieves little.** If the DNS query goes out in the clear, the name has already been seen before the greeting. RFC 9849 (Section 1) points this out and refers to DoH, DoT and DNS over QUIC.

**Encrypted DNS moves trust.** If the encrypted resolver belongs to your own provider, the provider still sees the queries; only a third party on the café WiFi no longer does. If it is another service, that service sees them instead.

**Other leaks outside the browser.** RFC 9849 (Section 10.7) mentions earlier unencrypted connections and certificate-status lookups as possible sources of information. Phone apps can use their own resolvers and connections, with rules different from the browser's (unverified in general).

**Traffic analysis depends heavily on the scenario.** Miller's figures come from a closed world of 10 sites, without padding, with data from 2013-2014. Panchenko's show that the problem gets much harder at real scale. Neither is a measure of what any particular provider does today. What the RFCs do establish is that TLS does not promise to hide lengths or timing.

**A VPN does not remove the problem, it moves it.** With a VPN, the provider sees a single encrypted connection to the VPN server, with its size and rhythm. The VPN provider takes the place of the internet provider: it sees destination IPs and, depending on its setup, DNS and SNI.

## Practical consequences

For anyone using the internet, HTTPS protects the most sensitive part of the content: passwords, messages, card numbers, the exact page and what you type into it. But it does not hide the list of sites you visit, or when. On a public WiFi network, the network owner can see that list just as your provider can.

Encrypted DNS and ECH shrink that list to "provider IPs and generic public names". They do not make it disappear. And there is always someone who sees more: the DNS resolver, the provider hosting the site, or the VPN.

For service designers, hosting a site behind a large provider with ECH enlarges the set each visit hides in. Padding sensitive responses to fixed sizes makes size analysis harder, at the cost of more traffic.

For network operators, filtering or prioritising by SNI stops working with ECH. What they will see is the provider's public name, not the site.

## Frequently asked questions

### Does the browser's incognito mode hide anything from the provider?

No. Incognito mode avoids storing history, cookies and data on the device itself. The traffic leaving for the network is the same, with the same IPs, DNS queries and SNI.

### If the site uses HTTPS, does my provider know which video I watch or which article I read?

It cannot read the path or the content. It does see the site, through the IP or the SNI, and the volume of data. Traffic analysis studies show that, under controlled conditions, the size pattern can give away specific pages of a known site. Under real conditions it is considerably harder.

### Is encrypting DNS the same as using ECH?

No. They are two different traces. Encrypted DNS hides the question "what IP does this name have?". ECH hides the name inside the TLS greeting. Both are needed for the name never to appear in the clear.

### What about the phone's MAC address?

The MAC address never leaves the local network: your internet provider does not see it. Its role in privacy is explained in [article 11·01](01-WhyPhonesRandomizeTheirMACAddress.md).

## Sources

- [RFC 8446 — The Transport Layer Security (TLS) Protocol Version 1.3 (IETF, August 2018)](https://www.rfc-editor.org/rfc/rfc8446.html). Encryption of everything after the `ServerHello` (Section 1.2), encrypted certificate (4.4.2), TLS does not hide lengths (1), optional record padding (5.4) and susceptibility to length and timing analysis (Appendix E.3).
- [RFC 8744 — Issues and Requirements for Server Name Identification (SNI) Encryption in TLS (IETF, July 2020)](https://www.rfc-editor.org/rfc/rfc8744.html). Cleartext SNI, shared hosting and CDNs (Sections 1 and 2.2) and third-party uses of SNI (2.1).
- [RFC 9849 — TLS Encrypted Client Hello (IETF, March 2026)](https://www.rfc-editor.org/rfc/rfc9849.html). Inner and outer greetings and `public_name` (Sections 3, 5 and 6.1), publishing the configuration in DNS (3.2), GREASE (6), limits of ECH and the need for encrypted DNS (1) and other leaks (10.7).
- [RFC 9076 — DNS Privacy Considerations (IETF, July 2021)](https://www.rfc-editor.org/rfc/rfc9076.html). What a cleartext DNS query reveals and to whom (Sections 1 and 5.1) and the resolver as the point that sees everything (6.1).
- [RFC 7858 — Specification for DNS over Transport Layer Security (TLS) (IETF, May 2016)](https://www.rfc-editor.org/rfc/rfc7858.html). DNS over TLS on port 853 (Section 3.1).
- [RFC 8484 — DNS Queries over HTTPS (DoH) (IETF, October 2018)](https://www.rfc-editor.org/rfc/rfc8484.html). DNS queries inside HTTPS and the effect of sharing port 443 (Section 8.1).
- [RFC 9460 — Service Binding and Parameter Specification via the DNS (SVCB and HTTPS Resource Records) (IETF, November 2023)](https://www.rfc-editor.org/rfc/rfc9460.html). HTTPS records that carry the ECH configuration.
- [RFC 9001 — Using TLS to Secure QUIC (IETF, May 2021)](https://www.rfc-editor.org/rfc/rfc9001.html). `ClientHello` in *Initial* packets (Section 4.3) and keys derived from public data (5.2).
- [RFC 7624 — Confidentiality in the Face of Pervasive Surveillance: A Threat Model and Problem Statement (IAB, August 2015)](https://www.rfc-editor.org/rfc/rfc7624.html). Inference from unencrypted headers and flow sizes (Sections 2 and 3).
- [Miller, B., Huang, L., Joseph, A. D., Tygar, J. D. — I Know Why You Went to the Clinic: Risks and Realization of HTTPS Traffic Analysis. Privacy Enhancing Technologies Symposium (PETS), 2014](https://petsymposium.org/2014/papers/Miller.pdf). 463,125 page loads from 10 sites, 90% per-page accuracy within a site and a defence that lowers it to 25% with 9% more traffic.
- [Panchenko, A. et al. — Website Fingerprinting at Internet Scale. NDSS Symposium, 2016](https://www.ndss-symposium.org/wp-content/uploads/2017/09/website-fingerprinting-internet-scale.pdf). More than 300,000 pages over Tor and the conclusion that no evaluated method scales to a realistic setting.
