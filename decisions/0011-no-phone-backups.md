# 0011. Phone backups never carry the journal

- **Status:** Accepted; built on cairn-app branch `fix/backups-and-key-sync` and parked until
  sync and restore (weeks 11–13, see the roadmap)
- **Date:** 2026-10-29
- **Amends:** [ADR 0010](0010-data-layer.md), "OS backups include the journal"

## Context

ADR 0010 put the encrypted journal into iCloud and Google backups, so a reset phone could get it
back. Mihir has since settled what the storage choices mean:

- **"This device only" means no cloud at all.** People pick it so that nothing goes to Apple or
  Google, encrypted or not.
- **Cloud users already have a restore path:** Cairn Cloud now, and their own iCloud or Google
  Drive in v1.5. A second copy in the phone's backup adds nothing they can't already restore.

Keeping the journal out of phone backups for everyone gives one fixed rule. The app never has to
move the journal in or out of backup when someone changes their storage choice.

## Decision

- **The journal folder (database and photos) is never in a phone backup,** whatever the storage
  choice:
  - **iOS:** the folder is marked excluded from iCloud and computer backups.
  - **Android:** the backup rules exclude it from both cloud backup and phone-to-phone transfer.
- **How a journal comes back:**

| Storage choice | After a reset or on a new phone |
|---|---|
| Cairn Cloud | Restore from Cairn Cloud; the key comes back as [ADR 0012](0012-key-on-a-new-phone.md) describes |
| Own cloud (v1.5) | Restore from the person's iCloud or Google Drive, the same way |
| This device only | Import a local backup file the person made (to-do, before the beta) |

- **Small settings that are not journal content** may still use the phone's backup.

## Consequences

- **Until the local backup file exists, resetting a phone loses a device-only journal.** That's
  acceptable while only testers use the app. **The local backup file must ship before the beta.**
- **"Safe as of…" has to be honest for device-only users:** it shows the date of their last
  local backup file.
- **ADR 0010's restore table no longer applies.** The parts about the recovery key and the
  account master key still do, with the changes in ADR 0012.
