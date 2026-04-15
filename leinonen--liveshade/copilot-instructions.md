## liveshade

> - built with typescript

# LiveShade

- built with typescript
- clojure like DSL that generates GLSL fragment shaders.
- web based IDE to write the DSL code.
- it generates GLSL
- it runs the GSLS in the browser in a dedicated output canvas.
- it shows error messages in a console at the bottom
- it tracks line numbers in the DSL so that we can debug the programs easily

## DSL

(defshader plasma
  "Classic plasma effect"
  [uv time resolution]
  
  (let [x (* (.-x uv) (.-x resolution))
        y (* (.-y uv) (.-y resolution))
        
        v1 (sin (+ (/ x 16.0) time))
        v2 (sin (+ (/ y 8.0) (* time 1.3)))
        v3 (sin (/ (+ x y) 16.0))
        v4 (sin (/ (distance uv [0.5 0.5]) 8.0))
        
        color (/ (+ v1 v2 v3 v4) 4.0)
        color (+ (* color 0.5) 0.5)]  ; normalize to 0-1
    
    (palette-lookup plasma-pal color)))

(defpalette plasma-pal
  (gradient 0.0 1.0
    [0.0 [0 0 0]]
    [0.25 [0 0 255]]
    [0.5 [255 255 255]]
    [0.75 [255 255 0]]
    [1.0 [255 0 0]]))


## language features
- vector operations
- 1:1 mapping of functions (DSL/GLSL)
  - (sin x) -> sin(x)
  - (distance x) -> distance(x)

## higher order effect composition

(defshader tunnel [uv time resolution]
  (let [center [0.5 0.5]
        to-center (vec- uv center)
        dist (max 0.001 (length to-center))
        angle (atan (.-y to-center) (.-x to-center))
        
        u (/ 1.0 dist)
        v (+ angle time)]
    
    (checker-pattern u v 32.0)))

(defshader rotator [uv time resolution]
  (let [angle (* time 0.5)
        cos-a (cos angle)
        sin-a (sin angle)
        centered (vec- uv [0.5 0.5])
        rotated [(- (* (.-x centered) cos-a) 
                    (* (.-y centered) sin-a))
                 (+ (* (.-x centered) sin-a) 
                    (* (.-y centered) cos-a))]
        uv-rot (vec+ rotated [0.5 0.5])]
    (tunnel uv-rot time resolution)))

;; Compose shaders!
(defshader combo [uv time resolution]
  (blend 
    (plasma uv time resolution)
    (tunnel uv (* time 0.5) resolution)
    :mode :add))

## texture/buffer sampling

(deftexture prev-frame
  :size [320 200]
  :format :rgba)

(defshader feedback [uv time resolution]
  (let [offset [0.001 0.001]
        prev (sample prev-frame (vec+ uv offset))
        new-color (plasma uv time resolution)]
    (mix prev new-color 0.1)))  ; Decay

## for-loops -> unrolling
;; DSL
(defshader rays [uv time resolution]
  (reduce + 0.0
    (for [i (range 8)]
      (let [angle (* i (/ TAU 8.0))
            dir [(cos angle) (sin angle)]
            ray-color (march-ray uv dir time)]
        ray-color))))

;; Compiles to unrolled GLSL
float rays(vec2 uv, float time, vec2 resolution) {
  float acc = 0.0;
  {
    float angle = 0.0 * TAU / 8.0;
    vec2 dir = vec2(cos(angle), sin(angle));
    acc += march_ray(uv, dir, time);
  }
  {
    float angle = 1.0 * TAU / 8.0;
    vec2 dir = vec2(cos(angle), sin(angle));
    acc += march_ray(uv, dir, time);
  }
  // ... 6 more iterations
  return acc;
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/leinonen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/leinonen)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
