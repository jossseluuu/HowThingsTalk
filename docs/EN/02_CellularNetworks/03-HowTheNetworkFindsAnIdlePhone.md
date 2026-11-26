---
title: "How does the network find your phone to tell it about a call when it is idle?"
section: "02_CellularNetworks"
language: "en"
status: "published"
order: 3
depth: "intermediate"
summary: "When your phone is idle, the network does not know which cell it is in: it only knows an area made of several cells, the one the phone declared when it registered. When a call arrives, every cell in that area broadcasts a page carrying the phone's temporary identity. The phone does not listen all the time: it wakes up at fixed moments, worked out from that identity, and goes back to sleep if the page is not for it."
prerequisites:
  - "How a cell tower shares its capacity: base station, PDCCH control channel and RNTI identifiers (article 02·01)"
  - "How a phone moves from one cell tower to another while connected (article 02·02)"
translation: "../../ES/02_RedesMoviles/03-ComoEncuentraLaRedATuMovilEnReposo.md"
---

# How does the network find your phone to tell it about a call when it is idle?

## Question and framing

Your phone has been in your pocket for an hour with the screen off. It has no open connection to the network. It may have switched cells several times without telling anyone. Yet when someone calls you, it rings within a second or two. The question is how the network pulls this off. How does it know where to look for you? And how does the phone learn that it is being looked for without keeping its radio on all the time?

The mechanism is called paging. This article explains it with the 3GPP 5G NR standard. 4G (LTE) uses the same scheme of areas, pages and listening moments, with different names and a different identity for working out those moments.

- **Depth:** intermediate. There is one formula with modulo operations and a worked example.
- **Prerequisites:** what a base station, the PDCCH control channel and RNTI identifiers are, explained in [How does a cell tower share its capacity among all the connected phones at once?](01-HowACellTowerSharesItsCapacity.md). It helps to have read [How does your phone switch from one cell tower to another without dropping the connection?](02-HowYourPhoneHandsOverBetweenCellTowers.md), which covers the opposite case: a connected phone.
- **Out of scope:** how the phone finds the network at power-on, the exact rules it uses to pick a cell while idle, how the voice call is set up afterwards, and registration security.

## Short answer

When your phone is idle, the network only knows which area of several cells you are in, so it sends the page through every cell in that area, and the phone hears it because it wakes up at fixed moments derived from its temporary identity.

### 60-second summary

When the phone registers, the core network hands it a list of tracking areas. Each area groups many cells. As long as the phone moves within that list, it tells no one. If it enters a cell in an area that is not on the list, it registers again. It also registers every so often even if it does not move: by default, every 54 minutes.

When a call or data arrives for the phone, the core network asks the cells in that area to page it. Each base station broadcasts a paging message carrying the phone's temporary identity.

The phone does not watch the paging channel all the time. It listens during just one paging occasion per cycle, and a cycle usually lasts from a fraction of a second to a couple of seconds. That occasion is not random: the phone and the base station each compute it with the same formula and the same identity. So the base station knows when to speak and the phone knows when to wake up.

If the page carries its identity, the phone asks for a connection. If not, it goes back to sleep until the next occasion. That is why an idle phone uses little battery, and why a call takes a moment to ring.

## Mental model

Think of a large airport. You are somewhere in terminal 2, but nobody knows at which gate. If someone needs you, they do not walk the terminal checking faces: they make an announcement over the speakers across the whole terminal. And you do not pay attention to every announcement. You only listen to the ones that start in the minutes assigned to you. If you move to another terminal, you stop at a desk so they know where to look for you.

The terminal is the tracking area. The speaker system is the page that every cell in the area broadcasts. The desk is the new registration. Your assigned minutes are your paging occasion.

**Where the analogy breaks.** In the airport you hear every announcement even if you ignore it. The phone, by contrast, really switches off most of its radio between occasions: if the network spoke outside its slot, the phone would not hear it. That is why the slot formula must be the same on both sides. Also, the announcement does not use your real name. It uses a temporary identity that the network changes often, so nobody can follow you by listening to the pages.

## Technical mechanism

### 1. What happens in the system

**The phone's three states.** The standard defines three radio connection states (TS 38.300, clause 7.2):

| State | Who knows where the phone is | Who starts the page |
|---|---|---|
| Connected (RRC_CONNECTED) | The base station knows its cell | No page needed |
| Inactive (RRC_INACTIVE) | The base station knows a notification area (RNA) | The radio network (RAN paging) |
| Idle (RRC_IDLE) | The core network knows its tracking area list | The core network, 5GC (CN paging) |

In idle and inactive, the phone chooses its own cell: what the standard calls camping on a cell (TS 38.304, clause 4.1). The phone monitors that cell's control channel, its system information and, in most cases, paging (TS 38.304, clause 3.1).

**Registration.** The phone registers in the tracking area of the cell it camps on. If it moves to a cell that belongs to none of its registered areas, it registers again. In inactive, if it leaves its notification area, it performs a RAN area update (TS 38.304, clause 4.1). The standard puts it this way: since the network knows, in most cases, the set of areas where the phone is camped, it can send the page on the control channels of all the cells in that set (TS 38.304, clause 4.1).

**The sequence of an incoming call, with the phone idle:**

1. The core network receives something for the phone: a call or a data packet. In idle, paging is started by the core network (TS 38.300, clause 7.2).
2. The core network asks the base stations in the phone's areas to page it. It can start with a smaller area and widen it on later attempts. To do this it passes the base station the attempt number and the intended scope of the next one (TS 38.300, clause 9.2.5). The strategy is the operator's choice.
3. Each base station works out when that phone listens and, at that moment, sends on the control channel an indication addressed to the common paging identifier, the P-RNTI. That indication points to the *Paging* message, which carries the phone's temporary identity (TS 38.300, clause 9.2.5; TS 38.331, clause 5.3.2).
4. The phone wakes up at its occasion, detects the P-RNTI, reads the message and finds its identity.
5. The phone starts random access and asks to set up a connection. From there, the call proceeds with the phone connected.

In inactive the scheme is the same, but the page is started by the base station that holds the phone's context. It sends it through the cells of the notification area and with a different identity: the full I-RNTI (TS 38.300, clauses 7.2 and 9.2.5; TS 38.331, clause 5.3.2).

### 2. What each component does

**The tracking area.** Every cell broadcasts which area it belongs to. The area is identified by a TAI: the operator's identity plus an area code, the TAC (TS 38.300, clause 8.2). When it registers, the phone receives a list of TAIs. It must perform a mobility registration when it detects that its cell's TAI is not in that list (TS 24.501, clause 5.5.1.3.2).

**Periodic registration.** Timer T3512 makes the phone register every so often even if it does not move. The operator sets its value at registration. If it does not, the value is 54 minutes. The phone starts it when it goes idle. When it expires, the phone performs a periodic registration (TS 24.501, table 10.2.1). This way the network knows the phone is still there.

**The network-side timer.** The core network starts a mobile reachable timer when the phone goes idle. By default it lasts 4 minutes longer than T3512: 58 minutes with the default value. If it expires without news from the phone, the network typically stops paging it and starts another timer. When that second timer expires, the network implicitly deregisters the phone (TS 24.501, table 10.2.2).

**The temporary identity.** The phone is not paged with its permanent identity. The network assigns it a temporary one, the 5G-S-TMSI, a 48-bit string (TS 38.304, clause 7.1). The paging message carries the 5G-S-TMSI for core network paging or the full I-RNTI for RAN paging (TS 38.331, clause 5.3.2).

**The *Paging* message.** It travels on the paging logical channel (PCCH), without any prior connection to the phone. It contains a list of records. Each record carries one identity. A single message can page several phones at once. Since release 17 of the standard, each record can also state that the cause is a voice call (TS 38.331, clause 5.3.2).

**Short messages.** The same P-RNTI is also used for short messages carried directly on the control channel. They tell every phone that system information has changed or that there is a public emergency alert (ETWS or CMAS). These messages reach connected phones too (TS 38.300, clause 9.2.5).

### 3. Key parameters

**The discontinuous reception (DRX) cycle.** In idle or inactive, the phone only has to monitor the paging channel during one paging occasion (PO) per DRX cycle (TS 38.300, clause 9.2.5). The cycle can come from three places (TS 38.300, clause 9.2.5; TS 38.304, clause 7.1):

- a default cycle broadcast by each cell;
- a phone-specific cycle agreed with the core network;
- in inactive, a phone-specific cycle set by the radio network.

The phone uses the shortest one that applies to it. The cycle is measured in radio frames. Each frame lasts 10 ms: the standard counts 100 frames per second (TS 38.304, clause 7.4). The usual values of the normal cycle are 32, 64, 128 and 256 frames, that is, 0.32, 0.64, 1.28 and 2.56 s (TS 38.331, *PagingCycle* field; **unverified** in the text consulted).

**The slot formula.** The paging frame (PF) is the radio frame in which the phone's occasion can start. Its system frame number (SFN) satisfies (TS 38.304, clause 7.1):

```text
(SFN + PF_offset) mod T = (T div N) · (UE_ID mod N)
i_s = floor(UE_ID / N) mod Ns
UE_ID = 5G-S-TMSI mod 1024
```

- **T:** the DRX cycle, in frames.
- **N:** how many paging frames there are in each cycle. Set by the network.
- **PF_offset:** an offset set by the network.
- **Ns:** how many paging occasions there are in each paging frame.
- **i_s:** which of those occasions the phone uses.
- **UE_ID:** the low 10 bits of the temporary identity. If the phone has no 5G-S-TMSI yet, it uses 0.

The base station knows the 5G-S-TMSI because the core network passes it along with the paging request. So both sides arrive at the same frame without talking to each other.

**An example.** Illustrative values, not from any particular operator: T = 128 frames (1.28 s), N = 32, PF_offset = 0, Ns = 2. Suppose the phone's 5G-S-TMSI gives UE_ID = 700.

```text
UE_ID mod N = 700 mod 32 = 28
T div N     = 128 / 32   = 4
SFN mod 128 = 4 · 28     = 112      → frames 112, 240, 368, …
i_s = floor(700 / 32) mod 2 = 21 mod 2 = 1   → second occasion in the frame
```

The phone wakes up every 1.28 s around one 10 ms frame. It can sleep for the rest of the cycle. If the call arrives just after its occasion, the network has to wait for the next one: up to 1.28 s in this example, plus whatever the rest of the process takes.

**Why one occasion means several "looks".** In 5G, the cell transmits on several beams. A paging occasion is a set of S·X consecutive opportunities to monitor the control channel. S is the number of synchronization blocks (SSB) the cell transmits, one per beam. X is how many opportunities there are for each (TS 38.304, clause 7.1). This lets the base station repeat the page on every beam, because it does not know which direction the phone is in. If X is greater than 1 and the phone detects the P-RNTI in one of them, it does not have to monitor the rest of that occasion (TS 38.304, clause 7.1).

**Extended DRX (eDRX).** For devices that can afford to answer late, the network can configure much longer cycles. They range from 2.56 s to 10,485.76 s, about 2.91 hours (TS 38.300, clause 9.2.10). Above 10.24 s, the phone does not monitor every cycle. It only listens during a periodic window (PTW) inside a specific hyperframe (PH) (TS 38.300, clause 9.2.10; TS 38.304, clause 7.4). A hyperframe is 1,024 frames, 10.24 s. The maximum cycle, 1,024 hyperframes, gives the standard's 10,485.76 s. With eDRX, UE_ID becomes the 5G-S-TMSI modulo 4096 (TS 38.304, clause 7.1).

**Paging early indication (PEI).** Many phones share the same paging occasion. If one of them is being paged, all the others wake up for nothing. Since release 17, the network can send an early indication (PEI) shortly before the occasion (TS 38.304, clause 7.2). The phones in an occasion are split into subgroups, up to 8 per cell (TS 38.300, clause 9.2.5). The phone monitors one PEI occasion per cycle. If it detects no PEI, or the PEI does not flag its subgroup, it does not have to monitor the paging occasion (TS 38.304, clause 7.2). The standard justifies this as a way to reduce power consumption, but does not quantify the saving.

### 4. Diagram or flow

```text
 REGISTRATION (before the call)
 Phone ── registers in tracking area TA 7 ─────────────────────────► Core network (AMF)
 Phone ◄──────────────────── area list {TA 7, TA 8} + 5G-S-TMSI ─── AMF
 Phone: goes idle, starts T3512 (54 min by default) and camps on its own

 DISCONTINUOUS RECEPTION (example: T = 1.28 s)
 time ─►  ·····█···········█···········█···········█·····
               PO          PO          PO          PO      (█ = listens; · = sleeps)

 INCOMING CALL
 AMF ── paging request (5G-S-TMSI) ──► gNB A (TA 7), gNB B (TA 7), gNB C (TA 8) …
 gNB A, B, C: compute PF and PO from UE_ID = 5G-S-TMSI mod 1024
 gNB A, B, C ── at the PO: PDCCH with P-RNTI + Paging message, beam by beam ──► all their cells
 Phone (camped on a cell of gNB B): wakes up at its PO
                                     is my 5G-S-TMSI in the list?
                                     ├─ no  → back to sleep
                                     └─ yes → random access + connection request
 Phone ── connection set up ──► gNB B ──► AMF: the network now knows the exact cell
```

## Limits, exceptions, and real-world cases

- **Area size is a trade-off.** A large area list avoids registrations, but every page occupies the control channel of many cells. A small list saves pages, but forces more frequent registrations. The standard provides the tools, not the size. Each operator decides.
- **The network may not find you.** In a basement or with no coverage, the phone does not hear the page. The network retries according to its strategy. If the mobile reachable timer expires, it typically stops paging (TS 24.501, table 10.2.2). What then happens to the call, for example whether it goes to voicemail, is up to the operator.
- **The slot depends on an identity that changes.** In 4G, the paging occasion was computed from the permanent identity (IMSI). In 5G it is computed from the temporary one, the 5G-S-TMSI (Ericsson, 2019). Also, in 5G the 5G-S-TMSI must be refreshed after paging, and the I-RNTI is used only once. In 4G, refreshing the temporary identity after paging was optional (Ericsson, 2019). As a result, listening to pages and occasions does not reveal the phone's permanent identity.
- **A phone without a temporary identity.** Before registering, the phone uses UE_ID = 0 (TS 38.304, clause 7.1). In practice this only happens right after power-on.
- **Inactive listens for two kinds of page.** A phone in inactive monitors both radio network paging and core network paging. Both use the same identity to compute the occasion, so they overlap in time (TS 38.300, clause 9.2.5).
- **eDRX is not for a phone in normal use.** With cycles of hours, a call would not get through in time. It is meant for devices that tolerate long waits. Whether a particular phone uses it, and with which values, depends on the device and the operator: **unverified** here.

## Practical consequences

- **A call takes a moment to ring.** Part of the delay is waiting for the next paging occasion: up to a full cycle. Then come random access, the connection and the rest of the call setup.
- **Idle is cheap on battery.** The phone sleeps for most of the cycle and only wakes around its occasion. PEI and eDRX cut those wake-ups even further.
- **Moving while idle generates almost no traffic.** Changing cells within the area list needs no message at all. Only crossing into a new area, or T3512 expiring, does.
- **App notifications take the same path.** A message arriving at an idle phone is incoming data. To deliver it, the network first has to page the phone and wait for it to connect.
- **For network designers, paging is a scarce resource.** Every page occupies the control channel in every cell of the area. That is why there is staged paging by area, grouping of several phones into one message, and subgroups.

## Frequently asked questions

### Does the network know which cell I am in when my phone is idle?

No. It only knows the tracking area list you registered in. It learns the exact cell when the phone connects. That is why the page is sent through many cells at once.

### Is this the same as cell reselection?

No. Reselection is how the phone chooses, on its own, the cell it camps on. Paging is how the network looks for it. They are related: the phone reads pages in the cell it has chosen, and if that cell is outside its area list, it registers again.

### Is this the same as discontinuous reception on a connected phone?

They are similar, but not the same. A connected phone can also sleep between listening windows, with cycles specific to that state. There, the base station knows its cell and addresses it by its own identifier, the C-RNTI. In idle there is no per-phone identifier in the cell: everyone listens to the same P-RNTI and each phone looks for its identity inside the message.

### Can someone track me by listening to pages?

The 5G design makes it harder. The page carries a temporary identity, not the permanent one. The occasion is computed from that temporary identity, and the network must refresh it after each page (Ericsson, 2019). This article does not go into specific attacks.

## Sources

- ETSI / 3GPP. *TS 38.304: NR; User Equipment (UE) procedures in Idle mode and RRC Inactive state* (V18.1.0, Rel-18). Clauses 3.1, 4.1, 7.1, 7.2, 7.3 and 7.4. <https://www.etsi.org/deliver/etsi_ts/138300_138399/138304/18.01.00_60/ts_138304v180100p.pdf>
- ETSI / 3GPP. *TS 38.300: NR; NR and NG-RAN Overall description; Stage-2* (V18.3.0, Rel-18). Clauses 7.2, 8.2 and 9.2.5. <https://www.etsi.org/deliver/etsi_ts/138300_138399/138300/18.03.00_60/ts_138300v180300p.pdf>
- 3GPP. *TS 38.300*, clause 9.2.10 (*Extended DRX for RRC_IDLE and RRC_INACTIVE*), Rel-19 text read from a copy of the standard. <https://itecspec.com/spec/3gpp-38-300-9-2-10-extended-drx-for-rrc_idle-and-rrc_inactive>
- ETSI / 3GPP. *TS 38.331: NR; Radio Resource Control (RRC) protocol specification* (V17.5.0, Rel-17). Clause 5.3.2 (*Paging* message). <https://www.etsi.org/deliver/etsi_ts/138300_138399/138331/17.05.00_60/ts_138331v170500p.pdf>
- 3GPP. *TS 24.501: Non-Access-Stratum (NAS) protocol for 5G System (5GS); Stage 3* (Rel-19). Tables 10.2.1 and 10.2.2 and clause 5.5.1.3.2, read from copies of the standard. <https://itecspec.com/spec/3gpp-24-501-10-list-of-system-parameters> · <https://whatthespec.net/sos/spec/ts-24501/5.5.1.3.2>
- Ericsson. Technical article by Ohlsson, Nakarmi and Hedman on 5G paging privacy (May 2019). Identity used to compute the occasion in 4G and 5G, identities allowed in paging, and mandatory refresh after paging. <https://www.ericsson.com/en/blog/2019/5/fighting-imsi-catchers-5g-cellular-paging-privacy>
