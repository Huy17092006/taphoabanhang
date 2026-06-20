# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Vũ Trụ Magic Hand** — a single-file interactive web experience (Vietnamese UI) where a user's hand gestures, detected via webcam, control a cosmic space animation. The entire application lives in `index.html` (~765 lines). There is no build step, no package manager, and no test framework.

## Running Locally

Serve the file over HTTP (required for camera API and CDN scripts):

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

Any static HTTP server works (`npx http-server`, `npx serve`, etc.). Opening `index.html` directly as a `file://` URL will break camera permissions and CDN loading.

## Architecture

All logic is in `index.html`. The sections are ordered as follows:

| Line range | Section |
|------------|---------|
| 7–145 | CSS — glassmorphism dark theme, gradient animations |
| 147–177 | HTML — video feed, two canvases, overlay/start screen |
| 182–248 | Background canvas animation (nebula, stars, milky way) |
| 249–311 | Utility functions: `lerp`, `clamp`, `easeOutBack`, `roundRect` |
| 312–368 | `Particle` class + `burst()`, `ShootingStar` class |
| 373–495 | `OrbRing`, `CosmicDust`, `StarBall` — ambient visual objects |
| 496–562 | `PhotoCard` — orbiting image display when hand is open |
| 566–605 | `drawSkeleton()` and `getOpenness()` — hand landmark rendering and gesture detection |
| 614–705 | Main `requestAnimationFrame` render loop |
| 708–763 | `startApp()` — MediaPipe Hands + Camera initialization |

### Key Data Flow

```
Webcam → MediaPipe Hands (CDN) → landmark array (20 joints)
    → getOpenness() → 0 (fist) or 1 (open hand)
    → Main loop renders either StarBall (fist) or PhotoCard gallery (open)
    → Gesture transitions trigger burst() particle effects
```

### Gesture States

- **No hand**: idle orbit animation, status "👋 Đưa tay vào vũ trụ"
- **Closed fist (`openness` < threshold)**: star ball forms, particles attract inward
- **Open hand (`openness` ≥ threshold)**: photo cards orbit outward, particles burst out

### External Dependencies (CDN only)

```html
https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js
https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js
```

MediaPipe is initialized with `maxNumHands: 1`, `modelComplexity: 1` inside `startApp()`. The error from a failed MediaPipe init is caught and displayed in `#status`.

### Image Assets

11 image files (`.jpg`/`.webp`) in the root directory are preloaded into an array (`images[]`) at startup. They display as `PhotoCard` instances orbiting the palm position when the hand is open. `startApp()` polls with `setInterval` until all images are loaded before starting the camera.

## Development Conventions

- **Single file**: keep all changes inside `index.html`. Do not introduce a build pipeline unless the scope of the project fundamentally changes.
- **No framework**: use plain Canvas 2D API and vanilla JS. Avoid adding npm dependencies.
- **Vietnamese strings**: UI text, status messages, and comments are in Vietnamese — preserve this.
- **Canvas layers**: the background starfield renders on `#bg-canvas`; all interactive content renders on the foreground `<canvas>` (the one without an id). Keep these separate.
- **Smooth transitions**: use `lerp()` for position smoothing and `easeOutBack()` for entrance animations to maintain the existing feel.

## Branch

Active development branch: `claude/claude-md-docs-n2lh2j`
