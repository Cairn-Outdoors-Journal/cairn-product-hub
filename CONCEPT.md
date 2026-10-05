# Cairn concept

> The product as agreed in planning (October 2026). Changes that cross repos are recorded in
> [decisions/](decisions/).

## The gap

Every major outdoor app is built around something other than memory:

| App | Built around |
|---|---|
| Strava | Performance: pace, segments, kudos |
| AllTrails | Finding trails: reviews, discovery |
| Gaia GPS, CalTopo, FarOut | Navigation |
| Day One | Journaling, with no knowledge of trails |
| Polarsteps | Travel journals; the closest analog, but built for trips abroad |

None of them records what a hike was like: the smell after rain at the treeline, who you were
with, the creek that was knee-deep. **Cairn is the expedition notebook.** Stats stay in the margins.

## Who it's for

**Multi-day hikers and thru-hikers first.** They already keep journals (Trailjournals, blogs, paper
notebooks) and are poorly served by apps. Each trip gets its own notebook, which suits them
exactly. Day hikers benefit from the same design: their trips go into a notebook for each year.

**The bar:** someone who has only ever carried a paper notebook tries Cairn and thinks, "I might
actually leave my notebook at home."

## Principles

1. **A journal, not a recorder.** Cairn has no GPS tracking. People already record with Strava,
   Garmin or a watch, and Cairn imports their GPX files.
2. **The physical feel comes from type, paper, layout and touch.** No fake leather, stitching or textures.
3. **Never fake handwriting.** Sketches and handwriting are real, drawn with a finger or Pencil.
4. **Your words are yours.** AI never writes in your journal. On-device help such as voice
   transcription is fine.
5. **Private by default.** Sharing means one page, not a feed.
6. **Works offline.** Trails have no signal.
7. **Words and photos lead.** Stats stay in the margins.
8. **Every helper can be switched off.** With everything off, Cairn is just paper and your words, and
   still feels complete.

## The notebook

### Bookshelf and notebooks

- Each trip is a notebook on a shelf, and day hikes go into a notebook for each year. The current
  notebook is shown large; the rest sit below as small covers facing out, each with the ridge cairn
  pressed into it and its name and year underneath.
- Notebooks have a cloth cover (waxed-canvas tan, forest, rust, slate, ochre). Over time the cover
  shows wear (v2).
- **The cover colour can be changed at any time**, not only when the notebook is created: long-press a
  notebook on the Shelf (or open its cover), choose Change cover, pick a cloth. Any wear carries over
  to the new colour.
- **The app opens to today's page** with a book-opening animation: the cover lifts and the pages
  riffle to the ribbon. The full animation plays on the first open of the day; after that it takes
  under a second.

**Notebook anatomy:**

1. **Cover:** trail, trail name, dates.
2. **"If found, please return to…"**
3. **One page per day.**
4. **Town and zero-day variants.**
5. **Back-of-book index:** People, Places, Specimens, Stamps.
6. **Closing page:** totals written in the margins. The notebook then goes back on the shelf.

**Physical details that do a job:**

- A **ribbon** bookmark jumps to today.
- **Tab dividers** lead to the index.
- The **elastic band** opens when Face ID unlocks the journal.

### Pages

- **Pages are fixed-size and paged, not scrolled.** What you see is what prints.
- **Two layers:**
  - Writing flows down the page.
  - **Taped-in items** (photos, sketches, stamps, scans, pressed specimens) sit on top anywhere, can
    hang into the margin, and the writing wraps around them.
- **Photo styles and tape:**
  - Prints: instant film, white-border, contact print.
  - Tape: masking, washi, photo corners.
  - Each item gets a slight random tilt, which you can adjust, and lands with a soft haptic.
- **Taped photos are copies,** so deleting one from the camera roll never breaks a page.

### Templates

- **The default day template:**
  - **Stamped header:** Day N · date · trail mile from→to · distance · elevation gain · weather · camp.
  - **Sections:** The Day · High / Low · People · Trail Notes · Tomorrow.
- **Variants** for trail days, town and resupply days, and zero days.
- **Templates are fully editable:** add, remove, rename and reorder sections. A blank page is one tap away.
- **Three-line fallback** for tired nights: best moment, hardest moment, one thing to remember.
- **Headlamp mode:** red on black, for writing in the tent.

### Back pocket

The inbox for things captured on the trail: voice notes (transcribed on the phone), marks (water,
summit, view, wildlife, hazard), text scraps, people met, and the day's photos from the camera
roll. Capturing takes zero decisions.

**At camp, each item is dragged onto the page:**

| Pocket item | Becomes |
|---|---|
| Photo | Taped onto the page |
| Voice note | Editable text |
| Person | An entry in the People index |
| Mark | A symbol in the margin and a pin on the map |

"Lay it out for me" (v1.5) drafts the page in route order instead. Items you don't use stay
attached to the day. The pocket can be switched off.

### Quick capture

- **iPhone:** home-screen and lock-screen widget, and the **Action button** ("Finally, a good use for
  the Action button").
- **Watch app** (v1.5).
- **Android:** a quick-settings tile, and a Wear OS tile (v1.5).

### Location points

- **One location fix each time the user opens Cairn or captures something.** Never continuous tracking.
- **On by default,** with the user choosing **Always**, **While using** or **Off**
  (see [ADR 0004](decisions/0004-location-points.md)).
- **What the points give you:**
  - a dotted route and per-day elevation without a GPX file;
  - pins on the route when a GPX file is imported;
  - the location for weather.
- **Elevation** is read on the phone from terrain-elevation data.
- **Trust, every time:**
  - a plain-language explanation before any system prompt;
  - a small benchmark mark settles into the margin when a point is saved;
  - any point can be deleted;
  - "Pause location for today" in one tap.

### GPX and maps

- **GPX import:** share a file from Strava or AllTrails, or pick one. Cairn splits it by day and
  places captures on the route by time. In v1 the route is drawn as an ink sketch on the day page.
- **v2:** a **Swiss-style shaded-relief 3D map** in the journal's colours, with the route as an
  ink line, moments pinned along it, and a fly-through.

### Collections (v1.5–v2)

- **Stamps** for summits, parks and named trails, like the National Park passport.
- **Specimens log:** cut a plant out of a photo and tape it in like a pressed flower.
- **Shared trips:** each person writes their own pages in a group notebook.
- **Almanac:** a year-end summary written like a logbook.
- **Printed books:** a finished notebook printed and bound.

## Visual identity

**The logic of a topographic map, with a modern look.** Think modern technical outdoor gear, not
an antique map.

- **Colour:**
  - **Base:** crisp paper; ink black for writing; contour brown for routes and rules; water blue;
    vegetation green; **blaze orange** as the single accent (ribbon, today, summits).
  - **Night paper** is deep blue-black. **Headlamp mode** is red on black.
- **Type:**
  - a book serif for your writing (e.g. Newsreader or Literata);
  - a modern grotesk for labels;
  - a clean technical mono for stamped data.
  - **Map-lettering rule:** water names in blue italic, landforms in brown.
- **Icons:** a custom map-legend set (spring, summit, campsite, trailhead, junction, hazard). No
  stock icons.
- **Mark:** a cairn drawn in one line. The app icon is a cloth notebook with the cairn pressed in.
- **Motion:**
  - Page turns are a quick slide with a paper shadow, not a 3D curl.
  - Ink settles in.
  - Stamps and tape land with a haptic.
- **Not allowed:** gradients, glass effects, emoji, purple, floating rounded cards, sunset-mountain
  illustrations, chummy greetings, confetti, stock icon sets.

## Data, privacy and ownership

- **Local-first:** the journal lives on the phone and works fully offline. An account is needed
  only for sync, backup, shared trips and printing.
- **The user chooses where it's kept:** this device only, their own iCloud / Google Drive (v1.5),
  or Cairn Cloud. Sync is end-to-end encrypted and the choice can be changed at any time
  (see [ADR 0003](decisions/0003-local-first-encrypted-storage.md)).
- **Honest backup status:** for example, "Safe as of Kennedy Meadows, 3 days ago".
- **Export at any time:** PDF of the pages as they print, plus an open archive (Markdown, original
  photos, GPX). Your journal outlives Cairn.
- **Sharing is safe by default:**
  - shared pages hide exact campsites;
  - "share with a delay" for live trips;
  - locations of rare species are always hidden.
- **Promises:** no ads, no selling data, no training AI on journals, and deleting your account
  actually deletes it.

## Business model

**Pay for what costs Cairn money, never for your own words.** Writing is never held behind a
paywall, and nobody ever loses access to what they wrote.

| Plan | Price (draft) | Includes |
|---|---|---|
| Free | $0 | The full journal on the device or in your own cloud, sketching, pocket, stamps, export, sharing a page as an image |
| Cairn+ | ~$4.99/mo or ~$39/yr | Cairn Cloud, shared trips, 3D maps, weather, private share links |
| Trail Pass | ~$19 per notebook | Everything in Cairn+ for one notebook, kept permanently, plus a print discount |
| Lifetime | ~$129 | Everything, once |
| Printed book | from ~$45 | A bound copy of a notebook |

**Pricing is transparent:** every feature is labelled free or paid with the reason. An in-app
"receipt" itemises what Cairn pays for (storage, map data, weather, servers).

## Later ideas (not scheduled)

- **Return visits:** your past notes appear when you come back to a trail.
- **Practical margins:** gear used, what to bring next time.
- **Full handwriting on iPad.**
