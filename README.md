# Cairn

**Your trails, remembered.**

Cairn is a field journal app for multi-day hikers, backpackers and thru-hikers. It's for writing
down what a trip was like, with photos taped onto the page, the route, and the people you met.
It is not a GPS tracker or a fitness app.

This repository is the **product hub**. It holds the concept, roadmap, architecture, decision records
and the specs other repos implement. No app or server code lives here.

## What's here

| Document | What it covers |
|---|---|
| [CONCEPT.md](CONCEPT.md) | The product: who it's for, principles, the notebook, design direction, privacy, business model |
| [ROADMAP.md](ROADMAP.md) | Releases planned around the 2027 hiking season, v1 scope, timeline, release checks |
| [ARCHITECTURE.md](ARCHITECTURE.md) | System diagram and what data crosses each boundary ([styled version](design/architecture.html)) |
| [decisions/](decisions/) | Architecture decision records (ADRs) |
| [specs/sync-format.md](specs/sync-format.md) | Sync and encryption format shared by the app and the API |
| [design/](design/) | Design sources: [architecture page](design/architecture.html) and the [moodboard](design/moodboard/) |

## Repositories

All repos live in the [Cairn-Outdoors-Journal](https://github.com/Cairn-Outdoors-Journal)
organization and have their own CI pipelines.

| Repo | Contents |
|---|---|
| [cairn-product-hub](https://github.com/Cairn-Outdoors-Journal/cairn-product-hub) | This product hub |
| [cairn-app](https://github.com/Cairn-Outdoors-Journal/cairn-app) | Flutter app for iOS and Android, with Swift and Kotlin extensions |
| [cairn-api](https://github.com/Cairn-Outdoors-Journal/cairn-api) | Python FastAPI service, Postgres migrations |
| [cairn-infra](https://github.com/Cairn-Outdoors-Journal/cairn-infra) | OpenTofu for Fly.io, Neon, Cloudflare R2 and DNS |

## Local setup

Clone all four repos side by side in one `cairn/` folder:

```sh
mkdir cairn && cd cairn
for r in cairn-product-hub cairn-app cairn-api cairn-infra; do
  git clone git@github.com:Cairn-Outdoors-Journal/$r.git
done
```

## How the repos stay in step

- **API contract:** `cairn-api` publishes a versioned `openapi.json` with each release. `cairn-app`
  generates its client from a pinned version and moves to a new one on purpose.
- **Sync format:** [specs/sync-format.md](specs/sync-format.md) defines what the app encrypts and
  what the API stores. Both sides test against its test vectors.
- **Decisions:** anything that affects more than one repo gets an ADR in [decisions/](decisions/).
