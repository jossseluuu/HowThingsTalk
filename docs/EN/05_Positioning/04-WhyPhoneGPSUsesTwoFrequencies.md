---
title: "Why do recent phones listen to GPS on two frequencies?"
section: "05_Positioning"
language: "en"
status: "published"
order: 4
depth: "intermediate"
summary: "The ionosphere delays satellite signals, and the delay depends on frequency. A phone that receives the same satellite on L1 (1575.42 MHz) and L5 (1176.45 MHz) can measure that delay instead of estimating it with a model. On top of that, the L5 code is ten times faster than the L1 code, so reflections off buildings distort the distance less."
prerequisites:
  - "Frequency, wavelength and decibels (article 00·01)"
  - "Multipath (article 00·03)"
  - "How GPS computes a position (article 05·01)"
translation: "../../ES/05_Geolocalizacion/04-PorQueElGPSDelMovilUsaDosFrecuencias.md"
---

# Why do recent phones listen to GPS on two frequencies?

## Question and framing

For years, a phone's GPS listened to a single civil signal: L1, at 1575.42 MHz. Today many phones also receive a second frequency, L5, at 1176.45 MHz. The question is what a receiver gains by listening to the same satellite twice.

This article explains two mechanisms. The first is how two frequencies make it possible to measure the delay added by the ionosphere. The second is why the L5 signal copes better with reflections. It does not cover centimetre-level techniques based on carrier phase (RTK, PPP), even though they also rely on several frequencies. Nor does it cover network assistance at start-up, which is in [article 05·03](03-HowTheNetworkHelpsGPSFindYouFast.md).

You need the principle from [article 05·01](01-HowGPSComputesYourPosition.md): C/A code, correlation and pseudoranges. Decibels ([article 00·01](../00_Foundations/01-WhyHigherFrequenciesReachLess.md)) and multipath ([article 00·03](../00_Foundations/03-WhySignalChangesOverAFewCentimetres.md)) also help. The depth is intermediate. There is one simple formula, explained step by step. Our own arithmetic is marked as "calculated".

## Short answer

Because the ionosphere delays the lower frequency more than the higher one, comparing the two measurements of the same satellite reveals that delay and lets the receiver remove it, while the ten-times-faster L5 code makes reflections distort the measurement less.

### 60-second summary

The receiver works out the distance to each satellite from the signal's travel time. But the signal does not always travel at the speed of light. Crossing the ionosphere, a layer of ionised gas hundreds of kilometres up, slows it down. That delay changes with the time of day, the location and solar activity. With a single frequency, the receiver can only estimate it with a model broadcast by the satellites. According to the specification, that model removes at least half of the error.

The ionospheric delay is inversely proportional to the square of the frequency. L5 is delayed about 1.79 times more than L1. If the receiver measures the pseudorange on both, the difference between them can only come from the ionosphere. From it, the receiver computes the delay and subtracts it.

The second advantage is the code. On L5 each chip lasts one tenth as long as on L1. The correlation peak is ten times narrower. A reflection off a building that arrives late separates from the true peak sooner and distorts it less. A study of the first dual-frequency phone measured a multipath error about three times smaller on L5 than on L1.

It is not all upside. Combining two measurements adds the noise of both. And not every GPS satellite broadcasts L5 yet.

## Mental model

Imagine measuring a table in a workshop with two rulers, one steel and one aluminium. The workshop is at a temperature you do not know. Both rulers have expanded, so both readings are wrong. But you know aluminium expands almost twice as much as steel. The difference between the two readings tells you how much the temperature has distorted them. From that you can work out the true length without a thermometer.

Two frequencies do the same thing. The ionosphere is the unknown temperature. L1 and L5 are the two rulers, and physics says exactly how much more L5 "expands": a factor of 1.79.

The analogy breaks in three places. First, the ionosphere is not the same for every satellite: each signal crosses a different patch of sky, so the calculation is done satellite by satellite. Second, each reading has its own noise, and subtracting two similar readings makes that noise grow; with well-engraved rulers it would hardly matter. Third, the analogy misses the other half of the story: L5 is not just another frequency, it is a better-designed signal with a faster code.

## Technical mechanism

### 1. What happens in the system

A modern GPS satellite transmits on several frequencies at once. All of them come from the same clock and carry synchronised codes. A dual-frequency phone follows this process for every satellite that broadcasts L5:

1. **Tracking in two bands.** The receiver locks onto the C/A code on L1 and the L5 code of the same satellite. It needs two radio chains, because the frequencies are almost 400 MHz apart.
2. **Two pseudoranges.** It gets one pseudorange on each frequency. Both contain the same geometric distance, the same clock error and the same tropospheric delay. They differ only in the ionospheric delay and in hardware delays.
3. **Hardware delay correction.** It applies the group delay corrections the satellite broadcasts in its message (TGD and ISC). They compensate for each signal taking a slightly different time to leave the satellite's electronics.
4. **Removing the ionosphere.** It combines the two pseudoranges so that the ionospheric term cancels.
5. **Solution.** It uses those corrected measurements in the usual position calculation, with four or more satellites.

Galileo broadcasts its E5a signal on the same frequency as L5, 1176.45 MHz, and its E1 signal on the same frequency as L1. So the same dual-band hardware serves both constellations.

### 2. What each component does

**The ionosphere.** It is the part of the atmosphere where solar radiation strips electrons from atoms. According to ESA (Navipedia), the peak density is around 10¹¹ to 10¹² electrons per cubic metre, between 300 and 500 km up. What matters for GPS is the total electron content along the signal path (STEC). It is measured in TECU: 1 TECU is 10¹⁶ electrons per square metre.

**The delay.** According to the same source, the code delay in metres is:

```text
I(f) = 40.3 × STEC / f²
```

- I(f) is the code delay, in metres.
- STEC is the electron content of the path, in electrons/m².
- f is the signal frequency, in hertz.

The formula captures more than 99.9% of the effect. Higher-order terms are typically less than 0.1% of the total delay. A curiosity: the code is delayed, but the carrier phase is advanced by the same amount.

**The L5 signal.** According to the IS-GPS-705 specification, its carrier is at 1176.45 MHz, which is 115 times the system's 10.23 MHz fundamental frequency. It occupies a 24 MHz band. It has two components in quadrature. I5 carries navigation data at 50 bit/s, with error-correction coding. Q5 carries no data: it is a pilot signal. Each component uses a 10,230-chip code that repeats every millisecond, at 10.23 million chips per second. The GPS service describes it as a signal with higher power and greater bandwidth, broadcast in a band reserved for aviation safety services.

**The pilot component.** With no data on top, the receiver does not have to guess where a bit flips sign. It can accumulate the signal for longer and track it more steadily. The specification gives no figure for this gain, so it is not quantified here.

**Group delay corrections.** The satellite's electronics do not delay every signal equally. The modern navigation message (CNAV) broadcasts TGD and the inter-signal corrections ISC_L5I5 and ISC_L5Q5. IS-GPS-705 states they are used both by single-frequency L5 users and by dual-frequency L1/L5 users. The same specification allows a mean group delay differential of up to 30 ns. That is about 9 m (calculated). Left uncorrected, the combination would carry that error.

**The ionosphere-free combination.** Call P1 and P5 the pseudoranges on L1 and L5, already corrected for hardware delays. If the delay on L1 is I1, on L5 it is γ·I1, with:

```text
γ = (f_L1 / f_L5)² = (1575.42 / 1176.45)² = (154/115)² ≈ 1.793
```

Then:

```text
P1 = ρ + I1
P5 = ρ + γ·I1

Ionospheric delay on L1:      I1   = (P5 − P1) / (γ − 1)
Ionosphere-free measurement:  P_IF = (γ·P1 − P5) / (γ − 1) ≈ 2.261·P1 − 1.261·P5
```

Here ρ groups everything that is the same on both frequencies: distance, clock and troposphere. The value of γ appears verbatim in IS-GPS-705. ESA describes the same combination for frequency pairs and notes that it removes the first-order effect.

**The L5 correlator.** It works like the L1 correlator in [article 05·01](01-HowGPSComputesYourPosition.md), but with chips ten times shorter. An L5 chip lasts about 97.8 ns, equivalent to about 29.3 m of travel. On L1 it was about 293 m (calculated).

### 3. Key parameters

**The two signals, side by side.**

| Parameter | L1 C/A | L5 |
|---|---|---|
| Frequency | 1575.42 MHz | 1176.45 MHz |
| Wavelength | 19.0 cm (calculated) | 25.5 cm (calculated) |
| Code rate | 1.023 Mchip/s | 10.23 Mchip/s |
| Code length | 1023 chips, 1 ms | 10,230 chips, 1 ms |
| Distance per chip | about 293 m (calculated) | about 29.3 m (calculated) |
| Occupied band | about 2 MHz | 24 MHz |
| Minimum received power | −158.5 dBW | −157.9 dBW per component (Block IIF); −157.0 dBW (GPS III) |
| Dataless component | no | yes (Q5) |
| Ionospheric delay per TECU | 0.162 m (calculated) | 0.291 m (calculated) |

The two L5 components together add 3 dB over a single one. That gives about −154.9 dBW for Block IIF and −154.0 dBW for GPS III (calculated). In other words, 3.6 to 4.5 dB more than the L1 C/A at the specified minimum.

**A numerical example.** Take a path with 50 TECU, a value chosen only for illustration. The delay is about 8.1 m on L1 and about 14.6 m on L5. The difference between the two pseudoranges is about 6.4 m. Divided by γ − 1 ≈ 0.793, it gives back the 8.1 m on L1 (calculated). According to ESA, the single-frequency delay can reach several tens of metres. The model broadcast by the satellites removes at least 50% of the RMS error, according to IS-GPS-705. What remains after the combination are the higher-order terms: in this example, less than 1 cm (calculated with ESA's 0.1%).

**The price: noise.** The combination multiplies P1 by 2.261 and P5 by 1.261. If both measurements have the same random noise σ, the result has a noise of about 2.6·σ (calculated). The receiver trades a large systematic error for a larger random one. It pays off when the ionospheric error exceeds that noise, or when the noise is reduced by averaging over time.

**Multipath.** A rough rule from correlator theory: a reflection that arrives more than one chip late barely distorts the main peak. On L1 that requires a path about 293 m longer. On L5, about 29 m is enough (calculated). In a street, most reflections off building fronts travel tens of metres extra, not hundreds. The study by Robustelli and colleagues (2019) measured it on a static phone for one hour, at a site with little multipath:

| Signal | Standard deviation of code multipath |
|---|---|
| GPS L1 | 6.11 m |
| GPS L5 | 2.09 m |
| Galileo E1 | 3.77 m |
| Galileo E5a | 1.21 m |

In the same test, the Galileo-only position had a horizontal error of 13.50 m with E1 and 4.57 m with E5a.

### 4. Diagram or flow

```text
 Satellite (a single clock)
   ├── L1 = 1575.42 MHz   C/A code, 1.023 Mchip/s
   └── L5 = 1176.45 MHz   L5 code, 10.23 Mchip/s (I5 data + Q5 pilot)
          │
          ▼
 Ionosphere: delay = 40.3·STEC/f²
   L1 delayed by I1        L5 delayed by 1.793·I1
          │
          ▼
 Phone: two radio chains, two correlators
   P1 = ρ + I1             P5 = ρ + 1.793·I1
          │
          ├─ subtract TGD/ISC (satellite hardware delays)
          ▼
 I1   = (P5 − P1) / 0.793           ─► delay measured, not modelled
 P_IF = 2.261·P1 − 1.261·P5         ─► ionosphere-free measurement (noise ×2.6)
          │
          ▼
 Position solution with 4 or more satellites (article 05·01)
```

## Limits, exceptions, and real-world cases

**L5 is not complete yet.** The GPS service states that L5 remains pre-operational and is used at the user's own risk until it is declared operational. It also warns that the new signals are of limited use until 18 to 24 satellites broadcast them. Older satellites only broadcast L1 C/A for civil use. In practice, the phone mixes satellites with one and with two frequencies. Galileo E5a helps fill the gap.

**The troposphere does not go away.** The lower atmosphere also delays the signal, but almost equally on both frequencies. The combination does not remove it; it still has to be modelled.

**Multipath is not cancelled, it is amplified.** The ionosphere-free combination carries the multipath of both measurements, multiplied by their coefficients. L5's resistance to reflections is better exploited by using L5 directly than by mixing it with a more contaminated L1.

**A phone is not a survey receiver.** In the Xiaomi Mi 8 study, the antenna was linearly polarised, which makes it more sensitive to reflections. L1/E1 signals arrived considerably stronger than L5/E5a, even though L5 is transmitted with more power. The chip also listened for 200 ms and rested for 800 ms to save battery. The figures in that study come from one 2018 phone, at two sites with one day each.

**What each chip does exactly is not public.** A manufacturer may use L5 to measure the ionosphere, to reduce multipath, or both, and combine it with other sensors. The specific algorithms of commercial chips could not be verified for this article.

**Two frequencies are not centimetre accuracy.** Code measurements still have errors of metres from noise, reflections and geometry. Centimetres require measuring the carrier phase, resolving its ambiguity and using external corrections. That is outside this article.

## Practical consequences

In a city, the most visible benefit usually comes from multipath. A reflection that shifts the L1 measurement by tens of metres matters much less on L5. The position tends to jump across the street less often.

The ionosphere matters most where and when it is most charged: near the equator, around midday and in solar-maximum years. There a single-frequency receiver depends on a model that only corrects part of the error. A dual-frequency one measures it.

Since Android 10, phones must expose raw GNSS measurements to apps. Multi-frequency support is optional and depends on the chip. According to Android's documentation, most flagship phones from the major manufacturers already support L1 and L5. These measurements let scientific and surveying apps process both frequencies themselves.

For the device designer, dual frequency has a cost. The antenna has to work well in two bands almost 400 MHz apart. A second radio chain is needed, and a 24 MHz signal must be processed instead of about 2 MHz. The extra power draw depends on the chip and is not quantified here.

## Frequently asked questions

### Is this the same as using several constellations?

No. Using GPS, Galileo, GLONASS and BeiDou together gives more satellites and better geometry, but each can still be single-frequency. Dual frequency means receiving the same satellite in two bands. A phone can have one without the other, although recent ones usually have both.

### Why L5 and not L2?

The civil L2C signal, at 1227 MHz, also allows ionospheric correction together with L1, according to the GPS service. L5 has a faster code, more bandwidth and a band protected for aviation. It also shares its frequency with Galileo E5a. Each manufacturer's specific reasons for choosing L5 are not documented in the sources consulted.

### Does L5 get through better inside buildings?

That cannot be stated in general. L5 is transmitted with slightly more power and its frequency is slightly lower. But in the Mi 8 study the phone received L5 weaker than L1, probably because of the antenna. Indoors, both remain signals far below the noise.

### If the phone measures the ionosphere, does it no longer need the model the satellite broadcasts?

For satellites it receives on two frequencies, it does not need it for the ionospheric delay. For those it only receives on L1, it still uses it. And with weak signals or a lot of noise, the model can work better than a noisy combination.

## Sources

- [IS-GPS-705J — Interface Specification, NAVSTAR GPS Space Segment / User Segment L5 Interfaces (2022)](https://www.gps.gov/technical/icwg/IS-GPS-705J.pdf). 1176.45 MHz carrier and 24 MHz band (3.3.1.1); 10,230-chip code at 10.23 Mchip/s (3.2.1.1, 3.3.2.2); I5 component with 50 bit/s data and dataless Q5 (3.2.2); minimum received power (Table 3-III); group delay differential (3.3.1.7.2); TGD, ISC and L1/L5 ionospheric correction with γ = (154/115)² (20.3.3.3.1); single-frequency model with at least a 50% error reduction (20.3.3.3.1.3).
- [IS-GPS-200N — Interface Specification, NAVSTAR GPS Space Segment / Navigation User Segment Interfaces (2022)](https://www.gps.gov/technical/icwg/IS-GPS-200N.pdf). C/A code of 1023 chips at 1.023 Mchip/s (3.2.1.3).
- [GPS Standard Positioning Service (SPS) Performance Standard, 5th edition (2020)](https://www.gps.gov/technical/ps/2020-SPS-performance-standard.pdf). L1 frequency and minimum received power of −158.5 dBW for C/A (section 2.2).
- [GPS.gov — New Civil Signals](https://gps.gov/systems/gps/modernization/civilsignals). L5 in a band reserved for aviation safety, with higher power and bandwidth; L5 and L2C pre-operational; ionospheric correction with dual frequency; limited use until 18 to 24 satellites broadcast them.
- [European GNSS Service Centre — Galileo Open Service Signal-In-Space ICD, issue 2.1 (2023)](https://www.gsc-europa.eu/sites/default/files/sites/all/files/Galileo_OS_SIS_ICD_v2.1.pdf). E1 (1575.420 MHz) and E5a (1176.450 MHz) carriers (Table 2) and E5a code rate of 10.230 Mchip/s (Table 5).
- [ESA Navipedia — Ionospheric Delay](https://gssc.esa.int/navipedia/index.php/Ionospheric_Delay). The 40.3·STEC/f² formula, TECU definition, code delay and phase advance, higher-order terms below 0.1%, and delays of up to several tens of metres.
- [ESA Navipedia — Ionosphere-free Combination for Dual Frequency Receivers](https://gssc.esa.int/navipedia/index.php/Ionosphere-free_Combination_for_Dual_Frequency_Receivers). Code and phase combination that removes the first-order ionospheric effect.
- [ESA Navipedia — Klobuchar Ionospheric Model](https://gssc.esa.int/navipedia/index.php/Klobuchar_Ionospheric_Model). Single-frequency model broadcast by GPS; reduction of about 50% of the RMS error.
- [U. Robustelli, V. Baiocchi and G. Pugliano, "Assessment of Dual Frequency GNSS Observations from a Xiaomi Mi 8 Android Smartphone and Positioning Performance Analysis", Electronics 8(1), 91 (2019)](https://www.mdpi.com/2079-9292/8/1/91). Broadcom BCM47755 chip; code multipath on L1/L5 and E1/E5a; position with E1 versus E5a; 200/800 ms duty cycle and linearly polarised antenna.
- [Android Developers — Raw GNSS measurements](https://developer.android.com/develop/sensors-and-location/sensors/gnss). Raw measurements mandatory since Android 10; multi-frequency support optional; most flagship phones support L1 and L5.
