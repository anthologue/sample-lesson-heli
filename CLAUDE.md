# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository is a single self-contained static web page, `index.html` — an interactive educational lesson titled "AeroFlight: The Aerodynamic Theory of Helicopters." There is no build system, package manager, server, or test suite; the entire site (markup, styling, and behavior) lives in this one file.

## Running / Previewing

There is nothing to build or install. Open `index.html` directly in a browser, or serve it locally, e.g.:

```
python3 -m http.server 8000
```

then visit `http://localhost:8000/index.html`.

There are no lint, test, or build commands configured for this repo.

## Architecture

`index.html` is structured as one long document with three layers, all inline:

1. **Styling**: Tailwind CSS is loaded via the CDN `<script src="https://cdn.tailwindcss.com">` and used utility-first throughout the markup. A small `<style>` block in `<head>` defines the two custom fonts (`Inter` for body text, `JetBrains Mono` for the `.mono`/monospace UI elements), scrollbar theming, and a couple of gradient/accent helper classes (`.rotor-blur`, range input accent color).

2. **Content sections** (in `<main>`, each an `<section id="...">` anchored from the top nav): rotary-wing lift theory, flight controls & swashplate mechanics, anti-torque solutions, advanced rotor dynamics (dissymmetry of lift, gyroscopic precession), special flight regimes (ground effect, vortex ring state, autorotation), the interactive simulator, and a quiz. Anchor links in the header nav (`#theory-rotary`, `#theory-controls`, etc.) must stay in sync with these section `id`s.

3. **Behavior**: a single `<script>` block at the end of the file containing three independent interactive pieces, all driven by plain DOM APIs (no framework):
   - **Airfoil pitch inspector** (Section 1): a range slider (`#airfoil-pitch-slider`) rotates an inline SVG airfoil and toggles a stall warning above 15°.
   - **Flight simulator** (`#simulator`): the core piece. A `sim` state object holds all physical parameters (collective pitch, RPM, mass, rotor radius, density altitude) and dynamic state (visual altitude, vertical speed, rotor angle). `computeAeroPhysics()` recomputes lift/weight/power/torque each frame from `sim` using simplified actuator-disc/blade-element formulas (air density by barometric approximation, blade lift coefficient, ground-effect multiplier via a Cheeseman–Bennett-style approximation, induced + profile power). `render()` is the `requestAnimationFrame` loop: it integrates vertical velocity/position from net force, then redraws the whole scene on `<canvas id="flightCanvas">` (sky/ground, downwash streamlines, helicopter body, spinning rotor disc, force vectors) before calling `updateTelemetryUI()` to refresh the HUD/telemetry DOM text and calling itself again. Sliders (`#slider-pitch`, `#slider-rpm`, `#slider-mass`, `#slider-radius`, `#slider-alt`) mutate `sim` directly via `input` listeners; `setPreset(name)` / `resetDefaults()` apply canned scenarios (hover, climb, high-altitude, autorotation) and call `syncInputs()` to reconcile slider positions and labels with `sim`.
   - **Quiz** (final section): `checkAnswer(button, isCorrect, feedbackId)` is called from inline `onclick` handlers on answer buttons; it toggles Tailwind classes for correct/incorrect styling and reveals feedback text.

## Working in this file

- Everything is one file — when editing, use exact string matches / line numbers to target the right section rather than assuming component boundaries.
- Keep the top-nav anchor hrefs, section `id`s, and any DOM `id`s referenced from the `<script>` block consistent — the script looks up elements by `id` with `document.getElementById` and will silently fail (or throw) if markup IDs are renamed without updating the script, or vice versa.
- Physics constants and formulas in `computeAeroPhysics()` (e.g. `bladeChord`, `cd0`, the `1.85` thrust fudge factor, the ground-effect cap of `1.28`) are simplified/approximate for educational illustration, not aerospace-accurate — keep changes consistent with the surrounding simplified model rather than introducing full blade-element-theory rigor.
- Tailwind classes are applied via the CDN's JIT compiler at page load; there is no Tailwind config file to edit.
