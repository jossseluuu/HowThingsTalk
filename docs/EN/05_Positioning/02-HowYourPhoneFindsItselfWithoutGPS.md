---
title: "How does your phone know where it is with GPS off and without joining any WiFi network?"
section: "05_Positioning"
language: "en"
status: "published"
order: 2
depth: "intermediate"
summary: "WiFi access points and cell towers constantly announce their identifiers, and anyone can read them without connecting. The phone notes which ones it hears and how strongly, then looks them up in a database that records where each one is. That database is built by other phones that did have GPS when they passed by."
prerequisites:
  - "Frequency, wavelength and decibels (article 00·01)"
  - "Multipath (article 00·03)"
  - "How GPS computes a position (article 05·01)"
translation: "../../ES/05_Geolocalizacion/02-ComoTeLocalizaElMovilSinGPS.md"
---

# How does your phone know where it is with GPS off and without joining any WiFi network?

## Question and framing

Inside a shopping centre or a station, the phone's map puts the blue dot in the right building. Sometimes GPS cannot see a single satellite. And the phone is not connected to the venue's WiFi. The question is where that position comes from.

This article explains WiFi and cellular positioning: which signals the phone reads, which database it queries, how that database is built and how a list of identifiers becomes a pair of coordinates. It does not cover how the network helps GPS start faster, Bluetooth or UWB positioning, or positioning methods computed by the mobile operator itself. Nor does it explain how to query these databases about other people's devices: that boundary is set in the [style guide](../../../guides/StyleGuide.md).

It helps to know decibels ([article 00·01](../00_Foundations/01-WhyHigherFrequenciesReachLess.md)), multipath ([article 00·03](../00_Foundations/03-WhySignalChangesOverAFewCentimetres.md)) and the GPS principle ([article 05·01](01-HowGPSComputesYourPosition.md)). The depth is intermediate, and there is no maths beyond a weighted average.

## Short answer

The phone does not measure its position: it recognises which access points and cell towers are around it and looks up where each one has been seen.

### 60-second summary

A WiFi access point sends an announcement frame, the beacon, several times per second. It carries the network name and a unique identifier for the device, the BSSID. It is not encrypted, because its whole purpose is to let others find the network. Any phone can read it without connecting. Cellular works in a similar way: the phone always knows which cell is serving it, because it needs that identifier to work at all.

With that list of identifiers and the strength of each one, the phone asks a location service. The service holds a huge database with the approximate position of hundreds of millions of access points and cells. Other phones built it: whenever they have GPS, they report in the background which networks they hear and where they are.

With the positions of the nearby access points, an average weighted by signal strength gives the phone's position. According to Google, with two or more access points the typical error is around 20 m. With only the cell, it is hundreds of metres.

## Mental model

Imagine being dropped blindfolded in a city you know well. You don't know where you are. But you hear one church bell very loudly, another further away, and the beeping of a particular pedestrian crossing. You know where each of them is. If the first bell sounds very loud, you are close to it. Without measuring anything, you can place yourself within a few streets.

Now suppose you did not draw the map of where each bell is. Thousands of walkers who could see drew it, noting which sounds they heard on each corner.

The analogy breaks in three places. Bells don't move, but an access point can move house, or be a portable router riding a train. Radio strength is not a reliable measure of distance: a wall or a body removes more decibels than many metres of air. And the map is not in your head: in many cases it has to be requested over the internet from a server, which therefore receives the list of what you can hear.

## Technical mechanism

### 1. What happens in the system

There are two phases, separated in time.

**Collection phase (building the database).** Millions of phones with a known position, usually from GNSS, record which access points and cells they detect, how strongly, and where they were. They send this to their operating-system vendor. Apple states that the iPhone periodically sends "geo-tagged locations" of nearby hotspots and cell towers, in anonymous and encrypted form. Rye and Levin (2024) describe the same scheme for Apple and Google. Once enough devices have seen a BSSID for long enough, the service estimates a position for it and adds it to the database.

**Query phase (locating you).**

1. **Listen.** The phone runs a WiFi scan: it steps through the channels and collects beacons, or sends a probe request and collects the responses. It records each access point's BSSID and signal strength.
2. **Cell.** It adds the identifier of its serving cell and, if available, those of neighbouring cells.
3. **Query.** It sends the list to the location service.
4. **Compute.** Depending on the vendor, the server returns a computed position, or the positions of the access points and the phone does the maths.
5. **Fuse.** The operating system combines the result with GPS (if any), motion sensors and the previous position.

### 2. What each component does

**The WiFi beacon.** It is defined by the IEEE 802.11 standard. It is a management frame that the access point sends periodically. It carries the BSSID in its header, the network name (SSID), channels and capabilities. It is sent unencrypted even on WPA3 networks, because encryption is negotiated later, when a device connects. The beacon interval is configurable. The usual value in equipment is 100 time units of 1024 µs, about 102.4 ms (a common default, not mandated by the standard; unverified against the standard's text).

**The BSSID.** A 48-bit identifier, normally the MAC address of the access point's radio. A dual-band router usually has two BSSIDs. For the positioning system, the BSSID is the lookup key. The network name is useless for this, because thousands of networks share the same name.

**The phone's own address.** An active scan reveals that a phone is present, but it does not have to reveal which one. Current systems use random addresses when searching for networks ([article 11·01](../11_PrivacyAndFootprint/01-WhyPhonesRandomizeTheirMACAddress.md)). To locate itself, the phone needs everyone else's identifiers, not its own.

**The cell identifier.** Every cell broadcasts its identity in the system information it transmits. The phone needs it to register on the network. It consists of the country code (MCC), the operator code (MNC), the area code (LAC or TAC) and the cell identity. In LTE that last part is 28 bits and in 5G NR it is 36 bits, according to Google's API documentation. Reading it requires no data connection.

**Timing advance.** The base station tells each phone to transmit earlier depending on how far away it is, so that all signals arrive aligned. In practice, that value measures the round-trip delay. It turns "I am in this cell" into "I am on a ring at a certain distance from the tower". Google's API accepts it as an optional input.

**The database.** For each BSSID or cell it stores an estimated position. Rye and Levin retrieved about 490 million BSSIDs with positions from Apple's database in their 2024 study. A new access point does not appear immediately: several devices need to see it over a period of time.

**The query service.** Two designs are documented.

- **Google** receives the list of BSSIDs with their strengths and returns a position and an accuracy radius. It requires at least two "physically distinct stationary" access points. With fewer, it answers that nothing was found.
- **Apple**, according to Rye and Levin, receives BSSIDs and returns the position of each one plus up to 400 more nearby BSSIDs. The final computation happens on the device.

**The position computation.** The simplest method is the centroid: the average of the positions of the access points heard. The weighted centroid gives more weight to the strongest ones. Finer methods exist, such as matching the strength pattern against surveyed maps (fingerprinting) or particle filters. In the study by Cheng and colleagues (2005), in dense urban areas the simple centroid was almost as accurate as the complex methods.

### 3. Key parameters

**Expected accuracy.** Google's documentation gives these typical radii for its service:

| Available data | Typical accuracy radius |
|---|---|
| Two or more WiFi access points | about 20 m |
| Small cells | 10 to 30 m |
| Macro cells | hundreds of metres; rarely below 100 m |
| IP address only | thousands of metres |

The documentation defines accuracy as the radius of a circle around the position, but does not state the confidence level of that circle.

**Access point density.** In 2005, Cheng and colleagues measured three areas around Seattle with 802.11b cards, using GPS as ground truth:

| Area | Access points per km² | Access points per scan (mean) | Median error (simple centroid) |
|---|---|---|---|
| Downtown | 1030 | 2.66 | 24.4 m |
| Medium-density residential | 1000 | 2.56 | 14.8 m |
| Sparse residential | 130 | 1.41 | 37.0 m |

The overall range was 13 to 40 m of median error. These are 2005 figures: access point density is higher today and phones hear more networks per scan, but there is no current equivalent measured with the same method. The trend holds: the more access points are heard, the lower the error.

**Received strength.** Android returns each scan result with the BSSID, the channel frequency in MHz and the level in dBm. Google's API expects values between −128 and −10 dBm. Strength is used for weighting, not for measuring exact distances. As [article 00·03](../00_Foundations/03-WhySignalChangesOverAFewCentimetres.md) explains, moving a few centimetres can change it by tens of decibels.

**Scan rate.** The system limits how often apps can scan. On Android 9 and later, each foreground app can request four scans every 2 minutes. All background apps combined get one every 30 minutes. Reading the results also requires location permission and location turned on: the system treats the list of nearby networks as location data.

### 4. Diagram or flow

```text
 COLLECTION PHASE (other phones, earlier)
 Phone with GPS ─► "at (lat, lon) I hear BSSID A −55 dBm, B −72 dBm, cell X"
                ─► anonymous, encrypted upload to the vendor
                ─► database: A ≈ (lat_A, lon_A), B ≈ (lat_B, lon_B), X ≈ ...

 QUERY PHASE (your phone, now, no GPS and no WiFi connection)
   WiFi beacons (unencrypted, every ~100 ms)      Serving cell
   BSSID A  −55 dBm                               MCC-MNC-TAC-cell ID
   BSSID B  −72 dBm                               (+ timing advance)
   BSSID C  −80 dBm
          │                                              │
          └──────────────┬───────────────────────────────┘
                         ▼
             Query to the service (over mobile data)
                         │
          ┌──────────────┴──────────────┐
          ▼                             ▼
   Google model:                  Apple model:
   server computes                server returns positions of A, B, C
   and returns position           and of up to 400 nearby BSSIDs;
   + accuracy radius              the device computes
          │                             │
          └──────────────┬──────────────┘
                         ▼
     Weighted centroid: more weight to A (the strongest)
                         ▼
     Fusion with sensors and the last known position
                         ▼
              Blue dot (typical error ~20 m with WiFi)
```

## Limits, exceptions, and real-world cases

**Access points that move.** A router that moves house keeps its BSSID. For a while, the database will still place it at the old address. If the phone hears only that one, it can appear in another city. Google says its service ignores moving access points, such as those on planes or trains, and requires at least two distinct ones. Queries with several access points let the service discard the one that does not fit the rest.

**Random BSSIDs.** Some access points change their BSSID. Rye and Levin note that the personal hotspot mode of iOS 17 and Android 14 uses random BSSIDs. Those access points are useless as landmarks, and that is the point: their trail cannot be followed.

**The "_nomap" opt-out.** Google and Apple let the owner of a network exclude it from their databases. The network name just has to end in "_nomap". Apple warns that this does not work for hidden networks, because phones cannot see the name. Google says each location provider decides independently whether to honour the tag.

**No data connection.** In the server-query model, with no mobile data and no connected WiFi there is no answer. All that is left is whatever the device stored from earlier queries. How each system manages that stored information is not publicly documented (unverified).

**Indoors and floors.** The centroid gives a position on a flat map. Telling which floor of a building you are on is much harder with WiFi, because an access point can be heard through the floor slabs. In the study by Cheng and colleagues, mean indoor error ranged from 9 to 98 m depending on the location.

**Areas without networks.** On a motorway or in the countryside there may be no access points at all. In the sparse area of the 2005 study, more than half of the scans heard none. Then only the cell is left, with errors of hundreds of metres or more.

**Differences between devices.** Each phone's antenna and receiver measure strength slightly differently. The same access point can arrive a few decibels apart on two phones side by side. Fingerprinting methods are more sensitive to this than the centroid.

## Practical consequences

Turning GPS off does not turn off the phone's positioning. As long as the WiFi radio can scan and there is a cell, the system can place itself within a few tens of metres in a city. That is why operating systems treat WiFi scan results as location data protected by permissions.

WiFi positioning is fast and works inside buildings, exactly where GPS fails. GPS is accurate under open sky, exactly where there are usually few networks. That is why phones combine the two: each covers the other's gap.

In the server-query model, the vendor receives the list of networks around you. Apple and Google state that this data is handled without identifying the person. Privacy depends on that policy, not on the mechanism, which by design reveals the surroundings.

For the owner of an access point, its BSSID ends up in large databases even if no outsider ever connects. It is a direct consequence of beacons having to be readable by everyone. The "_nomap" tag and random BSSIDs are the two documented ways to reduce this.

## Frequently asked questions

### Is this the same as cell tower triangulation?

No. With the cell, the phone knows which tower it is connected to, not its exact distance from three towers. The typical result is an area hundreds of metres across. WiFi is more precise because access points have shorter range and are far more numerous.

### Do I have to join the WiFi network for this to work?

No. Beacons are sent unencrypted so that anyone can discover the network. The phone only listens. Connecting adds nothing to the position.

### Does being connected to my home WiFi help locate me?

Only as one more access point. What matters is how many access points the phone hears and whether the database knows them, not which one it is connected to.

### Is IP geolocation the same thing?

No. An IP address identifies, at best, the internet provider's area. Google gives accuracies of thousands of metres for IP. WiFi identifies specific physical devices with a range of a few tens of metres.

## Sources

- [Google — Geolocation API: Geolocation requests](https://developers.google.com/maps/documentation/geolocation/requests-geolocation). Query fields (BSSID, strength in dBm between −128 and −10, cells with MCC, MNC, LAC/TAC, 28-bit identity in LTE and 36-bit in NR, timing advance), minimum of two stationary access points, and accuracy defined as a radius in metres.
- [Google — Geolocation API overview](https://developers.google.com/maps/documentation/geolocation/overview). Typical accuracy radii: about 20 m with two or more access points, 10–30 m with small cells, hundreds of metres with macro cells and thousands of metres with IP.
- [Apple — Location Services & Privacy](https://www.apple.com/legal/privacy/data/en/location-services/). Use of GPS, Bluetooth and crowd-sourced hotspot and cell tower locations; periodic, anonymous and encrypted upload of geo-tagged locations of nearby networks.
- [Apple — About privacy and Location Services in iOS, iPadOS, and watchOS](https://support.apple.com/en-us/102515). Opting an access point out with the "_nomap" suffix, and its limitation with hidden networks.
- [Google — Control access point inclusion in Google's Location services](https://support.google.com/maps/answer/1725632). Opt-out with the "_nomap" suffix; each provider decides whether to honour it.
- [Android Developers — Wi-Fi scanning overview](https://developer.android.com/develop/connectivity/wifi/wifi-scan). Scan limits (4 per 2 min in the foreground, 1 per 30 min for all background apps combined) and mandatory location permission.
- [Android Developers — ScanResult](https://developer.android.com/reference/android/net/wifi/ScanResult). Contents of a scan result: BSSID, SSID, level in dBm and channel frequency.
- [E. Rye and D. Levin, "Surveilling the Masses with Wi-Fi-Based Positioning Systems", arXiv:2405.14975 (2024)](https://arxiv.org/abs/2405.14975). Crowd-sourced construction of the databases, differences between Apple's and Google's services, up to 400 nearby BSSIDs returned by Apple, on-device computation, and random BSSIDs in iOS 17 and Android 14.
- [Y.-C. Cheng, Y. Chawathe, A. LaMarca and J. Krumm, "Accuracy Characterization for Metropolitan-scale Wi-Fi Localization", MobiSys 2005](https://usenix.org/legacy/events/mobisys05/tech/full_papers/cheng/cheng_html/index.html). Median error of 13 to 40 m, access point densities, comparison of centroid, fingerprinting and particle filters, and indoor error.
- [IEEE 802.11 Working Group — IEEE Std 802.11, Wireless LAN Medium Access Control (MAC) and Physical Layer (PHY) Specifications](https://www.ieee802.org/11/). Definition of beacon and probe frames, the BSSID and the 1024 µs time unit.
