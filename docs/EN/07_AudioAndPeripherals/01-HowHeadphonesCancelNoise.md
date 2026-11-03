---
title: "How can headphones remove noise by playing more sound?"
section: "07_AudioAndPeripherals"
language: "en"
status: "published"
order: 1
depth: "intermediate"
summary: "Active noise-cancelling headphones measure the noise with microphones and play a wave of the same shape and opposite sign through their speaker, so the two subtract inside the ear. It only works well in the bass: there the waves are long and slow, and errors in timing and position matter little. The treble is stopped by the ear cushions, not by the electronics."
prerequisites:
  - "What frequency and wavelength are (reviewed in article 00·01)"
  - "Decibels as a ratio between two powers"
translation: "../../ES/07_AudioYPerifericos/01-ComoQuitaRuidoUnAuricular.md"
---

# How can headphones remove noise by playing more sound?

## Question and framing

Switch on noise cancelling and the drone of the plane or the bus almost disappears. Yet the voices around you are still audible. It looks like a contradiction: the headphones do not block the ear any better, they add sound through their speaker. How can adding sound result in more silence? And why does it work on some noises and not on others?

This article explains the physical principle behind active noise control (ANC) and the three architectures headphones use: feedforward, feedback and hybrid. It also explains why the effect is concentrated in the low frequencies.

- **Depth:** intermediate. Some trigonometry appears with numbers, without derivations.
- **Prerequisites:** frequency and wavelength, reviewed in [Why does a higher-frequency radio signal reach a shorter distance?](../00_Foundations/01-WhyHigherFrequenciesReachLess.md). The relation λ = c/f holds for sound too, with the speed of sound in place of the speed of light. Decibels as a ratio between two powers.
- **Out of scope:** microphone noise reduction during calls, transparency mode, noise cancelling in cars and rooms, and comparisons between commercial models.

## Short answer

The headphones measure the noise and play an inverted copy that subtracts from it inside the ear; the subtraction only works when the wave is long and slow enough for the copy to arrive on time and in the right place, and that happens in the bass.

### 60-second summary

Sound is a variation in air pressure. Two variations meeting at the same point add up. If one pushes while the other pulls, with the same strength, the result is close to zero. This is destructive interference, and Paul Lueg patented its use for silencing noise in 1936.

ANC headphones carry microphones and a processor. An outer microphone hears the noise before it gets in. An inner one hears what is left next to the eardrum. The processor works out the wave the speaker must play to cancel that remainder, and keeps adjusting it.

The subtraction needs precision. A phase error of only 18° limits the attenuation to 10 dB. At 100 Hz, 18° is half a millisecond of margin. At 1 kHz, it is only 50 microseconds. That is why the electronics win in the bass and lose in the treble. The treble is stopped by the cushion material, which insulates better the higher the frequency.

## Mental model

Picture a swing that someone pushes at a steady rhythm. To stop it, you push the other way with the same force, right on each swing. If the swing is slow, timing your push is easy. If it swung ten times per second, you would be late for every push. You would end up pushing at the wrong moment and make it swing even more.

The swing is the air pressure next to the eardrum. Whoever pushes it is the noise. You are the headphone speaker. Being late is the delay of the microphones, the processor and the speaker itself.

**Where the analogy breaks.** The swing is in a single place. Sound is not: the pressure differs at every point in space. The opposite wave only cancels the noise in a small zone around the point where it is tuned. Outside that zone, the two waves may even add up. Also, real noise is not a regular back-and-forth: it mixes many frequencies at once, and all of them have to be matched.

## Technical mechanism

### 1. What happens in the system

The cycle of ANC headphones has five steps, repeated thousands of times per second:

1. **The noise reaches the headphones.** Part of it gets through the shell and the cushions. On the way it is attenuated and distorted. That route, from outside to the ear, is called the *primary path*.
2. **The microphones measure it.** A reference microphone on the outside picks up the noise before it gets in. An error microphone inside the cavity picks up what actually arrives near the ear.
3. **The processor computes the anti-noise.** A digital filter turns the microphone signal into the signal that must go to the speaker.
4. **The speaker plays it.** The emitted wave travels its own route to the error microphone. That route is called the *secondary path*.
5. **The remainder is measured and the filter is corrected.** The error microphone hears the sum of noise and anti-noise. An adaptive algorithm adjusts the filter to make that sum as small as possible.

### 2. What each component does

**Superposition.** In air, at normal levels, the pressures of two waves at a point add up. If the noise is p and the anti-noise is −p, the result is zero. In practice, the anti-noise is never exact. If it has the right amplitude but arrives with a phase error φ, the remainder is 2·sin(φ/2) times the original noise. With perfect amplitude and perfect phase, cancellation would be total. Any error leaves a remainder.

**The secondary path.** A lot sits between the signal the processor computes and what the error microphone hears: the amplifier, the speaker, the air in the cavity and the microphone itself. Each one delays and colours the signal. Rivera Benois, Nowak and Zölzer define this path as the characteristics of the speaker and the error microphone, plus the acoustic path between them. The processor needs a model of it to know what will actually sound when it sends a signal.

**The FxLMS algorithm.** It is the most widely used adaptive algorithm in ANC, according to the survey by Lu and co-authors (2021). It adjusts the filter coefficients step by step, in the direction that reduces the error. Its key feature is that it filters the reference signal through the secondary-path model before using it. Hence the "x" (*filtered-x*). Without that step, the algorithm would correct as if its output reached the ear instantly, and it would steer the wrong way. If the secondary-path model differs from the real one, convergence slows down and the system can become unstable.

**The three architectures.**

- **Feedforward.** Uses the outer microphone. It has one advantage: it hears the noise before the ear does. It has one strict condition: causality. The processor delay plus the secondary-path delay must be less than or equal to the primary-path delay (Rivera Benois *et al.*). If the anti-noise arrives after the noise, it can no longer cancel it. Also, it only cancels noise that the outer microphone picks up the same way the ear does.
- **Feedback.** Uses only the inner microphone. It needs no advance knowledge of the noise: it corrects what it hears. Its limit is the loop delay. According to Rivera Benois *et al.*, that delay limits the bandwidth over which it attenuates. That is why early versions were analogue: they avoided the delay of analogue-to-digital and digital-to-analogue conversion.
- **Hybrid.** Adds the outputs of the two. The feedforward part handles the noise it can anticipate. The feedback part picks up what is left.

**The waterbed effect.** A feedback system cannot attenuate at all frequencies at once. If it reduces noise in one part of the spectrum, it increases it in another (Lu *et al.*, 2021). The design chooses where to concentrate the attenuation and where to accept a small increase.

**Passive insulation.** The shell, the foam and the fit against the ear stop sound without electronics. According to Rivera Benois, Papantoni and Zölzer (2018), this passive attenuation is effective at mid and high frequencies. Low frequencies are treated with ANC. The two complement each other.

### 3. Key parameters

**Required precision.** Own calculation from the remainder formula, 2·sin(φ/2), with exact amplitude and a single tone:

| Target attenuation | Maximum phase error | Time margin at 100 Hz | Time margin at 1 kHz |
|---|---|---|---|
| 10 dB | 18.2° | 505 µs | 51 µs |
| 20 dB | 5.7° | 159 µs | 16 µs |
| 30 dB | 1.8° | 50 µs | 5 µs |

The same timing error is a larger phase error the higher the frequency. Example: at a 48 kHz sampling rate, one sample lasts 20.8 µs. A one-sample error is a phase error of 0.75° at 100 Hz and 7.5° at 1 kHz. With that error alone, the attenuation ceiling would be about 38 dB at 100 Hz and about 18 dB at 1 kHz.

**Size of the zone of quiet.** Picture a diffuse sound field in which the pressure is cancelled at one point. The zone where the noise falls by at least 10 dB is about one tenth of a wavelength across (Elliott *et al.*, 1988). Rafaely refines this to 0.088λ for a 300 Hz tone. With a speed of sound of about 343 m/s in air at 20 °C, that gives:

| Frequency | Wavelength | −10 dB zone (≈ λ/10) |
|---|---|---|
| 100 Hz | 3.43 m | 34 cm |
| 300 Hz | 1.14 m | 11 cm |
| 1 kHz | 34 cm | 3.4 cm |
| 3 kHz | 11 cm | 1.1 cm |

These figures are for a free, diffuse field, not a closed cavity. They serve the idea: at 100 Hz the zone covers the whole ear with room to spare. At 3 kHz it is smaller than the ear canal, and the position of the error microphone relative to the eardrum starts to matter.

**A measured example.** Rivera Benois, Papantoni and Zölzer (2018) measured a headphone prototype on an artificial head. They used broadband white noise amplitude-modulated at 70 Hz and sampled at 48 kHz. The overall levels at the ear were:

| Condition | Level at the ear |
|---|---|
| No headphones | 82.3 dB SPL |
| Headphones on, ANC off (passive insulation only) | 65.3 dB SPL |
| Classic feedforward ANC | 61.6 dB SPL |
| Hybrid ANC (with psychoacoustic weighting) | 60.1 dB SPL |

In that test, the passive part contributes about 17 dB and the active part about 5 dB more in overall level. Overall figures hide the split by frequency. The feedforward controller attenuated roughly between 60 Hz and 7.5 kHz, and amplified below 60 Hz. All three controllers showed an amplification peak around 6.3 kHz. The feedback controller in the hybrid system attenuates effectively up to 1 kHz. These are data from a research prototype, not from a commercial product.

### 4. Diagram or flow

```text
                     PRIMARY PATH (shell + cushion)
OUTSIDE NOISE ───────────────────────────────────────────────┐
     │                                                       ▼
     ▼                                               ┌───────────────┐
 REFERENCE MIC ──► FILTER W_ff ──┐                   │  EAR CAVITY   │
 (outside)                       │                   │               │
                                 ▼                   │               │
                               ( + ) ──► SPEAKER ──► anti-noise      │
                                 ▲    SECONDARY PATH                 │
                                 │                   │ noise + anti  │
                    FILTER W_fb ◄┴── ERROR MIC ◄─────────────────────┘
                         ▲           (inside)              │
                         │                                 │
                FxLMS ALGORITHM ◄── uses model Ŝ ◄─────────┘
                (adjusts W_ff and W_fb to minimise the remainder)

Feedforward condition: delay(W_ff) + delay(secondary) ≤ delay(primary)
```

## Limits, exceptions, and real-world cases

- **Fit changes both paths.** If the cushion does not seal well, more noise leaks in and the secondary path changes. The model Ŝ no longer matches reality, and cancellation gets worse. That is why adaptive systems keep re-tuning during use.
- **Noise that cannot be anticipated.** The feedforward part only cancels what its outer microphone picks up in a similar way to how it reaches the ear. A sharp knock, or wind blowing over the microphone itself, does not meet that condition.
- **Causality sets a hard limit.** Sound travels 1 cm in about 29 µs. If the outer microphone is, say, 2 cm from the speaker, the noise takes about 58 µs to cross that distance. That is less than three samples at 48 kHz to measure, compute and play. The distance is an illustrative assumption, not a figure from any model.
- **Some amplification is unavoidable.** The waterbed effect means paying at some frequency for what is gained at another. In the measured prototype, the cost fell below 60 Hz and around 6.3 kHz.
- **The zone of quiet is local.** Cancellation is optimised at the error microphone, not at the eardrum. In the bass the difference hardly matters. At mid and high frequencies, the few centimetres between the two points are already a noticeable fraction of the wavelength.

## Practical consequences

- **Drones yes, voices less.** Engine, aircraft and air-conditioning noise concentrates a lot of energy in the bass and is fairly steady. That is exactly what ANC cancels best. Speech changes quickly, and much of what makes it intelligible lies in the mid frequencies, above the zone where ANC performs best. What is removed from voices is mostly due to passive insulation.
- **Passive insulation is still half the job.** In the test cited, the shell and cushions contributed more overall reduction than the electronics. Poorly fitted headphones lose both at once.
- **For designers.** Every microsecond of delay saved in converters, filters and speaker widens the frequency range that can be cancelled. That is why the first feedback controllers were analogue: they avoided conversion delay (Rivera Benois *et al.*).

## Frequently asked questions

### Is this the same as microphone noise reduction during calls?

No. Call noise reduction cleans up the microphone signal before sending it to the other person. It does not change what you hear. ANC acts on the sound reaching your ear and plays a real wave through the speaker.

### Do the headphones "absorb" the noise?

No. The noise still arrives. The speaker creates a second pressure that subtracts from it inside the cavity. The noise energy is not destroyed: inside the zone of quiet the two waves offset each other, and outside it they are distributed differently.

### Why not cancel the treble too with a faster processor?

Processor speed is only part of the delay. The speaker, the microphones and the travel time of the sound also count. And even with zero delay, the zone of quiet at several kilohertz is a few millimetres across. It would have to coincide with the eardrum, which shifts with every ear and every fit. For the treble, passive insulation is more effective and cheaper.

### Is this a new idea?

No. In 1936 Paul Lueg patented the idea of picking up a sound with a microphone and playing it back in opposite phase through a loudspeaker. What has changed is adaptive digital processing, which allows the anti-noise to be adjusted in real time.

## Sources

- P. Lueg. *Process of silencing sound oscillations*. US Patent 2,043,416, filed 8 March 1934, granted 9 June 1936. <https://patents.google.com/patent/US2043416A/en>
- S. M. Kuo and D. R. Morgan. "Active Noise Control: A Tutorial Review". *Proceedings of the IEEE*, vol. 87, no. 6, 1999. DOI: [10.1109/5.763310](https://doi.org/10.1109/5.763310). Record: <https://nokia.com/bell-labs/publications-and-media/publications/active-noise-control-a-tutorial-review>
- L. Lu, K.-L. Yin, R. C. de Lamare, Z. Zheng, Y. Yu, X. Yang and B. Chen. "A survey on active noise control techniques, Part I: Linear systems". Preprint arXiv:2110.00531, 2021. <https://arxiv.org/abs/2110.00531>
- P. Rivera Benois, P. Nowak and U. Zölzer. "Hybrid Active Noise Control Structures: A Short Overview". Helmut Schmidt University, Hamburg. <https://www.hsu-hh.de/ant/wp-content/uploads/sites/699/2019/01/Hybrid-Active-Noise-Control-Structures_-A-Short-Overview-Rivera-Nowak-Zoelzer.pdf>
- P. Rivera Benois, V. Papantoni and U. Zölzer. "Psychoacoustic Hybrid Active Noise Control Structure for Application in Headphones". *25th International Congress on Sound and Vibration (ICSV25)*, Hiroshima, 2018. <https://www.hsu-hh.de/ant/wp-content/uploads/sites/699/2018/08/ICSV25_full_paper_Rivera_Papantoni_Zoelzer.pdf>
- S. J. Elliott, P. Joseph and P. A. Nelson. "Active control in diffuse sound fields". *Proceedings of the Institute of Acoustics*, vol. 10, part 2, pp. 605-614, 1988. <https://www.ioa.org.uk/system/files/publications/SJ%20ELLIOTT%20P%20JOSEPH%20PA%20NELSON%20ACTIVE%20CONTROL%20IN%20DIFFUSE%20SOUND%20FIELDS.pdf>
- B. Rafaely. "Zones of quiet in a broadband diffuse sound field". arXiv:2310.04191, 2023. <https://arxiv.org/abs/2310.04191>
