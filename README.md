# Greenroom

**A one-file slide studio.** Design a deck, present it full-screen, and carry the whole thing anywhere — designer, presenter, themes, and your saved decks all live in a single HTML file that runs straight from disk. No install, no server, no internet, no account.

> Current version: **1.6.0** (shown next to the logo in the app)

## Quick start

1. Download `greenroom.html`.
2. Double-click it (Chrome or Edge recommended; Firefox works too).
3. It opens in the **designer** with a sample deck — poke around, then hit **▶ Present**.

A built-in guide opens on first launch and stays available under the **?** button at the top right. Your work autosaves to the browser as you type.

## What's inside

### Design
- **27 slide layouts** — hero, section splash, statement, topic grid, split + card, comparison, before / after, image, video, code window, bulleted list, do / don’t, checklist, columns, feature cards, 2×2 matrix, big callout, quote, big stats, data bars, data table, funnel, progress ring, timeline, process flow, people grid, and closing — chosen from a **visual picker** with live thumbnails. Picked wrong? *Layout → Change…* previews **your own content** poured into every other layout before you commit.
- **Rich text markup** in any field: `[[gold accent]]`, `{{orange accent}}`, `[link text](url)` (opens in a new tab), and line breaks.
- **Sections & grouping** — child slides inherit their section's name as an eyebrow label and nest in the thumbnail rail.
- **Slide sorter** (⊞) — the whole deck as a drag-to-reorder grid. **Ctrl+F** searches all text across the deck.
- **Organization mark** — a deck-level company name, disclaimer, or copyright line shown small on the left of every slide (top or bottom), with a per-slide hide for title slides.
- **Undo/redo** (Ctrl+Z / Ctrl+Y), **hidden backup slides**, per-slide **presenter cues** and **speaker notes**, image and video embedding (base64 — the deck stays one portable file), and an overflow warning when content runs off the slide.

### Themes
Four built-in looks — **Emerald & Gold**, **Midnight Slate**, **Ember**, and the light **Boardroom Ivory** — plus a **custom theme builder**: pick five colors and a complete theme (gradients, cards, chrome, ambient particles) is derived and saved inside your deck.

### Present
- Fixed 1920×1080 stage scaled to any window, entrance animations, ambient animated backgrounds in four styles (**Calm**, lively and gentle **Particles**, **Aurora**), and slide transitions (fade / slide / zoom / none).
- Navigation via dots or a **thumbnail rail**, auto-hiding chrome, optional slide counter and Skip-to-end.
- **Presenter HUD** (`P`): elapsed timer, next-slide preview, and your notes — with target-length warning colors.
- **Target-time progress bar**: an optional thin line along the bottom edge that fills toward the deck's target length, with a notch showing where the current slide sits so you can see at a glance whether you're ahead or behind. The clock starts when the show opens or on the first slide change. `H` shows or hides it during the show.
- **Rehearsal mode** (`R`): records per-slide timings and shows a recap table.
- **Laser pointer** (`L`), **draw on the slide** (`D`, `C` clears), **black screen** (`B`), **slide grid** (`G`), and **number + Enter** to jump. Press `?` during a show for the full shortcut list.
- **Autoplay / kiosk mode**: seconds per slide (with per-slide overrides), optional loop — for lobby screens and self-running demos.

### Share
- **JSON menu** (Import · Export · View/edit) — a deck is a single portable JSON file. Import offers *open as new deck* or *append to the current one*.
- **Publish Player** — export a standalone HTML that **is** the presentation: recipients double-click and it opens straight into the show, no designer, nothing to import.
- **Print / PDF** — slide pages, a **Handout** mode with speaker notes under every slide, or a **Plain** mode: black-on-white, three slides per page with ruled note lines beside each.
- **Snapshots** — named restore points that survive reloads, managed in the Decks dialog.

### ✨ AI Kickstart
Don't start from a blank deck. The **✨ AI Prompt** button copies a prompt that teaches any AI assistant (Copilot, Claude, ChatGPT, …) Greenroom's exact JSON format. Paste it with your notes/outline — or attach an existing `.pptx`/PDF and ask for a conversion — then save the JSON reply and **Import** it here. The prompt's schema documentation is generated from the app's own layout registry, so it is always up to date.

## Data & privacy

Everything is local. Decks live in your browser's IndexedDB (per machine, per browser); nothing is uploaded anywhere. Export JSON is the backup and transfer format — images and video are embedded in it, so one file is always the whole deck.

## Browser notes

- **Chrome / Edge**: fully supported, including running from `file://`.
- **Firefox**: supported.
- **Safari**: presentation works; the SVG favicon may not display.
- The **player file** and the app itself never need a server — but decks are saved *per browser*, so when presenting from a new machine, bring your JSON (or a published player file).

## Development

The entire application is one hand-written HTML file — no build step, no dependencies. If you want to modify or extend it (especially with an AI coding agent), read **[CLAUDE.md](CLAUDE.md)** first: it documents the architecture, the extension points (layouts, themes), the invariants that must not break, and the release checklist.

## License

[MIT](LICENSE) — use it, modify it, share it.
