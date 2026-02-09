# ∞ Fractal Abyss v2.1.1

A real-time Mandelbrot set explorer that runs entirely in your browser. One HTML file, no dependencies, no build step — just open it and start zooming.

What makes it unusual is how deep it can go. Most browser-based fractal viewers hit a wall around 10^13× magnification because that's where 64-bit floating point numbers run out of precision. Fractal Abyss blows past that barrier using a stack of mathematical tricks — perturbation theory, series approximation, arbitrary-precision arithmetic, and a custom "wide-float" number format — to reach depths that are theoretically unlimited. You can zoom to 10^50× and beyond, all at interactive frame rates on a modern GPU.

The entire thing is roughly 1,550 lines in a single `fractal-zoom.html` file.

---

## Quick Start

Open `fractal-zoom.html` in any modern browser (Chrome, Firefox, Edge, Safari). That's it. No server, no install, no internet required after the initial font load.

You'll see the Mandelbrot set centered on Seahorse Valley. Scroll your mouse wheel to zoom in toward the cursor. Click and drag to pan. Everything else is optional.

---

## What You're Looking At

The Mandelbrot set is the most famous fractal in mathematics. For every point `c` on the complex plane, you repeatedly apply the formula `z = z² + c` starting from `z = 0`. If the value of `z` stays bounded (doesn't fly off to infinity), then `c` is in the set and gets colored black. If it escapes, you color it based on *how quickly* it escapes. That escape speed, mapped through a color palette, produces the intricate patterns you see.

The boundary of the set is infinitely detailed. No matter how far you zoom in, you find new spirals, miniature copies of the whole set, and structures that never repeat. The challenge is computing all of this fast enough for smooth interaction — especially at extreme zoom levels where the numbers involved have hundreds of significant digits.

---

## Controls

### Mouse & Touch

| Action | Effect |
|---|---|
| Scroll wheel | Zoom toward cursor |
| Click and drag | Pan the view |
| Pinch (touch) | Zoom in/out |
| One-finger drag (touch) | Pan |

### Keyboard

| Key | Effect |
|---|---|
| `+` / `-` | Zoom in / out (centered) |
| `P` | Cycle through 8 color palettes |
| `C` | Cycle through 4 coloring modes |
| `J` | Toggle Julia set mode |
| `L` / `K` | Next / previous gallery location |
| `B` | Bookmark current view |
| `Shift+B` | Delete last bookmark |
| `I` | Copy coordinates to clipboard |
| `S` | Save screenshot as PNG |
| `R` | Reset to default view |
| `?` | Toggle help overlay |

### On-Screen Buttons

The toolbar at the bottom mirrors the keyboard shortcuts for touch/mouse-only use.

---

## The HUD (Heads-Up Display)

The top-right corner shows real-time stats about what the renderer is doing:

- **Zoom**: Current magnification. Starts at 1.00× and goes up exponentially. Displayed in scientific notation once it exceeds 10^6.
- **Iters**: Maximum iteration count. The renderer automatically increases this as you zoom deeper — from 300 at the surface up to 8,192 at extreme depth. Higher values reveal finer boundary detail but cost more GPU time.
- **Skip**: How many of those iterations the Series Approximation skipped. If this says 4,000, it means the SA math figured out the first 4,000 iterations analytically and the GPU only had to compute the remaining ones. This is the main performance trick at deep zoom.
- **Prec**: Precision level, shown as MW2, MW3, MW4, etc. This is the number of 64-bit floating-point words used to represent coordinates. MW2 gives ~31 decimal digits of precision, MW3 gives ~47, MW4 gives ~63, and so on. The system automatically allocates more words as you zoom deeper.
- **Depth**: Zoom depth in orders of magnitude (decades). "45d" means you're at roughly 10^45× magnification.
- **Mode**: The rendering pipeline currently active. Possible values:
  - `DF` — Direct double-float iteration (shallow zoom, no perturbation)
  - `PT+SA5` — Perturbation theory with order-5 series approximation
  - `PT+SA7W` — Same, but with wide-float enabled (the `W` suffix). This kicks in at extreme depth where normal float32 numbers underflow.
  - `Julia+SA3` — Julia set mode with perturbation
- **Color**: Current coloring algorithm name.

---

## Coloring Modes

Press `C` to cycle through them:

1. **Smooth** — The classic smooth escape-time coloring. Maps escape speed to a continuous color gradient. This is what most people picture when they think of Mandelbrot visualizations.

2. **Stripe** — Stripe Average Coloring. Adds angular stripes based on the orbit's angle at each iteration, producing a marbled, almost geological look. The average is interpolated at escape to avoid banding.

3. **Trap ○** — Circle Orbit Trap. Colors pixels based on how close their orbit comes to a circle of radius 1.5 centered at the origin. Interior points (which never escape) can get rich coloring from this, unlike escape-time methods which leave them black.

4. **Trap +** — Cross Orbit Trap. Same idea, but measures distance to the real and imaginary axes instead of a circle. Produces cross-shaped patterns radiating from the origin.

All four modes include **distance estimation** edge enhancement. The renderer tracks the derivative of the iteration alongside the orbit, and uses it to estimate how far each pixel is from the set boundary. Pixels near the boundary get darkened, creating crisp visual edges even at modest iteration counts. The darkening fades out at high iteration counts to prevent deep zooms from becoming solid silhouettes.

---

## Color Palettes

Press `P` to cycle through 8 palettes:

1. **Ultra Fractal classic** — Blue-black with amber highlights. The "default Mandelbrot" look.
2. **Rainbow sine** — Smooth RGB sine waves. Vivid and even.
3. **Fire** — Black → red → amber → white. Good for dramatic renders.
4. **Ocean** — Cool-toned variation on the sine palette.
5. **Teal** — Muted blue-green. Calm, good for orbit trap modes.
6. **Spectrum** — Hard primary color bands. Useful for seeing iteration count structure.
7. **Pastel** — Soft complementary palette.
8. **Sepia** — Warm monochrome. Classic engraving look.

The color shift (the starting offset into the palette) can be changed by modifying the `colShift` variable in code, though there's no UI control for it currently.

---

## Gallery Locations

The app ships with 22 preset locations at interesting spots around the Mandelbrot set. Press `L` to step forward through them, `K` to go backward. The names appear as toast notifications.

Some highlights:

- **Seahorse Valley** — The default starting view. The curling spiral structures between the main cardioid and the period-2 bulb.
- **Elephant Valley** — Similar valley on the real axis side, with elephant-trunk-like tendrils.
- **Spiral Arms** — High up on the set boundary where filaments form logarithmic spirals.
- **Needle** — The thin spike extending to the left of the set at x ≈ -1.4. Zooming in reveals miniature copies.
- **Spike Tip** — The rightmost point of the set at x ≈ 0.25. Features extremely dense detail.
- **Western Tip** — The leftmost point at x ≈ -1.94. Another antenna with mini-sets.

When in Julia mode, navigating gallery locations sets the Julia parameter `c` to that location's coordinates instead of moving the view.

---

## Bookmarks

Press `B` to save your current view as a bookmark. The bookmark captures:

- Full multi-word precision coordinates (not just float64 approximation)
- Exact zoom level
- Julia mode state, including the `c` parameter

Bookmarks persist across browser sessions via `localStorage`. They appear after the 22 gallery locations when pressing `L`/`K`. Press `Shift+B` to delete the most recently created bookmark.

Press `I` to copy the current coordinates to your clipboard in a text format that includes the MW-precision word breakdown for exact reconstruction.

---

## Julia Set Mode

Press `J` to toggle Julia mode. Here's what it does:

In the normal Mandelbrot view, every pixel represents a different value of `c` in the formula `z = z² + c`, and we start from `z = 0`. The Julia set flips this: we pick ONE fixed value of `c`, and every pixel represents a different *starting value* of `z`. The resulting pattern is a Julia set, and its shape depends entirely on which `c` you chose.

When you press `J`, the app takes your current Mandelbrot view center as the Julia parameter `c`, then drops you into the Julia set at zoom 1×. Press `J` again to return to the Mandelbrot view at your previous location.

Julia mode is labeled with ⚠ because deep zoom in Julia sets may have stability issues. Basic exploration works well.

---

## How It Actually Works (The Rendering Pipeline)

This is where it gets interesting. The app uses four interlocking mathematical systems to render the Mandelbrot set at arbitrary depth.

### Layer 1: Direct Double-Float Iteration (Shallow Zoom)

At magnifications below about 10^4×, the renderer uses "Mode 0" — straightforward Mandelbrot iteration. But even here, it's not naive. Instead of using single `float32` values, it uses **double-float** arithmetic: each number is represented as a pair of `float32` values (a "hi" part and a "lo" part) whose sum gives roughly 48 bits of mantissa precision, or about 14 decimal digits. This is enough for smooth rendering up to ~10^13× zoom.

The double-float operations (add, subtract, multiply) use Dekker's error-free transformations — algorithms from the 1970s that split each floating-point multiplication into an exact product plus a small error term, then track both. The GPU runs these entirely in the fragment shader.

### Layer 2: Perturbation Theory (Medium Zoom)

Beyond 10^4× magnification, Mode 0 runs out of precision. The pixel spacing becomes smaller than what double-float can represent, and the image turns to noise.

The solution is **perturbation theory**. Instead of computing every pixel's orbit independently, you compute ONE reference orbit at full precision on the CPU, then for each pixel you only compute how that pixel *differs* from the reference. The key insight is that neighboring pixels in a deep zoom are almost identical — their orbits differ by a tiny perturbation `δ`. The recurrence relation for the perturbation is:

```
δ_{n+1} = 2·Z_n·δ_n + δ_n² + δc
```

where `Z_n` is the reference orbit (known) and `δ_n` is the perturbation (small). Since `δ` is small, this can be computed in ordinary float precision even when `Z` and `c` themselves require hundreds of digits.

The CPU computes the reference orbit using multi-word arithmetic (see Layer 4), packs it into a floating-point texture, and uploads it to the GPU. The GPU fragment shader then runs the perturbation recurrence per-pixel using double-float math.

**Glitch detection**: Sometimes the perturbation grows large relative to the reference (typically near mini-sets), causing visual artifacts like orange circles. The shader detects this (`|Z_full|² < |δ|² × 0.001`) and falls back to full double-float iteration for those pixels using the known full-precision `Z` value. This is called "rebasing."

### Layer 3: Series Approximation (SA)

Even with perturbation theory, each pixel still needs to iterate through thousands of steps. Series Approximation eliminates most of them.

The idea: at the start of the orbit, all pixels in the view are close together, so their perturbations are well-approximated by a polynomial in `δc` (the pixel's offset from the reference point):

```
δ_skip ≈ A·δc + B·δc² + C·δc³ + ... + G·δc⁷
```

The CPU computes the coefficients A through G alongside the reference orbit. At each iteration, it checks whether the highest-order term is still small enough relative to the pixel scale. The last iteration where the approximation is valid becomes `saSkip` — the shader jumps directly to that iteration and starts the perturbation loop from there.

The order is adaptive: the system tries order 7 first, falls back to 6, 5, 4, or 3 as coefficients overflow or become inaccurate. At deep zoom with high iteration counts, SA can skip 90%+ of iterations, turning a 5,000-iteration render into a 500-iteration one.

### Layer 4: Wide-Float Perturbation (Extreme Zoom)

At depths beyond ~10^27 (where the pixel scale is smaller than 10^-27), even the perturbation `δ` underflows `float32`. Normal floats can't represent numbers smaller than about 10^-38, and at these depths the initial pixel perturbations are far smaller.

The solution is **wide-float**: represent `δ` as a mantissa (float32) times 2^exponent (int32). The mantissa stays in a normal range (roughly 0.5 to 2.0), and the exponent tracks the actual magnitude. This gives unlimited dynamic range while staying within WebGL 2's integer and float capabilities.

The wide-float perturbation loop looks like this:

1. Initialize: `δ = A · pixelScale · pixelOffset` (from SA), split into mantissa + exponent
2. Each iteration: compute `2·Z·δ + δc` with exponent alignment (shifting mantissas to match exponents before adding)
3. Renormalize: adjust exponent so mantissa stays near 1.0
4. Check escape: if exponent ≥ 4 (meaning `|δ| ≥ 16`), the point has escaped
5. Transition: when exponent ≥ -20, `δ` is large enough for normal float32. Switch to standard perturbation for the remaining iterations.

Wide-float activates automatically with hysteresis: it turns on when the pixel-scale exponent drops below -90, and turns off when it rises above -60. This prevents mode oscillation near the threshold.

### Layer 4: Multi-Word Arithmetic (CPU-Side Precision)

The reference orbit must be computed at full precision. At 10^45× zoom, you need ~50 decimal digits of precision for the coordinates. Standard 64-bit floats give you ~16 digits.

Multi-Word (MW) arithmetic solves this by representing each number as an array of N `float64` values. The key insight (originally from Dekker and later refined by others) is that you can add two float64 values and recover the *exact* rounding error as a separate float64. By chaining these "error-free transformations," you build up arbitrary precision.

- **MW2** (N=2): ~31 decimal digits. Enough for zoom depths up to ~10^25.
- **MW3** (N=3): ~47 digits. Good to ~10^41.
- **MW4** (N=4): ~63 digits. Good to ~10^57.
- And so on. The system auto-sizes N based on zoom depth.

Multiplication uses fast paths for small N: DD (double-double) at N≤2, TD (triple-double) at N=3, QD at N=4, with a general O(N²) loop for N≥6. The DD fast path is roughly 4× faster than the general case, which matters because reference orbit computation runs this multiplication thousands of times per frame.

---

## Reference Orbit Selection

The quality of the render depends heavily on choosing a good reference point — one whose orbit doesn't escape too early. If the reference escapes at iteration 500 but some pixels need 3,000 iterations, those pixels lose their reference and may render incorrectly.

At shallow zoom (pixel scale > 10^-15), the system searches a grid of candidate points across the viewport and picks the one with the longest orbit.

At deep zoom, the system strongly prefers the view center as the reference point, because:
- Any offset from center degrades wide-float accuracy (the shader uses only the high-precision parts of SA coefficients)
- Reference switches between frames cause visible view jumps
- The center is usually a good reference in deep zoom (you tend to be zoomed into a non-escaping region)

If the center escapes too early, a tight grid search (±25-50 pixels from center) looks for a better candidate. If nothing survives the full iteration count but the center makes it to 80%, the center is used anyway.

---

## Performance Characteristics

The main performance bottleneck shifts depending on depth:

**Shallow (0-15d)**: GPU-bound. Double-float iteration in the fragment shader is the limiting factor. 300-400 iterations per pixel at full resolution. Essentially instant on any modern GPU.

**Medium (15-30d)**: Balanced. SA skip eliminates most GPU work. CPU reference orbit computation takes a few milliseconds. Smooth interactive zooming.

**Deep (30-50d)**: CPU-bound. MW3-MW4 reference orbit computation takes 50-500ms. Each multiplication involves 3-6 `tp()` calls (Dekker products). The orbit buffer (computed 2,000-3,000 iterations ahead of current maxIter) helps amortize this — you can zoom in several steps before needing a recompute. But when a recompute does happen, there's a visible stall.

**Extreme (50d+)**: Heavily CPU-bound. MW5+ arithmetic, O(N²) multiplication, thousands of iterations. Each reference orbit can take multiple seconds. The app remains usable but zooming feels sluggish. This is where a Web Worker or WASM implementation would help.

---

## File Structure

Everything lives in one HTML file:

| Line Range | Contents |
|---|---|
| 1–35 | HTML structure, CSS (HUD, toolbar, help overlay, toast notifications) |
| 36–70 | HTML body: canvas, HUD elements, control buttons, help card |
| 71–85 | WebGL context setup, float texture extension detection |
| 86–90 | Vertex shader (trivial full-screen quad) |
| 91–450 | **Fragment shader** (GLSL 300 es) — the core renderer |
| 450–500 | Shader compilation, program linking, uniform locations |
| 500–510 | Reference orbit texture setup |
| 510–620 | Multi-word arithmetic library |
| 620–660 | Application state, constants, gallery locations |
| 660–800 | Location system (gallery, bookmarks, serialization) |
| 800–840 | Reference selection (shallow grid search) |
| 840–1050 | Reference orbit computation, SA coefficients, orbit caching |
| 1050–1170 | `render()` — uniform upload, mode selection, draw call |
| 1170–1200 | HUD update |
| 1200–1230 | `applyZoom()` — zoom-toward-cursor with cancellation-free math |
| 1230–1380 | Julia toggle, PNG export, reset, coordinate copy |
| 1380–1550 | Input handlers (mouse, touch, keyboard, button zoom) |
| 1550–1554 | Boot sequence |

### The Fragment Shader In Detail

The shader is the heart of the renderer. It has three main paths:

**Mode 0** (lines ~91–200 in the GLSL): Direct double-float Mandelbrot/Julia iteration. Used when zoom is shallow enough that perturbation isn't needed. Includes cardioid/bulb check for early bailout of the main Mandelbrot body.

**Mode 1, Standard** (lines ~200–380): SA initialization (computes the polynomial `A·δc + B·δc² + ... + G·δc⁷` for each pixel), then perturbation loop using double-float math. Includes glitch detection and rebasing.

**Mode 1, Wide-Float** (lines ~200–340): SA initialization using raw (unscaled) coefficients with separate mantissa/exponent tracking. Perturbation loop with int-exponent arithmetic. Handles δc addition in both directions (when δ > δc and when δ < δc). Transitions to standard when exponent normalizes.

**Phase 2 Fallback** (lines ~340–380): If the reference orbit escapes before maxIter and the pixel hasn't escaped yet (and we're not in wide-float mode at extreme depth), falls back to direct double-float iteration for the remaining iterations.

**Coloring** (lines ~380–430): Smooth iteration count, stripe averaging, orbit traps, and distance estimation, all combined into the final pixel color.

---

## Known Issues

These are documented here so you know what to expect, not as bugs that prevent use:

- **Orange circle artifacts at ~23d**: A known glitch in standard perturbation where `δ` becomes comparable to `Z_ref`. The glitch detection catches most cases, but some slip through. A proper fix would involve multi-reference rebasing.

- **Performance stalls at 40d+**: Reference orbit recomputation blocks the main thread. The orbit buffer helps but doesn't eliminate all stalls. A Web Worker implementation would fix this.

- **Julia mode deep zoom**: Labeled "⚠" for a reason. Basic Julia exploration works, but deep zoom in Julia sets may have reference selection or precision issues.

- **Depth 75d+**: "Permanent" wide-float territory where the perturbation never becomes large enough to transition to standard double-float. The renderer handles this, but it's less tested than the transition zone.

---

## Browser Compatibility

**Required**: WebGL 2 with float texture support. This covers:
- Chrome 56+ (2017)
- Firefox 51+ (2017)
- Edge 79+ (2020)
- Safari 15+ (2021)

There is a WebGL 1 fallback path (the shader is auto-downgraded), but it loses perturbation/SA/wide-float capabilities and is limited to shallow zoom.

The app uses the `Space Mono` font from Google Fonts for the UI. If the font CDN is unavailable, the browser falls back to the system monospace font. This is the only external resource; everything else is self-contained.

---

## Version History

### v2.1.1 (Current)
Camera stability fixes for depth 45-55d:
- Cancellation-free zoom math using `Math.expm1` instead of scale subtraction
- Tighter reference drift thresholds at deep zoom (2px in wide-float mode)
- Center-pinned reference tracking to prevent unnecessary recomputes
- Depth-scaled orbit buffer (grows with zoom depth instead of fixed +2000)
- Explicit orbit length validation before cache reuse

### v2.1
- Wide-float perturbation for depths beyond 27d
- `psMant` bug fix in wide-float initialization
- Unified center-first reference selection for deep zoom
- Adaptive SA order (3–7) with progressive degradation
- Orbit buffer (+2000 iterations ahead)
- 22 gallery locations, persistent bookmarks
- 4 coloring modes, 8 palettes
- Distance estimation edge enhancement
- PNG export, Julia set mode

---

## Technical Glossary

| Term | Meaning |
|---|---|
| **DF** | Double-float. A pair of float32 values whose sum gives ~48-bit precision. |
| **MW** | Multi-word. N × float64 arrays giving ~15.7N decimal digits of precision. |
| **PT** | Perturbation theory. Compute one reference orbit at full precision, then compute each pixel as a small perturbation from it. |
| **SA** | Series approximation. Skip the first N iterations analytically using a polynomial in δc. |
| **Wide-float** | Mantissa × 2^exponent representation that avoids float32 underflow at extreme depth. |
| **δ (delta)** | The perturbation — the difference between a pixel's orbit and the reference orbit. |
| **δc** | A pixel's offset from the reference point in complex-plane coordinates. |
| **Reference orbit** | The single orbit computed at full MW precision. All pixels are computed relative to this. |
| **Glitch** | An artifact where perturbation theory breaks down because δ is too large relative to Z_ref. |
| **Rebase** | Recovering from a glitch by switching from perturbation arithmetic to full-value arithmetic. |
| **Escape** | When |z|² exceeds 256 (the bailout radius). Points that escape are outside the Mandelbrot set. |
| **psE** | Pixel scale exponent. `floor(log2(pixelScale))`. Below -90 triggers wide-float mode. |
| **Depth / decades** | `log10(zoom)`. "45d" means 10^45× magnification. |
| **Dekker's algorithm** | A method to split a float64 multiplication into an exact product plus an error term, both representable as float64. The foundation of all the extended-precision arithmetic here. |

---

MIT License
