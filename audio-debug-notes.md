# Quadrivium Audio Debug Notes

## Current State (2026-04-01)

**Working:** Edge (no dev tools) — all 5 shape types produce sound, including hex kick and square tick rhythm.  
**Broken:** Edge with dev tools open (no sound at all), iPad (no sound), Android (no sound).  
Interaction (dragging shapes) works on all platforms.

## Root Cause Analysis

### The core tension
Tone.js v14.7.77 **auto-creates an AudioContext at script load** (before any user gesture). Browsers increasingly block this — the context starts `suspended` and on strict platforms (mobile Safari, mobile Chrome, Edge with dev tools) may refuse to resume it even when `resume()` is called later from a gesture handler.

### What we tried and why each failed

| Approach | Result | Why it failed |
|----------|--------|---------------|
| `Tone.start()` in gesture handler (current code) | Works on Edge, fails mobile + dev tools | The auto-created context may be permanently "poisoned" on strict platforms — created without gesture = won't resume |
| `Tone.setContext(new AudioContext())` to replace with gesture-blessed context | Rhythm loops never fired | **Known Tone.js bug:** `Tone.Destination` stays bound to the OLD context. `.toDestination()` routes to a dead context. See [issue #1037](https://github.com/Tonejs/Tone.js/issues/1037) |
| `async/await` in onDown handler | Froze all interaction | If `Tone.start()` promise hangs (which it does on poisoned contexts), the async handler never returns, blocking all pointer events |
| Close auto-created context, then setContext | `Tone.start()` hangs forever | Closing the context that Tone.js is bound to leaves it in a broken state |

### Key discoveries from research

1. **`Tone.setContext()` breaks Destination routing** — This is the fundamental reason rhythm didn't work with setContext. The Transport/Loop *might* have been fine, but audio was routed to the old context's destination. ([GitHub #1037](https://github.com/Tonejs/Tone.js/issues/1037))

2. **You CAN build an audio graph while context is suspended** — Per Web Audio spec and confirmed by Tone.js docs, creating nodes, connecting them, even calling `triggerAttack` while suspended is fine. Everything activates when context resumes. ([MDN AudioContext.resume()](https://developer.mozilla.org/en-US/docs/Web/API/AudioContext/resume))

3. **iOS 17.5.1+ has a visibility bug** — Web Audio contexts get "interrupted" when tab loses/gains focus. Need a `visibilitychange` handler to call `resume()`. ([WebKit bug #276016](https://bugs.webkit.org/show_bug.cgi?id=276016))

4. **iOS mute switch silences Web Audio but not HTML audio** — Playing a silent MP3 via `<audio>` element can "unlock" the audio pipeline. ([Audjust blog](https://www.audjust.com/blog/unmute-web-audio-on-ios))

5. **`Tone.context` vs `Tone.getContext()` behave differently** — Using the wrong one can result in silent audio. ([GitHub #1129](https://github.com/Tonejs/Tone.js/issues/1129))

## Recommended Fix (Not Yet Implemented)

### Approach: Build graph synchronously, let context resume catch up

```js
function initAudio() {
  if (audioOK) return;
  audioOK = true;

  // 1. Fire-and-forget Tone.start() — triggers internal resume
  Tone.start();

  // 2. Belt-and-suspenders: also resume raw context directly in gesture
  const raw = Tone.context.rawContext;
  if (raw && raw.state !== 'running') raw.resume();

  // 3. Build entire graph SYNCHRONOUSLY — don't wait for .then()
  //    Nodes work on suspended context; audio starts when context resumes
  Tone.Transport.bpm.value = 120;
  Tone.Transport.start();
  masterGain = new Tone.Gain(volumeVal).toDestination();
  shapes.forEach(s => attachSynth(s));
  recomputeProximity();

  // 4. Handle iOS visibility bug
  document.addEventListener('visibilitychange', () => {
    if (document.visibilityState === 'visible' && raw) raw.resume();
  });
}
```

**Why this should work:**
- No `setContext` = Destination stays correct
- No `.then()` = no hanging promise can block the graph build
- `resume()` called synchronously in gesture handler = browser should allow it
- Graph built on suspended context = everything starts when context catches up
- `visibilitychange` handler = survives iOS tab switching

### Additional hardening

- Move `initAudio()` call BEFORE `e.preventDefault()` in `onDown` — some browsers may consider `preventDefault` as breaking the gesture chain for audio unlock purposes
- Add document-level `click` and `touchstart` listeners as fallback unlock triggers
- Consider the silent `<audio>` MP3 trick for iOS mute switch edge case

## Nuclear Option: Drop Tone.js

If the above still doesn't work on mobile, consider replacing Tone.js entirely with raw Web Audio API:

- **Wunderkammer proves raw Web Audio works on iPad** — it uses `new AudioContext()` in gesture handler, raw oscillators, gain nodes, no library
- Tonal shapes: raw `OscillatorNode` + `GainNode` (simple)
- Rhythm shapes: `setInterval` or `AudioContext.currentTime` scheduling (medium complexity)
- Lose Tone.js conveniences (Synth presets, Transport, Loop, effects) but gain full control
- Could keep Tone.js for desktop only with raw fallback for mobile, but that's messy

## Files

- `index.html` — the entire app, single file
- `concept.md` — creative vision for the collaboration
- `plan.md` — original implementation plan
- Live: https://rmauceri.github.io/quadrivium
- Repo: https://github.com/rmauceri/quadrivium
