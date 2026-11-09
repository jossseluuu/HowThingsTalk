---
title: "How does a phone microphone turn sound into numbers?"
section: "07_AudioAndPeripherals"
language: "en"
status: "published"
order: 2
depth: "intermediate"
summary: "A phone microphone is a tiny capacitor: a half-millimetre membrane that moves towards and away from a fixed, perforated plate. That change in capacitance becomes a voltage, and a modulator turns it into a stream of ones and zeros at about 3 MHz whose density follows the wave. A digital filter averages that stream and delivers ordinary audio samples, for example 48,000 per second."
prerequisites:
  - "What a sound wave is: a variation in air pressure"
  - "Decibels as a ratio between two quantities"
translation: "../../ES/07_AudioYPerifericos/02-ComoConvierteElMicrofonoElSonidoEnNumeros.md"
---

# How does a phone microphone turn sound into numbers?

## Question and framing

When you speak on the phone, your voice is air pushing and pulling. What arrives at the other end is a list of numbers. In between sits a part smaller than a grain of rice. What happens inside it? How does a pressure that changes continuously become bits?

This article explains the MEMS (*microelectromechanical system*) microphone, the type used in phones, laptops and earbuds. It covers two stages: how the membrane turns pressure into an electrical signal, and how that signal becomes a bit stream. It focuses on digital microphones with PDM output, the most common kind in these devices.

- **Depth:** intermediate. There are a couple of simple formulas and some worked numbers, without derivations.
- **Prerequisites:** knowing that sound is a pressure variation, and handling decibels as a ratio between two quantities. If you have read [How can headphones remove noise by playing more sound?](01-HowHeadphonesCancelNoise.md), you already have the idea of acoustic pressure; it is not essential.
- **Out of scope:** combining several microphones to focus on a voice (*beamforming*), call noise reduction, the codecs that compress the audio afterwards, and comparisons between commercial models.

## Short answer

Sound pressure moves a membrane that forms a capacitor with a fixed plate; the change in capacitance becomes a voltage, a sigma-delta modulator turns it into a stream of single bits whose share of ones follows the wave, and a digital filter averages that stream into audio samples.

### 60-second summary

A 94 dB SPL sound, which is very loud, is a pressure of just 1 Pa. That is one hundred-thousandth of atmospheric pressure. The microphone has to detect that and much less.

To do so it uses a capacitor. A silicon membrane roughly half a millimetre across is separated from a fixed plate by a few microns of air. The plate has holes so the air can move. When pressure pushes the membrane, the distance changes and so does the capacitance. With a bias voltage applied, this produces an output voltage proportional to the motion.

A chip next to the membrane amplifies that voltage and digitizes it in an unusual way. It does not produce 16-bit numbers, but one bit at a time, about 3 million times per second. When pressure is high, more ones come out. When it is low, more zeros. This is pulse density modulation (PDM).

The device's processor averages that stream and keeps one sample in every 64. With a 3.072 MHz clock, that gives 48,000 samples per second in the usual audio format.

## Mental model

Think of a photo printed in a newspaper. There is only black ink and white paper. Yet you see greys. Dark areas have many dots; light areas have few. Your eye cannot resolve each dot: it averages a small patch and perceives a tone.

The PDM stream does the same thing in time. Each bit is a black or white dot. The density of ones over a short interval marks the level of the wave at that moment. The digital filter plays the role of the eye: it averages and rebuilds the "grey", which here is the pressure value.

**Where the analogy breaks down.** In the newspaper, dot density is chosen by looking at the tone of each patch on its own. In the microphone, each bit is chosen by looking at the error accumulated by the previous bits. That feedback loop is what pushes quantization error up to very high frequencies, far from the audio. The analogy also says nothing about the first stage: how pressure becomes voltage. The membrane does that.

## Technical mechanism

### 1. What happens in the system

The path of sound inside the microphone has five steps:

1. **Sound enters through a hole.** The package has an acoustic port that carries the pressure to the membrane.
2. **The membrane moves.** Pressure displaces it relative to a fixed plate. Together they form a capacitor.
3. **The change in capacitance becomes voltage.** A bias voltage keeps charge on the capacitor. As the distance changes, the voltage between the plates changes.
4. **The chip amplifies and modulates it.** An integrated circuit (ASIC) in the same package amplifies the signal and turns it into a 1-bit stream.
5. **The receiver filters and decimates it.** The device's processor averages the stream with a low-pass filter and keeps one sample in every N. The result is PCM audio at 16 bits or more.

### 2. What each component does

**The membrane and the plate.** According to the review by Zawawi and co-authors (2020), a MEMS capacitive microphone has a movable membrane and a fixed backplate, separated by an air gap. An insulator acts as the spacer. Capacitance follows the parallel-plate formula:

C = ε₀ · A / g

Here A is the membrane area and g the gap between the plates. If the membrane moves closer, g drops and C rises. The sizes are small. The same review cites membranes of about 0.5 mm diameter in the Knowles microphones of the iPhone 4, 1 mm in an Infineon model and 0.59 mm in an Analog Devices one. The research designs it compiles have air gaps between 2 and 3.6 µm.

Our own calculation, with illustrative values: a circular membrane 0.5 mm across, 2 µm from the plate, forms a capacitor of about 0.87 pF. That is a tiny amount. A few centimetres of trace or cable would add a comparable stray capacitance. The amplifier sits inside the same package, millimetres from the membrane.

**Bias and the electrical signal.** The capacitor needs charge to produce a signal. Zawawi *et al.* write the sensitivity as the product of two factors:

S = (V_b / g) · (Δg / ΔP)

The first factor is electrical: how much voltage comes out per micron of motion. It depends on the bias voltage V_b and on the gap g. The second is mechanical: how far the membrane moves per pascal. It depends on its size, its mechanical stress and its thickness. In a modern digital microphone, the bias voltage is generated by a charge pump inside the ASIC. The block diagram of the Infineon IM68D128BV01 shows it as a block of its own. Classic *electret* microphones, by contrast, hold a permanent charge in a special material (Zawawi *et al.*).

**The holes in the plate.** A very thin film of air sits between the membrane and the plate. If it could not escape, it would act as a damper and slow the membrane, especially at high frequencies. Sensitivity would drop sharply. The plate is perforated so air can flow in and out of the gap. More holes and a larger gap reduce that damping (Zawawi *et al.*).

**The 1-bit modulator.** The ASIC does not use a converter that measures the voltage with 16 bits at once. It uses a single-bit modulator that runs very fast. MT-022, the Analog Devices tutorial on sigma-delta converters, describes its structure: an integrator, a comparator and a 1-bit digital-to-analog converter in a loop. In each cycle, the loop compares the input with what it has already output. If it has fallen short, it outputs a one. If it has overshot, a zero. The average output voltage is set by the share of ones in the stream. In the Infineon datasheet, positive pressure increases the density of ones, and a regular alternation of ones and zeros represents zero.

**Noise shaping.** A single bit is a very coarse quantization. An ideal N-bit converter has a signal-to-noise ratio of 6.02·N + 1.76 dB (MT-022). With N = 1, that is about 7.8 dB. The fix has two parts:

- **Oversampling.** Sampling much faster than necessary spreads quantization error over a wider band. Each doubling of the rate lowers the share that falls in the audio band by about 3 dB. This follows from the noise model in MT-022; the tutorial does not state the figure explicitly.
- **Shaping.** The integrator loop pushes the error up to high frequencies, out of the useful band. According to MT-022, the signal-to-noise ratio improves by 9 dB per doubling of oversampling with a first-order loop and by 15 dB with a second-order loop. Its example: with an oversampling ratio of 64, an ideal second-order system reaches about 80 dB, equivalent to about 13 effective bits.

**The decimation filter.** The receiver gets millions of bits per second with a lot of noise outside the audio band. A digital low-pass filter removes it and averages the bits into higher-resolution numbers. Then the surplus samples are discarded: one in every M is kept. MT-022 calls this decimation. M can be any integer, as long as the output rate stays above twice the signal bandwidth.

**Two microphones on one wire.** STMicroelectronics application note AN5027 explains how two microphones share the same data line. Each has a channel-select pin. One drives its bit while the clock is high and leaves the line in high impedance while it is low. The other does the opposite. The receiver then separates the interleaved bits. Two microphones thus share one clock and a single data line.

### 3. Key parameters

**The pressure scale.** The reference for measuring microphones is a 1 kHz tone at 94 dB SPL, equivalent to 1 Pa (Lewis, 2012; AN-1112). Some values, our own calculation from the 20 µPa reference implied by that equivalence:

| Level | Pressure | Fraction of atmospheric pressure (101,325 Pa) |
|---|---|---|
| 29 dB SPL | 0.56 mPa | 6 × 10⁻⁹ |
| 60 dB SPL | 0.02 Pa | 2 × 10⁻⁷ |
| 94 dB SPL | 1 Pa | 1 × 10⁻⁵ |
| 120 dB SPL | 20 Pa | 2 × 10⁻⁴ |

**Sensitivity.** For an analog microphone it is given in dBV relative to 1 V/Pa. According to Lewis, most analog *electret* and MEMS microphones fall between −46 and −35 dBV, that is, between 5.0 and 17.8 mV/Pa. A typical example is −38 dBV, about 12.6 mV/Pa (our calculation). For a digital microphone it is given in dBFS: what fraction of digital full scale a 94 dB SPL tone produces. In Lewis's example, full scale corresponds to 120 dB SPL and sensitivity is 120 − 94 = −26 dBFS. That is why, in a digital microphone, changing the internal gain also changes the maximum level.

**Noise, overload and dynamic range.** InvenSense application note AN-1112 defines these quantities:

- **SNR:** the difference between the 94 dB SPL reference and the microphone's own noise, A-weighted over a 20 kHz band. It includes noise from the membrane and from the ASIC.
- **Equivalent input noise (EIN):** the noise expressed as a sound. It equals 94 dB − SNR. With an SNR of 65 dB, the EIN is 29 dB SPL.
- **Acoustic overload point (AOP):** the level at which total harmonic distortion reaches 10%. Beyond it, the signal clips.
- **Dynamic range:** AOP − EIN. In the AN-1112 example, 120 − 29 = 91 dB.

**A specific microphone.** The datasheet of the Infineon IM68D128BV01 (V1.1, April 2025) gives these figures:

| Parameter | Value |
|---|---|
| Package size | 3.5 × 2.65 × 0.98 mm |
| Sensitivity | −37 dBFS typical (1 kHz, 94 dB SPL) |
| SNR, normal mode | 67.5 dB(A), 20 Hz-20 kHz, 3.072 MHz clock |
| AOP | 128 dB SPL at 10% distortion |
| Low-frequency roll-off (−3 dB) | 20 Hz |
| Resonance | 36 kHz |
| Current, normal mode | 580 µA at 1.8 V, about 1.0 mW (our calculation) |
| Current, low-power mode | 190 µA at 768 kHz, about 0.34 mW, with a 20 Hz-8 kHz band |

**Clock and decimation.** AN5027 gives the basic relation: PDM frequency = audio sampling frequency × decimation factor. The factor is usually between 48 and 128. Its examples oversample by 64 to obtain audio at 16 or 48 kHz. With a factor of 64, a 3.072 MHz clock gives 48 kHz (our calculation). The ST library delivers 16-bit PCM. The data rate thus drops from 3.072 Mbit/s to 768 kbit/s, a quarter (our calculation).

### 4. Diagram or flow

```text
  SOUND PRESSURE (e.g. 1 Pa at 94 dB SPL)
        │  acoustic port
        ▼
 ┌──────────────────────────────────────────── PACKAGE ≈ 3.5 × 2.65 mm ─┐
 │                                                                      │
 │  MEMBRANE ~0.5-1 mm ─┐                                               │
 │     ↕ gap ~2-4 µm    ├─ capacitor C = ε₀·A/g  (≈ 1 pF)               │
 │  PERFORATED PLATE ───┘        │                                      │
 │        ▲                      ▼                                      │
 │  charge pump (V_b)        AMPLIFIER                                  │
 │                               │                                      │
 │                               ▼                                      │
 │                    1-BIT SIGMA-DELTA MODULATOR                       │
 │                  input ─► (−) ─► integrator ─► comparator ─┬─► bit   │
 │                            ▲                               │         │
 │                            └──── 1-bit DAC ◄───────────────┘         │
 └──────────────────────────────────────────────────┬───────────────────┘
                                                    │ PDM DATA: 1011011101101...
                         CLOCK 3.072 MHz ──────────►│ (density of ones ≈ pressure)
                                                    ▼
                         DEVICE PROCESSOR: low-pass filter (e.g. sinc)
                                                    │  keep 1 in every 64
                                                    ▼
                                    16-bit PCM · 48,000 samples/s
```

## Limits, exceptions, and real-world cases

- **The microphone's own noise sets the floor.** The ideal modulator in the MT-022 example would reach about 80 dB of SNR. The complete Infineon microphone reaches 67.5 dB(A). According to AN-1112, SNR includes noise from the membrane and the ASIC, not only from the conversion. A sound below the EIN, about 29 dB SPL in the AN-1112 example, is buried in that noise.
- **Loud sounds clip.** Above the AOP, distortion exceeds 10%. In the Infineon model, distortion is 0.3% at 94 dB SPL, 1% at 122 dB SPL and 10% at 128 dB SPL. A very loud, very close sound can exceed that limit.
- **The response is not flat at the extremes.** Below the low-frequency roll-off (20 Hz in the model cited), sensitivity drops. AN-1112 describes that roll-off as first order, 6 dB per octave, and the high-frequency one as second order, 12 dB per octave. The 36 kHz resonance lies outside the audio band.
- **The modulator is not linear above second order.** MT-022 warns that loops above second order do not follow the simple linear model. Designers have to watch their stability.
- **Sensitivity has a tolerance.** AN-1112 cites tolerances of ±1 to ±3 dB between units. The Infineon model specifies −38 to −36 dBFS. In a system with several microphones, that difference shows if it is not calibrated.
- **Start-up takes time.** The Infineon microphone outputs a zero pattern (alternating ones and zeros) within 3 ms of receiving power and clock. Its sensitivity is not within ±1 dB until 25 ms later with clocks of 768 kHz or more.

## Practical consequences

- **A phone can carry several microphones.** They are cheap to integrate: a package a few millimetres across and one shared line per pair. Zawawi *et al.*, citing IHS Markit, note that a phone may use up to eight.
- **Low-power mode exists for a reason.** Dropping the clock from 3.072 MHz to 768 kHz cuts consumption from about 1.0 mW to about 0.34 mW in the Infineon model. In exchange, the useful band drops to 8 kHz. That is enough for speech, not for music.
- **Recordings of quiet rooms sound like hiss.** The microphone's own noise is equivalent to a sound of about 29 to 33 dB SPL in the Lewis and AN-1112 examples. In a very quiet room, that noise becomes audible when the gain is raised.
- **For designers.** In a digital microphone, sensitivity and maximum level are tied together: full scale is fixed inside the chip (Lewis). You cannot gain headroom for loud sounds without losing level on quiet ones. And the processor has to set aside resources for each channel's decimation filter.

## Frequently asked questions

### Is a MEMS microphone the same as an *electret* microphone?

Both are capacitors: a membrane facing a plate. The difference is the charge. An *electret* holds a permanent charge in a special material (Zawawi *et al.*). A silicon MEMS microphone such as the Infineon one gets its bias voltage from a charge pump on its chip. MEMS microphones are also built with semiconductor micromachining techniques (Zawawi *et al.*).

### If the clock runs at 3 MHz, does it record ultrasound?

Not in the final output. The high clock rate serves to push quantization noise out of the audio band, not to capture more bandwidth. The decimation filter removes everything above the useful band. With a 48 kHz output, the highest frequency that can be represented is 24 kHz.

### Why not use a 16-bit converter directly?

There are microphones with other digital outputs: Lewis cites one with an I²S output. The 1-bit approach keeps the microphone's analog part simple: an integrator, a comparator and a single-bit converter. Oversampling also relaxes the analog anti-aliasing filter (MT-022). The complex part, the digital filter, moves to the device's processor.

### What does "−26 dBFS" mean on a microphone datasheet?

That a 94 dB SPL tone produces a digital output 26 dB below the maximum the microphone can represent. In Lewis's example, that maximum corresponds to about 120 dB SPL. It is a measure relative to digital full scale, not a voltage.

## Sources

- S. A. Zawawi, A. A. Hamzah, B. Y. Majlis and F. Mohd-Yasin. "A Review of MEMS Capacitive Microphones". *Micromachines*, vol. 11, no. 5, 484, 2020. DOI: [10.3390/mi11050484](https://doi.org/10.3390/mi11050484). <https://www.mdpi.com/2072-666X/11/5/484>
- J. Lewis. "Understanding Microphone Sensitivity". *Analog Dialogue*, vol. 46, no. 2, Analog Devices, May 2012. <https://www.analog.com/en/resources/analog-dialogue/articles/understanding-microphone-sensitivity.html>
- InvenSense. *AN-1112: Microphone Specifications Explained*. Revision 1.1, June 2016. <https://www.mouser.lt/pdfDocs/AN-1112-v11-3.pdf>
- W. Kester. *MT-022: ADC Architectures III: Sigma-Delta ADC Basics*. Analog Devices, Rev. A, October 2008. <https://www.analog.com/media/en/training-seminars/tutorials/MT-022.pdf>
- STMicroelectronics. *AN5027: Interfacing PDM digital microphones using STM32 MCUs and MPUs*. Rev. 2, July 2019. <https://www.st.com/resource/en/application_note/an5027-interfacing-pdm-digital-microphones-using-stm32-mcus-and-mpus-stmicroelectronics.pdf>
- Infineon Technologies. *IM68D128BV01: XENSIV MEMS microphone, datasheet*. V1.1, 14 April 2025. <https://www.mouser.com/catalog/specsheets/Infineon_04-30-2025_DS_IM68D128BV01_V1.1.pdf>
