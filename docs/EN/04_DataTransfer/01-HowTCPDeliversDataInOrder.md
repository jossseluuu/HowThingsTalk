---
title: "How does TCP deliver data in order when packets arrive out of order?"
section: "04_DataTransfer"
language: "en"
status: "published"
order: 1
depth: "intermediate"
summary: "TCP turns application data into a numbered byte stream. It uses acknowledgments, buffers, and retransmissions to deliver that stream in order even when segments arrive late, duplicated, or out of order."
prerequisites:
  - "Network packets and layers"
  - "IP addresses and ports"
  - "Latency and throughput"
translation: "../../ES/04_TransferenciaDeDatos/01-ComoEntregaTCPLosDatosEnOrden.md"
---

# How does TCP deliver data in order when packets arrive out of order?

## Question and framing

When an application sends data over the Internet, the network does not guarantee that every packet follows exactly the same route or arrives in the same order. Some packets may be delayed, lost, or duplicated. Yet an application using TCP receives a reliable, ordered byte stream.

This article explains how TCP achieves that ordered delivery. Its scope is the TCP transport service between two endpoints. It does not explain how an application frames a file into messages, how IP routes datagrams, or how an application verifies that a saved file matches the original. Readers should understand the distinction between an IP address, a port, and a packet. The target depth is intermediate.

## Short answer

TCP numbers every byte in the stream, temporarily buffers data that arrives out of order, and requests or performs retransmission of missing bytes before delivering the stream to the application.

### 60-second summary

TCP does not treat each segment as a separate application message. It provides a continuous byte stream. Every byte occupies a position in a sequence number space, and segments carry ranges from that sequence.

The receiver acknowledges cumulatively the next byte it expects. If later bytes arrive beyond a gap, it can store them in a buffer, but it does not advance the cumulative acknowledgment or deliver a stream with that gap to the application. The sender interprets acknowledgments and, when it detects loss or its retransmission timer expires, sends the outstanding data again. Once the missing range arrives, the receiver can join it with the data already buffered and advance delivery.

## Mental model

Imagine a book whose pages are shipped separately. Each page has a number, and the reader puts them on a shelf until they can be read in order. If pages 1, 2, and 4 arrive, the reader keeps page 4 but cannot read the complete book by skipping page 3. When page 3 arrives, reading can continue.

The analogy helps explain temporary storage and ordering, but it breaks at an important point: TCP does not number messages or pages. It numbers individual bytes in a stream. A segment may contain part of an application message, several messages, or a number of bytes unrelated to the application's write calls.

## Technical mechanism

### 1. What happens in the system

1. The application gives bytes to TCP. TCP treats them as a continuous stream and does not necessarily preserve the boundaries of individual write calls.
2. TCP divides the stream into segments that fit the link conditions and negotiated limits. Each segment identifies the sequence number of its first byte.
3. IP carries each segment inside a datagram. The network may lose, duplicate, or deliver datagrams in a different order.
4. The receiving TCP checks the segment and uses its sequence numbers to identify the positions of the bytes it contains. Valid bytes received beyond a gap can be stored temporarily.
5. The receiver sends acknowledgments. The cumulative acknowledgment identifies the next sequence number it expects; it therefore confirms that every earlier byte has arrived.
6. The sender keeps data that has not yet been acknowledged. If it detects signs of loss or its retransmission timer expires, it retransmits the necessary data.
7. Once the gap is filled, the receiver can advance its receive position and deliver the available consecutive bytes to the application.

### 2. What each component does

**Sequence number.** Every data byte occupies a position in TCP's 32-bit sequence number space. A segment's sequence number identifies its first byte. The number space wraps at its limit, so comparisons use modular arithmetic.

**Cumulative acknowledgment (ACK).** The Acknowledgment Number field indicates the next byte the receiver expects. If it sends ACK = 5001, it confirms that every byte before 5001 has arrived. This does not mean that a packet numbered 5001 arrived: the numbers refer to positions in the byte stream.

**Receive buffer.** It temporarily holds received data that cannot yet form a continuous sequence starting at the next expected byte. A later segment therefore does not have to be discarded merely because an earlier one has not arrived. The available space is advertised through the receive window.

**Retransmission queue.** The sender keeps transmitted data that has not yet been acknowledged. When a cumulative acknowledgment advances, the acknowledged portion can be removed from that queue. Data that remains outstanding may need retransmission.

**Timer and loss detection.** TCP estimates when an acknowledgment should arrive and uses a retransmission timer. It can also infer loss from duplicate acknowledgments. The classic fast retransmit mechanism described in RFC 5681 uses three duplicate ACKs as an indication to retransmit without waiting for the timer. This is not a description of every modern implementation or every loss-recovery algorithm.

**SACK (Selective Acknowledgment).** If both endpoints negotiate it, the receiver can report specific blocks of data already received beyond a gap. The sender gets more precise information and can retransmit only the missing ranges. SACK supplements cumulative acknowledgment; it does not change the meaning of the main ACK field.

### 3. Key parameters

- **Sequence number:** a 32-bit value identifying byte positions, not packet numbers or application messages (RFC 9293, Section 3.4).
- **MSS (Maximum Segment Size):** limits the amount of TCP data carried in a segment, excluding TCP and IP headers. Its effective value depends on the path MTU, options, and negotiation between endpoints.
- **Receive window (`rwnd`):** indicates how much receive-buffer space the remote endpoint is willing to accept. It prevents the sender from overrunning the receiver's buffer.
- **Congestion window (`cwnd`):** limits how much data the sender may keep in flight for congestion-control reasons. It is not the same as the receive window. The amount that can be sent without further acknowledgment is constrained by both.
- **RTT and retransmission timer:** RTT is the round-trip time for segments and their acknowledgments. TCP adjusts its timer using RTT estimates and their variation; there is no single fixed timeout that applies to every connection.
- **SACK:** a negotiable extension. It should not be assumed to be available on every TCP connection.

### 4. Diagram or flow

```text
Sending application
       |
       v
Byte stream -> TCP numbers and segments it
       |
       v
IP datagrams -> network (may be lost or reordered)
       |
       v
Receiving TCP -> validates sequence numbers
       |
       +--> Bytes beyond a gap: temporary buffer
       |
       +--> Cumulative ACK: next expected byte
       |           |
       |           v
       |     Sending TCP: updates outstanding data
       |           |
       |           +--> loss detected/timeout -> retransmission
       |
       v
Gap filled -> continuous stream -> receiving application
```

## Limits, exceptions, and real-world cases

**TCP orders bytes, not messages.** If an application performs two writes, the receiver is not guaranteed to get two reads with the same boundaries. The application needs its own framing if it must distinguish messages, for example through lengths or delimiters defined by its protocol.

**Out-of-order arrival does not by itself prove loss.** A segment may simply be delayed or may have followed a different route. Duplicate acknowledgments are clues, not infallible proof. Recovery algorithms therefore combine multiple signals and timers.

**SACK does not replace cumulative ACK.** It describes blocks received out of order, while the main ACK still points to the first missing byte. Its use depends on whether it was permitted during connection establishment.

**Recovery has a cost.** Data beyond a gap occupies memory and cannot be delivered to the application as though the stream were complete. A loss can add latency even when the remaining segments have already arrived. If the connection ultimately fails, TCP cannot guarantee that the application receives the rest of the stream.

**TCP does not guarantee the semantic integrity of a file.** Its checksum detects certain segment errors, but it is not a substitute for a cryptographic file hash and does not prove that the saved content is the expected one. That check belongs to higher layers or to the application itself.

**Not all traffic uses TCP.** UDP and other transports have different models. The ordered and reliable delivery provided by TCP should not be attributed to every data transfer.

## Practical consequences

A TCP download can appear continuous even when retransmissions, duplicate segments, or out-of-order arrivals occur underneath. The application receives ordered bytes and does not need to manage each IP packet.

The cost of that abstraction is that a loss can temporarily block delivery of later bytes. For a file transfer, waiting and reconstructing the stream is usually desirable. In interactive applications that are sensitive to delay, that wait may be more disruptive than losing a small amount of information; such applications may therefore choose transports or protocols with different guarantees.

This also explains why the size of a `send()` call does not predict the size of the segments observed on the network or the size of the `read()` calls made by the receiver. TCP preserves byte order, not application-operation boundaries.

## Frequently asked questions

### Does TCP reorder IP packets?

It does not change the order in which the network delivered them. It uses sequence numbers to place bytes at their proper positions and presents only the available consecutive stream to the application.

### What happens if the same segment arrives twice?

Sequence numbers let the receiver recognize repeated bytes. It must not deliver a second copy of those bytes to the application stream, and it may acknowledge the position it is still expecting.

### Does an ACK mean the application has saved the data to disk?

No. It confirms receipt by the remote endpoint's TCP stack, not that the application has processed the data, written it to disk, or validated it.

### Does TCP always retransmit exactly the same packet?

Not necessarily. TCP retransmits outstanding data, but the segment carrying it may be built again and does not have to reproduce the exact encapsulation of the earlier transmission.

## Sources

- [RFC 9293 — Transmission Control Protocol (TCP), IETF (2022)](https://www.rfc-editor.org/rfc/rfc9293.html). Base specification: byte stream, sequence numbers, acknowledgments, segmentation, and windows.
- [RFC 2018 — TCP Selective Acknowledgment Options, IETF (1996)](https://www.rfc-editor.org/rfc/rfc2018.html). Defines SACK negotiation and selective acknowledgment blocks.
- [RFC 5681 — TCP Congestion Control, IETF (2009)](https://www.rfc-editor.org/rfc/rfc5681.html). Describes congestion control and the classic fast retransmit mechanism based on duplicate ACKs.
