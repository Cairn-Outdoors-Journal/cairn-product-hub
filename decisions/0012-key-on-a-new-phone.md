# 0012. Getting the key onto a new phone

- **Status:** Accepted
- **Date:** 2026-10-29
- **Amends:** [ADR 0010](0010-data-layer.md), "the master key never leaves the phone", and the
  recovery key's place at setup

## Context

Everything in the journal is encrypted with keys that come from one account master key
([ADR 0010](0010-data-layer.md), [sync spec](../specs/sync-format.md)). Until now the only way to
get that key onto a new phone was the recovery key, written down at setup. Most people won't
keep it safe, and few will remember it when they switch phones.

The goal: moving to a new phone should just work for nearly everyone, while Cairn still can never
read the key or the journal.

## Decision

The key reaches a new phone in up to three ways.

1. **Synced keychain, by default for everyone.** These are the phone makers' end-to-end encrypted
   key stores, protected by the phone's passcode; neither the maker nor Cairn can read them.
   - **iOS:** a copy is a synchronizable Keychain item (iCloud Keychain), readable after the
     first unlock, next to the this-device-only item the app works with. A new iPhone signed in
     to the same Apple account has it. The copy is shared live between the person's devices, so
     it is only written when missing: a second iPhone that started a journal before the copy
     arrived keeps its own key instead of replacing the first phone's.
   - **Android:** a copy goes into Block Store, saved to Google's cloud only when Google can
     encrypt it end to end, which needs a screen lock. The Keystore copy stays the one the app
     works with. A new Android phone restored from the same Google account has it.
   - Each phone reads its own key first and takes the copy only when it has none.
   - This covers moving between phones of the same kind. It doesn't cover iPhone to Android or
     back.
2. **Passkey, for Cairn Cloud users.**
   - The person creates a passkey for Cairn (Face ID or fingerprint, synced by their passkey
     manager).
   - The passkey's PRF extension gives a secret that never leaves the person's devices. That
     secret wraps the master key, and the wrapped copy is stored in Cairn Cloud.
   - On any phone where the passkey is available, the app unwraps the key. This covers iPhone ↔
     Android when the passkey manager runs on both, such as Google Password Manager or 1Password.
   - The same wrapped key goes inside local backup files, so a passkey can open one on another
     phone too.
   - Built with Cairn Cloud and the local backup file. It needs Cairn's domain set up for
     passkeys.
3. **Recovery key, as a fallback.**
   - No longer a setup step that everyone must complete.
   - Settings offers "Show recovery key" for anyone who wants a paper copy.
   - It still unlocks everything, anywhere.

## Consequences

- **Cairn still can't read anything.** Every path keeps the key inside the person's own devices
  or end-to-end encrypted stores.
- **A journal can still be lost** if all of these are true: the synced keychain is off (iCloud
  Keychain disabled, or no screen lock on Android), the person has no passkey and no recovery
  key, and the phone is gone. The app should say when the key isn't syncing, for example on the
  storage screen.
- **Apple and Google hold an encrypted copy of the key, but never the journal**
  ([ADR 0011](0011-no-phone-backups.md)). The key alone opens nothing without the data.
- **Setup (stage 7d) changes:** no mandatory recovery-key page. A short note says how the key
  travels, and Settings has "Show recovery key".
- **Built now:** the synced keychain on both platforms. **Later:** passkeys, with Cairn Cloud and
  the local backup file.
