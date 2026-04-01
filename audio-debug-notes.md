# Quadrivium Audio Debug Notes

## Resolution (2026-04-01)

**FIXED — audio works on all platforms (Edge desktop, Edge touch, iOS Safari, Android Chrome) on first touch.**

### What we did
Dropped Tone.js entirely and rewrote the audio engine with raw Web Audio API.

### Root causes (two separate issues)

**1. Tone.js auto-creates AudioContext at script load**
Tone.js v14 internally calls `new AudioContext()` when its `<script>` tag executes — before any user gesture. Strict mobile browsers permanently block contexts created outside gestures. Every workaround failed:

| Approach | Result |
|---|---|
| `Tone.start()` in gesture handler | Context already poisoned — `resume()` ignored |
| `Tone.setContext(new AudioContext())` | Breaks `Tone.Destination` and `Tone.Transport` routing ([#1037](https://github.com/Tonejs/Tone.js/issues/1037)) |
| `await Tone.start()` | Promise hangs forever on poisoned context |
| Deferred eval (load Tone.js source as text, eval in gesture) | Browser doesn't grant gesture-blessing to AudioContexts created inside `eval()` |

**Solution:** Replace Tone.js with raw Web Audio API. `new AudioContext()` created directly in the gesture handler — no library intermediary.

**2. `pointerdown` from touch is NOT a valid user activation for AudioContext**
Even with raw Web Audio, the context stayed `suspended` on touch. The diagnostic showed `ctx: suspended | t: 0.00` — the context never resumed.

The event sequence on touch devices is:
1. `pointerdown` fires first → `initAudio()` creates AudioContext → **stays suspended** (pointerdown from touch is not a valid user activation)
2. `touchstart` fires second → `audioOK` already true → **skips init**

Only `touchstart` and `click` are valid user activations for AudioContext on touch devices. `pointerdown` from a mouse works, but `pointerdown` from touch does not.

**Solution:** The `touchstart` handler calls `actx.resume()` even when `audioOK` is already true, catching the case where `pointerdown` already ran `initAudio` but the context is stuck suspended.

### Raw Web Audio replacements for Tone.js

| Tone.js | Raw Web Audio |
|---|---|
| `Tone.Synth` (sine) | `OscillatorNode` + `GainNode` |
| `Tone.FMSynth` | Two `OscillatorNode`s (modulator → carrier frequency) |
| `Tone.NoiseSynth` | `AudioBufferSourceNode` with noise buffer + `BiquadFilterNode` |
| `Tone.MembraneSynth` | `OscillatorNode` with `frequency.exponentialRampToValueAtTime` |
| `Tone.Tremolo` | LFO `OscillatorNode` → `GainNode` on gain param |
| `Tone.Chorus` | `DelayNode` with LFO on `delayTime` |
| `Tone.Phaser` | `BiquadFilterNode` (allpass) with LFO on frequency |
| `Tone.Loop` / `Tone.Transport` | `setInterval` |
| `.rampTo()` | `param.linearRampToValueAtTime()` |
| `.toDestination()` | `node.connect(actx.destination)` |

## Current audio architecture (post-session)

### Signal chain
Each synth → per-shape `GainNode` (volume knob) → `masterGain` → dry/wet split:
- Dry: `GainNode(0.75)` → `destination`
- Wet: `ConvolverNode` (generated IR, 3.5s decay) → `GainNode(0.35)` → `destination`

### Anti-drone design
Tonal shapes (circle, triangle, pentagon) use **swell cycling** — each independently fades in over 3-7s, holds 2-5s, fades to near-silence over 3-7s, rests 2-7s, then repeats. All timings randomized per cycle. Additional slow pitch drift (~10 cents) and FM depth modulation prevent static timbre.

### Per-shape volume
Edge-drag on shape rim rotates a volume knob (3 full turns = full range). Mouse wheel also works. `volAngle` property on each shape, mapped: `clamp(1 + volAngle / (3π), 0, 2)`.

## Files

- `index.html` — the entire app, single file
- `concept.md` — creative vision for the collaboration
- `plan.md` — original implementation plan
- Live: https://rmauceri.github.io/quadrivium
- Repo: https://github.com/rmauceri/quadrivium
