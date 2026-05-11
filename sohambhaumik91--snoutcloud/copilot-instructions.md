## snoutcloud

> Handles saving frames and metadata after capture triggers.

# CLAUDE.md — SnoutCloud (Dog Nose Biometric App)

> Read this file at the start of every session.

---

## Project Overview

**SnoutCloud** is a React Native / Expo Go mobile app that serves as the UI layer for a dog nose biometric identification system. It was converted from a Next.js prototype and shares the same visual design.

**Companion ML repo:** `D:/biometrics_v2/` (NoseEncoder SupCon pipeline)

### Two screens:
1. **Home** — Landing page ("Know Your Dog, Always.") with CTA to scan
2. **Results** — Match found display with dog card, confidence bar, owner info

Camera integration (Phase 2) will add a scan flow between Home → Results.

---

## Tech Stack

| Layer | Choice |
|-------|--------|
| Framework | React Native via Expo SDK 54 |
| Navigation | React Navigation 7 (`@react-navigation/native-stack`) |
| Storage | `@react-native-async-storage/async-storage` |
| Icons | `@expo/vector-icons` (wraps react-native-vector-icons) |
| Fonts | `@expo-google-fonts/cormorant-garamond` |
| Gradients | `expo-linear-gradient` |
| Animations | React Native `Animated` API |
| TypeScript | Yes |

---

## File Map

```
App.tsx                          Root — font loading, splash, navigation mount
src/
  design/
    tokens.ts                    Design system: colors, typography, spacing, shadows, radii
  navigation/
    index.tsx                    Stack navigator + RootStackParamList type
  screens/
    HomeScreen.tsx               Landing page
    ResultsScreen.tsx            Match results page
  components/
    ProgressBar.tsx              Animated confidence progress bar
assets/
  dog.png                        Dog illustration (484×605 PNG, same as Next.js)
  icon.png                       App icon
  splash-icon.png                Splash screen icon
```

---

## Design System (`src/design/tokens.ts`)

All visual constants are centralized here. Never hardcode colors or sizes in components.

### Color Palette

```typescript
Colors.primaryBrown    = '#905d3c'   // Brand — CTA buttons, logo, headings
Colors.secondaryBrown  = '#c07a52'   // Progress bar gradient end
Colors.darkGreen       = '#2c3520'   // Body text, icons
Colors.mediumGreen     = '#44532e'   // Secondary text, badges
Colors.backgroundWarm  = '#f7f3f0'   // Main background
Colors.backgroundCard  = '#f0e8e9'   // Card / section background
Colors.accentPink      = '#e5d6d7'   // Borders, banner gradient
Colors.white           = '#ffffff'
```

Semi-transparent variants follow the pattern `Colors.primaryBrown70`, `Colors.mediumGreen50`, etc.

### Typography

Font family: **Cormorant Garamond** (serif — loaded via Expo Google Fonts)

```typescript
FontFamily.light    = 'CormorantGaramond_300Light'
FontFamily.regular  = 'CormorantGaramond_400Regular'
FontFamily.medium   = 'CormorantGaramond_500Medium'
FontFamily.semiBold = 'CormorantGaramond_600SemiBold'
FontFamily.bold     = 'CormorantGaramond_700Bold'
```

### Shadows

Pre-built shadow objects — apply with spread operator:
```typescript
style={[styles.myView, Shadows.card]}
```

Available: `logo`, `ctaResting`, `card`, `avatar`, `tabPill`, `actionBtn`

---

## Navigation

Stack with no header (custom headers in each screen):

```
Home → Results (navigate)
Results → Home (goBack)
```

Type-safe navigation with `RootStackParamList`:
```typescript
export type RootStackParamList = {
  Home: undefined;
  Results: undefined;
};
```

Use `useNavigation<HomeNavProp>()` in screens, where `HomeNavProp = NativeStackNavigationProp<RootStackParamList, 'Home'>`.

---

## Animations (React Native Animated API)

| Original CSS | RN Equivalent |
|---|---|
| `animate-scale-in` | `useScaleIn()` hook — opacity + scale + translateY, 550ms bezier |
| `animate-fade-in-up` | `useFadeInUp(delay)` hook — opacity + translateY |
| `animate-float` | `useFloat()` — looping translateY ±10px over 3s |
| `animate-pulse-glow` | `usePulseGlow()` — looping opacity + scale over 2.5s |
| Progress bar fill | `Animated.timing` on width interpolation, 1000ms |

All animations use `useNativeDriver: true` where possible (opacity/transform). The progress bar `width` interpolation requires `useNativeDriver: false`.

---

## AsyncStorage Usage

For future use — storing user preferences, scan history, auth tokens:

```typescript
import AsyncStorage from '@react-native-async-storage/async-storage';

// Write
await AsyncStorage.setItem('key', JSON.stringify(value));

// Read
const raw = await AsyncStorage.getItem('key');
const value = raw ? JSON.parse(raw) : null;

// Remove
await AsyncStorage.removeItem('key');
```

Currently no AsyncStorage calls exist — will be added in camera/auth phases.

---

## Run Commands

```bash
# Start for Expo Go (scan QR code on device)
npx expo start

# Platform-specific (still uses Expo Go)
npx expo start --ios
npx expo start --android

# Clear cache if needed
npx expo start --clear
```

**Testing on device:** Install Expo Go from the App Store / Google Play, then scan the QR code from `npx expo start`.

---

## Phase Roadmap

- [x] Phase 0 — Screen UI (Landing + Results, exact Next.js parity)
- [ ] Phase 1 — Camera layer (expo-camera, nose scan flow)
- [ ] Phase 2 — API integration (call NoseEncoder inference endpoint)
- [ ] Phase 3 — Auth (login/register, AsyncStorage tokens)
- [ ] Phase 4 — Dog profile management (add/edit registered dogs)

---

## ML Model Context (companion repo `D:/biometrics_v2/`)

The mobile app will eventually call an inference API backed by:

- **NoseEncoder:** ConvNeXt-Tiny backbone + AttentionHead (CLS token) + projection head
- **Training:** Supervised Contrastive Learning (SupConLoss), LLRD AdamW optimizer
- **Embedding:** `model.embed(x)` → 384-d normalized L2 vector
- **Retrieval:** cosine similarity against gallery embeddings
- **Dataset:** Pet Biometric Challenge 2022
- **Metric:** Rank-1 / Rank-5 retrieval accuracy

See `D:/biometrics_v2/CLAUDE.md` for full ML architecture details.

---

## Known Issues / Notes

- The decorative blur circles use `backgroundColor` with opacity — not a true CSS blur (would need `expo-blur` / `BlurView` for exact parity, but it's decorative)
- Figma API asset URLs from the Next.js version (logo, search, scan icons) are replaced with `@expo/vector-icons`: `paw`, `search-outline`, `camera-iris`, `arrow-back-outline`, `share-social-outline`, `refresh-outline`, `check-circle`
- `clamp()` CSS font sizes are replaced with fixed px values tuned for mobile (24px hero, 13px subtext)




# SnoutCloud — JSI Camera + YOLO Frame Processor

## Context
React Native Expo app (bare workflow via Expo Dev Client) for dog nose biometric scanning.
The goal is to auto-capture 8 high quality, diverse nose frames when user points phone at a dog.
These frames are sent to the server to generate biometric embeddings (ConvNeXt-Tiny + ArcFace).
Later, incoming nose images are matched against stored embeddings for dog identity verification.

## Stack
- `react-native-vision-camera` v4 — camera + frame processor runtime
- `react-native-fast-tflite` — synchronous JSI TFLite inference
- `vision-camera-resize-plugin` — native frame resize (stays in native memory)
- `react-native-worklets-core` — JSI worklet thread
- `expo-file-system` — local frame storage
- `expo-location` — capture location metadata
- New Architecture enabled (`newArchEnabled: true`)

## Model Spec
- File: `assets/models/best_float16.tflite`
- Task: single-class object detection (class 0 = `nose`)
- Input tensor: `[1, 320, 320, 3]` — NHWC, float32, pixel values in `[0, 1]`
- Output tensor: `[1, 5, 2100]` — layout is `[batch, attributes, anchors]`
  - attributes: `[x_center, y_center, width, height, confidence]`
  - coordinates are normalized `[0, 1]` relative to 320x320 input
- NMS: NOT baked in — must be done manually
- No end2end postprocessing

---

## What to Build

### 1. `hooks/useNoseDetector.ts`
A React hook that:
- Loads the TFLite model via `useTensorflowModel`
- Boxes the model for worklet access via `NitroModules.box()`
- Returns:
  ```ts
  {
    frameProcessor,
    detectionState,
    poolProgress,       // number 0-8, drives progress bar
    captureSession,     // CaptureSession | null — populated after capture completes
  }
  ```

---

### 2. Frame Processor Pipeline
A VisionCamera frame processor worklet. Ordered strictly cheap-to-expensive.
Every gate that fails returns early — never do more work than necessary.

**Gate 0 — Frame sampling:**
1. Increment `frameCount` shared value on every frame
2. Skip if `frameCount % FRAME_SAMPLE_INTERVAL !== 0` (~15fps effective at 30fps)
3. Skip if `isProcessing.value === true` — previous inference still running

**Gate 1 — Coarse quality check (cheap, full frame, before YOLO):**
4. Resize frame to 320x320 RGB using `vision-camera-resize-plugin`
   - Do this ONCE. Reuse this exact buffer for quality checks, YOLO input, and crop extraction
5. **Brightness check** — compute mean pixel value across full resized buffer
   - Reject if outside `[BRIGHTNESS_MIN, BRIGHTNESS_MAX]`
   - Set `detectionState = 'bad_lighting'`, reset pool, return early
6. **Coarse blur check** — compute Laplacian variance of full resized buffer
   - Reject if below `COARSE_SHARPNESS_THRESHOLD` — catches gross motion blur before YOLO
   - Set `detectionState = 'too_blurry'`, reset pool, return early

**Gate 2 — YOLO inference (expensive — only runs if Gate 1 passes):**
7. Normalize pixel values: divide each byte by 255.0 → `[0, 1]` float32
8. Run `tflite.runSync([inputBuffer])`
9. Parse output tensor `[1, 5, 2100]`:
   - Transpose to per-anchor layout: `[x_center, y_center, w, h, conf]`
   - Filter anchors where `conf > CONF_THRESHOLD`
   - Run NMS at `IOU_THRESHOLD` → single best box
   - If no box found → `detectionState = 'searching'`, return early

**Autofocus — fires every frame a nose is detected (after Gate 2):**
- Call `runOnJS(focusOnNose)(cx, cy)` — bbox center in normalized coords `[0,1]`
- `focusOnNose` maps to screen pixel coords, calls `cameraRef.current.focus({ x, y })`
- Debounce: only refocus if bbox center shifted > `FOCUS_DEBOUNCE_RATIO` of frame width since last call
- Fires before proximity/stability gates so focus locks on the nose immediately on detection

**Gate 3 — Box proximity check:**
10. Reject if normalized box width < `MIN_BOX_WIDTH_RATIO` (≥ 40% of frame width required)
    - `detectionState = 'too_far'`, return early — do NOT reset pool

**Gate 4 — Fine sharpness on bbox crop only:**
11. Extract nose bbox crop from 320x320 resized buffer:
    - Convert normalized `[x, y, w, h]` → pixel coords in 320x320 space
    - Slice crop as new ArrayBuffer (rows × cols × 3 bytes)
12. Compute Laplacian variance of crop only — background sharpness is irrelevant
    - Reject if below `FINE_SHARPNESS_THRESHOLD`
    - `detectionState = 'too_blurry'`, return early

**Gate 5 — Temporal stability:**
13. Compute IoU of current box vs `previousBox.value`
    - Reject if `iou < STABILITY_IOU` — nose moved
    - `detectionState = 'detected'`, return early without adding to pool
14. Update `previousBox.value`, set `detectionState = 'stable'`

**Diverse frame pool:**
15. Temporal spacing: skip if `frameCount - lastAcceptedFrame.value < MIN_FRAME_SPACING`
16. Score: `score = confidence * cropLaplacianVariance`
17. Extract bbox crop buffer — store ONLY the crop, not the full 320x320 frame
18. Add to pool: `{ cropBuffer, cropWidth, cropHeight, box, score, frameIndex }`
19. Update `lastAcceptedFrame.value`, `poolProgress.value = pool.length`
20. If `pool.length >= FRAMES_TO_CAPTURE`:
    - Sort pool by score descending
    - Call `runOnJS(triggerCapture)(pool)`
    - Reset pool, `previousBox.value`, set `detectionState = 'done'`

**Constants:**
```ts
const CONF_THRESHOLD = 0.5
const IOU_THRESHOLD = 0.45
const MIN_BOX_WIDTH_RATIO = 0.40
const STABILITY_IOU = 0.85
const COARSE_SHARPNESS_THRESHOLD = 60
const FINE_SHARPNESS_THRESHOLD = 100
const BRIGHTNESS_MIN = 50
const BRIGHTNESS_MAX = 220
const FRAMES_TO_CAPTURE = 8
const FRAME_SAMPLE_INTERVAL = 2
const MIN_FRAME_SPACING = 3
const FOCUS_DEBOUNCE_RATIO = 0.10
```

---

### 3. `utils/frameUtils.ts`
Pure arithmetic helpers — worklet-compatible, no imports, no async:

```ts
function sliceCropFromBuffer(
  buffer: ArrayBuffer,
  bufferWidth: number,  // 320
  bufferHeight: number, // 320
  x: number, y: number, cropWidth: number, cropHeight: number
): ArrayBuffer

function laplacianVariance(buffer: ArrayBuffer, width: number, height: number): number
function meanPixelValue(buffer: ArrayBuffer): number
function computeIoU(a: Box, b: Box): number
function bufferToBase64(buffer: ArrayBuffer): string  // for file system writes
```

---

### 4. `utils/nms.ts`
```ts
function nonMaxSuppression(boxes: Box[], iouThreshold: number): Box[]
type Box = { x: number, y: number, w: number, h: number, conf: number }
```

---

### 5. `types/capture.ts`
```ts
type DetectionState =
  | 'searching' | 'too_far' | 'bad_lighting'
  | 'too_blurry' | 'detected' | 'stable'
  | 'capturing' | 'done'

type Box = { x: number, y: number, w: number, h: number, conf: number }

type CapturedNoseFrame = {
  cropBuffer: ArrayBuffer  // raw RGB pixel data of bbox crop ONLY
  cropWidth: number
  cropHeight: number
  box: Box                 // normalized bbox in original frame
  score: number            // confidence * cropSharpness
  frameIndex: number
  jpegUri: string          // local file:// URI after save — used for tray preview
}

type CaptureSessionMetadata = {
  // Timing
  capturedAt: number           // Unix timestamp ms
  capturedAtISO: string        // ISO 8601 string

  // Location — request permission before scan starts
  location: {
    latitude: number | null
    longitude: number | null
    accuracy: number | null    // metres
    altitude: number | null
  } | null

  // Device
  device: {
    brand: string              // e.g. "samsung"
    model: string              // e.g. "SM-G991B"
    osVersion: string          // e.g. "14"
    appVersion: string         // from expo-constants
    deviceId: string           // anonymous device ID from expo-constants
  }

  // Camera
  camera: {
    facing: 'back'
    frameWidth: number         // native camera resolution width
    frameHeight: number        // native camera resolution height
    inferenceResolution: '320x320'
  }

  // Capture quality summary
  quality: {
    totalFramesProcessed: number
    totalFramesAccepted: number
    averageScore: number
    minScore: number
    maxScore: number
  }
}

type CaptureSession = {
  sessionId: string            // uuid
  sessionDir: string           // local file:// path to session folder
  frames: CapturedNoseFrame[]  // 8 frames sorted by score desc
  metadata: CaptureSessionMetadata
}
```

---

### 6. `hooks/useCaptureStorage.ts`
Handles saving frames and metadata after capture triggers.

```ts
const saveCaptureSession = async (
  frames: CapturedNoseFrame[],
  metadata: CaptureSessionMetadata
): Promise<CaptureSession>
```

Steps:
1. Generate `sessionId` (uuid)
2. Create session directory: `FileSystem.documentDirectory + 'captures/' + sessionId + '/'`
3. For each frame:
   - Convert `cropBuffer` (raw RGB ArrayBuffer) → base64
   - Write as `.bin` file: `frame_${frameIndex}_score_${score.toFixed(3)}.bin`
   - Also write as displayable file for tray preview: encode crop as JPEG using `expo-image-manipulator`
     - Write to `frame_${frameIndex}_preview.jpg`
     - Store `jpegUri = sessionDir + 'frame_' + frameIndex + '_preview.jpg'`
     - Set `frame.jpegUri` to this URI — used by the tray component
4. Write `meta.json` to session directory containing full `CaptureSessionMetadata` + per-frame summary
5. Return populated `CaptureSession`

**For JPEG encoding of raw RGB buffer:**
`expo-image-manipulator` does not accept raw ArrayBuffers.
Use this approach instead — write raw RGB as a data URI and let Image handle it:
```ts
// Write raw bin, then use expo-image-manipulator on a temp PNG via canvas
// OR: Use react-native-image to create a temporary image from buffer
// Simplest viable approach: write .bin files for server, write a separate
// full-frame JPEG via VisionCamera's snapshot() for the preview tray
```
See note below on preview strategy.

**Preview JPEG strategy:**
Instead of encoding raw RGB crops as JPEG (complex), take a VisionCamera **snapshot** when capture triggers:
- `cameraRef.current.takeSnapshot({ quality: 85 })` returns a full-frame JPEG file URI immediately
- Save snapshot URI alongside each session — use it as the tray preview background
- Overlay the bbox coordinates on the tray card to show which region was captured
- This avoids raw buffer → JPEG encoding entirely and gives a better quality preview

---

### 7. `components/NoseScannerCamera.tsx`
Full-screen camera view with overlay and capture tray.

**Camera + overlay:**
- `useCameraDevice('back')`
- Holds `cameraRef` for focus and snapshot
- `focusOnNose(normX, normY)` → `cameraRef.current.focus({ x: normX * W, y: normY * H })`
- Overlay states:
  - `searching`    → grey ring + "Point at dog's nose"
  - `too_far`      → orange ring + "Move closer"
  - `bad_lighting` → yellow ring + "Improve lighting"
  - `too_blurry`   → yellow ring + "Hold steady"
  - `detected`     → blue ring + "Keep steady..."
  - `stable`       → green ring + "Scanning..."
  - `capturing`    → green ring + progress bar "4 / 8 frames"
  - `done`         → flash animation → transition to tray

**Capture tray (shown after `detectionState === 'done'`):**
- Slides up from bottom of screen after capture completes
- Shows 8 captured frames as a horizontal scrollable tray of thumbnail cards
- Each card:
  - Displays snapshot JPEG with bbox rectangle overlaid (orange border showing nose crop region)
  - Shows frame score as a small badge (e.g. "0.87")
  - Tappable to expand full preview
- Below tray: two buttons — "Retake" (resets pipeline) and "Use These" (TODO: triggers upload)
- Tray is NOT dismissable until user explicitly chooses Retake or Use These

Accepts props:
```ts
{
  onCapture: (session: CaptureSession) => void  // called when tray is confirmed
  onRetake: () => void
}
```

---

### 8. Location + Device Metadata Collection
Collect metadata before scan starts, not during — avoids async calls in hot path.

In the parent screen that renders `NoseScannerCamera`:
```ts
import * as Location from 'expo-location'
import * as Device from 'expo-device'
import Constants from 'expo-constants'

// Request on screen mount, before scan begins
const [locationPermission] = await Location.requestForegroundPermissionsAsync()
const location = locationPermission.granted
  ? await Location.getCurrentPositionAsync({ accuracy: Location.Accuracy.Balanced })
  : null

const deviceMeta = {
  brand: Device.brand ?? 'unknown',
  model: Device.modelName ?? 'unknown',
  osVersion: Device.osVersion ?? 'unknown',
  appVersion: Constants.expoConfig?.version ?? 'unknown',
  deviceId: Constants.deviceId ?? 'unknown',
}
```

Pass `location` and `deviceMeta` as props into `NoseScannerCamera` — they get bundled into `CaptureSessionMetadata` when `saveCaptureSession` is called.

Required packages (install and add to app.json plugins if needed):
```bash
npx expo install expo-location expo-device expo-constants
```

---

### 9. Inspecting captures on desktop
Pull session from device:
```bash
adb pull /data/data/com.snoutcloud.app/files/captures ./captures
```

Decode `.bin` crop files:
```python
import numpy as np
from PIL import Image
import json

meta = json.load(open('meta.json'))
for f in meta['frames']:
    data = np.frombuffer(open(f'frame_{f["frameIndex"]}_score_{f["score"]:.3f}.bin', 'rb').read(), dtype=np.uint8)
    img = Image.fromarray(data.reshape((f['cropHeight'], f['cropWidth'], 3)), 'RGB')
    img.save(f'frame_{f["frameIndex"]}.jpg')
```

---

## Performance Notes
- GPU delegate: `enableAndroidGpuLibraries: true` in `app.json` — required for target latency
- Target inference latency: ~8-15ms per frame (YOLOv11n, 320x320, GPU delegate)
- Brightness + coarse blur: ~0.5ms each — always gate before YOLO
- `sliceCropFromBuffer`: tight loop, no allocations inside — keep it fast
- Never log inside the worklet — destroys frame rate
- Location fetch happens once on screen mount — never in the frame processor

---

## Important Implementation Notes
- Frame processor must be tagged `'worklet'`
- No React state inside worklet — shared values only
- `boxedModel` pattern for New Architecture + fast-tflite v2:
  ```ts
  const boxedModel = useMemo(
    () => model != null ? NitroModules.box(model) : undefined, [model]
  )
  // inside worklet:
  const tflite = boxedModel.unbox()
  const outputs = tflite.runSync([inputBuffer])
  ```
- Input buffer after resize:
  ```ts
  const inputBuffer = resized.buffer.slice(resized.byteOffset, resized.byteOffset + resized.byteLength)
  ```
- Normalize: divide raw uint8 values by 255.0 on a Float32Array view

---

## File Structure
```
hooks/
  useNoseDetector.ts
  useCaptureStorage.ts
components/
  NoseScannerCamera.tsx
utils/
  nms.ts
  frameUtils.ts
types/
  capture.ts
assets/
  models/
    best_float16.tflite
```

---

## Do Not
- Do NOT store full 320x320 buffer in pool — bbox crop only
- Do NOT use async/await inside worklet
- Do NOT access React state inside worklet
- Do NOT call takePhoto() for capture — use takeSnapshot() for preview only
- Do NOT compute fine sharpness on full frame — crop first
- Do NOT use consecutive frames — diverse pool with MIN_FRAME_SPACING
- Do NOT log inside worklet
- Do NOT refocus every frame — debounce with FOCUS_DEBOUNCE_RATIO
- Do NOT fetch location inside the frame processor or worklet
- Do NOT encode raw buffer to JPEG inside the worklet

---

## Phase 1 — Local Storage (Current)
Save frames locally. No network calls.

After capture triggers:
1. `saveCaptureSession(frames, metadata)` creates session dir, writes `.bin` files + `meta.json`
2. Take snapshot via `cameraRef.current.takeSnapshot()` for tray preview JPEGs
3. Populate `CaptureSession` with all frame URIs and metadata
4. Show capture tray — user reviews 8 thumbnails before confirming
5. "Use These" button is wired to `onCapture(session)` — parent receives session for later upload
6. Mark network call as TODO in `onCapture` handler

### Do Not (Phase 1)
- Do NOT implement network upload
- Do NOT implement Supabase storage
- Do NOT implement embedding generation
- Leave a clear `// TODO Phase 2: upload session to server` comment in onCapture handler

---

## Phase 2 — Network Layer (Next, not now)
TODO: Replace local save with upload:
- Convert crop buffers to multipart JPEG POST
- Send to embedding server endpoint
- Associate embeddings with dog profile in Supabase
- Remove local `.bin` files after successful upload

---
> Source: [sohambhaumik91/SnoutCloud](https://github.com/sohambhaumik91/SnoutCloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
