# VOID 1680 AM

**▶ Play it live: https://djessemann.github.io/void1680/**

A digital adaptation of **VOID 1680 AM**, a solo journaling RPG by Ken Lowery (Bannerless Games).

You play a late-night AM radio DJ broadcasting into the dark. In one sitting you build a
four-part playlist and take one caller during each block — the writing is the deliverable,
the songs are just what you reach for. Your night autosaves as you go and exports to a plain-text log at the end.

## Status

- **Phase 1 (live):** complete single-file web app — no build step, no dependencies (other than Google Fonts). Runs entirely in the browser; your writing stays on your device.
- **Phase 2 (planned):** Spotify search + save-to-playlist via OAuth PKCE. Specced but not started. No in-app playback.

## Run it locally

It's one static file. Either open `index.html` directly, or serve it:

```sh
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Deployment

Hosted on GitHub Pages, deployed from the `main` branch root. Pushing to `main` republishes the site automatically.

## More detail

See [`VOID-1680-handoff.md`](./VOID-1680-handoff.md) for the full design notes, architecture, scope decisions, and the Phase 2 Spotify spec.

## Attribution

An unofficial, non-commercial homage. Adaptation of *VOID 1680 AM* by Ken Lowery, published by
Bannerless Games (2023), under the Creative Comrades License Agreement 1.0. Credit to the original
game and author.
