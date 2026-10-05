# 0001. Separate repositories with independent pipelines

- **Status:** Accepted
- **Date:** 2026-10-05
- **Note:** the hub repo was renamed from `cairn` to `cairn-product-hub` on 2026-10-05, and local
  clones live together in one `cairn/` folder.

## Context

Cairn has three very different kinds of code:

- **A mobile app**, whose release pipeline is slow and gated by app-store review.
- **A Python API**, which deploys continuously.
- **Infrastructure code**, whose changes need their own review flow.

In a single repository, every change waits on the slowest pipeline. As more people join, that
pipeline becomes the bottleneck.

## Decision

Use four repositories in the `Cairn-Outdoors-Journal` GitHub organization, all private:

| Repo | Owns | Pipeline |
|---|---|---|
| `cairn-product-hub` | Product hub: concept, roadmap, architecture, ADRs, specs, design sources | Markdown lint |
| `cairn-app` | Flutter app plus Swift and Kotlin extensions | analyze + test on PR; tag → TestFlight and Play internal testing |
| `cairn-api` | FastAPI service and SQL migrations | test + lint on PR; merge → staging; tag → prod; each release publishes `openapi.json` |
| `cairn-infra` | OpenTofu for Fly.io, Neon, R2, DNS | plan on PR; apply on merge; prod apply needs manual approval |

Two contracts connect the repos:

1. **API:** `cairn-app` generates its client from an `openapi.json` version it pins. API paths are
   versioned (`/v1`), and changes within a version stay backward compatible.
2. **Sync format:** [specs/sync-format.md](../specs/sync-format.md) in this repo has a format
   version and test vectors. The app implements it and the API stores what it produces without
   reading it.

## Consequences

- **Easier:** each repo ships at its own pace, and access can be granted per repo.
- **Harder:**
  - A change that touches both the API and the app needs two pull requests in a set order: API
    first, released, then the app pins the new version.
  - A contract change needs an ADR or a spec version bump here first.
- **Infrastructure values** (hostnames, bucket names) reach the API as Fly secrets and config, never
  hard-coded.
