---
title: "Why does a smart bulb go through a distant server to turn on from the same room?"
section: "09_IOT"
language: "en"
status: "published"
order: 1
depth: "intermediate"
summary: "A home router does not let in connections that nothing inside asked for. So the bulb opens a permanent connection to its manufacturer's server itself and keeps it alive. The app does not talk to the bulb: it talks to that server, which forwards the command down the connection that is already open. This is a design choice, not a physical necessity: local paths exist, such as the one Matter standardizes."
prerequisites:
  - "What a TCP connection is and how it delivers data in order (article 04·01)"
  - "What an IP address and a local network are (the essentials are explained in the article)"
translation: "../../ES/09_IOT/01-PorQueLaBombillaInteligentePasaPorLaNube.md"
---

# Why does a smart bulb go through a distant server to turn on from the same room?

## Question and framing

You are in the living room. Your phone and the bulb are on the same WiFi router, a few metres apart. You tap the button in the app and the light comes on. But the command did not go from the phone to the bulb. It left the house, reached the manufacturer's data centre, perhaps hundreds or thousands of kilometres away, and came back. The question is why something that looks so simple is designed this way.

This article covers the most common case: a WiFi bulb controlled through its manufacturer's app. It also compares that design with the local control that Matter defines.

- **Depth:** intermediate. There are simple delay calculations and a message diagram.
- **Prerequisites:** knowing what a TCP connection is helps. It is explained in [How does TCP deliver data in order when packets arrive out of order?](../04_DataTransfer/01-HowTCPDeliversDataInOrder.md). What you need about IP addresses and routers is explained here.
- **Out of scope:** how to configure a router or a bulb, comparisons between brands, how TLS encryption works inside, and bulbs that use only Bluetooth.

## Short answer

Nobody outside the home can open a connection to the bulb, so the bulb opens and keeps a connection to the manufacturer's server, and the app always uses that path, even when you are next to the bulb, because it is the only one that works the same everywhere.

### 60-second summary

Inside the home, each device has a private address. The router shares a single public address among all of them. To do that, it records which device inside has talked to which server outside. When a reply arrives, it knows whom to deliver it to. When something arrives that nobody inside asked for, it has no entry telling it where that should go.

That is why the bulb, when it powers up, connects to its manufacturer's server on its own. That connection leaves the house, so the router lets it through and records the way back. The bulb keeps it open by sending a tiny message now and then.

When you tap the app, the phone sends the command to the server. The server forwards it to the bulb over the connection the bulb opened. It works the same from the sofa as from another country. The server also stores the state, the routines, and the user's account.

The price is depending on the internet and on the company. Local alternatives exist, but they have to solve at home what the server solves for free: finding the bulb and proving who is allowed to send it commands.

## Mental model

Think of an office building with a very strict front desk. The receptionist lets any outgoing call through. If someone outside calls back, she puts them through to whoever made the call. But if a stranger calls asking for "the lamp person", she hangs up: she doesn't know who that is or whether anyone is expecting the call.

The bulb works in one of those offices. When it arrives in the morning, it calls its company's head office and leaves the line open. You, even from the office next door, don't go and see it: you call head office, and head office passes the message down that line.

The receptionist is the router with NAT. Head office is the manufacturer's server. The open line is the persistent connection.

**Where the analogy breaks.** An open phone line occupies something the whole time. The bulb's connection does not: between messages nothing travels, apart from an occasional small packet so the router doesn't forget it. Also, in the building there is no way to walk to the office next door. On a local network there is: the phone and the bulb can talk to each other directly. That they don't is a design decision, not a physical impossibility.

## Technical mechanism

### 1. What happens in the system

**Setup.** The bulb joins the home WiFi network. The router gives it a private IP address. During setup, the app links it to the user's account on the manufacturer's server.

**The outgoing connection.** The bulb opens an encrypted connection to a messaging server run by the manufacturer. A widely used protocol for this is MQTT. Its standard notes that TCP port 8883 is registered with IANA for MQTT over TLS and 1883 for unencrypted MQTT (MQTT 5.0, section 4.2). One documented platform, AWS IoT Core, accepts MQTT over TLS on port 8883, and also on 443 with a TLS extension called ALPN (AWS, *Protocols*). Port 443 is the same port HTTPS uses.

**The subscription.** In MQTT, the client is the one that opens the network connection to the server (MQTT 5.0, section 1.2). Over that connection, the bulb subscribes to a topic, a name that identifies its messages. From then on, the server uses a PUBLISH packet to deliver each message to every client with a matching subscription (MQTT 5.0, section 3.3.4). It transmits it on that session's connection when the connection is active (MQTT 5.0, section 4.8.1). So messages flow from the server to the bulb along a path the bulb opened.

**The command.** When you tap the app:

1. The phone sends the command to the manufacturer's API, over the internet.
2. The server records the new desired state: "on".
3. The server publishes a message on the bulb's topic.
4. The message travels down the connection the bulb keeps open.
5. The bulb turns on its LED and publishes its new state.
6. The server updates the state and the app shows it.

AWS documents this pattern with a document called the device *shadow* (AWS, *Device Shadow*). The shadow holds two states: the desired one, written by the app, and the reported one, written by the device. The difference between them is what the server sends to the device. If the device was offline, it receives the current state when it reconnects. Not every manufacturer uses AWS, but the idea is common: the bulb's state lives in the cloud, and the bulb follows it.

### 2. What each component does

**The router with NAT.** A home has many private addresses and usually one public one. The router translates between them (NAT). When a device inside sends a packet out, the router creates a mapping: "this public port belongs to this device and this private port". IETF standards describe how those mappings should behave (RFC 4787 for UDP and RFC 5382 for TCP).

The key part is filtering. With the most permissive filtering, any packet arriving at a public port that is already mapped is delivered to the device inside. With the stricter ones, it only gets through if it comes from an address, or an address and port, that the device has already sent something to (RFC 4787, section 5). In every case, the mapping is created by an outgoing packet. A packet that arrives with no prior mapping has no known destination inside the home.

**The router's timers.** Mappings expire if they are not used.

- For UDP, a mapping must not expire in less than 2 minutes, and a default of 5 minutes or more is recommended (RFC 4787, REQ-5). Outgoing traffic must refresh it (REQ-6).
- For TCP, an established connection must not be removed after less than 2 hours and 4 minutes of silence (RFC 5382, REQ-5).

Reality does not always follow the standard. The IAB cites a 2010 test of 34 home routers: more than half did not follow the recommended UDP timeouts (RFC 7452, section 4).

**The keep-alive message.** That is why the bulb sends something periodically even when it has nothing to say. In MQTT, the client sets an interval, the *Keep Alive*. It is a 16-bit number in seconds: at most 65,535 s, about 18 hours. If it has nothing else to send, the client must send a PINGREQ. If the server receives nothing within 1.5 times that interval, it must close the connection (MQTT 5.0, section 3.1.2.10). The server can impose a shorter interval in its reply (section 3.2.2.3.14). The value each bulb actually uses is the manufacturer's choice: **unverified** here.

**The manufacturer's server.** It does much more than forward. It stores the link between account and bulb. It decides who may send the bulb commands. It runs routines and schedules. It connects to voice assistants. The IAB calls this the "device-to-cloud" pattern (RFC 7452, section 2.2).

**The app.** It only needs to know how to talk to the manufacturer's API. It doesn't need to know where the bulb is or whether it is on the same network.

### 3. Key parameters

**The cost of distance.** The ITU gives a planning value of 5 µs per kilometre for an optical fibre system, marked as provisional (ITU-T G.114, table A.1). With a server 2,000 km away:

```text
phone → server:   2000 km × 5 µs/km = 10 ms
server → bulb:    2000 km × 5 µs/km = 10 ms
full command (propagation only):    ≈ 20 ms
confirmation back to the app:      +≈ 20 ms
```

These are minimums. The real fibre route is longer than the straight-line distance. On top of that come the home WiFi, router queues, and processing on the server. The typical total delay of a commercial bulb depends on the case and is **unverified** here. A local command, from phone to bulb through the same router, covers a few metres: propagation is negligible.

**What travels.** A switch-on command is a message of a few bytes. Keep-alive traffic is small packets every so often. What costs is not bandwidth but distance and hops.

**What a local path would need.** To talk to the bulb directly, the app would have to solve two problems the server solves for free:

- **Finding it.** The bulb's private address can change. One standard method is multicast DNS (mDNS): queries go to address 224.0.0.251, or FF02::FB in IPv6, on UDP port 5353 (RFC 6762, sections 2 and 3). Names ending in `.local` are only meaningful on the link where they originate (RFC 6762, section 3). It works inside the home, not from outside.
- **Trusting it, and being trusted by it.** Both sides need shared credentials without going through the server. Otherwise anyone on the WiFi could switch the light on.

### 4. Diagram or flow

```text
 WHEN THE BULB POWERS UP (once)
 Bulb (192.168.1.23) ── TCP + TLS to broker:8883 ──► NAT router ──► Internet ──► MQTT broker
 Router: creates the mapping  public port 40001 ↔ 192.168.1.23:51515
 Bulb ── SUBSCRIBE "home/bulb-23/command" ──────────────────────────────────────► Broker

 KEEP-ALIVE (every Keep Alive interval)
 Bulb ── PINGREQ ──► Broker ── PINGRESP ──► Bulb      (refreshes the router's mapping)

 COMMAND FROM THE SOFA (phone on the same network)
 Phone ── HTTPS "turn on" ──► Router ──► Internet ──► Manufacturer API
 API: desired state = on ──► Broker
 Broker ── PUBLISH "home/bulb-23/command" ──► Internet ──► Router (port 40001)
 Router: mapping exists → deliver to 192.168.1.23:51515
 Bulb: turns on ── PUBLISH state = on ──► Broker ──► API ──► app

 WHAT DOES NOT WORK (no prior mapping)
 Any server ── new connection to the public IP ──► Router: who is this for? → not delivered
```

## Limits, exceptions, and real-world cases

- **No internet, no app.** If the home connection goes down, the command never reaches the server and the bulb never gets it, even with the phone a metre away. Some manufacturers also offer a local mode. Which ones, and how each works, is outside this article.
- **If the company shuts the service down.** The IAB flags this as a risk of the pattern: a change of business model, or bankruptcy, can leave the hardware unusable (RFC 7452, section 2.2). As mitigation it mentions releasing the source code or allowing other software on the device.
- **Zigbee or Thread bulbs.** They do not speak IP to the WiFi router directly: they need a bridge. The IAB calls this the "device-to-gateway" pattern (RFC 7452, section 2.3). The gateway itself may depend on the cloud or work locally. In Thread, that bridge is a border router, which routes IPv6 between the Thread network and the home network (Google, *Thread and IPv6*).
- **Matter standardizes the local path.** Matter's control and other operational communication run over IPv6 (Google, *What is Matter?*). Devices that share a security domain form a *fabric*, with a common root certificate (Google, *Fabric*). When a device is added, the controller gives it credentials for that fabric. The two then open a session authenticated with certificates, called CASE (Google, *Commissioning*). The controller finds already-configured devices with DNS-SD, browsing for the `_matter._tcp` service with `.local` names (Google, *Discovery*). A device can belong to several fabrics at once: this is called multi-admin (Google, *Fabric*). That solves both problems of the local path: finding the device and trusting it. Whether each Matter product also keeps using the cloud for remote access is up to the manufacturer.
- **IPv6 does not fix it on its own.** With IPv6 each device can have a public address and NAT is not needed. But home routers usually filter unsolicited incoming connections. This article has not verified that against a standard: **unverified**.
- **The wall switch.** If you cut the power with a classic switch, the bulb goes off and is no longer connected. When power returns, it has to reconnect to the WiFi and to the server before it obeys commands from the app. What it does when power comes back is up to each manufacturer.

## Practical consequences

- **The delay is small but variable.** Propagation adds at least a few tens of milliseconds with a distant server. Queues and processing can add more, and that changes from one time to the next.
- **The light depends on things outside the home.** The internet connection, the manufacturer's server, and the user's account all have to work at once.
- **The manufacturer sees every command.** Every switch-on goes through its server, which also stores the bulb's state. What it keeps, and for how long, depends on its policy, not on the protocol.
- **For the device designer, the cloud is the easy path.** An outgoing connection works behind almost any router without the user configuring anything. The same path works inside and outside the home. And adding new features only requires changing the server.
- **Local control demands more work at home.** Discovering the device and distributing credentials without a server is exactly what standards such as Matter have to define.

## Frequently asked questions

### Does it go through the cloud because the bulb is too simple for anything else?

No. The bulb already runs a full IP stack and a TLS connection to talk to the server. It could talk to the phone locally. It goes through the cloud because that path works behind the router without configuration and is the same inside and outside the home.

### Is this the same as opening a port on the router?

No. Opening a port asks the router to deliver to a specific device whatever arrives at a public port, even if nobody inside asked for it. The bulb's connection doesn't need that: the bulb opens it outwards. Configuring the router is outside this article.

### Does Matter get rid of the cloud?

It defines a standard local path: discovery on the home network, shared credentials in a fabric, and authenticated sessions. With that, a home controller can send commands without going through an external server. Whether a given product also uses the cloud for remote access or voice assistants is each manufacturer's decision.

### What about a Bluetooth bulb?

With Bluetooth, the phone talks directly to the bulb while it is nearby. There is no router or server in between, but there is also no control from outside the home without a bridge. It is a different design and is outside this article.

## Sources

- IETF / IAB. Tschofenig, Arkko, Thaler, and McPherson. *RFC 7452: Architectural Considerations in Smart Object Networking* (March 2015). Sections 2.2, 2.3, and 4. <https://www.rfc-editor.org/rfc/rfc7452>
- IETF. Audet and Jennings. *RFC 4787: Network Address Translation (NAT) Behavioral Requirements for Unicast UDP* (January 2007). Sections 4.3 and 5. <https://www.rfc-editor.org/rfc/rfc4787>
- IETF. Guha et al. *RFC 5382: NAT Behavioral Requirements for TCP* (October 2008). Section 5. <https://www.rfc-editor.org/rfc/rfc5382>
- IETF. Cheshire and Krochmal. *RFC 6762: Multicast DNS* (February 2013). Sections 2 and 3. <https://www.rfc-editor.org/rfc/rfc6762>
- OASIS. *MQTT Version 5.0*, OASIS Standard (7 March 2019). Sections 1.2, 3.1.2.10, 3.2.2.3.14, 3.3.4, 4.2, and 4.8.1. <https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html>
- Amazon Web Services. *AWS IoT Core Developer Guide*: "Device communication protocols" and "AWS IoT Device Shadow service". <https://docs.aws.amazon.com/iot/latest/developerguide/protocols.html> · <https://docs.aws.amazon.com/iot/latest/developerguide/iot-device-shadows.html>
- Google Home Developers. *Matter Primer*: "What is Matter?", "Fabric", "Commissioning", "Commissionable and operational discovery", and "Thread and IPv6". <https://developers.home.google.com/matter/primer> · <https://developers.home.google.com/matter/primer/fabric> · <https://developers.home.google.com/matter/primer/commissioning> · <https://developers.home.google.com/matter/primer/commissionable-and-operational-discovery> · <https://developers.home.google.com/matter/primer/thread-and-ipv6>
- ITU-T. *Recommendation G.114: One-way transmission time* (05/2003). Table A.1. <https://www.itu.int/rec/T-REC-G.114>
