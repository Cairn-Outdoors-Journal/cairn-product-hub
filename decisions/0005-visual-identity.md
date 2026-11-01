# 0005. Visual identity, app icon and shelf

- **Status:** Accepted; cover colours and shelf details amended by [0013](0013-shelf-and-covers.md)
- **Date:** 2026-10-05

## Context

Cairn has to feel like a field journal without looking like a 2011 skeuomorphic app or a generic
AI-generated one. The first moodboard round settled the overall look; follow-up rounds chose the app
icon, the spring icon in the map legend, and how the Shelf shows notebooks. The working canvas is in
[design/moodboard/](../design/moodboard/), with rejected options kept on its Scrapped page.

## Decision

### Look

**The topographic map is the design system,** drawn with the restraint of modern technical gear.

**Colour tokens:**

| Token | Light | Night |
|---|---|---|
| Paper | `#F7F7F4` | `#11161A` |
| Ink | `#1C1D1A` | `#E6E8E2` |
| Contour | `#8A6A45` | `#B58F63` |
| Water | `#2F6690` | `#74AEDB` |
| Vegetation | `#4F6B4A` | — |
| Blaze (accent) | `#E2531B`, text `#B8430F` | `#FF7A3D` |
| Rule | `#D9DAD3` | — |
| Pencil (captions) | `#5E615A` | `#9AA19B` |

- **Headlamp mode:** `#0A0606` ground, `#E0503F` text, `#4A1A14` rules.
- **Cover cloths:** waxed canvas `#B59A6E`, forest `#2F4A3A`, rust `#A2492B`, slate `#46525C`,
  ochre `#C49A2C`.

**Type:**

- **Newsreader** for the user's writing.
- **Schibsted Grotesk** for labels and interface.
- **Martian Mono** for stamped data.
- **Map-lettering rule:** water names in blue italic, landforms in brown.

### App icon

- **The drawing:**
  - an ink background with a darker book spine down the left edge;
  - a ridge line over filled slate ground (`#3A464F`);
  - a cairn of three stones on the peak, the top stone blaze orange;
  - a small blaze ribbon hanging from the top edge (7 × 20 units on a 100-unit grid).
- **The source** is `design/moodboard/project/AppIcon.dc.html`.
- **The same icon** is used beside the wordmark in headers.

### Map legend

- Twelve custom icons follow survey-map colour conventions: water blue, terrain brown, hazards
  blaze, the rest ink.
- The **spring** icon is a ringed wave (a wave inside a circle), chosen because it reads at the 16px
  margin size.

### Shelf

- **The current notebook** is shown large, opening to today's page.
- **Every other notebook** sits below as a small cover facing out, four per row, with the ridge
  cairn pressed into its face and its name and year underneath. No text is printed on the cover
  itself.
- **A notebook's cover colour can be changed at any time** (long-press → Change cover). Wear carries
  over to the new colour.
- **Creating a notebook** is a small "+ New" control in the header, not a large button.

## Alternatives considered

- **App icon:** 19 alternatives across stacked slabs, contour rings, a trail blaze, a survey mark, a
  monogram, the notebook itself, and ribbon placements.
- **Spring icon:** 12 alternatives.
- **Shelf:** spines with varied heights, uniform spines, two meaningful sizes, a flat stack, symbols
  on spines.
- **Cover faces:** title, the trip's route, a paper label, a taped photo, route plus title.

All are kept on the moodboard's Scrapped page for reference.

## Consequences

- **Android adaptive icons crop to a mask** (circle or squircle, with roughly the middle 66% safe).
  The spine and ribbon sit at the icon's edges, so Android needs a variant with both moved inside
  the safe zone. Check this when the icons are exported.
- **The ribbon is the brand's "today" signal** (icon, Day page, Shelf cover). Keep blaze orange
  reserved for it, the top stone and other single accents.
- **Face-out covers take more vertical space than spines.** With many notebooks, the Shelf needs to
  scroll or group by year. Decide that when the screen is built.
