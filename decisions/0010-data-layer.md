# 0010. Data layer: an encrypted journal on the phone

- **Status:** Accepted
- **Date:** 2026-10-12
- **Extends:** [ADR 0003](0003-local-first-encrypted-storage.md), which encrypts everything that
  leaves the phone but left the copy on the phone open

## Context

Stage 6 builds the data layer every v1 screen reads and writes through. Two questions came first:

- **Should the journal be encrypted on the phone too?** iOS and Android already encrypt app files
  while the phone is locked. A second layer also protects against copied backups and tools that
  read an unlocked phone. Mihir chose to encrypt it, for total privacy.
- **What is the lowest Android version?** The app PR #2 review found that the capture code would
  crash below Android 10, and needs Android 12 as written.

The code is on the `feat/data-layer` branch of cairn-app (`lib/data/`).

## Decision

**Storage:**

- **One SQLite database, `journal.db`, through drift.** It holds notebooks, pages, page items
  (writing, taped photos, sketches), back-pocket items, location points and media records.
- **Photos and other files sit in `media/`**, one file per media record, named by its id.
- **Screens go through repositories** on one `Journal` object and never write SQL.

**Encryption on the phone:**

| What | How |
|---|---|
| `journal.db` | SQLite3MultipleCiphers, the encrypted SQLite build drift recommends, with its default ChaCha20-Poly1305 cipher |
| Each media file | libsodium secretstream (XChaCha20-Poly1305) in 64 KiB chunks. Every chunk is bound to the media id, so a file moved to another id fails to decrypt |
| Keys | The account master key from specs/sync-format.md: 32 random bytes, created with the journal. A database key and a media key are derived from it with libsodium's KDF |

- **The master key lives in the Keychain or Keystore.** On iOS it can be read after the first
  unlock (so captures work while the phone is locked). It comes back when a backup is restored to
  the same iPhone, but is never copied to another phone. On Android it never leaves the phone, and
  the "wipe everything on a read error" default is turned off.
- **The person holds the same key as their recovery key,** shown once at setup to print or write
  down. Every user gets one, whichever storage they choose.
- **The recovery key is Crockford Base32: 56 characters in 14 groups of four.** It is the 32 key
  bytes plus a 3-byte checksum.
  - Reading it accepts lower case, spaces, and O, I or L typed for 0 and 1.
  - The checksum catches a mistyped character.
  - This settles the encoding question in specs/sync-format.md.
- **If the journal is here but its key isn't, Cairn asks for the recovery key.** It checks the key
  opens this journal before keeping it. It never quietly starts a new, empty journal over the old
  one.

**OS backups include the journal, still encrypted:**

| Situation | What comes back | How it opens |
|---|---|---|
| iPhone reset, restored from its own backup | Everything | Automatically: the key comes back too |
| New iPhone from an iCloud backup | Everything | Recovery key |
| Android reset, restored from Google's backup | Database (writing, points); no photos | Recovery key |
| New Android phone by phone-to-phone transfer | Everything | Recovery key |

- **Android cloud backup holds only the database.** Android skips an app's whole backup above
  25 MB, so photos are left out to keep the writing safe. Phone-to-phone transfer has no cap.
- **Apple and Google can't read these backups.** Without the recovery key, nobody can.

**Records and the change log** (specs/sync-format.md):

- **Ids are UUIDv7, generated on the phone.** Each record carries the hybrid logical clock stamp of
  its last change.
- **Deletes leave tombstones.** There are no foreign keys, because packs from other devices can
  arrive in any order. A save never overwrites a tombstone, so a stale copy, such as a late
  autosave, can't undo a delete.
- **The device id belongs to the install, not the journal.** It is stored in the journal but
  replaced whenever a journal is unlocked with the recovery key. A transferred or restored copy
  therefore never writes as the phone it came from, which may still be in use.
- **The clock never stops writes.** A full counter carries into the next millisecond, so a phone
  whose clock was once ahead keeps working until real time catches up.
- **Every write also logs a change in the same transaction:** record type, id, stamp, upsert or
  delete, and the whole record. Sync will send the changes not yet packed.
- **Deletes reach everything that depends on them.** Deleting a notebook deletes its pages, their
  items, its pocket, the location points pinned to them, and photos nothing else uses (record and
  file).

**Android 12 is the minimum (minSdk 31):**

- The capture code already needs it.
- Google's figures for December 2025 put Android 12 or newer on about 69% of active devices.
  Android 10 would reach about 91%, but would need a second way of reading location; Mihir chose
  the simpler build.

## Consequences

- **Losing the recovery key and the phone loses the journal.** That's the price of Apple and Google
  never being able to read it. Setup has to make keeping the recovery key feel important: a
  printable page, and a check that it was written down.
- **"This device only" no longer means nothing leaves the phone** (as ADR 0003 put it). It means
  Cairn sends nothing anywhere, while the phone's own backup includes the journal, encrypted. The
  storage choice has to say so plainly.
  - **Open question:** should that option let people keep the journal out of the phone's backup
    too? Both platforms allow it (iOS can exclude the folder, Android has a no-backup folder), at
    the cost of losing the journal on a reset.
- **On Android with "This device only" storage, photos survive a reset only through Cairn Cloud,
  own cloud (v1.5) or an export.** The storage choice has to say so plainly.
- **A restored journal opens on an unlock screen,** and pages whose photos didn't come back show a
  placeholder.
- **The widget, tile and Action button can't write into the encrypted database** while Cairn isn't
  running. The capture stage has to design a small inbox that the app imports on its next launch.
- **Photos are decrypted every time they're shown.** That is fast for one photo; the page composer
  will need an in-memory cache of decoded thumbnails.
- **About 3 in 10 Android phones can't install Cairn.** Revisit if beta testers are shut out; Google
  Play Console will show the real share.
- **Tested:**
  - unit tests on the computer, covering encryption, wrong and missing keys, the change log, HLC
    ordering, cascading deletes and media tampering;
  - an integration test on the iOS simulator and the Android emulator, with the real Keychain and
    Keystore, including a reset followed by unlocking with the recovery key;
  - the data inspector on both devices, confirming the database file is unreadable and photos
    decrypt;
  - a real Android backup, uninstall and restore on the emulator, unlocked by typing the recovery
    key. The iPhone same-device restore needs a real phone to test.
