---
title: "How does TCP know how fast it can send without overloading the network?"
section: "04_DataTransfer"
language: "en"
status: "published"
order: 2
depth: "intermediate"
summary: "No device knows in advance how much spare capacity the path to the other end has. TCP finds out by probing: it increases the amount of data in flight while acknowledgments keep arriving and cuts it back when it detects loss or congestion marks."
prerequisites:
  - "How TCP delivers data in order (sequence numbers, ACKs, and retransmission)"
  - "Latency, RTT, and throughput"
translation: "../../ES/04_TransferenciaDeDatos/02-ComoSabeTCPAQueVelocidadEnviar.md"
---

# How does TCP know how fast it can send without overloading the network?

## Question and framing

When a phone downloads a file, dozens of links sit between the server and the screen: fibre, routers, the home WiFi. Each has its own capacity and shares it with other users who come and go without warning. The server sees none of this. It only sees the data it sends and the acknowledgments that come back.

The question is how TCP decides how much data it can have "in the air" at any moment without overflowing the network or wasting it. This article explains sender-side congestion control: slow start, congestion avoidance, the reaction to loss, CUBIC, ECN, and the idea behind BBR. It does not cover how TCP orders and retransmits bytes; that is in [How does TCP deliver data in order when packets arrive out of order?](01-HowTCPDeliversDataInOrder.md), which is worth reading first. It also does not cover queue management inside routers or other transports such as QUIC. The depth is intermediate: there are a few formulas, each with a worked number.

## Short answer

TCP does not know the speed of the path: it discovers it by trial and error, widening a window of data in flight while acknowledgments arrive smoothly and shrinking it as soon as a congestion signal appears.

### 60-second summary

The sender keeps a variable called the congestion window (`cwnd`). It says how many bytes may be sent and still unacknowledged. Each window takes one round-trip time (RTT) to be acknowledged, so the average rate is roughly `cwnd` divided by the RTT.

At the start the window is small and roughly doubles every RTT. This is "slow start", which is slow in name only. When a loss appears, the sender takes it to mean some link has filled up. It cuts the window and switches to more cautious growth. The result is a sawtooth: climb, hit the limit, drop, climb again.

The classic rule (Reno) adds one segment per RTT and halves the window after a loss. CUBIC, the default algorithm in Linux, Windows, and Apple's systems, regrows the window along a cubic curve and cuts it by only 30%. BBR, still experimental at the IETF, takes a different approach: instead of waiting for loss, it measures the path's bandwidth and minimum RTT and sends at that pace.

## Mental model

Imagine filling a long hose you cannot see, which ends in a tank with a drain of unknown capacity. You only know the water arrives because someone at the far end calls out each time a litre goes in. You open the tap a little and keep opening it while the calls come back steadily. If the tank overflows, you notice because water is lost and calls go missing. So you close the tap a good deal and start opening it again, more slowly this time.

The analogy captures the core idea: the sender regulates without seeing the network and learns from what comes back. It breaks down in three places. First, TCP does not regulate a continuous flow but an amount of data awaiting acknowledgment; the rate follows from how long the acknowledgments take to return. Second, there is not one "tank": it is whichever link on the path is slowest at that moment, and that can change halfway through a download. Third, overflowing water does not always mean the tank is full: on a radio link a packet can be lost to a transmission error, and classic TCP mistakes that for congestion.

## Technical mechanism

### 1. What happens in the system

1. **Start.** After the three-way handshake, the sender sets the initial window (IW). Under the experimental rule in RFC 6928, used by many systems, it can be up to 10 segments, capped at 14,600 bytes.
2. **Slow start.** For each ACK that acknowledges new data, `cwnd` grows by at most one sender maximum segment size (SMSS). If each ACK covers one segment, a full window of ACKs adds a full window of segments: the window doubles every RTT. This phase lasts while `cwnd` is below the threshold `ssthresh`.
3. **Congestion avoidance.** Once `cwnd` exceeds `ssthresh`, growth slows to about one segment per RTT in the classic algorithm. CUBIC uses a different curve (see below).
4. **Congestion signal.** The sender detects trouble in one of three ways: three duplicate ACKs arrive (a hole in the stream), the retransmission timer expires, or the receiver echoes an ECN mark.
5. **Reaction.** On three duplicate ACKs, the classic rule retransmits the lost segment, lowers `ssthresh` to half the data in flight, and continues in congestion avoidance from there. On a timer expiry the reaction is harsher: `cwnd` drops to a single segment and slow start repeats up to the new `ssthresh`.
6. **Start over.** The window grows again until the next signal. If the connection sits idle for longer than one retransmission timeout, the window is reduced to at most the initial window before sending resumes.

### 2. What each component does

**Congestion window (`cwnd`).** An internal sender variable. It limits the data sent and unacknowledged for network reasons. It never travels in any packet: the receiver does not know it.

**Receive window (`rwnd`).** Advertised by the receiver in every segment, it protects the receiver's buffer. RFC 5681 states that the sender may not have more in flight than the minimum of `cwnd` and `rwnd`. One protects the network; the other protects the receiver.

**Slow-start threshold (`ssthresh`).** Marks the boundary between exponential growth and cautious growth. It can start very high. After congestion, RFC 5681 sets it to the larger of half the data in flight and two segments.

**Retransmission timer (RTO).** RFC 6298 computes it from a smoothed RTT (`SRTT`) and its variation (`RTTVAR`): `RTO = SRTT + 4·RTTVAR`. It recommends a minimum of 1 s and doubling the value on each consecutive expiry. That is why an expiry is costly: the sender has been stalled for at least a second.

**Growth algorithm.** Decides how `cwnd` recovers after congestion:

- **Reno (RFC 5681):** adds one segment per RTT and multiplies by 0.5 after a loss. This is called "additive increase, multiplicative decrease".
- **CUBIC (RFC 9438, a standard since 2023):** the window follows the function `W(t) = C·(t − K)³ + W_max`. `W_max` is the window just before the last reduction, `t` is the seconds elapsed since then, and `C = 0.4` segments/s³. `K` is the time the curve takes to return to `W_max`. The curve rises fast at first, flattens near the point where congestion happened and, if nothing goes wrong, accelerates again to look for new capacity. After a loss it multiplies the window by 0.7, not 0.5.
- **BBR (IETF draft, version 3):** does not use loss as its main signal. It estimates two things: the maximum recently delivered bandwidth and the minimum recent RTT. Their product is the bandwidth-delay product (BDP): just enough data to fill the path without building a queue. BBR spaces out packets (pacing) at that rate and keeps data in flight around the BDP.

**ECN (RFC 3168).** Two bits in the IP header let a router mark a packet as "congestion experienced" (CE) instead of dropping it. The receiver echoes the signal with the ECE flag in its ACKs and the sender answers with CWR. The sender's reaction must be essentially the same as to a loss: reduce the window, at most once per window of data. The difference is that no packet has to be thrown away or retransmitted.

### 3. Key parameters

- **Initial window:** 2 to 4 segments under RFC 5681; up to 10 segments (14,600 bytes maximum) with the experimental extension in RFC 6928.
- **Window-to-rate relationship:** average rate ≈ `cwnd / RTT`. With `cwnd` = 64 KB and RTT = 40 ms you get about 13 Mbit/s, whatever the link capacity.
- **Path BDP:** bottleneck capacity × RTT. A 100 Mbit/s link with a 40 ms RTT has a BDP of 500,000 bytes, about 342 segments of 1,460 bytes. With less data in flight the link is underused. With more, the excess piles up in queues.
- **Start-up time (worked calculation):** from 10 segments, doubling each RTT, the window passes 342 segments after 6 RTTs, about 240 ms at a 40 ms RTT. This is the ideal case of one ACK per segment. If the receiver acknowledges every second segment (delayed ACKs), growth per RTT is smaller.
- **Recovery time after a loss (worked calculation):** on a 1 Gbit/s path with a 100 ms RTT, the BDP is around 8,560 segments. Reno halves the window and regrows it by one segment per RTT: about 4,280 RTTs, more than 7 minutes. With CUBIC, `K = ∛(W_max·0.3/C)` ≈ 18.6 s, and that time does not depend on the RTT. This is the problem CUBIC was designed to solve: fast, long paths.
- **CUBIC parameters:** `C = 0.4` and a reduction factor of 0.7 (RFC 9438, Sections 5.1 and 4.6).
- **RTO:** recommended minimum of 1 s, initial value of 1 s before the RTT is measured, and doubling on each expiry (RFC 6298).

### 4. Diagram or flow

```text
                 cwnd < ssthresh                 cwnd ≥ ssthresh
              +-------------------+          +-----------------------+
 connection ->|  SLOW START       | -------> | CONGESTION            |
 (cwnd = IW)  |  +1 SMSS per ACK  |          | AVOIDANCE             |
              |  ≈ ×2 per RTT     |          | Reno: +1 SMSS per RTT |
              +-------------------+          | CUBIC: cubic curve    |
                  ^                          +-----------------------+
                  |                             |               |
                  |              3 duplicate ACKs or ECN mark   |
                  |                             v               |
                  |                ssthresh = inflight/2 (Reno) |
                  |                window ×0.7 (CUBIC)          |
                  |                stay in avoidance            |
                  |                                             |
                  |               retransmission timer (RTO)    |
                  +---------------------------------------------+
                       ssthresh = max(inflight/2, 2·SMSS)
                       cwnd = 1 segment, RTO ×2

 cwnd
  ^        /|        /|        /|
  |       / |       / |       / |      sawtooth (Reno)
  |      /  |      /  |      /  |
  |     /   |/    /   |/    /   |/
  |    /
  |  _/   <- slow start
  +----------------------------------------> time
```

## Limits, exceptions, and real-world cases

**A loss is not always congestion.** Reno and CUBIC treat any loss as a sign the network is full. On radio links the lower layers retransmit many errors, but not all of them. Cardwell and colleagues (2016) measured, on a 100 Mbit/s link with a 100 ms RTT, that CUBIC's throughput fell about 10-fold at 0.1% random loss. BBRv1 kept throughput close to the maximum up to 5% loss. These are BBRv1 figures from a test bed, not results for the version 3 described in the current draft.

**Oversized queues (bufferbloat).** A loss-based algorithm keeps growing the window until something overflows. If the router has a large buffer, it fills the whole buffer first. Packets wait in the queue and the RTT climbs. As a worked calculation, 1 MB queued in front of a 10 Mbit/s uplink adds 0.8 s of waiting to every packet. In Google's tests described in the same article, switching YouTube servers from CUBIC to BBRv1 cut median RTT by 53% on average worldwide.

**The reaction depends on who is sending.** Congestion control lives in the sender. In a download, the server's algorithm is in charge, not the phone's. Two identical downloads can behave differently depending on the server.

**Algorithms sharing a path.** CUBIC includes a "Reno-friendly" region so it does not take capacity from Reno flows on small-BDP paths (RFC 9438, Section 3.2). The BBR draft is still under discussion at the IETF and its intended status is experimental. How it shares capacity with CUBIC flows on the same path is not covered in this article.

**Short connections.** A small web page can finish before leaving slow start. In that case the initial window and the RTT matter more than the link capacity. That was RFC 6928's motivation for raising the initial window to 10 segments.

**ECN needs everyone to cooperate.** The mark only works if the sender, the receiver, and the routers in between support and respect it. If a middlebox clears the bits, the sender falls back to relying on loss.

## Practical consequences

A download does not start at full speed. During the first RTTs the rate climbs in steps, and with long RTTs that ramp is more noticeable. That is why a distant server takes longer to "get up to speed" than a nearby one on the same line.

The speed of a TCP connection is not a fixed property of the line. It depends on the window, and the window depends on the RTT and on losses. On the same fibre, a connection with a 10 ms RTT regains its window far sooner than one at 150 ms.

When a large download saturates the home link, a simultaneous video call can get worse even though "bandwidth is left". The download fills the queues and everything passing through them waits longer. This is the bufferbloat effect described above.

For service designers, choosing the server's algorithm is an engineering decision with measurable consequences. It affects latency, throughput on lossy networks, and how capacity is shared with other flows.

## Frequently asked questions

### Is congestion control the same as flow control?

No. Flow control protects the receiver: it advertises in `rwnd` how much free space its buffer has. Congestion control protects the network: the sender estimates `cwnd` without anyone telling it. The real limit is the smaller of the two.

### Does TCP measure my connection speed?

The classic algorithms do not. Reno and CUBIC do not estimate capacity: they widen the window until something fails. BBR does build an explicit estimate of bandwidth and minimum RTT, and updates it while sending.

### If the ACK confirms the data, why is a timer needed?

Because if every segment in a window is lost, or all their ACKs, nothing arrives to raise the alarm. The timer is the safety net. Since it has a recommended minimum of 1 s and doubles on each expiry, it is much slower than fast retransmit on duplicate ACKs.

### Is slow start actually slow?

No. The name refers to starting with a small window instead of sending everything the receiver allows in one burst. In practice the window doubles every RTT, which is exponential growth.

## Sources

- [RFC 5681 — TCP Congestion Control, IETF (2009)](https://www.rfc-editor.org/rfc/rfc5681.html). Definitions of `cwnd`, `rwnd`, and `ssthresh`; slow start, congestion avoidance, fast retransmit, and the response to timer expiry.
- [RFC 6928 — Increasing TCP's Initial Window, IETF (2013, experimental)](https://www.rfc-editor.org/rfc/rfc6928.html). Initial window of up to 10 segments and 14,600 bytes.
- [RFC 6298 — Computing TCP's Retransmission Timer, IETF (2011)](https://www.rfc-editor.org/rfc/rfc6298.html). RTO computation, 1 s minimum, and doubling on each expiry.
- [RFC 9438 — CUBIC for Fast and Long-Distance Networks, IETF (2023)](https://www.rfc-editor.org/rfc/rfc9438.html). Cubic window function, `C = 0.4`, factor 0.7, and default adoption in Linux, Windows, and Apple stacks.
- [RFC 3168 — The Addition of Explicit Congestion Notification (ECN) to IP, IETF (2001)](https://www.rfc-editor.org/rfc/rfc3168.html). ECN bits in the IP header, the ECE and CWR flags, and the sender's response.
- [draft-ietf-ccwg-bbr — BBR Congestion Control, IETF (draft -07, November 2026)](https://datatracker.ietf.org/doc/draft-ietf-ccwg-bbr/). BBRv3 specification: bandwidth model, minimum RTT, BDP, and pacing.
- [N. Cardwell, Y. Cheng, C. S. Gunn, S. H. Yeganeh, and V. Jacobson. "BBR: Congestion-Based Congestion Control". *ACM Queue*, vol. 14, no. 5 (2016)](https://queue.acm.org/detail.cfm?id=3022184). Motivation for BBRv1 and measurements on Google's network (random loss, YouTube RTT).
