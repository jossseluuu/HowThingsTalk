---
title: "How does your phone know what percentage of battery it has left?"
section: "08_PowerAndBatteries"
language: "en"
status: "published"
order: 1
depth: "intermediate"
summary: "A battery has no charge sensor: the percentage is an estimate. A chip called a fuel gauge counts the current flowing in and out, and corrects the drift of that count with the voltage the cell shows at rest. It then predicts how much charge can still be drawn before the voltage falls to the minimum at the present load and temperature, and divides that by the learned capacity."
prerequisites:
  - "Current, voltage and Ohm's law (V = I · R)"
  - "What a milliamp-hour (mAh) is"
translation: "../../ES/08_EnergiaYBaterias/01-ComoSabeElMovilCuantaBateriaLeQueda.md"
---

# How does your phone know what percentage of battery it has left?

## Question and framing

The battery icon says 37%. It looks like a direct reading, like a thermometer. It is not. Nothing inside a lithium-ion cell measures "how much charge is left". This article asks a concrete question: where does that number come from, and why does it sometimes behave strangely?

The path runs from what can be measured (voltage, current and temperature) to the number on the screen. It goes through the chip that does the estimating, called a **fuel gauge** or gas gauge. The details come from two gauge families documented by their manufacturers: Texas Instruments' Impedance Track and Maxim's (now Analog Devices') ModelGauge m5. This is not a guide to "calibrating" a battery or extending its life, and it compares no models. Why a battery runs out sooner in the cold or with age appears only as a consequence of the mechanism.

The depth is intermediate. You only need to know what current and voltage are, Ohm's law and what a milliamp-hour means.

## Short answer

The percentage is a prediction: the fuel gauge counts the charge that leaves, corrects that count with the voltage the cell shows at rest, and works out how much charge it can still deliver before the voltage falls to the minimum at the present load and temperature.

### 60-second summary

There are two simple ways to estimate charge, and each fails on its own.

1. **Read the voltage.** At rest, a cell's voltage depends on its state of charge. But as soon as the phone draws current, the voltage drops because of the cell's internal resistance. And in the middle range the curve is very flat: a few millivolts separate quite different states of charge.
2. **Count the charge.** A coulomb counter adds up the current flowing in and out, second by second. It is accurate in the short term. But any small measurement error accumulates without limit, and the count drifts.

The fuel gauge combines both. The counter tracks fast changes. The rest voltage corrects the drift whenever it gets the chance. With two rest readings and the charge counted between them, the chip also learns the cell's real capacity, which shrinks over the years.

The last step is a prediction. The percentage does not measure the chemical charge left. It measures the charge that can be drawn before the voltage falls to the minimum operating level. That amount depends on load and temperature. So the number can move even when the charge in the cell has not changed.

## Mental model

Picture a ship without GPS. The navigator works out the position by dead reckoning: heading, speed and time. The calculation is accurate in the short term, but every small error adds up, and after hours at sea the computed position drifts away from the real one. When a known lighthouse appears on the coast, the navigator snaps the position to it and carries on reckoning from there.

The coulomb counter is the dead reckoning. The rest voltage is the lighthouse.

The analogy breaks in two places. First, the lighthouse is only visible when the ship is almost stopped: the voltage only tells the truth after the cell has rested for a good while at very low current. Second, at sea the distance left to port does not depend on how fast you go. In a battery it does: what is left "to port" is the charge that can be drawn before the voltage falls to the minimum, and that charge is smaller at high loads or in the cold. The gauge works out not only where it is but also how far it will get at the current pace.

## Technical mechanism

### 1. What happens in the system

The fuel gauge is a small chip next to the battery or inside the charging circuit. It runs five tasks in a loop.

1. **Measure.** It reads the cell voltage, the current through a sense resistor, and the temperature.
2. **Count charge.** It integrates current over time. Every second it subtracts or adds the charge that left or entered.
3. **Anchor on the rest voltage.** When the cell has spent a while at very low current, its voltage approaches the open-circuit voltage (OCV). A table specific to the cell chemistry converts that OCV into a state of charge. That value replaces or corrects the accumulated count.
4. **Learn the cell.** It compares how much the state of charge changes according to the OCV with how much charge it has counted. From that it derives the real capacity. It also compares the voltage under load with the expected OCV, and from that it derives the internal resistance.
5. **Predict and report.** It simulates the discharge from now on, at the present load and temperature, down to the minimum voltage. The charge that comes out of that simulation, divided by the usable capacity, is the percentage. The operating system reads it and displays it.

### 2. What each component does

**The sense resistor and the coulomb counter.** Current flows through a very small resistor. The chip measures the tiny voltage drop across it and converts it into current. In the MAX17055 reference design the resistor is 10 mΩ (data sheet, Figure 1). At 1 A that leaves a 10 mV drop. The smallest measurement step is 1.5625 µV, which at 10 mΩ is 156.25 µA. The offset error is ±1.5 µV as a long-term average, and the gain error is ±1% of reading (data sheet, p. 4). The current figures in amps are this article's own Ohm's-law arithmetic.

That offset looks negligible, but it is integrated. ±1.5 µV across 10 mΩ is ±150 µA. Over 24 hours that adds up to 3.6 mAh, and over 30 days about 108 mAh. On a 4000 mAh cell, chosen as an example, that would be an error of about 2.7 points a month. This is our own calculation of a worst case: it assumes the error always has the same sign. The data sheet puts it this way: offset error "cannot be eliminated", and the count needs periodic correction (p. 15).

**Voltage and its table.** The chip measures the cell voltage with an error of ±7.5 mV at 25 °C and ±20 mV from −40 to +85 °C (MAX17055, p. 4). The table that translates OCV into state of charge depends on the cell chemistry. Texas Instruments stores separate tables for each chemistry, for example lithium cobalt oxide with a carbon anode, and identifies them with a chemistry code (SLUA364B, section 2.2). Analog Devices states that the OCV-to-state-of-charge relationship stays largely independent of cell age (ModelGauge m5 EZ technical article). That stability is what makes it a good lighthouse.

**Internal resistance.** Under load, the terminal voltage is lower than the OCV:

```text
V_terminal = OCV − I · R
```

Every 100 mΩ of internal resistance at a 1 A load takes away 100 mV. That is Ohm's law, not a measured value for any cell. The resistance is not fixed. Texas Instruments models it with an exponential temperature dependence and stores a table of values at 15 depth-of-discharge points (SLUA364B, sections 2.4 and 2.6). Apple describes the same thing qualitatively: impedance rises with the battery's chemical age and rises temporarily at a low state of charge and in the cold (Apple, "iPhone battery and performance").

**The rest anchor.** In Impedance Track, the chip enters relaxation when current falls below a threshold. The recommendation is that it not exceed C/20, the current that would empty the battery in 20 hours. It takes OCV readings after 30 minutes of rest and then every 100 seconds. It treats the cell as well relaxed when the voltage changes by less than 4 µV per second, or after a maximum of 5 hours. With the battery between 0% and 80% depth of discharge, that takes about 1 hour. Above 80%, 3 to 4 hours, and longer in the cold. Before using the reading, the chip subtracts the small drop still caused by the rest current (SLUA364B, sections 2.2 and 2.3).

**Learning capacity.** With two rest OCV readings, one before and one after a discharge, and the charge counted between them, the capacity is a division:

```text
Qmax = counted charge / (DOD2 − DOD1)
```

DOD is depth of discharge, the complement of state of charge. An example with round numbers: if the OCV indicates 20% depth, then 70%, and 1500 mAh were counted in between, the capacity is 1500 / 0.5 = 3000 mAh. The example is this article's. Texas Instruments only accepts the measurement under three conditions. The temperature must be between 10 and 40 °C. The counted charge must exceed 37% of design capacity. And neither reading may fall between 3737 and 3800 mV, a range the document calls "flat" and that depends on the chemistry (SLUA364B, section 2.3). ModelGauge does something similar continuously: it compares the percentage change indicated by the OCV with the counted charge (MAX17055 user guide, section 1.2.6).

**Continuous mixing.** ModelGauge m5 does not wait for rest to correct itself. When the battery is first connected, it relies mostly on the voltage-based estimate, because the counter does not know where it starts. Over cycles, the weight shifts to the counter. From then on it applies a small, fixed-size correction, up or down, based on the sign of the difference between the count and the voltage model. According to Analog Devices, it does this more than 200,000 times a day, under load and at rest. The result does not accumulate offset drift (ModelGauge m5 EZ technical article; user guide, section 1.2.1).

### 3. Key parameters

| Parameter | Value | Scenario and source |
|---|---|---|
| Sense resistor | 10 mΩ | MAX17055 reference design (data sheet, Figure 1) |
| Current measurement step | 1.5625 µV (156.25 µA at 10 mΩ) | MAX17055 (data sheet, p. 4) |
| Current offset | ±1.5 µV long-term average | MAX17055 (data sheet, p. 4) |
| Voltage error | ±7.5 mV at 25 °C · ±20 mV from −40 to +85 °C | MAX17055 (data sheet, p. 4) |
| Gauge's own consumption | 7 µA typical in hibernate · 18 µA active | MAX17055 (data sheet, p. 4) |
| Rest before reading OCV | 30 min, then every 100 s | Impedance Track (SLUA364B, 2.2) |
| Relaxed-cell criterion | dV/dt < 4 µV/s or 5 h maximum | Impedance Track (SLUA364B, 2.3) |
| Range too flat to learn from | 3737 to 3800 mV | Impedance Track default chemistry (SLUA364B, 2.3) |
| Discharge simulation step | 4% depth | Impedance Track (SLUA364B, 2.6) |
| Default empty voltage | 3.3 V | MAX17055 VEmpty register (data sheet, p. 12) |
| Published accuracy | 97% of tests below 3% error | MAX17055 after the first cycle, at +20 and +40 °C, over 300 batteries (data sheet, p. 12) |

The accuracy in the last row comes with conditions. The tests are at room temperature or above, with discharges longer than 3 hours. Analog Devices also excludes chemistries whose OCV curve differs greatly from the usual ones (ModelGauge m5 EZ technical article).

### 4. Diagram or flow

```text
            ┌──────────────── lithium-ion cell ──────────────────┐
            │                                                     │
   voltage V│              current I (through 10 mΩ)   temperature T
            ▼                       ▼                         ▼
   ┌─────────────────┐    ┌──────────────────┐     ┌─────────────────────┐
   │ at rest?        │    │ coulomb          │     │ internal resistance │
   │ small dV/dt     │    │ counter          │     │ R(depth, T)         │
   │ → OCV → table   │    │ Q += I · Δt      │     │ = (OCV − V) / I     │
   └────────┬────────┘    └────────┬─────────┘     └──────────┬──────────┘
            │ corrects the drift   │                          │
            └──────────►  current state of charge  ◄──────────┘
                                   │
                                   ▼
            simulation: V = OCV − I · R, step by step, down to V_minimum
                                   │
                                   ▼
          remaining usable charge / usable capacity × 100  =  37%
                                   │
                                   ▼
               chip register → kernel → system → screen
```

Where "0%" sits inside the chemical charge:

```text
total chemical charge (Qmax)
├────────── already used ──────────┼───── usable at this pace ─────┼── not extractable at this pace ──┤
                                   ▲                               ▲
                                  now              OCV − I·R hits the minimum voltage: 0%

With more load or more cold, I·R grows and the second mark moves to the left.
```

The charge to the right of the second mark exists, but it cannot be used at that pace without the voltage falling below the minimum.

## Limits, exceptions, and real-world cases

**What the screen shows is not the chemical capacity.** Texas Instruments distinguishes two capacities. Qmax is the total chemical capacity. Full-charge capacity (FCC) is the charge drawn down to the terminate voltage, minus a reserve. The usable figure is lower at high loads and in the cold, because the I · R drop brings the cutoff forward (SLUA364B, section 1). The percentage is computed on the usable figure: remaining usable charge × 100 / FCC (section 2.7). So the same chemical state can give different percentages depending on load and temperature.

**The number can rise without charging.** If the load drops or the temperature rises, the I · R drop shrinks and the simulation finds more usable charge. The MAX17055 guide says so explicitly: the available capacity can change abruptly and can even increase during discharge (section 1.2.2). To keep this from showing as jumps, the chip publishes a filtered version that converges on the real value gradually (section 1.2.2).

**The end is forced to line up.** Near empty, ModelGauge adjusts how fast the percentage falls so that it reaches 0% just as the voltage reaches the minimum. It can also hold the percentage at a low value, 1% by default, until the voltage crosses the empty threshold (user guide, sections 1.2.3 and 1.2.4). Without these adjustments, the phone would shut down showing 5%, or sit at 0% for days.

**Pulsed loads make rest harder to find.** A phone is rarely fully idle: the radio wakes in bursts. Texas Instruments documents a GSM phone with pulses of 1 A for 0.48 ms and 72 mA for 4.76 ms. For these cases rest does not require zero current, only a low current, below C/20. A brief spike does not take the chip out of relaxation: the current must stay high for more than 1 second (SLUA450A, sections 2.1 and 3.1). In that test, the error was around 4% at the start and stayed within 2% from 80% down to 0% (section 3.1). It is a single test, not a general guarantee.

**Not every phone counts coulombs.** Some gauges use only voltage and a cell model. The Android documentation cites the Nexus 5, with a MAX17048 without a coulomb counter, which only reports state of charge at a resolution of 1/256% (AOSP, "Measure device power"). On phones that do count, Android exposes remaining charge in µAh and current in µA. But the resolution and update rate depend on the chip, the sense resistor and its configuration. The Nexus 6, for example, updates current every 175.8 ms in steps of 156.25 µA (same source).

**Flat chemistries are harder.** The MAX17055 guide notes that the OCV curve of lithium iron phosphate (LiFePO4) is much flatter than that of cobalt chemistries and has two plateaus. That makes the algorithm more sensitive to how it interprets voltage (section 1.2.8). The flatter the curve, the worse the voltage works as a lighthouse.

**The model ages with the cell.** Texas Instruments acknowledges that pure charge-counting and voltage methods lose accuracy as the battery ages (SLUA364B, abstract). Its own estimated error margin grows with every cycle without a new capacity learn (section 2.8). Combined algorithms compensate by relearning capacity and resistance, but they need opportunities: long rests or full charges.

## Practical consequences

For the person using the phone, the percentage is a forecast, not a tank with a dipstick. Gaining a point after closing a game, or dropping fast when opening the camera in the cold, does not mean a fault. It is the gauge recomputing usable charge for a new load and temperature.

Shutdowns with charge still showing have the same root. Apple explains that with high impedance the voltage drops further under a peak load. If it falls below the minimum the components need, the system shuts down to protect them. From the device's point of view the shutdown is intentional, even if it surprises the user. It is more likely at a low state of charge, with an aged battery or in the cold (Apple, "iPhone battery and performance"). The gauge predicts from average load; a brief, heavy peak can get ahead of that prediction.

For the person designing the device, accuracy starts in hardware. A small offset in current measurement is integrated over weeks. That is why the sense resistor, its placement and the gauge's own consumption matter. And it is why the algorithm needs a source of truth that does not drift, such as the rest voltage, and a chemistry with a well-characterised OCV curve.

## Frequently asked questions

### Is the percentage the same as the battery's "maximum capacity" or health?

No. The percentage compares the usable charge left with the current usable capacity. Maximum capacity compares the current capacity with what the battery had when new (Apple, "iPhone battery and performance"). An aged battery can show 100% and last less: it is full, but its 100% is smaller.

### Do you need to run the battery down to 0% to "calibrate" the gauge?

In the two algorithms reviewed here, it is not a requirement. Impedance Track learns capacity from two rest readings separated by a discharge of more than 37% of design capacity (SLUA364B, section 2.3). ModelGauge m5 is described as an algorithm that does not require empty, full or idle states (MAX17055 data sheet, p. 1). If a particular phone uses another method, its behaviour is **unverified** here.

### Why does the percentage go up a little if I leave the phone alone for a while?

Because when the load stops, the voltage recovers towards the OCV and the I · R drop disappears. The gauge recomputes and finds more usable charge. The chemical charge has not increased; only how much of it can be drawn has changed.

### Is a coulomb counter the same as an energy meter?

No. The counter adds up charge, in mAh. Energy, in Wh, is charge multiplied by the voltage at which it is delivered. Android exposes both, charge in µAh and energy in nWh. On the Nexus 6 and Nexus 9, energy is obtained by multiplying charge by a fixed nominal voltage of 3.7 V (AOSP, "Measure device power").

## Sources

- [Texas Instruments — Theory and Implementation of Impedance Track Battery Fuel-Gauging Algorithm in bq20zxx Product Family. Application report SLUA364B, November 2005, revised December 2006](https://ti.com/lit/SLUA364). Chemical and usable capacity (section 1), OCV and per-chemistry table (2.2), relaxation, the 4 µV/s criterion and Qmax learning with its conditions (2.3), resistance and its temperature dependence (2.4), discharge simulation in 4% steps (2.6), continuous percentage (2.7), estimated error (2.8) and limitations with ageing (abstract).
- [Texas Instruments — Theory and Implementation of Impedance Track Battery Fuel-Gauging Algorithm in bq2750x Family. Application note SLUA450A, January 2008, revised November 2022](https://ti.com/lit/pdf/slua450). Entering and leaving relaxation (section 2.1) and a GSM phone test under pulsed load (3.1).
- [Analog Devices — Accurate Fuel Gauging Without Battery Characterization Using ModelGauge m5 EZ Algorithm. Technical article, August 2019](https://www.analog.com/en/resources/technical-articles/accurate-fuel-gauging-without-battery-characterization-using-modelgauge-m5-ez-algorithm.html). Mixing of counter and voltage model, over 200,000 corrections a day, OCV independence from age, and accuracy results.
- [Maxim Integrated (Analog Devices) — MAX17055 7µA 1-Cell Fuel Gauge with ModelGauge m5 EZ. Data sheet, revision 0, January 2017](https://www.analog.com/media/en/technical-documentation/data-sheets/MAX17055.pdf). Measurement and supply electrical characteristics (p. 4), reference sense resistor (Figure 1), VEmpty and published accuracy (p. 12), counter drift and mixing (p. 15).
- [Maxim Integrated (Analog Devices) — MAX17055 ModelGauge m5 EZ User Guide. Revision 0, December 2016](https://wcm-cce.cldnet.analog.com/media/en/technical-documentation/user-guides/max17055-user-guide.pdf). Mixing (section 1.2.1), empty compensation and filtering (1.2.2), convergence and hold near empty (1.2.3 and 1.2.4), capacity learning (1.2.6) and LiFePO4 (1.2.8).
- [Android Open Source Project — Measure device power](https://source.android.com/docs/core/power/device). Battery properties in µAh, µA and nWh, dependence on fuel gauge hardware, Nexus 6 and 9 resolution and update period, counter offset, and the Nexus 5 without a coulomb counter.
- [Android Developers — BatteryManager](https://developer.android.com/reference/android/os/BatteryManager). Definitions of BATTERY_PROPERTY_CAPACITY, CHARGE_COUNTER, CURRENT_NOW, CURRENT_AVERAGE and ENERGY_COUNTER.
- [Apple — iPhone battery and performance, June 2026](https://support.apple.com/en-us/101575). Impedance, chemical age, cold, low charge, shutdowns to protect components, and maximum capacity.
