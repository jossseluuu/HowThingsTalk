---
title: "How can a website recognize your browser without using cookies?"
section: "11_PrivacyAndFootprint"
language: "en"
status: "published"
order: 4
depth: "intermediate"
summary: "Every browser reveals dozens of ordinary details when it connects: version, language, time zone, screen size, fonts, or how its graphics card draws a piece of text. None of them identifies anyone on its own, but together they form a combination that is often rare or unique. A website can compute that fingerprint on every visit and recognize the browser without storing anything on it. That is why clearing cookies does not remove it."
prerequisites:
  - "What your ISP sees despite HTTPS (article 11·02)"
  - "Why phones randomize their MAC address (article 11·01)"
translation: "../../ES/11_PrivacidadYHuella/04-ComoTeReconoceUnaWebSinCookies.md"
---

# How can a website recognize your browser without using cookies?

## Question and framing

A cookie is a tag a website leaves in your browser so it can recognize it on the next visit. Delete it and the site loses that trail. But there is another way to recognize a browser, one that stores nothing on it. The question is concrete: how can a website tell you are the same visitor without leaving any mark on your device?

This article explains browser fingerprinting: which data make up the fingerprint, how to measure how much it identifies, and why some defenses work while others make things worse. It is not a guide to building a tracking system, and it does not compare browsers. It also leaves out tracking by IP address, which is covered in [article 11·02](../11_PrivacyAndFootprint/02-WhatYourISPSeesDespiteHTTPS.md).

The depth is intermediate. It helps to know what travels in a web request ([article 11·02](../11_PrivacyAndFootprint/02-WhatYourISPSeesDespiteHTTPS.md)) and the idea of a stable identifier from [article 11·01](../11_PrivacyAndFootprint/01-WhyPhonesRandomizeTheirMACAddress.md). The only mathematical tool is the bit, explained along the way.

## Short answer

Your browser has no serial number, but the sum of its ordinary traits works as if it did: a combination it shares with very few other browsers, which can be measured again on every visit and which does not go away when you clear cookies.

### 60-second summary

To display a page properly, the browser tells the site things about itself. Some it sends with every request: name and version, operating system and preferred languages. Others can be queried by the page's JavaScript: screen size, time zone, available fonts or graphics card.

Each trait on its own is shared by millions of people. Together, things change. In the first large-scale study, in 2010, 83.6% of 470,161 browsers had a combination that no other browser in the sample shared.

The most powerful technique is to ask the browser to draw some text or a shape off-screen and read back the resulting pixels. Different systems, fonts and graphics cards produce tiny differences, invisible to the eye but measurable.

The site boils everything down to a code and compares it with the ones it has already seen. It stores nothing on your device, so clearing cookies does not help. What does shrink the fingerprint is many browsers giving exactly the same answers.

## Mental model

Think of a nightclub bouncer with no guest list. He does not ask for ID, but he notices details: green coat, red trainers, round glasses, a backpack with a badge on it. None of these is unusual on its own. The combination is. The following week he recognizes you without having written you down anywhere.

If you change your coat, he may hesitate. But if everything else matches, he will probably get it right. And if every customer wore the same uniform, he would have nothing to go on.

The analogy breaks down in two places. First, the bouncer only sees what you have on show. A website can ask your browser active questions, such as asking it to draw something, and the result depends on internal parts nobody sees, like the graphics card. Second, the bouncer compares from memory and easily gets it wrong. A website compares exact codes against millions of stored visits, and it can measure precisely how rare each trait is.

## Technical mechanism

### 1. What happens in the system

The W3C, the body that standardizes the web, distinguishes three ways to recognize a browser from its traits (note *Mitigating Browser Fingerprinting in Web Specifications*, section 3):

1. **Passive fingerprinting.** It uses only what arrives in the request itself, without running code: HTTP headers such as `User-Agent` or `Accept-Language`, the order of those headers, and network data such as the IP address.
2. **Active fingerprinting.** The page runs JavaScript to ask more questions: window size, fonts, performance, or how a graphic is rendered. The W3C points out a key difference: this kind is potentially detectable on the device itself, because the code runs there.
3. **Cookie-like state.** The site stores something on the device through a channel other than cookies and retrieves it later. This is not fingerprinting in the strict sense, because it does leave a mark.

A typical visit follows this sequence:

1. **The request arrives.** The server records the headers and the IP address.
2. **The script runs.** It collects dozens of properties of the browser and the system.
3. **Something is drawn off-screen.** The script asks the browser to paint text or a 3D scene on an invisible canvas and reads the pixels.
4. **Everything is summarized.** All the values are combined into a short code with a hash function.
5. **The code is compared.** The server looks for that code, or a very similar one, among those it has already seen.

### 2. What each component does

**HTTP headers.** They travel with every request. The `User-Agent` header traditionally gave the browser with its exact version, the operating system and, on phones, sometimes the model. That is why Chrome trimmed it starting in 2022. According to Google's documentation, it now sends a reduced string like this one:

```text
Mozilla/5.0 (Linux; Android 10; K) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.0.0 Mobile Safari/537.36
```

The Android version, the model (`K`) and the browser's minor version are fixed values, the same for everyone. Detailed data are still available through *Client Hints*, but the server has to ask for them with the `Accept-CH` header. By default only three are sent: brand and major version, whether it is a mobile device, and the operating system name. The device model (`Sec-CH-UA-Model`) is a "high-entropy" hint that must be requested.

**Properties readable from JavaScript.** Screen resolution and color depth, time zone, languages, number of processor cores, whether cookies are enabled, or the list of installed plugins. In 2010, the plugin list and the font list were the most identifying traits (Eckersley, section 4).

**Drawing on a canvas (canvas fingerprinting).** HTML5 lets a page draw on a `<canvas>` element and then read its pixels. Mowery and Shacham demonstrated this in 2012: they drew a sentence in a known typeface and a 3D scene with WebGL, and hashed the pixels into a code. The differences came from the operating system, the browser version, the graphics card, the installed fonts, subpixel hinting, and anti-aliasing. With only 300 participants, one sentence in Arial produced 50 different renderings. The 3D images looked identical to the eye but produced 50 pixel-level variants.

**The server that compares.** It stores the fingerprints it has seen and looks for matches. Because a fingerprint changes over time, searching for the exact code is not enough: a new fingerprint has to be linked to the old one when only one trait has changed, such as the browser version after an update.

### 3. Key parameters

- **The unit is the bit.** Each bit of information halves the group of candidate browsers. A trait shared by half the population contributes 1 bit. One shared by 1 in 1,000 contributes about 10 bits, because 2¹⁰ = 1,024. The W3C puts it this way: one more bit is usually of little concern, but thirty-some bits are enough to tell apart every person on the planet. For comparison, 2³³ is about 8.6 billion.
- **Bits do not simply add up.** Traits are related: the operating system shapes the fonts, and the browser version shapes its capabilities. Eckersley warns that the combined information is less than the sum of each trait on its own.
- **What each trait measures.** In the three large studies compiled by the survey by Laperdrix and colleagues (table 3), the entropy of some traits was:

  | Trait | Panopticlick (2010) | AmIUnique (2016) | Hiding in the Crowd (2018) |
  |---|---|---|---|
  | Plugin list | 15.4 bits | 11.1 bits | 9.5 bits |
  | Font list | 13.9 bits | 8.4 bits | 6.9 bits |
  | User-Agent | 10.0 bits | 9.8 bits | 7.2 bits |
  | Canvas rendering | not measured | 8.3 bits | 8.5 bits |
  | Screen and color | 4.8 bits | 4.9 bits | 4.8 bits |
  | Time zone | 3.0 bits | 3.3 bits | 0.2 bits |

  The time zone contributes almost nothing in the 2018 study because nearly all visitors were in France, with the same time zone. Entropy depends on who makes up the sample.
- **How many browsers are unique.** Panopticlick (2010): 83.6% of 470,161 browsers, and 94.2% of those with Flash or Java. AmIUnique (2016): 89.4% of 118,934, and 81% on mobile. Hiding in the Crowd (2018), with 2,067,942 fingerprints from one of France's fifteen most visited sites: 33.6% overall, 35.7% on desktop and 18.5% on mobile.
- **Stability.** Fingerprints change. In 2010, 37.4% of 8,833 repeat visitors showed some change between visits more than 24 hours apart. Even so, a simple linking method was right 99.1% of the times it ventured an answer, with a 0.86% false-positive rate (Eckersley, section 5.2). The FP-Stalker system, with one fingerprint every three days, followed each device for 51.8 days on average, and 26% of devices for more than 100 days (Laperdrix survey, section 3.2.2).

### 4. Diagram or flow

```text
Browser                                          Website server
-------                                          --------------
HTTP request ────────────────────────────────►  Headers: User-Agent,
                                                 Accept-Language, order...
                                                 + IP address          (passive)

◄─────────────────────────────── Page + script

Script in the browser:
  screen 1920x1080, 24-bit color
  time zone Europe/Madrid
  languages es-ES, en
  available fonts
  canvas drawing ─► pixels ─► code
  graphics card (WebGL)
        │
        ▼
  summary (hash) ─────────────────────────────►  Seen before?    (active)
                                                   ├─ same ────► same browser
                                                   ├─ 1 trait changed ► likely
                                                   └─ new ─────► add to database
```

## Limits, exceptions, and real-world cases

**Unique in a sample is not unique in the world.** Panopticlick and AmIUnique collected visits from privacy-minded people, who tend to have unusual configurations. Eckersley warned about this and did not extrapolate his figures to the whole web. With a general sample of two million fingerprints, the share of unique ones fell to a third. The Laperdrix survey attributes the difference to the type of audience and the size of the sample.

**Phones look more alike.** A given phone model leaves the factory with the same hardware, the same fonts and the same screen. That is why, in the 2018 study, only 18.5% of phones were unique. The W3C notes this too: mobile devices have substantially larger anonymity sets than desktop browsers.

**Plugins disappeared, canvas did not.** In 2010, the plugin list was the most revealing trait. Today's browsers no longer support Flash or Java, and that trait loses weight over time. Canvas rendering and graphics card information took its place: in 2018, canvas was the trait with the most entropy.

**Passive fingerprinting is shrinking.** Trimming the `User-Agent` and moving to *Client Hints* means fine-grained details no longer travel unasked: the server has to request them. That does not hide them, but it makes collection visible. The W3C recommends this approach: making the server request data so that collection can be detected (best practice 8).

**Disguises can give you away.** An add-on that changes the `User-Agent` to say "Linux" while JavaScript still says "Windows" creates a combination almost nobody has. The Laperdrix survey (section 4.1.1) documents that such inconsistencies make users easier to recognize, not harder. Being rare is exactly what a fingerprint looks for.

**There are two defense strategies.** One is uniformity: every user answers the same way. Tor Browser blocks canvas reading and WebGL, removes plugins and ships its own set of fonts. Firefox has offered a similar mode since version 59. The other is randomization: adding small noise to the drawing or to other values so the fingerprint changes each session. Uniformity fails if the user steps out of the group, for example by changing settings. Noise fails if it can be detected or averaged out. The W3C prefers standardizing or coarsening values over randomizing them.

**There is no complete solution.** The W3C considers it implausible to eliminate fingerprinting entirely with widely deployed technical means. It speaks of mitigations, not solutions.

## Practical consequences

For people browsing the web, the main takeaway is that clearing cookies or using a private window is not enough to avoid being recognized. What shrinks the fingerprint is looking like many other people: using a common browser in its default configuration, or one designed so that all its users answer the same way. Every unusual setting or add-on adds bits.

The fingerprint does not depend on the network either. A VPN changes the IP address, but it does not change the fonts or how the graphics card draws.

Not every use is tracking. The Laperdrix survey (section 5) covers its use for fraud and bot detection: if someone logs into your account from a browser with a never-seen fingerprint, the system can ask for extra verification. The same technique serves both to protect and to follow.

For people designing specifications or browsers, the W3C proposes concrete rules. Every new feature should expose only the information it needs. If an API reports the battery level as 0.81234567, then 0.8 or a category such as "low" or "charging" is enough. Detailed data should be requested explicitly so that collection is visible.

## Frequently asked questions

### Is this the same as a cookie?

No. A cookie is data the site stores in your browser and reads back later. A fingerprint stores nothing: it is recomputed on every visit from what the browser already is. That is why it does not show up in the list of site data and is not deleted with it.

### Is this the same as a MAC address?

No. The MAC address identifies the network card to the WiFi access point and does not leave the local network, as [article 11·01](../11_PrivacyAndFootprint/01-WhyPhonesRandomizeTheirMACAddress.md) explains. A browser fingerprint is made of software and hardware traits that a website can see from anywhere on the internet.

### Does incognito mode change my fingerprint?

Barely. It starts without cookies and keeps no history, but the browser, the system, the screen and the graphics card are the same. Some browsers apply extra protections in that mode (unverified browser by browser).

### Can a website learn my name from the fingerprint?

Not by itself. The fingerprint lets a site recognize the same browser across visits, or across different sites that share a tracking system. A name enters the picture when, in one of those visits, you log in or hand over personal data. From then on, what was recognized without a name can be tied to it.

## Sources

- [W3C — Mitigating Browser Fingerprinting in Web Specifications (Group Note, Privacy Working Group, 25 September 2025)](https://www.w3.org/TR/fingerprinting-guidance/). Definition; passive, active and cookie-like fingerprinting (section 3); levels of mitigation success (4.1); entropy and "thirty-some bits" (5); best practices on standardization, coarsening and explicit requests (6.1 to 6.4); implausibility of eliminating it entirely (1.3).
- [Eckersley, P. — How Unique Is Your Web Browser? (Privacy Enhancing Technologies Symposium, 2010)](https://gwern.net/doc/www/coveryourtracks.eff.org/f56d1c28d08db1a7c3b0eeba1a51a40fede53f57.pdf). Copy on gwern.net of the PDF published by the EFF. 470,161 fingerprints, 83.6% and 94.2% unique, 18.1 bits, attributes measured (table 1), changes between visits and linking algorithm (section 5.2).
- [Mowery, K. and Shacham, H. — Pixel Perfect: Fingerprinting Canvas in HTML5 (W2SP, IEEE, 2012)](https://www.ieee-security.org/TC/W2SP/2012/papers/w2sp12-final4.pdf). Canvas method, causes of the differences, 300 participants and 50 distinct renderings of one sentence in Arial.
- [Laperdrix, P., Bielova, N., Baudry, B. and Avoine, G. — Browser Fingerprinting: A Survey (ACM Transactions on the Web, 2020; arXiv version)](https://arxiv.org/pdf/1905.01051). Compared figures for Panopticlick, AmIUnique and Hiding in the Crowd (tables 2 and 3), FP-Stalker (section 3.2.2), defenses by uniformity, randomization and blocking, the inconsistency problem (section 4) and anti-fraud uses (section 5).
- [Gómez-Boix, A., Laperdrix, P. and Baudry, B. — Hiding in the Crowd: an Analysis of the Effectiveness of Browser Fingerprinting at Large Scale (The Web Conference, 2018)](https://hal.archives-ouvertes.fr/hal-01718234). Original study of the 2,067,942 fingerprints; the figures in this article are taken from the survey above.
- [Google — Privacy Sandbox: User-Agent reduction](https://privacysandbox.google.com/protections/user-agent). Reduced `User-Agent` string, the three *Client Hints* sent by default, requesting high-entropy data with `Accept-CH`, and the rationale against passive fingerprinting.
