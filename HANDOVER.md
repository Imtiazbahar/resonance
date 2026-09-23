# Resonance — Sound Visualisers · Project Handover

**What this repo is:** a set of live, microphone-driven sound visualisers. Each one turns audio the browser hears into moving visuals in real time. All are self-contained HTML pages — no build step, no package install, no server-side code.

**Host:** published via GitHub Pages from this repo (`Imtiazbahar/resonance`).
**Audience for this doc:** any developer picking these up to integrate, extend, or maintain.
**Last updated:** 2026-09-24

---

## 1. Contents at a glance

| Page | Path | What it is | Look |
|------|------|-----------|------|
| **Birdsong** (landing) | `index.html` | A 3D "spatial score" of live sound — every point is a frequency peak, placed at a fixed spot in space, with a thread tracing the melody. Full-screen, with a HUD (peaks / pitch / level meters). | Dark, glowing points on black; violet→cyan→green→white by pitch |
| **Agent Orb** | `agent-orb/index.html` | A listening-AI voice visualiser: a liquid-glass marble orb that distorts and reacts to sound (WebGL texture warp over a real photo). | Dark, glassy 3D orb |
| **Cough** (coil) | `cough/index.html` | A "Respiratory Health" check card. Mic-driven sculpture that coils sound into a tight helix. | Mint/teal card, mobile frame |
| **Cough (Bloom)** | `cough-bloom/index.html` | Same card, but the sound folds into a looser organic "bloom" shape. **Colours recently changed to blue** — see its own `cough-bloom/HANDOVER.md`. | Mint/teal card, mobile frame |

> The two Cough pages are near-identical; the difference is the 3D shape (coil vs bloom). **`cough-bloom/` has its own detailed handover file** with the full design spec and tunables — start there for anything Cough-related.

---

## 2. Shared architecture (all pages)

Every page follows the same pattern, so once you understand one you understand all of them:

1. **One HTML file**, containing inline CSS and inline JavaScript.
2. **WebGL** renders the visuals into a `<canvas>` (custom shaders, no 3D library).
3. **Web Audio API** (`AnalyserNode`, `fftSize = 4096`) reads the microphone and runs an FFT (a "fast Fourier transform" — it breaks the sound into its individual frequencies).
4. A shared idea: **pitch → colour** and **pitch → a fixed position in 3D space**, so the same note always lands in the same place and repeated notes pile up into clusters. A "melody thread" line connects the dominant note over time.
5. The camera slowly orbits; louder sound speeds it up. `prefers-reduced-motion` is honoured.

**No audio ever leaves the device.** Everything is analysed in-browser; nothing is recorded, stored, or transmitted.

---

## 3. How to run / preview

Because they use the microphone, these must run in a **secure context**:

- Open the file directly (`file://…`) — fine for local review.
- Serve over **HTTPS** (GitHub Pages — the live host — qualifies).
- Serve from **`localhost`** during development.

They will **not** get microphone access over plain `http://` on a remote host — that's a browser security rule, not a bug.

To test any page: open it → tap the mic / Start button → allow microphone access → make sound (talk, hum, whistle, cough).

---

## 4. Requirements & browser support

| | |
|---|---|
| **Needs** | WebGL, Web Audio API, `getUserMedia`, a secure context |
| **Target** | Mobile Safari (iOS) and Chrome; desktop Chrome/Safari for the landing + orb |
| **Degradation** | If WebGL is missing, an inline message shows and visuals stay blank. If the mic is blocked, an inline error replaces the prompt; the page still loads |
| **Autoplay rule** | Audio only starts after a user tap (browser policy). On iOS the AudioContext is resumed inside that tap — already handled |

---

## 5. External dependencies

Minimal, and different per page:

- **Cough pages** load the **Ubuntu** web font from Google Fonts (`fonts.googleapis.com`). If the target blocks external font CDNs, self-host Ubuntu or rely on the `system-ui` fallback already in place.
- **Birdsong** and **Agent Orb** use only system fonts — no external CSS/JS.
- **Agent Orb** embeds a photo as a data URI, which is why that file is large (~573 KB). No separate image file to ship — it's inside the HTML.

No npm packages, no bundler, no back end anywhere in the repo.

---

## 6. Per-page notes

### `index.html` — Birdsong (landing)
- Full-screen dark canvas with an intro "veil" overlay, a wordmark, a legend, and a live meter console (Peaks / Pitch / Level).
- Pitch colour ramp: violet → cyan → green → white (`const CS`, ~line 219).
- Frequency is coiled into a helix (`anchor()`, ~line 225). Point lifetime `TTL = 30s`.
- Mic button toggles mute/resume after start.

### `agent-orb/index.html` — Agent Orb
- A glassy marble orb (liquid-glass distortion driven by sound, warping a real reference photo baked into the file).
- Same dark theme tokens as Birdsong.
- Large file due to the embedded image — expected; don't "optimise" it away without a replacement asset.

### `cough/index.html` — Cough (coil) & `cough-bloom/index.html` — Cough (Bloom)
- Mobile card component (390 × 736 design frame), mint/teal surface.
- Difference is only the `anchor()` shape function (coil vs bloom).
- **See `cough-bloom/HANDOVER.md`** for the complete design spec (colour tokens, type scale, layout, controls) and the full list of tunable constants. It applies to both Cough pages except the shape and the recent blue palette (bloom only).

---

## 7. Recent changes

**2026-09-24 — Cough Bloom dot colours → range of blues.**
The bloom's pitch→colour ramp was changed from orange/coral/magenta/purple to a wide deep-to-vibrant blue range that reads against the mint card. Documented in full in `cough-bloom/HANDOVER.md` (§8). No other page was affected.

---

## 8. Editing guide (for future changes)

Common things a dev might be asked to change, and where they live in each file:

- **Colours of the visual** → the `const CS = [...]` palette array (search for `colour by pitch`). Interpolated automatically; add/remove stops freely.
- **Shape of the sculpture** → the `anchor(hz)` function.
- **Sensitivity / density** → `PEAK_THRESH`, `SPAWN_K`, `MEL_THRESH` constants near the top of the script.
- **How long trails persist** → `TTL`.
- **Performance caps** → `MAX_PTS`, `MAX_MEL`.
- **Copy / labels** → plain text in the HTML (and the `LABELS` array on the Cough pages).

---

## 9. Placeholders to resolve

- On the Cough pages, the tab labels ("Skin Spot AI", "Skin Health AI") and the "Explore" footer are **static design placeholders** — wire them to real navigation/IA or remove.
- The Cough card is a single component; if it's embedded in the real app shell, the outer `.screen` frame and responsive scaler may be replaced by the host layout.
