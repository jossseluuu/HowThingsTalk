---
title: "How does a cell tower share its capacity among all the connected phones at once?"
section: "02_CellularNetworks"
language: "en"
status: "published"
order: 1
depth: "intermediate"
summary: "The base station splits its channel into a grid of frequency and time. Every 0.5 or 1 ms its scheduler decides which cells of the grid each phone gets and announces it on a control channel. In the uplink, a phone must ask for permission before it transmits. The decision algorithm is not set by the standard: each vendor designs its own."
prerequisites:
  - "Signal-to-noise ratio, modulation and CQI (article 00·02)"
  - "Why signal changes when you move a few centimetres (article 00·03)"
translation: "../../ES/02_RedesMoviles/01-ComoReparteUnaAntenaSuCapacidad.md"
---

# How does a cell tower share its capacity among all the connected phones at once?

## Question and framing

At a concert or a train station, hundreds of phones use the same cell. They all download and upload at the same time, yet their signals do not collide. The question is how that sharing is organised. Who decides which part of the channel each phone uses, and how often is that decision made?

This article explains the mechanism using 3GPP's 5G NR standard, because it is public and it is the most recent. 4G (LTE) is built on the same idea of a grid and a scheduler, with different values.

- **Depth:** intermediate. There are simple calculations with microseconds and kilohertz.
- **Prerequisites:** what signal-to-noise ratio, modulation and the CQI indicator are, covered in [What decides how many bits per second a radio signal can carry?](../00_Foundations/02-WhatSetsTheSpeedOfARadioLink.md). It helps to know why signal fluctuates as you move, covered in [Why does signal strength change so much when you move a phone a few centimetres?](../00_Foundations/03-WhySignalChangesOverAFewCentimetres.md).
- **Out of scope:** how a phone finds the cell when it powers on, moving from one cell to another (handover), the operator's core network, and interference between neighbouring cells.

## Short answer

The cell splits its channel into a grid of frequency-and-time blocks, and a scheduler inside the base station decides every fraction of a millisecond which blocks each phone uses, in the downlink and in the uplink.

### 60-second summary

A 5G base station does not send one signal for everyone. It slices its channel into thousands of narrow subcarriers, grouped twelve at a time into "resource blocks". In time, it slices the channel into slots of 14 symbols that last 1 ms or 0.5 ms, depending on the configuration.

In every slot, a program in the base station called the scheduler hands out those blocks. It considers how much data is waiting for each phone, how good each phone's signal is, and what each type of traffic requires. It then publishes the allocation on a control channel. Each phone reads it and looks for the messages addressed to its own identifier.

In the uplink a phone cannot transmit whenever it likes. It first signals that it has data, then waits for the base station to assign it blocks. It also transmits early according to its distance, so that all signals reach the antenna aligned.

The standard fixes the format of the grid and of the messages. It does not fix how the scheduler decides: each vendor designs that algorithm.

## Mental model

Think of a post office with many counters and a manager who hands out turns. Every few seconds, the manager looks at the queue. They assign each counter to a customer and show it on a screen. Customers do not walk up on their own: they watch the screen and go to the counter they have been given. Anyone who wants to send a parcel must take a ticket first.

The counters are the frequency blocks. Each refresh of the screen is a time slot. The manager is the scheduler and the screen is the control channel. Taking a ticket is the scheduling request a phone makes before sending data.

**Where the analogy breaks down.** At the post office, every counter serves any customer equally fast. In radio it does not: the same block carries many more bits to a nearby phone with good signal than to a distant one. Each phone's quality also changes from one slot to the next because of fading. A good scheduler exploits those changes, which the post office manager never has to do. And each round lasts 0.5 ms, not seconds.

## Technical mechanism

### 1. What happens in the system

**Downlink (cell to phone):**

1. **The phone reports its channel.** It measures reference signals (CSI-RS) and sends the base station a channel state report, which includes the CQI (3GPP TS 38.300, clause 5.2.5.1).
2. **The scheduler decides.** In each slot it picks which phones receive data, on which resource blocks and with which modulation and coding (TS 38.300, clause 10.1).
3. **The base station announces the allocation.** It sends a control message (DCI) on the PDCCH, tagged with the phone's temporary identifier, the C-RNTI (TS 38.300, clause 10.2).
4. **The phone looks for its message.** It tries to decode several possible positions in the control channel. It only accepts messages whose checksum (CRC) is scrambled with its C-RNTI (TS 38.213, clause 10.1).
5. **The phone receives the data** on the PDSCH, in the indicated blocks.
6. **The phone acknowledges.** It replies with HARQ-ACK on the PUCCH. If the block arrived damaged, the base station retransmits it (TS 38.213, clause 9; TS 38.300, clause 5.2.5.4).

**Uplink (phone to cell):**

1. **Data arrives at the phone** and it has no resources assigned. That triggers a buffer status report (BSR).
2. **The phone sends a scheduling request (SR).** It is a short signal on the PUCCH, on resources the network reserved for it (TS 38.321, clause 5.4.4).
3. **The base station grants an initial permission** (a *grant*) on the PDCCH.
4. **The phone sends the BSR** on the PUSCH. The BSR says how much data is waiting, grouped by traffic type (TS 38.321, clause 5.4.5).
5. **The base station grants more resources** to match that volume. The phone sends its data on the PUSCH.

If the phone has no resource to send the SR, or has repeated it the maximum number of times without an answer, it starts a random access procedure to ask for access again (TS 38.321, clause 5.4.4).

### 2. What each component does

**The resource grid.** 5G NR uses OFDM: the channel is split into many narrow subcarriers that transmit in parallel. The downlink uses OFDM with a cyclic prefix. The uplink can use the same scheme or a DFT-spread variant, which eases the work of the phone's power amplifier (TS 38.300, clause 5.1). Subcarrier spacing is 15 kHz × 2^μ, where μ is the "numerology": 15, 30, 60, 120 kHz and more (TS 38.211, Table 4.2-1).

**The resource block.** It is 12 consecutive subcarriers (TS 38.211, clause 4.4.4.1). It is the unit in which the scheduler allocates frequency (TS 38.300, clause 10.1). A carrier has at most 275 blocks (TS 38.300, clause 5.1).

**The slot.** In time, a frame lasts 10 ms and is divided into 10 subframes of 1 ms. Each slot has 14 OFDM symbols with normal cyclic prefix. At 15 kHz there is 1 slot per subframe; at 30 kHz, 2; at 60 kHz, 4 (TS 38.211, clause 4.3 and Table 4.3.2-1). The wider the subcarrier spacing, the shorter the symbols and the more decisions fit in a second.

**The scheduler.** It is part of the base station's (gNB's) MAC layer. It allocates resources dynamically in the downlink and the uplink. It takes into account each phone's buffer and the quality-of-service requirements of each flow. It may also use the station's own radio measurements and the phones' reports (TS 38.300, clause 10.1). The standard does not describe any decision algorithm.

**The control channel (PDCCH) and the DCI.** The DCI is the message that says "this phone, these blocks, this modulation". The phone monitors the PDCCH whenever its reception is enabled (TS 38.300, clause 10.2). It does not know in advance where its message will be. So it tries a set of candidate positions, called a search space. At 30 kHz, the standard caps those attempts at 36 candidates per slot on a single carrier (TS 38.213, Table 10.1-2).

**The C-RNTI.** It is a temporary identifier the base station assigns to each connected phone. The base station scrambles the CRC of each DCI with the recipient's C-RNTI. Only that phone gets a valid CRC when decoding it (TS 38.213, clause 10.1).

**SR and BSR.** The SR is a minimal notice with no data: "I have something to send". The BSR gives the volume, split into up to 8 logical channel groups (TS 38.321, clause 5.4.5). With that information the scheduler sizes the grant.

**HARQ.** If a block arrives with errors, the receiver keeps what it got. The retransmission sends new redundancy that is combined with the earlier copy ("incremental redundancy HARQ", TS 38.300, clauses 5.2.5.4 and 5.3.5.4). Several HARQ processes run in parallel, so waiting for one acknowledgement does not stall transmission. The number of processes is set in TS 38.214; it is not given here because it has not been verified.

**Timing advance (TA).** Signals from phones at different distances take different times to arrive. The base station measures that delay and tells each phone how far ahead to transmit relative to the frame it receives (TS 38.211, clause 4.3.1; TS 38.300, clause 5.3.5.3). If the phone stops receiving corrections for a configured time, it stops sending data and must resynchronise with a random access procedure (TS 38.321, clause 5.2).

**Configured grants.** For periodic traffic, such as voice, the network can reserve recurring resources without a DCI each time. The uplink allows up to 12 active configurations per bandwidth part; the downlink, up to 8 (TS 38.300, clauses 10.2 and 10.3).

### 3. Key parameters

**Grid size.** Examples from TS 38.101-1 (Table 5.3.2-1, Rel-15):

| Channel | Spacing | Resource blocks | Slot | Slots per second |
|---|---|---|---|---|
| 20 MHz | 15 kHz | 106 | 1 ms | 1000 |
| 20 MHz | 30 kHz | 51 | 0.5 ms | 2000 |
| 100 MHz | 30 kHz | 273 | 0.5 ms | 2000 |

With 100 MHz at 30 kHz, the scheduler allocates 273 blocks in every slot. That is 546,000 block × slot cells per second. Each cell holds 12 × 14 = 168 resource elements. Some of them carry reference and control signals, not user data.

**Cyclic prefix.** It is a copy of the end of each symbol placed in front of it. It absorbs echoes and small differences in arrival time. Computed from the formulas in TS 38.211 (clause 5.3.1), with normal prefix:

| Spacing | Useful symbol | Cyclic prefix (typical symbol) |
|---|---|---|
| 15 kHz | 66.7 µs | 4.69 µs |
| 30 kHz | 33.3 µs | 2.34 µs |

**Why timing advance is needed: a calculation.** A phone 3 km from the antenna receives the signal 10 µs late (3000 m ÷ 3 × 10⁸ m/s). If it replied without transmitting early, its signal would reach the antenna 20 µs late, counting the round trip. At 30 kHz, the cyclic prefix only absorbs 2.34 µs. That is equivalent to about 350 m of distance difference between two phones (2.34 µs × 3 × 10⁸ m/s ÷ 2). Without TA, each symbol from the distant phone would overlap the next symbol from the other phones. The subcarriers would no longer be independent and would interfere with each other.

**How the scheduler decides: an illustrative example.** The figures below are made up to show the logic, not measured. Two phones share a cell. A is close and its channel would allow about 6 bit/s/Hz. B is far away and would get 1 bit/s/Hz.

| Rule | What it does | Result |
|---|---|---|
| Equal turns | Each gets half the slots | Fair in time; B receives 6 times less data than A |
| Maximum rate | Always serve whoever carries most bits right now | Maximum total; B almost never receives anything |
| Proportional fair | Serve whoever has the best current rate relative to what they have received on average | Both make progress; each tends to be served in its moments of best signal |

The proportional fair rule is known from CDMA-HDR data systems. Jalali, Padovani and Pankaj described a scheduler that picks the next terminal from the rate it requests and the data it has already received (IEEE VTC 2000). Commercial schedulers start from ideas like this and add quality-of-service priorities. Exactly what each vendor does is not public.

### 4. Diagram or flow

```text
GRID (100 MHz, 30 kHz): 273 blocks × 0.5 ms slots

 frequency ↑
  block 272 │ A │ A │ C │ B │ ...
  ...       │ A │ B │ C │ B │
  block 1   │ B │ B │ A │ A │
  block 0   │ B │ C │ A │ A │
            └───┴───┴───┴───┴──→ time (one column = one slot)
  In every column the scheduler decides again.

DOWNLINK
 Phone ──CSI (CQI)───────────────────────────────→ gNB
                                                   │ scheduler
 Phone ←──PDCCH: DCI (CRC with C-RNTI) ──────────── gNB
 Phone ←──PDSCH: data in the indicated blocks ───── gNB
 Phone ──PUCCH: HARQ-ACK / NACK ─────────────────→ gNB  (NACK → retransmission)

UPLINK
 Phone ──PUCCH: SR ("I have data") ──────────────→ gNB
 Phone ←──PDCCH: small grant ────────────────────── gNB
 Phone ──PUSCH: BSR (+ data if it fits) ─────────→ gNB
 Phone ←──PDCCH: grant sized to the volume ──────── gNB
 Phone ──PUSCH: data (with timing advance TA) ───→ gNB
```

## Limits, exceptions, and real-world cases

- **"Hundreds connected" does not mean "hundreds at once in every slot".** A phone with its screen off is usually idle or in discontinuous reception (DRX). In that state it does not monitor the PDCCH in every slot (TS 38.300, clause 10.2). In each slot only some of the phones with pending data are scheduled. How many connected phones a cell supports depends on the equipment and configuration; the standard gives no single figure.
- **The control channel fills up too.** Each DCI takes PDCCH resources. With many phones asking for little, the bottleneck can be announcing the allocation, not the data. That is why configured grants exist for periodic traffic.
- **The uplink has a start-up delay.** Between the SR, the first grant and the BSR, several slots pass before data flows. Configured grants shorten that start-up.
- **The channel report is a little stale.** The CQI describes the channel of a moment ago. If the phone moves fast, fading changes before the scheduler reacts. HARQ fixes some of the errors this causes.
- **Several phones on the same block.** With multiple antennas, the base station can send data to two phones on the same blocks if they lie in different directions (spatial multiplexing, TS 38.300, clause 5.1). The grid is still the basis of the allocation.
- **Interference from neighbouring cells is not handled by this scheduler.** Each cell schedules its own resources. Coordination between cells exists, but is outside the scope of this article.

## Practical consequences

- **In a crowded place, speed drops even with good signal.** The bars measure received power, not how many cells of the grid you get. With more active phones, each gets a smaller share.
- **Poor signal also means a smaller share.** A distant phone needs more blocks for the same bits. Many schedulers compensate for part of that gap, but not all of it.
- **Uplink latency has a floor.** Asking for permission costs several slots. That is why numerologies with shorter slots and reserved resources matter for delay-sensitive applications.
- **The same hardware can perform differently depending on the operator.** The scheduler algorithm and its configuration are choices of the vendor and the operator, not of the standard.

## Frequently asked questions

### Is this the same as how WiFi shares the channel?

No. In classic WiFi, each device listens to the channel and transmits when it finds it free, with random back-offs to avoid collisions. In a cellular network nobody sends data without permission: the base station decides who uses each block. Recent WiFi versions (IEEE 802.11ax onwards) add a similar per-subcarrier allocation (OFDMA), but it coexists with contention-based access.

### Do "full bars" guarantee speed?

No. The bars reflect signal power. Speed also depends on how many blocks the scheduler assigns you, and that depends on how many phones compete for them.

### Does my phone hear other people's data?

It receives the same radio signal, but it only decodes the blocks its DCI assigns to it. User data is also encrypted at higher layers. The C-RNTI is for addressing, not for protection.

### Why not just split everything equally?

Because the same block is worth many more bits to a nearby phone than to a distant one. Splitting equally in time wastes capacity; splitting only by rate leaves distant phones without service. Schedulers look for a middle ground.

## Sources

- 3GPP. *TS 38.300: NR; NR and NG-RAN Overall description; Stage-2* (V18.11.0, Rel-18, 2026-09). Clauses 5.1, 5.2.5, 5.3.5, 10.1-10.4. <https://www.3gpp.org/ftp/specs/PDF/2026-09/Rel-18/38_series/38300-ib0.pdf>
- ETSI / 3GPP. *TS 38.211: NR; Physical channels and modulation* (V17.8.0, Rel-17). Tables 4.2-1 and 4.3.2-1, clauses 4.3.1, 4.4.4.1 and 5.3.1. <https://etsi.org/deliver/etsi_TS/138200_138299/138211/17.08.00_60/ts_138211v170800p.pdf>
- ETSI / 3GPP. *TS 38.213: NR; Physical layer procedures for control* (V17.9.0, Rel-17). Clauses 9 and 10.1, Tables 10.1-2 and 10.1-3. <https://etsi.org/deliver/etsi_TS/138200_138299/138213/17.09.00_60/ts_138213v170900p.pdf>
- ETSI / 3GPP. *TS 38.321: NR; Medium Access Control (MAC) protocol specification* (V18.4.0, Rel-18). Clauses 5.2, 5.3.1, 5.4.1-5.4.5. <https://etsi.org/deliver/etsi_ts/138300_138399/138321/18.04.00_60/ts_138321v180400p.pdf>
- ETSI / 3GPP. *TS 38.101-1: NR; User Equipment (UE) radio transmission and reception; Part 1: Range 1 Standalone* (V15.5.0, Rel-15). Table 5.3.2-1. <https://etsi.org/deliver/etsi_TS/138100_138199/13810101/15.05.00_60/ts_13810101v150500p.pdf>
- A. Jalali, R. Padovani, R. Pankaj. "Data throughput of CDMA-HDR a high efficiency-high data rate personal communication wireless system". *IEEE VTC2000-Spring*, vol. 3, pp. 1854-1858. <https://doi.org/10.1109/VETECS.2000.851593>
