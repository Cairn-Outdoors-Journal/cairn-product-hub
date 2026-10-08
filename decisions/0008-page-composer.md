# 0008. Page composer: a custom editor that wraps text while typing

- **Status:** Accepted
- **Date:** 2026-10-08

## Context

Cairn's pages are fixed-size, with writing that flows around photos and other items taped on at an
angle (see [CONCEPT.md](../CONCEPT.md)). Flutter's built-in text fields can only lay text out in a
plain rectangle. This was the biggest technical risk in v1, so the roadmap called for a test build
before anything else.

There were two possible outcomes:
- **Live wrap:** text flows around items while you type.
- **Fallback:** while you type, the section uses the full width, and when typing stops, the text
  re-flows around the items.

The test build is on the `spike/page-composer` branch of cairn-app (`lib/spikes/page_composer/`).

## Decision

**Build the page composer on live wrapping, with a custom editor.**

- **Layout (`WrapLayout`):**
  - Text is laid out one line at a time. Each line's height band is split into the segments the
    page's items leave free, and each segment takes as much text as fits.
  - Taped items are rotated rectangles plus a margin, clipped against each line band.
  - **An item near a page edge pushes text to its other side only.** That stops words landing in
    the gap beside the page edge or the sliver beside a tilted corner. Text flows on both sides only
    of an item with at least 100 px free on each side.
- **Editing (`WrappedEditor`):**
  - The editor connects straight to the platform keyboard.
  - It draws text, the caret, selections and the underline for in-progress text from the layout.
  - It handles tap, double-tap (word) and drag selection.
  - It also handles **key presses**, because Android keyboards send backspace as a key press, not
    as a text change: backspace, delete, arrows, Shift-selection and Enter. Deletions work on
    user-perceived characters, so an emoji is removed whole.
- **PDF (`exportPagePdf`):**
  - Lines are never re-wrapped for print. **Every character is drawn where the screen placed it**,
    with the same font file embedded.
  - The PDF library doesn't kern text on its own; positioning per character keeps printed lines
    identical to the screen.
- **Fonts are bundled in the app.** Newsreader (16pt cut, SIL Open Font License) is bundled now,
  so writing works offline and screen and PDF share one font file. The other typefaces get
  bundled when the interface needs them.

## Evidence

- **Tested by hand on the iOS simulator and the Android emulator:** typing, deleting, selecting,
  new lines, dragging and tilting the photo. Responsive with no visible lag on either.
- **Speed in debug builds** (debug mode is slower than what users get): layout takes
  about 1–2 ms per edit while typing, and up to about 9 ms per frame while dragging the photo. An
  edit reaches the screen in about 13 ms on the Android emulator.
- **42 automated tests,** covering:
  - every character placed once and in order, with no text over a photo;
  - tapping where the caret is drawn finds the same position;
  - 300 keystrokes in a row with none lost;
  - emoji-safe deletion and key-press handling;
  - screen and PDF line widths within 0.41 pt. The only drift is inside the "fi" ligature.

## Consequences

The real composer starts from this code, and has to add what the test build skipped:

- **Copy, cut and paste menu,** and the platform's selection handles and magnifier.
- **Accessibility.** VoiceOver and TalkBack need explicit text information
  from a custom editor. A built-in text field provides this for free, so it is a hard requirement
  before release, not a nice-to-have.
- **iOS keyboard extras:**
  - the space-bar trackpad (floating cursor);
  - the autocorrect underline;
  - Scribble (Apple Pencil handwriting).
- **Right-to-left and mixed-direction text** isn't supported yet.
- **Long pages:** re-lay out only the lines after an edit, rather than the whole page, if the
  profiling below shows it's needed.
- **Speed on real phones with an optimised build.** Simulators only run debug builds, so this has
  to be measured once there is a developer account and a test device.
