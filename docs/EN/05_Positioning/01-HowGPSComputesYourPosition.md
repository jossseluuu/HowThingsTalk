---
title: "How does a GPS receiver work out its position from satellite signals?"
section: "05_Positioning"
language: "en"
status: "published"
order: 1
depth: "intermediate"
summary: "Each GPS satellite broadcasts its time and its orbit. The receiver measures how long those signals take to arrive, turns them into distances and solves for four unknowns at once: three coordinates and the error of its own clock. That is why it needs at least four satellites, not three."
prerequisites:
  - "Frequency, wavelength and decibels (article 00·01)"
  - "Signal-to-noise ratio (article 00·02)"
  - "Multipath (article 00·03)"
translation: "../../ES/05_Geolocalizacion/01-ComoCalculaElGPSTuPosicion.md"
---

# How does a GPS receiver work out its position from satellite signals?

## Question and framing

A phone or a watch with GPS does not send anything to the satellites. It only listens. Even so, within seconds it knows where it is to within a few metres. The question is how it turns some very weak radio signals into a position.

This article explains the basic calculation inside a civilian GPS receiver: what each satellite transmits, how the travel time of the signal is measured and why four satellites are needed. It does not cover how a phone finds itself with GPS switched off (via WiFi or the cellular network), the help the network gives GPS to start faster, or centimetre-level techniques. Relativity is mentioned but not developed. Other satellite systems (Galileo, GLONASS, BeiDou) follow the same principle, but the figures here are for GPS.

It helps to know decibels ([article 00·01](../00_Foundations/01-WhyHigherFrequenciesReachLess.md)) and the signal-to-noise ratio ([article 00·02](../00_Foundations/02-WhatSetsTheSpeedOfARadioLink.md)). The depth is intermediate: there is one equation, explained step by step.

## Short answer

The receiver measures how long each satellite's signal takes to arrive, turns that into a distance and finds the one point and the one clock correction that fit at least four of those distances at the same time.

### 60-second summary

Every GPS satellite carries atomic clocks. It broadcasts a signal that effectively says "I am satellite N, this is my exact time and this is my orbit". The signal travels at the speed of light and takes about 70 to 85 milliseconds to reach the ground.

The receiver compares the time carried by the signal with the time on its own clock. The difference, multiplied by the speed of light, gives a distance. If the receiver's clock were perfect, three distances would be enough to pin down a point. But its clock is cheap quartz and can be off by microseconds or more. One microsecond of error is about 300 metres.

The fix is to treat that clock error as one more unknown. With four satellites there are four equations and four unknowns: latitude, longitude, height and clock offset. The receiver solves them by successive approximation. With more satellites, it averages and throws out bad measurements.

## Mental model

Picture yourself in a dark field. Several church towers around the horizon strike the hour exactly, and you know where each one stands. Sound takes time to travel: if you hear the twelve o'clock bell 3 seconds late, that tower is about 1,000 metres away. With several towers you can work out where you are.

Now suppose your wristwatch runs a little fast and you do not know by how much. Every distance will be wrong by the same amount. Even so, with one extra tower you can work out both where you are and how far off your watch is. Only one combination makes every measurement fit.

The analogy breaks in three places. The "towers" move at almost 4 km/s, so the receiver must know where each one was at the exact moment of transmission. The signal is not heard above the noise: it arrives below it and must be pulled out with a known code. And the "bell" is not a single chime: it is a continuous pattern that the receiver lines up with its own copy to measure the delay.

## Technical mechanism

### 1. What happens in the system

The system has three parts. The space segment is the satellites. The control segment is the ground stations that track orbits and clocks and upload corrections. The user segment is the receivers, which only receive.

According to gps.gov, the United States commits to keeping at least 24 operational satellites available 95% of the time. They fly in medium Earth orbit, at about 20,200 km, spread across six orbital planes. According to the US Coast Guard navigation center (NAVCEN), the orbits are circular, with a 55° inclination and an 11 hour 58 minute period.

A position fix follows these steps:

1. **Acquisition.** The receiver searches for which satellites are in the sky. For each one it tries code delays and Doppler shifts until a correlation stands out above the noise.
2. **Tracking.** Once locked, it keeps its copy of the code aligned with the received signal. That alignment is the time measurement.
3. **Reading the navigation message.** It decodes the data the satellite sends at 50 bit/s: its clock, its corrections and its precise orbit (the ephemeris).
4. **Pseudoranges.** For each satellite it computes (reception time − transmission time) × c.
5. **Solution.** With four or more pseudoranges and the satellite positions, it solves for position and clock offset.

### 2. What each component does

**The satellite clocks.** They are the reference for the whole system. The control segment measures their drift and broadcasts it as correction parameters inside the navigation message. The receiver applies that correction before computing anything.

**The C/A code.** Each satellite modulates the L1 carrier (1575.42 MHz) with its own pseudorandom code of 1,023 chips that repeats every millisecond. It runs at 1.023 million chips per second. The codes of different satellites look very little alike. That lets them all share one frequency: the receiver separates each satellite by multiplying the signal by a copy of its code.

**The navigation message.** It rides on top of the code at 50 bit/s. According to the SPS performance standard, it is organised into frames of five subframes, and each frame repeats every 30 s. It contains the clock corrections, the satellite's own ephemeris and an almanac, which is a rough description of the whole constellation. The ephemeris is used to compute where the satellite was when it transmitted.

**The receiver's correlator.** It generates a replica of the code and slides it in time. When the replica matches the received signal, the correlation peaks. Where that peak falls within the code's millisecond measures the delay to a fraction of a chip. One chip lasts 1/1.023 MHz ≈ 0.98 µs, which is about 293 m of travel (calculated). That is why the receiver has to interpolate within the chip to get down to metres.

**The receiver's clock.** It is a quartz oscillator. It is not accurate, but it is fairly stable for the duration of the calculation. Its error is the same for every satellite measured at the same instant. That property is what makes it possible to solve for it.

**The navigation algorithm.** It solves the system of equations, usually with iterative least squares or with a filter that combines successive measurements.

### 3. Key parameters

**The equation.** For each satellite *i*, with known position (xᵢ, yᵢ, zᵢ):

```text
ρᵢ = √[(xᵢ − x)² + (yᵢ − y)² + (zᵢ − z)²] + c·b
```

- ρᵢ is the measured pseudorange, in metres.
- (x, y, z) is the receiver position: three unknowns.
- b is the receiver clock offset, in seconds: the fourth unknown.
- c = 299,792,458 m/s, the speed of light (exact by SI definition).

It is called a *pseudo*range because it includes the c·b term. It is not the true range until b is known. With four satellites there are four equations and one useful solution. The receiver starts from a rough position, linearises the system, corrects and repeats. It usually converges in a few iterations.

**Times and distances (calculated from the 20,200 km altitude and a mean Earth radius of 6,371 km).**

| Situation | Range to satellite | Travel time |
|---|---|---|
| Satellite at zenith | 20,200 km | 67 ms |
| Satellite on the horizon | about 25,800 km | 86 ms |
| 1 µs clock error | — | equals 300 m |
| 1 ns clock error | — | equals 30 cm |

**How weak the signal is.** The SPS standard guarantees a minimum received power of −158.5 dBW for the L1 C/A code. That is about 1.4 × 10⁻¹⁶ W (calculated). Thermal noise at 290 K is about −204 dBW per hertz. Across the roughly 2 MHz the signal occupies, noise adds up to about −141 dBW (calculated, not counting the receiver's own noise). The signal sits about 17 dB below the noise.

**Why it can still be recovered.** The code runs at 1.023 Mchip/s and the data at 50 bit/s. That is 20,460 chips per bit, about 43 dB (calculated). By correlating with the right code, the receiver gathers the energy of each bit while the noise stays spread out. That processing gain turns a signal buried in noise into a usable measurement. It is the same signal-to-noise trade-off as in [article 00·02](../00_Foundations/02-WhatSetsTheSpeedOfARadioLink.md), paid for with a tiny data rate.

**Accuracy achieved.** The 2020 SPS standard states that, with 2018 signal accuracy, well-designed receivers were achieving 3 m or better horizontally and 5 m or better vertically 95% of the time. This is observed performance, not the guaranteed minimum. The standard's guaranteed values could not be verified for this article.

**Geometry.** The same measurement quality gives worse positions if the satellites are bunched in one part of the sky. The spheres intersect at shallow angles, and a small range error moves the intersection a lot. Receivers express this effect as dilution of precision (DOP): a number that multiplies range error to give position error. The more spread out the satellites, the lower it is.

### 4. Diagram or flow

```text
 Satellite i (atomic clock + ephemeris)
   │  L1 = 1575.42 MHz
   │  own C/A code (1023 chips, 1 ms) + data at 50 bit/s
   ▼
 67–86 ms trip  ──►  arrives at −158.5 dBW, ~17 dB below noise
   │
   ▼
 Receiver
   ├─ Correlator: slides the code replica ─► peak = measured delay
   ├─ Decodes the message ─► transmit time + satellite position
   └─ Its quartz clock ─► reception time (with unknown offset b)
   │
   ▼
 ρᵢ = (t_reception − t_transmission) × c      (for each satellite)
   │
   ▼
 4 or more equations  ─►  iterative least squares
   │
   ▼
 (x, y, z) + b   ─►  position and, as a bonus, the exact time
```

## Limits, exceptions, and real-world cases

**The ionosphere and the troposphere.** The signal does not always travel at c. Crossing the ionosphere delays it according to the electron density, which changes with time of day and solar activity. The troposphere adds another delay, which depends on pressure and humidity. A single-frequency receiver corrects the ionosphere with an approximate model that the satellite itself broadcasts. Whatever the model misses remains as error. Dual-frequency receivers can measure this delay, because it depends on frequency.

**Multipath.** In a city the signal bounces off building fronts. If only the reflection arrives, the receiver measures a longer path than the real one. If both arrive, the correlation peak is distorted and the measured delay shifts. It is the same phenomenon as in [article 00·03](../00_Foundations/03-WhySignalChangesOverAFewCentimetres.md), but here it does not just change power: it falsifies the distance. That is why the position "jumps" across the street in narrow roads.

**Indoors.** Starting from −158.5 dBW, every wall or roof takes decibels from a signal that is already below the noise. Inside a building the receiver often loses some or all satellites.

**Cold start.** With no almanac or ephemeris, the receiver must search blind and then read the navigation message at 50 bit/s. One full frame takes 30 s. That is why a standalone receiver with no outside help can take tens of seconds to give its first position. Phones cut that time sharply by downloading this data over the network, a mechanism outside the scope of this article.

**Satellite clocks and relativity.** Clocks in orbit do not tick at the same rate as on the ground, because of the satellite's speed and the difference in gravity. The system compensates for these effects in its design and in the corrections it broadcasts. Without that compensation, the error would grow day by day. The exact figures have not been verified for this article.

**Four satellites is the minimum, not the norm.** With today's constellation there are usually many more in view under open sky. The extra satellites are used to average, to detect an inconsistent measurement and to improve geometry.

## Practical consequences

A GPS receiver is passive. It transmits nothing to the satellites, so the system supports an unlimited number of users and the satellites do not know who is listening. The privacy of your position depends on what the device does with it afterwards, not on GPS.

A receiver with a position fix also knows the time very precisely, because it solves for b. That is why GPS is used to synchronise networks, not just to navigate.

Accuracy depends on how much sky is visible. Under open sky it gives a few metres. Between tall buildings, geometry gets worse and reflections appear. Indoors, the signal may not arrive at all. That is why phones combine GPS with other sources of position.

For a device designer, the antenna and the first amplifier are critical. The signal already arrives below the noise. Every decibel of noise added by the receiver comes straight off the margin.

## Frequently asked questions

### Is GPS the same as triangulation?

Not quite. Triangulation measures angles. GPS measures distances, derived from times, and also solves for the clock error. The correct name for the principle is trilateration with a clock term.

### Why aren't three satellites enough?

They would be if the receiver had an atomic clock synchronised with the system. With a quartz clock, the unknown offset is a fourth unknown. Some receivers use three satellites if they fix the height by other means, for example at sea, but that is an added constraint, not a measurement.

### Does GPS need mobile data or the internet?

Not to work. All the information it needs comes in the satellite signal. Mobile data only speeds up the start and helps when the signal is weak.

### Do the satellites know where I am?

No. They only transmit. The whole calculation happens in the receiver.

## Sources

- [GPS.gov — Space Segment](https://www.gps.gov/systems/gps/space/). Commitment to 24 operational satellites 95% of the time, six orbital planes and an altitude of about 20,200 km.
- [U.S. Coast Guard NAVCEN — Global Positioning System Overview](https://navcen.uscg.gov/global-positioning-system-overview). Circular orbits at 20,200 km, 55° inclination and an 11 h 58 min period.
- [IS-GPS-200N — Interface Specification, NAVSTAR GPS Space Segment / Navigation User Segment Interfaces (2022)](https://www.gps.gov/technical/icwg/IS-GPS-200N.pdf). Definition of the C/A code: 1 ms Gold code of 1,023 chips at 1.023 Mchip/s (section 3.2.1.3).
- [GPS Standard Positioning Service (SPS) Performance Standard, 5th edition (2020)](https://www.gps.gov/technical/ps/2020-SPS-performance-standard.pdf). L1 frequency of 1575.42 MHz, LNAV data at 50 bit/s, frame structure, minimum received power of −158.5 dBW (section 2.2) and observed accuracy of 3 m horizontal and 5 m vertical at 95% (executive summary).
- [BIPM — The International System of Units (SI), 9th edition](https://www.bipm.org/en/publications/si-brochure). Exact value of the speed of light, c = 299,792,458 m/s.
