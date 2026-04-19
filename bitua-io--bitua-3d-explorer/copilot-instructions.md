## bitua-3d-explorer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

# Bitua 3D Explorer

## Project Overview
Interactive web-based 3D product visualization for the **Bitua Smart Cabinet Series** (medical/laboratory equipment). Users can explore the cabinet in 3D, interact with features (door, biometric reader, lock, camera), and simulate product storage capacity.

## Tech Stack
- **Three.js v0.162.0** — 3D rendering (loaded via CDN)
- **GSAP** — smooth camera transitions and door animations (loaded via CDN)
- **Vanilla JS ES6 modules** — no framework, no build step
- **GLB/GLTF** — 3D model format with DRACO compression support

## Architecture
The entire application lives in a single file: `index.html` (56 KB). There is no build system, no package.json, no bundler. All dependencies are loaded from CDN at runtime.

### Key Assets
- `Cabinet v2.5 con eje.glb` — main 3D model (~5.7 MB)
- `GUI_Standby.png` — texture applied to the GUI screen mesh (solid1_61)

### Responsive Design
The app is mobile-responsive with three breakpoints:
- **0–480px** (Mobile): Bottom-sheet sidebar UI with floating action button (FAB)
- **481–768px** (Tablet): Narrower sidebar (240px), optimized spacing
- **769px+** (Desktop): Original layout with right-side panel (280px)

See `RESPONSIVE_DESIGN_IMPLEMENTATION.md` for detailed CSS changes (lines 506–767 in `index.html`).

## Key Systems (all inside index.html)

### 1. Scene & Rendering
- WebGLRenderer with MSAA, PCFSoftShadowMap shadows, ACESFilmicToneMapping
- PerspectiveCamera (45° FOV), OrbitControls with damping

### 2. Material System
Materials are assigned by matching mesh names (regex patterns):
- Glass trays → MeshPhysicalMaterial (transmission: 1.0, IOR: 1.5)
- Green polycarbonate → MeshPhysicalMaterial (transmission: 0.95, green tint)
- Lock/hinges → MeshStandardMaterial (metalness: 1.0)
- LED bar → emissive MeshStandardMaterial (emissiveIntensity: 10)

### 3. Door/Hinge Animation
- Door meshes (names containing "PUERTA") are reparented into a THREE.Group pivot at the hinge axis
- GSAP animates rotation −120° (−2.09 rad) around Y-axis, 1.2s "power2.inOut"

### 4. Interaction System
Raycasting on click detects:
- **Lock** (solid1_102, CERRADURA, CILINDRO) → zoom + annotation
- **Biometric readers** (solid1_114, solid1_37–41) → zoom + auto-open door
- **Facial recognition** (solid1_103) → zoom + annotation
- **Camera** (solid1_105, MESHFEATURE1) → zoom + annotation
- **Door/hinges** (PUERTA, BISAGRA) → toggle door open/closed
- **GUI/Console** (solid1_61) → info panel

### 5. Annotation System
Semi-transparent callout panels with SVG leader lines connecting to 3D objects in screen space. Updates position every frame.

### 6. Capacity Simulator
Right sidebar with product dimension inputs. Spawns blue semi-transparent boxes inside the cabinet using a grid-packing algorithm. Interior dimensions: {x: 0.8, y: 1.6, z: 0.5} meters.

### 7. Debug Overlay
Press `D` to toggle camera position/target display with copy-to-clipboard.

## Common Development Tasks

### Adding a New Interaction
1. Find the target mesh name using Node.js scripts: `node search_meshes.js <pattern>`
2. Add mesh name pattern to the `raycastTargets` array in `index.html`
3. Add handler in the click event listener's if-else chain
4. Test with debug overlay (press `D`) to verify camera zoom targets

### Modifying Materials
1. All material assignments happen in `initMaterials()` - materials are matched by regex patterns on mesh names
2. Glass trays use `MeshPhysicalMaterial` with `transmission: 1.0`
3. To adjust colors/metalness/roughness: find the regex pattern and modify the material properties
4. Re-serve the app to see changes (hard refresh in browser)

### Adjusting Capacity Simulator Grid
- Interior cabinet dimensions are hardcoded: `{x: 0.8, y: 1.6, z: 0.5}` meters
- Grid-packing algorithm spawns blue boxes inside these bounds
- Modify `spawnProductBox()` function to change packing behavior or visual style

### Inspecting the 3D Model
Use Node.js utility scripts to understand the GLB structure without modifying code:
```bash
node list_names.js              # All mesh names
node print_hierarchy.js         # Full scene tree
node search_meshes.js "pattern" # Find meshes by regex
node inspect_solid1_61.js       # Details on GUI screen mesh
```

## Development

### Running Locally
Serve the directory with any static file server:
```bash
# Python
python -m http.server 8080

# Node.js (npx)
npx serve .

# VS Code Live Server extension also works
```
Then open `http://localhost:8080` in a browser.

**Note:** Must be served over HTTP (not opened as a `file://` URL) due to CORS restrictions on GLB loading.

## Utility Scripts
Node.js analysis scripts for inspecting the GLB model (not part of the runtime app):

| Script | Purpose |
|---|---|
| `list_names.js` | List all mesh/node names in the model |
| `print_hierarchy.js` | Print full scene hierarchy |
| `search_meshes.js` | Search for meshes by name pattern |
| `search_target_meshes.js` | Find specific interaction target meshes |
| `extract_hinges.js` | Extract BISAGRA (hinge) components |
| `find_led.js` | Locate LED bar mesh |
| `inspect_solid1_61.js` | Deep inspect GUI screen mesh |
| `check_indices.js` | Validate geometry index data |
| `compare_models.js` | Compare two model variants |

Run with: `node <script>.js`

## Git & Deployment

### Repository
- **Remote URL:** https://github.com/MartinSereyC/Bitua-3D-Explorer.git
- **Production branch:** `main` (stable, deployed code)
- **Development branch:** `dev3D` (active development)
- **Clone:** `git clone https://github.com/MartinSereyC/Bitua-3D-Explorer.git`

### Workflow
```bash
# Check out development branch
git checkout dev3D

# Make changes and commit
git add -A
git commit -m "Your message"

# Push to development
git push origin dev3D

# When ready for production, merge dev3D → main (via PR or direct merge)
```

**Branch strategy:**
- Work on `dev3D` for feature development and testing
- Push to `main` only when code is production-ready
- `main` is always stable and deployable

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bitua-io) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
