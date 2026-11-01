---
title: "Why does a higher-frequency radio signal reach a shorter distance?"
section: "00_Foundations"
language: "en"
status: "published"
order: 1
depth: "intermediate"
summary: "A 5 GHz wave is not used up by the air any faster than a 2.4 GHz wave. It reaches less far because the small antennas in our devices collect less energy when the wave is shorter, because materials absorb more at higher frequencies, and because short waves bend around obstacles less well."
prerequisites:
  - "What an electromagnetic wave is, and what its frequency and wavelength are"
  - "Decibels and dBm (reviewed inside the article)"
translation: "../../ES/00_Fundamentos/01-PorQueMasFrecuenciaLlegaMenosLejos.md"
---

# Why does a higher-frequency radio signal reach a shorter distance?

## Question and framing

It is a common experience: 2.4 GHz WiFi reaches the back bedroom and 5 GHz does not. The question is why. What is it about a higher frequency that makes it reach less far?

The usual answer says that "high frequencies are attenuated more along the way". This article explains why that sentence is misleading and which three physical mechanisms actually produce the effect. It uses WiFi as the example, but the idea applies to any radio link: cellular, Bluetooth or GPS.

- **Depth:** intermediate. A few simple formulas appear, always explained first.
- **Prerequisites:** knowing that a wave has a frequency (cycles per second) and a wavelength (the distance between two crests). The two are linked: wavelength = speed of light / frequency. Decibels are reviewed below.
- **Out of scope:** antenna design, WiFi protocols, each country's transmit-power rules, and absorption by atmospheric gases at millimetre waves.

**Quick decibel refresher.** The decibel (dB) expresses a ratio between two powers on a logarithmic scale. +3 dB is roughly double the power. +10 dB is ten times more. A 20 dB loss leaves one hundredth of the power. Losses in dB add up instead of multiplying, which is why radio engineers use them.

## Short answer

A higher-frequency wave is not used up faster in empty space: it arrives weaker because the small antenna receiving it captures less energy, because walls absorb more of it, and because it bends around obstacles less well.

### 60-second summary

A transmitter spreads its power over a sphere that grows with distance. That spreading is the same at every frequency. At 10 m, each square metre receives the same power at 2.4 GHz as at 5.5 GHz.

The difference is at the receiver. An antenna that listens equally in all directions only collects the energy falling on an effective area proportional to the square of the wavelength. At 5.5 GHz the wave is less than half as long as at 2.4 GHz. So that area shrinks about five times. That is about 7 dB less signal in free space.

Inside a building, two more effects add up. Common materials (concrete, plasterboard, wood, glass) conduct electricity better at higher frequencies, so they absorb more energy per centimetre of thickness. And a shorter wave bends less well around obstacles that block the direct path.

## Mental model

Picture a sprinkler watering a lawn. The water spreads in every direction. The farther away you are, the less water lands on each square metre. To measure it, you put a cup on the ground. What you collect depends on two things: how much water falls per square metre and the size of the cup's mouth.

In radio, the "water per square metre" does not depend on frequency. What changes is the cup. A small antenna that listens in every direction has an effective mouth that shrinks with the wavelength. Raising the frequency is like using the same sprinkler and measuring with a narrower cup.

**Where the analogy breaks.** An antenna's mouth is not a physical hole: it is an equivalent area that comes from the antenna's physics. If you could keep the real size of the "cup" fixed (for example, a dish antenna of fixed diameter), the higher frequency would win instead of lose. Also, water does not go through walls or bend around corners. Radio waves do, and frequency matters there for reasons the sprinkler does not capture.

## Technical mechanism

### 1. What happens in the system

The signal path has three stages where it can lose power:

1. **Geometric spreading.** The transmitter spreads its power over a sphere of area 4πd², where d is the distance. The power flux density is S = P / (4πd²), in W/m² (Rec. ITU-R P.525-5, eq. 3). Frequency does not appear in this formula.
2. **Capture at the receiving antenna.** Received power is the power flux density times the antenna's effective area: Pr = S · Ae (P.525-5, eq. 4). For an isotropic antenna, the theoretical reference that listens equally in every direction, Ae = λ² / (4π), where λ is the wavelength. This is where frequency enters.
3. **Obstacles along the path.** In a building, the wave crosses materials that absorb energy and bends around objects by diffraction. Both depend on frequency (Rec. ITU-R P.2040-3 and P.526-16).

Combining stages 1 and 2 gives the **free-space basic transmission loss** between isotropic antennas (P.525-5, eqs. 5 and 6):

```text
Lbf = 20·log10(4π·d / λ)   [dB]
Lbf = 32.4 + 20·log10(f) + 20·log10(d)   [dB]   with f in MHz and d in km
```

The 20·log10(f) term is often read as "air attenuates high frequencies more". It does not. That term comes entirely from the effective area λ²/(4π) of the isotropic receiving antenna. Free space absorbs nothing: the energy only spreads out.

### 2. What each component does

**The transmitter and its antenna.** They set the radiated power and how it is distributed across directions (the gain). A router or a phone uses low-gain antennas because it must cover all directions. They behave much like the isotropic reference antenna.

**The space in between.** In vacuum or clean air, at these frequencies, it only spreads the energy. Doubling the distance cuts the power flux density to a quarter (−6 dB) at any frequency.

**The receiving antenna.** Its effective area is Ae = G · λ² / (4π), where G is its gain. If G is held fixed, the area falls with the square of the frequency. At 2437 MHz (WiFi channel 6), λ = 12.3 cm and Ae = 12.0 cm² for G = 1. At 5500 MHz (channel 100), λ = 5.45 cm and Ae = 2.4 cm². That is five times less capture area, and the difference is 7.1 dB.

Friis showed the other side of the same fact in 1946. If both antennas keep a **fixed physical area**, the ratio is Pr/Pt = Ar · At / (d² · λ²). In that case, raising the frequency *improves* the link. This is why dish-based radio links use high frequencies. Whether a high frequency "reaches less" depends on what is held constant: the antenna's gain or its size.

**Materials.** Rec. ITU-R P.2040-3 models each material with two numbers: relative permittivity η′ and conductivity σ (in S/m). For almost all building materials, η′ barely changes with frequency, but σ grows with it (σ = c · f^d, with f in GHz). For low-loss materials, the attenuation inside the material is A ≈ 1636 · σ / √η′ dB/m (P.2040-3, eq. 27a). More conductivity means more energy turned into heat per metre of material.

**Obstacles that block the path.** Diffraction lets the wave bend around an edge, but how well it works depends on the obstacle's size compared with the wavelength. Rec. ITU-R P.526-16 sums up the geometry in a dimensionless parameter ν. This parameter grows as λ shrinks (ν ∝ 1/√λ for the same geometry), and a larger ν means more diffraction loss. The zone that must be clear for a path to count as unobstructed is the first Fresnel ellipsoid. Its radius is R1 = 550 · √(d1·d2 / ((d1+d2)·f)) metres, with d1 and d2 in km and f in MHz (P.526-16, eq. 3).

### 3. Key parameters

All the values below are calculated with the formulas cited. The scenario is 2437 MHz versus 5500 MHz.

| Quantity | 2437 MHz | 5500 MHz | Formula source |
|---|---|---|---|
| Wavelength λ | 12.3 cm | 5.45 cm | λ = c / f |
| Effective area of an isotropic antenna | 12.0 cm² | 2.4 cm² | P.525-5, eq. 4 |
| Free-space loss at 10 m | 60.1 dB | 67.2 dB | P.525-5, eq. 6 |
| First Fresnel ellipsoid radius at the midpoint of a 20 m link | 0.79 m | 0.52 m | P.526-16, eq. 3 |

Internal attenuation of some materials according to the Rec. ITU-R P.2040-3 model (Table 3 and eq. 27a). The figures only count absorption inside the material. They do not include reflection at its surfaces, which adds further loss.

| Material | 2437 MHz | 5500 MHz | Example: thickness and loss (2437 → 5500 MHz) |
|---|---|---|---|
| Concrete | 66 dB/m | 125 dB/m | 15 cm wall: 9.9 dB → 18.8 dB |
| Brick | 23 dB/m | 26 dB/m | 10 cm partition: 2.3 dB → 2.6 dB |
| Wood | 14 dB/m | 34 dB/m | 4 cm solid door: 0.6 dB → 1.4 dB |
| Plasterboard | 19 dB/m | 42 dB/m | 1.25 cm board: 0.2 dB → 0.5 dB |
| Glass | 8 dB/m | 23 dB/m | 6 mm pane: 0.05 dB → 0.14 dB |

Two readings of the table:

- Concrete almost doubles its attenuation per metre between the two bands. It is the material where the effect shows most.
- Brick barely changes, because in the ITU model its conductivity hardly depends on frequency (exponent d = 0.16).

**What 7 dB means in distance.** In free space, loss rises by 20 dB for every factor of 10 in distance, the same in both bands. To receive the same power with antennas of equal gain, the 5500 MHz link must be 2437/5500 ≈ 0.44 times as long. Indoors the real ratio changes, because walls and reflections add up.

### 4. Diagram or flow

```text
TRANSMITTER (power P, gain Gt)
   │
   ▼
Spherical spreading: S = P·Gt / (4π·d²)       ← independent of f
   │
   ▼
Obstacles along the path:
   ├─ absorption in materials: A ≈ 1636·σ(f)/√η′ dB/m   ← σ grows with f
   └─ diffraction at edges: loss grows with ν ∝ 1/√λ    ← worse at higher f
   │
   ▼
RECEIVING ANTENNA: Pr = S · Gr·λ²/(4π)        ← falls as 1/f² if Gr is fixed
   │
   ▼
RECEIVER: compares Pr with the noise → can it be decoded?
```

## Limits, exceptions, and real-world cases

- **The free-space model is an ideal case.** It assumes a clear path and no reflections. In a house, the signal arrives over many paths at once (multipath). Their additions and cancellations cause variations of several dB when the device moves a few centimetres. The size of those variations depends on the environment and is not quantified here.
- **The table's attenuation is not a wall's total loss.** The Rec. ITU-R P.2040-3 figures describe absorption inside a homogeneous material. A real wall has layers, air gaps, steel reinforcement and pipes. Part of the energy is also reflected at each surface. The measured loss of a specific wall can differ a lot from the model.
- **Metal blocks both bands.** With a conductivity of 10⁷ S/m (P.2040-3, Table 3), a metal sheet is practically opaque at both frequencies. The signal passes through gaps, doors and windows, and there diffraction matters again.
- **Validity ranges matter.** The Table 3 parameters in P.2040-3 come from measurements between 1 and 100 GHz for most materials. The recommendation itself warns that the ranges are indicative.
- **Antenna gain can compensate.** If a device fits several antennas in the same space and combines their signals (beamforming), it recovers effective area. This is easier at higher frequencies, because each antenna is smaller. It is why millimetre-wave systems use arrays of many antennas.
- **Not everything is propagation.** The wider channels typically used at 5 GHz collect more thermal noise, which is proportional to bandwidth. This reduces the receiver's margin for a reason unrelated to distance.

## Practical consequences

- **For the person using the device.** A low band gets through walls and floors better. A high band reaches less far inside a home, but its channels are usually less crowded and offer more capacity near the access point. Watching a phone switch bands as it moves away from the router is a direct consequence of this article.
- **For whoever plans a network.** Covering a home with a high band usually takes more access points. Each one covers less area, and in exchange neighbouring cells interfere less with each other.
- **For cellular networks.** By the same physics, low bands are used to provide coverage over wide areas and inside buildings. High bands are used where capacity is needed and distances are short.
- **For device designers.** At higher frequencies the antenna is smaller. But a small, low-gain antenna collects less energy. Gaining range requires more gain, which means steering the beam or using several antennas.

## Frequently asked questions

### Does air absorb high frequencies more?

In the usual WiFi and cellular bands, air absorption is negligible compared with the effects in this article. Atmospheric gases (oxygen and water vapour) do absorb noticeably at millimetre waves, at tens of GHz. That phenomenon is described by Rec. ITU-R P.676 and is outside the scope of this article.

### Is free-space loss energy that gets lost?

No. It is energy that spreads out. The total power is still on the sphere; what drops is the share that lands on the receiving antenna. That is why the ITU calls it "basic" transmission loss and defines it between isotropic antennas: it is a geometric reference, not absorption.

### Does a higher-frequency wave travel more slowly?

No. In air, all radio waves travel at practically the speed of light, about 300,000 km/s. Frequency changes the wavelength, not the speed.

### Then why do satellite dishes use high frequencies?

Because they keep the antenna's physical size fixed, not its gain. With a fixed physical area, gain grows with the square of the frequency. According to Friis' formula, the link improves as frequency rises. It is the opposite case to a phone, whose antenna must listen in every direction.

## Sources

- ITU-R. *Recommendation ITU-R P.525-5: Calculation of free-space attenuation* (11/2024). Equations 3 to 6. <https://www.itu.int/dms_pubrec/itu-r/rec/p/R-REC-P.525-5-202411-I!!PDF-E.pdf>
- ITU-R. *Recommendation ITU-R P.2040-3: Effects of building materials and structures on radiowave propagation above about 100 MHz* (08/2023). Table 3 and equation 27a. <https://www.itu.int/dms_pubrec/itu-r/rec/p/R-REC-P.2040-3-202308-I!!PDF-E.pdf>
- ITU-R. *Recommendation ITU-R P.526-16: Propagation by diffraction* (11/2025). Equations 2, 3 and the ν parameter. <https://www.itu.int/dms_pubrec/itu-r/rec/p/R-REC-P.526-16-202511-I!!PDF-E.pdf>
- ITU-R. *Recommendation ITU-R P.676: Attenuation by atmospheric gases and related effects*. Cited only as a pointer to what is out of scope. <https://www.itu.int/rec/R-REC-P.676/en>
- H. T. Friis. "A Note on a Simple Transmission Formula". *Proceedings of the IRE*, vol. 34, no. 5, pp. 254-256, May 1946. DOI: [10.1109/JRPROC.1946.234568](https://doi.org/10.1109/JRPROC.1946.234568)
