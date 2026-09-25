# ByOcompos — Audio Engineer Portfolio

A static, dependency-free portfolio site for **Ocompos**, a recording / mixing / mastering
engineer. Two pages, one stylesheet, one script — no build step, no framework, no package
manager.

- **`index.html`** — the landing page: hero, services, a curated portfolio preview, about,
  contact form, client carousel, footer.
- **`portfolio.html`** — the full gallery, with category filter tabs (`videos`, `cover art`,
  `film sound`) driven by a `?category=` URL parameter.

## Running it locally

The site is plain HTML/CSS/JS, so opening `index.html` in a browser mostly works. Prefer a
local server, though — `file://` breaks the `?category=` links between pages and blocks the
YouTube embeds in some browsers:

```powershell
# Python
python -m http.server 8000

# or Node
npx serve .
```

Then visit <http://localhost:8000>.

## Layout

```
index.html       landing page
portfolio.html   full gallery
style.css        all styling for both pages; design tokens in :root at the top
script.js        all shared behavior, loaded by both pages
resource/
  logo.jpeg                  about-section portrait
  logo-removebg-preview.png  transparent hero logo
  cover1–7.jpeg              cover-art images
```

`style.css` is organized into commented banner sections (base/reset, header, hero, services,
portfolio grid, about, contact, trusted carousel, footer, modals, media queries). Add new rules
inside the matching section rather than at the bottom of the file.

## What `script.js` does

Both pages load the same script; each block guards against missing elements, so it's safe on a
page that doesn't use a given feature.

| # | Block | Notes |
|---|-------|-------|
| 1 | Preloader | Hides on `load`, plus a 1.5s fallback so slow fonts never trap the page |
| 2 | Header scroll effect | Shrinks and darkens the header past 50px of scroll |
| 3 | Mobile nav | Hamburger toggles `.mobile-nav.open` and locks body scroll |
| 4 | Scroll reveal | `IntersectionObserver` adds `.active` to any `.reveal` element, once |
| 5 | Starfield canvas | Ambient particles; **skipped entirely** under `prefers-reduced-motion`, paused when the tab is hidden |
| 7 | YouTube modal | Any `.work-card[data-video="<id>"]` opens `#video-modal` with an autoplay embed |
| 8 | Cover-art modal | `openCoverModal(src, title, artist)` builds `#cover-modal` lazily on first use |
| 9 | Escape key | Closes both modals |
| 11 | Smooth scroll | Offsets `#anchor` jumps by the header height |
| 12 | 3D hero tilt | Mouse/touch tilt on `.hero-container`, capped at ±10° |

Blocks 6 (`.audio-item`), 10 (`.booking-btn`), and the `.video-item` half of block 7 are
currently inert — no markup on either page uses those classes. They're wiring left over for an
audio player and a booking flow; harmless, but don't assume they run.

## Editing content

### Add a portfolio video

Videos are keyed by YouTube ID — the thumbnail URL, the `data-video` attribute, and the modal
embed all derive from it. In `portfolio.html`, copy an existing `<article class="work-card">`
inside the videos group and swap the four occurrences of the ID plus the title and meta text:

```html
<article class="work-card reveal" data-category="videos" data-video="YOUTUBE_ID">
  <div class="thumb-wrapper">
    <img src="https://img.youtube.com/vi/YOUTUBE_ID/hqdefault.jpg"
         onerror="this.src='https://img.youtube.com/vi/YOUTUBE_ID/mqdefault.jpg'"
         alt="Track name" loading="lazy" />
    ...
```

To feature it on the landing page too, add a matching card to the `mix & master` row in
`index.html`. That row is a hand-picked subset — it is not generated from `portfolio.html`, so
the two lists have to be kept in sync by hand.

### Add a cover art

1. Drop the image in `resource/` as `cover8.jpeg` (square, and compressed — these are served
   raw).
2. In `portfolio.html`, copy a `.work-card.cover-art` article and update the image path, title,
   and artist.
3. Update the landing page's "view all 7 →" label in `index.html`, which is hardcoded.

### Add a client to the carousel

The `#trusted` carousel in `index.html` scrolls infinitely by holding **two sets** of
`.client-item` blocks — the second set is marked `aria-hidden="true"`. Add your entry to both
sets or the loop will visibly jump. (Note: the two sets currently list *different* names, which
is why the carousel's contents change as it wraps — see below.)

### Design tokens

Colors, radii, and shadows are CSS custom properties in `:root` at the top of `style.css`. The
palette is a warm near-black "mastering room" with a signal-amber accent — `--ink-900: #080706`
for the ground, `--signal: #ff7a18` for the accent, and matte panels with bevel hairlines rather
than glass. Change the token, not the individual rules.

A block of legacy aliases (`--accent`, `--accent-light`, `--glass`, `--tonal-*`) sits at the end
of `:root`, remapped onto the new palette. They exist because inline SVG `stroke` attributes in
both HTML files reference `var(--accent-light)` directly — grep the markup before removing any.

## Known gaps

Things a new contributor will trip over:

- **The contact form doesn't send anything.** `index.html`'s `onsubmit` shows an `alert()` and
  resets the form. Wiring it to a real endpoint (Formspree, Netlify Forms, a serverless
  function) is the main outstanding task.
- **Missing referenced assets.** `favicon.ico` and `og-image.jpg` are referenced by both pages'
  `<head>` but do not exist in the repo — favicons 404 and social-share previews will be blank.
- **Film sound is placeholder content.** The three `data-category="film"` cards in
  `portfolio.html` point at stock YouTube IDs (including the well-known `dQw4w9WgXcQ`) with
  invented titles. Replace or remove them before going live.
- **Client carousel duplicate set diverges.** The second, `aria-hidden` set of client items has
  different names and roles from the first (e.g. `Icelaye` vs `Fettyndos`), so the marquee isn't
  a true loop and screen readers see only half the list.
- **Client avatars are empty.** Every `.client-avatar-img` has `src=""`, falling back to the SVG
  placeholder icon. An empty `src` re-requests the page in some browsers; remove the `<img>` or
  give it a real image.
- **Heavy inline styles.** Portfolio cards carry long inline `style` and `onerror` attributes.
  Moving those into `style.css` classes is the cheapest readability win available.

## Deploying

No build step — publish the directory as-is to any static host (GitHub Pages, Netlify, Vercel,
Cloudflare Pages, or plain shared hosting). Before the first deploy:

- Add the missing `favicon.ico` and `og-image.jpg`.
- Confirm the canonical/OG URLs in both `<head>`s match the real domain (they currently say
  `https://byocompos.com/`).
- Replace the placeholder social links in the footer — Instagram, YouTube, Spotify, SoundCloud,
  X, and TikTok all point at the bare platform homepages.
