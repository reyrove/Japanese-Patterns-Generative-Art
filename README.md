# Japanese-Patterns-Generative-Art

A seed-based generative art catalogue for **Japanese Patterns**, a computational traditional-motif system. Built as a single-file HTML page with p5.js, styled as a gallery-ready portfolio and licensing surface.

**Live:** `https://reyrove.github.io/Japanese-Patterns-Generative-Art/`

---

## Overview

**Japanese Patterns** renders eight traditional Japanese motifs from numeric seeds. Each composition picks one of eight motifs — Seigaiha, Asanoha, Shippou, Kikkou, Sayagata, Uroko, Tatewaku, Ichimatsu — and tiles it across a 60 × 60 grid. A curated soft palette of four colours and a background hue is drawn from one of six curated sets, all rendered at three transparency levels for depth.

The page functions as a **catalogue and licensing surface** for fashion houses, textile studios, and surface designers. Every composition is reproducible by seed, meaning the same number always produces the identical pattern.

---

## Features

### Generative Engine (p5.js)

- **60 × 60 tile grid** — 3,600 cells per composition, each lit or dark per the motif's rule
- **Eight traditional Japanese motifs**:
  - **Seigaiha** — waves, drawn as flattened hexagons on a sine-thresholded field
  - **Asanoha** — hemp leaf, drawn as hexagons on an 8 × 8 block grid
  - **Shippou** — seven treasures, drawn as overlapping circles from a radial field
  - **Kikkou** — tortoise shell, drawn as hexagons on a diagonal diamond grid
  - **Sayagata** — interlocking keys, drawn as squares on a checkerboard
  - **Uroko** — fish scales, drawn as diamonds on a 16 × 8 block grid
  - **Tatewaku** — rising steam, drawn as vertical rectangles on a sine threshold
  - **Ichimatsu** — checkerboard, drawn as squares on a coarse 6-cell grid
- **Six curated palettes** — Soft Blues, Gentle Greens, Warm Earth, Soft Purples, Ocean Blues, Sunset
- **Three transparency levels** — 40%, 60%, 80% alpha, assigned per-cell by a deterministic hash
- **Subtle gradient overlay** — a bottom-darkening wash for depth
- **Optional outlines** — for Asanoha, Kikkou, and Uroko, a faint contour adds crispness

### Same Seed → Many Surfaces

A single seed drives every instance on the page:

| Instance | Purpose | Tiling |
|----------|---------|--------|
| **Cover** | Hero composition | 1 × 1 |
| **Plate** | Main work (master) | 1 × 1 |
| **Frame** | Framed archival print | 1 × 1 |
| **Print** | Archival print surface | 1 × 1 |
| **Scarf** | Silk scarf (3:1) | 3 × 1 |
| **Textile** | Fabric yardage (4:3) | 2 × 2 |
| **Wall** | Wallpaper (2:3) | 1 × 2 |
| **Archive** | 8 unique seeds | 1 × 1 each |

### Controls

- **New Seed** button / `R` key — regenerate with a fresh seed
- **Download** button / `S` key — export the plate canvas as PNG
- **Click any archive plate** — load that seed into the main view

### Performance

- Debounced resize handling (250ms)
- `IntersectionObserver` pauses off-screen sketches
- `visibilitychange` pauses rendering when the tab is hidden
- `pixelDensity(1)` for a lighter canvas footprint
- Two-frame render (`maxFrames: 2`) so tiles draw once and stop

---

## File Structure

```
Japanese-Patterns-Generative-Art/
├── index.html          # Single-file page (structure + styles + generative engine)
├── README.md
└── images/
    ├── fav.svg         # Favicon
    ├── tote.png        # Mockup: tote bag
    ├── tee.png         # Mockup: t-shirt
    └── cushion.png     # Mockup: cushion
```

No build step. No dependencies to install. p5.js is loaded from a CDN.

---

## Getting Started

1. Clone the repo:
   ```bash
   git clone https://github.com/reyrove/Japanese-Patterns-Generative-Art.git
   cd Japanese-Patterns-Generative-Art
   ```
2. Add the three mockup images to `images/` (optional — they appear in the Commission section).
3. Open `index.html` in a browser.

That's it. The page self-initialises on `DOMContentLoaded`.

---

## How the Generative System Works

### Seeded Random

A linear congruential generator replaces p5's native randomness inside each sketch:

```js
class SeededRandom {
  constructor(s) { this.state = (Math.abs(s) % 2147483647) || 1; }
  rand() { this.state = (this.state * 9301 + 49297) % 233280; return this.state / 233280; }
  range(min, max) { return this.rand() * (max - min) + min; }
  int(min, max) { return Math.floor(this.range(min, max + 1)); }
  pick(arr) { return arr[Math.floor(this.rand() * arr.length)]; }
}
```

Every aesthetic decision — motif, palette, per-cell colour index, per-cell transparency — is drawn from this generator in a fixed order. Same seed, same sequence, same composition.

### Parameter Space (per seed)

| Parameter | Range |
|-----------|-------|
| Grid size | 60 × 60 |
| Motif | 8 options (Seigaiha → Ichimatsu) |
| Palette | 6 options (Soft Blues → Sunset) |
| Palette colours | 4 (from chosen palette) |
| Transparency levels | 3 (0.4, 0.6, 0.8) |
| Cell scale | 0.95 of cell size |

### The Eight Motifs

Each motif is defined by two functions: a *grid rule* (which cells to light) and a *shape function* (how to draw a lit cell). The grid rules are:

| Motif | Grid Rule |
|-------|-----------|
| **Seigaiha** | `sin(x · 12) > 0.2` — wave-sliced field |
| **Asanoha** | `(i % 8 < 4 && j % 8 < 4)` — 8 × 8 block quadrants |
| **Shippou** | `sin(distance · 8 + angle · 3) > -0.2` — radial field |
| **Kikkou** | `(i + j) % 12` in `(3, 9)` — diagonal band |
| **Sayagata** | `(i % 12 < 6) !== (j % 12 < 6)` — checkerboard |
| **Uroko** | `i % 16` in `(4, 12)` and `j % 8` in `(2, 6)` — scale band |
| **Tatewaku** | `sin(x · 10) · 0.5 + 0.5 > y` — sine threshold |
| **Ichimatsu** | `floor(i / 6) % 2 === floor(j / 6) % 2` — coarse checkerboard |

The shape functions are drawn as polygonal primitives — waves as flattened hexagons, Asanoha and Kikkou as hexagons, Shippou as stacked circles, Uroko as diamonds, and Sayagata, Tatewaku, Ichimatsu as rectangles.

### The Six Palettes

Each palette is a curated pair: a background HSB triple, plus four companion HSB triples at slightly different hue, saturation, and brightness. The palettes are:

```js
const PALETTES = [
  [[210, 10, 95], [[210, 40, 85], [200, 35, 90], [220, 30, 88], [230, 25, 92]]],  // Soft Blues
  [[120, 15, 94], [[130, 40, 80], [110, 35, 85], [140, 30, 90], [100, 45, 75]]],  // Gentle Greens
  [[30, 20, 92],  [[20, 50, 80],  [40, 45, 85],  [10, 55, 75],  [50, 40, 88]]],  // Warm Earth
  [[280, 15, 93], [[270, 40, 82], [290, 35, 87], [260, 45, 78], [300, 30, 90]]],  // Soft Purples
  [[190, 15, 92], [[180, 45, 78], [200, 40, 83], [170, 50, 75], [210, 35, 87]]],  // Ocean Blues
  [[350, 20, 91], [[340, 50, 80], [10, 45, 85],  [330, 55, 75], [20, 40, 88]]]    // Sunset
];
```

Each palette pairs a light background hue with four mid-tone companions — a harmonic family of five colours in total.

### Transparency and Depth

Each lit cell picks its transparency level via a deterministic hash:

```js
const alphaIndex = (i * 5 + j * 7) % TRANSPARENCY_LEVELS.length;
const alpha = TRANSPARENCY_LEVELS[alphaIndex];
```

The three levels are 0.4, 0.6, 0.8. Because the hash is deterministic (not random), the transparency distribution is regular — producing a subtle interleaving of light and dark cells rather than a chaotic scatter.

### Tiling Logic

Rectangular surfaces (scarf, textile, wall) tile the square composition N×M. Each tile is drawn into its own translated coordinate space, centred within its cell:

```js
const tileW = canvasW / tileCols;
const tileH = canvasH / tileRows;
const tileSize = Math.min(tileW, tileH);
// draw square of side tileSize at (ox, oy) for each cell
```

This preserves the square composition's proportions regardless of surface aspect ratio.

---

## Customisation

### Change the default seed

Edit the init function:

```js
function init() {
  seed = 123456789;  // fixed seed instead of random
  // ...
}
```

### Adjust the grid resolution

Edit the constant near the top of the script:

```js
const GRID_SIZE = 60;
```

Higher values (100, 120) produce finer, denser patterns. Lower values (30, 40) produce chunkier, more architectural motifs.

### Add a new motif

1. Add the name to `MOTIF_NAMES`:
   ```js
   const MOTIF_NAMES = ['Seigaiha', ..., 'YourNewMotif'];
   ```
2. Add a `case` in the grid-generation switch inside `drawComposition`:
   ```js
   case 8: // YourNewMotif
     keepCell = /* your grid rule */;
     break;
   ```
3. Add a `case` in the render switch:
   ```js
   case 8:
     // YourNewMotif
     p.rect(...) or drawCustomShape(p, x, y, size);
     break;
   ```
4. Update `sr.int(0, 7)` to `sr.int(0, 8)`.

### Add a new palette

Extend the `PALETTES` array with a new `[background, [colour1, colour2, colour3, colour4]]` entry, and extend `PALETTE_NAMES` with a matching name. The seed will automatically pick it up.

### Adjust the transparency levels

```js
const TRANSPARENCY_LEVELS = [0.4, 0.6, 0.8];
```

Add more levels for finer gradation, or make them all equal for a uniform pattern.

### Adjust the transparency hash

```js
const alphaIndex = (i * 5 + j * 7) % TRANSPARENCY_LEVELS.length;
```

The coefficients `5` and `7` control the diagonal rhythm of the transparency pattern. Change them for different regularities.

### Change the gradient overlay

```js
const alpha = (i / cs) * 0.03 + 0.02;
```

- Change `0.03` to alter how much the gradient strengthens from top to bottom.
- Change `0.02` to shift the baseline alpha across the whole canvas.

### Change surface tiling

Edit `surfaceDefs`:

```js
const surfaceDefs = [
  { id: 'surfacePrint',   wrapId: 'wrap-surfacePrint',   tile: { cols: 1, rows: 1 } },
  { id: 'surfaceScarf',   wrapId: 'wrap-surfaceScarf',   tile: { cols: 3, rows: 1 } },
  // ...
];
```

The `tile` object accepts any `{ cols, rows }` pair.

### Replace mockup images

Swap the files in `images/` or update the `src` attributes in the Commission section:

```html
<img src="images/your-mockup.png" alt="..." loading="lazy" />
```

---

## Design System

| Token | Value | Use |
|-------|-------|-----|
| `--bg` | `#0a0a0a` | Page background |
| `--paper` | `#f4f1ea` | Light surface (Surfaces section) |
| `--gold` | `#b8935a` | Primary accent |
| `--gold-soft` | `#d4b483` | Italic emphasis |
| `--serif` | Cormorant Garamond | Headings, titles |
| `--mono` | DM Mono | Labels, metadata, UI |

Type scale, spacing, and section rhythm match the Skyline, Portrait, Girih, Gothic Grid, Order to Chaos, Wallpaper Groups, Wandering Tiles, Food Grid, Framed Line, Deconstructed Stage, and Layered Diagram templates. Responsive breakpoints at 900px, 720px, 560px, and 400px.

---

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `R` | Regenerate with a new seed |
| `S` | Download plate as PNG |
| `Enter` / `Space` | Activate focused archive item |
| `Esc` | Close legal modal |

---

## Browser Support

Modern evergreen browsers with:

- Canvas 2D (universal)
- `IntersectionObserver` (universal in modern browsers)
- `backdrop-filter` (Safari 9+, Chrome 76+, Firefox 103+)
- CSS custom properties (universal)

Fallback: if `IntersectionObserver` is unavailable, sketches render continuously.

---

## Accessibility

- Archive items are keyboard-focusable with `role="button"` and `aria-label`
- Legal modal traps focus, closes on `Esc` and overlay click
- `prefers-reduced-motion` disables transitions and smooth scroll
- All interactive elements have visible focus states (via browser default)

---

## Section Map

| Section | ID | Purpose |
|---------|-----|---------|
| Cover | — | Hero title + live pattern |
| Statement | `#statement` | Artist statement |
| Plate | `#work` | Main work + metadata + controls |
| Surfaces | `#application` | Framed print + 4 tiled surfaces |
| Process | `#process` | Three process cards |
| Archive | `#archive` | 8 selectable seeds |
| Commission | `#commission` | Services + mockups + CTA |
| Colophon | — | Footer, links, legal |
| Legal modal | `#legalModal` | Licensing / Terms / Credits |

---

## Deployment (GitHub Pages)

1. Push the repo to GitHub.
2. Go to **Settings → Pages**.
3. Under **Source**, select `Deploy from a branch`.
4. Choose branch `main` and folder `/ (root)`.
5. Save — the site goes live at `https://reyrove.github.io/Japanese-Patterns-Generative-Art/`.

---

## Notes on the Source Sketch

The generative engine is adapted almost verbatim from the original p5 sketch. The adaptations that turn it into a catalogue page:

1. **Seeded random replaces p5's `random()`** — a local `SeededRandom` class drives every choice. The original sketch calls `setSeeds()` for fxhash compatibility; that's replaced with the seeded generator directly.
2. **Sketch factory** — `makeSketch({ slotId, seedValue, ... })` returns a p5 sketch that can be instantiated into any DOM slot, enabling cover / plate / frame / surfaces / archive from one seed.
3. **DOM-driven canvas sizing** — `cs = min(windowHeight, windowWidth)` becomes slot-rect dimensions.
4. **Tiling support** — the `tile: { cols, rows }` parameter repeats the square composition across rectangular surfaces.
5. **Drawing logic takes `p` explicitly** — the original used p5's global mode; here `p` is passed so the factory closure works.
6. **`TWO_PI` replaced with `Math.PI * 2`** — for clarity inside the factory closure.
7. **`map(i, 0, height, 0.02, 0.05)` inlined** — replaced with `(i / cs) * 0.03 + 0.02`, mathematically identical.
8. **Motif drawing helpers extracted** — `drawWave`, `drawHexagon`, `drawCircle`, `drawScale` are top-level functions that take `p` as a parameter, so they work from any sketch instance.
9. **`windowResized` replaced by a debounced `handleResize`** — all sketches are destroyed and recreated on resize (250ms debounce), since their canvases are locked to slot dimensions.
10. **`triggerPreview` removed** — the original calls it for fxhash thumbnail generation; not needed in the plain-browser catalogue.

---

## Related Repositories

| Repo | Theme |
|------|-------|
| [`Skyline-Series`](https://github.com/reyrove/Skyline-Series) | Themed cityscapes |
| [`Portrait-Series-Generative-Art`](https://github.com/reyrove/Portrait-Series-Generative-Art) | Generative faces |
| [`Girih-2-Generative-Art`](https://github.com/reyrove/Girih-2-Generative-Art) | Islamic geometric ornament |
| [`Gothic-Grid-Generative-Art`](https://github.com/reyrove/Gothic-Grid-Generative-Art) | Mosaic grid composition |
| [`Order-to-Chaos-Generative-Art`](https://github.com/reyrove/Order-to-Chaos-Generative-Art) | Gradient grid composition |
| [`Wallpaper-Groups-Generative-Art`](https://github.com/reyrove/Wallpaper-Groups-Generative-Art) | Crystallographic symmetry |
| [`Wandering-Tiles-Generative-Art`](https://github.com/reyrove/Wandering-Tiles-Generative-Art) | Line-drawing composition |
| [`Food-Grid-Generative-Art`](https://github.com/reyrove/Food-Grid-Generative-Art) | Illustrated food grid |
| [`Framed-Line-Generative-Art`](https://github.com/reyrove/Framed-Line-Generative-Art) | Gesture line composition |
| [`Deconstructed-Stage-Generative-Art`](https://github.com/reyrove/Deconstructed-Stage-Generative-Art) | Suprematist composition |
| [`Layered-Diagram-Generative-Art`](https://github.com/reyrove/Layered-Diagram-Generative-Art) | Layered linework composition |
| **`Japanese-Patterns-Generative-Art`** | Traditional motif composition *(this repo)* |

All twelve share the same catalogue template, design tokens, and seed-based reproducibility model.

---

## Credits

- **Design & Generative System** — Reyhaneh Daneshdoost
- **Typefaces** — Cormorant Garamond, DM Mono
- **Engine** — p5.js 1.9.0
- **Motif Reference** — Seigaiha · Asanoha · Shippou · Kikkou · Sayagata · Uroko · Tatewaku · Ichimatsu
- **Edition** — Japanese Patterns, Autumn 2026

---

## License

All artwork, code, and generative systems are the intellectual property of the artist. Compositions may not be reproduced, redistributed, resold, or adapted without a written license.

For licensing, commissions, or private systems: **reyhanehdaneshdoost@gmail.com**