---
title: "Why does signal strength change so much when you move a phone a few centimetres?"
section: "00_Foundations"
language: "en"
status: "published"
order: 3
depth: "intermediate"
summary: "The signal reaches a phone along many paths at once: direct, and bounced off walls, furniture and people. Each copy arrives with a different delay, and the copies add up or cancel depending on their phase. Moving half a wavelength (6 cm at 2.4 GHz) is enough to go from a peak to a 20 or 30 dB dip."
prerequisites:
  - "Decibels and wavelength (reviewed in article 00·01)"
  - "What the signal-to-noise ratio is (article 00·02)"
translation: "../../ES/00_Fundamentos/03-PorQueLaCoberturaCambiaEnPocosCentimetros.md"
---

# Why does signal strength change so much when you move a phone a few centimetres?

## Question and framing

It is easy to see for yourself. The laptop is slow at one corner of the desk and fine a hand's width away. The phone loses a bar when you turn it in your hand. The router has not moved and the walls are the same. The question is why the signal changes so much over such short distances.

This article explains multipath fading. It is the mechanism that makes received power rise and fall over a few centimetres and a few milliseconds. It affects WiFi, cellular, Bluetooth and any radio link that lacks a single clean path.

- **Depth:** intermediate. A vector sum and a probability distribution appear, both explained step by step.
- **Prerequisites:** decibels and the relation wavelength = speed of light / frequency, reviewed in [Why does a higher-frequency radio signal reach a shorter distance?](01-WhyHigherFrequenciesReachLess.md). The signal-to-noise ratio (SNR), explained in [What decides how many bits per second a radio signal can carry?](02-WhatSetsTheSpeedOfARadioLink.md).
- **Out of scope:** equaliser design, full MIMO theory, the channel models of each standard, and millimetre-wave propagation.

## Short answer

The signal arrives along many paths at once, and its copies add up or cancel depending on the exact distance each one travelled; moving half a wavelength is enough to change that sum.

### 60-second summary

In a home or on a street, the transmitter's wave bounces off walls, floors, furniture, cars and people. Many copies of the same signal reach the receiver. Each has travelled a different path and arrives with a different delay.

What matters is not only how much energy each copy carries. It also matters where in its cycle it arrives. If two copies arrive in phase, they reinforce each other. If they arrive half a wave apart, they subtract. At 2.4 GHz the wave is about 12.5 cm long. A 6 cm change in the length of one path is enough to flip from adding to subtracting.

With many copies and none dominant, the sum behaves like a random value. It follows what is called a Rayleigh distribution. Under it, power is more than 10 dB below the mean over 9.5 % of the route, and more than 20 dB below over 1 %. The deepest dips are a few centimetres apart.

Devices defend themselves in three ways: several antennas a few centimetres apart, many subcarriers spread across the channel, and error-correcting codes that spread each piece of data over several of them.

## Mental model

Picture the surface of a swimming pool with several people slapping the water at the same rhythm, from different spots. The ripples cross. In some places two crests meet and the water rises a lot. In others a crest meets a trough and the water barely moves. A cork that drifts a hand's width goes from a choppy spot to a calm one.

The people are the signal paths. The cork is the phone's antenna. The pattern of choppy and calm spots is the interference pattern the receiver moves through.

**Where the analogy breaks down.** In the pool you can see every source and predict the pattern. In a room there are dozens of paths, and each one changes when someone opens a door or walks down the hall. So the exact value at each point is not predicted. Its statistics are described instead. Also, radio does not only vary in space: within a single channel, some frequencies fall into a dip and others do not. The cork has no equivalent for that.

## Technical mechanism

### 1. What happens in the system

1. **The transmitter radiates.** The wave leaves in many directions at once.
2. **The environment spreads it.** Part arrives in a straight line, if there is line of sight. The rest is reflected, diffracted at edges or scattered by small objects.
3. **The copies arrive with different delays.** Each path has its own length. A 3 m difference equals about 10 ns of delay.
4. **The antenna adds everything up.** The antenna does not tell copies apart. It receives the sum of all fields at that point and that instant.
5. **The result depends on the phases.** Each copy arrives with an amplitude and a phase. The sum can be larger, or much smaller, than any of them.
6. **Moving changes the phases.** Shifting the antenna by a fraction of a wavelength changes the relative path lengths. The sum changes with them.

### 2. What each component does

**Paths and their phase.** A copy that travels a path d arrives with a phase proportional to d/λ, where λ is the wavelength. Every extra λ of travel is one full turn. Half an extra λ is half a turn: the copy arrives inverted relative to another.

**The vector sum.** Each copy can be drawn as an arrow. Its length is the amplitude and its angle is the phase. The received signal is the sum of all the arrows. Rec. ITU-R P.1057-7 describes the Rayleigh distribution in exactly these terms: the length of the sum of many vectors of similar amplitude with uniformly random phase (section 5). When no copy dominates, the magnitude of the sum follows that distribution.

**The Rayleigh distribution.** Its cumulative distribution function is F(x) = 1 − exp(−x²/b²), where b is the root-mean-square amplitude (P.1057-7, section 5, eq. 10). Converted to power, the probability of being more than X dB below the mean power is:

```text
P(power < mean − X dB) = 1 − exp(−10^(−X/10))
```

At low levels this curve has a fixed slope. Each decade of probability corresponds to 10 dB of power (P.1057-7, section 5, eq. 11).

**The direct component.** With line of sight, one copy is much stronger than the rest. The sum then follows a Nakagami-Rice distribution: a fixed vector plus a Rayleigh vector (P.1057-7, section 7). It is summarised by the K factor, the ratio between the power of the fixed component and that of the random part. With K = 0 the distribution becomes Rayleigh again (Rec. ITU-R P.1406-2, section 3.2.2). The larger K is, the shallower the dips.

**Correlation in space.** Clarke (1968) calculated how similar the field is at two points a distance δ apart. He assumed waves arrive from every horizontal direction with equal probability. The result is the Bessel function J0(2πδ/λ) (Clarke, eq. 9). The envelope correlation is approximately its square (eq. 16). J0 first reaches zero at 2πδ/λ = 2.405, that is, at δ ≈ 0.38 λ. At that distance, two antennas see almost independent fading.

**The Doppler effect.** If the receiver moves at speed v, each copy gets a different frequency shift, depending on its direction of arrival. The maximum is f_m = v/λ (Clarke, eq. 21). Moving through the spatial pattern at speed v turns it into a variation in time. The time correlation is J0(2π·f_m·τ) (Clarke, eq. 18).

**Delay spread.** The copies do not only have different phases. They also arrive spread out in time. The root-mean-square value of that spread is called the r.m.s. delay spread. If it is large compared with the duration of a symbol, one symbol bleeds into the next (P.1406-2, section 4.1). In frequency, the same phenomenon makes some parts of the channel fall into a dip while others do not. Clarke relates the bandwidth over which the signal stays coherent to the inverse of that spread.

### 3. Key parameters

**Length scales.** Wavelength λ = c/f. Distance to the first decorrelation in Clarke's model: 0.38 λ.

| Frequency | λ | λ/2 | 0.38 λ |
|---|---|---|---|
| 700 MHz | 42.8 cm | 21.4 cm | 16.4 cm |
| 2.4 GHz | 12.5 cm | 6.2 cm | 4.8 cm |
| 5.5 GHz | 5.5 cm | 2.7 cm | 2.1 cm |
| 6 GHz | 5.0 cm | 2.5 cm | 1.9 cm |

P.1406-2 states that the minima of an interference pattern are at least half a wavelength apart (section 3.3). In urban or wooded areas, the field measured over a few tens of wavelengths approximately follows a Rayleigh distribution. Fades of 30 dB or more below the mean "are common" (P.1406-2, section 3.3).

**Depth and frequency of dips (Rayleigh, no direct component).** Calculated with the formula above.

| Level relative to mean power | Fraction of the route below it |
|---|---|
| −3 dB | 39 % |
| −10 dB | 9.5 % |
| −20 dB | 1.0 % |
| −30 dB | 0.1 % |

The median power sits 1.6 dB below the mean. A single measurement point says almost nothing about real coverage.

**Time scales.** Illustrative assumptions: a person walking at 1.4 m/s and a car at 120 km/h. The decorrelation time is the first zero of J0(2π·f_m·τ), at τ ≈ 0.38/f_m.

| Scenario | f_m = v/λ | Time to first decorrelation |
|---|---|---|
| Walking, 2.4 GHz | 11 Hz | 34 ms |
| Walking, 5.5 GHz | 26 Hz | 15 ms |
| Car at 120 km/h, 2 GHz | 222 Hz | 1.7 ms |

**Indoor delay spread.** Rec. ITU-R P.1238-13, table 7. Omnidirectional antennas, 10 ns resolution. 10 %, median and 90 % values, for the largest rooms typical of each environment.

| Environment | 3.7 GHz | 5.2 GHz |
|---|---|---|
| Residential | 15 · 22 · 27 ns | 17 · 23 · 30 ns |
| Office | 30 · 38 · 45 ns | 38 · 60 · 110 ns |
| Commercial | 105 · 145 · 170 ns | 135 · 190 · 205 ns |

With a 60 ns spread, the inverse is about 17 MHz. That is only an order of magnitude: the exact factor depends on how coherence is defined. But it is enough to show that a WiFi channel of 20 MHz or more is wider than the coherent zone. Inside the channel there will be frequencies in a dip and frequencies reinforced.

**Two scales not to confuse.** P.1406-2 separates fast variation, due to multipath, which happens over a few wavelengths, from slow variation, due to changes in shadowing by buildings and obstacles (section 3.2). The slow part is described with a log-normal distribution. In a large urban area, its standard deviation at 2 GHz is about 7.5 dB (P.1406-2, eq. 2, valid from 100 to 3000 MHz). To separate the two, the recommendation suggests measuring over about 40 wavelengths. About 36 samples spaced at least 0.8 λ apart are needed to get the median within 1 dB with 90 % probability (section 3.2).

### 4. Diagram or flow

```text
              wall                            furniture
                │  path 2 (longer)              │
   TRANSMITTER ─┼──────────────────────────────┼────► ANTENNA
     │ path 1 (direct)                          │  path 3
     └──────────────────────────────────────────┘
                                │
                                ▼
       each copy arrives with amplitude a_i and phase φ_i = 2π·d_i/λ
                                │
                                ▼
              received signal = vector sum  Σ a_i·e^(jφ_i)
                                │
          ┌─────────────────────┴─────────────────────┐
          ▼                                           ▼
   similar phases: peak                     opposite phases: dip
                                                 (−20 to −30 dB)
                                │
                                ▼
   move the antenna ~0.38 λ ──► another, almost independent sum
   move at speed v          ──► the pattern is traversed in time (f_m = v/λ)
   copies spread in τ       ──► some channel frequencies in a dip, others not
```

## Limits, exceptions, and real-world cases

- **Clarke's model assumes waves from every direction.** If most copies arrive from a narrow sector, correlation falls much more slowly with distance. That is why an elevated base station, with the scatterers far away and in one direction, must space its antennas much further apart. P.1406-2 gives about 8 wavelengths horizontally, with optimum orientation, and about 17 vertically to bring correlation down to 0.7 (section 5.3).
- **With line of sight, dips are shallower.** The distribution moves from Rayleigh to Nakagami-Rice. The more the direct path dominates, the more the signal looks like a fixed value with small ripples.
- **You do not have to move to suffer it.** If the phone is still, the environment keeps changing. People walking by, doors and vehicles change the paths. Reflections from moving vehicles can cause fading even with a stationary receiver (P.1406-2, section 3.3).
- **The body matters.** Hand and head absorb and block part of the signal. That effect adds to multipath, but it is a different mechanism. Its size depends on the device and how it is held. No general figure is given here.
- **The statistics are local.** The Rayleigh distribution describes variation within an area of a few tens of wavelengths. The mean of that area in turn changes with distance and shadowing. Mixing the two scales leads to wrong conclusions.

## Practical consequences

- **Moving the device a few centimetres can change the speed a lot.** If the mean signal is weak, a 10 or 20 dB dip lowers the SNR enough for the link to pick a slower modulation. [Article 00·02](02-WhatSetsTheSpeedOfARadioLink.md) explains why.
- **That is why routers and phones have several antennas.** With two antennas more than 0.38 λ apart, in Clarke's model, the probability that both are 10 dB below the mean at the same time drops from 9.5 % to about 0.9 %. At 2.4 GHz that is less than 5 cm of spacing. It fits inside a phone.
- **That is why WiFi and 5G spread data over many subcarriers.** If some frequencies in the channel fall into a dip, the rest keep carrying data. Error-correcting codes spread each piece of data across several subcarriers and recover what is lost.
- **Measuring coverage requires averaging.** A single dBm reading can be 20 dB below the mean for that area. Serious measurements cover tens of wavelengths and take the median.
- **For device designers.** Antenna spacing, polarisation and position relative to the hand decide how much diversity is actually gained. It is one reason antenna placement is studied so carefully.

## Frequently asked questions

### Is this the same as attenuation with distance?

No. Attenuation with distance and through walls, explained in [article 00·01](01-WhyHigherFrequenciesReachLess.md), sets the mean signal level in an area. Multipath makes the signal swing around that mean over centimetres. The two effects add up.

### Are the dips energy that gets lost?

No. The energy does not disappear: it is redistributed. Where the copies subtract there is a dip, and a few centimetres away, where they add, there is a peak. Averaged over the area, the power is what the distance and obstacles dictate.

### Why does the phone show the same bars while the speed changes?

The coverage indicator usually averages received power over some time. Fast dips last milliseconds and are hidden in that average. The link, on the other hand, reacts to them when choosing modulation and coding.

### Is multipath always bad?

No. Systems with several antennas at both ends (MIMO) take advantage of the paths being different. With many reflections, they can send several data streams at once over the same channel. Without multipath, much of that gain disappears. The capacity limit of MIMO is covered in [article 00·02](02-WhatSetsTheSpeedOfARadioLink.md).

## Sources

- R. H. Clarke. "A Statistical Theory of Mobile-Radio Reception". *The Bell System Technical Journal*, vol. 47, no. 6, pp. 957-1000, July-August 1968. Equations 9, 16, 18 and 21. Scanned full text: <https://archive.org/details/bstj47-6-957>
- ITU-R. *Recommendation ITU-R P.1057-7: Probability distributions relevant to radiowave propagation modelling* (08/2022). Sections 5 (Rayleigh) and 7 (Nakagami-Rice). <https://www.itu.int/dms_pubrec/itu-r/rec/p/R-REC-P.1057-7-202208-I!!PDF-E.pdf>
- ITU-R. *Recommendation ITU-R P.1406-2: Propagation effects relating to terrestrial land mobile and broadcasting services in the VHF and UHF bands* (07/2015). Sections 3.2, 3.3, 4.1 and 5.3. <https://www.itu.int/dms_pubrec/itu-r/rec/p/R-REC-P.1406-2-201507-I!!PDF-E.pdf>
- ITU-R. *Recommendation ITU-R P.1238-13: Propagation data and prediction methods for the planning of indoor radiocommunication systems and radio local area networks in the frequency range from 300 MHz to 450 GHz* (09/2025). Section 4 and table 7. <https://www.itu.int/dms_pubrec/itu-r/rec/p/R-REC-P.1238-13-202509-I!!PDF-E.pdf>
