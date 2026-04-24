## magic-window

> Working with WebGL rendering in Electron applications, especially in renderer.js files that use PIXI.js for screen capture or video processing

{
  "description": "Correctly initialize and render with PIXI.js in Electron for screen recording",
  "when": "Working with WebGL rendering in Electron applications, especially in renderer.js files that use PIXI.js for screen capture or video processing",
  "rule": "1. Add 'unsafe-eval' to CSP in index.html for PIXI.js to work in Electron\n2. Set explicit canvas dimensions before initializing PIXI\n3. Implement a Canvas 2D API fallback when PIXI fails\n4. Use version-agnostic property access (app.view.width || app.renderer.width)\n5. Log PIXI version and capabilities on startup\n6. Validate media stream and texture creation separately",
  "examples": [
    {
      "before": "// PIXI setup with implicit dimensions\nconst app = new PIXI.Application({\n  view: canvasElement,\n  width: 3840,\n  height: 2160,\n  backgroundColor: 0x000000\n});\n\n// Direct access to specific properties\nvideoSprite.width = app.screen.width;\nvideoSprite.height = app.screen.height;",
      "after": "// Set explicit canvas dimensions first\ncanvasElement.width = 3840;\ncanvasElement.height = 2160;\n\n// Try PIXI initialization with error handling\ntry {\n  const app = new PIXI.Application({\n    view: canvasElement,\n    width: canvasElement.width,\n    height: canvasElement.height,\n    backgroundColor: 0x000000\n  });\n  usePixi = true;\n} catch (error) {\n  console.error('PIXI initialization failed:', error);\n  // Fall back to Canvas 2D API\n  usePixi = false;\n  canvasContext = canvasElement.getContext('2d');\n}\n\n// Use version-agnostic property access\nconst canvasWidth = app.view.width || app.renderer.width || 3840;\nconst canvasHeight = app.view.height || app.renderer.height || 2160;\nvideoSprite.width = canvasWidth;\nvideoSprite.height = canvasHeight;"
    },
    {
      "before": "<meta http-equiv=\"Content-Security-Policy\" content=\"default-src 'self'; media-src 'self'; style-src 'self' 'unsafe-inline';\">",
      "after": "<meta http-equiv=\"Content-Security-Policy\" content=\"default-src 'self'; media-src 'self' data: blob:; style-src 'self' 'unsafe-inline'; script-src 'self' 'unsafe-eval' https://cdn.jsdelivr.net;\">"
    }
  ],
  "tags": ["electron", "pixi.js", "webgl", "screen-recording", "csp", "fallback-rendering"],
  "validation": "Check console for successful PIXI initialization: 'PIXI.js version: X.X.X' and 'PIXI.js initialized successfully'. For the fallback, verify 'Canvas 2D initialized successfully' appears. Final validation is a completed recording process with log: 'Recording saved: /path/to/recording.mp4'",
  "ruleType": "Agent requested",
  "taskDescription": "This rule is helpful when implementing or debugging video rendering features in Electron applications that use PIXI.js for WebGL canvas rendering. It prevents common pitfalls with Content Security Policy restrictions, version-specific API differences, and ensures proper fallback rendering when WebGL is unavailable or fails."
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shreyas-makes) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
