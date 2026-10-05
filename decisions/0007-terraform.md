# 0007. Terraform instead of OpenTofu

- **Status:** Accepted (amends [ADR 0001](0001-multi-repo.md), [ADR 0002](0002-stack.md) and
  [ADR 0006](0006-scaffolding-choices.md))
- **Date:** 2026-10-05

## Context

ADR 0002 chose OpenTofu for cairn-infra. Mihir, who reviews and runs the infrastructure, already
works with Terraform. For a one-person project, a tool the reviewer knows well matters more than the
differences between the two.

## Decision

- **cairn-infra uses Terraform** (1.16, installed with `brew install hashicorp/tap/terraform`) in
  place of OpenTofu.
- **The configuration is unchanged HCL.** Only these change:
  - the CLI;
  - the provider lock files, which now point at `registry.terraform.io`;
  - the CI setup action (`hashicorp/setup-terraform`);
  - the state bucket name (`cairn-terraform-state`).
- **Everything else in ADR 0006 still holds:**
  - Cloudflare R2 and Neon are managed in code;
  - Fly.io apps are managed with flyctl, because Fly.io's Terraform provider is archived;
  - deploy pipelines wait until the accounts and tokens exist.

## Consequences

- **Terraform's licence** (Business Source License) allows this use: Cairn runs it to manage its own
  infrastructure and doesn't offer Terraform as a service.
- **Where earlier records say `tofu plan` and `tofu apply`,** read `terraform plan` and
  `terraform apply`.
