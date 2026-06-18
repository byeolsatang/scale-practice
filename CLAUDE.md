# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Scale Circle is a single-page music scale practice tool for the browser. The entire application lives in one file — `index.html` — with no build system, no package manager, no test framework, and no bundler. To "run" the app, open `index.html` directly in a browser. The live deployment is at `https://scale-practice-delta.vercel.app` (Vercel, auto-deploys from main).

External dependencies load from CDN at runtime:
- **Tone.js** — piano audio synthesis
- **Salamander Grand Piano** MP3 samples — from `https://tonejs.github.io/audio/salamander/`
- **Bravura Text** — SMuFL music notation font (for treble clef rendering)
- **Google Fonts** — Space Mono and Noto Sans JP

## Architecture

`index.html` is structured in three inline sections:

1. **CSS (lines 7–107)** — CSS custom properties on `:root` for dark/light theme colors; `body.light` class toggles the theme. Container is `max-width: 440px` with flex layout targeting mobile use.

2. **HTML (lines 109–164)** — Semantic shell with canvas elements for the circle (`#cwrap`) and staff notation (`staff-sec`), plus tab/button rows for scale/key selection and playback controls.

3. **JavaScript (lines 166–777)** — All procedural, globally scoped (no modules). Key global state vars: `keyIdx` (0–11, current key), `scaleName`, `rotation` (animation angle), `bpm`, `oct`, `isPlaying`, `litNotes` (highlighted note indices).

### JS Subsystems

| Subsystem | Approx. Lines | Description |
|---|---|---|
| Data tables | 166–260 | `MAJKEYS`, `MINKEYS`, `KEY2SEMI`, `NOTENAMES`, `NOTENAMES_FLAT`, `CATEGORIES`, `SCALES`, `FLAT_SCALES` |
| Circle of 5ths | 262–332 | Canvas-rendered rotating ring; inner = major keys, outer = minor; selected key fixed at 12 o'clock |
| Staff notation | 380–489 | 5-line staff + Bravura Text treble clef, ledger lines, accidentals, note stems |
| Audio engine | 515–647 | Salamander piano samples loaded async (progress bar shown); `playbackRate` pitch-shifting; additive synth fallback (7 harmonics) if samples fail |
| Playback engine | 649–705 | `setInterval`-based sequencer; ascending (+ optional descending) scale playback; BPM/octave sliders |
| Circle drag/touch | 717–763 | Pointer/touch drag rotates circle; cubic easing animates snap to nearest key on release |
| Theme toggle | 765–770 | Toggles `body.light`, redraws circle and staff |

### Scale Data Format

Scales are stored as semitone-offset arrays from the root: `[0,2,4,5,7,9,11]` = major. `FLAT_SCALES` is a `Set` of scale names that should default to flat notation. `KEY2SEMI` maps key name strings (`"C"`, `"G#"`, …) to their semitone offset for audio pitch calculation.

## Conventions

- **State → redraw**: All UI is derived from global state; mutation of `keyIdx`, `scaleName`, etc. must be followed by calling the relevant `draw*()` function(s).
- **Canvas for graphics**: The circle and staff are drawn imperatively via Canvas 2D — no SVG or DOM manipulation for those elements.
- **Touch parity**: Every mouse interaction has a corresponding touch event handler; keep them in sync.
- **Flat vs. sharp notation**: Check `FLAT_SCALES` before choosing `NOTENAMES` vs. `NOTENAMES_FLAT` when rendering note names or staff accidentals.
- **Audio fallback**: The synth fallback path (`buildSynth()`) must remain functional in case CDN samples are unavailable.
