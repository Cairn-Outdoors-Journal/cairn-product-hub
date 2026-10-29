# Decision records

Short records of decisions that shape more than one repo or are expensive to reverse. Each one
says what was decided, why, and what follows from it.

| # | Decision | Status |
|---|---|---|
| [0001](0001-multi-repo.md) | Separate repositories with independent pipelines | Accepted |
| [0002](0002-stack.md) | Technology stack | Accepted |
| [0003](0003-local-first-encrypted-storage.md) | Local-first journal with user-chosen, end-to-end encrypted storage | Accepted; extended by 0010 |
| [0004](0004-location-points.md) | Location points with user-chosen permission levels | Accepted; amended by 0009 |
| [0005](0005-visual-identity.md) | Visual identity, app icon and shelf | Accepted; amended by 0013 |
| [0006](0006-scaffolding-choices.md) | Scaffolding choices; Fly.io managed with flyctl | Accepted |
| [0007](0007-terraform.md) | Terraform instead of OpenTofu | Accepted |
| [0008](0008-page-composer.md) | Page composer: a custom editor that wraps text while typing | Accepted |
| [0009](0009-location-capture-results.md) | Location capture: no "Always" on Android; iOS confirmed on a real iPhone later | Accepted |
| [0010](0010-data-layer.md) | Data layer: an encrypted journal on the phone; Android 12 minimum | Accepted; amended by 0011, 0012 |
| [0011](0011-no-phone-backups.md) | Phone backups never carry the journal | Accepted |
| [0012](0012-key-on-a-new-phone.md) | Getting the key onto a new phone: synced keychain, passkey, recovery key | Accepted |
| [0013](0013-shelf-and-covers.md) | The shelf and covers: Lake, custom colours, empty places | Accepted |

## Writing a new record

Copy the shape below into `NNNN-short-title.md` with the next number and add it to the table above.
To change a decision, write a new record that supersedes the old one, and set the old one's status
to `Superseded by NNNN`. Never rewrite an accepted record.

```markdown
# NNNN. Title

- **Status:** Proposed | Accepted | Superseded by NNNN
- **Date:** YYYY-MM-DD

## Context
What forces are at play and why a decision is needed.

## Decision
What we decided, stated plainly.

## Consequences
What becomes easier, what becomes harder, and what we now have to do.
```
