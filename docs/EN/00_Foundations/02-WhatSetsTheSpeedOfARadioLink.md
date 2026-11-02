---
title: "What decides how many bits per second a radio signal can carry?"
section: "00_Foundations"
language: "en"
status: "published"
order: 2
depth: "intermediate"
summary: "The maximum speed of a radio link is set by two things: the channel bandwidth and the ratio between signal and noise. Shannon summed it up as C = B·log2(1 + S/N). Real devices pick a modulation and coding scheme based on the SNR they measure, and stay somewhat below that limit."
prerequisites:
  - "Decibels and dBm (reviewed in article 00·01)"
  - "What a channel's bandwidth is, measured in hertz"
translation: "../../ES/00_Fundamentos/02-QueDecideLaVelocidadDeUnEnlaceDeRadio.md"
---

# What decides how many bits per second a radio signal can carry?

## Question and framing

With full coverage, a phone downloads quickly. With a single bar, the same network and the same plan are much slower. The question is what sets that ceiling. Why can't a radio channel carry any number of bits per second we like?

This article explains the physical and mathematical limit shared by WiFi, cellular, Bluetooth and every other radio link. It also shows how real devices get close to that limit. It uses the 5G NR tables as the example, because they are public.

- **Depth:** intermediate. One formula with logarithms appears, explained step by step.
- **Prerequisites:** decibels and dBm, reviewed in [Why does a higher-frequency radio signal reach a shorter distance?](01-WhyHigherFrequenciesReachLess.md). Knowing that a channel occupies a bandwidth measured in hertz (for example, 20 MHz).
- **Out of scope:** the details of each modulation, error-correcting code theory, medium access protocols (who talks when) and interference between neighboring networks.

## Short answer

The maximum number of bits per second is set by the channel bandwidth and the ratio of signal power to noise power; neither more signal alone nor more channel alone is enough.

### 60-second summary

Every receiver hears noise. Part of that noise is thermal: electrons jiggling with temperature produce it, and it is always there. Its power grows with the bandwidth being listened to.

Claude Shannon proved in 1948 the maximum rate that a channel with this kind of noise can carry without errors. The formula is C = B·log2(1 + S/N). B is the bandwidth in hertz and S/N is the signal-to-noise ratio (SNR). Below C, data can be sent with as few errors as desired. Above it, it cannot.

The formula shows two consequences. Doubling the bandwidth nearly doubles capacity when the signal is strong. But when the signal is weak, the extra noise of the wider channel eats much of the gain. And with a strong signal, each extra 3 dB of SNR only adds 1 bit per second per hertz.

Devices do not reach the limit. They measure the SNR and choose a modulation and an error-correcting code that work at that level. In 5G NR the standard defines 15 combinations, from 0.15 to 7.41 bits per second per hertz.

## Mental model

Imagine someone dictating numbers to you in a room with background noise. They can dictate faster if the room is quieter or if they speak louder. They could also use several voices at once, each at a different pitch. If the noise rises, they must slow down or repeat things so you don't make mistakes.

Speaking louder is raising the signal power. The room's noise is the receiver's noise. Using more pitches is occupying more bandwidth. Repeating and checking is what error-correcting codes do.

**Where the analogy breaks.** In the room, a fixed background noise does not change when more pitches are used. In radio it does: thermal noise is proportional to the bandwidth the receiver listens to, so every new hertz brings its own noise. Also, the person dictating has no sharp speed limit. The radio channel does, and it can be calculated precisely.

## Technical mechanism

### 1. What happens in the system

The bits travel through four stages:

1. **The receiver measures channel quality.** It estimates the power of the useful signal and that of noise plus interference.
2. **The receiver reports to the transmitter.** In 5G NR it does so with a number called the CQI (channel quality indicator), from 0 to 15 (3GPP TS 38.214, clause 5.2.2.1).
3. **The transmitter chooses how to send.** It picks a modulation (how many bits each symbol carries) and a code rate (which share of the transmitted bits is information and which is redundancy for correcting errors).
4. **The receiver decodes.** If the actual SNR was enough for that combination, it recovers the bits. If not, it asks for a retransmission and the transmitter steps down.

This loop repeats continuously. That is why speed changes as you move, even if nothing else changes.

### 2. What each component does

**Thermal noise.** Any resistance above absolute zero generates noise. The available power is p = k·T·b, where k is Boltzmann's constant (1.38 × 10⁻²³ J/K), T is the temperature in kelvin and b the bandwidth in hertz. Rec. ITU-R P.372-16 takes T0 = 290 K as the reference. With that value, 10·log10(k·T0) = −204 dBW/Hz, that is, −174 dBm/Hz (P.372-16, clause 1.2.1 and eq. 6). Over 20 MHz, thermal noise is −174 + 73 = −101 dBm.

**The receiver's noise figure.** The amplifier and the rest of the electronics add their own noise. It is summarized as a figure in dB added to the thermal noise. Its value depends on each design. The examples in this article use 7 dB as an illustrative assumption, not as data for any specific device.

**Signal-to-noise ratio (SNR).** It is the signal power divided by the noise power. In dB it is a subtraction: SNR = S (dBm) − N (dBm). A −70 dBm signal over a −94 dBm noise floor gives 24 dB of SNR.

**The Shannon limit.** Shannon treated a band-limited channel with white Gaussian noise, which is the model for thermal noise (Shannon, 1948, Part IV). With limited average transmit power, the capacity is:

```text
C = B · log2(1 + S/N)     [bit/s]
B in Hz · S/N as a ratio (not in dB)
```

The theorem has two sides. Below C, there is a coding scheme that makes the error rate as low as desired. Above C, no code can do it. The formula does not say how to build that code: it only says it exists.

**Modulation.** Each transmitted symbol picks one point from a "constellation". QPSK has 4 points and carries 2 bits per symbol. 16QAM carries 4 bits, 64QAM carries 6 and 256QAM carries 8. More points means points closer together. With points closer together, less noise is enough to mistake one for another.

**The error-correcting code.** It adds calculated redundancy. A code rate of 948/1024 means 92.6% of the transmitted bits are information. A rate of 78/1024 means only 7.6% are; the rest is there to correct errors.

**Spectral efficiency.** It is the product of the two: bits per symbol × code rate. It is measured in bit/s/Hz. For 64QAM at rate 948/1024 it comes to 6 × 948/1024 = 5.55 bit/s/Hz. This is the figure 3GPP publishes in its tables.

### 3. Key parameters

**What the formula says in numbers.** Scenario: noise floor = thermal noise at 290 K + an assumed 7 dB noise figure. Shannon capacity calculated with C = B·log2(1 + S/N).

| Received signal | 20 MHz channel (noise −94 dBm) | 40 MHz channel (noise −91 dBm) |
|---|---|---|
| −50 dBm | SNR 44 dB · 292 Mbit/s | SNR 41 dB · 545 Mbit/s |
| −70 dBm | SNR 24 dB · 160 Mbit/s | SNR 21 dB · 279 Mbit/s |
| −85 dBm | SNR 9 dB · 63 Mbit/s | SNR 6 dB · 92 Mbit/s |
| −95 dBm | SNR −1 dB · 17 Mbit/s | SNR −4 dB · 19 Mbit/s |

Three readings of the table:

- With a strong signal (−50 dBm), doubling the channel multiplies capacity by 1.86.
- With a weak signal (−95 dBm), doubling the channel only multiplies it by 1.15. The wide channel collects twice the noise and the same signal.
- With negative SNR (the signal is below the noise) capacity is not zero. It drops, but the link can still carry data with plenty of redundancy.

**The 3 dB rule.** At high SNR, log2(1 + S/N) ≈ log2(S/N). Doubling S/N (+3 dB) adds 1 to that logarithm. Each extra 3 dB contributes 1 bit/s/Hz. That is why going from 20 to 23 dB of SNR helps little in relative terms, while going from 0 to 3 dB helps a lot.

**What a real system uses: 5G NR CQI tables.** The 3GPP TS 38.214 standard (Rel-19, Table 5.2.2.1-3) defines these combinations. The last column is the minimum SNR the Shannon limit would require for that efficiency, calculated as 2^η − 1. It is not the SNR a real receiver needs, which is higher.

| CQI | Modulation | Code rate (×1024) | Efficiency (bit/s/Hz) | Shannon minimum SNR |
|---|---|---|---|---|
| 1 | QPSK | 78 | 0.1523 | −9.5 dB |
| 3 | QPSK | 449 | 0.8770 | −0.8 dB |
| 6 | 16QAM | 616 | 2.4063 | 6.3 dB |
| 9 | 64QAM | 666 | 3.9023 | 11.4 dB |
| 12 | 256QAM | 711 | 5.5547 | 16.6 dB |
| 15 | 256QAM | 948 | 7.4063 | 22.3 dB |

The device picks the highest CQI at which a block of data would be received with an error probability no greater than 0.1 (TS 38.214, clause 5.2.2.1). That 10% of failed blocks is not lost: it is recovered through retransmissions.

The standard does not specify which SNR each CQI requires. That depends on each vendor's receiver and on the real channel. That is why this article does not give that figure.

### 4. Diagram or flow

```text
THERMAL NOISE: −174 dBm/Hz × bandwidth B   (+ receiver noise figure)
        │
        ▼
SNR = received signal (dBm) − noise (dBm)
        │
        ├──► THEORETICAL LIMIT:  C = B · log2(1 + S/N)
        │
        ▼
RECEIVER measures SNR ──► reports CQI (0…15) ──► TRANSMITTER
                                                   │
                                                   ▼
                       picks modulation (2…8 bits/symbol)
                       and code rate (78…948 /1024)
                                                   │
                                                   ▼
               real rate ≈ B × efficiency × (share of resources for data)
                                                   │
                                                   ▼
               block error? ── yes ──► retransmission and lower CQI
                            └─ no ──► bits delivered
```

## Limits, exceptions, and real-world cases

- **The formula assumes white Gaussian noise.** Interference from other networks does not always behave that way. It arrives in bursts and concentrates in parts of the channel. In practice, systems measure signal against noise plus interference (SINR) and treat it as if it were SNR.
- **Not all of the channel carries user data.** Part of the resources goes to reference, control and synchronization signals. The useful rate is lower than bandwidth × efficiency. The exact share depends on each network's configuration.
- **SNR changes within milliseconds.** As you move, reflections add up and cancel out (multipath). That is why the CQI is reported over and over, and speed fluctuates even if the user doesn't notice.
- **The limit is per channel, not per device.** Several antennas can create several parallel channels over the same band (MIMO). Telatar showed in 1999 that, in an environment with rich scattering, capacity grows roughly in proportion to the smaller number of antennas at the two ends. MIMO does not violate the Shannon limit: it applies it to each channel.
- **Time sharing divides capacity.** If ten devices share the same WiFi channel or the same cell, they split the capacity. An excellent signal does not guarantee speed if the channel is busy.

## Practical consequences

- **Bars are not speed.** The coverage icon usually reflects received power. Speed depends on SNR or SINR and on how many devices share the channel. With good power and heavy interference, speed can be low.
- **A wider channel does not always pay off.** Far from the access point, an 80 MHz channel receives four times more noise than a 20 MHz one. At −90 dBm and with the example's assumptions, capacity goes from 36 to 56 Mbit/s: a factor of 1.55, not 4.
- **Moving closer to the router helps most when the signal is weak.** From −95 to −85 dBm, the capacity in the 20 MHz example nearly quadruples. From −60 to −50 dBm, it only rises by 29%.
- **For device designers.** Every dB of noise figure saved in the receiver is a dB of SNR gained without touching transmit power. In the weak-signal zone, that dB is worth a lot.

## Frequently asked questions

### Are bandwidth and speed the same thing?

No. Bandwidth is the width of the channel in hertz. Speed is a bitrate in bit/s. The link between the two is spectral efficiency, which depends on SNR. This project reserves "bandwidth" for hertz, as the [glossary](../../../guides/TaxonomyAndGlossary.md) states.

### Can the Shannon limit be beaten?

Not in the model it was proven for. Modern codes get close, but stay below. What can be done is to change the problem: more bandwidth, more antennas (more parallel channels) or more SNR.

### Why is the advertised speed never reached?

The commercial figure usually corresponds to the best modulation and coding combination, with the widest channel and all antennas, and nobody else sharing the channel. It requires a high SNR that only occurs close to the transmitter and without interference.

### Is there a minimum energy per bit?

Yes, and it follows from the same formula. If a fixed power is spread over an ever wider bandwidth, the SNR falls and capacity stops growing. The result is that each bit needs at least ln 2 times the noise density, about −1.6 dB. No system sends reliable data with less energy per bit.

## Sources

- C. E. Shannon. "A Mathematical Theory of Communication". *The Bell System Technical Journal*, vol. 27, pp. 379-423 (July) and 623-656 (October), 1948. Part IV: continuous channel with white noise. Copy of the reprint with corrections: <https://people.math.harvard.edu/~ctm/home/text/others/shannon/entropy/entropy.pdf>
- ITU-R. *Recommendation ITU-R P.372-16: Radio noise* (08/2022). Clause 1.2.1, equations 2 and 6. <https://www.itu.int/dms_pubrec/itu-r/rec/p/R-REC-P.372-16-202208-I!!PDF-E.pdf>. Current version: P.372-18 (09/2026), <https://www.itu.int/rec/R-REC-P.372/en>
- 3GPP. *TS 38.214: NR; Physical layer procedures for data* (Rel-19). Clause 5.2.2.1, Tables 5.2.2.1-2 and 5.2.2.1-3. <https://www.etsi.org/deliver/etsi_ts/138200_138299/138214/>
- I. E. Telatar. "Capacity of Multi-antenna Gaussian Channels". *European Transactions on Telecommunications*, vol. 10, no. 6, pp. 585-595, 1999. DOI: [10.1002/ett.4460100604](https://doi.org/10.1002/ett.4460100604)
