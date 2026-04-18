# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A single-page static website for **Geek Coders**, a tech club at KRCT (est. 2026). No build step, no bundler, no framework — just `index.html` served directly in a browser.

## Development

Open `index.html` directly in a browser, or serve it locally:

```bash
# Any static server works, e.g.:
npx serve .
python -m http.server 8080
```

There are no tests, no lint commands, and no CI pipeline.

The `node_modules/` directory contains only `sharp`, which was used one-time for JPEG → WebP image conversion and is not part of the runtime.

## File Layout

- **`index.html`** — the entire site (~2600 lines). All HTML, CSS, and JavaScript live here.
- **`geek-coders-v3.html`** — previous stable snapshot; useful as a reference when reverting.
- **`cd*.webp` / `cd*.jpeg`** — team and event photos. `.webp` versions are used in `<img>` tags; `.jpeg` originals are kept as source.
- **`apply_*.js`, `fix_*.js`, `*.py`** — one-off patch scripts from prior development iterations. They are **not** part of the build or runtime; safe to ignore.

## Architecture of `index.html`

### Sections (top to bottom)

| Section | ID | Description |
|---|---|---|
| Scramble Hero | `.scramble-hero` | Full-height intro with fluid particle canvas and text scramble effect |
| Hero | `.hero` | Full-height group photo with overlaid "GEEK CODERS" typography and stats strip |
| Events / Hall of Fame | `#achievements` | Event cards (currently one: DBA Internship) |
| Team / Crew | `#team` | 6-member grid, each card has a per-member WebGL smoke canvas |
| About + CTA + Footer | `#about` | Two-column split section; footer is embedded here |

### CSS Design Tokens

```css
--ink:   #0a0a0a   /* near-black background */
--chalk: #f0ede6   /* warm off-white text */
--mid:   #6b6560   /* muted mid-tone */
--neo:   #00ffcc   /* neon cyan accent */
```

Fonts: **Unbounded** (900) for display headings, **Outfit** (300/400/500) for body, **DM Mono** (300) for mono.

### JavaScript / Animation Layers

1. **Global smoke canvas** (`#smokeCanvas`, fixed, `z-index: -1`) — native WebGL2. The `WebGLSmokeRenderer` class is defined inline. Renders a full-screen FBM noise smoke shader tinted `#00ffcc`.

2. **Fluid particles** (`#fluidCanvas` inside `.scramble-hero`) — Perlin-noise particle system defined in the `<script id="fluidParticlesJS">` block.

3. **Per-crew-member smoke** (`.crewSmokeCanvas` elements, one per `.tc` card) — reuses the same `WebGLSmokeRenderer` class but with custom `data-color` RGB values per member.

4. **Three.js shader lines** (`#shaderLinesContainer`, inside the President card only) — loaded from CDN (`three.js r89`). Renders animated GLSL line patterns.

5. **Text scramble** — `TextScramble` class triggers on `.scramble-allowed` elements via `IntersectionObserver`.

6. **Reveal animations** — `.reveal` class + `IntersectionObserver`; adds `.vis` class when elements enter viewport.

### Mobile Handling

`@media (max-width: 768px)` overrides handle: 2-column stats strip in hero bottom, hiding `#heroShaderContainer` and `#shaderLinesContainer`, stacking nav, and reducing font sizes. The shader canvases are hidden on mobile to avoid performance issues.

### z-index Stack (Hero Section)

```
z-index 1  — hero image / group photo
z-index 2  — optional shader overlay
z-index 3  — atmospheric gradient overlay (.hero-photo-inner::before)
z-index 4  — hero text wrap
z-index 100 — hero bottom strip
```
