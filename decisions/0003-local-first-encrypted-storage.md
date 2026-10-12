# 0003. Local-first journal with user-chosen, end-to-end encrypted storage

- **Status:** Accepted; extended by [0010](0010-data-layer.md) (the copy on the phone is encrypted too)
- **Date:** 2026-10-05

## Context

A journal is intimate, and thru-hikers spend weeks without signal. To trust Cairn over paper,
users need:

- an app that works fully offline;
- a journal that cannot be lost when a phone ends up in a river;
- proof that Cairn doesn't need their data.

## Decision

- **The phone holds the journal of record**: SQLite plus photo files. Every other copy is a backup
  or sync target.
- **The user chooses where copies go,** and can switch at any time; Cairn moves the data:
  1. **This device only.** Nothing leaves the phone. Cairn reminds the user to export a backup.
  2. **Their own iCloud / Google Drive** (v1.5). Encrypted files go straight from the phone to the
     user's account. Cairn's servers are not involved.
  3. **Cairn Cloud.** Encrypted blobs are stored in R2. This option supports switching between
     iPhone and Android, and shared trips.
- **Encryption happens on the phone** before anything leaves it, using XChaCha20-Poly1305 through
  libsodium. Keys live in the Keychain or Keystore. The recovery key is given to the user as a
  printable page. The format is defined in [specs/sync-format.md](../specs/sync-format.md).
- **The server sees only metadata:** account and device ids, sync cursors, blob ids and sizes.
  Readable content reaches Cairn only when the user shares or prints a page.
- **Storage sits behind a `StorageProvider` interface** in the app (`None`, `CairnCloud`, later
  `ICloud` and `GoogleDrive`).
- **Export** to PDF and an open archive (Markdown, original photos, GPX) is always available and
  always free.

## Consequences

- **Cairn cannot recover a journal** if the user loses every device and their recovery key. The
  app has to make the recovery key hard to skip and easy to keep.
- **Server-side features that need to read content cannot exist,** for example search or AI
  summaries on the server. This fits the principle that AI never writes in your journal.
- **Shared trips need per-notebook keys** that can be given to trip partners (outlined in the spec,
  built in v1.5).
- **Three storage backends is real work.** v1 ships `None` and `CairnCloud`; own-cloud storage
  follows in v1.5.
