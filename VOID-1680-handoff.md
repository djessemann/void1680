# VOID 1680 AM — Web App Handoff

A digital adaptation of **VOID 1680 AM**, a solo journaling RPG by Ken Lowery (Bannerless Games). This doc is the full context for continuing the build in Claude Code.

Status: **Phase 1 complete** (`index.html`, single file, vanilla JS, no build step). Phase 2 (Spotify) is specced below but not started.

---

## 1. What the game is

A solo journaling RPG where you play a late-night AM radio DJ broadcasting into the void. One play session = one "broadcast," done in a single sitting (~1–2 hrs). You build a four-part playlist and take one caller per part. The *writing* is the deliverable — the songs are just what you reach for, and the callers are imagined characters you write against.

The physical game uses a deck of cards (split into a face-card "Caller" deck and four numbered "Playlist" decks by suit), a d6, and any playlist tool. We've replaced all physical components with in-app RNG, and replaced the original voice-recording mechanic with text journaling.

### The night, in four fixed blocks (always this order)

| Block | Suit | Arc | Caller mood |
|------|------|-----|-------------|
| 1 | ♣ Clubs | Thesis statement, higher energy | Mellow, introspective |
| 2 | ♦ Diamonds | Rising emotional intensity | Agitated, worried |
| 3 | ♠ Spades | The comedown, contemplative | Quiet, melancholy |
| 4 | ♥ Hearts | The finale | Passionate, energized |

### One block (the core loop, repeated 4×)

1. **Queue** — draw 3 numbered cards (2–10) from the block's suit → each maps to a writing prompt → pick a song that answers each, queue them.
2. **On air** — write your spoken intro (this replaces the original voice recording).
3. **Caller** — a caller is generated: draw a face card (age) + their suit (why they called) + roll d6 (conversation topic). Write the conversation.
4. **Backsell** — optional reflection on the songs and the call. Advance.

### Caller resolution

- **Face card → age:** Jack = youth (cusp of adulthood), Queen = adult (20s–40s), King = elder, Ace = ambiguous (deliberately).
- **Caller's suit → why they called:**
  - Same suit as the **current** block → calling about a song you just played (the rules say pick the played song whose card is numerically closest to the face card; since face cards are high, this is effectively the highest-numbered drawn card).
  - Same suit as the **next** block → making a request. Optional to honor. If honored: roll the Request Table (d6), and the requested song opens the next block — you draw one fewer prompt there to make room.
  - Neither → they just want to talk.
- **d6 on the caller-suit's mood table → conversation topic.** Each entry has two flavor options (e.g. "Family, overwhelming or nonexistent"); the player picks one or coin-flips.

---

## 2. Scope decisions (what we cut and why)

These were deliberate simplifications. Don't reintroduce them without reason.

- **No audio.** Cut the original voice-recording mechanic (→ replaced with text fields) AND any in-app music playback/sampling. This is also what keeps the Spotify integration easy (see §5): no Web Playback SDK, no preview URLs, none of the deprecated/dev-mode-restricted endpoints.
- **No multi-broadcast / recurring callers.** The original has an optional system for playing many broadcasts over weeks where callers phone back and their stories evolve across up to 4 calls (Second/Third/Fourth Call tables, marked-card tracking). **Cut entirely.** Each caller in our app is a single, self-contained conversation. This removes the only subsystem that needed cross-session persistence and a caller data model. Each night ends clean.
- **Cut the real-world flavor** (the author's actual AM station, the call-in voicemail line, the syndication/affiliate program, the bannerless.games caller archive). All tied to the audio side we removed.

What we **kept**: the request-carry wrinkle (it's the one mechanic that links blocks and gives the night a narrative spine, and it's cheap). And light localStorage autosave as crash insurance for the single sitting — this is *not* the multi-broadcast system, just so a closed tab doesn't lose an hour of writing.

---

## 3. Architecture (Phase 1 as built)

Single file: `index.html`. Vanilla HTML/CSS/JS, no dependencies except Google Fonts (IM Fell English, IM Fell English SC, Space Mono). Mirrors the stack of the author's other tribute app (Alone Among the Stars) so it deploys to GitHub Pages unchanged.

- **One linear state machine**, no router, no framework. Stages: `title → signon → block (×4) → signoff`.
- **State object `S`** holds everything: dj/loc/theme, current `blockIndex`, `pendingRequest` (carries an honored request to the next block), and a `blocks` map keyed by suit, each with `{draws, songs, requestPrefill, onair, caller, callerNotes, backsell}`. Plus `lastwords`.
- **Persistence:** `S` is JSON-serialized to `localStorage` (key `void1680.session.v1`) on every change. On boot, if a saved session exists past the title screen, it resumes. "Begin a new night" clears it.
- **Data tables** (prompts, mood/topic tables, face cards, request table, block metadata) are plain JS objects near the top of the `<script>` — this is where to edit game content.
- **Key functions:** `renderBlock` (queue), `renderOnAir`, `rollCaller` + `renderCaller`, `renderBacksell`, `buildLog`/`exportLog`, `paintDial` (the dial chrome), `tune` (the static transition), `go(stage)` (navigation).
- **Export:** `buildLog()` assembles a plain-text log of the whole night; `exportLog()` downloads it as `void-1680-YYYY-MM-DD.txt`.

### Known seams / Phase-1 limitations

- Resuming mid-block always re-renders the **queue** sub-screen (songs preserved). Caller is *not* re-rolled on resume (it's persisted once generated). Acceptable; tighten later if desired.
- Song slots are plain `<input class="song">` text fields. These are the exact elements Phase 2 replaces with Spotify search.

---

## 4. Design system

Grounded in the source material — a *vintage AM-transmitter owner's manual*, not neon "dark mode." Deliberately avoids the generic AI dark-UI-with-one-acid-accent look.

**Palette (CSS custom properties in `:root`):**
- `--void #15101b` deep plum night (bg; not pure black)
- `--void-deep #0c0910` vignette pockets
- `--paper #e8dcc4` warm aged paper (card surfaces / writing areas)
- `--ink #241b12` text on paper
- `--tungsten #e8943a` tube-glow amber — the one accent / the "signal"
- `--ember #b8482a` deeper warm secondary
- `--signal #a8c08f` dim radium green — used *only* for the ON AIR pip

**Type:** IM Fell English / IM Fell English SC (display + the manual's in-world voice — this is the game's actual typeface). Space Mono for instrument/dial readouts and labels. User's own writing uses a plain system sans — a deliberate contrast: the antique instrument vs. the operator's present-day words.

**Signature element:** the **AM tuning dial** is the persistent top chrome AND the navigation. A frequency band (1620→1680 kHz) with a needle that tunes from block to block; the active block glows tungsten, completed blocks dim. It encodes real progress (where you are in the night = where the needle is). Going live lights the green ON AIR pip; the caller arrives through a brief static-shimmer "tuning in" transition (`#static` overlay + `tune()`).

**Quality floor:** responsive to phone widths, visible keyboard focus rings, `prefers-reduced-motion` disables the static/animations. Copy is written in the game's in-world register, kept minimal so the screen leaves room for the player's writing; all rules detail lives behind the `?` manual overlay.

**Possible enhancement (not built):** an AAtS-style tactile "draw" reveal — showing the card/die physically turn over on the Queue and Caller steps. This is presentation only; the logic already resolves the draw into a sentence.

---

## 5. Phase 2 — Spotify (specced, not built)

Goal: type-and-search songs against a real Spotify catalog inside the app, then save the night's picks as a playlist on the player's account. **No in-app playback** (intentional — keeps it simple and avoids fragile APIs).

**Approach:**
- **Auth:** OAuth 2.0 Authorization Code flow **with PKCE**. No backend, no client secret — works on a static GitHub Pages site. Token stored client-side.
- **Only two endpoints needed**, both stable and outside the dev-mode-restricted set:
  - Track **search** (`GET /v1/search?type=track`) — powers the song slots.
  - Playlist **create + add tracks** (`POST /v1/users/{id}/playlists`, `POST /v1/playlists/{id}/tracks`) — powers "Save to Spotify" on sign-off.
- **Scopes:** `playlist-modify-public` and/or `playlist-modify-private`.
- **Do NOT use:** Web Playback SDK (Premium-gated, heavy), preview URLs, audio-features/recommendations/related-artists (deprecated for dev-mode apps).

**Quota reality:** a personal app stays in Spotify "development mode" — only ~25 manually allowlisted Spotify accounts can use it, and extended quota is routinely denied for hobby apps. Fine for personal/homage use. If this ever needs to be public, that's the constraint to plan around.

**Integration points in the existing code:**
- Sign-on screen has a disabled "Connect Spotify · later" button → wire to the PKCE login.
- Each `<input class="song">` in `renderBlock` → replace with a search-box component that resolves to a `{name, artist, uri}` and stores the URI in `S.blocks[suit].songs` (currently stores plain strings; extend the shape).
- Sign-off has a disabled "Save to Spotify · later" button → collect all stored track URIs across the four blocks and create the playlist.
- **Keep graceful degradation:** if Spotify isn't connected, the text-field behavior must remain. The engine must never depend on Spotify being available.

---

## 6. Attribution / licensing

Adaptation of *VOID 1680 AM* by Ken Lowery, published by Bannerless Games (2023), licensed under the Creative Comrades License Agreement 1.0. Treat this as a personal, non-commercial homage (same posture as the author's Alone Among the Stars tribute). Credit the original game and author visibly. Confirm the license terms before any public/commercial distribution.
