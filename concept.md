# Quadrivium

## The name

The *quadrivium* was the medieval curriculum that unified four disciplines: arithmetic, geometry, music, and astronomy — treating them as facets of the same underlying truth. Geometry and music are the same thing, expressed differently. This project takes that literally.

## What it is

A spatial audio playground built as a single-page webapp (`index.html`). No build step, no package manager. CDN-loaded libraries are fair game. Hosted on GitHub Pages. Open it in a browser and it works.

Geometric shapes live on a dark drafting table. Each shape has a sonic identity — it breathes, hums, pulses. The sound it makes is shaped by what it is, where it is, and what's near it. You drag shapes around, place new ones, remove them. The arrangement *is* the composition.

There is no sequencer, no timeline, no score. The soundscape is purely spatial and ambient — a living thing that responds to how you arrange the geometry. Proximity between shapes creates harmonics, interference, resonance. The relationships between shapes matter more than the shapes themselves.

## Aesthetic

**Hand-drawn, not digital.** Shapes look sketched — slightly wobbly lines, visible stroke weight variation, the imperfection of a compass that slipped a little. Think 2B pencil on heavy paper, or chalk on a dark surface. Construction marks linger: ghost center points, faint radius arcs.

**The drafting table.** The canvas is a surface of creation, not a finished product. Dark and warm — not sterile black, more like aged walnut or deep indigo. Subtle grain/tooth in the surface. Faint environmental details that say "someone was here before you": a dot grid, a smudge, a compass rose. It's a workspace that expects things to be rearranged.

**Luminous, not glowing.** Shapes are matte and sketched in character, but they pulse with soft warmth when sounding — the light comes from within the stroke lines, like chalk catching lamplight. Active resonance between shapes might show as faint connecting threads or shared halos.

**Sibling to the wunderkammer.** The pencil-sketch aesthetic creates kinship with the puppet theater without repeating it. Same DNA, different organism.

## Interaction

Works on phones and desktops alike — touch-first, responsive canvas that fills the viewport.

You open it and shapes are already there — a few geometric forms arranged on the table, already sounding a gentle ambient hum. The world is alive before you touch it.

- **Drag** shapes to move them. Sound changes continuously as you reposition.
- **Add** new shapes from a minimal palette (not a toolbar — something that belongs on the table).
- **Remove** shapes to simplify the soundscape.
- Shapes **breathe** — gentle rhythmic animation tied to their sonic character.
- **Proximity** between shapes creates emergent audio: harmonics, beating, interference patterns.
- **No instructions.** Discovery-based. You touch things and learn.

## Sound design

Web Audio API via Tone.js, synthesized in real time. No samples.

**Tonal character:** Distinctive and modern — not the wunderkammer's kalimba/folk palette. Think ambient electronic: clean sine fundamentals, glassy FM harmonics, sub-bass warmth, crystalline highs. Rich enough to complement the visual complexity of the geometry, but never busy. The sound should feel like the shapes are *resonating*, not performing.

- Each shape type has a distinct sonic character (see plan.md for specifics).
- Vertical position on the table influences pitch register (high = higher frequency).
- Shape size influences presence/volume.
- Color may influence timbre or harmonic content.
- When two shapes are close, their tones interact — beat frequencies from near-unison, consonant intervals from sympathetic ratios, tension from dissonance.
- The overall soundscape should feel organic, not mechanical. Slow modulation, gentle drift.
- Reverb: starting warm and medium-diffuse, expect to tweak. Not cathedral-huge, not dry — something that gives the sounds a shared space.

## What this leaves open

This first turn establishes aesthetic and core interaction. It deliberately does **not** answer:

- What the shapes "mean" (alchemical elements? constellations? something else entirely?)
- Whether time/rhythm/sequence enters the picture
- Whether there's a goal, puzzle, narrative, or score
- What other interactions are possible beyond place/move/remove
- Whether shapes have relationships beyond proximity (orbits? bonds? repulsion?)
- What lives at the edges of the table
- Whether the table itself can change

These are seams, not gaps. They're invitations.

---

## Collaboration context

This is a creative collaboration between two friends of 30+ years: **rmauceri** and **gulley** (GitHub: gulley). They take turns making changes, both vibe-coding with AI. Neither knows exactly what any project will become — the work is part art, part discovery, part exploration.

**Gulley** built the original wunderkammer (the predecessor to this project). He's a professional software engineer, active writer, and student of mathematics, art, theater, and music. He values originality, the absurd, and humor. He's fascinated by pseudo-science, alchemy, astrology, and the esoteric. His wife tragically passed from cancer a few years ago; they were high school sweethearts.

**rmauceri** has a personal interest in architectural rendering and layered generative systems. In the wunderkammer he built the procedural play system, the reactive audience, the mood-driven audio, and mobile support.

**Creative principles:**
- Favor the surprising and original over the polished and expected
- Leave creative seams — don't over-engineer or close off directions the other collaborator might pull on
- The tone is playful on the surface, serious craft underneath
- Embrace absurdity and humor
- Both collaborators are technically strong — code can be clever and expressive
- A turn should set aesthetic and concept strongly enough to build *on*, not away from
