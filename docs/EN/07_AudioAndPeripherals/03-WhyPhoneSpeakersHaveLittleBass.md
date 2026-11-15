---
title: "Why does a phone speaker produce so little bass?"
section: "07_AudioAndPeripherals"
language: "en"
status: "published"
order: 3
depth: "intermediate"
summary: "The sound that reaches your ear depends on how much air the speaker pushes and how fast that push changes. Every octave down demands four times more diaphragm travel to stay equally loud. A one-square-centimetre diaphragm that can only move a few tenths of a millimetre runs out of travel long before it reaches the bass. On top of that, its resonance sits near 1 kHz, and below it the response drops steeply."
prerequisites:
  - "What a sound wave is: a variation in air pressure"
  - "What frequency and an octave are (doubling or halving the frequency)"
  - "Decibels as a ratio between two quantities"
translation: "../../ES/07_AudioYPerifericos/03-PorQueElAltavozDelMovilApenasDaGraves.md"
---

# Why does a phone speaker produce so little bass?

## Question and framing

You play music on your phone's speaker and it sounds "tinny". Voices are clear. The kick drum and the bass line almost vanish. Is that a quality problem, or is there something physical in the way?

This article explains why a speaker the size of a fingernail cannot reproduce bass at any real level. It covers three things: how it turns current into motion, how much air must move to make a low note, and what limits that motion. It ends with what "smart" amplifiers do to squeeze out a little more.

- **Depth:** intermediate. There are two simple formulas and some worked numbers of our own, without long derivations.
- **Prerequisites:** knowing that sound is a pressure variation, what an octave is, and how decibels work. The article [How does a phone microphone turn sound into numbers?](02-HowAPhoneMicrophoneTurnsSoundIntoNumbers.md) covers the reverse path and helps, but it is not required.
- **Out of scope:** digital-to-analogue converters and the amplifier stage in detail, Bluetooth audio, psychoacoustic tricks that fake bass, headphones, and comparisons between commercial models.

## Short answer

To produce the same sound pressure, the diaphragm must move four times further for every octave you go down, and a one-square-centimetre diaphragm with only a few tenths of a millimetre of travel runs out of room below 1 kHz, which is exactly where its resonance sits.

### 60-second summary

A speaker is a piston. A coil inside a magnet receives the current and pushes a diaphragm. The diaphragm pushes the air.

What you hear at a distance does not depend on how far the diaphragm moves. It depends on the acceleration of the volume of air it displaces. Halve the frequency and the same travel gives a quarter of the acceleration. To make up for it, the travel must grow fourfold per octave.

A phone speaker has a diaphragm of about 1 cm². Its maximum travel is a few tenths of a millimetre: 0.4 mm in the units of a 2020 study. In round numbers, a 1 kHz tone at 80 dB at half a metre needs about 0.4 mm. At 250 Hz it would need 6 mm. At 100 Hz, almost 4 cm.

There is a second brake. The diaphragm and its suspension form a mass on a spring, with a resonance near 1 kHz. Below it, the displacement stops growing as the frequency falls, and the pressure drops by about 12 dB per octave.

Smart amplifiers track the coil temperature and the diaphragm travel in real time. That lets them drive the speaker up to its real limit without breaking it. They gain a few decibels. They do not change the physics.

## Mental model

Picture making waves in a swimming pool with a hand-held board. Move the board back and forth quickly and a few centimetres are enough for small, fast ripples. For long, slow waves you have to push much more water on every stroke. With a small board, that means a much longer stroke.

Now picture the board tied down with a short elastic band. It can vibrate quickly. But as soon as you try a slow, wide stroke, the band goes tight and stops you.

The phone speaker is that small board on a short band. It has plenty for the treble. For the bass it would need travel it does not have.

**Where the analogy breaks.** In the pool, water barely compresses and the waves travel along the surface. Sound is a pressure wave that spreads through the air in three dimensions. What counts is not the volume of water moved but how fast the air flow changes: its acceleration. That is why the bass penalty is so harsh, four times the travel per octave rather than just double. The elastic band does not explain resonance either: the diaphragm's mass and its suspension's stiffness form a system that vibrates most easily at one particular frequency.

## Technical mechanism

### 1. What happens in the system

The path from signal to air has five steps:

1. **The processor delivers audio samples.** A digital-to-analogue converter turns them into a voltage.
2. **The amplifier applies that voltage to the speaker.** In phones it is usually a class-D amplifier with a boost converter, because the battery supplies only a few volts (TI, SSZT848).
3. **The current creates a force.** The coil sits in a magnet's field. The current through it produces a force F = Bl · i (TI, SLAA857).
4. **The diaphragm moves.** The coil is glued to the diaphragm. How it moves depends on the force, its mass, friction and the stiffness of the suspension.
5. **The diaphragm pushes the air.** The air flow it displaces creates the pressure wave that reaches the ear.

### 2. What each component does

**The motor: magnet and coil.** The speaker in the study by Park, Jiang and Hwang (2020) measures 16.0 × 10.0 × 2.11 mm and has a 6.0 Ω coil. It uses three neodymium magnets, a centre diaphragm and a side diaphragm that acts as the suspension. The product Bl, called the force factor, gives how many newtons the motor produces per ampere. In a speaker this size it is small. Klippel's microspeaker training material puts it below 1 N/A. The iPhone 7 speaker that TI uses as an example has Bl = 0.841 T·m (SLAA857). Park *et al.* put it this way: shrinking the speaker forces a smaller magnetic circuit, and that lowers the force at all frequencies.

**The diaphragm and suspension.** These are the moving parts. A microspeaker has no spider, the corrugated part that centres the coil in a large speaker. The rim of the diaphragm itself acts as the suspension (Klippel). Klippel describes the radiating surface as roughly 1 cm in diameter.

**The mass-spring system.** Park *et al.* write the diaphragm's equation of motion:

M · z'' + C · z' + K · z = F(t)

Here z is the displacement, M the moving mass, C the mechanical damping and K the suspension stiffness. It is the same system as a weight hanging from a spring. It has a resonance frequency:

f₀ = 1 / (2π · √(M_ms · C_ms))

where C_ms is the compliance, the inverse of stiffness (SLAA857). With the iPhone 7 speaker's values, M_ms = 0.116 g and C_ms = 0.21 mm/N, this gives f₀ = 1020 Hz. The microspeakers in a study from OPPO and Realme engineers (Peng *et al.*, 2020) resonate at 700 ± 100 Hz. Klippel gives a general range of 200 Hz to 4 kHz.

**The back cavity.** The speaker is mounted in a small sealed cavity. Without it, the wave leaving the back of the diaphragm would have the opposite sign and cancel the front wave in the bass. The Analog Devices note gives this as the reason for enclosing the speaker. The price is that the air in the cavity acts as another spring. According to Klippel, air adds mass to the diaphragm and, inside an enclosure, adds stiffness. More stiffness means a higher resonance. In a phone that cavity is a few tenths of a cubic centimetre: Park *et al.* measure their prototypes with 0.6 cm³.

**From diaphragm to ear.** This is the key step. Garrett (*Understanding Acoustics*, 2020) shows that the pressure radiated by a small source depends on its air flow, which he calls the volume velocity U. In the far field, the pressure amplitude at distance r is (Eq. 12.21):

|p| = ρ · c · k · |U| / (4π · r)

Since k = ω / c, that is ρ · ω · |U| / (4π · r). And the flow is the diaphragm area times its velocity: U = S_d · ω · X, where X is the displacement amplitude. Substituting (our own calculation):

|p| = ρ · ω² · S_d · X / (4π · r)

Pressure rises with the **square** of frequency. At half the frequency, the same displacement gives a quarter of the pressure: 12 dB less. To hold the level, X must quadruple every octave. Garrett sums up the consequence in that chapter: low frequencies are harder to radiate.

**Why the response falls below resonance.** Putting the two pieces together gives the shape of the response (our own reasoning from the equations in Park *et al.* and Garrett):

- **Above f₀** mass dominates. Displacement falls as 1/f² with rising frequency, and pressure, which rises as f², stays flat. This is the useful range.
- **Below f₀** stiffness dominates. Displacement per volt becomes constant. Pressure then falls as f²: about 12 dB per octave.

For a speaker with f₀ = 1020 Hz, a 250 Hz tone sits two octaves below. At the same voltage it would play roughly 24 dB quieter than in the flat range (our own calculation, ignoring the effect of the quality factor near resonance).

### 3. Key parameters

**The travel that would be needed.** Our own calculation with the formula above. Assumptions: 1 cm² diaphragm, pure tone at 80 dB SPL (0.2 Pa RMS) measured at 0.5 m, free field, air at 1.2 kg/m³:

| Frequency | Peak displacement needed |
|---|---|
| 4000 Hz | 0.023 mm |
| 2000 Hz | 0.094 mm |
| 1000 Hz | 0.38 mm |
| 500 Hz | 1.5 mm |
| 250 Hz | 6.0 mm |
| 100 Hz | 37.5 mm |

**The travel available.** The microspeakers in Peng *et al.* allow X_max = 0.4 mm. With that limit, the example in the table just reaches 1 kHz and cannot go much lower.

**Comparison with a large speaker.** A home bass driver with a diaphragm of about 130 cm² (illustrative value) would need 0.29 mm at 100 Hz for the same level (our own calculation). That is 130 times less travel because it moves 130 times more area. What matters is the product S_d · X: the volume of air displaced.

**Two published speakers.** Their data sheets:

| Parameter | iPhone 7 speaker (TI, SLAA857) | Microspeaker in Peng *et al.* (2020) |
|---|---|---|
| Coil resistance | 7.35 Ω | 7 Ω ±15% at 2 kHz |
| Force factor Bl | 0.841 T·m | not published |
| Moving mass M_ms | 0.116 g | not published |
| Compliance C_ms | 0.21 mm/N | not published |
| Resonance | 1020 Hz | 700 ± 100 Hz |
| Maximum travel | not published | 0.4 mm |
| Rated power | not published | 1.13 W (1.7 W short-term) |
| Maximum temperature | not published | 100 °C |
| Sensitivity | not published | 93 dB at 2 kHz and > 84 dB at 500 Hz (2.83 V, 10 cm, baffle) |

The sensitivity in Peng *et al.* already shows the drop: at least 9 dB less at 500 Hz than at 2 kHz for the same voltage.

**The thermal limit.** Almost all the electrical power ends up as heat in the coil: P = i² · R_e (SLAA857). TI models the temperature with two nodes. The coil has a time constant of about 4.2 s and the magnet about 412 s. The coil heats up in seconds; the magnet, in minutes. Peng *et al.* measured that, at rated power, the coil went past 100 °C within 20 to 70 seconds.

### 4. Diagram or flow

```text
 AUDIO SAMPLES ─► DAC ─► CLASS-D AMPLIFIER (with boost)
                               │ voltage u      ▲ voltage and current sensing
                               ▼                │ (smart amplifier)
 ┌──────────────── MICROSPEAKER ≈ 16 × 10 × 2 mm ┴────────────────────┐
 │  COIL in the MAGNET's field:  F = Bl · i     (Bl < 1 N/A)          │
 │        │                                                           │
 │        ▼                                                           │
 │  DIAPHRAGM ~1 cm² + SUSPENSION:  M·z'' + C·z' + K·z = F            │
 │        │                         resonance f₀ ≈ 0.7-1 kHz          │
 │        │  maximum travel X_max ≈ 0.4 mm                            │
 │  BACK CAVITY ~0.6 cm³ (more stiffness → higher f₀)                 │
 └────────┬───────────────────────────────────────────────────────────┘
          │ air flow U = S_d · ω · X
          ▼
  PRESSURE AT THE EAR:  |p| = ρ · ω² · S_d · X / (4π · r)
          │
          ├─ above f₀: flat pressure   (useful range)
          └─ below f₀: falls ~12 dB per octave
             and each octave would need ×4 travel to compensate
```

## Limits, exceptions, and real-world cases

- **Resonance is not exact.** Peng *et al.* give a tolerance of ±100 Hz around 700 Hz between units of the same model. Two identical phones do not sound exactly alike in the bass.
- **The linear model falls short.** Klippel notes that most microspeakers show substantial creep: the suspension gives way more at low frequencies than the simple model predicts, so the real displacement is larger. Bl, stiffness and inductance also change with diaphragm position. Distortion appears near maximum travel.
- **Over-excursion destroys the speaker.** The Analog Devices note warns that pushing bass below resonance can drive the diaphragm past its limit and break it.
- **Heat limits before rated power does.** The power needed for high sound levels often exceeds the continuous rating of a small speaker (TI, SSZT848). The headroom lasts only a few seconds, because of the coil's thermal time constant.
- **Ambient temperature matters.** A phone in the sun starts with a warm coil. Peng *et al.* calibrated their temperature estimate with ambient temperatures from 25 to 95 °C.
- **The cavity matters as much as the speaker.** The same speaker gives a different response in a different cavity. The back volume, the ducts and the phone's output grille are all part of the acoustic system.

## Practical consequences

- **Smart amplifiers squeeze the margin.** Instead of limiting the voltage with a generous safety cushion, they measure the voltage and current at the output (*IV sense*). From that they compute the coil resistance and, from it, the coil temperature, because copper resistance rises as it heats (SSZT848, SLAA857). TI uses an inaudible low-frequency pilot tone, for example 60 Hz, because there the back-EMF almost vanishes and the measurement gives the bare resistance. A speaker model, updated in real time, predicts the displacement before it happens. If it would exceed X_max, the signal is attenuated in time thanks to a small delay (SLAA857).
- **They gain decibels, not miracles.** Analog Devices claims its dynamic speaker management raises the level by 6 to 8 dB and extends the bass down to a quarter of the resonance frequency. These are vendor figures, with no measurement data published in the note.
- **That is why maximum volume depends on the music.** If a track has a lot of bass, the protector must attenuate earlier to stay within the travel limit (SLAA857). A solo voice can play louder than a song with a heavy kick drum.
- **Two speakers help little in the bass.** Adding area adds flow: what counts is the total product S_d · X. But two 1 cm² diaphragms are still about 65 times less area than a 130 cm² home speaker (our own calculation).
- **For designers.** Every tenth of a cubic centimetre of cavity and every tenth of a millimetre of travel competes with the battery and the camera for space. Park *et al.* propose redesigning the magnetic circuit to gain bass force without growing.

## Frequently asked questions

### If I turn up the volume, does the bass appear?

Not in the same proportion. Raising the volume raises every frequency equally until something hits its limit. In the bass, the limit comes first: the diaphragm's travel. From then on, the protector cuts the bass or the sound distorts.

### Can an equaliser make up for the missing bass?

Only a little. An equaliser raises the voltage in the bass, which demands more travel and more heat. Below resonance, every 12 dB of boost quadruples the displacement. The 0.4 mm limit is reached almost at once. That is exactly what the smart amplifier watches.

### Is the resonance the same as the lowest frequency you can hear?

No. The resonance is where the mass-spring system moves most easily. Below it the speaker keeps playing, but less and less: about 12 dB per octave. The microspeakers in Peng *et al.* resonate around 700 Hz and still deliver more than 84 dB at 500 Hz in their reference test.

### Then why can I still hear a song's bass line, faintly?

Because the low notes of a real instrument have harmonics: components at two, three or more times their frequency. The speaker reproduces those harmonics well, since they fall in its useful range. What is lost is mostly the fundamental and the sense of body. Processing that boosts those harmonics to fake bass is outside the scope of this article.

## Sources

- S. L. Garrett. *Understanding Acoustics: An Experimentalist's View of Sound and Vibration*, ch. 12 "Radiation and Scattering". Graduate Texts in Physics, Springer, 2020. DOI: [10.1007/978-3-030-44787-8_12](https://doi.org/10.1007/978-3-030-44787-8_12). <https://link.springer.com/chapter/10.1007/978-3-030-44787-8_12>
- S. Xue, S. Palit, P. Wei and A. Du. *SLAA857: SmartPA Speaker Protection Algorithm*. Texas Instruments, August 2018. <https://www.ti.com/lit/pdf/slaa857>
- M. Kucic. *SSZT848: Don't Limit Your Audio: How to Achieve Loud Sound from a Small Speaker*. Texas Instruments, December 2017. <https://www.ti.com/lit/pdf/sszt848>
- K.-H. Park, Z.-X. Jiang and S.-M. Hwang. "Design and Analysis of a Novel Microspeaker with Enhanced Low-Frequency SPL and Size Reduction". *Applied Sciences*, vol. 10, no. 24, 8902, 2020. DOI: [10.3390/app10248902](https://doi.org/10.3390/app10248902). <https://www.mdpi.com/2076-3417/10/24/8902>
- C. Peng, Y. Shi, B. Yan, L. Wu, Z. Chen, C. D. Salvador and D. Liu. "Power-based thermal limits for micro-speaker protection algorithms". Audio Engineering Society, 148th Convention, e-Brief 594, June 2020. <https://cesardsalvador.github.io/doc/Peng2020PowerBasedThermalLimitsForMicrospeakers.pdf>
- Klippel GmbH. *Hands-On Training 7: Microspeaker*. <https://www.klippel.de/training/attachments/training7/Training_7_Microspeaker_en.pdf>
- Analog Devices (Maxim Integrated). "Unleash the Full Potential of Your Micro Speakers". Technical article, 24 September 2019. <https://www.analog.com/en/resources/technical-articles/unleash-the-full-potential-of-your-micro-speakers.html>
