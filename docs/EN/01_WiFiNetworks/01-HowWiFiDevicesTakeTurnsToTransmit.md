---
title: "How do devices on a WiFi network take turns so they don't transmit at once?"
section: "01_WiFiNetworks"
language: "en"
status: "published"
order: 1
depth: "intermediate"
summary: "On WiFi nobody hands out turns frame by frame. Each device listens to the channel, waits for it to stay idle for a fixed time, then counts down a random number of 9 µs slots. If its frame is not acknowledged, it assumes a collision and doubles the draw. This method, CSMA/CA, explains why WiFi loses performance with many devices or with one slow one."
prerequisites:
  - "Signal-to-noise ratio and why link speed depends on it (article 00·02)"
  - "What a packet and an acknowledgement are (glossary)"
translation: "../../ES/01_RedesWiFi/01-ComoSeTurnanLosAparatosDeUnaRedWiFi.md"
---

# How do devices on a WiFi network take turns so they don't transmit at once?

## Question and framing

In a home, the router, two laptops, three phones and a TV share the same radio channel. If two of them transmit at the same time on that channel, their signals mix and the receiver understands neither. Nobody tells these devices when to speak, frame by frame. Yet the network works. How do they decide when to transmit without talking over each other?

This article explains the medium access mechanism of the IEEE 802.11 family: the distributed coordination function (DCF) and its prioritized version (EDCA). It uses the values of the OFDM physical layer in the 5 GHz band, shared by 802.11a, n and ac.

- **Depth:** intermediate. There are simple calculations with microseconds and probabilities.
- **Prerequisites:** it helps to know that link speed depends on the signal-to-noise ratio, explained in [What decides how many bits per second a radio signal can carry?](../00_Foundations/02-WhatSetsTheSpeedOfARadioLink.md).
- **Out of scope:** how a device joins the network (association and authentication), encryption, channel selection, and the per-subcarrier allocation of 802.11ax, which is only mentioned under limits.

## Short answer

Each device listens before it talks, waits for the channel to stay idle for a while, then lets a random number of slots go by; if its frame is not acknowledged, it assumes a collision and tries again with a draw twice as wide.

### 60-second summary

The method is called CSMA/CA: carrier sense multiple access with collision avoidance. A device that wants to send a frame first checks whether the channel is busy. If it stays idle for a fixed interval (34 µs at 5 GHz), the device still does not transmit right away. It picks a random number between 0 and 15 and counts down that many 9 µs slots. If someone starts transmitting during the countdown, it freezes the count and resumes when the channel is idle again.

When it reaches zero, it transmits. The receiver answers with an acknowledgement (ACK) 16 µs later. If the ACK does not arrive, the sender cannot tell why, but it assumes a collision. It then doubles the range of the draw (0-31, 0-63…) up to a maximum of 1023 and tries again.

Since each device draws on its own, two rarely pick the same number. Rarely is not never: with more devices competing, collisions increase. Also, turns are shared out by transmission opportunities, not by time. A slow device holds the channel much longer on each turn and slows everyone else down.

## Mental model

Think of a dinner with friends and no moderator. Nobody talks while someone else is talking. When someone finishes, everyone waits a moment. If two people want to speak, each lets a different instant pass before opening their mouth. Whoever starts first gets the floor; the other stays quiet and waits for the next silence. If two start at once, both stop and next time wait a little longer before trying.

The silence is the idle channel. The shared moment of waiting is the fixed interval. Each person's different instant is the random draw. Stopping and waiting longer after a clash is the doubling of the range.

**Where the analogy breaks down.** At the table, the speaker hears if someone else starts at the same time. A WiFi radio does not: while it transmits, its own signal reaches its antenna with far more power than a neighbour's, and it cannot listen at the same time. It only discovers the collision because the ACK is missing. Also, at the table everyone hears everyone. On WiFi, two devices at opposite ends of a home may not hear each other, even though both hear the router. And the floor is shared by turns, not by minutes: whoever speaks very slowly hogs the conversation.

## Technical mechanism

### 1. What happens in the system

Sequence of one transmission with basic access (IEEE 802.11-2024, distributed coordination function; values for the OFDM physical layer at 5 GHz):

1. **A frame arrives in the queue.** The device checks the channel with two mechanisms. The physical one (CCA, clear channel assessment) measures whether there is an 802.11 signal or energy on the radio. The virtual one (NAV) consults an internal timer that says whether another device has reserved the channel.
2. **Wait for DIFS.** The channel must be idle for a DIFS interval. DIFS = SIFS + 2 × slot = 16 + 2 × 9 = 34 µs (Jain, slide 5-17; Fairhurst).
3. **Random countdown.** If it has no pending count, it draws an integer between 0 and CW (contention window). Initially CW = CWmin = 15. It subtracts one for each 9 µs slot during which the channel stays idle (Jain, slides 5-15 and 5-16).
4. **Freezing.** If the channel becomes busy during the countdown, the counter stops without losing its value. When the channel is idle again for a DIFS, it continues from where it left off (Jain, slide 5-20).
5. **Transmission.** With the counter at zero, it sends the frame. Its header carries a Duration field, in microseconds, with the time it will still occupy the channel until the end of the ACK (Jain, slides 5-18 and 5-24).
6. **Acknowledgement.** The receiver checks the frame and replies with an ACK after a SIFS interval of 16 µs. SIFS is shorter than DIFS. That way no other device can slip in between the frame and its ACK.
7. **Success or retry.** If the ACK arrives, CW returns to CWmin. If not, CW becomes min(2·CW + 1, CWmax), that is 15 → 31 → 63 … → 1023, and the process repeats from step 2 (Jain, slide 5-16). After a maximum number of attempts, the frame is dropped and the problem passes to higher layers, such as TCP.

### 2. What each component does

**Physical carrier sense (CCA).** The radio decides whether the channel is busy. It distinguishes two cases. If it detects the preamble of an 802.11 frame, it treats the channel as busy even for very weak signals. If there is only energy without 802.11 format (a microwave oven, another technology), it takes a level about 20 dB higher to call it busy (Coleman, Extreme Networks). The standard sets a −82 dBm threshold for the preamble on 20 MHz channels; that figure is **unverified** in the sources read for this article, which only confirm the gap of about 20 dB between the two thresholds.

**Virtual carrier sense (NAV).** Every device that decodes a frame reads its Duration field and loads that value into its NAV. Until the NAV reaches zero, it treats the channel as busy even if it hears nothing (Jain, slides 5-15 and 5-18). This protects the ACK and multi-frame exchanges.

**Interframe spaces.** These are waits of different lengths that set priorities without any messages:

- SIFS (16 µs): between a frame and its immediate response, such as the ACK.
- PIFS (SIFS + 1 slot = 25 µs): for prioritized coordination functions (Jain, slide 5-17).
- DIFS (SIFS + 2 slots = 34 µs): for normal access without priorities.
- AIFS (SIFS + AIFSN × slot): the prioritized version used by EDCA, described below (IEEE 802.11-2024).

Whoever has to wait less always beats whoever waits more. That is the whole hierarchy.

**The slot.** It is the unit of the countdown. Its length covers the time a radio needs to detect that another has started transmitting, plus the propagation delay. It is 9 µs for OFDM at 5 GHz and 20 µs in the original 802.11b (Jain, slide 5-17; Fairhurst). If two devices finish their countdown in the same slot, neither sees the other in time and they collide.

**The contention window (CW).** It is the range of the draw. It starts small so as not to waste time when there is little competition. It grows after each failure to spread devices over more slots when there is a lot. This growth is called binary exponential backoff.

**The ACK.** It is the only way to know the frame arrived. A half-duplex radio cannot listen while it transmits. Also, the collision happens at the receiver, which may be far from the sender. That is why 802.11 does not detect collisions the way wired Ethernet did: it acknowledges every frame and infers failure from absence (Jain, slides 5-10 and 5-12).

**RTS/CTS.** It is an optional exchange before a long frame. The sender transmits a short RTS frame with the duration of the whole exchange. The receiver answers with a CTS that repeats the reservation. Anyone who hears either one loads its NAV and stays quiet (Jain, slide 5-12). If two RTS frames collide, very little time is lost, because they are short.

**EDCA: four queues with different rules.** The quality-of-service extensions split traffic into four access categories (AC). Each competes with its own AIFSN, its own window and an occupancy limit (TXOP). Default values from a commercial chip, the TI CC33xx, which uses them both as an access point and as a client:

| Category | AIFSN | AIFS at 5 GHz (calculated) | CWmin | CWmax | Max TXOP |
|---|---|---|---|---|---|
| Voice (AC_VO) | 2 | 34 µs | 3 | 7 | 1.504 ms |
| Video (AC_VI) | 2 | 34 µs | 7 | 15 | 3.008 ms |
| Best effort (AC_BE) | 3 | 43 µs | 15 | 1023 | one frame |
| Background (AC_BK) | 7 | 79 µs | 15 | 1023 | one frame |

Source: TI SWRU616B, table 2-3. The AIFSN values match the standard's defaults for conventional devices (IEEE 802.11-2024). A TXOP of 0 in the table means the category sends a single frame per channel access won. The access point announces in its beacons the values that apply on its network.

### 3. Key parameters

**Basic timings.** OFDM physical layer at 5 GHz versus 802.11b at 2.4 GHz:

| Parameter | OFDM 5 GHz (802.11a/n/ac) | 802.11b |
|---|---|---|
| Slot | 9 µs | 20 µs |
| SIFS | 16 µs | 10 µs |
| DIFS (calculated) | 34 µs | 50 µs |
| CWmin | 15 | 31 |
| CWmax | 1023 | 1023 |

Sources: Jain, slide 5-17; Fairhurst; Heusse *et al.* for 802.11b.

**The cost of waiting.** With CW = 15, the draw averages 7.5 slots: 7.5 × 9 µs = 67.5 µs. Adding DIFS and SIFS, each frame pays about 117.5 µs of waiting before counting the preamble and the ACK (calculated). At 600 Mbit/s, a 1500-byte frame (12,000 bits) takes only 20 µs of data. The waiting far exceeds the data. That is why 802.11n and later aggregate several frames into a single transmission: they spread the same fixed cost over more bytes.

**Probability of colliding in the first round.** Illustrative assumption: n devices start a fresh countdown at the same time with CW = 15. There is a collision if two or more draw the lowest number. Calculated:

| Devices competing | Collision probability |
|---|---|
| 2 | 6.3% |
| 3 | 9.2% |
| 5 | 15.0% |
| 10 | 28.3% |
| 20 | 50.4% |

In practice not everyone competes at once, and the window grows after each collision. Bianchi modelled the whole network under saturation, with every device always having data. He concluded that with basic access throughput falls as the number of devices grows. With RTS/CTS it stays practically constant, because what collides are short frames (Bianchi, IEEE JSAC 2000).

**The slow device.** CSMA/CA gives every device the same long-term probability of winning the channel, not the same time. Heusse *et al.* measured this on 802.11b. With two devices at 11 Mbit/s, each got between 3.1 and 3.4 Mbit/s of UDP. With one at 11 Mbit/s and the other at 1 Mbit/s, both dropped to about 0.76 Mbit/s. The fast one fell below 1 Mbit/s, because on each turn the slow one holds the channel about eleven times longer (Heusse *et al.*, INFOCOM 2003, table II).

### 4. Diagram or flow

```text
BASIC ACCESS (OFDM 5 GHz: slot 9 µs, SIFS 16 µs, DIFS 34 µs)

A ─[ A's frame ]─16µs─────────[B's ACK]
                                       ├─ DIFS 34µs ─┤ count ┤[ C's frame ]...
C (wants to send, drew 5)
   channel busy → counter frozen at 5           5→4→3→2→1→0 ┘
D (wants to send, drew 9)
   channel busy → frozen at 9                   9→8→7→6→5→4 ║ C transmits: D freezes at 4
                                                            ║ and resumes at the next gap

AFTER A FAILURE (no ACK)
  CW: 15 → 31 → 63 → 127 → 255 → 511 → 1023 → 1023 …   (after success: back to 15)

HIDDEN NODE AND RTS/CTS
  A ))) ((( router ))) ((( C        A and C hear the router, but not each other
  A ──RTS (Duration = whole exchange)──→ router
  A ←─CTS (remaining Duration)──── router ────→ C   C hears the CTS, loads its NAV, stays quiet
  A ──DATA──→ router
  A ←─ACK──── router
```

## Limits, exceptions, and real-world cases

- **The hidden node.** If A and C cannot hear each other, both see the channel as idle and may transmit to the router at the same time (Jain, slide 5-10). Their frames collide at the router and neither knows until the ACK is missing. RTS/CTS mitigates this, but adds two frames and two SIFS to every transmission. Many devices only use it for long frames, above a configurable threshold.
- **Neighbouring networks on the same channel.** CSMA/CA does not distinguish your network from your neighbour's. Every device that can hear the others on that channel takes turns as if they were a single network. A channel shared by several flats is divided among all of them.
- **A lost ACK is not always a collision.** A frame can fail because of noise or a weak signal. The sender still doubles the window, even if nobody else was there. Waiting longer does not fix a link with a poor signal-to-noise ratio; lowering the modulation rate is what addresses that.
- **Non-WiFi interference.** A foreign signal without an 802.11 preamble only blocks the channel if it exceeds the energy threshold, about 20 dB above the preamble detection threshold (Coleman). Below that, the device transmits on top of it and the frame may be lost.
- **802.11ax adds central allocation, but does not replace contention.** Since 802.11ax, the access point can divide subcarriers among several clients in a single transmission, similar to what a cell tower does (see [how a cell tower shares its capacity](../02_CellularNetworks/01-HowACellTowerSharesItsCapacity.md)). To start that transmission, the access point must win the channel under the same rules described here (IEEE 802.11-2024).
- **The access point decides the EDCA values.** The table above shows defaults. The access point may announce others in its beacons, and the devices on its network must use them (TI SWRU616B).

## Practical consequences

- **Speed is shared, with losses.** If three devices download at once, each gets somewhat less than a third. Part of the time goes on waits, ACKs and collisions.
- **An old or distant device slows everyone down.** Because turns are per frame and not per unit of time, a low-rate client holds the channel much longer on each turn (Heusse *et al.*). Some access points compensate by sharing out airtime, but that is a vendor decision, not part of the standard.
- **Voice and video calls go first.** With EDCA, a voice frame waits 34 µs and draws between 0 and 3. A background frame waits 79 µs and draws between 0 and 15. Voice almost always wins.
- **More devices on the same channel means more contention, not just more sharing.** The probability of a collision grows with the number of competing devices. That is why a channel crowded with neighbouring networks performs poorly even when the signal is good.
- **For designers:** the slot and the interframe spaces are the fixed cost of having no coordinator. At high rates that cost dominates, and efficiency depends on aggregating frames and reducing the number of channel accesses.

## Frequently asked questions

### Is this the same as wired Ethernet's CSMA/CD?

No. In classic shared-cable Ethernet, the sender listened to the cable while transmitting and cut the frame short when it detected a collision (CD, collision detection). On radio that is not possible, because of the power gap between one's own signal and someone else's, and because the collision happens at the receiver. WiFi tries to avoid collisions before transmitting (CA, collision avoidance) and infers them afterwards from the missing ACK.

### Does the router decide who talks?

Not in basic access. The router competes for the channel under the same rules as any client, even though it sends the most traffic. What it does decide are the EDCA parameters of its network and, since 802.11ax, it can divide subcarriers in the transmissions it starts.

### Why a random number and not a fixed order?

A fixed order would require everyone to know how many devices there are and who goes next. That needs a coordinator and extra messages. The draw works without knowing the others, and devices that join or leave break nothing.

### Do more signal bars avoid the waiting?

No. The bars show the power your device receives. The waiting depends on how many devices compete for the channel and how long each one holds it.

## Sources

- IEEE. *IEEE Std 802.11-2024: Wireless LAN Medium Access Control (MAC) and Physical Layer (PHY) Specifications*. Approved 2024-09-26, published 2025-04-28. <https://standards.ieee.org/standard/802_11-2024.html>
- R. Jain. *IEEE 802.11 Wireless LANs, Part I: Basics*. Washington University in St. Louis, course CSE574, 2022. Slides 5-10 to 5-24. <https://research.engineering.wustl.edu/~jain/cse574-22/ftp/j_05lan.pdf>
- G. Fairhurst. *Carrier Sense Multiple Access with Collision Avoidance (CSMA/CA)*. University of Aberdeen, Electronics Research Group. <https://erg.abdn.ac.uk/users/Gorry/course/lan-pages/csma-ca.html>
- Texas Instruments. *CC33xx WLAN Features Guide* (SWRU616B), section 2.7, table 2-3. <https://www.ti.com/document-viewer/lit/html/SWRU616B/GUID-8D9EA799-1FFC-40DC-A124-0FC66CCD82D4>
- D. Coleman. *What is a Clear Channel Assessment (CCA)?* Extreme Networks, 2017. <https://www.extremenetworks.com/resources/blogs/what-is-a-clear-channel-assessment-cca>
- G. Bianchi. "Performance Analysis of the IEEE 802.11 Distributed Coordination Function". *IEEE Journal on Selected Areas in Communications*, vol. 18, no. 3, pp. 535-547, March 2000. <https://s2.smu.edu/~camp/courses/ee8378/materials/bianchi2000.pdf>
- M. Heusse, F. Rousseau, G. Berger-Sabbatel, A. Duda. "Performance anomaly of 802.11b". *IEEE INFOCOM 2003*. <https://ieeexplore.ieee.org/document/1208921> · open copy: <https://hal.archives-ouvertes.fr/hal-01199120>
