---
title: "What happens inside flash memory cells when you delete a photo?"
section: "06_Storage"
language: "en"
status: "published"
order: 1
depth: "intermediate"
summary: "When you delete a photo, the flash cells do not change at that moment. The file system marks the space as free and tells the memory with TRIM, UNMAP or Deallocate. The controller stops mapping those pages, but the electrons stay put until garbage collection erases the whole block with high voltage. On a modern phone whatever remains is encrypted, and the real guarantee of erasure comes from destroying keys, not cells."
prerequisites:
  - "What a bit is and what a file is"
  - "Basic idea of encryption with a key (article 10·01 helps)"
translation: "../../ES/06_Almacenamiento/01-QuePasaEnLaFlashCuandoBorrasUnaFoto.md"
---

# What happens inside flash memory cells when you delete a photo?

## Question and framing

You tap "Delete" on a photo and it vanishes from the gallery. It feels as if the phone has destroyed it. This article asks a concrete, physical question: what happens at that moment, and in the minutes or hours that follow, to the flash memory cells that held those bytes?

The path runs from the tap on the screen down to the electrons. It goes through the file system, the notice the operating system sends to the memory, and the internal controller that manages the flash. It ends with the physical erase of a block and the role of encryption. This is not a guide to recovering files or wiping a drive, and it does not compare models. The "Recently deleted" bin in many gallery apps is an app feature: this article starts from the moment the file is actually deleted.

The depth is intermediate. Knowing what a bit and a file are is enough. For the final part it helps to know what an encryption key is ([article 10·01](../10_SecurityAndCryptography/01-HowTwoDevicesAgreeOnASecretKey.md)).

## Short answer

Deleting a photo touches no cell: it deletes the reference to the file, and the electrons that encode the photo stay trapped until the controller decides to erase the whole block they sit in, possibly much later.

### 60-second summary

Flash memory stores each bit as electric charge trapped in a cell. Two rules explain almost everything:

1. **It writes by pages and erases by blocks.** A page is roughly 8 to 16 kB. A block groups hundreds of pages and can only be erased as a whole.
2. **It does not overwrite in place.** To change a page, the new version is written to another free page. The old one is marked invalid.

So when you delete a photo, this happens:

1. The file system marks the photo's logical sectors as free. It writes nothing over them.
2. The operating system may tell the memory that those sectors are no longer in use. That notice is called TRIM in ATA, UNMAP in SCSI and UFS, and Deallocate in NVMe.
3. The memory controller removes those pages from its translation table and marks them invalid. If anyone reads those sectors, the controller may return zeros.
4. Later, garbage collection picks a block with many invalid pages. It copies the valid pages elsewhere and applies a high voltage to the whole block. Only then do the electrons leave.

Between step 1 and step 4 the data is still physically on the chip. On a current phone it is encrypted, and that changes what "still there" means.

## Mental model

Think of a notebook written in ink, with an index at the front. You cannot erase a line. You can only tear out a whole signature of many sheets. To "correct" a page, you copy it, corrected, onto a blank sheet at the back and update the index. To "throw away" a chapter, you strike its entry from the index.

The sheets are still written. Every so often someone goes through the notebook, copies the sheets the index still uses into a fresh signature and tears the old one out.

The analogy breaks in three places. First, you keep the notebook's index yourself, whereas flash has two indexes: the file system's, run by the operating system, and the controller's, which the operating system cannot see. Second, "tearing out the signature" destroys no paper: it returns the block to a reusable state by applying voltage. And third, on a modern phone the sheets are written in cipher. Whatever survives in the notebook cannot be read without the matching key.

## Technical mechanism

### 1. What happens in the system

The full sequence has five moments, and only the last one touches the cells.

1. **The app asks to delete the file.** The gallery calls the operating system to remove the photo's file.
2. **The file system frees the space.** It removes the directory entry and marks the logical blocks the photo used as free. Those logical blocks are sector numbers (LBAs), not physical positions on the chip. Wei and colleagues measured in 2011 that after a normal file-system delete, between 4.3% and 91.3% of the files' contents could be recovered from the chip on SSDs (FAST 2011, Table 3).
3. **The operating system notifies the memory.** It sends a discard command for those LBAs: TRIM, UNMAP or Deallocate, depending on the interface. It may send it immediately or batch it for later. The NVMe specification defines it as **advisory**: the controller "may" deallocate the ranges, and a compliant controller may do nothing at all (NVM Command Set 1.1, section 3.3.3).
4. **The controller updates its map.** The controller keeps a table that translates each LBA to a physical page. On a discard, it breaks that link and marks the physical pages invalid. In NVMe, a later read of a deallocated block returns all 00h, all FFh, or one of the two unspecified, depending on what the DLFEAT field declares (section 3.3.3.2.1).
5. **Garbage collection erases the block.** When free blocks run short, or during idle periods, the controller picks a block with few valid pages. It copies those valid pages to another block and erases the old block as a whole (Cai et al., 2017, section 2.3). That, and not earlier, is when the photo's cells change.

### 2. What each component does

**The cell.** It is a transistor with an insulated region that traps electrons: a floating gate or, in many current chips, a charge-trap layer. The trapped charge shifts the transistor's threshold voltage, the voltage at which it starts to conduct. Reading means checking which threshold-voltage window the cell falls into. An SLC cell distinguishes 2 states (1 bit), an MLC cell 4 (2 bits) and a TLC cell 8 (3 bits) (Cai et al., section 3.1). In MLC cells of the 15 to 19 nm generation, a cell holds only around 100 electrons (section 4.4).

**Programming.** Programming pushes electrons into the insulated region by tunnelling, with a high positive voltage on the control gate. It is done in pulses that raise the threshold voltage step by step and verify after each pulse. This method can only **raise** the threshold voltage, never lower it (Cai et al., section 3.4). That is why a written page cannot be rewritten with a different value.

**Erasing.** Erasing pulls the electrons out into the substrate. The control gate is grounded and the substrate is raised to a high voltage. Every cell in a block shares that substrate, so the erase hits the whole block at once (section 3.4). The source reviewed gives no figure for that voltage, so none is given here.

**Page and block.** A page is the unit of reading and writing. It is 8 to 16 kB. In MLC, a block has 256 to 1024 pages (Cai et al., sections 2.1 and 3.2). Multiplying, a block is roughly 2 to 16 MB. That arithmetic is this article's, not the source's. A phone photo of a few megabytes therefore takes up hundreds of pages spread over one or more blocks that also hold data from other files.

**The controller and its FTL.** The flash translation layer (FTL) is the firmware that makes flash look like an ordinary disk. It translates LBAs to physical pages, always writes to fresh pages, marks old ones invalid, collects garbage and spreads wear across blocks (Cai et al., section 2.3). The operating system sees none of this.

**The hidden space.** The memory has more physical flash than it advertises. In the SSDs Wei and colleagues studied, between 6% and 25% more (section 2.2). That margin gives garbage collection room to work. It is also space the operating system cannot read or overwrite directly.

**Encryption.** On current phones, file contents reach the flash already encrypted. On Android, every device launching with Android 10 or later must use file-based encryption, and the normal configuration encrypts contents with AES-256 in XTS mode (AOSP documentation). On iOS, each file is protected with its own key, stored wrapped by a class key in the file's metadata (Apple Platform Security).

### 3. Key parameters

| Parameter | Value | Scenario and source |
|---|---|---|
| Write unit | 8 to 16 kB page | contemporary NAND flash (Cai et al., 2.1) |
| Erase unit | block of 256 to 1024 pages | MLC (Cai et al., 3.2) |
| States per cell | 2 / 4 / 8 | SLC / MLC / TLC (Cai et al., 3.1) |
| Electrons per cell | ~100 | 15 to 19 nm MLC (Cai et al., 4.4) |
| Program/erase cycles | ~3000 (MLC) · ~1000 (TLC) | per block, 15 to 19 nm generation (Cai et al., 1 and 4) |
| Extra hidden flash | 6% to 25% | SSDs studied in 2011 (Wei et al., 2.2) |
| Stale copies of one file | up to 16 | one SSD with 1000 small files (Wei et al., Figure 1) |
| Read after Deallocate | 00h, FFh or unspecified | NVMe DLFEAT field (NVM Command Set 1.1, 3.3.3.2.1) |
| Android contents encryption | AES-256-XTS | AOSP default configuration |

Wear explains why the controller does not erase every block as soon as it can. Each erase uses up one of a few thousand cycles. Erasing a block to remove a single page would force a copy of the rest of the block and spend a cycle (Wei et al., section 4.2). The controller prefers to wait until a block has piled up many invalid pages.

### 4. Diagram or flow

```text
You          Gallery        File system              Controller (FTL)            NAND cells
 │ "Delete"      │                   │                       │                         │
 │──────────────>│ delete file       │                       │                         │
 │               │──────────────────>│ entry removed         │                         │
 │               │                   │ LBA 8000-8999 free    │                         │
 │               │                   │ TRIM/UNMAP/Deallocate │                         │
 │               │                   │──────────────────────>│ map: LBA -> (none)      │
 │               │                   │                       │ pages = invalid         │  charge intact
 │               │                   │                       │                         │
 │               │          ... minutes, hours or days ...   │                         │
 │               │                   │                       │ garbage collection:     │
 │               │                   │                       │ copy valid pages ──────>│ new block
 │               │                   │                       │ erase old block ───────>│ electrons out
 │               │                   │                       │ block to free list      │
```

One block before and after garbage collection:

```text
Block 412 (before)                    Block 97 (free)          Block 412 (after)
┌────────┬────────┬────────┐          ┌────────┬────────┐      ┌────────┬────────┬────────┐
│ photo ✗│ photo ✗│ chat ✓ │  copy    │ chat ✓ │ app ✓  │      │ erased │ erased │ erased │
├────────┼────────┼────────┤  ─────>  ├────────┼────────┤      ├────────┼────────┼────────┤
│ photo ✗│ app ✓  │ photo ✗│          │ free   │ free   │      │ erased │ erased │ erased │
└────────┴────────┴────────┘          └────────┴────────┘      └────────┴────────┴────────┘
✗ = invalid page (still charged)       ✓ = valid page           erased = no electrons
```

## Limits, exceptions, and real-world cases

**The discard notice may not arrive, or may not be honoured.** If the file system sends no discard, the controller believes those pages are still in use. It will keep them and copy them in every garbage collection. If the discard is sent, the controller may ignore it, because the command is advisory. In 2011, Wei and colleagues concluded that TRIM had no reliable effect on data security (footnote 2). Controllers have changed since then. No equivalent recent study of phones was reviewed here, so the current behaviour of any given model is **unverified**.

**Overwriting the file does not work.** On a magnetic disk, writing over a file destroys the original. On flash, the write goes to fresh pages and the old one stays. In the same study, overwriting a single file with various methods left between 4% and 75% of its contents recoverable on SSDs (section 3.3). NIST SP 800-88 Rev. 2 (2025) states it in general terms: on flash memory with spare cells and wear levelling, a user cannot sanitize all previous data by overwriting (section 3.1.1).

**Not even overwriting the whole drive is foolproof.** Two full passes were enough on most drives in the 2011 study. One drive still held about 1% (1 GB) after 20 passes (section 3.2.2). Three drives reported support for a built-in erase command and did not execute it correctly (abstract and section 3.2.1).

**Reading "zeros" does not mean the cells are empty.** After a Deallocate, the controller may return zeros through its normal interface even though the pages are still charged. Reading the cells requires desoldering the chip and bypassing the controller. That is lab work, not something an app does.

**Encryption changes what remains.** On a phone with file-based encryption, what survives in invalid pages is ciphertext. Reading it would require the file's key, which depends on keys tied to the hardware and, for many protection classes, to the user's passcode. Apple's guide does not claim that deleting a file destroys its key. What it does document is the other route: "Erase All Content and Settings" deletes a key held in a region of the NAND that can be addressed directly and wiped securely, which leaves every file cryptographically inaccessible. On Android, in the default configuration, the kernel derives a different key for each file from the class key. With certain inline-encryption optimizations, one key per class is used instead of one per file (AOSP documentation). That is why this article does not claim that a deleted photo is unrecoverable in every case: it claims that what remains is encrypted.

**The copies you do not see.** The photo may also exist as thumbnails, in caches or as cloud copies. Deleting the file on the phone does not touch those copies. They are outside the scope of this article.

## Practical consequences

For the person using the phone, "deleting" a file is a bookkeeping operation, not a destruction. It is fast precisely because it does not touch the cells. The space is truly reclaimed later, when the controller collects garbage.

If the concern is that nobody can recover the data when the device is sold or given away, the right tool is a factory reset of an encrypted phone. It works by destroying keys, which NIST SP 800-88 calls cryptographic erase, a *purge*-level technique. Apps that promise "secure deletion" by overwriting files run into the no-overwrite-in-place rule.

For people who design memories or systems, discard also matters for performance and lifetime. A controller that knows which pages are worthless does not copy them in every garbage collection. That cuts write amplification, the number of bytes the flash writes for every byte the system asks for (Cai et al., section 2.4). Less amplification means fewer cycles spent and a memory that lasts longer.

## Frequently asked questions

### Is TRIM the same as erasing the cells?

No. TRIM, UNMAP and Deallocate are notices that some logical sectors no longer hold anything useful. The physical erase is an internal controller operation on whole blocks. It may happen long after the notice, or not until the controller needs space. The NVMe specification itself compares Deallocate to ATA TRIM and SCSI UNMAP (section 3.3.3.2.1).

### Why can a recovery program sometimes bring back a deleted photo?

Because if the file system has not yet reused those sectors and the controller has not applied the discard, the pages are still mapped and readable. The program reads what the file system no longer references. Once the controller has applied the discard, a read through the normal interface returns fixed zeros or ones.

### Does deleting wear out the memory?

Deleting a file does not: it only changes metadata. What causes wear is the physical erase of blocks, which is part of garbage collection. Each block withstands a few thousand program/erase cycles in the generations cited: about 3000 for MLC and about 1000 for TLC (Cai et al.).

### Can a magnet wipe flash memory?

No. Flash stores electric charge, not magnetization. In the 2011 study, degaussing the chips with 14,000 and 8,000 gauss fields left the data intact in every case (Wei et al., section 3.2.3).

## Sources

- [Cai, Y., Ghose, S., Haratsch, E. F., Luo, Y., Mutlu, O. — Error Characterization, Mitigation, and Recovery in Flash-Memory-Based Solid-State Drives. Proceedings of the IEEE, 105(9), September 2017 (arXiv:1706.08642 version)](https://arxiv.org/pdf/1706.08642). States per cell (section 3.1), page and block organization (2.1 and 3.2), pulse programming and block erase with a shared substrate (3.4), FTL and garbage collection (2.3), write amplification (2.4), endurance (1 and 4) and electrons per cell (4.4).
- [Wei, M., Grupp, L. M., Spada, F. E., Swanson, S. — Reliably Erasing Data From Flash-Based Solid State Drives. USENIX FAST, 2011](https://static.usenix.org/event/fast11/tech/full_papers/Wei.pdf). Out-of-place writes and digital remnants (section 2.2), 6% to 25% extra flash and up to 16 stale copies (2.2 and Figure 1), recovery after file delete and overwrite (3.3 and Table 3), full-drive overwrite (3.2.2), built-in commands (3.2.1), degaussing (3.2.3) and TRIM (footnote 2).
- [NVM Express — NVM Command Set Specification, Revision 1.1 (August 2024)](https://nvmexpress.org/wp-content/uploads/NVM-Express-NVM-Command-Set-Specification-Revision-1.1-2024.08.05-Ratified.pdf). Dataset Management as an advisory command and the Deallocate attribute (section 3.3.3 and Figure 45), reads of deallocated blocks and the DLFEAT field (3.3.3.2.1 and Figure 114) and equivalence with TRIM and UNMAP.
- [NIST SP 800-88 Rev. 2 — Guidelines for Media Sanitization (September 2025)](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-88r2.pdf). Clear, purge and destroy levels (section 3.1), limits of overwriting on flash with spare cells (3.1.1) and cryptographic erase (3.2).
- [Apple — Apple Platform Security: Data Protection in Apple devices](https://support.apple.com/guide/security/sece8608431d/web). Per-file key wrapped by a class key in the metadata, file system key and Erase All Content and Settings.
- [Apple — Apple Platform Security: Effaceable Storage](https://support.apple.com/guide/security/aside/sec0183122de/1/web/1). NAND region for keys that can be addressed directly and wiped securely.
- [Android Open Source Project — File-based encryption](https://source.android.com/docs/security/features/encryption/file-based). Mandatory since Android 10, AES-256-XTS for contents and key derivation with HKDF-SHA512.
- [Android Open Source Project — Hardware-wrapped keys](https://source.android.com/docs/security/features/encryption/hw-wrapped-keys). Per-file or per-class key depending on the inline-encryption option.
