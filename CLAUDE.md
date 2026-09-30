# CLAUDE.md — working on Greenroom

Guidance for Claude Code (or any developer/agent) iterating on this project. Read this fully before editing `greenroom.html`.

## What this is

Greenroom is a **single self-contained HTML file** (`greenroom.html`) containing a slide-deck designer and presenter. There is no build step, no package.json, no dependencies, and no server. The hard product constraints, in order:

1. **It must keep working when opened directly from `file://` on a machine with no internet.** Never add CDN links, fetch calls, external assets, or anything requiring a server. Fonts are already base64-embedded.
2. **One file.** All CSS, JS, fonts, and the favicon live inside `greenroom.html`. New assets must be embedded (base64/data URI).
3. **Old decks must keep loading.** `validateDeck()` fills defaults for every field; when you add a schema field, add its default there *and* in `newDeck()`/`newSlide()`. Never rename or repurpose existing JSON fields.

## File map (top to bottom of greenroom.html)

- `<head>`: title, SVG favicon (data URI), `<style id="embedded-fonts">` (base64 Poppins/Figtree — ~115KB, do not touch), main `<style>` (reset → stage keyframes/classes → studio/designer UI → modals → present chrome/HUD/laser/ink → print/handout).
- `<script>` sections, in order:
  - `APP_VERSION`, `BAKED`/`PLAYER` detection (player files carry a `#baked-deck` JSON tag in `<head>`)
  - utilities (`$`, `esc`, `rich`, `an`, `debounce`)
  - **themes**: `genAmbient()`, `THEMES` registry, color utils (`hexRgb`, `mixHex`, `lum`), `customThemeObj()`, `theme(deck)`
  - **layouts**: field helpers (`F`, `FA`, `FS`, `FC`, `FI`, `FL`), `LAYOUTS` registry, `eyebrowBar/Wrap`, `renderStage()`
  - deck model: `newSlide`, `newDeck`, `validateDeck`
  - `DB` (IndexedDB wrapper — db `greenroom` v2, stores `decks` and `snaps`)
  - state, undo/redo history, `touch()`/`persist()`
  - designer: `renderDesign`, slide list, properties panel (`renderProps`, `fieldHTML`, bindings), preview, deck settings
  - modals: theme picker, sorter, search, layout picker (`layoutPicker`, `mapContent`), JSON, guide (`GUIDE_TABS`), AI prompt (`buildHelperPrompt`), decks/snapshots, import/export, `exportPlayer`
  - **present mode**: `visIdx`/`navStep`, `renderPresent`, autoplay, HUD, laser/ink, grid/jump/blackout/rehearsal, `renderChrome`, `goTo`, transitions
  - print/handout, `switchMode` + hash routing, global event handlers, `sampleDeck()`, `migrateLegacy()`, `init`

## The two registries do most of the work

**Adding a slide layout** = one entry in `LAYOUTS`: `{ name, icon, desc, fields, defaults, render(content, eyebrow) }`. The designer form, visual pickers, AI-prompt schema docs, and cross-layout content mapping are all generated from `fields`. Rules for `render()`:
- Output inline-styled HTML for a **1920×1080 stage**; sizes are absolute px at that scale (titles ~100–150px, body 30–60px).
- Colors **only** via theme variables: `var(--ink)`, `var(--acc)`, `var(--acc2)`, `rgba(var(--acc-rgb),.x)`, and **card surfaces via `rgba(var(--card-rgb),.03–.05)`** — never hardcoded white/black, or the light theme breaks.
- Entrance animations via `an('rise'|'fade'|'growX'|'zoomIn'|'slideL', dur, delay)` with staggered delays; base styles must equal the final state (animations use `both` fill; `.static` disables them for thumbnails/print).
- Escape all user content with `rich()` (markup + escaping) or `esc()` (plain).
- Honor the `eyebrow` parameter (see `eyebrowWrap`) so section grouping works.

**Adding a theme** = one entry in `THEMES` with the full `vars` set (`--ink`, `--ink-rgb`, `--acc`, `--acc-rgb`, `--accB`, `--acc2`, `--acc2-rgb`, `--acc2Soft`, `--deep`, `--card-rgb`, `--chromebg`, `--railbg`, `--panelbg`), `page`, `bg`, `ambient(style)`. `ambient` must accept the deck's `settings.ambientStyle` (`'calm'` | `'particles'` | `'particlesSoft'` | `'aurora'`, listed in `AMBIENT_STYLES`) and pass it to `genAmbient()`, which dispatches to `genMotes()` / `genAurora()`; particle fields use the seeded PRNG so every render is identical. For light themes set `--card-rgb` to the ink RGB (dark-tinted cards). Custom user themes are derived in `customThemeObj()` — extend it if you add vars.

## Invariants and hard-won gotchas

- **Encoding**: the file is UTF-8 **with BOM**. On Windows, PowerShell 5.1 `Get-Content` without `-Encoding` reads BOM-less UTF-8 as CP1252 and will corrupt every non-ASCII character on rewrite (this happened once; the BOM now prevents it). If you script against the file, always pass explicit UTF-8 encodings.
- **Never write a literal `</script>` sequence inside JS strings** — it terminates the inline script tag when the HTML is parsed, and it breaks `exportPlayer()`'s self-serialization. Build it by concatenation (`'</scr' + 'ipt>'`), as the existing code does. Be similarly thoughtful about other closing tags in strings (`exportPlayer` inserts at the *first* `</head>`, which precedes all script text).
- **`exportPlayer()` serializes the live DOM** (`documentElement.cloneNode`) after emptying `#app`. Anything you add outside `#app` at runtime will leak into published player files — keep runtime DOM inside `#app`, or strip it in `exportPlayer`.
- **PLAYER mode**: when `PLAYER` is true the app is locked to present mode (no designer, no Esc-out, hash routing disabled). Gate any new "exit to designer" affordance behind `!PLAYER`.
- **Hidden slides**: never navigate with raw indices in present mode — always go through `visIdx()` / `navStep()`. Dots, rail, counter, autoplay, HUD-next, Skip-to-end, and print all filter hidden slides.
- **Undo history**: every mutation path must call `touch()` (it snapshots for undo, debounce-groups typing bursts, and autosaves). When an action *replaces* the deck wholesale (open/new/import-as-new), call `resetHistory()` after, so undo can't cross decks. JSON-apply and snapshot-restore intentionally do NOT reset — they're undoable edits.
- **Storage must be optional**: IndexedDB/localStorage calls are wrapped in try/catch and the app runs fine when they throw (sandboxed iframes, previews). Preserve that. IDB schema changes require bumping the version in `indexedDB.open('greenroom', N)` with an `onupgradeneeded` that creates missing stores.
- **Form re-render discipline**: text inputs update state on `input` without re-rendering the form (re-rendering loses focus); structural changes (list add/remove, layout change) re-render via `renderProps()`. Preview refreshes are debounced (`previewSoon`).
- **Keep the docs in sync** — three places describe features to users: the AI prompt (`buildHelperPrompt` — mostly auto-generated from registries, but the DECK STRUCTURE/SLIDE OBJECT prose is manual), the guide (`GUIDE_TABS`), and the `?` shortcut overlay (`helpHTML`). A feature isn't done until all three know about it.
- The deck JSON's `version: 1` is the **schema** version — only change it with a migration plan. `APP_VERSION` is the app release number.

## Testing (no test framework — verify in a browser)

Open `greenroom.html` via `file://` (or a preview pane). Useful console smoke tests:

```js
// every layout renders with defaults
Object.keys(LAYOUTS).every(k => renderStage(newSlide(k), deck).length > 100)
// deck survives a JSON round trip
!!validateDeck(JSON.parse(JSON.stringify(deck)))
// sample deck fully renders
(d => d.slides.every(s => renderStage(s, d).length > 100))(validateDeck(sampleDeck()))
// the AI prompt's embedded example is valid importable JSON
(p => !!validateDeck(JSON.parse(p.slice(p.lastIndexOf('{\n  "version": 1'), p.indexOf('MY OUTLINE')).trim())))(buildHelperPrompt())
```

Then check by hand: designer editing updates the preview; **Present** (nav, transitions, HUD `P`, grid `G`, rail `T`); **Print** both modes; **Publish Player** — smoke-test the generated file by loading its HTML string into an `<iframe srcdoc>` and confirming `#present` exists and `.studio`/`.editbtn` don't. Note: in sandboxed previews (`data:` origin) IndexedDB throws and the top bar shows "Save failed" — that's expected there and only there; also `history.replaceState` is try/caught for the same reason.

## Release checklist

1. Bump `APP_VERSION` (semver-ish: features → minor, fixes → patch).
2. New features documented in guide + help overlay + AI prompt (see sync note above).
3. New schema fields defaulted in `newDeck`/`newSlide`/`validateDeck`; confirm an old exported JSON still imports.
4. Run the console smoke tests and a manual pass of designer → present → print → player.
5. Update README.md if user-facing behavior changed.

## Repo hygiene

`reference_deck.html` and `better-deck-tool.md` are local-only working materials (the reference deck contains private company content) — they are gitignored and must never be committed or published.
