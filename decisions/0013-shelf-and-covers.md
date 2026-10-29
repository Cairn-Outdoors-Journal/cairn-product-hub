# 0013. The shelf and covers

- **Status:** Accepted
- **Date:** 2026-10-29
- **Amends:** [ADR 0005](0005-visual-identity.md), "five cover cloths"

## Context

Building the shelf (stage 7a, app PR #4) settled several things ADR 0005 left open, and changed
one it decided.

## Decision

- **Covers:**
  - **Six cloths:** the five in ADR 0005, plus **Lake** `#2E5A73`, because the set had no blue.
  - **Plus a custom colour,** picked with a hue and saturation wheel, a brightness bar or a hex
    code.
  - Spine, band and pressed mark are worked out from the cover colour.
  - **Lettering is ink or paper, whichever has more contrast with the cover,** so every cover
    reads at 3:1 or better. The six cloths keep the lettering they were designed with.
- **The current notebook** (shown large under "On the trail") is the one opened last on this
  phone. It's stored per phone and never synced. Long-press, then Make current, overrides it.
- **Notebooks are trips or yearly notebooks.** A trip records its trail, the hiker's trail name
  and its first day, which numbers the days ("Day 47"). Mileage waits for GPX import.
- **The header** holds the app mark and "+ New", with no title or divider. "+ New" is hidden
  while the shelf is empty.
- **Empty places:** an empty shelf shows dashed outlines of what's to come, one large and a row
  of four. Notebooks take those places as they're added, and empty places always finish the last
  row. Tapping one starts a new notebook.

## Consequences

- The moodboard shows these: the Shelf, Empty shelf and Cover colour boards, and Lake on the
  palette.
- Custom colours are stored as `coverColor` (0xRRGGBB), with the cloth kept as a fallback.
