# Moodboard

The source of the Cairn moodboard canvas (a private claude.ai Design artifact):
https://claude.ai/artifact/XJqWbFESLYGSQU2EiBZz4U

## Pages

**Moodboard: the current design.**

| Board | File |
|---|---|
| Shelf | `project/Shelf.dc.html` |
| Day 47 page | `project/DayPage.dc.html` |
| Headlamp mode | `project/Headlamp.dc.html` |
| Location points setup | `project/Location.dc.html` |
| App icon | `project/AppIcon.dc.html` |
| Palette and type | `project/Main.dc.html` |
| Map legend | `project/Icons.dc.html` |

**Scrapped: options that were considered and not chosen.**

- App icon rounds: `AppIconOptions*`, `AppIconRidge`, `AppIconHybrid`
- Spring icon options: `SpringOptions*`
- Shelf layouts: `ShelfOptions`
- Cover faces: `CoverFaceOptions`

## Files

- `project/canvas.json` is the canvas index: board positions, pages and sticky notes.
- Each `*.dc.html` file is one artboard: self-contained HTML with inline styles.

To update the canvas, edit these files and republish them to the same artifact URL. Edits made
directly on the canvas should be pulled back into this folder before committing.

The decisions this canvas led to are recorded in
[ADR 0005](../../decisions/0005-visual-identity.md).
