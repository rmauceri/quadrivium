# Quadrivium — Implementation Plan

## Architecture

Single-page webapp (`index.html`), no build step, no package manager. Hosted on GitHub Pages. Open the file in a browser and it works.

**Libraries (CDN-loaded):**
- **[Rough.js](https://roughjs.com/)** — hand-drawn/sketchy shape rendering. Gives us the pencil-sketch aesthetic out of the box: wobbly lines, fill hachure, stroke variation. Eliminates the need to build a custom sketchy renderer from scratch.
- **[Tone.js](https://tonejs.github.io/)** — musical abstractions over Web Audio API. Synths, effects (reverb, compressor, filter), scales, smooth parameter ramping. Lets us think in musical terms instead of raw oscillator wiring.

Both are small, well-maintained, and load from CDN with a single `<script>` tag.

## Code Organization

The file will be organized into clearly-labeled sections (matching the wunderkammer's `=== SECTION ===` banner convention):

1. **CONSTANTS & CONFIG** — colors, dimensions, audio tuning, shape definitions
2. **AUDIO ENGINE** — Web Audio setup, oscillator management, spatial mixing, proximity harmonics
3. **SHAPE SYSTEM** — shape types, their visual and sonic identities, breathing animation
4. **SKETCHY RENDERER** — hand-drawn line drawing: wobbly strokes, construction marks, pencil texture
5. **DRAFTING TABLE** — surface rendering: grain, grid, environmental details
6. **INTERACTION** — mouse/touch input: drag, place, remove, palette
7. **SCENE DRAWING** — draw order, compositing, z-sorting
8. **MAIN LOOP** — requestAnimationFrame loop, state updates

## Implementation Todos

### 1. Canvas & Drafting Table Surface
- Responsive canvas filling the viewport (`100vw × 100vh`), resizing on window change
- Dark warm background (deep walnut/indigo, not pure black)
- Subtle surface grain texture (pre-baked stipple, similar to wunderkammer wood grain)
- Faint dot grid pattern suggesting a drafting surface
- Environmental details: a faint compass rose in a corner, a smudge mark or two

### 2. Sketchy Renderer (Rough.js)
- Initialize Rough.js canvas overlay (it draws to an existing canvas)
- Configure global defaults: roughness, bowing, stroke width for the pencil-sketch feel
- Per-shape-type drawing configs: fill style (hachure, cross-hatch, or solid), fill weight, stroke color
- Construction marks drawn with Rough.js lines: faint center dots, radius tick marks
- Rough.js handles the wobbly strokes, corner overshoot, and organic feel natively

### 3. Shape Types
Four initial shape types, each with distinct visual and sonic character:

| Shape | Visual | Sonic character | Breathing |
|-------|--------|----------------|-----------|
| **Circle** | Warm ochre/amber sketch | Smooth sine tone, warm fundamental | Gentle radius pulse, slow |
| **Triangle** | Pale rose/coral sketch | Bright, harmonic-rich (triangle wave + overtones) | Subtle rotation wobble |
| **Square** | Dusty blue/slate sketch | Rhythmic pulse, percussive click pattern | Slight scale throb, metronomic |
| **Pentagon** | Soft violet/lavender sketch | Ethereal, detuned chorus (multiple near-unison sines) | Slow rotation drift |

Each shape has:
- `type` — circle, triangle, square, pentagon
- `x, y` — position on table
- `size` — radius/dimension (slightly randomized on creation)
- `color` — base hue (from type, with slight random variation)
- `phase` — breathing animation phase (randomized start)
- `sketchSeed` — random seed for consistent hand-drawn appearance (so the wobble doesn't change every frame)

### 4. Audio Engine (Tone.js)
- Initialize Tone.js on first user interaction (`Tone.start()` — handles browser autoplay policy)
- Each shape owns a Tone.js synth instance:
  - Circle: `Tone.Synth` with sine waveform + `Tone.LFO` on volume for gentle throb
  - Triangle: `Tone.AMSynth` or `Tone.Synth` with triangle wave + `Tone.Filter` (bandpass)
  - Square: `Tone.MetalSynth` or short-envelope `Tone.Synth` triggering on a slow `Tone.Loop`
  - Pentagon: `Tone.FatOscillator` (built-in detuned unison chorus)
- **Pitch mapping:** vertical position → frequency. Map to a pentatonic scale (A minor pentatonic — modern, clean, no dissonant intervals within the scale itself; proximity interactions supply the tension)
- **Volume mapping:** shape size → synth volume via `synth.volume.rampTo()`
- **Proximity harmonics:** for each pair of shapes within threshold distance:
  - Compute interval between pitches
  - If near-consonant (octave, fifth, fourth), boost a subtle shared harmonic via a quiet auxiliary oscillator
  - Near-unison beating happens naturally with close frequencies
  - Closer = stronger interaction gain
- Master chain: `Tone.Compressor` → `Tone.Reverb` (warm, medium decay ~2-3s, wet ~0.3 — starting point, will tweak) → `Tone.Destination`
- All parameter changes use Tone.js `.rampTo()` for smooth, click-free transitions

### 5. Shape Palette
- Not a toolbar — a small cluster of "template" shapes sitting in a corner of the table, slightly faded/ghostly
- Drag a template shape off the palette to create a new instance
- The palette shapes don't produce sound (or produce a very faint whisper)
- Visually: smaller, lighter stroke, with a faint "pull me" quality

### 6. Interaction (Pointer Events — mouse & touch unified)
- Use `pointer` events throughout (`pointerdown`, `pointermove`, `pointerup`) — no separate mouse/touch code paths
- **Drag to move:** pointerdown on shape → track pointer → update position → audio params update in real time
- **Create:** drag from palette (clone template, full opacity, start audio)
- **Remove:** drag off the edge of the table, or double-tap to dismiss (shape fades out, audio fades)
- Hit targets: minimum 44px effective radius at any viewport scale (finger-friendly)
- Hit testing: distance-based for circles, point-in-polygon for others
- Z-order: last-touched shape drawn on top
- All sizes/distances scale relative to shorter viewport dimension

### 7. Proximity Visualization
- When two shapes are within interaction range, draw a faint sketchy line between them
- Line opacity/weight proportional to interaction strength (closer = more visible)
- Line style: dashed or dotted, same hand-drawn quality
- Optional: subtle shared glow/halo where interaction is strong

### 8. Initial Scene
- On load, place 3-4 shapes already on the table in a pleasing arrangement
- They're already sounding — the visitor arrives to an ambient hum
- Arrangement chosen to demonstrate proximity effects (two shapes close, one farther away)
- This is the "you open it and it's alive" moment

### 9. Breathing Animation
- Each shape has a gentle periodic animation tied to its sonic character:
  - Circle: radius pulses ±3% on a slow sine wave (~0.3 Hz)
  - Triangle: rotates ±2° on a slow wobble
  - Square: scales ±2% with a slightly faster, more rigid rhythm
  - Pentagon: rotates very slowly, continuously (like drifting)
- Phase is randomized per shape so they're not synchronized
- Breathing rate could subtly tie to the shape's pitch (higher = slightly faster)

### 10. Visual Polish
- Subtle dust motes or floating particles in the dark surround (wunderkammer DNA)
- Shapes cast a very faint "shadow" or ground mark on the table
- When dragging, shape lifts slightly (larger, lighter) — the "picked up" feel
- Smooth fade-in on creation, fade-out on removal

## Build Order

The plan is to build in layers, each layer functional before moving to the next:

1. **Canvas + table surface** — get the visual ground right
2. **Sketchy renderer** — the hand-drawn line/shape primitives
3. **Static shapes** — draw the four types on the table with breathing animation
4. **Drag interaction** — move shapes around, touch support
5. **Audio engine** — each shape makes its sound, pitch mapped to position
6. **Proximity system** — interaction lines, harmonic blending
7. **Shape palette + create/remove** — full interaction loop
8. **Initial scene** — the pre-populated starting arrangement
9. **Polish** — particles, shadows, fade transitions, reverb tuning

## Responsive & Touch-First Design

The piece must work on phone screens and with touch, not just desktop and mouse. This is a design constraint from the start, not an afterthought.

**Layout:**
- Canvas fills the viewport (`100vw × 100vh`, no scroll). No fixed pixel dimensions.
- All shape sizes, hit targets, and interaction distances scale relative to the shorter viewport dimension (so the table "zooms" on small screens rather than cropping).
- The palette repositions itself sensibly at small widths (e.g., bottom edge instead of corner).

**Touch interaction:**
- Touch-drag for moving shapes (same as mouse drag, `pointer` events unify both).
- Long-press or drag-from-palette to create. Double-tap to remove.
- Hit targets must be finger-sized (~44px minimum at any viewport scale).
- Multi-touch is a stretch goal but not required for v1 (one shape at a time is fine).

**Design concerns (honest):**
- **Audio on mobile:** iOS and Android both gate audio behind a user gesture. Tone.js handles this with `Tone.start()`, but we'll need a clear "tap to begin" moment — which actually fits the aesthetic (tap the drafting table to wake it up).
- **Screen real estate:** On a phone, 4-5 shapes will feel crowded. That's OK — fewer shapes, more intimate. The soundscape should still be interesting with 2-3 shapes.
- **Precision:** Fat fingers on small screens can't do fine positioning. Shapes should be forgiving — no interactions that require pixel-perfect placement. The spatial audio mapping should have enough "grain" that moving a shape 20px in any direction produces a noticeable change.
- **Performance:** Rough.js redraws are more expensive than plain canvas. On low-end phones we may need to cache shape drawings to offscreen canvases and only redraw on change (not every frame).

## Open Questions (not blocking — just noted)

- Reverb character: starting warm/medium-diffuse (~2-3s decay, ~0.3 wet). Will tweak by ear.
- Dot grid is purely aesthetic (not functional snap-to-grid).
