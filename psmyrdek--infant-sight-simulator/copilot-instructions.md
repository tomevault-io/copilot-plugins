## infant-sight-simulator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a scientific Baby Vision Simulator that demonstrates how infants see the world at different developmental stages (1-3 months old). The application uses advanced computer vision processing to simulate:

- Spatial frequency filtering based on Contrast Sensitivity Function (CSF)
- LMS color space processing with cone sensitivity modeling
- Optical effects (light scatter, chromatic aberration)
- Visual field limitations and peripheral vision effects
- Neural processing effects (photoreceptor noise, lateral inhibition)

## Architecture

### Core Files Structure
- `index.html` - Main HTML document with UI components and modal dialogs
- `app.js` - Main JavaScript application containing all vision simulation logic
- `styles.css` - Complete CSS styling with responsive design and animations

### Key Components

**Vision Processing Pipeline** (`app.js`):
1. Camera input capture with mirroring support
2. Spatial frequency filtering using separable convolution
3. LMS color space conversion with age-specific cone sensitivities
4. Optical effects simulation (scatter, aberrations)
5. Visual field processing with peripheral blur
6. Neural effects (noise, lateral inhibition)

**Scientific Constants** (`app.js:10-117`):
- Age presets with scientifically-based parameters for each month (1-3 months)
- LMS to RGB conversion matrices for color space processing
- Vision development constants (contrast sensitivity, spatial cutoffs, etc.)

**UI Components** (`index.html`):
- Real-time camera feed processing
- Age selector with visual acuity indicators (20/800, 20/150, 20/60)
- Settings panel with mirror and peripheral vision toggles
- Scientific information panel with technical parameters
- About modal with research references

## Development

### Commands

**Development Server** (with hot reload):
```bash
npm run dev          # Start Vite dev server at localhost:5173
```

**Production Build**:
```bash
npm run build        # Build to /docs directory
npm run build:pages  # Build for GitHub Pages with deployment message
npm run preview      # Preview production build locally
```

### Running the Application

1. **Development Mode**: `npm run dev`
   - Vite dev server with hot reload
   - Automatic browser opening
   - Camera access works on localhost

2. **Production Deployment**: 
   - Run `npm run build:pages` to build to `/docs` folder
   - Configure GitHub Pages to serve from `/docs` directory
   - Requires HTTPS for camera access in production

3. **Legacy Method**: Any static file server
   - `python -m http.server 8000`
   - `npx serve .`

### Core Processing Functions

**Spatial Frequency Processing** (`app.js:171-259`):
- `getContrastSensitivity()` - Models CSF development
- `applySpatialFrequencyFilter()` - Implements separable convolution
- `generateCSFKernel()` - Creates frequency-based filtering kernels

**Color Vision Processing** (`app.js:308-349`):
- RGB ↔ LMS color space conversion using Hunt-Pointer-Estevez matrices
- Age-specific cone sensitivity application
- Von Kries adaptation modeling

**Optical Effects** (`app.js:354-394`):
- Light scattering simulation using blur and screen blending
- Chromatic aberration via channel-specific spatial offsets

### Scientific Accuracy

The simulation is based on peer-reviewed vision development research:
- Contrast sensitivity function development curves
- Cone photoreceptor maturation timelines
- Optical properties of infant eyes (pupil size, scatter, accommodation)
- Visual field development and peripheral processing limitations

### Browser Compatibility

- Requires modern browser with camera access support
- Uses Canvas 2D API (no WebGL dependency)
- Implements progressive enhancement for mobile devices
- Responsive design with safe area support for mobile browsers

### Performance Considerations

- Optimized processing pipeline using separable convolution
- Multiple canvas buffers for efficient multi-step processing
- Real-time 30fps processing on modern devices
- Memory-efficient image data manipulation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/psmyrdek)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/psmyrdek)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
