# Femto Framework — User Guide

Slide-deck documentation on how to configure and run the PWGCF/Femto analysis framework
(O2Physics). One of three sibling repos:

- `femto-docs-code` — code-level documentation (architecture, class reference)
- `femto-docs-userguide` — **this repo**: how to configure and run the framework
- `femto-docs-deriveddata` — the derived data format: tables, columns, versioning

## Editing

The deck lives in [`userguide.md`](userguide.md), written in CodiMD slide syntax
(reveal.js, enabled via the `slideOptions` key in the YAML front matter):

- `---` on its own line starts a new horizontal slide
- `----` (four dashes) starts a vertical sub-slide of the slide above it

## Viewing as slides on CERN CodiMD

1. Push this repo to GitHub.
2. Create a GitHub Gist containing the raw contents of `userguide.md` (or link straight to
   the raw file URL on GitHub).
3. On CERN's CodiMD instance, use "Import from URL" (or paste the raw content into a new
   note) with that link — the `slideOptions` front matter switches the note into slide mode
   automatically when viewed in presentation mode.

Keep the front matter at the very top of the file intact when copying — CodiMD only
recognizes it as metadata (rather than as the first slide) if it's the first thing in the
document.
