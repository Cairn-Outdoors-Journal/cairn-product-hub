# Sync and encryption format

- **Format version:** 0 (draft outline; not frozen)
- **Implemented by:** `cairn-app` (writes and reads everything), `cairn-api` (stores objects and
  tracks heads without reading contents)
- **Related:** [ADR 0003](../decisions/0003-local-first-encrypted-storage.md),
  [ARCHITECTURE.md](../ARCHITECTURE.md)

This document defines what the phone encrypts, how the encrypted objects are laid out in storage, and
how devices bring each other up to date. It will be frozen as **format version 1** before the v1 beta,
with test vectors that both repos run in CI.

## Goals

- **Readable only on the user's devices:** journal content, including location points, can only be
  read on the user's own devices. Storage (R2, iCloud, Google Drive) holds encrypted objects only.
- **The same objects work with any storage provider.** Moving between providers copies objects
  without re-encrypting them.
- **Restore on a new phone** needs only the recovery key.
- **Weeks offline then one sync** must be fast and safe.

**Non-goals for v1:** real-time collaboration on one page, server-side search, key rotation.

## Terms

| Term | Meaning |
|---|---|
| Record | One row of journal data: a notebook, page, section, taped item, pocket item, location point, person, and so on |
| Change | An upsert or delete of one record, stamped with a hybrid logical clock |
| Pack | An encrypted batch of changes written by one device, numbered in sequence for that device |
| Media | An encrypted photo, sketch, voice note or GPX file |
| Snapshot | An encrypted full copy of a notebook's records up to a set of pack sequence numbers |
| Head | The highest pack sequence number each device has written for a notebook |

## Keys

```
Recovery key (shown once, printed as a page)
  └── Account master key (AMK, 32 random bytes; Keychain / Keystore)
        └── wraps ── Notebook key (NK, 32 random bytes, one per notebook)
                       ├── encrypts packs, snapshots, media of that notebook
                       └── derives media-id key (keyed hash for content addressing)
```

- **The AMK** is generated on the user's first device. The **recovery key** is the AMK encoded
  for people to write down. The encoding is an open question.
- **Each notebook has its own key (NK).** It is stored wrapped by the AMK at `k/{notebook_id}.key`.
  Per-notebook keys let a trip be shared later without exposing the rest of the journal.
- **Subkeys** are derived with libsodium `crypto_kdf_derive_from_key` (context `"cairn_v1"`), never
  reused across purposes.
- **Shared trips (v1.5):** the NK is wrapped to each partner's X25519 public key
  (`crypto_box_seal`).

## Encrypted object envelope

Every stored object except large media uses this envelope:

| Field | Size | Notes |
|---|---|---|
| magic | 4 bytes | `CRN1` |
| format version | 1 byte | `0` while draft |
| kind | 1 byte | 1 pack · 2 snapshot · 3 wrapped key · 4 media manifest |
| key id | 16 bytes | Notebook id, or zeros for the account key |
| nonce | 24 bytes | Random |
| ciphertext + tag | rest | `crypto_aead_xchacha20poly1305_ietf` |

**Associated data** = all header bytes plus the object's storage path. This binds an object to its
location, so a blob cannot be swapped or moved without decryption failing.

**Large media** (photos, voice notes) use `crypto_secretstream_xchacha20poly1305` in 64 KiB chunks,
with the same header, so they can be encrypted and decrypted without loading the whole file.

## Storage layout

All paths sit under the account's prefix. Ids in paths are random (UUIDv4 or keyed hashes) so they
reveal nothing about content or creation time.

```
a/{account_id}/
  k/{notebook_id}.key                         wrapped notebook key
  n/{notebook_id}/
    p/{device_id}/{seq:010}.pack              change packs, per device, gap-free sequence
    s/{snapshot_id}.snap                      snapshots
    m/{media_id}                              media (secretstream)
```

- **Media ids** are a keyed BLAKE2b hash of the plaintext, using a key derived from the NK.
  Identical photos are stored once within a notebook, but equality never leaks across notebooks
  or accounts.
- **Cairn Cloud:** the API issues short-lived pre-signed URLs, only for paths under the caller's own
  account prefix, with size limits.
- **Own cloud (v1.5):** the same paths, mapped to files in an app-private folder.

## Records and changes

- **Inside packs,** records use UUIDv7 ids. This is safe because packs are encrypted.
- **Each change contains:** `record_type`, `record_id`, `hlc`, `device_id`, `op` (`upsert` |
  `delete`), and for upserts the full record fields.
- **A pack** is a compressed, serialized list of changes, then encrypted. The serialization
  (JSON vs CBOR) is an open question.
- **Clock:** a hybrid logical clock `(wall_ms, counter, device_id)`, so ordering stays correct when
  phone clocks drift during weeks offline.

### Resolving conflicts

- **The last write wins per record,** ordered by HLC.
- **Conflicts are rare by design:** each page has a single author, even on shared trips. The
  remaining case is the same person editing the same page on two of their own devices while both
  are offline.
- **The losing version is never silently dropped.** It is kept as an "earlier version" of the page
  for 30 days and shown in the page history.
- **Deletes are tombstones.** Media is garbage-collected once no live record references it.

## Sync procedure

**Push (device has new changes):**

1. Upload any new media first.
2. Write the next pack for this device (`seq = last + 1`).
3. Advance the head through the API with compare-and-set on the expected previous `seq`. If that
   fails, pull first, then retry.

Packs never reference media that hasn't been uploaded yet.

**Pull:**

1. Fetch the heads for the notebook.
2. Download the missing packs for each device in sequence order.
3. Apply them in HLC order.
4. Download media lazily, when a page needs it.

**Restore on a new device:**

1. Enter the recovery key → AMK.
2. List the notebook keys.
3. For each notebook, load the newest snapshot, then the packs after it.

**Snapshots:** a device writes one after about 200 packs or 30 days, so a restore never replays
months of packs.

**The server stores only:** account and device ids, notebook ids, heads per device, and object
sizes. It never receives keys or plaintext.

## Test vectors (to be written before format version 1)

| Vector | Checks |
|---|---|
| Envelope round-trip | Fixed key, nonce and path give the exact expected bytes |
| Path binding | Decrypting with a different path fails |
| Media id | Keyed hash of a fixed input under a fixed NK |
| HLC ordering | Clock skew and equal timestamps break ties by device id |
| Resolving conflicts | Concurrent upserts, upsert vs delete, and keeping the earlier version |

## Open questions

- **Recovery key encoding:** a 24-word list vs grouped Base32 with a checksum. Which is easier to
  write in a tent?
- **Pack serialization:** JSON (easy to debug) vs CBOR (compact).
- **Pack size and batching** when coming back online after weeks: one pack per change, or
  batching by time?
- **Key rotation** and removing a partner from a shared trip (v1.5).
- **Snapshot policy** thresholds, to be tuned with real thru-hike data volumes.
