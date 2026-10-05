# 0006. Scaffolding choices

- **Status:** Accepted (amends [ADR 0002](0002-stack.md) for Fly.io); OpenTofu replaced by Terraform in [ADR 0007](0007-terraform.md)
- **Date:** 2026-10-05

## Context

Setting up the three code repos surfaced a few decisions that the stack ADR didn't cover. One also
corrects it: Fly.io's Terraform provider (`fly-apps/terraform-provider-fly`) was archived in
November 2023, so OpenTofu can't manage the Fly apps as planned.

## Decision

### App (`cairn-app`)

- **App ID:** `com.cairnjournal.app` on iOS and Android. It can change freely until the first upload
  to App Store Connect or Google Play.
- **Packages:** Flutter 3.47 / Dart 3.13, with `flutter_riverpod` for state and `go_router` for
  navigation.
- **Analysis:** strict (`strict-casts`, `strict-inference`, `strict-raw-types`).
- **Colour tokens from ADR 0005 live in code** (`lib/design/tokens.dart`). A test checks that every
  text colour meets 4.5:1 contrast on its paper colour in the light, night and headlamp palettes.

### API (`cairn-api`)

- **Tooling:** Python 3.14 managed by **uv**; ruff for lint and format; mypy in strict mode; pytest.
- **Package layout:** `src/cairn_api`.
- **Migrations:** a small built-in runner instead of a migration framework. It applies plain-SQL
  files in `migrations/` once each, each in its own transaction, recorded in `schema_migrations`,
  with an advisory lock against concurrent deploys. It only moves forward. On Fly.io it runs as the
  release command.
- **Health endpoints:** `/healthz` (process up) and `/readyz` (database reachable).
- **Pipelines:**
  - every pull request runs lint, types, and tests against a real Postgres;
  - a `v*` tag publishes `openapi.json` as a GitHub release asset, for cairn-app to pin.

### Infrastructure (`cairn-infra`)

- **OpenTofu** manages the Cloudflare R2 buckets (`cairn-<env>-journal`, `cairn-<env>-shares`) and the
  Neon Postgres project for each environment, with state in an R2 bucket.
- **Fly.io apps are managed with flyctl, not OpenTofu.** This amends ADR 0002.
  `scripts/sync-fly-secrets.sh` creates the app if it's missing and copies OpenTofu outputs
  (database URL, bucket names) into Fly secrets, so infrastructure values still reach the API only
  as configuration.

## Consequences

- **Fly app configuration lives in `cairn-api/fly.toml`.** App creation and secrets go through the
  flyctl script rather than a reviewed `tofu plan`. Keep that script small, and run it only after
  `tofu apply`.
- **Deploy pipelines don't exist yet.** These wait until the Fly, Cloudflare and Neon accounts and
  tokens exist:
  - Fly deploys (staging on merge, prod on tag);
  - `tofu plan` on pull requests and `tofu apply` on merge.
- **DNS is not managed yet.** It's added once Cairn has a domain.
