# 0009. Location capture: no "Always" on Android; iOS confirmed on a real iPhone later

- **Status:** Accepted
- **Date:** 2026-10-10
- **Amends:** [ADR 0004](0004-location-points.md), Android permission levels

## Context

[ADR 0004](0004-location-points.md) offers three permission levels (Always, While using, Off) and
left one question open: on Android, does a quick-settings tile tap count as "while using"? If it
does, Cairn can skip "Allow all the time", which means sending the user to Settings and filing
Google Play's background-location declaration.

The test build is on the `spike/location-capture` branch of cairn-app
(`lib/spikes/location_capture/`, plus native code under `android/` and `ios/`). Every capture is
written to a log with its trigger, permission level, app state, result and time taken.

## Evidence

**Android emulator (API 37),** scripted and then tapped by hand:

| Permission | Capture inside Cairn | Tile, reading location directly | Tile, then a short foreground service |
|---|---|---|---|
| Off | denied | denied | not started |
| While using | fix in about 4 s | **no location** (Android returns nothing within 50 ms) | **fix in 4–5 s**, with Cairn in the background or not running |
| Always | fix in about 4 s | by hand, a fix after 18.7 s; scripted, only a 75–129 s old location after the 20 s timeout | not tried |

**iOS simulator:**

- **Captures inside Cairn,** on screen or in the background: a fix at Always and at While using.
  Denied when location is off.
- **Widget, Control Center button and Shortcut, tapped by hand:** a fix at Always and at While
  using. The widget and the Control Center button run in the widget extension's own process. The
  Shortcut ran in Cairn in the background at Always, and in the extension at While using.
- **The simulator is more lenient than a real iPhone.** It can't show what a locked phone allows,
  and it has no Action button.

## Decision

**Android:**

- **Offer only While using and Off.** Cairn never asks for "Allow all the time".
- **A tile tap starts a short location foreground service** that takes one fix and stops. It
  goes straight to the service, without trying a direct read first, since that never works in
  the background.
- While the service runs, Android shows a notification for a few seconds. Its text is
  "Saving a point…".

**iOS:**

- **Keep ADR 0004's plan for now:** ask for While using first, then offer the upgrade to Always.
- **Real-iPhone testing is parked** until there is an Apple Developer account. It covers the
  Action button, captures while the phone is locked, and whether While using is enough there as
  it was on the simulator. If it is, iOS drops Always too, in a new record.

## Consequences

- **Android needs no background-location permission and no Play background-location
  declaration.** The app still declares a location foreground service, and Google Play asks for a
  short justification of that.
- **The explanation page differs by platform:** Android shows two choices, iOS shows three.
- **A tile capture shows a brief notification.** Android requires it, and it is visible proof of
  what Cairn is doing, which fits ADR 0004's trust goals.
- **Still to check on real phones:**
  - that Android phone makers' battery savers don't block the foreground service;
  - everything on iOS listed above.
- **Not yet tested:** an Android home-screen widget. It should be able to use the same
  foreground service, but that needs its own test.
