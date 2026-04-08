## horse-position-2d-animation

> This project generates 2D animations of horse positions during a race using HTML Canvas. The animation can then be saved using the MediaRecorder API.

# Horse Position 2D Animation

This project generates 2D animations of horse positions during a race using HTML Canvas. The animation can then be saved using the MediaRecorder API.

## Project Structure

- `index.html`: The main HTML file that sets up the canvas and includes the stylesheet and JavaScript.
- `style.css`: Provides basic styling for the HTML elements.
- `script.js`: Contains the core logic for drawing the horse animation on the canvas and utilizing the MediaRecorder API to capture the animation.
- `horsePositions.js`: Defines the `horsePositions` array, which contains keyframe data for the race. Each element is an object with a `second` property (the time of the keyframe) and a `positions` array (the `x` and `y` coordinates for each horse).

## Technology Stack

This project exclusively uses **HTML, CSS, and vanilla JavaScript**. It avoids modern frameworks, TypeScript, or ES6+ features that require a build toolchain. This approach ensures the project can be run directly in a browser without a complex local environment setup, sidestepping issues like Cross-Origin Resource Sharing (CORS) that often arise when serving assets or testing with more advanced JavaScript features without a dedicated server.

## Animation Logic

The animation simulates a 1500-meter horse race. The `horsePositions.js` file stores keyframes in an array of objects. Each object contains the `second` of the keyframe and an array of `positions` for each horse.

-   **`x` position (horse lengths from lead):** Represents the horse's distance in horse lengths behind the current lead horse. A value of 0 indicates the horse is the lead horse. Negative values indicate the horse is ahead of the previous theoretical lead.
-   **`y` position (pixels):** Represents the horse's vertical position relative to the "Inner Rail". 
    *   **If direction is 'left'**: `y` is measured from the **top** edge of the canvas.
    *   **If direction is 'right'**: `y` is measured from the **bottom** edge of the canvas.
    This convention ensures that a constant `y` value represents the horse maintaining its lane relative to the track's center.

## Cornering Animation Logic (Proposed)

To simulate the horses running on a circular track (corners) and switching directions (e.g., from the backstretch to the homestretch), we will implement a field rotation mechanism.

1.  **Corner Detection:**
    -   A "corner" is identified when the `direction` property changes between the `fromKeyFrame` and `toKeyFrame` (e.g., from `'left'` to `'right'`).

2.  **Rotation Center:**
    -   The pivot point for the rotation is the center of the canvas:
        ```javascript
        const cx = canvas.width / 2;
        const cy = canvas.height / 2;
        ```

3.  **Interpolation & Rotation:**
    -   **Linear Phase:** Calculate the horses' positions as if they were on a horizontal straight line. 
    -   **Coordinate System**: We define the base "Inner Rail" at the top of the coordinate system. Thus, `baseY = y`.
    -   **Angle Calculation:** Interpolate a rotation angle (`theta`) based on the segment progress.
        -   'left' -> 'right': 0 to PI.
        -   'right' -> 'left': PI back to 0.
    -   **Coordinate Transformation:**
        -   Translate the horse's position so the canvas center is the origin `(0, 0)`.
        -   Apply 2D rotation and elliptical compression (Aspect Ratio).
        -   Translate back and render.

4.  **Rendering:**
    -   The horse's body (circle + pointer) rotates with the field, but the horse's number remains upright for readability.


This logic allows for a smooth visual transition between the two viewing directions without an abrupt cut.

## Keyframe Data (`horsePositions.js`)

The race data is manually set by analyzing the race movie. Each entry in the `horsePositions` array is a keyframe, representing the state of the race at a specific time.

**DATA STRUCTURE:**
The `horsePositions` array is an array of objects:
- `second`: The time in seconds from the start of the animation.
- `direction`: (Optional) 'left' or 'right'. If omitted, it defaults to the previous keyframe's direction.
- `positions`: An array of 8 objects, each representing a horse's position:
    - `x`: Horse lengths from the lead. A value of 0 indicates the lead horse. Negative values (e.g., -5) indicate the horse is trailing.
    - `y`: The vertical position in pixels on the canvas.

**HORSE ORDER:**
The horses are always in the same order in the `positions` array:
1. White, 2. Black, 3. Red, 4. Blue, 5. Yellow, 6. Green, 7. Orange, 8. Pink.

**RANKING COMMENTS:**
Comments above each `positions` array indicate the confirmed or estimated ranking of horses at that specific second. These should be respected when updating data.

Example keyframe:
```javascript
{
    second: 33,
    direction: 'left',
    rotation: 15,
    // 1: PINK, 2: WHITE, 3: BLUE, 4: RED, 5: BLACK, 6: ORANGE 7: YELLOW, 8: GREEN
    positions: [
        { x: -0.5, y: 25 }, { x: -5, y: 10 }, { x: -3.5, y: 15 }, { x: -1.5, y: 10 },
        { x: -7, y: 35 }, { x: -10, y: 50 }, { x: -6, y: 25 }, { x: 0, y: 10 }
    ]
},
```

## Gemini Agent Instructions

When interacting with this project, Gemini should strictly adhere to explicit instructions. Do not perform uninstructed actions, such as reverting commented-out code or making assumptions beyond the clear scope of the current request.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hayashi-ay)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hayashi-ay)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
