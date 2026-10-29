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
- **Restore on a new phone** needs only the account master key, which reaches the new phone through
  the synced keychain, a passkey or the recovery key (ADR 0012).
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
Account master key (AMK, 32 random bytes; Keychain / Keystore on each phone)
  ├── reaches a new phone via ── synced keychain · passkey · recovery key (ADR 0012)
        ├── derives ── local journal keys (database, media) on each phone
        └── wraps ── Notebook key (NK, 32 random bytes, one per notebook)
                       ├── encrypts packs, snapshots, media of that notebook
                       └── derives media-id key (keyed hash for content addressing)
```

- **The AMK** is generated with the journal on the user's first device, whatever storage they
  choose.
- **Synced keychain:** on iOS the AMK is a synchronizable Keychain item (iCloud Keychain); on
  Android a copy is kept in Block Store, backed up only when Google can encrypt it end to end.
  Both are end-to-end encrypted by the platform; a new phone of the same kind gets the AMK.
- **Passkey (with Cairn Cloud):** the WebAuthn PRF extension, evaluated with the fixed salt
  `"cairn-amk-v1"`, gives a 32-byte secret that never leaves the person's devices. A wrapping key
  is derived from it (`crypto_kdf_derive_from_key`, context `"cairn_pk"`, subkey 1), and the AMK
  is stored wrapped with it at `k/amk/{credential_id}.key` (envelope kind 3, key id zeros). Any
  phone where the passkey is available can unwrap the AMK. Local backup files carry the same
  wrapped copy.
- The **recovery key**, an optional fallback, is the AMK written out for people: Crockford Base32, 56 characters
  in 14 groups of four, holding the 32 key bytes and a 3-byte checksum (the first 3 bytes of a
  16-byte BLAKE2b hash of `"cairn-recovery-v1"` followed by the key). See ADR 0010.
- **The journal on the phone** is encrypted with keys derived from the AMK (context `"cairn_lc"`:
  subkey 1 for the database, 2 for media), so whatever brings the AMK to a phone also opens a
  local backup file restored there. Phone backups never carry the journal (ADR 0011).
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
  k/amk/{credential_id}.key                   AMK wrapped by a passkey (ADR 0012)
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
  phone clocks drift during weeks offline. A full counter carries into `wall_ms` (+1 ms) rather
  than failing.
- **`device_id` identifies the install,** not the journal: a journal restored with the recovery key,
  on a new phone or after a reset, gets a new one, so two phones never write under the same id.

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

1. Get the AMK: from the synced keychain, by signing in with a passkey and unwrapping
   `k/amk/{credential_id}.key`, or from the recovery key.
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
| HLC ordering | Clock skew, equal timestamps broken by device id, and a full counter carrying into the next millisecond |
| Resolving conflicts | Concurrent upserts, upsert vs delete, and keeping the earlier version |

## Open questions

- **Pack serialization:** JSON (easy to debug) vs CBOR (compact).
- **Pack size and batching** when coming back online after weeks: one pack per change, or
  batching by time?
- **Key rotation** and removing a partner from a shared trip (v1.5).
- **Snapshot policy** thresholds, to be tuned with real thru-hike data volumes.
