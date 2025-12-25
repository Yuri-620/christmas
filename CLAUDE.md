# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a single-page interactive Christmas tree visualization built with Three.js. The project consists of a standalone HTML file that creates a 3D particle system forming a Christmas tree with animated transitions.

## Development

**Running the Application:**
- Open `index.html` directly in a web browser (supports file:// protocol)
- Or serve via any HTTP server: `python3 -m http.server 8000`

**Testing Changes:**
- Simply refresh the browser after editing `index.html`
- Use browser DevTools Console to debug (particle count, animation behavior)

## Architecture

**Core Components:**
- **Particle System**: 800 particles (spheres) with dual position states stored in `userData.treePos` and `userData.scatterPos`
- **Animation Loop**: Uses `requestAnimationFrame` with linear interpolation (Lerp) at 5% per frame for smooth transitions
- **State Management**: Binary mode toggle (`isTreeMode`) controls target positions for all particles

**Particle Distribution:**
- **Tree Mode**: Spiral distribution calculated via ratio-based angle (24π rotations), decreasing radius (6 to 0), and vertical spread (y: -7 to +7)
- **Scatter Mode**: Random cubic distribution (40x40x40 units)

**Color Scheme:**
- Green (0x228b22): Every 3rd particle (i % 3 === 0)
- Gold (0xffd700): Every 3rd+1 particle
- Red (0xcc0000): Every 3rd+2 particle

## Key Technical Details

- No external animation libraries (no GSAP/Tween.js) - uses manual Lerp in animate loop
- Camera positioned at (0, 5, 20) for optimal tree viewing
- Scene rotation at 0.003 rad/frame for ambient motion
- Three.js r128 loaded from CDN (cdnjs.cloudflare.com)
- Chinese UI language ("圣诞魔法世界", "魔法散开", "聚集成树")

## Modification Guidelines

**Changing particle count**: Update `PARTICLE_COUNT` constant (impacts performance/density)

**Adjusting animation speed**: Modify Lerp factor `0.05` in animate() - higher = faster transitions

**Tree shape parameters**:
- `Math.PI * 24`: Controls spiral tightness (rotations)
- `(1 - ratio) * 6`: Base radius scaling
- `ratio * 14 - 7`: Vertical height range

**Adding new particle colors**: Modify modulo logic in color assignment loop (lines 65-66)
