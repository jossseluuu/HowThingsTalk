---
title: "How does the mobile network help a phone's GPS find your position in a few seconds?"
section: "05_Positioning"
language: "en"
status: "published"
order: 3
depth: "intermediate"
summary: "A standalone GPS receiver has to search blindly for satellites and then read their orbits at 50 bit/s, which takes tens of seconds. A phone skips most of that work: the network tells it which satellites are in the sky, what Doppler they will arrive with, their orbits and the approximate time. So it searches a small window, does not need to decode the message, and can work with much weaker signals."
prerequisites:
  - "Frequency, wavelength and decibels (article 00·01)"
  - "Signal-to-noise ratio (article 00·02)"
  - "How GPS computes a position (article 05·01)"
translation: "../../ES/05_Geolocalizacion/03-ComoAyudaLaRedMovilAlGPSAArrancar.md"
---

# How does the mobile network help a phone's GPS find your position in a few seconds?

## Question and framing

A hiking GPS that has just been switched on can take a while to give its first position. A phone, by contrast, usually places you within seconds, even next to a window. Both listen to the same satellites. The question is what the phone does differently.

This article explains assisted GPS (A-GPS, or A-GNSS when several constellations are used): what a receiver starting on its own is missing, what data the network sends, how the phone uses it to search less and hear weaker signals, and who ends up doing the calculation. It does not cover positioning by WiFi or cells without GPS, which is in [article 05·02](02-HowYourPhoneFindsItselfWithoutGPS.md). Nor does it cover centimetre-level corrections (RTK, SSR), even though they travel over the same protocols.

You need the GPS principle from [article 05·01](01-HowGPSComputesYourPosition.md): C/A code, correlation, pseudoranges and the navigation message. Decibels ([article 00·01](../00_Foundations/01-WhyHigherFrequenciesReachLess.md)) and signal-to-noise ratio ([article 00·02](../00_Foundations/02-WhatSetsTheSpeedOfARadioLink.md)) also help. The depth is intermediate. The arithmetic is multiplication, marked as "calculated".

## Short answer

The network hands the phone in advance what a standalone GPS would have to work out by listening: which satellites to look for, where to look, their orbits and the time.

### 60-second summary

A receiver starting with no information does not know which satellites are overhead or what Doppler shift they arrive with. It has to try combinations of satellite, code delay and frequency until one stands out from the noise. Then it must read the satellite's own ephemeris, which arrives at 50 bit/s. That reading alone can take up to 36 s.

The phone asks a location server in the network for help. The server has reference receivers that track every satellite non-stop. It sends the approximate time, a starting position and, for each visible satellite, its expected Doppler and its orbits. With that, the phone searches a much narrower window. And it does not need to decode the navigation message: it only has to measure the code.

The second point matters most indoors. Reading data takes more signal than detecting the code. Without having to read it, the phone can get a fix with signals about 18 dB weaker than the minimum guaranteed under open sky. 3GPP tests require a position within 20 s at most at that level.

## Mental model

Picture looking for a friend in a packed stadium at night, without knowing which stand they are in. You sweep section by section with binoculars. When you finally spot them, they shout their seat number one letter at a time, very slowly, over the crowd.

Now picture them texting you: "North stand, row 12, red cap". You go straight to a small area. And you no longer need to hear them say where they are. You only have to recognise them, which is much easier than understanding a sentence through the noise.

The analogy breaks in two places. Your friend barely moves, but a GPS satellite approaches or recedes from you at hundreds of metres per second, and that shifts the frequency its signal arrives at. And the text does not tell you where your friend is right now, but describes their trajectory. The phone has to use it to work out where the satellite was at the exact instant it transmitted.

## Technical mechanism

### 1. What happens in the system

**The cold start of a standalone receiver.** With no stored memory, the receiver does not know the time, its position or which satellites are above the horizon. Three things happen in sequence.

1. **Blind search.** For every possible satellite code, it tries every code delay and a wide range of frequencies. This is the acquisition from [article 05·01](01-HowGPSComputesYourPosition.md), with no hints.
2. **Reading the message.** Once locked, it decodes the navigation message. According to the IS-GPS-200 specification, it runs at 50 bit/s in 300-bit subframes, so 6 s per subframe. The satellite clock is in subframe 1 and the ephemeris in subframes 2 and 3. All three must be received. Blay, Wang and Akos (2021) note this can take up to 36 s.
3. **Calculation.** With four or more satellites read, it solves for position and clock offset.

**The assisted start.** The phone asks for help before or alongside the search.

1. **Request.** The phone requests assistance data from a location server. In LTE the server is called the E-SMLC and the protocol is LPP (3GPP TS 37.355).
2. **Assistance.** The server sends the reference time, an approximate position, clock and orbit models for each satellite, the ionospheric model and acquisition assistance: expected Doppler and search windows.
3. **Targeted search.** The phone searches only the visible satellites, in a narrow frequency window and, if it has fine time, a narrow delay window too.
4. **Measurement.** It measures the code delay and Doppler of each satellite. It does not need to wait for the navigation message.
5. **Calculation.** Either the phone does it with the orbits it received (*UE-based* mode), or it sends the measurements to the server, which computes the fix (*UE-assisted* mode). TS 36.305 defines the two modes by which node does the calculation.

### 2. What each component does

**The reference receiver network.** TS 36.305 describes A-GNSS as relying on a continuously operating network of GNSS reference receivers. Those receivers, under open sky, do read the message from every satellite. The server then distributes that content.

**The location server.** On the LTE control plane it is the E-SMLC. It decides which method to use and can combine results from several methods. Assistance can also travel over the user plane, as ordinary IP data: that is SUPL, from the Open Mobile Alliance. In that case the server is called the SLP and the phone the SET. TS 36.305 states that LPP works over both paths.

**The reference time.** There are two grades. Coarse time has an error of seconds. In 3GPP tests (TS 36.171) it is ±2 s. Fine time ties the cell's timing to GPS time with an error of microseconds: ±10 µs in those tests. Not every phone supports it.

**The reference location.** An approximate point, usually tied to the serving cell. In 3GPP tests it is a 3 km radius circle with 500 m of altitude uncertainty.

**The navigation model.** The ephemeris and clock corrections: the same thing the satellite transmits in subframes 1 to 3. According to IS-GPS-200, in normal operation each data set is transmitted for two hours and fitted over a four-hour interval.

**The almanac.** A coarse description of the whole constellation. TS 37.355 says it is used to tell which satellites are visible and remains valid for a few weeks. In the full GPS message it takes 25 frames to receive in full: 25 × 30 s = 12.5 min (calculated from IS-GPS-200).

**Acquisition assistance.** For each satellite it includes the expected Doppler at the reference location, its uncertainty, the expected code delay, a code search window, and elevation and azimuth. Doppler is expressed as a velocity: the field spans −1024 to +1023.5 m/s in 0.5 m/s steps. The uncertainty ranges from 2.5 to 40 m/s in the basic version.

**Data bit assistance.** TS 37.355 allows sending the very bits the satellite is about to transmit. They are used to strip them out of the signal (*data wipe-off*) so the receiver can integrate for longer.

### 3. Key parameters

**Doppler and frequency.** At L1, each metre per second of relative velocity shifts the frequency by about 5.25 Hz (calculated: 1575.42 MHz ÷ c). Inside GNSS puts the maximum rate at which the range to a satellite changes at about 800 m/s. That is about ±4.2 kHz (calculated). Add the receiver oscillator error: with a 1 ppm error, ±1.6 kHz (calculated; 1 ppm is an example value).

**The size of the search (calculated example).** Assume half-chip code cells and 500 Hz frequency cells. These are illustrative values, not normative ones.

| Situation | Code cells | Frequency cells | Cells per satellite | Satellites to try |
|---|---|---|---|---|
| Cold start | 2046 (1023 chips) | about 24 (±5.8 kHz) | about 49,000 | every possible code, up to 32 |
| Assisted, coarse time | 2046 (code still unbounded) | about 8 (±40 m/s + oscillator) | about 16,000 | visible ones only |
| Assisted, fine time | about 80 (±20 chips) | about 8 | about 640 | visible ones only |

The fine-time code window comes from adding two errors. A 10 µs error is about 10 chips. A 3 km position error is at most another 10 chips, because one chip is about 293 m (calculated). With coarse time, however, the phone does not know where within the code millisecond each satellite falls. TS 37.355 in fact states that the code window covers position uncertainty, not time uncertainty.

The oscillator remains the dominant factor in the frequency column. It is often said that phones calibrate it against the cell carrier, which would shrink the search further. Unverified against a primary source for this article.

**How weak the signal can be.** The SPS standard guarantees at least −158.5 dBW at L1 under open sky, that is −128.5 dBm. The TS 36.171 sensitivity tests put seven of the eight satellites at −147 dBm. That is 18.5 dB less, about 70 times less power (calculated). At that level, the phone must give a position with a horizontal error of 100 m or less in 95% of attempts and within 20 s at most. That time is counted from the request, assuming the assistance has already arrived.

**Why reading data takes more signal than measuring the code.** To measure the delay, the accumulated correlation only has to stand out from the noise. The longer it accumulates, the more it stands out. But each data bit lasts 20 ms and can flip the sign of the signal. If the receiver does not know where those flips are, it cannot accumulate coherently beyond one bit. Inside GNSS explains that with a weak signal the bit error rate prevents reliable decoding of the time of week. With assistance, the phone does not need to read it. And with data bit assistance it can accumulate beyond 20 ms: going from 20 to 100 ms gives up to about 7 dB more signal-to-noise ratio (calculated, ideal). In practice, oscillator stability and motion cut into that gain.

**The millisecond ambiguity.** The C/A code repeats every millisecond. Measuring its phase gives the range up to a whole number of light-milliseconds, about 300 km each (calculated). A standalone receiver resolves it by reading the time from the message. An assisted one uses the reference location. Blay et al. state that the integer is resolved if the combined error is under half a light-millisecond, about 150 km. A starting position good to a few kilometres easily meets that.

**Coarse time as a fifth unknown.** With ±2 s of time error, the computed position of each satellite is wrong. Inside GNSS puts the error at up to ±1.7 km of geometric range. The solution is to add the time error as a fifth unknown, alongside the three coordinates and the clock offset. Blay et al. describe it this way. A fifth satellite is then needed, and the uncertainty of the other unknowns grows somewhat.

### 4. Diagram or flow

```text
 COLD START (standalone receiver)
   Blind search: up to 32 codes × 2046 delays × ~24 frequencies
        │
        ▼
   Lock ─► read subframes 1-3 at 50 bit/s (up to 36 s)
        │   (needs enough signal to decode bits)
        ▼
   4 satellites with ephemeris ─► position

 ASSISTED START (phone)
   Reference receivers ──► location server (E-SMLC or SLP)
   (open sky, tracking every            │
    satellite non-stop)                 │  LPP over the control plane,
                                        │  or SUPL over IP data
                                        ▼
   Phone receives: time (±2 s or ±10 µs), position (~3 km),
                   orbits and clocks, ionosphere, expected Doppler and windows
        │
        ▼
   Targeted search: visible satellites only, narrow window
        │
        ▼
   Measures code delay + Doppler (does not read the message; works at −147 dBm)
        │
        ├── UE-based:    the phone computes with the received orbits
        └── UE-assisted: sends measurements, the server computes
        ▼
   Position (3GPP test requirement: ≤ 100 m at 95%, ≤ 20 s)
```

## Limits, exceptions, and real-world cases

**Assistance does not create signal.** Underground or deep inside a large building, nothing useful may arrive, not even at −147 dBm. The phone then falls back on WiFi, cells or the last known position, as explained in [article 05·02](02-HowYourPhoneFindsItselfWithoutGPS.md).

**No data connection.** User-plane A-GPS needs IP data. Without data coverage, the phone's receiver behaves like a standalone one. If it used GPS recently, it still holds valid orbits: the ephemeris is good for about four hours and the almanac for a few weeks. That middle case is usually called a warm or hot start.

**3GPP requirements are not real-world performance.** TS 36.171 sets lab conditions: white noise, known geometry, modelled multipath. In a narrow street, real multipath can be worse. And the 20 s is a maximum, not a typical figure. Typical times for each manufacturer have not been verified for this article.

**First fixes are worse.** The sensitivity test accepts 100 m of error. The nominal accuracy test, with signals at −130 dBm, requires 30 m with GPS alone and 15 m with several constellations. The blue dot usually starts large and shrinks as the receiver tracks more satellites.

**Coarse time costs a satellite.** With the fifth unknown, five satellites are needed instead of four. With few satellites in view, as in an urban canyon, that can delay the first fix. Fine time avoids it, but both network and phone must support it.

**Stale data.** Orbits outside their fit interval or an old almanac give worse predictions. That is why the server also reports satellite status: TS 37.355 requires it to always send the current list of unhealthy signals.

## Practical consequences

**For the person using the phone.** A fast first fix depends on the network as much as on the sky. With data on, the position arrives in seconds. In airplane mode with GPS on, the phone can still locate itself, but it takes longer if it has not been used for hours.

**Power.** TS 36.305 states that assistance lets the phone use less energy than standalone GNSS. That follows: the receiver spends less time powered up searching, and does not have to track each satellite for tens of seconds to read its message.

**Emergency calls.** *UE-assisted* mode leaves the calculation to the network server. The position therefore ends up directly in the network, which is the party that needs it in services such as emergency calls. Each country's regulatory requirements are not covered here.

**Privacy.** Asking for assistance means talking to a server. On the control plane, the network already knows the phone's cell. Over the user plane, the request travels over the data connection to whichever server the system uses. What each provider keeps from those requests has not been verified for this article.

**For the designer.** A phone receiver's sensitivity is not only about the antenna. It depends on how long it can accumulate, and that depends on what the network tells it.

## Frequently asked questions

### Is this the same as positioning by WiFi or by the cell tower?

No. Here the position still comes from measuring satellites, as in [article 05·01](01-HowGPSComputesYourPosition.md). The network only supplies data to find and measure them sooner. Positioning by WiFi or cells, in [article 05·02](02-HowYourPhoneFindsItselfWithoutGPS.md), does not measure any satellite. That said, the reference location the server sends is usually a cell-based estimate.

### Does GPS need the internet to work?

No. A GPS receiver only listens. Without assistance it works the same way, but it takes longer to cold-start and needs more signal to read the navigation message.

### Does the phone send its position to the satellites?

No. The satellites receive nothing from users. In *UE-assisted* mode the phone sends measurements to a network server, not to the satellites.

### What is the difference between a cold, warm and hot start?

It depends on what the receiver remembers. In a cold start it has no time, no position and no orbits. With an almanac and approximate time, it knows which satellites to look for but must still read the ephemeris. With current ephemeris less than about four hours old, it can compute almost as soon as it locks onto the signal. Assistance amounts to receiving a hot start all at once over the network.

## Sources

- [U.S. Space Force — IS-GPS-200N, NAVSTAR GPS Space Segment / Navigation User Segment Interfaces (2022)](https://www.gps.gov/technical/icwg/IS-GPS-200N.pdf). C/A code of 1023 chips at 1.023 Mchip/s and 1 ms (3.2.1.3); message at 50 bit/s, 300-bit 6 s subframes, 25 pages in subframes 4 and 5 (20.3.1–20.3.3); clock and ephemeris in subframes 1 to 3; two-hour transmission and four-hour fit interval (20.3.4.4).
- [GPS Standard Positioning Service (SPS) Performance Standard, 5th edition (2020)](https://www.gps.gov/technical/ps/2020-SPS-performance-standard.pdf). Minimum received power of −158.5 dBW for the C/A code at L1.
- [3GPP TS 36.305 V17.0.0, Stage 2 functional specification of UE positioning in E-UTRAN (ETSI TS 136 305)](https://www.etsi.org/deliver/etsi_ts/136300_136399/136305/17.00.00_60/ts_136305v170000p.pdf). Definition of *UE-based* and *UE-assisted* modes (3.1, 8.1.1); continuously operating reference receiver network; benefits of assistance for start-up time, sensitivity, power and accuracy (8.1.1); assistance data types (8.1.2.1); role of the E-SMLC and use of LPP with SUPL (5.3.3, 6.2.1).
- [3GPP TS 37.355 V17.0.0, LTE Positioning Protocol (LPP) (ETSI TS 137 355)](https://www.etsi.org/deliver/etsi_ts/137300_137399/137355/17.00.00_60/ts_137355v170000p.pdf). GNSS assistance elements (6.5.2): reference time and location, ionospheric model, navigation model, almanac, real-time integrity, data bit assistance and acquisition assistance (Doppler from −1024 to +1023.5 m/s, uncertainty from 2.5 to 40 m/s, code window).
- [3GPP TS 36.171 V18.0.0, Requirements for support of A-GNSS (ETSI TS 136 171)](https://www.etsi.org/deliver/etsi_TS/136100_136199/136171/18.00.00_60/ts_136171v180000p.pdf). Sensitivity tests at −147 dBm with coarse (±2 s) and fine (±10 µs) time: 100 m at 95% within 20 s (5.1); nominal accuracy at −130 dBm: 30 m (5.2) and 15 m multi-constellation (6.2); 3 km reference location (Annex B.1.4).
- [R. Blay, B. Wang and D. M. Akos, "Deriving Accurate Time from Assisted GNSS Using Extended Ambiguity Resolution", NAVIGATION 68(1), 217–229 (2021)](https://navi.ion.org/content/68/1/217). Up to 36 s to receive subframes 1 to 3; millisecond ambiguity resolved with an error under half a light-millisecond (about 150 km); five-unknown solution with coarse time.
- [Inside GNSS, "Coarse Time Positioning" (2012)](https://insidegnss.com/coarse-time-positioning/). Time of week as an additional unknown; ±2 s of error equals up to ±1.7 km; maximum rate of about 800 m/s in the range to a satellite; decoding failure with weak signals due to bit error rate.
