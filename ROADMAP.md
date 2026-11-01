# Cairn roadmap

> **The dates here are a guide, not deadlines.** Cairn is a side project built in spare time.
> Some weeks will move fast, and there may be days or weeks with no changes at all. Expect the
> timeline to drift, possibly a lot. What matters is the order of the work: the riskiest pieces
> come first, and each release stays focused on what hikers need at that point in the season.
> If v1 misses spring 2027, the same plan applies to the next season.

Releases follow the thru-hiking season. Hikers need a dependable daily journal when they start
(Appalachian Trail Feb–Mar, Pacific Crest Trail April). They want maps and printed books when
they finish (Aug–Oct).

| Release | Aiming for | Focus |
|---|---|---|
| **v1 "Trail"** | Beta with ~30 thru-hikers mid-Feb 2027 → public early April 2027 | The everyday journal, capture on the move, never losing data |
| **v1.5 "Town"** | May–Jul 2027 | Own-cloud storage, "lay it out for me", watch apps, shared trips, stamps, specimens |
| **v2 "Home"** | Aug–Oct 2027 | 3D relief map and fly-through, printed books, closing a notebook, Almanac, cover wear, Lifetime |

## v1 "Trail" scope

- **Notebooks:**
  - trip and yearly notebooks;
  - cover cloths;
  - bookshelf;
  - opens to today's page with the book-opening animation.
- **Pages:**
  - fixed-size pages and the editable day template (sections can be switched on or off, renamed, reordered);
  - blank page;
  - "Plain notebook" option;
  - three-line fallback.
- **Stamped header, offline:** day, date, location, sunrise and sunset, moon. Weather is filled in
  when there is signal.
- **Taped items:** photos with 2–3 print styles and 2–3 tape styles; writing wraps around them; sketch boxes.
- **Back pocket:** text, voice transcribed on the phone, camera-roll suggestions. Can be switched off.
- **Quick capture:** iOS widget and Action button; Android quick-settings tile.
- **Location points:** Always / While using / Off on iOS, While using / Off on Android ([ADR 0009](decisions/0009-location-capture-results.md)); trust UX; terrain-data elevation.
- **Look and feel:** headlamp, light and dark themes; Face ID lock; ribbon; Days and People tabs.
- **Storage:**
  - device-only, and Cairn Cloud with end-to-end encryption;
  - a local backup file that imports on any phone, iPhone or Android (the only backup for
    device-only journals, which stay out of phone backups: ADR 0011);
  - the key reaches a new phone through the synced keychain, a passkey or the recovery key
    (ADR 0012);
  - "safe as of…" backup status;
  - PDF and archive export.
- **Sharing:** a page as an image, with locations hidden by default.
- **GPX import:** the route as an ink sketch on the day page.
- **Payments:** Free, Cairn+, Trail Pass, with the pricing "receipt".

## v1.5 "Town"

- Own iCloud / Google Drive storage.
- "Lay it out for me".
- Watch app and Wear OS tile.
- Shared trips.
- Stamps and the specimens log (cutting plants out of photos).

## v2 "Home"

- Swiss shaded-relief 3D map with fly-through.
- Printed books through the Lulu Print API.
- Closing a notebook.
- Almanac.
- Cover wear.
- Lifetime plan.

## Timeline to v1 (a guide)

The dates show the intended order and rough size of each block of work. They are not commitments.

| Weeks | Dates | Work |
|---|---|---|
| 1 | Oct 5–11 | Product hub docs; visual moodboard; scaffold `cairn-app`, `cairn-api` and `cairn-infra`, each with CI; design tokens |
| 2–3 | Oct 12–25 | **Test build: text wrapping around taped photos while editing.** Decide between live wrap and re-flow when typing stops. Test build: location from widget, Action button and tile |
| 4–7 | Oct 26–Nov 22 | Data layer; bookshelf; notebook; page composer; templates; offline header; themes; opening animation |
| 8–10 | Nov 23–Dec 13 | Back pocket, voice, camera roll; widget, Action button, Android tile; location points; Face ID lock; GPX → ink route |
| 11–13 | Dec 14–Jan 10 | API; encrypted sync; restore on a new device; land the parked backups and key-sync branch; export; sharing as an image; payments; weather filled in later |
| 14–16 | Jan 11–Feb 7 | Hardening: offline, battery and data-loss testing; a real multi-day trip; recruiting testers |
| 17+ | Feb–Mar | Beta through TestFlight and Play testing with ~30 thru-hikers; fixes; store review → public launch early April |

**Biggest risk:** keeping editing reliable while writing wraps around taped items. It is tested in
weeks 2–3, before anything is built on top of it.

## Parked work

Built ahead of time, then parked so the screens come first (2026-11-01). It is picked up with
sync and restore in weeks 11–13, and must land before the beta.

- **Backups and key travel** (ADRs 0011, 0012), cairn-app branch `fix/backups-and-key-sync`
  (7 commits, 03633e7 to 9873b4a):
  - the journal kept out of iCloud and Google backups;
  - a copy of the key in iCloud Keychain (iOS) and Block Store (Android) for the next phone,
    with unit tests and a device test that pass on the simulator and emulator.
  - **Left to do:** rebase on main, re-run the tests and the device test, and test a real move to a
    new phone (needs real devices; Block Store's cloud restore needs a Play Store install).
  - **Until it lands,** `main` keeps ADR 0010's rule: the encrypted journal goes into phone
    backups. That is harmless before the beta.

## v1 release checks

- **Airplane mode:** 5 days of daily entries with no signal → reconnect → sync completes.
- **New phone:**
  - wipe the phone, then restore from Cairn Cloud with the key from the synced keychain and, on a
    phone of the other kind, from a passkey: every page, photo and location point is back;
  - the same from a local backup file, across iPhone and Android.
- **Export:** the PDF and the archive both open correctly.
- **Location:** each permission level the platform offers (iOS: Always, While using, Off; Android:
  While using, Off) behaves as documented in [ADR 0004](decisions/0004-location-points.md) and
  [ADR 0009](decisions/0009-location-capture-results.md).
- **Real trip:** at least one real overnight trip with the app.
