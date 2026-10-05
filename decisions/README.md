# Decision records

Short records of decisions that shape more than one repo or are expensive to reverse. Each one
says what was decided, why, and what follows from it.

| # | Decision | Status |
|---|---|---|
| [0001](0001-multi-repo.md) | Separate repositories with independent pipelines | Accepted |
| [0002](0002-stack.md) | Technology stack | Accepted |
| [0003](0003-local-first-encrypted-storage.md) | Local-first journal with user-chosen, end-to-end encrypted storage | Accepted |
| [0004](0004-location-points.md) | Location points with user-chosen permission levels | Accepted |

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
