---
title: "Why does your phone use a different MAC address on each WiFi network?"
section: "11_PrivacyAndFootprint"
language: "en"
status: "published"
order: 1
depth: "intermediate"
summary: "The factory MAC address is a fixed number that the phone sends unencrypted in every WiFi frame, so it can be used to recognise the device anywhere. Current systems replace it with random addresses: one for scanning and a separate one for each network. That stops the easiest form of tracking, but not all the others."
prerequisites:
  - "What a WiFi network and an access point are, at a general level"
  - "Layers of a protocol stack (glossary)"
translation: "../../ES/11_PrivacidadYHuella/01-PorQueElMovilCambiaSuDireccionMAC.md"
---

# Why does your phone use a different MAC address on each WiFi network?

## Question and framing

Open the WiFi settings on a recent phone and you will find something like "private address" or "randomised MAC". The address the phone uses on your home network is not the one it uses at work. And neither is the one burned in at the factory. The question is why operating systems do this and what it actually achieves.

This article explains what a MAC address is, why the factory one can be used to follow a device, and how iOS and Android replace it. It also explains which traces remain even when the address changes. It is not a settings guide. Nor does it describe techniques for tracking other people's devices: weak points are named at a conceptual level, to understand your own exposure. It does not cover Bluetooth, which has its own random address mechanism.

The depth is intermediate. It is enough to know what a WiFi network is and to have a rough idea of protocol stack layers, defined in the [glossary](../../../guides/TaxonomyAndGlossary.md).

## Short answer

The factory MAC is a fixed identifier that travels unencrypted in every WiFi frame; by replacing it with random addresses, different for scanning and different on each network, the phone stops being the same number everywhere.

### 60-second summary

Every WiFi card ships with a 48-bit address assigned by its manufacturer. That address sits in the header of every frame the device sends. WiFi encryption protects the content, but not that header. On top of that, a phone with WiFi on keeps asking "is there a network around here?", even when it is not connected. Every one of those questions carries its address.

With a fixed address, any receiver in a shop, a station or a stadium can note "this device passed by at 9:14". With several receivers, that becomes a route.

The fix is to make addresses up. The standard reserves one bit to mark an address as "locally administered", meaning not assigned by a manufacturer. The phone sets that bit to 1 and fills almost everything else at random. It uses a random address to scan for networks. When it connects, it uses a separate address of its own for that network, different on every network. So your home network and the café do not see the same number.

The change does not make the phone invisible. Other data in the frames, the content of the connection and signing in can still identify it.

## Mental model

Think of a coat with your name embroidered on the back. Wherever you go, anyone who looks can note that you were there. The factory MAC address is that embroidery.

Randomisation is like removing the embroidery and wearing a different sticker in each place. At the gym you always wear the same one, so they recognise you as a member. At work you wear another. When you are just walking down the street, you wear one that you swap often.

The analogy breaks at an important point. A new sticker does not change the way you walk, your clothes or your voice. WiFi is the same: the phone still sends a combination of capabilities, a way of scanning and traffic of its own. Those details can give it away even when the address changes. And as soon as you sign in to a network, the sticker no longer matters: the network already knows who you are.

## Technical mechanism

### 1. What happens in the system

A MAC address identifies a network interface at the link layer. In WiFi, every frame carries a source and a destination address. Access points need them to know whom to answer.

A phone sends its address at three moments:

1. **Scanning.** To find known networks, the phone sends *probe request* frames. RFC 9724 notes that these frames carry the source MAC address. They are sent even when the phone is not connected to anything.
2. **Association.** When it joins a network, the phone introduces itself to the access point with an address and negotiates the connection.
3. **Active connection.** Every data frame carries the address. The content is encrypted with WPA2 or WPA3, but the header with the addresses is not.

Randomisation acts at all three moments, with different rules for each.

### 2. What each component does

**The factory address.** It has 48 bits. According to RFC 9724 (section 2.1), the first three octets are the OUI, which identifies the organisation that issued it. The manufacturer assigns the other three to each card. The result is globally unique and never changes.

**The two control bits.** The first octet holds two special bits (RFC 9724, figure 1):

- the I/G bit, the least significant one, says whether the address belongs to a single device (0) or a group (1);
- the U/L bit, the next one, says whether it is universal, assigned by a manufacturer (0), or locally administered (1).

A phone's random address has U/L = 1 and I/G = 0. Android's documentation confirms it: those two bits are fixed and the other 46 are random. One practical consequence (calculated): the second hexadecimal character of a random address is always 2, 6, A or E.

**The address space.** With 46 free bits there are 2⁴⁶ ≈ 7.0 × 10¹³ possible addresses (calculated). Two phones on the same network almost never collide. RFC 9724 (section 2.3) reports experiments that found duplicates at a negligible rate.

**Scanning addresses.** Systems use a random address for scans when they are not connected. According to Apple's security guide, the address changes on disconnection. Apple also randomises frame sequence numbers. It does so whenever the address changes and on every new scan while unconnected. This exists since iOS 16.1 and macOS 13.1. Before that, a continuous counter could link two consecutive random addresses.

**The per-network address.** When it connects, the phone does not use the scanning address. It uses an address tied to that specific network. The network always sees the same number, which keeps reconnection and IP reservations working. Another network sees a different number.

**The operating system.** It decides when each address is generated and how long it lasts. iOS and Android policies differ, as detailed below.

### 3. Key parameters

**Android (according to the Android Open Source Project documentation).**

- Since Android 10, connections use a *persistent* random address by default.
- It is generated from the network profile's data: the SSID, the security type or, for Passpoint networks, the FQDN.
- It stays the same until a factory reset. Forgetting the network and adding it again does not change it, because the profile data is the same.
- Since Android 12 there is a *non-persistent* variant for some networks. It applies if a network-suggestion app asks for it, or on open networks without a captive portal if the manufacturer enables an option that is off by default.
- The non-persistent variant only changes at the start of a new connection. It changes if the DHCP lease has expired and more than 4 hours have passed since the last disconnection, or if the address is more than 24 hours old.

**Apple (according to support article 102509 and the platform security guide).**

- Per-network private addresses exist since iOS 14, iPadOS 14 and watchOS 7.
- Since iOS 18, iPadOS 18 and visionOS 2 there are three options: off, fixed and rotating. They also exist in watchOS 11 and macOS Sequoia 15.
- **Fixed:** does not change over time. It is the default for new networks with WPA2 or stronger security.
- **Rotating:** changes every 2 weeks. It is the default for new networks with weak or no security.
- On those versions, forgetting a network also deletes its private address, unless that network was forgotten less than 24 hours earlier. On earlier versions the window was 2 weeks.
- Erasing the device or resetting network settings gives new addresses on the next connection.

**Other systems.** RFC 9724 summarises tests carried out in 2021. Windows 10, Linux, Android 10 and iOS 14 all used per-network addresses and random addresses for scanning. Windows 10 was the only one that also offered daily rotation. Only Android 10 and iOS 14 had scan randomisation enabled by default. The RFC states that it detected no significant changes up to its publication in March 2025.

**The IEEE recommendation.** According to RFC 9724, IEEE 802E-2020 recommends temporary identifiers unless there is a strong reason for them to be permanent.

### 4. Diagram or flow

```text
 Factory address (48 bits; not used while randomisation is on)
   manufacturer OUI (24 bits) │ card number (24 bits)

 Random address
   first octet: xxxxxx10   ← U/L = 1 (local), I/G = 0 (single device)
   rest: 46 random bits    → 2⁴⁶ ≈ 7.0 × 10¹³ combinations

 ┌───────────────────────────┐
 │ Unconnected: scanning     │── probe request ──► address A₁, A₂, A₃…
 └───────────────────────────┘                     (change often)
              │
              ▼ joins "Home"
 ┌───────────────────────────┐
 │ Network "Home"            │── data frames ────► address H
 └───────────────────────────┘   (the same every time it returns to "Home")
              │
              ▼ later, at "Office"
 ┌───────────────────────────┐
 │ Network "Office"          │── data frames ────► address O ≠ H
 └───────────────────────────┘

 Result: "Home" and "Office" share no link-layer identifier,
 and anyone who only listens to scans sees addresses that do not repeat.
```

## Limits, exceptions, and real-world cases

**The address is not the only trace.** Martin and colleagues analysed about 66 million *probe requests* captured between January 2015 and December 2016. They contained about 2.6 million distinct addresses. Around 53% of those addresses were random. But the authors warn that the figure counts addresses, not devices: a single randomising phone generates many. In their study they identified several ways to link random addresses to the same device:

- the combination of capabilities each model announces in its scans, which works as a fingerprint;
- legacy fields that were derived from the factory address;
- implementation flaws that made the device send its real address in certain frames.

Using passive methods, the study states that randomisation failed for about 96% of the Android phones analysed. These are 2015–2016 figures. Current versions have fixed many of those flaws, as Apple's sequence number randomisation shows. There is no equivalent public, up-to-date figure for today's systems (unverified).

**Other higher-layer identifiers.** RFC 9724 (sections 2.3 and 5) points out that the DHCP client identifier can link sessions even when the MAC address changes. That is why RFC 7844 asks for the link-layer address, the IP address and the DHCP identifier to change together. If one stays fixed, the others lose their value.

**Within a network, the address is stable on purpose.** With Android's persistent address or Apple's fixed one, your home access point always sees the same number. That is desirable: the network recognises the device. But it also means that particular network can follow its visits. A random address protects you between networks, not from the network you connect to.

**Signing in cancels link-layer anonymity.** If a network asks for a username and password, a certificate or a form with your name, it already knows who you are. RFC 9724 (section 6.5) notes that an address change forces a new 802.1X session and new IP addresses. It does not hide the account identity.

**Networks that relied on a fixed address.** Many services used the MAC as an identifier: access filters, IP reservations, per-device controls or portals that remember a customer. RFC 9724 explains that default randomisation affected those services. In 2020 the IEEE opened two projects to address this: 802.11bh, on the effects on networks and their services, and 802.11bi, on new privacy mechanisms at the link layer. According to the RFC, both were still under way in 2024.

**Scanning while unconnected.** A phone that is not connected to any network keeps scanning, for example to find known networks or for location services. Apple states that those scans use random addresses. Not every manufacturer documents this behaviour (unverified for Android as a whole).

## Practical consequences

For the person using the phone, randomisation removes the cheapest kind of tracking: noting a fixed address in many places. It does not remove tracking by a network you sign in to, or tracking based on content, account or app.

A network seeing you with the same address every time is normal and not a bug. It is the trade-off between privacy and a network that works well.

If a router or a parental control "loses" a device, the reason may be that its address changed. That happens, for example, when you forget the network on iOS or with the rotating address on open networks.

For network designers, the MAC no longer works as a device identity. Authentication has to rely on credentials, certificates or accounts. RFC 9724 (section 9) asks future specifications to give operators ways to authenticate devices even when addresses change.

For device designers, changing the address is not enough. Every other identifier has to change at the same time: sequence numbers, DHCP identifiers and any field derived from the factory address.

## Frequently asked questions

### Is a MAC address the same as an IP address?

No. The MAC identifies the interface on the local link, between the phone and the access point. It does not go past the router. The IP identifies the device on the network and is what internet servers see. Randomising the MAC does not change the public IP a website sees.

### Doesn't WPA2 or WPA3 encryption already hide the address?

No. It encrypts the content of data frames, but the addresses in the header travel in the clear. The access point needs them to deliver each frame. Scans are also sent before any encryption is in place.

### Why doesn't the address change every few minutes within a network too?

Because every change counts as a new device. According to RFC 9724, it causes a disconnection, a new authentication session and a new IP address. Open connections would drop. That is why default policies keep one address per network and reserve frequent changes for scanning or for open networks.

### Is this the same as Bluetooth random addresses?

They pursue the same goal, but it is a different mechanism, defined by the Bluetooth SIG with its own rules. It is outside the scope of this article.

## Sources

- [RFC 9724 — State of Affairs for Randomized and Changing Media Access Control (MAC) Addresses (IETF, March 2025)](https://www.rfc-editor.org/rfc/rfc9724.html). Address structure and I/G and U/L bits (section 2.1, figure 1), address in *probe requests* (2.2), duplicates and correlation through the DHCP identifier (2.3), IEEE 802E-2020, 802.11bh and 802.11bi (3), RFC 7844 (5), effects of an address change (6.5), 2021 tests across several systems (7, tables 1 and 2) and future requirements (9).
- [Android Open Source Project — MAC randomization behavior](https://source.android.com/docs/core/connect/wifi-mac-randomization-behavior). Fixed bits and 46 random bits, persistent randomisation since Android 10, generation from the network profile, non-persistent variant since Android 12 and its 4-hour and 24-hour rules.
- [Apple — Use private Wi-Fi addresses on Apple devices (article 102509)](https://support.apple.com/en-us/102509). Versions that support private addresses, off, fixed and rotating options, rotation every 2 weeks, defaults based on network security and the effect of forgetting a network.
- [Apple Platform Security — Privacy features when connecting to wireless networks](https://support.apple.com/guide/security/privacy-features-connecting-wireless-networks-secb9cb3140c/web). Random address for unconnected scans, change on disconnection and sequence number randomisation since iOS 16.1 and macOS 13.1.
- [Martin, J. et al. — A Study of MAC Address Randomization in Mobile Devices and When it Fails. Proceedings on Privacy Enhancing Technologies, 2017 (4), 365-383](https://petsymposium.org/popets/2017/popets-2017-0054.pdf). Dataset of about 66 million *probe requests* and 2.6 million addresses (2015-2016), share of random addresses and the ways in which randomisation failed.
