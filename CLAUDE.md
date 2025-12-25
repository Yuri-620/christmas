# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a gesture-controlled 3D Christmas tree visualization built with Three.js and MediaPipe Hands. The single-page application (`index.html`) creates an interactive particle system that responds to hand gestures captured via webcam, with optional photo upload functionality to incorporate user images into the particle system.

## Development

**Running the Application:**
- Open `index.html` directly in a browser (requires HTTPS or localhost for webcam access)
- Recommended: `python3 -m http.server 8000` then navigate to `http://localhost:8000`
- **Note:** Webcam permissions required for gesture control to function

**Testing Changes:**
- Refresh browser after editing `index.html`
- Use browser DevTools Console to debug (check particle count, gesture detection, MediaPipe status)
- Monitor console for MediaPipe loading errors or camera access issues

## Architecture

**State Management:**
- Three states: `'folded'` (Christmas tree), `'scattered'` (particles dispersed), `'photoZoom'` (photo close-up)
- State transitions triggered by hand gestures detected via MediaPipe
- Global config object `CONFIG` at index.html:177 controls particle count (800), colors, lerp speed (0.08), camera distance (28)

**Particle System:**
- 800 total particles with three position states stored in `userData`:
  - `foldedPos`: Layered conical Christmas tree (9 layers, height 14 units, base radius 6)
  - `scatteredPos`: Random spherical distribution (radius 15-30 units)
  - Photo particles use PlaneGeometry with uploaded textures
- Three particle types: spheres (70%), cubes (15%), small light points (15%)
- All particles use manual Lerp interpolation at 8% per frame (`CONFIG.lerpSpeed`)

**Tree Structure (Folded State):**
- Top 5% of particles form star tip (ratio > 0.95)
- Remaining 95% distributed across 9 horizontal layers
- Each layer has conical radius: `6.0 - (layerProgress * 5.5)` units
- Particles biased toward outer edges with slight downward droop effect
- Layered approach at index.html:464-535 creates realistic tree silhouette

**Color Scheme:**
- All-gold aesthetic with gradient based on vertical position:
  - Top (star region): `0xffffdd` (bright white-gold), emissive intensity 2.0
  - Upper layers: `0xffd700` (bright gold), emissive intensity 1.2
  - Middle layers: `0xffaa00` (orange-gold), emissive intensity 1.0
  - Lower layers: `0xffe066` (pale gold), emissive intensity 0.9
- Photo particles have gold emissive edges (0xffd700, intensity 0.5)

**Gesture Recognition (MediaPipe Hands):**
- Initialized at index.html:591 with single hand tracking
- Three gestures detected (index.html:650-720):
  1. **Fist (握拳)**: All fingertips within 0.2 units of wrist → triggers 'folded' state
  2. **Open Hand (张开)**: All fingertips beyond 0.28-0.35 units from wrist → triggers 'scattered' state + camera rotation based on hand movement
  3. **Pinch (捏合)**: Thumb-index distance < 0.05 units → triggers 'photoZoom' for first uploaded photo
- 100ms throttle on gesture detection to prevent jitter

**Rendering Pipeline:**
- Three.js r128 + EffectComposer with UnrealBloomPass for glow effects
- Bloom parameters at index.html:271: strength 2.0, radius 1.0, threshold 0.1
- ACESFilmic tone mapping with exposure 1.5 for enhanced luminosity
- Camera at (0, 5, 28) looking at (0, 4, 0) with 60° FOV

**Photo Upload System:**
- Accepts up to 20 images via file input (index.html:311)
- Creates PlaneGeometry particles with uploaded textures
- Photo particles have MeshStandardMaterial with metalness 0.3, semi-transparent (opacity 0.8)
- Photos integrated into particle array, reducing geometric particle count accordingly

**UI Components:**
- Chinese language interface
- Upload panel with "选择照片" (select photos) and "跳过，直接体验" (skip) buttons
- Live webcam preview (200x150px, bottom-right, mirrored via `scaleX(-1)`)
- Gesture status display and instructions overlay

## Key Technical Details

- **Dependencies (CDN):**
  - Three.js r128 (core + post-processing modules)
  - MediaPipe Hands + Camera Utils
  - All loaded from CDN, no build step required

- **Camera Control:** Hand movement in 'scattered' state rotates camera around tree via atan2 angle calculation (index.html:691-702)

- **Animation:** Scene auto-rotates at 0.002 rad/frame only in 'folded' state (index.html:738)

- **Performance:** 800 particles with bloom effects - may require GPU acceleration for smooth 60fps

## Modification Guidelines

**Adjusting particle count:** Modify `CONFIG.particleCount` at index.html:178 (impacts performance significantly)

**Changing animation speed:** Modify `CONFIG.lerpSpeed` at index.html:185 (currently 0.08 = 8% interpolation per frame)

**Tree shape modifications:**
- Layer count: `numLayers` at index.html:470 (currently 9)
- Tree height: `treeHeight` at index.html:468 (currently 14 units)
- Base radius formula: `6.0 - layerProgress * 5.5` at index.html:501

**Color customization:** Modify `CONFIG.colors` object at index.html:179-184 and conditional color assignment at index.html:402-420

**Gesture sensitivity:**
- Fist detection threshold: index.html:668 (currently 0.2 units)
- Open hand threshold: index.html:681-685 (0.28-0.35 units)
- Pinch threshold: index.html:709 (currently 0.05 units)

**Bloom intensity:** Modify UnrealBloomPass parameters at index.html:271-275 (strength, radius, threshold)
