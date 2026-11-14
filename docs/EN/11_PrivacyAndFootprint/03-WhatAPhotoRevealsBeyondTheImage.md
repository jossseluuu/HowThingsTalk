---
title: "What does a phone photo tell about you, beyond what it shows?"
section: "11_PrivacyAndFootprint"
language: "en"
status: "published"
order: 3
depth: "intermediate"
summary: "Every phone photo carries an Exif data block that nobody sees when looking at it. It usually holds the exact date and time with the time zone offset, the make and model of the device and, if the camera has location permission, the coordinates of the spot with street-level precision. That block travels with the file wherever it is sent, unless someone strips it along the way."
prerequisites:
  - "How a GPS receiver computes its position (article 05·01)"
  - "How a phone finds itself without GPS (article 05·02)"
translation: "../../ES/11_PrivacidadYHuella/03-QueCuentaUnaFotoAdemasDeLaImagen.md"
---

# What does a phone photo tell about you, beyond what it shows?

## Question and framing

A photo looks like just an image. But the file a phone saves holds more than pixels. The question is specific: what information travels hidden inside a photo file, where is it stored, and who can read it when you share the photo?

This article explains the Exif format: how it sits inside a JPEG or HEIC file, how the date and the coordinates are encoded, and how precise they are. It also covers what iOS and Android do to avoid handing out location by accident. It is not a step-by-step guide to cleaning photos and it does not compare apps. It does not cover recognising places from the image content either, which is a different problem.

The depth is intermediate. It helps to know how a phone gets its position, with GPS ([article 05·01](../05_Positioning/01-HowGPSComputesYourPosition.md)) or without it ([article 05·02](../05_Positioning/02-HowYourPhoneFindsItselfWithoutGPS.md)). No programming knowledge is needed: the bytes shown are explained one by one.

## Short answer

A phone photo carries an attached data sheet that anyone holding the file can read: when it was taken, with which device and, if the camera has location permission, where, at a resolution of about 30 cm.

### 60-second summary

When you press the shutter, the camera does not store just the image. It adds an **Exif** block at the start of the file. Exif is a standard from the Japanese industry bodies CIPA and JEITA, used by almost every camera and phone.

That block is a list of fields, each with a number and a value. The most common ones are:

1. **When.** Date and time of the shot, to the second, plus the offset from universal time.
2. **With what.** Make, model, software version and sometimes lens data.
3. **How.** Exposure, aperture, sensitivity, flash and orientation.
4. **Where.** Latitude, longitude, altitude and even which way the phone was pointing. Only if the camera has location permission.
5. **A thumbnail.** A small copy of the image.

None of this shows when you look at the photo. But it travels inside the file: if you send the original file, the recipient can read it. Some services strip it on upload and others do not. iOS lets you share without location, and Android hides the coordinates from apps that lack a specific permission.

## Mental model

Think of a printed photo with a sticker on the back. The camera wrote the sticker: date, time, model and the address where you were standing. Someone looking at the front of the photo does not see it. Someone who receives the photo and turns it over does.

Sharing the original file is like handing over the photo with the sticker still on. Some middlemen peel it off before passing it on. Others leave it.

The analogy breaks in two places. First, a paper sticker is visible to anyone who looks; the Exif block can only be read by software that understands the format, although that includes any desktop photo viewer. Second, a paper sticker holds only what was written once. The file can carry copies of the same data in several places, such as the thumbnail or blocks private to the manufacturer. Deleting one field does not guarantee you deleted them all.

## Technical mechanism

### 1. What happens in the system

When you press the shutter, this is the sequence:

1. **The sensor captures and the processor compresses.** The result is a JPEG or HEIC image.
2. **The camera gathers the data of the moment.** It reads the system clock and time zone, the exposure settings and the device model. If the camera app has location permission, it also asks the system for the latest position. According to Apple, that position may come from GPS, WiFi networks, the cellular network or Bluetooth.
3. **The camera writes the Exif block.** It arranges the data as a small set of tables inherited from the TIFF image format.
4. **The block goes at the start of the file.** In JPEG it sits right after the start-of-image marker. In HEIC it goes inside the format's metadata box structure.
5. **The file is saved to the gallery.** From then on, every exact copy of the file carries the block with it.

### 2. What each component does

**The JPEG container.** A JPEG file is a sequence of segments, each preceded by a two-byte marker. The JPEG standard (ITU-T T.81) reserves markers `FFE0` to `FFEF` for application data, and each segment carries a 16-bit length field. Exif uses the **APP1** segment (`FFE1`), which must come immediately after the start of image (`FFD8`). Because the length fits in 16 bits, the block cannot exceed 64 KB, and the Exif standard says so explicitly (CIPA DC-008, section 4.7.2).

**The header.** Inside APP1, the first six bytes are `Exif` followed by two zero bytes. Then comes an 8-byte TIFF header. It states whether numbers are stored most significant byte first (`MM`) or the other way round (`II`), and where the first table starts.

**The tables (IFDs).** Each table, called an IFD, is a list of 12-byte entries. Each entry gives: field number (2 bytes), data type (2 bytes), how many values it holds (4 bytes) and the value or, if it does not fit, where to find it (4 bytes). The standard defines several linked tables:

- **IFD 0:** basic data about the main image, such as make (field 271), model (272) and software (305).
- **Exif IFD:** pointed to by field 34665. It holds the capture date (`DateTimeOriginal`, 36867), its time offset (`OffsetTimeOriginal`, 36881), exposure and other data. The optional fields for body serial number (42033), lens model (42036), camera owner name (42032) and unique image ID (42016) also live here.
- **GPS IFD:** pointed to by field 34853. It holds latitude, longitude, altitude, GPS time, image direction and estimated horizontal error.
- **IFD 1:** the thumbnail, a small but complete JPEG image inside the block.

**The maker note.** The `MakerNote` field (37500) is a block of bytes whose content each manufacturer decides. The standard does not describe it. It can hold internal camera data that no standard viewer displays.

**The location permission.** The camera only writes the GPS IFD if the system gives it a position. On iPhone, Apple states that location is stored only if the Camera app has access to Location Services.

### 3. Key parameters

- **Coordinate format.** Latitude (GPS IFD field 2) and longitude (field 4) are stored as three rational numbers: degrees, minutes and seconds. Each rational is two 4-byte integers, numerator and denominator, so each coordinate takes 24 bytes. Two separate fields (1 and 3) say north or south and east or west.
- **Resolution of the value.** One second of latitude is about 30.9 m on the ground. If the seconds denominator is 100, the value is stored in steps of 0.01″, which is about 0.31 m. For longitude the step shrinks with the cosine of the latitude: at 40° latitude, one second is about 23.7 m. The denominator is chosen by whoever writes the file; 100 is a common example, not a rule of the standard.
- **Real accuracy.** The resolution of the number is not the accuracy of the measurement. GPS.gov states that high-quality single-frequency GPS receivers achieve 2.168 m or better horizontally 95% of the time, based on FAA data. It also warns that real accuracy depends on the atmosphere, sky blockage and receiver quality. Indoors, with a WiFi or cellular position, the error is usually larger. Exif has a field to declare it, `GPSHPositioningError` (field 31), but it is optional.
- **Date and time.** `DateTimeOriginal` is text in the form `YYYY:MM:DD HH:MM:SS`, to the second. `OffsetTimeOriginal` adds the offset from universal time, for example `+01:00`. Together they give the exact instant and, as a side effect, the time zone the device was in.
- **Size.** The whole APP1 block fits in 64 KB, thumbnail included. Extra data that does not fit can go in additional APP2 segments (CIPA DC-008, section 4.7.3).
- **Standard version.** Exif 3.0 (CIPA DC-008-2023) was published in 2023. It adds a UTF-8 text data type for writing in any language, new fields such as photographer and editor, and guidance on which fields to update or keep when an image is edited.

### 4. Diagram or flow

These are the first real bytes of a JPEG with Exif generated for this article, with sample data:

```text
FF D8                  Start of image (SOI)
FF E1  00 D6           APP1 segment, 214 bytes long
45 78 69 66 00 00      "Exif" + 00 00
4D 4D  00 2A           TIFF header: "MM" byte order and magic number 42
00 00 00 08            IFD 0 starts 8 bytes after the header
```

And this is how the tables link together inside the block:

```text
APP1 "Exif"
 └─ TIFF header
     └─ IFD 0 ── Make = "Brand"   Model = "Model"   Software = "..."
         ├─ field 34665 ──► Exif IFD
         │                   DateTimeOriginal   = "2026:11:14 10:30:00"
         │                   OffsetTimeOriginal = "+01:00"
         │                   MakerNote          = [manufacturer bytes]
         ├─ field 34853 ──► GPS IFD
         │                   GPSLatitudeRef  = "N"
         │                   GPSLatitude     = 40/1, 25/1, 1234/100  → 40° 25′ 12.34″
         │                   GPSLongitudeRef = "W"
         │                   GPSLongitude    = 3/1, 42/1, 5678/100   → 3° 42′ 56.78″
         └─ next ────────► IFD 1: JPEG thumbnail
FF DB ... FF DA ...        From here on, the compressed image
```

The sample coordinates, `40/1, 25/1, 1234/100`, equal 40.420094° north latitude.

## Limits, exceptions, and real-world cases

**HEIC stores the same thing in a different wrapper.** iPhones can capture in HEIF since iOS 11, according to the US Library of Congress. HEIF (ISO/IEC 23008-12) has no APP1 segments: it organises the file into typed boxes and carries metadata, Exif included, inside its metadata structure. The Exif content is the same; only its location changes.

**The location in the file is not the one in the gallery.** Apple states that you can hide or change a photo's location in the Photos app. But copies you have already sent are independent files: changing the photo in your gallery does not change those copies.

**Coordinates are not proof.** Exif is text and numbers that any program can rewrite. The standard includes no signature or tamper protection. A photo with coordinates shows where the device that wrote it said it was, not where it certainly was.

**Removing location does not always remove everything.** The date with its time offset, the device model and the maker note stay if only the GPS IFD is removed. The thumbnail may keep an earlier version of the image if the program that cropped the photo did not regenerate it. Exif 3.0 adds guidance on which fields to update when editing precisely because not every program did so.

**Each service does something different.** Some services strip metadata when a photo is uploaded, and others keep the file as it is, especially when it is sent "as a document" or "as a file". No standard requires either behaviour, and each service may change over time (unverified service by service).

**Android protects at the app level, not the file level.** Android's documentation states that, with scoped storage, the system hides photo location from apps by default. An app targeting Android 10 (API level 29) or higher that wants unredacted metadata must declare the `ACCESS_MEDIA_LOCATION` permission and request it from the user at runtime. It must also ask for the original bytes with `MediaStore.setRequireOriginal()`. That protects against apps installed on the phone itself, but it does not change the file: if it is copied over a cable or sent as is, the coordinates are still inside.

## Practical consequences

For people sharing photos, the key point is that the file says more than the image. A photo taken at home with location on carries the home address down to the front door. A series of photos with date, time and coordinates traces a routine. Apple puts it plainly in its personal safety guide: people who receive photos with location may access that data and learn where they were taken.

Controls exist at three levels. At the camera, you can deny location permission and new photos will not carry it. When sharing, iOS lets you turn off location under "Options" before sending, and shows that location is not included above the button. In apps, Android requires a separate permission to read photo location.

For people designing apps or services, the key point is to decide explicitly what happens to metadata when an image is received. Keeping all of it spreads data the user may not have known they were sending. Removing all of it loses useful information, such as orientation or date. Exif 3.0 offers guidance for editing with judgement instead of copying or deleting blindly.

## Frequently asked questions

### If I turn off location on the phone, does the photo say nothing?

It says less, but not nothing. Without the GPS IFD, the date and time to the second, the time offset, the make, the model, the software version and the camera settings remain. The image itself can also give away the place, but that no longer depends on Exif.

### Is a photo's location the same as the phone's location history?

No. The history is kept by the system or a cloud service, separate from the photos. Exif location sits inside each file and travels with it. You can have one without the other.

### When a social network strips metadata, does it also delete it from its servers?

There is no way to tell from outside. The fact that the file others see has no Exif says nothing about what the service stored when it received the original. That depends on its policy, not on the Exif standard.

### Why are there coordinates even though I was in a basement with no GPS?

Because the camera does not require GPS. It uses whatever position the system provides, which may come from WiFi or the cellular network, as [article 05·02](../05_Positioning/02-HowYourPhoneFindsItselfWithoutGPS.md) explains. It may also be a recent position saved before you went in. That is why a photo's coordinates can be off by tens of metres.

## Sources

- [CIPA DC-008-2016 / JEITA CP-3451D — Exchangeable image file format for digital still cameras: Exif Version 2.31 (CIPA and JEITA, July 2016)](https://people.cs.umass.edu/~liberato/courses/2017-spring-compsci365/files/DC-008-Translation-2016-E.pdf). Copy hosted by the University of Massachusetts. APP1 segment after SOI, `Exif` identifier, 64 KB limit and TIFF header (section 4.7.2), APP2 segments (4.7.3) and field numbers for IFD 0, Exif, GPS and IFD 1 (tables 17 to 21).
- [CIPA DC-008-Translation-2023 — Exif Version 3.0 (CIPA, 2023), official download page](https://www.cipa.jp/std/documents/download_e.html?DC-008-Translation-2023-E). Current version of the standard.
- [IPTC — Exif 3.0 released, featuring UTF-8 support (1 June 2023)](https://iptc.org/news/exif-3-0-released-featuring-utf-8-support/). Summary of Exif 3.0 changes: UTF-8 type, new fields and guidance on updating fields when editing (Annex H).
- [ITU-T T.81 / ISO/IEC 10918-1 — Digital compression and coding of continuous-tone still images (ITU, 1992)](https://www.w3.org/Graphics/JPEG/itu-t81.pdf). JPEG marker structure, APPn markers `FFE0`–`FFEF` and 16-bit length field.
- [Library of Congress — High Efficiency Image File Format (HEIF)](https://www.loc.gov/preservation/digital/formats/fdd/fdd000525.shtml). HEIF as a box-based format built on ISO/IEC 23008-12, metadata carriage and iPhone capture since iOS 11.
- [Apple — Personal Safety User Guide: Manage location metadata in Photos](https://support.apple.com/guide/personal-safety/manage-location-metadata-in-photos-ips0d7a5df82/web). Where the location comes from, sharing without location, hiding it in Photos and denying the Camera permission.
- [Android Developers — Access media files from shared storage](https://developer.android.com/training/data-storage/shared/media). Location hidden under scoped storage, the `ACCESS_MEDIA_LOCATION` permission and `MediaStore.setRequireOriginal()`.
- [GPS.gov — GPS Accuracy](https://www.gps.gov/systems/gps/performance/accuracy/). 2.168 m or better horizontally 95% of the time with high-quality single-frequency receivers, and factors that degrade accuracy.
