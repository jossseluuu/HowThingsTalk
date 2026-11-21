---
title: "Why does streaming video quality change on its own with your connection?"
section: "04_DataTransfer"
language: "en"
status: "published"
order: 3
depth: "intermediate"
summary: "The video is encoded several times, at different bitrates, and cut into segments a few seconds long. Before requesting each segment, the phone's player picks which version to download based on what it has measured of the network and how full its buffer is."
prerequisites:
  - "How TCP delivers data in order (segments, ACKs, and retransmission)"
  - "How TCP knows how fast it can send (congestion window and RTT)"
  - "Bitrate and throughput"
translation: "../../ES/04_TransferenciaDeDatos/03-PorQueElStreamingCambiaDeCalidad.md"
---

# Why does streaming video quality change on its own with your connection?

## Question and framing

You start a video on your phone and the first few seconds look blurry. Shortly afterwards the picture sharpens. You step into a lift and it blurs again, but the video keeps playing. Nobody touched the quality setting.

The question is who makes that change and with what information. This article explains adaptive bitrate streaming over HTTP (ABR): how the video is prepared on the server, what the manifest describes, and how the player picks the quality of each chunk. It uses HLS (RFC 8216) as the reference and mentions MPEG-DASH, which follows the same principle. It does not cover how the codec compresses each picture, encryption and rights management, or low-latency variants for live streams. Nor does it describe the specific algorithm of any commercial service: almost none of them publish it. The depth is intermediate. It helps to have read [How does TCP deliver data in order when packets arrive out of order?](01-HowTCPDeliversDataInOrder.md) and [How does TCP know how fast it can send without overloading the network?](02-HowTCPDecidesHowFastToSend.md).

## Short answer

The server holds the video in several qualities, cut into segments a few seconds long, and the phone's player decides before each segment which one to request, based on the speed it has measured and how many seconds of video it has stored.

### 60-second summary

Whoever publishes the video encodes it several times. Each version has a different resolution and bitrate: for example, from 416×234 at 145 kbit/s up to 1920×1080 at 7800 kbit/s. Each version is cut into segments of about 6 s. The cuts fall at the same instants in every version, so the player can jump from one to another at any boundary.

An index file, the manifest, lists the versions and the bitrate of each. The player downloads it first. It then requests the segments one by one with ordinary HTTP requests and stores them in a buffer before showing them.

Before each request, the player picks the version. It has two clues. One is how fast the last segments downloaded. The other is how many seconds of video are in the buffer. If the buffer runs dry, the picture freezes: the dreaded "loading" wheel. That is why the player would rather drop quality in time than run out of video.

The standard does not say how that decision is made. Each player uses its own algorithm. That is why two apps on the same network can behave differently.

## Mental model

Picture a water tank feeding a tap that is always open. The tap is playback: it uses exactly one second of video per second of clock time. The tank is the buffer. You refill it with buckets carried from a well whose flow changes without warning. You can choose small buckets of murky water or large buckets of clean water. When the tank is nearly full, you risk the large buckets. When it is dropping, you bring small buckets, which arrive sooner, so the tap does not run dry.

The analogy captures the core idea: the choice depends on the tank level and on how fast the well delivers, and the worst outcome is running out of water. It breaks in two places. First, the "bucket" here cannot change size at will: every segment always lasts the same number of seconds of video, and what changes is how many bytes it weighs. Second, the well has no flow rate of its own that can simply be measured. The speed the player sees depends on how TCP manages its window, and that window is affected by the player's own pauses.

## Technical mechanism

### 1. What happens in the system

1. **Preparation.** The video is encoded into several versions, called variants (HLS) or representations (DASH). The set is known as the bitrate ladder. Each variant is cut into segments of almost fixed duration.
2. **Manifest.** A master playlist listing the variants is published. In HLS, each variant carries its `BANDWIDTH` attribute and a media playlist that enumerates its segments. In DASH a single MPD file plays that role.
3. **Startup.** The player downloads the manifest. It picks an initial variant and requests the first segments. At first the buffer is empty and carries no information, so the first choice is usually cautious or based on the speed of the first downloads.
4. **Download loop.** Each segment is requested with an ordinary HTTP `GET`, carried over TCP or QUIC. When it completes, the player records how many bytes arrived and how long it took. The segment goes into the buffer.
5. **Decision.** Before the next `GET`, the adaptation (ABR) algorithm picks the variant from the estimated speed, the buffer level, or both.
6. **Pause.** If the buffer reaches its maximum size, the player stops downloading and waits for playback to consume video. The download proceeds in bursts: periods of activity separated by silences.
7. **Variant switch.** Because segment boundaries line up across variants, the next segment can come from a different quality. The decoder starts it with a complete picture and the transition is clean.

### 2. What each component does

**Bitrate ladder.** This is the menu of qualities. Apple's HLS specification proposes a nine-rung H.264 example. It goes from 416×234 at 145 kbit/s up to 1920×1080 at 7800 kbit/s, with 1280×720 at 3000 and at 4500 kbit/s in between. The ladder is not universal: each service designs its own.

**Segment.** Apple's specification asks for nominal 6 s segments (rule 7.5) that begin with an IDR frame (rule 7.4). An IDR frame is a picture that decodes without depending on earlier ones. It also recommends that segment boundaries fall at the same instants in every variant (rule 8.22). Without that alignment, switching quality would mean discarding video or waiting.

**Manifest (HLS).** The `EXT-X-TARGETDURATION` tag sets the maximum segment duration: each duration, rounded to the nearest integer, must be less than or equal to that value (RFC 8216, section 4.3.3.1). `EXT-X-STREAM-INF` describes each variant. Its `BANDWIDTH` attribute is the peak segment bitrate, in bits per second. The optional `AVERAGE-BANDWIDTH` is the average (section 4.3.4.2). For video on demand, Apple requires measured values to be within 10% of the declared ones (rules 1.26 and 1.27).

**Player and buffer.** The player is software on the phone: the app or the browser. The buffer holds segments that have been downloaded but not yet shown, measured in seconds of video. Its level follows a simple rule. While downloading a variant of bitrate `R` at speed `C`, the buffer gains `C/R` seconds of video per second of clock time and loses 1 to playback. If `C` is lower than `R`, the buffer drains.

**Adaptation (ABR) algorithm.** RFC 8216 states explicitly that the algorithms for switching between variants are beyond its scope (section 6.3.1). There are three families:

- **Throughput-based.** They estimate capacity from recent downloads and pick the highest variant that fits with a margin. One example estimator is the harmonic mean of the last five downloads, used by the RobustMPC algorithm according to the comparison by Spiteri and colleagues.
- **Buffer-based.** They pick the variant from the buffer level alone. In Netflix's 2013 experiment (the BBA algorithm), below 90 s of buffer the player always requested the lowest quality. Between 90 and 216 s, quality stepped up with the level. Above that, it requested the highest. BOLA formalizes this idea as an optimization problem: it maximizes average quality plus a term that penalizes freezes, with a logarithmic utility of bitrate.
- **Hybrid.** They use measured speed during startup, while the buffer still says nothing, and the buffer level afterwards. This is the case of the DYNAMIC algorithm in the DASH reference player, dash.js.

### 3. Key parameters

- **Segment duration:** a nominal 6 s in Apple's specification, with IDR frames every 2 s (rule 1.13). The Netflix experiment used 4 s segments; the BOLA study, 3 s.
- **Segment size (worked example):** 6 s at 7800 kbit/s is 46,800 kbit, about 5.85 MB. The same segment at 145 kbit/s weighs about 109 kB. The ratio is more than 50 to 1.
- **Buffer dynamics (worked example):** with 20 Mbit/s available, a 6 s segment at 7800 kbit/s downloads in 2.34 s, and the buffer gains a net 3.66 s per segment. If capacity drops to 4 Mbit/s, that same segment takes 11.7 s and the buffer loses 5.7 s per segment. With 30 s stored, the picture would freeze in roughly one minute. Dropping to the 3000 kbit/s variant, each segment takes 4.5 s and the buffer grows again by 1.5 s per segment.
- **Buffer size:** the Netflix player in the 2013 experiment held up to 240 s. The BOLA study notes that dash.js then defaulted to 12 s. A large buffer gives headroom; a small one reacts sooner and wastes less data if the viewer quits.
- **Live:** in a playlist with no end, the client should not start at a segment less than three target durations from the end of the playlist (RFC 8216, section 6.3.3). With 6 s segments, that keeps the viewer at least 18 s behind the latest published segment. The client reloads the playlist no sooner than one target duration later if it changed, or half of one if it did not (section 6.3.4).
- **Data use (worked example):** one hour at 7800 kbit/s is about 3.5 GB. One hour at 145 kbit/s is about 65 MB.

### 4. Diagram or flow

```text
 SERVER (HTTP)                                     PHONE (player)
 -------------                                     --------------
 master playlist ----------- GET ---------------->  reads variants and BANDWIDTH
   variant 1080p 7800 kbit/s: seg1 seg2 seg3 ...
   variant  720p 3000 kbit/s: seg1 seg2 seg3 ...   (aligned boundaries)
   variant  234p  145 kbit/s: seg1 seg2 seg3 ...
                                                    +---------------------+
                     +----------------------------> | ABR ALGORITHM       |
                     |                              | inputs:             |
 chosen segment <----+--- GET segN (variant X) ---- |  measured speed     |
        |                                           |  buffer level       |
        +---- bytes over TCP/QUIC ----------------> +---------------------+
                                                              |
                                                              v
                                    BUFFER [seg seg seg seg ....] --> screen
                                    gains C/R s per second            uses
                                    when full: pauses download        1 s/s

 buffer level
  ^   maximum ......____......____......    (download in bursts)
  |            ___/    \____/    \___
  |       ___/                         \    capacity drops: buffer drains
  |   ___/                              \__ -> ABR picks a lower variant
  |  /  <- startup                         \_/ -> buffer recovers
  +-----------------------------------------------> time
```

## Limits, exceptions, and real-world cases

**Measured speed is a noisy estimate.** In the Netflix study, the average per-segment speed of a single session swung between 17 Mbit/s and 500 kbit/s. In a sample of 300,000 sessions, about 10% had a median speed below half of their 95th percentile. The causes cited include WiFi interference, network congestion, overloaded servers, and processes on the device itself. The authors estimated that 20% to 30% of the freezes they observed might be due to a poor quality choice rather than a real lack of capacity.

**The player's pauses confuse TCP.** When the buffer is full, the player stops requesting segments. If the connection sits idle for longer than the retransmission timeout, RFC 5681 says the sender should cut its window to no more than the initial window before sending again. The next segment then starts with a small window and downloads more slowly than the line would allow. An algorithm that only looks at speed can read this as a drop in capacity.

**Startup is the hardest moment.** With an empty buffer there is no information about the network. Netflix found that its first purely buffer-based version spent about 90 s of video at the lowest quality when starting. That cost it about 100 kbit/s of average quality at peak hours compared with the previous algorithm. Later versions used measured speed to ramp up sooner. That is why many videos start blurry and improve within a few seconds.

**Segments do not all weigh the same.** Encoding is variable bitrate: a scene with lots of motion takes more bits than a still shot. `BANDWIDTH` declares the peak and `AVERAGE-BANDWIDTH` the average. Apple recommends that for video on demand the peak should not exceed 200% of the average (rule 1.30). An algorithm that only uses the nominal average can fall short in a demanding scene.

**Results depend on the test bed.** The Netflix study is a real experiment with over half a million users per test, in 2013. The authors warn that the production algorithm has changed since. The BOLA figures come from simulations with network traces: between 84% and 95% of the maximum utility achievable with advance knowledge of the network. They are not measurements of a production service.

**Abandoning mid-download.** If capacity collapses in the middle of a large download, waiting for it to finish can drain the buffer. Some players cancel the segment in progress and request it again at a lower quality. The practical version of BOLA includes this option.

## Practical consequences

A speed test's figure is not what the player sees. The test measures a long, continuous download. The player measures short downloads, separated by pauses, with a TCP window that sometimes starts again from the bottom.

Dropping quality is the defence against freezing, not a fault. A blurrier picture for a few seconds is usually less annoying than a pause. That is why algorithms drop early and climb with caution.

Manually picking a fixed quality turns adaptation off. If the network cannot sustain it, the buffer drains and the picture freezes. The automatic option exists precisely to avoid that.

Top quality costs far more data than the lowest: about 3.5 GB per hour against about 65 MB with Apple's example ladder. On a capped mobile plan, the difference matters.

A live stream over the internet runs behind the broadcast. HLS rules require keeping several segments of margin, and with 6 s segments that means tens of seconds. Low-latency extensions exist, which this article does not cover.

For whoever designs a service, the ladder, the segment duration, the buffer size, and the algorithm are trade-offs against each other. Short segments allow faster reactions, but each one starts with an IDR frame that takes more bits. A large buffer prevents freezes, but on live streams it adds delay.

## Frequently asked questions

### Does the server decide what quality it sends me?

In HLS and DASH, no. The server only publishes files and answers HTTP requests. The phone's player chooses which segment to request. That is why the same server can serve a phone at 234p and a TV at 1080p at the same time.

### Is this the same as TCP congestion control?

No, although both adapt to the network. TCP decides how fast to send the bytes of a given download. The ABR algorithm decides which file to download. They work at different layers and influence each other: TCP's window shapes the speed the player measures.

### Is this the same as codec compression?

No. The codec decides how to represent each picture with few bits and is applied when each variant is prepared. Adaptive streaming chooses among variants that are already compressed. A more efficient codec allows the same quality at a lower bitrate, but it adapts nothing on its own.

### Why does it drop fast and climb slowly?

Because the two mistakes do not cost the same. Climbing too soon can drain the buffer and freeze the picture. Climbing late only means a few seconds of lower quality. Buffer-based algorithms reflect that asymmetry: they only climb once a margin has built up.

## Sources

- [RFC 8216 — HTTP Live Streaming, R. Pantos and W. May, IETF, Informational (2017)](https://www.rfc-editor.org/rfc/rfc8216.html). Master and media playlists, variants, `EXT-X-TARGETDURATION`, `BANDWIDTH` and `AVERAGE-BANDWIDTH`, switching algorithms out of scope (6.3.1), three-target-durations rule (6.3.3), and playlist reloading (6.3.4).
- [HLS Authoring Specification for Apple Devices, Apple](https://developer.apple.com/streaming/hls-authoring-specification-for-apple-devices.html). Example H.264 ladder (rule 1.25), IDR every 2 s (1.13), `BANDWIDTH` accuracy (1.26 to 1.30), 6 s segments starting with an IDR (7.4 to 7.7), and alignment across variants (8.15 and 8.22).
- [ISO/IEC 23009-1 — Dynamic adaptive streaming over HTTP (DASH), Part 1: Media presentation description and segment formats, ISO/IEC](https://www.iso.org/standard/23009-1). The MPEG-DASH standard, with the MPD manifest and its representations.
- [T.-Y. Huang, R. Johari, N. McKeown, M. Trunnell, and M. Watson. "A Buffer-Based Approach to Rate Adaptation: Evidence from a Large Video Streaming Service". *ACM SIGCOMM* (2014)](https://web.stanford.edu/class/cs244/papers/sigcomm2014-video.pdf). Variability of measured speed, the BBA algorithm, 90 s reservoir, 240 s buffer, 4 s segments, and fewer freezes at Netflix.
- [K. Spiteri, R. Urgaonkar, and R. K. Sitaraman. "BOLA: Near-Optimal Bitrate Adaptation for Online Videos". *IEEE/ACM Transactions on Networking* (2020), arXiv version](https://www.arxiv.org/pdf/1601.06748). Lyapunov optimization formulation, buffer-only decision, implementation in dash.js, the DYNAMIC algorithm, and comparison with other algorithms.
- [RFC 5681 — TCP Congestion Control, IETF (2009)](https://www.rfc-editor.org/rfc/rfc5681.html). Restarting the congestion window after an idle period (section 4.1).
