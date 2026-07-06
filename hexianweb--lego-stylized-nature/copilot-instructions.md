## post-processing

> Post-processing effects for Three.js WebGPU via the TSL node-based pipeline. Covers PostProcessing class, built-in passes (bloom, blur, FXAA, DOF), custom effects with Fn(), and effect chaining. Use when building render pipelines with post effects.


# Three.js WebGPU Post-Processing (TSL)

Post-processing uses the `PostProcessing` class from `three/webgpu` with node-based effect composition. Effects chain via TSL nodes rather than EffectComposer passes.

```javascript
import { PostProcessing } from 'three/webgpu';
import { pass, mrt, output, emissive } from 'three/tsl';
import { bloom } from 'three/addons/tsl/display/BloomNode.js';

const postProcessing = new PostProcessing(renderer);
const scenePass = pass(scene, camera);
postProcessing.outputNode = scenePass.add(bloom(scenePass.getTextureNode()));
```

Call `postProcessing.renderAsync()` instead of `renderer.renderAsync()`.

@skills/webgpu-threejs-tsl/docs/post-processing.md
@skills/webgpu-threejs-tsl/examples/post-processing.js

---
> Source: [hexianWeb/lego-stylized-nature](https://github.com/hexianWeb/lego-stylized-nature) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
