# Cough (Bloom) — Sound Visualiser · Dev Handover

**Component:** Respiratory Health "Cough" check card with a live, microphone-driven 3D sound sculpture.
**Deliverable:** `index.html` (single, self-contained file — no build step, no dependencies to install).
**Status:** Design-approved, ready for integration.
**Last updated:** 2026-09-24

---

## 1. What this is

A mobile card (390 × 736 design frame) showing a mint/teal "Cough" panel. When the user taps **Start** and grants microphone access, live audio is analysed in real time and rendered as a 3D "bloom" of coloured dots plus a thin connecting "melody thread". Pitch drives colour; loudness drives density, size and rotation speed.

There are **two variants** of this visual in the repo — this handover covers the **bloom** only:

| Variant | Folder | Shape the sound forms |
|---------|--------|-----------------------|
| **Bloom** (this doc) | `cough-bloom/` | Loose, organic folded space-curve |
| Coil | `cough/` | Tight coiling helix |

They are otherwise identical. **The colour change described in section 8 was applied to the bloom only.**

---

## 2. Files to ship

```
cough-bloom/
└── index.html      ← the entire component (HTML + CSS + JS inline)
```

That's it. One file. Everything is inline except one external dependency:

- **Google Fonts – Ubuntu** (weights 300/400/500/700), loaded via `<link>` from `fonts.googleapis.com`. If the target environment blocks external font CDNs, self-host Ubuntu or swap the `font-family` fallback (currently `system-ui, sans-serif`).

---

## 3. How to run / preview

Because it uses the microphone, it must run in a **secure context**. Any of these work:

- Open the file directly (`file://…/index.html`) — fine for local review.
- Serve over **HTTPS** (e.g. GitHub Pages — the current host).
- Serve from **`localhost`** during development.

It will **not** get microphone access over plain `http://` on a remote host (browser security rule, not a bug).

Steps to test:
1. Open the page.
2. Tap **Start**.
3. Allow microphone access when prompted.
4. Make sound (talk / hum / cough) — the bloom builds up.

---

## 4. Integration notes

- **Self-contained & framework-agnostic.** Can be dropped into any page or wrapped in a WebView. No React/Vue/build tooling required.
- **Responsive scaling.** The design is authored at a fixed 390 px width and scaled to fit the viewport by JS (`fit()` in the first `<script>` block). It scales up to a 1.25× cap.
- **Rendering.** WebGL (`canvas#viz`). Falls back to *nothing rendered* (silent no-op) if WebGL is unavailable — the card and controls still show.
- **Audio.** Web Audio API `AnalyserNode`, `fftSize = 4096`. Mic stream requested with echo cancellation / noise suppression / auto-gain **off** (deliberate — we want the raw signal).
- **No data leaves the device.** Audio is analysed in-browser only; nothing is recorded, stored or transmitted.
- **Accessibility.** Honours `prefers-reduced-motion` (rotation/bob are stilled). Control buttons have `aria-label`s.

---

## 5. Browser support

| | |
|---|---|
| **Needs** | WebGL, Web Audio API, `getUserMedia`, secure context |
| **Tested target** | Mobile Safari (iOS) and Chrome |
| **Graceful degradation** | If mic is blocked, an inline error message replaces the descriptor text; the visual simply stays empty |

---

## 6. Design spec — layout & tokens

**Design frame:** 390 × 736 px. **Base font:** Ubuntu.

### Colours

| Role | Value |
|------|-------|
| Page background | `#eef0f3` |
| Card surface (mint) | `#acf5f4` |
| Card border | 2px `#ffffff` |
| Stack card 1 (behind) | `#d3fafa` |
| Stack card 2 (behind) | `rgba(211,250,250,0.5)` |
| Base gradient (bottom of card) | transparent → `#19dbe3` |
| Start disk gradient | `#51e8e2` → `#00b1c1` |
| Control buttons | `rgba(17,17,17,0.6)` + backdrop blur |
| Primary text | `#111` |
| Error text | `#7a1220` |

### Card

- Size 342 × 570, radius 24, 2px white border, shadow `0 4px 24px rgba(17,17,17,0.12)`.
- Two dimmed duplicate cards stacked behind (offset 12/16 and 24/32).
- Top scrim: mint→transparent gradient over the top 150 px so the title sits clear of the sculpture.

### Type

| Element | Spec |
|---------|------|
| App header "Respiratory Health" | Ubuntu 500, 18/24, `#111` |
| Card title "Cough" | Ubuntu 300, 30/32, letter-spacing −0.5px |
| "AI" badge | 28 px black circle, white 15.75px 500 |
| Descriptor | Ubuntu 400, 14/20, centred |
| Tabs (active / dim) | 500 16/22 / 400 14/20 `#666` |
| Floating labels ("tag") | Ubuntu 400, 10px, uppercase, 0.5px black outline, 4px radius, ~60% opacity |

### Controls

- **Start** button: 87.48 px outer, 76 px inner gradient disk. While listening it emits a pulsing ring (`@keyframes pulse`, 1.8s).
- Bookmark + Info buttons: 44 px circles, blurred dark background, white 1.5px stroke icons.

---

## 7. Visualiser behaviour — tunable constants

All near the top of the second `<script>` block:

| Constant | Value | Meaning |
|----------|-------|---------|
| `F_LO` / `F_HI` | 300 / 12000 Hz | Frequency range mapped into the sculpture |
| `TTL` | 5.0 s | How long each dot lives before fading out |
| `PEAK_THRESH` | 58 | Sensitivity for spawning cluster dots |
| `MEL_THRESH` | 86 | Sensitivity for the melody thread |
| `SPAWN_K` | 4 | Max new dots per analysed frame |
| `MAX_PTS` / `MAX_MEL` | 4500 / 1500 | Point budget caps (performance) |
| `anchor(hz)` | — | Maps a frequency to its fixed 3D home. **This function is what makes the bloom shape** (vs. the coil in `cough/`). |

Floating labels cycle through: **calibrating → analysing → mapping** (`LABELS` array). Pure decoration; edit freely.

---

## 8. Change log

**2026-09-24 — Dot colour palette → range of blues.**
The pitch→colour ramp (`const CS`) was changed from the original orange→coral→magenta→purple to a wide **deep-to-vibrant blue** range that reads against the mint card. This recolours both the dots and the melody thread (they share the ramp).

Current palette (low pitch → high pitch), RGB normalised 0–1:

```js
const CS = [
  [0.0,  [0.16, 0.70, 1.00]],  // vibrant electric blue
  [0.28, [0.11, 0.52, 0.98]],  // vivid blue
  [0.55, [0.10, 0.36, 0.90]],  // royal blue
  [0.80, [0.07, 0.22, 0.68]],  // deep blue
  [1.0,  [0.03, 0.10, 0.38]]   // rich navy
];
```

To adjust the palette later, edit only this array — the number of stops is flexible; `freqColor()` interpolates between them automatically.

---

## 9. Known caveats / notes for dev

- The mic must be a **user gesture** away — the Start tap is what unlocks audio (browser autoplay policy). Don't auto-start.
- On iOS, the AudioContext must be resumed inside the tap handler (already handled in `startBtn` click).
- The tab labels in the footer/tabs area ("Skin Spot AI", "Skin Health AI", "Explore") are **static placeholders** from the design — wire them up or remove per the real IA.
- Point budgets (`MAX_PTS`) are tuned for mobile; raise cautiously on low-end devices.

---

## 10. Live location

Currently published via GitHub Pages under the `resonance` repo:
`/<pages-base>/cough-bloom/`
