## wgsl-integration

> Mixing custom WGSL code into Three.js TSL shaders via wgslFn(). Use when integrating raw WGSL functions, porting existing WGSL, or optimizing hot paths that need hand-written shader code.


# TSL + WGSL Integration

`wgslFn()` lets you embed raw WGSL functions inside a TSL node graph. Inputs and outputs are declared in the WGSL signature and bound from TSL-side nodes at call time.

```javascript
import { wgslFn } from 'three/tsl';

const myNoise = wgslFn(`
  fn myNoise(p: vec3<f32>) -> f32 {
    return fract(sin(dot(p, vec3<f32>(12.9898, 78.233, 37.719))) * 43758.5453);
  }
`);

material.colorNode = myNoise(positionWorld);
```

@skills/webgpu-threejs-tsl/docs/wgsl-integration.md

---
> Source: [hexianWeb/lego-stylized-nature](https://github.com/hexianWeb/lego-stylized-nature) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
