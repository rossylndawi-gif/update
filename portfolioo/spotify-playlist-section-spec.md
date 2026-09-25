# Spec: "Latest Releases" Playlist Section

## Page Placement
```
Hero
Services
→ NEW: Playlist Section (this spec)
About
Contact
```

## 1. Purpose
A section showcasing songs released on Spotify, playable inline on the portfolio site with polished motion — no user should have to leave the page to preview a track.

## 2. Data Source & Playback Method
Use the **Spotify oEmbed / Embed API** (`open.spotify.com/embed/...`) — works for any public playlist/track, requires no OAuth or backend, and gives a real 30-second (or full, if the visitor is logged into Spotify) inline preview player.

- Playlist source: `[PASTE YOUR SPOTIFY PLAYLIST URL HERE]`
- Convert to embed src format: `https://open.spotify.com/embed/playlist/{PLAYLIST_ID}?utm_source=generator`
- Fetch track metadata (title, artist, cover art, duration, release date) via the **Spotify Web API** (`GET /v1/playlists/{id}/tracks`) using the Client Credentials flow (no user login needed for public data) — this powers the *custom* card UI; the embed iframe itself only handles actual audio playback.
- If Client Credentials setup is out of scope for now, fall back to hardcoded track metadata in a local JSON file matching the schema in Section 4, and swap in the live fetch later.

## 3. Layout
- **Desktop:** horizontal scroll-snap carousel, 3.5 cards visible at once, peek of next card to signal scrollability
- **Mobile:** vertical stack, full-width cards
- Section header: "Latest Drops" (or similar), left-aligned, with a small "Listen on Spotify" pill-link to the full playlist, top-right on desktop

## 4. Track Card — Data Schema
```json
{
  "id": "spotify_track_id",
  "title": "Track Name",
  "artist": "Artist Name",
  "coverArtUrl": "https://...",
  "releaseDate": "2026-03-01",
  "durationMs": 210000,
  "previewUrl": "https://p.scdn.co/mp3-preview/..." 
}
```

## 5. Card Anatomy
- Square cover art (rounded per design-system `--radius-card` token)
- Track title (`--text-heading`), artist (`--text-body`, `--color-text-muted`)
- Play/pause button, centered overlay on cover art, hidden until hover/focus (always visible on mobile/touch)
- Thin progress bar along the bottom edge of the cover art, fills as the track plays
- Release date, small, bottom-right corner of card

## 6. Motion Specification
Use `framer-motion`. Keep everything on **transform/opacity only** — no layout-triggering properties — for smooth 60fps.

| Interaction | Effect | Duration | Easing |
|---|---|---|---|
| Card entrance (on scroll into view) | Fade + slide up 16px, staggered 60ms per card | 400ms | `cubic-bezier(0.16, 1, 0.3, 1)` (ease-out-expo) |
| Card hover | Scale 1 → 1.03, cover art brightness 0.9 → 1 | 200ms | ease-out |
| Play button reveal | Opacity 0 → 1, scale 0.8 → 1 | 150ms | ease-out |
| Play button press | Scale 1 → 0.92 → 1 | 120ms | spring (stiffness 400, damping 25) |
| Track playing (active card) | Cover art gets a subtle pulsing glow ring (box-shadow opacity 0.4 → 0.7 loop) | 1.5s loop | ease-in-out, `repeat: Infinity, repeatType: "reverse"` |
| Equalizer bars (replaces play icon while playing) | 3 vertical bars animating height randomly between 20%-100% | 400-600ms per bar, offset per bar | ease-in-out loop |
| Progress bar fill | Width transitions in sync with `previewUrl` audio `currentTime` | linear, tied to playback | linear |
| Carousel scroll (desktop) | Native scroll-snap, but arrow-button clicks animate scrollLeft | 500ms | ease-in-out |

Only **one card plays at a time** — starting a new track must fade out and pause any currently-playing card (200ms fade on its glow ring / progress bar reset).

## 7. Interaction Rules
- Clicking the cover art or play button toggles play/pause for that track's `previewUrl` via the native `<audio>` element (do NOT use the Spotify iframe for this — reserve the iframe for a single "Open in Spotify" full-player, embedded separately at the end of the carousel as a bonus card, so you get real inline audio scrubbing without relying on Spotify's iframe UI for every card).
- Keyboard: cards are focusable, `Space`/`Enter` toggles play/pause on the focused card.
- Add `aria-label="Play preview of {title} by {artist}"` on each play button; announce play/pause state changes via `aria-live="polite"`.

## 8. Responsive Rules
- Below 640px: disable hover-only reveal of the play button (always show it, semi-transparent overlay at 60% opacity)
- Reduce entrance stagger to 0 on mobile (all cards fade in together) to avoid a sluggish feel on slower devices
- Respect `prefers-reduced-motion`: disable the pulsing glow, entrance slide (fade only), and equalizer animation (swap to a static "playing" icon)

## 9. Design System Integration
Pull spacing, radius, color, and typography tokens from the synced Claude Design system (`/design-sync`) rather than hardcoding — this component should look native to the rest of the site, not like a bolted-on widget.

## 10. Component Breakdown (suggested file structure)
```
/components/playlist/
  PlaylistSection.tsx      — section wrapper, header, "Listen on Spotify" link
  TrackCarousel.tsx        — scroll-snap container, arrow controls
  TrackCard.tsx            — individual card, play state, progress bar
  useAudioPlayer.ts         — hook: manages single global "currently playing" state across cards
  spotify.ts                — API helper: client credentials fetch + token caching
```

## 11. Open Items for You to Fill In Before Handing to Claude Code
- [ ] Real Spotify playlist URL
- [ ] Whether you have Spotify API credentials (Client ID/Secret) already, or want the JSON-fallback approach first
- [ ] Section heading copy ("Latest Drops" / "New Music" / your own title)
- [ ] Whether "Open in Spotify" should link to the whole playlist or each track individually
