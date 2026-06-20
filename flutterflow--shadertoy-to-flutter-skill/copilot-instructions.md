## shadertoy-to-flutter-skill

> Convert Shadertoy GLSL fragment shaders into Flutter-compatible GLSL fragment shaders (.frag files) for use with Flutter's FragmentProgram / FragmentShader API, Impeller, or the material_palette "fill shader" / "wrap shader" pattern. Use whenever the user pastes a Shadertoy shader (or points to one on shadertoy.com) and wants to run it in Flutter, asks to port or adapt a fragment shader to Flutter, mentions making Shadertoy-style uniforms (iResolution, iTime, iMouse, iChannel*, mainImage, fragCoord) work in Flutter, or has a .frag file that won't compile in Flutter because of Shadertoy-style constructs. Also use when the user wants to build a Flutter "fill shader" or "wrap shader" starting from a shadertoy source. This is a real code transformation — entry point, uniforms, sampler calls, unsupported-feature substitution — not a simple rename.


# Shadertoy → Flutter shader converter

Convert a Shadertoy fragment shader into a Flutter-compatible `.frag` file that will compile and run under Flutter's `FragmentProgram`/`FragmentShader` API.

## The guiding principle

**Change only what Flutter's runtime forces you to change. Preserve everything else.**

Shadertoy shaders vary wildly — one might be a 3000-line raymarcher using three noise textures and mouse input; another might be a twelve-line procedural pattern with `sin` and `cos`. A good port leaves each shader's shape intact and only rewrites what would fail to compile under Flutter's restrictions.

That means:

- If the source doesn't sample `iChannel*`, **don't add any noise/hash helper functions to the output**. The shader already has its own procedural content — leave it alone.
- Preserve comments, preprocessor blocks (`#if`/`#else`/`#endif`, `#define`), commented-out alternative implementations, author attribution headers, and the author's variable names.
- Don't eagerly resolve preprocessor toggles to a single active branch. If there's a `#define LOOK 1` with `#if LOOK==0` / `#else` branches, keep both branches intact; translate any Shadertoy-isms inside each branch, but leave the toggle structure. The only exception is when an inactive branch uses a feature that can't be ported at all — then call it out in chat but still don't silently delete it.
- Don't "clean up" formatting, renumber things, or reshape the code. The user picked this shader because they like how it looks on Shadertoy; the Flutter version should look the same.

Your job is translation, not refactoring.

## Fill shader or wrap shader?

This is the one structural decision you have to make. It follows from how the source uses its channels:

- **No iChannel sampling at all** → fill shader. Nothing to wire up.
- **iChannel sampled as noise/dither** (patterns in "iChannel handling" below) → fill shader with procedural substitution. The presence of a sampler in the Shadertoy source does not promote it to wrap.
- **iChannel sampled as a scene/image/asset** the user wants to keep → wrap shader. Route through `uTexture`.
- **Multiple image channels** → at most one can become `uTexture`; see "iChannel handling" for how to handle the others.

When the channel's role is genuinely ambiguous from the code alone, ask the user once: "Is this meant to be a standalone visual, or an effect layered over your app UI / an image you supply?"

## The output template

Every output file, fill or wrap, starts with:

```glsl
#include <flutter/runtime_effect.glsl>
precision highp float;
```

**Do not emit `#version` directives.** Flutter injects its own version header during the offline GLSL → SPIR-V compile; writing your own triggers a duplicate-version error or a version mismatch depending on the toolchain. If you see `#version 460 core` in an older example, treat it as stale.

Then uniforms, in this order:

1. `uniform vec2 uSize;`
2. `uniform float uTime;`
3. Any additional non-sampler uniforms the port genuinely needs (see "Additional uniforms" below — default to *not* adding any)
4. `uniform sampler2D uTexture;` — **wrap shaders only**, always the last uniform

Then the output declaration:

```glsl
out vec4 fragColor;
```

Then the shader's helper functions, preserved verbatim except for targeted substitutions (see "Targeted substitutions" below).

Then `void main() { ... }`.

## Entry point rewrite

Shadertoy:
```glsl
void mainImage(out vec4 <outName>, in vec2 <inName>) {
    // ... body uses <outName> and <inName> ...
}
```

The parameter names vary — most shaders use `fragColor`/`fragCoord`, but iq's tiny raymarcher uses `c`/`p`, others use `outColor`/`uv`, etc. Preserve whatever names the original used inside the body; rebind them at the top of `main()`:

```glsl
void main() {
    vec2 <inName> = FlutterFragCoord().xy;
    vec4 <outName>;
    // ... original body, verbatim except for Shadertoy uniform substitutions ...
    fragColor = <outName>;
}
```

If `<outName>` is literally `fragColor`, just use the file-level `fragColor` directly and skip the local declaration + final assignment — it's already in scope.

If `<inName>` is literally `fragCoord`, just write `vec2 fragCoord = FlutterFragCoord().xy;` at the top.

This pattern lets you leave compact/obfuscated shaders (like short-char-count golfed ones) exactly as-is in the body.

## Targeted substitutions inside the body

These are the substitutions you apply everywhere in the body, including inside preprocessor branches (even inactive ones, for consistency) but **not** inside comments.

| Shadertoy identifier | Flutter replacement |
|----------------------|---------------------|
| `iTime` | `uTime` |
| `iResolution.xy` | `uSize` |
| `iResolution.x`, `iResolution.y`, `iResolution.z` | `uSize.x`, `uSize.y`, `1.0` |
| `iResolution` (whole vec3) | `vec3(uSize, 1.0)` |
| `iMouse` | usually `vec4(0.0)` — see "iMouse handling" below |
| `iFrame` | `int(uTime * 60.0)` (if actually used) |
| `iTimeDelta` | `0.016` (if actually used) |
| `iFrameRate` | `60.0` (if actually used) |
| `iDate` | `vec4(2024.0, 1.0, 1.0, uTime)` (if actually used) |
| `iChannel0`–`iChannel3` | see "iChannel handling" below |
| `iChannelTime[*]`, `iChannelResolution[*]` | substitute reasonable constants if used |
| `iSampleRate` | `44100.0` (if used — but audio-reactive shaders can't fully port) |
| `gl_FragCoord` | `FlutterFragCoord()` (unlikely to appear in Shadertoy source, but handle it if so) |
| `texture(s, uv, bias)` | `texture(s, uv)` — drop the third arg |
| `textureLod(s, uv, lod)` | `texture(s, uv)` — drop the lod |
| `texelFetch(s, ivec_coord, lod)` | `texture(s, (vec2(ivec_coord) + 0.5) / TEXSIZE)` — or replace the whole call if the sampler is being substituted procedurally |
| `textureGrad`, `textureProj`, `texture2D`, etc. | `texture(s, uv)` — best effort |
| `textureCube(...)` | no clean port — substitute procedural or flag to user |

Full table with edge cases and ordering guidance: `references/uniform_mapping.md`.

## iMouse handling

Most Shadertoy shaders use `iMouse` to let viewers drag around for camera/parameter control. Flutter users generally don't want that as a uniform — they want a fixed visual they can drop into their UI. The default adaptation is to **hardcode `iMouse` to `vec4(0.0)`**, which is Shadertoy's unclicked state. Add a comment near the substitution explaining this and showing how to expose `uMouseX`/`uMouseY` uniforms if the user wants pointer interactivity.

If the shader uses `iMouse` in a way that produces a degenerate image at `vec4(0.0)` (e.g., the whole thing depends on a non-trivial mouse position), substitute a sensible default like `vec4(uSize * 0.5, 0.0, 0.0)` and say so in your summary.

## iChannel handling

If the source doesn't touch `iChannel*` at all, skip this section entirely — nothing to do.

If it does, the job is diagnostic first, then substitutive. Shadertoy channels are wildly flexible — users configure them as noise textures, baked images, cubemaps, the microphone, the webcam, keyboard state, another buffer's output, a video, or anything else the platform supports. The pasted source rarely says which; you have to infer from usage. **Your three tools are: procedural substitution (confident), `uTexture` routing (wrap shader), and honest partial conversion (acknowledge what's missing).** Pick based on how confidently you can classify the channel's content.

### Confident substitutions

Only reach for these when the shader's sampling pattern is a strong tell for what the channel contains.

- **Noise texture.** Symptoms: divisions by 256 or 1024, `.yx` swizzles, `* 2.0 - 1.0`, coordinates like `(uv + 0.5) / 256.0`, `texelFetch(..., ivec & 255, 0)`. Shadertoy's default RGBA/greyscale noise tiles. → Replace the whole noise-producing routine with a procedural equivalent from `references/noise_library.md`. Don't translate the texture call mechanically.
- **Blue-noise / dither tile.** Symptoms: single lookup producing a per-pixel random value used for jitter or dithering. → Replace the lookup with `hash11(fragCoord)` or `hash11(vec2(ivec_coord))`.
- **Scene / framebuffer / "Buffer X" in a simple single-pass overlay.** Symptoms: `texture(iChannelN, uv)` where `uv` is just the fragment UV and the result is composited with other colors. → Wrap shader: replace with `texture(uTexture, uv)` and use the wrap template.

### When you can't classify confidently

This is the common case. If the channel is sampled in ways that don't match the patterns above — a photograph, a pattern texture, a color ramp, a LUT, an asset the user baked into the Shadertoy page — don't guess at procedural substitutions. Instead, produce honest best-effort output:

1. **Default to offering the wrap path.** A single user-supplied image can be wired to `uTexture`. This is the most common realistic answer: the user has some image they care about, and they'll pass it into the Flutter shader the same way they passed it into Shadertoy. Structure the output as a wrap shader, sample `uTexture` where the original sampled the channel, and note in the summary that the user needs to supply the image via `FragmentShader.setImageSampler(0, image)` from Dart.
2. **If there are multiple image channels**, only one can become `uTexture` (Flutter's limit). For the others, substitute something visually neutral — `vec4(0.5)`, `vec4(0.0, 0.0, 0.0, 1.0)`, or a simple procedural pattern that at least won't crash — and leave a clear TODO comment at the substitution site saying what was lost. Surface this prominently in your summary.
3. **For channels that definitely can't port** (audio/FFT, microphone, keyboard, webcam, cubemap, multi-pass feedback buffers), substitute a safe placeholder (`0.0`, `vec4(0.0)`, a flat color — whatever keeps the math from NaN'ing), comment the substitution, and explain in the summary that this part of the shader will look wrong and what the user would need to do to reproduce the effect (pipe audio through a custom texture, render a skybox to `uTexture`, etc.).

### Confidence, not coverage

If you find yourself mentally stretching to call something a "noise channel" just to apply a procedural substitute, stop. A wrong confident substitution is worse than an honest `vec4(0.5)` placeholder with a comment. The user can read your summary, fix the gap, and move on. They cannot easily undo a port that silently replaced their photo with Perlin noise.

Only add `references/noise_library.md` content to the output file if some channel was a confident noise/dither substitution. Don't include it speculatively.

## Additional uniforms

Every extra uniform makes the shader harder for a Flutter user to wire up — they have to `setFloat(index, value)` for each one from Dart, in declaration order. **Default to hardcoded constants, not uniforms.** Only add a uniform when the value is something the Flutter user will plausibly want to animate, theme, or expose as a parameter (e.g., a color that should match the app's theme; a scale the user wants to drive from a slider). When in doubt, hardcode.

If you do add uniforms, place them between `uTime` and `uTexture` (or at the end of the uniform block if it's a fill shader), give them a `u`-prefixed camelCase name, and keep declaration order stable within a single port.

**Declaration order is load-bearing.** Flutter binds uniforms by index, not name. In the material_palette / `shader_definitions.dart` setup, the `UniformLayout([...])` list on the Dart side must match the `.frag`'s uniform order exactly — same count, same types, same sequence. Samplers are auto-bound at the end (index 0 of the sampler list, which is `uTexture` for a wrap shader); non-sampler uniforms flow into the float buffer in declaration order.

Two pairing rules when generating the Dart-side entries alongside a shader:

- `UniformField.color` is a `vec3`; `UniformField.colorRgba` is a `vec4`. Pick the one that matches the shader's declared type. A `uniform vec3 uTint;` paired with `UniformField.colorRgba` will silently misalign the float offsets of every uniform after it.
- Keep the header `uniform vec2 uSize; uniform float uTime;` first, then custom uniforms, then samplers last. The `UniformLayout` list mirrors that order.

## Remove or rewrite unsupported features

These will fail to compile under Flutter; rewrite them even if the original Shadertoy shader relied on them:

- **UBOs / SSBOs** (`uniform SomeBlock { ... }`, `buffer ...`). Unpack into individual top-level uniforms.
- **Uniform arrays indexed by runtime ints** (`uniform vec4 colors[N]; colors[idx]`). Unroll into individual uniforms (`uColor0`…`uColorN`) and use an `if`/`else if` chain to index. Local arrays inside functions are fine.
- **`uint` / `bool` uniforms or varyings.** Use `int` or `float`.
- **Non-`sampler2D` samplers** (`samplerCube`, `sampler3D`, `sampler2DArray`). No direct port; substitute procedural if feasible or flag to user.
- **Additional `in`/`varying` declarations** beyond what the runtime provides. Not supported.

Things that are *fine* to leave alone (don't "clean up" these just to be safe):

- Precision qualifiers (`highp`, `mediump`, `lowp`) — ignored but harmless.
- `dFdx`, `dFdy`, `fwidth` — standard GLSL ES derivatives; they work.
- Local arrays, structs, function `out` parameters, multi-return via structs.
- `const` globals, including `const mat2`, `const vec3`, etc.
- Preprocessor macros (`#define`), including multi-line and expression-level macros.
- Loops with integer bounds (`for (int i = 0; i < N; i++)`). Very large bounds (>several hundred) may perform poorly but will compile.

## SkSL runtime restrictions (the big one)

Flutter runs a **two-stage pipeline**: GLSL → SPIR-V (offline, via Impeller's compiler) → SkSL (runtime, via spirv-cross, consumed by Skia). Stage 1 is permissive; stage 2 is strict. `flutter build` passing does **not** mean the shader will load — invalid-SkSL errors only surface when the app actually instantiates `FragmentProgram`. Symptoms look like:

```
Exception: Invalid SkSL: ...
error: NNN: no match for min(int, int)
error: NNN: initializers are not permitted on arrays
```

The restrictions that matter in practice, and the rewrites to apply proactively when porting:

- **No array initializers.** `vec4 palette[10] = vec4[](a, b, …);` is rejected at the SkSL stage. This overlaps with the uniform-array unrolling rule above, but also applies to `const` and local arrays built from a list literal. Replace with a lookup function that returns the uniform (or a hardcoded branch) directly:
  ```glsl
  vec4 paletteAt(int i) {
      if (i <= 0) return uColor0;
      if (i == 1) return uColor1;
      // …
      return uColor9;
  }
  ```
- **No integer overloads of `min`, `max`, `clamp`, `abs`, `mod`.** SkSL only has the float forms. Do the math in floats and `int()`-cast only at the final use site (array index, loop bound). `min(iCount, 8)` → `int(min(float(iCount), 8.0))`.
- **No non-`const` global arrays.** Promote to `const` if the values are known at compile time; otherwise move the array inside the function that uses it.
- **No reserved GLSL keywords as identifiers.** `sample` is an interpolation qualifier and will break even though Shadertoy accepts it. Also avoid `input`, `output`, `mediump`, `highp`, `lowp`, `buffer`, `shared`, `coherent`, `readonly`, `writeonly` as variable or function names. When you see such a name in the source, rename it (e.g., `sample` → `s`, `samp`, or `texSample`) everywhere in the body.

## Alpha output — Flutter expects premultiplied

The Flutter compositor expects premultiplied alpha. Write the final assignment as:

```glsl
fragColor = vec4(rgb * alpha, alpha);
```

Straight-alpha `vec4(rgb, alpha)` composites incorrectly: transparent regions leak opaque RGB onto the canvas, so you won't see the background through `alpha = 0`. For fully opaque output (`alpha == 1.0`) the two forms are identical and no rewrite is needed; the distinction matters whenever the shader produces non-opaque pixels.

When the Shadertoy source ends with `fragColor = vec4(col, 1.0);` and the shader is opaque, leave it as-is. When it ends with something like `fragColor = vec4(col, a);` with a variable `a`, rewrite to `fragColor = vec4(col * a, a);`.

## Texture sampling at boundaries (wrap shaders)

With Flutter's default sampler, taps outside `[0, uSize]` return `vec4(0.0)` — a transparent black pixel. For a single-tap wrap shader this is usually fine. For multi-tap filters (large-kernel blurs, sector averaging, anything that reads neighbors), those zero taps drag the result toward black, producing visible smudges at canvas corners.

When a wrap shader takes multiple taps and the kernel can reach past the child bounds:

- Alpha-weight each tap: accumulate `sum += tap * tap.a; weight += tap.a;` and divide by `weight` at the end.
- Handle the `weight ≈ 0` case explicitly (return a sentinel high variance, the original pixel, or a neutral color) so the math doesn't NaN when every tap landed out of bounds.

A single-tap sample-at-this-UV wrap shader doesn't need any of this.

## Numerical safeguards

Uniforms that land in a divisor, a `normalize()`, or similar are hazards if the Flutter user can drive them from a slider that reaches 0. Clamp at the use site:

```glsl
float eps = max(uSampleEps, 1e-4);
```

The canonical failure is a finite-difference normal: at `uSampleEps = 0`, three taps collapse onto the same point and `normalize(vec3(0.0))` returns NaN — which shows up as black sparkles or full-frame aliasing. `1e-4` is a safe default for most screen-space work; pick a bigger floor when the uniform is in world units.

Apply this pattern whenever you expose a uniform whose Dart-side range could plausibly include 0 and whose math would blow up there. Don't add the clamp speculatively on uniforms where 0 is a legitimate value.

## Coordinate quirks (Y-axis)

Shadertoy's `fragCoord` has origin at the **bottom-left** (GL convention). Flutter's `FlutterFragCoord()` has origin at the **top-left**. For symmetric effects this is invisible; for anything with a clear vertical orientation (a sunset gradient with the sun up, a scene with a ground plane and sky, text/glyphs) the shader will render upside-down.

When producing the output, add a commented-out Y-flip line right after the `FlutterFragCoord()` call:

```glsl
vec2 fragCoord = FlutterFragCoord().xy;
// If the image appears upside-down, uncomment:
// fragCoord.y = uSize.y - fragCoord.y;
```

This is cheaper than guessing. The user can toggle it in seconds.

## Workflow

1. **Read the source end-to-end.** Note: which `iChannel*` slots are sampled and what for; whether `iMouse` is used; whether the shader has preprocessor toggles, multi-pass dependencies (`iChannelN` = "Buffer A"), or features that simply can't port (cubemaps, audio, keyboard).
2. **Decide fill vs wrap** (see above).
3. **Build the output file**, in this order:
   - The fixed header (`#include <flutter/runtime_effect.glsl>` + `precision highp float;`, **no `#version` directive**).
   - Uniforms in the fixed order: `uSize`, `uTime`, any additional uniforms you're adding, and `uTexture` (wrap only) last.
   - `out vec4 fragColor;`.
   - Any helper functions the substitutions require (noise hashes from `references/noise_library.md`, **only if the original sampled noise from `iChannel*`**). Don't add them otherwise.
   - The original's helper functions and globals, preserved verbatim except for targeted substitutions in their bodies — with the SkSL-level rewrites applied (no array initializers, no integer `min`/`max`/`clamp`/`abs`/`mod`, no reserved-keyword identifiers like `sample`).
   - `void main()` with the mainImage body, adapted per the entry-point rewrite rule, ending in a premultiplied `fragColor = vec4(rgb * alpha, alpha);` when the shader emits non-opaque pixels.
4. **Save** the file to `/mnt/user-data/outputs/` with a descriptive name (`<shader-name>_fill.frag` or `<shader-name>_wrap.frag`) and surface it with the `present_files` tool.
5. **Summarize in chat** (not in the .frag): fill-vs-wrap decision, what each `iChannel*` became, how `iMouse` was handled, any features that couldn't be ported and what was done instead, and any Y-flip the user may want to toggle.

If the shader fundamentally can't be ported (multi-pass feedback, audio-reactive, cubemap-dependent), produce the best partial conversion and list what the user would need to do manually to finish — don't refuse to produce output.

## Verification gate

No command-line step catches every shader error, and it's worth being explicit about what each one does (and doesn't) cover before telling a user "this compiles":

- `flutter analyze` — Dart static analysis only. Never looks at `.frag` files. Useless as a shader check.
- `flutter build <platform> --debug` (e.g. `flutter build macos --debug`) — runs Impeller's offline GLSL → SPIR-V compile. Catches stage-1 errors: syntax, undeclared identifiers, `#version` directives you forgot to remove, unsupported uniform types.
- Actually launching the app and instantiating the shader — the only thing that surfaces **stage-2** (SkSL) errors like `no match for min(int, int)` or `initializers are not permitted on arrays`. These shaders build fine; they fail when Skia tries to link the SkSL at runtime.

When you hand off a ported shader, say which gate you ran (usually none, if you're just writing the file) and remind the user that a green `flutter build` is not the same as a green shader load. Proactively applying the SkSL restrictions above is what keeps stage-2 errors from biting on first run.

## References

- `references/flutter_glsl_constraints.md` — the complete list of what Flutter's GLSL runtime does and doesn't support, and why.
- `references/uniform_mapping.md` — exhaustive Shadertoy-uniform → Flutter-uniform mapping, including the heuristics for classifying `iChannel*` usage.
- `references/noise_library.md` — procedural hash / value noise / Perlin functions to drop into the output **only when** replacing an `iChannel*` noise texture. Not for speculative inclusion.
- `references/templates.md` — fill and wrap shader skeletons, plus worked before/after examples (including cases with unusual mainImage parameter names).

---
> Source: [FlutterFlow/shadertoy_to_flutter_skill](https://github.com/FlutterFlow/shadertoy_to_flutter_skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
