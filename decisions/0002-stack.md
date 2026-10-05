# 0002. Technology stack

- **Status:** Accepted; Fly.io management amended by [ADR 0006](0006-scaffolding-choices.md)
- **Date:** 2026-10-05

## Context

Cairn ships on iOS and Android together for the spring 2027 thru-hike season. Its interface is
entirely custom: fixed-size paper pages, writing that wraps around tilted taped photos, page
turns, and later a 3D relief map. The same page layout must also produce print-ready PDFs.

Claude writes the code and Mihir reviews it. Mihir is a backend engineer who works in Python and SQL.

## Decision

**App: Flutter (Dart).**

- Flutter draws every pixel itself, so pages look identical on both platforms and the PDF can come
  from the same layout code.
- Main packages: `drift` (SQLite), `riverpod`, `go_router`, `sodium_libs`, `photo_manager`,
  `local_auth`, `home_widget`, `purchases_flutter`, `pdf`/`printing`, `gpx`, and later `maplibre_gl`.

**Native extensions, connected through platform channels:**

- **Swift:** WidgetKit, App Intents (Action button), Speech, Core Location. Later: VisionKit, watchOS.
- **Kotlin:** quick-settings tile, SpeechRecognizer, location. Later: ML Kit, Wear OS.

**API:**

- Python **FastAPI**, running as a container on **Fly.io**.
- **Postgres on Neon**, with plain-SQL migrations.
- **Cloudflare R2** for encrypted blobs (no egress fees). The phone uploads directly through
  pre-signed URLs.
- Sign in with Apple and Google.

**Infrastructure:**

- **OpenTofu**, with remote state in R2.
- Two environments: `staging` and `prod`.

**External services:**

| Service | Used for | From |
|---|---|---|
| RevenueCat | In-app purchases and subscriptions | v1 |
| Open-Meteo | Historical weather, filled in later | v1 |
| Terrain tiles | Elevation data | v1 |
| MapLibre and self-hosted map tiles | 3D map | v2 |
| Lulu Print API | Printed books | v2 |

**Sun and moon** times are calculated on the phone.

## Alternatives considered

- **React Native / Expo:** TypeScript and a large ecosystem, but custom page rendering goes through
  Skia as an add-on rather than being the core of the framework.
- **Native iOS first, Android later:** the best iPhone experience, and iOS has text wrapping
  around shapes built in. Rejected because Android would miss the 2027 season.
- **Supabase Edge Functions (TypeScript) for the backend:** rejected in favour of Python, which the
  reviewer knows best.

## Consequences

- **The page composer is the biggest technical risk.** Editing text while it wraps around shapes
  is hard in Flutter, so it gets a test build before anything else. If it isn't reliable, the
  fallback is: while typing, the section uses the full width; when typing stops, the text re-flows
  around the taped items.
- **Native code is still needed** for widgets, the Action button, the tile and watch apps, so the
  app repo contains Swift and Kotlin alongside Dart.
- **The backend stays small:** it coordinates sync and accounts but never touches journal bytes.
