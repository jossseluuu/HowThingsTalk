---
title: "How does your phone keep its files from getting corrupted when it shuts off suddenly?"
section: "06_Storage"
language: "en"
status: "published"
order: 2
depth: "intermediate"
summary: "Saving a file means writing to several places in storage, and a power cut between two of those writes would leave the file system's structures half-updated. To prevent this, the file system groups changes into transactions that are applied completely or not at all. It does so with a journal (ext4), with copy-on-write and checkpoints (APFS), or with a log and alternating checkpoints (F2FS). All of them depend on the storage honouring requests to flush its cache. What they protect is the file system's structure, not the most recent data that had not yet been committed."
prerequisites:
  - "What a file and a directory are"
  - "Pages, blocks and out-of-place writes in flash memory (article 06·01)"
translation: "../../ES/06_Almacenamiento/02-ComoSobreviveElSistemaDeArchivosAUnApagon.md"
---

# How does your phone keep its files from getting corrupted when it shuts off suddenly?

## Question and framing

Your phone freezes while saving a photo. Or it is force-restarted, or its battery dies abruptly. When you turn it back on, the gallery, the chats and the settings are almost always fine. This article asks why: what does the file system do so that a cut in the middle of a write does not leave storage in an inconsistent state?

The path starts with the problem: a single user action turns into several writes in different places. It then explains the three strategies used by phone file systems. They are ext4's journal, APFS's copy-on-write and F2FS's checkpoints. It ends with what these techniques do not guarantee. It is not a guide to recovering data or repairing a disk, and it does not compare phone models.

The depth is intermediate. You only need to know what a file and a directory are. It helps to have read [article 06·01](01-WhatHappensInFlashWhenYouDeleteAPhoto.md), which explains why flash memory writes in pages and never overwrites in place.

## Short answer

The file system never leaves a half-finished update in its final location: it first records the complete change somewhere else, marks the end with a checksummed record, and at boot discards anything that lacks that record.

### 60-second summary

Saving a new photo takes several writes. The photo's data has to be written. Its inode has to be created: the record with its size and the list of blocks it occupies. Those blocks have to be marked as used, and the name has to be added to the directory. If power goes out between two of those writes, the directory may point to an empty record, or blocks may stay marked as used with nothing using them.

The solution is to treat the group of writes as a transaction: it is applied entirely or not at all. There are three ways to do it:

1. **Journal (ext4).** Before touching the structures, the complete change is written to a separate area called the journal and closed with a commit block. At boot, committed transactions are replayed and the rest are discarded.
2. **Copy-on-write (APFS).** Nothing is modified in place. New versions are written to free blocks, and then a checkpoint that points to them is published. If the checkpoint is left half-written, it is ignored and the previous one stands.
3. **Log with checkpoints (F2FS).** Writes always go to the end of a log, and two copies of the checkpoint alternate. At boot, the most recent valid copy is chosen.

All three depend on one condition: that the storage really writes what it says it has written, in the order it is asked to.

## Mental model

Picture an accountant keeping a ledger. Before moving money between accounts, they write everything they are about to do in a separate notebook: "subtract 50 from account A, add 50 to account B". Once the note is complete, they sign "done" underneath. Only then do they correct the ledger. If interrupted, they check the notebook when they come back. If the last note is signed, they redo it in full. If it is not signed, they cross it out. The ledger never ends up with money taken from A and not added to B.

The analogy breaks down in three places. First, the accountant sees the paper and knows the order they wrote in. The operating system does not: the storage has a cache that can acknowledge a write before saving it, and it can reorder writes. It has to be forced with explicit commands. Second, not every system uses a notebook. APFS and F2FS never correct the ledger in place: they write a new page and change a single pointer at the end. Third, the notebook protects the bookkeeping, not what is inside the envelopes. With ext4's default settings, the journal records metadata, not the contents of your files.

## Technical mechanism

### 1. What happens in the system

**The problem: one action, several writes.** Creating a photo file changes at least four structures: the data blocks, the inode, the free-block map and the directory entry. They live in different places in storage. No storage device writes several locations atomically at once. A cut between two writes produces combinations that should never exist:

- the directory contains the name, but the inode has not been written yet;
- the inode points to blocks that the map still marks as free, which another file could reuse;
- the blocks are marked as used, but no inode uses them, and that space is lost.

**The aggravating factor: order gets lost along the way.** Between the app and the cells there are two caches. The first is the operating system's page cache, in RAM. The second is the storage device's own write cache. The Linux kernel documentation warns that many devices signal that a write has completed before the data reaches non-volatile storage. That is why the operating system must force that cache to be flushed on `fsync`, on `sync` and at unmount (kernel documentation, *Explicit volatile write back cache control*).

The kernel has two tools to impose order:

- **Pre-flush (`REQ_PREFLUSH`).** Before running a write, the device flushes its cache. Everything written earlier is then in non-volatile storage.
- **Forced unit access (`REQ_FUA`).** The device only reports that write as complete once its data is in non-volatile storage.

With these two commands, the file system can guarantee that a given block reaches the medium after certain others. Every strategy below is built on that guarantee.

### 2. What each component does

**ext4's journal (jbd2).** ext4 is one of the file systems that Android keeps patched on a frequent security schedule, alongside F2FS (AOSP documentation). Its journal works like this (kernel documentation, *Journal (jbd2)*):

1. **Descriptor block.** It opens the transaction and lists the final location of each block that follows.
2. **Journal data blocks.** Exact copies of the modified metadata blocks: inodes, maps, directories.
3. **Commit block.** It closes the transaction. It is written only once everything before it is on the medium and out of the write cache. It can carry a checksum of the transaction, usually CRC32 or CRC32C.
4. **Checkpoint.** Later, the blocks are copied to their final location and the transaction is retired from the journal.

When mounting after a cut, ext4 walks the journal. Transactions with a valid commit block are replayed up to the last committed one. If the commit is missing or the checksum does not match, the transaction is discarded. The result is that a metadata update is never left half-done.

ext4 has three data modes (ext4 admin guide):

- **`data=ordered`**, the default. Only metadata goes through the journal. File data is written to its final location before the metadata pointing to it is committed. That way, an inode never points to blocks holding leftovers of another file.
- **`data=journal`**. Data and metadata both go through the journal. It is slower because everything is written twice.
- **`data=writeback`**. It imposes no order between data and metadata. After a cut, a file may contain old data.

Write barriers, which enforce the order of commits while the cache is enabled, are on by default. The running transaction is committed at least every 5 seconds by default (the `commit` option).

**APFS copy-on-write.** APFS is the file system of Apple devices. Its specification is blunt: objects on disk are never modified in place (Apple File System Reference, "Objects"). To change an object, a new copy is written to another block. Every object carries a 64-bit Fletcher checksum and the number of the last transaction that modified it. Those transaction numbers only increase and are never reused.

New versions become visible through **checkpoints**. The state held in memory is periodically written into a checkpoint, followed by a copy of the container superblock, the root structure. There are two areas: a descriptor area, with the superblocks and maps, and a data area, with the state objects. If writing a checkpoint is interrupted, that checkpoint is invalid. At mount, the most recent valid checkpoint is used, and the system rolls back to the last consistent state (Apple File System Reference, "About Apple File System").

**F2FS's log and checkpoints.** F2FS is a log-structured file system designed for flash memory. It writes new blocks to free space instead of overwriting old ones (Lee et al., 2015, section 1). Its protection against cuts has two parts (section 2.7):

- **Two alternating checkpoint packs.** One holds the last stable version and the other the intermediate one. Each pack has a header and a footer with a version number that grows with every checkpoint. At mount, a pack is valid if its header and footer match. If both are valid, the one with the higher version wins. F2FS creates a checkpoint at moments such as `sync`, unmount or foreground cleaning.
- **Roll-forward recovery for `fsync`.** Creating a full checkpoint on every `fsync` would be slow. Instead, F2FS writes only the data blocks and the direct node blocks pointing to them, with a special flag. At boot, it looks for flagged nodes newer than the last checkpoint and applies those changes on top (section 2.7.2).

On a Galaxy S4 running Android 4.2.2, F2FS ran a SQLite workload up to 2 times faster than ext4 and wrote about 46% less data (section 3.2.1). These are 2015 measurements on one specific device. They show why the cost of each `fsync` matters on a phone, not what it will be on a current device.

**The app, the last layer.** The file system protects its own structures. It does not know what the contents of your files mean. Apps that need atomicity build it on top. SQLite, the database many apps use for messages, contacts or settings, does it with a rollback journal (SQLite documentation, *Atomic Commit In SQLite*):

1. It copies the original contents of the pages it is about to change into a journal file.
2. It flushes that journal to storage with `fsync`. In the default mode it does so twice: first the contents and then the header with the page count.
3. It writes the changes into the database and flushes it to storage.
4. It deletes the journal. That deletion is the instant the transaction commits.

If it finds a complete journal when opening the database, it restores the original pages. The database returns to its state before the interrupted transaction.

### 3. Key parameters

| Parameter | Value | Scenario and source |
|---|---|---|
| ext4 default data mode | `data=ordered` | ext4 admin guide |
| ext4 maximum commit interval | 5 s | `commit` option, default value (ext4 admin guide) |
| ext4 write barriers | on | default (ext4 admin guide) |
| Commit block checksum | CRC32 or CRC32C, usually | jbd2 journal (kernel documentation) |
| Per-object checksum in APFS | 64-bit Fletcher, 8 bytes | `o_cksum` field (Apple File System Reference) |
| Checkpoint packs in F2FS | 2, alternating | Lee et al., section 2.1 |
| SQLite on F2FS vs ext4 | up to 2× faster, ~46% fewer writes | Galaxy S4, Android 4.2.2, WAL mode (Lee et al., 3.2.1) |
| Journal flushes per transaction in SQLite | 2 | `synchronous=FULL`, the default (SQLite documentation) |
| SSDs that failed under power cuts | 13 of 15 | 15 SSDs from 5 vendors, over 3,000 induced cuts (Zheng et al., 2013) |

### 4. Diagram or flow

An ext4 transaction in `ordered` mode and what happens if power goes out at each point:

```text
Time ────────────────────────────────────────────────────────────────────────>

 1. Photo data              2. Journal:              3. Journal:          4. Checkpoint:
    to its final place         descriptor + inode,      commit block         inode, map and
    (flush cache)              map and directory        (FUA)                directory to their place
                               (flush cache)
        │                          │                        │                     │
   cut A                      cut B                    cut C                 cut D
        │                          │                        │                     │
        v                          v                        v                     v
 Blocks written that        No commit: it is         Committed: replayed   Committed: replayed at
 nothing references yet.    discarded. The file      at mount. The photo   mount (replaying twice
 The photo does not exist.  does not exist.          exists.               gives the same result).
```

At no point is there a directory pointing to a half-written inode. The worst outcome is losing the whole photo, never half a photo with corrupted metadata.

F2FS checkpoint alternation:

```text
           CP pack #0                  CP pack #1
           ┌──────────────┐            ┌──────────────┐
 before    │ version 41 ✓ │            │ version 42 ✓ │  <- most recent valid
           └──────────────┘            └──────────────┘
 writes    │ header 43    │  cut
 pack 0    │ ...          │  ──>  header 43 ≠ footer 41: pack 0 invalid
           └──────────────┘
 at mount ───────────────────────>  pack 1 is used (version 42)
```

## Limits, exceptions, and real-world cases

**Consistent does not mean complete.** These techniques guarantee that the file system's structures stay consistent. They do not guarantee that the last thing you wrote survives. On ext4, a cut can lose up to the last 5 seconds of metadata changes. With delayed block allocation it can lose even older data, because writing it does not start until another kernel timer expires (ext4 admin guide). An app that needs something to survive must call `fsync` and wait for it to finish.

**The storage may not comply.** Everything above assumes the storage honours cache flushes and ordering. Zheng and colleagues induced more than 3,000 power cuts on 15 SSDs from 5 vendors. Thirteen of the fifteen showed failures (FAST 2013, abstract). They found corrupted bits on 3 drives and writes cut off midway on another 3. On 8 they found unserializable writes: the final state matched no possible order of the requested writes. One drive lost internal metadata and another became unusable (table 5). Four drives had power-loss protection, such as a supercapacitor. Two of them did not fail and the other two did (tables 3 and 5). The study used 2013 computer SSDs, not phone storage. No equivalent study of the UFS or eMMC storage in current phones was reviewed here, so their behaviour under power cuts is **unverified**.

**The file system does not catch every data error.** On ext4 in `ordered` mode, file data is not covered by a journal checksum. SQLite does not add redundancy to the database either, and its documentation admits it cannot defend against an operating system or storage device that does not keep its promises (sections 2 and 9.4).

**Recovery has a cost.** Replaying the journal or finding the valid checkpoint takes time at boot. That is why a phone may take a little longer to start after a forced restart than after a normal shutdown. The size of the delay depends on the device and on how much was pending; there is no verified general figure.

**A phone rarely loses power abruptly.** The battery is built in and the system shuts down in an orderly way when it runs out. The real cases are mostly kernel crashes and forced restarts. In a crash, everything in RAM is lost, including the page cache, just as in a power cut. For the file system, recovery is the same.

## Practical consequences

For the person using the phone, a forced restart rarely damages storage. What can be lost is whatever was being saved at that moment: the photo from a second ago, the message you just typed or a settings change. Whether it survives depends on whether the app had confirmed the write with `fsync`.

A normal shutdown is safer than a forced one for a concrete reason. At unmount, the operating system flushes every cache and writes a clean checkpoint. There is nothing left to recover at boot.

For people designing devices and apps, every `fsync` costs time and flash writes. SQLite flushes its journal twice per transaction in the default mode. That is why phone file systems work hard to make `fsync` cheaper, as F2FS's roll-forward recovery does. It is also why apps group many changes into one transaction instead of committing them one by one.

## Frequently asked questions

### Is this the same as flash memory's out-of-place writes?

No, though they look alike. The flash translation layer (FTL) writes out of place because of a physical limitation: it cannot overwrite a page ([article 06·01](01-WhatHappensInFlashWhenYouDeleteAPhoto.md)). APFS's copy-on-write and F2FS's log write out of place as a design choice: to keep the previous version intact until the new one is complete. They are two separate layers, and each one protects its own translation table.

### Does the journal keep a copy of my photos?

With ext4's default settings, no. `ordered` mode only sends metadata through the journal. The photo's data is written once, to its final location, before the metadata is committed. `data=journal` mode does write the data twice, once to the journal and once to its place, which is why it is slower.

### Why do you have to "eject" a USB stick before removing it?

Because of the caches. Part of what you copied may still be in the computer's RAM or in the USB stick's cache. Ejecting unmounts the file system, and the kernel forces everything into non-volatile storage at unmount. If you pull it out first, you interrupt the pending writes just like a power cut.

### Do you need to check the disk after a power cut, like in the old days?

With a journaling or copy-on-write file system, usually not. Replaying the journal or picking the last valid checkpoint makes the structures consistent quickly. Full checking tools still exist for other kinds of damage, such as blocks the storage device has corrupted.

## Sources

- [The Linux Kernel documentation — ext4 Data Structures and Algorithms: Journal (jbd2)](https://www.kernel.org/doc/html/latest/filesystems/ext4/journal.html). Transaction structure (descriptor, data, revocation and commit blocks), discarding transactions with no commit or a bad checksum, commit block checksums and the default mode that only journals metadata.
- [The Linux Kernel documentation — ext4 General Information](https://docs.kernel.org/admin-guide/ext4.html). `data=journal`, `data=ordered` and `data=writeback` modes, the 5 s default `commit` interval and possible loss of the latest changes, write barriers on by default and journal replay at mount.
- [The Linux Kernel documentation — Explicit volatile write back cache control](https://docs.kernel.org/block/writeback_cache_control.html). Volatile write caches that acknowledge before saving, mandatory flushing on `fsync`, `sync` and unmount, and the semantics of `REQ_PREFLUSH` and `REQ_FUA`.
- [Apple — Apple File System Reference (2020)](https://developer.apple.com/support/apple-file-system/Apple-File-System-Reference.pdf). Container crash protection, multiple superblock copies, checkpoint descriptor and data areas, interrupted checkpoints treated as invalid ("About Apple File System"), objects never modified in place, 64-bit Fletcher checksum and increasing transaction identifiers ("Objects").
- [Lee, C., Sim, D., Hwang, J., Cho, S. — F2FS: A New File System for Flash Storage. USENIX FAST, 2015](https://www.usenix.org/conference/fast15/technical-sessions/presentation/lee). Log-structured design (section 1), two checkpoint packs and choosing the valid one (2.1 and 2.7), roll-forward recovery for `fsync` (2.7.2) and measurements on a Galaxy S4 (3.1 and 3.2.1).
- [The Linux Kernel documentation — WHAT IS Flash-Friendly File System (F2FS)?](https://docs.kernel.org/filesystems/f2fs.html). Log-structured basis, `checkpoint`, `fsync_mode` and `disable_roll_forward` options.
- [Zheng, M., Tucek, J., Qin, F., Lillibridge, M. — Understanding the Robustness of SSDs under Power Fault. USENIX FAST, 2013](https://www.usenix.org/system/files/conference/fast13/fast13-final80.pdf). Fifteen SSDs from five vendors, over 3,000 induced cuts, thirteen with failures (abstract), drives with power-loss protection (table 3) and failure types per drive (table 5).
- [SQLite — Atomic Commit In SQLite](https://www.sqlite.org/atomiccommit.html). Rollback journal and commit point (sections 3.4 to 3.12), hot-journal recovery (4), double flush under `synchronous=FULL` (6.2), storage assumptions (2) and limits when flushes are not honoured (9.2 and 9.4).
- [Android Open Source Project — Android kernel file system support](https://source.android.com/docs/core/architecture/android-kernel-file-system-support). ext4 and F2FS among the file systems that receive frequent security patches.
