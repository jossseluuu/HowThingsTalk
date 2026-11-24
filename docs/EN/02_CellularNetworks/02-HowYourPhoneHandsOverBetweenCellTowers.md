---
title: "How does your phone switch from one cell tower to another without dropping the connection?"
section: "02_CellularNetworks"
language: "en"
status: "published"
order: 2
depth: "intermediate"
summary: "While connected, the phone keeps measuring the signal of its own cell and of the neighbouring ones. When a neighbour beats the current cell by a margin, and for a time, both set by the network, the phone sends a report. The current base station decides the handover, prepares the new one over the network and tells the phone to jump. The jump interrupts data for a few tens of milliseconds, but packets in transit are forwarded to the new base station instead of being lost."
prerequisites:
  - "How a cell tower shares its capacity: base station, control channel and C-RNTI (article 02·01)"
  - "Why signal strength changes over a few centimetres (article 00·03)"
translation: "../../ES/02_RedesMoviles/02-ComoPasaElMovilDeUnaAntenaAOtra.md"
---

# How does your phone switch from one cell tower to another without dropping the connection?

## Question and framing

You are on a train or in a car with a video call open. In half an hour you cover tens of kilometres and cross the coverage of many different cells. The call keeps going. The question is how that relay works. Who decides it is time to switch cells, on what basis, and what happens to the data that was on its way at that moment?

This article explains handover using the 3GPP 5G NR standard. 4G (LTE) uses the same scheme of measurements, events and preparation between base stations, with different message names.

- **Depth:** intermediate. There is one inequality in decibels and some arithmetic with milliseconds.
- **Prerequisites:** what a base station, the control channel and the temporary C-RNTI identifier are, explained in [How does a cell tower share its capacity among all the connected phones at once?](01-HowACellTowerSharesItsCapacity.md). It helps to know why the signal fluctuates as you move, explained in [Why does signal strength change so much when you move a phone a few centimetres?](../00_Foundations/03-WhySignalChangesOverAFewCentimetres.md).
- **Out of scope:** how the phone finds the network when it is switched on, how it picks a cell while idle (cell reselection, which the phone decides itself), switching between 4G and 5G, and the details of the operator's core network.

## Short answer

The phone keeps measuring its own cell and the neighbours, reports when a neighbour is clearly better, and the current base station prepares the new one over the network before telling the phone to jump; so the gap lasts a few tens of milliseconds and pending data is forwarded instead of lost.

### 60-second summary

With an active connection, the phone does not decide on its own which cell to talk to. The network tells it what to measure and when to report. The phone listens to synchronisation signals that every cell broadcasts periodically. It measures their power and averages it to smooth out fast fluctuations.

A typical rule is "tell me if a neighbour beats the current cell by 3 dB for 320 ms". When that happens, the phone sends a measurement report. The current base station reads it and decides whether to hand over.

If it does, it asks the target base station for permission over a link between the two. The target reserves resources and returns a ready-to-use configuration. The current cell passes it to the phone. The phone lets go of the old cell, synchronises with the new one and confirms its arrival.

No data flows during the jump. In a typical case the standard limits that gap to about 40 ms plus the wait until the next access opportunity in the new cell. Meanwhile, the old base station forwards to the new one the packets it did not manage to deliver. Finally, the core network moves the traffic to the new base station, and the old one releases everything it held for the phone.

## Mental model

Think of a relay race. The runner holding the baton sees the next runner approach. The baton does not go to the first person who shows up, but to the one clearly in position. Before the handoff, the next runner has already been warned and is moving. When the baton changes hands, the race goes on.

The baton is your connection and its context: identity, encryption keys and pending data. The runners are the base stations. The advance warning is the handover preparation between stations. "Clearly in position" is the margin and the time the network demands before switching.

**Where the analogy breaks down.** In a relay, the runners see each other and decide for themselves. Here, the base station cannot see where you are: it depends on what the phone measures and reports. Also, the decision is made by the station letting go of the baton, not the one receiving it, although the new one can refuse if it has no room. And in a normal handover there is a moment when nobody holds the baton. The phone lets go of the old cell before latching on to the new one, which is why there is a brief gap.

## Technical mechanism

### 1. What happens in the system

The standard distinguishes two kinds of mobility while the phone is connected (TS 38.300, clause 9.2.3.1). Switching **beams** within the same cell is handled by the physical and MAC layers, with no RRC messages. Switching **cells** requires explicit signalling: that is handover. The network controls the process and the phone supplies the measurements.

A typical handover between two 5G base stations linked by the Xn interface follows these steps (TS 38.300, clause 9.2.3.2.1, figure 9.2.3.2.1-1):

1. **Measurement configuration.** The current (source) base station tells the phone which frequencies to measure and which conditions should trigger a report. The phone measures and, when a condition is met, sends a measurement report.
2. **Decision.** The source station decides on handover based on the report and its own resource management information.
3. **Handover request.** The source sends a *Handover Request* message to the target station over Xn. It includes the target cell, a key derived for the target, the phone's current configuration, its capabilities and its data sessions.
4. **Admission control.** The target checks whether it can accept the phone.
5. **Acceptance.** The target prepares its resources and replies with *Handover Request Acknowledge*. Inside is the message the phone will receive, already built.
6. **Command to the phone.** The source forwards that message to the phone: an *RRCReconfiguration*. At a minimum it contains the target cell's identity, the new C-RNTI and the target's security algorithms. It can include reserved random access resources and the new cell's system information, so the phone does not have to read it.
7. **Transfer status.** The source tells the target which sequence numbers the packets have reached in each direction (*SN Status Transfer*). That way the target knows what is still to be delivered and what has already arrived.
8. **Jump.** The phone detaches from the old cell, synchronises with the new one, performs random access and sends *RRCReconfigurationComplete* to the target. From that moment the phone is talking to the new cell.
9. to 11. **Path switch.** The target asks the core network (AMF) to redirect traffic (*Path Switch Request*). The UPF, which routes user data, switches the downlink path to the new station. On the old path it sends a few "end marker" packets to signal that nothing more will arrive that way.
12. **Release.** The target notifies the source (*UE Context Release*) and the source frees the phone's resources.

Between steps 5 and 12, the source can forward to the target the downlink packets it had not yet delivered (TS 38.300, clause 9.2.3.2.2). The target sends those first, before the new packets arriving over the switched path.

### 2. What each component does

**The signals being measured.** Each cell periodically broadcasts synchronisation blocks (SS/PBCH, or SSB) carrying its physical identifier (PCI). The phone can also measure CSI-RS reference signals if the network configures them. From these it computes three quantities: RSRP (reference signal received power), RSRQ (quality) and SINR (signal to interference plus noise ratio) (TS 38.331, clause 5.5.3.1). In FR1, the phone must be able to measure at least 8 cells per frequency (TS 38.133 V15.4.0, clause 9.2.3.1).

**The layer 3 filter.** A single measurement fluctuates a lot because of fading. The phone smooths it with an exponential average (TS 38.331, clause 5.5.3.2):

```text
F(n) = (1 − a) · F(n−1) + a · M(n),   with a = 1 / 2^(k/4)
```

M(n) is the new measurement and F(n) the filtered value. The network sets the coefficient k. Its default is 4, which gives a = 0.5: each new measurement weighs half and the history the other half. Filtering is done in decibels.

**Events.** The network does not ask for constant reports. It defines named conditions, called events (TS 38.331, clauses 5.5.4.2 to 5.5.4.7). The ones most used for handover:

| Event | Triggers when… |
|---|---|
| A2 | the current cell drops below a threshold |
| A3 | a neighbour beats the current cell by a margin |
| A4 | a neighbour rises above a threshold |
| A5 | the current cell drops below one threshold while a neighbour rises above another |

**The A3 condition in detail.** It is the most common trigger. It is met when (TS 38.331, clause 5.5.4.4):

```text
Mn + Ofn + Ocn − Hys  >  Mp + Ofp + Ocp + Off
```

- Mn and Mp: filtered measurement of the neighbour and of the current cell, in dBm (RSRP) or dB (RSRQ, SINR).
- Ofn and Ofp: per-frequency adjustment; Ocn and Ocp: per-cell adjustment. They are used to favour or penalise cells. They are 0 dB if not configured.
- Off: the margin (a3-Offset), from −15 to +15 dB in 0.5 dB steps.
- Hys: the hysteresis, from 0 to 15 dB in 0.5 dB steps.

To stop being met, the inequality is reversed with the hysteresis added instead of subtracted. That gap between "entering" and "leaving" keeps the condition from flickering when both signals are level.

**Time to trigger (timeToTrigger).** The condition must hold in every filtered measurement for a fixed time before the report is sent (TS 38.331, clause 5.5.4.1). There are 16 possible values, from 0 to 5120 ms: 0, 40, 64, 80, 100, 128, 160, 256, 320, 480, 512, 640, 1024, 1280, 2560 and 5120 ms (TS 38.331, clause 6.3.2).

**The measurement report.** It carries the results for the current cell and for the neighbours that triggered the event. Each neighbour comes with its PCI and its already filtered RSRP, RSRQ or SINR values (TS 38.331, clause 5.5.5). The phone only sends it once the connection's security is active.

**The source station.** It decides on handover. The standard defines the messages but not the decision algorithm: which events to configure and with which values is chosen by the vendor and the operator.

**The Xn interface.** It links 5G base stations to each other. The request, the acceptance, the transfer status and the forwarded data travel over it. If two stations have no Xn link, handover goes through the core network (not detailed here).

**The T304 timer.** It starts when the phone begins executing the handover command and stops when it completes random access in the new cell (TS 38.331, clauses 5.3.5.5.2 and 5.3.5.3). Possible values: 50, 100, 150, 200, 500, 1000, 2000 or 10,000 ms. If it expires, the handover has failed and the phone starts a connection re-establishment with the cause "handover failure" (TS 38.331, clauses 5.3.5.8.3, 5.3.7.2 and 5.3.7.4).

### 3. Key parameters

**A triggering example.** Illustrative values, not from any particular operator: Off = 2 dB, Hys = 1 dB, no per-cell or per-frequency adjustments, timeToTrigger = 320 ms. The A3 condition becomes:

```text
Mn − 1 > Mp + 2   →   Mn > Mp + 3 dB
```

If the current cell measures −95 dBm, the neighbour must exceed −92 dBm for 320 ms in a row. At 120 km/h (33.3 m/s), the phone travels about 10.7 m in those 320 ms. A larger margin or a longer time delays the handover. Smaller ones bring it forward, but raise the risk of bouncing back and forth between two cells.

**The gap during the jump.** The standard sets how long the phone may take (TS 38.133 V15.4.0, clause 6.1.1.2, FR1 to FR1 handover). The total delay is the time to process the RRC message plus the interruption time:

```text
T_interrupt = T_search + T_IU + 20 ms + T_Δ
```

- **T_search:** 0 ms if the target cell is "known". It is known if the phone has identified it within the last 5 seconds. If not, the time to find it is added.
- **T_Δ:** time for fine timing adjustment. It equals one period of the target cell's SSB measurement window (Trs).
- **T_IU:** the wait until the first random access opportunity in the new cell. It depends on how the target cell configures those opportunities.
- **20 ms:** fixed processing allowance.

With a known cell and Trs = 20 ms: 0 + T_IU + 20 + 20 = **40 ms plus T_IU**. This is an upper bound the phone must meet, not a measured value. The RRC processing time added on top is in TS 38.331, clause 12; the figure is not given here because it has not been verified.

**Data loss.** On acknowledged channels (RLC acknowledged mode), the transfer status and forwarding allow a lossless handover. For that, the target must keep the same mapping of quality of service flows to bearers (TS 38.300, clauses 9.2.3.2.1 and 9.2.3.2.2).

### 4. Diagram or flow

```text
 Phone                Source (gNB A)          Target (gNB B)         Core (AMF/UPF)
   │ ◄── meas. config ──────│                        │                        │
   │  measures SSB of A and B; filters; A3 holds 320 ms                        │
   │ ── measurement report ─►│                        │                        │
   │                         │ decides handover       │                        │
   │                         │ ── Handover Request ──►│ admission control      │
   │                         │ ◄── Request Ack ───────│ (includes the command) │
   │ ◄── RRCReconfiguration ─│                        │                        │
   │   (cell B, new C-RNTI, security, RACH)           │                        │
   │                         │ ── SN Status Transfer ►│                        │
   │                         │ ══ data forwarding ═══►│                        │
   │ ✂ leaves cell A ········│······ gap: ~40 ms + T_IU ·······                │
   │ ── random access + RRCReconfigurationComplete ───►│                       │
   │ ◄═════════════ data (forwarded packets first) ════│                       │
   │                         │                        │ ── Path Switch ───────►│
   │                         │ ◄ end marker ──────────────────────────────────── UPF
   │                         │                        │ ◄── Path Switch Ack ───│
   │                         │ ◄── UE Context Release │                        │
   │                         │ releases resources     │                        │
```

## Limits, exceptions, and real-world cases

- **Ping-pong.** At a border where two cells arrive almost equally strong, fading can make each one win in turn. Every handover costs signalling and a gap. Hysteresis, margin and time to trigger exist to damp that bouncing. In exchange, cautious settings delay the handover when it is really needed.
- **Arriving late.** If the phone moves fast or the signal drops suddenly, for example on entering a tunnel, the current cell may be lost before the command arrives. The handover then fails. The phone looks for a cell and asks to re-establish the connection. The new station retrieves its context from the previous one (TS 38.300, clause 9.2.3.3). The gap is much longer than in a normal handover.
- **Conditional handover (CHO).** To avoid depending on the command arriving in time, the network can send it in advance (TS 38.300, clause 9.2.3.4). It prepares one or more candidate cells and gives the phone their configurations together with an execution condition based on events A3 or A5. The phone stays on the current cell and evaluates the condition. When it is met for a candidate, the phone jumps there on its own, without waiting for further messages.
- **Handover without a downlink gap (DAPS).** With DAPS, the phone keeps both connections at once (TS 38.300, clauses 9.2.3.1 and 9.2.3.2.2). It keeps receiving from the old cell until it releases it, and keeps transmitting to it until access to the new one succeeds. It requires the phone to run two protocol stacks in parallel and disables features such as carrier aggregation. It cannot be combined with conditional handover.
- **Layer 1/2 triggered mobility (LTM).** Since Release 18, the network can preconfigure candidate cells over RRC and order the switch with a short MAC message, based on layer 1 measurements (TS 38.300, clause 9.2.3.5). It reduces signalling at the moment of the switch.
- **Without random access.** In some cases, such as a handover between cells of the same base station, the phone can reuse its timing advance and skip random access (TS 38.300, clause 9.2.3.6).
- **What the standard does not fix.** Which events are configured, with which margins and times, and how to choose among several candidates are chosen by the vendor and the operator. Two networks using the same equipment can behave differently.

## Practical consequences

- **The phone does not switch "to the strongest cell" the moment it appears.** It waits for the neighbour to be better by a margin and for a time. That is why you sometimes stay on a weak cell even though a better one is nearby.
- **A video call survives handover, but not always without a trace.** A gap of a few tens of milliseconds with forwarded data is usually absorbed by the application's buffer. A failed handover followed by re-establishment is much longer and can be noticeable.
- **Signalling grows with speed.** On a fast train, handovers are more frequent and come with less margin. Conditional handover and per-cell parameter tuning exist for these cases.
- **The bars can jump suddenly.** After a handover, the indicator reflects the new cell. An abrupt change in bars while you barely move may be just that.

## Frequently asked questions

### Is this the same as an idle phone changing cells?

No. When idle there is no active connection to preserve. The phone itself chooses the cell according to rules the network broadcasts (cell reselection), without asking permission or preparing anything. The handover in this article only happens while the phone is connected, and the network decides it.

### Is this the same as roaming between WiFi access points?

Not quite. In WiFi, traditionally the device decides when to change access point and associates with the new one on its own. In a cellular network, the source station decides, prepares the target and hands the phone a ready-made configuration. There are WiFi extensions that speed up that switch, but the division of roles is different.

### Is switching beams a handover?

No. Within one cell, a 5G antenna can transmit several beams. Beam switching is handled by the physical and MAC layers, with no RRC messages and no change of C-RNTI (TS 38.300, clause 9.2.3.1). Handover is the change of cell.

### Is data lost during handover?

On acknowledged flows, in principle no. The old station tells the new one where it had got to and forwards whatever is pending. During the gap there is delay, not loss. On unacknowledged flows, whatever was on the air at that instant can be lost.

## Sources

- ETSI / 3GPP. *TS 38.300: NR; NR and NG-RAN Overall description; Stage-2* (V18.6.0, Rel-18). Clauses 9.2.3.1, 9.2.3.2.1, 9.2.3.2.2, 9.2.3.3, 9.2.3.4, 9.2.3.5 and 9.2.3.6. <https://www.etsi.org/deliver/etsi_ts/138300_138399/138300/18.06.00_60/ts_138300v180600p.pdf>
- ETSI / 3GPP. *TS 38.331: NR; Radio Resource Control (RRC) protocol specification* (V15.3.0, Rel-15). Clauses 5.3.5, 5.3.7, 5.5.3, 5.5.4, 5.5.5 and 6.3.2. <https://www.etsi.org/deliver/etsi_ts/138300_138399/138331/15.03.00_60/ts_138331v150300p.pdf>
- ETSI / 3GPP. *TS 38.133: NR; Requirements for support of radio resource management* (V15.4.0, Rel-15). Clauses 6.1.1.2 and 9.2.3.1. <https://www.etsi.org/deliver/etsi_ts/138100_138199/138133/15.04.00_60/ts_138133v150400p.pdf>
