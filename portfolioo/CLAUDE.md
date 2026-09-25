# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Static two-page portfolio site for **ByOcompos**, a recording/mixing/mastering audio engineer.
Vanilla HTML/CSS/JS. There is no build step, no package manager, no test suite, no linter, and
no git repository — the four source files are the deliverable, and edits are live immediately.

## Running

Open `index.html` directly for quick CSS checks, but prefer a local server: `file://` breaks the
`?category=` links between the two pages and blocks the YouTube embeds in some browsers.

```powershell
npx serve . -l 5173   # Python is not installed on this machine; python -m http.server won't work
```

The audio players also need a server: some browsers won't seek `file://` media reliably.

## Architecture

Two pages share one stylesheet and one script:

- `index.html` — landing page. Sections in order: hero (`#home`), `#services`, `#portfolio`,
  `#sounds`, `#before-after`, `#trusted`, `#about`, `#contact`, footer. Nav (desktop and
  `.mobile-nav`) lists every section except hero; `portfolio.html`'s nav links back with
  `index.html#…`. The desktop nav collapses to the hamburger at **960px**, not 768.
- `portfolio.html` — full gallery with category filter tabs.
- `style.css` — **all** styling for both pages. Design tokens (ink ramp, signal amber, radii,
  shadows) are CSS custom properties in `:root` at the top; the rest is divided into commented
  banner sections. Change tokens, not individual rules. New rules go in the matching banner
  section.
- `script.js` — all shared behavior, loaded by both pages.

### script.js is shared and guard-based

Every block in `script.js` null-checks its elements, so the same file runs on both pages and
silently no-ops where markup is absent. Three blocks are **currently inert** — no markup on
either page uses their selectors:

- block 6, `.audio-item` (audio player)
- block 10, `.booking-btn` (booking prefill)
- the `.video-item` half of block 7

Don't assume they execute; don't "fix" them unless adding the corresponding markup.

Blocks added in the UI revision: 5b `AudioBus` + `formatTime`, 13 keyboard access for cards
(adds `tabindex`/`role=button`/`aria-label` at runtime to `.work-card[data-video]` and
`[onclick*="openCoverModal"]` — don't duplicate these in markup), 14 scroll-spy (`aria-current`
on nav links), 15 sounds player, 16 before/after.

Modals: `#video-modal` exists in both pages' HTML. `#cover-modal` does **not** — block 8 builds
it lazily via `openCoverModal(src, title, artist)` on first call. `openCoverModal`,
`closeCoverModal`, `openVideoModal`, and `closeVideoModal` are deliberately assigned to `window`
because cover cards invoke them through inline `onclick`. Both modals move focus to their close
`<button>` on open and restore it to the trigger on close.

The starfield canvas (block 5) is skipped entirely under `prefers-reduced-motion` and paused on
tab hide — verify animation work against both states.

### Audio sections

**One sound at a time.** Every player registers a `{ pause() }` controller and calls
`AudioBus.claim(ctrl)` before playing; `openVideoModal` calls `AudioBus.pauseAll()`. A new player
must do the same.

**`#sounds` (block 15)** — one `Audio` object driven by a playlist of `.track` buttons. A track is
entirely its `data-audio` / `data-title` / `data-meta` / `data-year` attributes; copy an `<li>`
to add one. On load every file is probed for metadata (then the probe is torn down so it doesn't
download the whole file); failures get `.is-unavailable` + `disabled` and meta "coming soon". If
nothing loads, the player shows "tracks coming soon".

**`#before-after` (block 16)** — tabs are built from the `.ab-pair` data holders
(`data-before`, `data-after`, `data-title`). Each pair gets two `Audio` elements that always play
together; the A/B toggle only flips `.muted`, so switching never seeks or reloads. Raw is the
master clock; an rAF loop snaps the final file back when drift exceeds 0.05s, and `waiting` on
one pauses the other until both can play. Re-seeking is only done when actually out of sync —
an unconditional seek re-triggers `waiting` and loops. The waveform is decorative: seeded
`.ab-bar` spans generated in JS.

Audio files live in `resource/audio/`: `track-01.mp3`…`track-06.mp3`, `before-N.mp3` /
`after-N.mp3` for N = 1..3. Paired files must share length and start sample.

### Portfolio content model

A portfolio video is keyed entirely by its **YouTube ID**, which appears four times per card:
the `data-video` attribute, the `hqdefault` thumbnail URL, the `mqdefault` `onerror` fallback,
and (at runtime) the modal embed. Changing a video means changing all four.

Category filtering lives in the inline `<script>` at the bottom of `portfolio.html`, not in
`script.js`. It reads `?category=` (defaulting to `videos`), toggles `.work-card` visibility by
`data-category`, and writes the choice back to the URL via `history`. The landing page links in
with `portfolio.html?category=videos` and `portfolio.html?category=cover`.

**The landing page's portfolio rows are a hand-picked subset of `portfolio.html`, not generated
from it.** Adding a work to the gallery does not surface it on the landing page, and the two
lists drift unless synced manually. The "view all 7 →" count in `index.html` is likewise
hardcoded.

The landing page's **cover-art row is hidden, not deleted** (`.port-row.is-hidden`, with a comment
above it). Remove `is-hidden` to restore it; `.port-row:has(+ .port-row.is-hidden)` strips the
mix & master row's bottom rule while it's hidden.

### Styling lives in one file

`portfolio.html` used to carry a page-local `<style>` block for gallery-only rules. That block
is gone — `.portfolio-page`, `.filter-bar`, `.filter-btn`, `.works-grid`, `.thumb-wrapper`,
`.play-badge`, and the rest now live in `style.css` under the "PORTFOLIO PAGE" banner. Put new
gallery styling there, not back in the page head.

### Palette

The design is a warm near-black "mastering room" with a signal-amber accent (`--signal:
#ff7a18`), matte panels, and bevel hairlines. There is no glassmorphism: `backdrop-filter`
survives only on the fixed header and modal scrims.

Block 2 only toggles `header.is-scrolled` (styled in CSS); the starfield particles in block 5 are
warm white and amber. One JS/markup override remains: `index.html` sets a cool-blue `background`
inline on each `.img-skeleton`, and `.port-grid .img-skeleton` uses one `!important` to keep empty
tiles (the "view all" cards) on the warm ground.

There is no purple anywhere despite class names like `.purple-footer` / `.purple-italic` — they
are legacy names. Semantic tokens added in the UI revision: `--text-legend` (0.72rem floor for
small uppercase text), `--control-size` (44px touch target), `--surface-panel`,
`--header-offset` (used by `scroll-margin-top`). `--text-muted` was raised to `#8f867c` for AA
contrast.

Legacy token aliases (`--accent`, `--accent-light`, `--glass`, `--tonal-*`, …) are kept at the
bottom of `:root` and remapped onto the new palette, because inline SVG `stroke` attributes in
both HTML files reference `var(--accent-light)` directly. Don't delete them without grepping the
markup first.

### Client carousel

The `#trusted` marquee in `index.html` achieves its infinite loop by holding **two sets** of
`.client-item` blocks, the second marked `aria-hidden="true"`. Any client added to one set must
be added to the other or the loop visibly jumps.

## Known-broken state

Pre-existing issues, not regressions. Leave them unless asked:

- The contact form does not submit anywhere — its `onsubmit` fires an `alert()` and resets.
- `favicon.ico` and `og-image.jpg` are referenced in both `<head>`s but absent from the repo.
- The three `data-category="film"` cards use stock YouTube IDs (including `dQw4w9WgXcQ`) under
  invented titles — placeholder content.
- The carousel's duplicate set lists *different* names than the first set, so it is not a true
  loop.
- Every `.client-avatar-img` has `src=""`, falling through to the SVG placeholder.
- Footer social links point at bare platform homepages.

## Conventions

- Cards carry long inline `style` and `onerror` attributes. This is the existing idiom; match it
  when copying a card, and don't refactor it wholesale as a side effect of another task.
- Images use inline SVG data-URI `onerror` fallbacks so a missing asset degrades to a labeled
  placeholder rather than a broken-image icon.
- New landing sections use the `.sec-head` pattern (`.eyebrow` + `.section-title` + `.sec-desc`).
- Touch devices (`@media (hover: none)`) always show tile captions/play badges — don't make new
  affordances hover-only.
- Section anchors are lowercase; nav labels and section titles are lowercase by design.
- Copy mixes English and French (`qui suis-je ?`, client roles like `Bijouterie`,
  `Créateur de contenu`). Preserve the language of whatever you edit.
