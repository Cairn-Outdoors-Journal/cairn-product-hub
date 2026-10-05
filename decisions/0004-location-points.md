# 0004. Location points with user-chosen permission levels

- **Status:** Accepted
- **Date:** 2026-10-05

## Context

Cairn deliberately has no GPS recorder. Without some location data, though, users without a GPX
file get no route, no elevation in the page header, and no weather for where they actually were.
A precise log of where someone was every day is also the most sensitive data in the app, so it has
to be handled in a way that earns trust every time.

## Decision

**What is recorded:**

- **One location fix each time the user opens Cairn or captures something:** Action button,
  widget, tile or pocket. **There is never continuous tracking,** whatever the permission level.
- **On by default,** after a plain-language explanation.

**The user chooses the permission level** on Cairn's own explanation page, before any system prompt:

| Choice | Points recorded | What's missing |
|---|---|---|
| **Always** | Every app open and every capture, including captures while the phone stays locked | Nothing |
| **While using** | App opens, and captures that open the app | Locked-phone captures use a last-known location under ~2 min old, or have none |
| **Off / denied** | None | Dotted route, automatic elevation, weather for exact spots |

- **The "Always" note reads:** *"So the Action button and widget can record where you are while your
  phone stays in your pocket. Cairn still records a point only when you press something, never
  continuously."*
- **iOS:** ask for "While using" first, then the upgrade to "Always" if the user chose it. The
  system usage description matches the in-app note.
- **Android:** "Allow all the time" requires sending the user to Settings, plus Google Play's
  background-location declaration. Skip it if testing shows that a tile or widget tap counts as
  "while using".

**How capture works:**

- **Capture never waits for GPS.** The item saves instantly and the fix is attached when it
  arrives (about a 30 s timeout).
- **App opens skip duplicates:** a new point is saved only if the user has moved more than about
  50 m or 10 minutes have passed.
- **Raw readings are stored:** time, latitude and longitude, accuracy, GPS altitude, barometric
  pressure, what triggered the point, and links to the pocket item and page.

**Elevation and weather:**

- **Elevation is read on the phone** from zoom-12 terrain tiles. The tile server learns only which
  ~10 km area was requested. The barometer refines readings between points.
- **Weather** comes from Open-Meteo, called directly from the phone with a location rounded to
  about 1 km plus the exact elevation. That is one lookup per page or every few hours.

**Trust every time:**

- When a point is saved, a small benchmark mark settles into the page margin.
- A "Trail marks" list on each day lets the user view or delete any point.
- "Pause location for today" takes one tap.
- There is a setting per notebook, and "Plain notebook" turns location off.

**Points are journal content:** they are encrypted like pages ([ADR 0003](0003-local-first-encrypted-storage.md))
and removed from shared pages.

## Consequences

- **With location off, the journal still works:**
  - GPX import places captures on the route by time;
  - photos keep their camera locations;
  - a place name typed into the page header drives weather;
  - pages say "no location recorded", with no nagging.
- **A test build in weeks 2–3** confirms what iOS (widget, Action button App Intent) and Android
  (tile) allow at each permission level.
- **App Store and Google Play review** of "Always" / background location needs written
  justification and, on Android, a demo video. Plan time for it before the April launch.
