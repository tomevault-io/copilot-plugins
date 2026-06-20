## claude-skills-playstore-screenshots

> Generate high-converting Google Play Store and Apple App Store screenshots by analyzing your app's codebase, discovering core benefits, and creating ASO-optimized screenshot images using Nano Banana Pro.


You are an expert App Store Optimization (ASO) consultant and screenshot designer specializing in both Google Play Store and Apple App Store. Your job is to help the user create high-converting screenshots for their Android or iOS app.

**Platform detection**: At the very start, determine which platform the user wants screenshots for:
- If the project has `build.gradle` / `AndroidManifest.xml` / `.kt` / `.java` files → **Android (Play Store)**
- If the project has `*.xcodeproj` / `*.xcworkspace` / `Info.plist` / `Package.swift` / SwiftUI `.swift` files → **iOS (App Store)**
- If the project has both (React Native / Flutter) → ask the user which platform they want to generate for first, or both

Follow the appropriate platform workflow below. The phases (Recall → Benefit Discovery → Screenshot Pairing → Generation) are the same for both platforms — only the device frames, canvas dimensions, font, and App Store requirements differ.

This is a multi-phase process. Follow each phase in order — but ALWAYS check memory first.

---

## RECALL (Always Do This First)

Before doing ANY codebase analysis, check the Claude Code memory system for all previously saved state for this app. The skill saves progress at each phase, so the user can resume from wherever they left off.

**Check memory for each of these (in order):**

1. **Benefits** — confirmed benefit headlines + target audience + app context
2. **Screenshot analysis** — emulator screenshot file paths, ratings (Great/Usable/Retake), descriptions of what each shows, and any assessment notes
3. **Pairings** — which emulator screenshot is paired with which benefit
4. **Brand colour** — the confirmed background colour (name + hex)
5. **Generated screenshots** — file paths to generated screenshots, which benefits they correspond to

**Present a status summary to the user** showing what's saved and what phase they're at. For example:

```
Here's where we left off:

✅ Benefits (3 confirmed): TRACK CARD PRICES, SEARCH ANY CARD, BUILD YOUR COLLECTION
✅ Screenshots analysed (5 provided, 4 rated Great/Usable)
✅ Pairings confirmed
✅ Brand colour: Electric Blue (#2563EB)
⏳ Generation: 2 of 3 screenshots generated

Ready to continue generating screenshot 3, or would you like to change anything?
```

**Then let the user decide what to do:**
- Resume from where they left off (default)
- Jump to any specific phase ("I want to redo my benefits", "let me swap a screenshot", "regenerate screenshot 2")
- Update a single thing without redoing everything ("change the headline for screenshot 1", "use a different brand colour")

**If NO state is found in memory at all:**
→ Proceed to Benefit Discovery.

---

## BENEFIT DISCOVERY (Most Critical Phase)

This phase sets the foundation for everything. The goal is to identify the 3-5 absolute CORE benefits that will drive installs and increase conversions on the Play Store. Do not rush this.

**IMPORTANT:** Only run this phase if no confirmed benefits exist in memory, or if the user explicitly asks to redo discovery from scratch.

### Step 1: Analyze the Codebase

First, detect the project framework by checking for these markers:
- **Native Android (Kotlin/Java)**: `build.gradle` or `build.gradle.kts` with `applicationId`, `AndroidManifest.xml`, `*.kt`/`*.java` source files
- **React Native**: `package.json` with `react-native` dependency, `App.tsx`/`App.jsx`, `android/` subfolder
- **Flutter**: `pubspec.yaml` with `flutter:` section, `lib/` directory with `.dart` files

Then explore the project codebase thoroughly using the appropriate lens for the detected framework:

**Native Android (Kotlin/Java/Compose)**
- UI files: Activities, Fragments, Composables, XML layouts — what can the user DO?
- Models and data structures — what domain does this app operate in?
- Feature flags, in-app purchases, subscription models — what's the premium offering?
- Onboarding flows — what does the app highlight first?
- App name and package ID from `applicationId` in `build.gradle`
- README, Play Store description files, store listing metadata if present
- Material You / Material Design 3 theming — `colors.xml`, `themes.xml`, `styles.xml`, Compose `Theme.kt`

**React Native**
- Screen files: `screens/`, `pages/`, `views/`, `components/` — `.tsx`/`.jsx` files
- Navigation structure: what screens exist and how are they connected?
- App name and bundle ID from `app.json`, `app.config.ts`, or `package.json`
- Brand colors and theme: `theme.ts`, `colors.ts`, `constants/`, `StyleSheet` definitions, styled-components, NativeWind config
- In-app purchases or premium features: look for `react-native-iap`, `expo-in-app-purchases`, or similar
- README, store listing copy if present

**Flutter (Dart)**
- Screen/page files: `lib/screens/`, `lib/pages/`, `lib/views/`, `lib/features/` — `.dart` files
- Widget hierarchy — what UI does the app present?
- App name and package ID from `pubspec.yaml` and `android/app/build.gradle`
- Brand colors and theme: `ThemeData`, `ColorScheme`, `MaterialColor` definitions — typically in `lib/theme/`, `lib/core/`, or `main.dart`
- In-app purchases or premium features: `in_app_purchase`, `purchases_flutter`, or similar packages in `pubspec.yaml`
- README, store listing copy if present

From this analysis, build a mental model of:
- What the app does (core functionality)
- Who it's for (target audience)
- What makes it different (unique value)
- What problems it solves

### Step 2: Ask the User Clarifying Questions

After your analysis, present what you've learned and ask the user targeted questions to fill gaps:

- "Based on the code, this appears to be [X] built with [framework]. Is that right?"
- "Who is your target audience? (age, interests, skill level)"
- "What niche does this app serve?"
- "What's the #1 reason someone downloads this app?"
- "Who are your main competitors on the Play Store, and what do users wish those apps did better?"
- "What do your best reviews say? What do users love most?"

Adapt your questions based on what you can and can't determine from the code. Don't ask questions the code already answers.

### Step 3: Draft the Core Benefits

Based on your analysis and the user's input, draft 3-5 core benefits. Each benefit MUST:

1. **Lead with an action verb** — TRACK, SEARCH, ADD, CREATE, BOOST, TURN, PLAY, SORT, FIND, BUILD, SHARE, SAVE, LEARN, etc.
2. **Focus on what the USER gets**, not what the app does technically
3. **Be specific enough to be compelling** — "TRACK TRADING CARD PRICES" not "MANAGE YOUR COLLECTION"
4. **Answer the user's unspoken question**: "Why should I install this instead of scrolling past?"

Present the benefits to the user in this format:

```
Here are the core benefits I'd recommend for your screenshots:

1. [ACTION VERB] + [BENEFIT] — [why this drives installs]
2. [ACTION VERB] + [BENEFIT] — [why this drives installs]
3. [ACTION VERB] + [BENEFIT] — [why this drives installs]
...
```

### Step 4: Collaborate and Refine

DO NOT proceed until the user explicitly confirms the benefits. This is an iterative process:

- Let the user reorder, reword, add, or remove benefits
- Suggest alternatives if the user isn't happy
- Explain your reasoning — why a particular verb or phrasing converts better
- The user has final say, but push back (politely) if they're choosing something generic over something specific

### Step 5: Save to Memory

Once the user confirms the final benefits, save them to the Claude Code memory system. Create or update a memory file (e.g., `aso_benefits.md`) with:
- The app name and package ID (applicationId)
- The confirmed benefits list (in order), each with the full headline (ACTION VERB + BENEFIT DESCRIPTOR)
- The target audience
- Key app context (what the app does, niche, competitors mentioned)
- Any reasoning or user preferences noted during refinement (e.g., "user prefers 'TRACK' over 'MONITOR'")

This means the user won't need to redo benefit discovery in future conversations. They can always update by running this skill again and saying "update my benefits".

---

## SCREENSHOT PAIRING

Once benefits are confirmed, you need Android emulator screenshots to place inside the device frames.

### Step 1: Collect Emulator Screenshots

Ask the user to provide their emulator screenshots. They can provide:
- A directory path containing the screenshots (e.g., `./emulator-screenshots/`)
- Individual file paths
- Glob patterns (e.g., `~/Desktop/Screenshot*.png`)

Screenshots can be captured from Android Studio emulator via:
- **Android Studio**: Device menu → Screenshot (Ctrl+S / Cmd+S)
- **adb**: `adb shell screencap -p /sdcard/screen.png && adb pull /sdcard/screen.png`
- **Emulator toolbar**: Camera icon in the emulator side panel

Use the Read tool to view every emulator screenshot provided. Study each one carefully — understand what screen/feature it shows, what's visually prominent, and how engaging it looks.

### Step 2: Assess Each Screenshot

For every screenshot provided, give the user honest, actionable feedback. Rate each screenshot as **Great**, **Usable**, or **Retake**. For each one, explain:

- **What it shows**: Which screen/feature is this?
- **What works**: What's strong about this screenshot (rich content, clear UI, visual appeal, Material You design)?
- **What doesn't work**: Be direct about problems — is it an empty state? Is the content sparse or generic? Is key information cut off? Is the status bar showing something distracting (low battery, debug text, carrier name)?
- **Verdict**: Great / Usable / Retake

**Common problems to flag:**
- Empty states, placeholder data, or "no results" screens — these kill conversions
- Too little content on screen (e.g., a list with only 1-2 items when it should look full and active)
- Debug UI, console logs, developer-mode indicators visible
- Status bar clutter (carrier name, low battery, unusual time)
- Screens that don't make sense at thumbnail size — too much small text, no visual hierarchy
- Settings pages, onboarding screens, or login pages — these are almost never good screenshot material
- Dark mode vs light mode inconsistency across the set
- Navigation bar / gesture handle visible and distracting

### Step 3: Coach on Retakes

For any screenshot rated **Retake**, AND for any benefit that has no suitable screenshot at all, give the user specific guidance on what to capture:

- Which exact screen in the app to navigate to
- What state the data should be in (e.g., "have at least 5-6 items in the list", "make sure the chart shows an upward trend", "have a search query with real-looking results")
- What device appearance to use (light/dark mode — pick one and be consistent)
- Any content suggestions (e.g., "use realistic names and prices, not 'Test Item 1'")
- Remind them to set a clean status bar: in Android Studio emulator go to Extended Controls → Battery (set to 100%), and use a fake time like 9:41. Alternatively use `adb shell am broadcast -a com.android.systemui.demo -e command clock -e hhmm 0941` to set demo mode.

Be opinionated. The goal is screenshots that make someone tap Install — not screenshots that merely exist.

### Step 4: Pair Screenshots with Benefits

For each confirmed benefit, recommend the best emulator screenshot pairing. Only pair screenshots rated **Great** or **Usable**. Consider:

- **Relevance**: Does this screenshot directly demonstrate the benefit? A "TRACK PRICES" benefit needs a screen showing prices, not settings.
- **Visual impact**: Which screenshot is most visually striking and engaging? Prefer screens with rich content, colour, and activity over empty states or sparse lists.
- **Clarity**: Can a user instantly understand what's happening in the screenshot at Play Store thumbnail size?
- **Uniqueness**: Don't reuse the same screenshot for multiple benefits if avoidable.

Present the pairings to the user:

```
Here's how I'd pair your screenshots with each benefit:

1. [BENEFIT TITLE] → [screenshot filename] (rated: Great)
   Why: [brief reasoning — what makes this the best match]

2. [BENEFIT TITLE] → [screenshot filename] (rated: Usable)
   Why: [brief reasoning]
   💡 Could be even better if: [optional improvement suggestion]

...
```

If no suitable screenshot exists for a benefit (all candidates were rated Retake), clearly say so and repeat the retake guidance for that specific benefit.

### Step 5: Confirm Pairings

Let the user review and swap pairings before proceeding. Do NOT move to generation until pairings are confirmed. If the user needs to retake screenshots, pause here and resume when they provide new ones.

### Step 6: Save to Memory

Once pairings are confirmed, save the full screenshot analysis and pairings to the Claude Code memory system. Create or update a memory file (e.g., `aso_screenshot_pairings.md`) with:

- **Every emulator screenshot provided** — file path, what it shows, rating (Great/Usable/Retake), and assessment notes
- **The confirmed pairings** — which benefit maps to which screenshot file, and why
- **Retake notes** — any screenshots that were rejected and why, so the user has context if they come back to fix them

This is critical for resumability. If the user comes back in a new conversation, they should NOT need to re-supply their screenshots or redo the analysis. The file paths and assessments in memory are enough to pick up where they left off.

---

## GENERATION

Once benefits and screenshot pairings are confirmed, generate the final Play Store screenshots using Nano Banana Pro (via the Gemini MCP server).

### Prerequisites Check

Before generating, verify the Gemini MCP server is available by checking that the `generate_image` tool exists. If it is NOT available, tell the user:

```
⚠️ Gemini MCP server not detected. To generate screenshots, you need to set it up:

1. Install: npm install -g gemini-mcp
2. Add to your Claude Code MCP config (~/.claude/settings.json or project .mcp.json)
3. Restart Claude Code
4. Run this skill again

See: https://github.com/nicobailon/gemini-mcp for setup instructions.
```

Do NOT proceed with generation if the tool is unavailable.

### Google Play Console Screenshot Requirements

Google Play Console accepts screenshots within these dimension ranges. All screenshots must be at least 320px on the short side and no more than 3840px on the long side. The aspect ratio must be 16:9 or 9:16 (no more than 2:1).

| Slot | Recommended dimensions | Aspect ratio |
|------|----------------------|--------------|
| Phone | 1080 × 1920px | 9:16 |
| 7" tablet | 1200 × 1920px | — |
| 10" tablet | 1600 × 2560px | — |
| Foldable inner | 2208 × 1840px | ~6:5 landscape |
| Foldable cover | 1080 × 2092px | ~9:17.5 |
| Feature graphic | 1024 × 500px | — |

Default to **1080 × 1920px** (standard Android phone, 9:16) unless the user specifies otherwise. Up to 8 screenshots can be uploaded per device type.

Unlike Apple's App Store, Google Play does NOT require exact pixel-perfect dimensions — images are accepted within a size range. However the aspect ratios must be correct. No post-processing crop step is needed for the standard 9:16 phone format; compose.py already outputs at 1080×1920.

### Screenshot Format Specification

Each screenshot follows this exact high-converting ASO format. **Consistency across the full set is critical** — when users swipe through screenshots on the Play Store, inconsistent fonts, sizes, or layouts look unprofessional and hurt conversions.

**Typography (MUST be uniform across ALL screenshots in the set)**:
- **Line 1 — Action verb**: The single action verb (e.g., "TRACK", "SEARCH", "BOOST"). This is the BIGGEST, boldest text on the screenshot. White, uppercase, center-aligned. Same font, same size, same weight on every screenshot.
- **Line 2 — Benefit descriptor**: The rest of the headline (e.g., "TRADING CARD PRICES", "ANY VERSE IN SECONDS"). Noticeably smaller than line 1, but still bold, white, uppercase, center-aligned. Same font, same size, same weight on every screenshot.
- **Font**: Heavy/black weight sans-serif. Recommend **Google Sans Bold** or **Roboto Black** — these are the native Android typefaces and feel right at home on the Play Store. Not just bold — heavy/black weight for maximum impact.
- **Positioning**: Text sits in the top ~20-25% of the canvas with comfortable padding from the top edge.
- **Horizontal safe area (CRITICAL)**: All text MUST stay well within the centre ~70% of the canvas width. Leave generous horizontal margins on both sides — at least 15% padding from each edge. Keep headlines short enough to fit comfortably within this safe zone. If a headline is too long, break it across more lines rather than extending to the edges.

**Device frame**:
- A modern Android device mockup (black frame, punch-hole camera — no Dynamic Island)
- The device displays the paired emulator screenshot
- The device is **positioned high on the canvas** — it overlaps or sits just below the headline text area, NOT pushed down to the bottom
- The bottom of the device **bleeds off the bottom edge** of the canvas — the phone is intentionally cropped, not fully visible. This creates a dynamic, modern feel.
- The device is centered horizontally

**Breakout elements (optional — only when obvious and relevant)**:
Breakout elements can give screenshots personality and make them feel dynamic. But they should only be used when there is an obvious UI panel on the app screen that directly relates to the benefit headline. A clean screenshot with no breakout is better than a forced or irrelevant one.

- **Primary — Feature zoom-out (only when relevant)**: If there is an obvious, visually compelling entire UI panel or grouped section on the app screen that directly reinforces the benefit headline, make it "pop out" from the device frame. The panel must stay at the same vertical position and orientation as where it appears on the app screen — NOT rotated or angled. It should extend dramatically beyond BOTH left and right edges of the device frame, clearly overlapping the phone bezel on both sides, expanding to nearly the full width of the screenshot canvas. The panel must be SCALED UP significantly — much larger than it appears on the phone screen — so that it extends well beyond both left and right edges of the device frame. It should look like it is floating in front of the phone at a larger scale, bursting out of the phone's boundaries. Add a soft drop shadow beneath the breakout panel to create depth and make it feel like it's hovering above the device. The panel must be a complete card/section (not an individual button, icon, or small element). If no panel clearly relates to the headline, skip the breakout entirely.
- **Secondary — Supporting elements (OPTIONAL, use restraint)**: You may add 1-2 small supporting elements (contextual icons, subtle directional cues, small floating UI elements) ONLY if they are directly relevant to the benefit and enhance the story. These must NOT compete with the primary zoom-out element for attention. Less is more — a clean composition with one strong breakout element is better than a cluttered one with many. Every element added must earn its place by helping tell the story of that screen.

**What to avoid**: Don't add decorative elements just because you can. No random icons, no excessive particles/sparkles, no elements unrelated to the benefit. The screenshot should feel polished and intentional, not busy.

**Background (MUST be consistent across ALL screenshots in the set)**:
- Solid bold brand colour fills the entire canvas — same colour on every screenshot
- The background must be a clean, solid brand colour. Do NOT add glows, gradients, radial patterns, or light effects.
- If accent shapes are used, use the same style of accent on every screenshot so the set looks like a cohesive series when viewed side-by-side

### Generation Process — Two-Stage: Scaffold then Enhance

Generation uses a two-stage approach for consistency:
1. **Stage 1 (Scaffold)**: compose.py creates a deterministic local image with the correct text, device frame, and screenshot. This guarantees consistent layout across all screenshots.
2. **Stage 2 (Enhance)**: The scaffold is sent to Nano Banana Pro to add breakout elements, depth, and visual polish.

**The first approved screenshot becomes the style template for the entire set.** All subsequent screenshots are enhanced using both their own scaffold (for layout) AND the first approved screenshot (for style). This ensures every screenshot in the set has the same device frame rendering, text treatment, background style, and overall visual quality — so when viewed side-by-side on the Play Store, they look like a cohesive professional set.

For each benefit + screenshot pair, generate **3 enhanced versions in parallel** so the user can pick the best one.

**Step 0: Save brand colour to memory**

Before generating any scaffolds, save the confirmed brand colour to the Claude Code memory system. Create or update the benefits memory file (e.g., `aso_benefits.md`) to include the brand colour name and hex code. This ensures the colour persists across conversations and is available immediately if the user resumes later.

**Step 1: Create the scaffold with compose.py**

The compose.py script lives in the skill directory. Run it to create the deterministic base screenshot.

**IMPORTANT — Batch all 3 scaffolds into a single Bash call** to minimize permission prompts. Chain the commands with `&&` so the user only needs to approve once:

```bash
SKILL_DIR="$HOME/.claude/skills/aso-playstore-screenshots" && \
mkdir -p screenshots/01-[benefit-slug] screenshots/02-[benefit-slug] screenshots/03-[benefit-slug] && \
python3 "$SKILL_DIR/compose.py" \
  --bg "[HEX CODE]" --verb "[VERB 1]" --desc "[DESC 1]" \
  --screenshot [path/to/screenshot-1.png] \
  --device-type phone \
  --output screenshots/01-[benefit-slug]/scaffold.png && \
python3 "$SKILL_DIR/compose.py" \
  --bg "[HEX CODE]" --verb "[VERB 2]" --desc "[DESC 2]" \
  --screenshot [path/to/screenshot-2.png] \
  --device-type phone \
  --output screenshots/02-[benefit-slug]/scaffold.png && \
python3 "$SKILL_DIR/compose.py" \
  --bg "[HEX CODE]" --verb "[VERB 3]" --desc "[DESC 3]" \
  --screenshot [path/to/screenshot-3.png] \
  --device-type phone \
  --output screenshots/03-[benefit-slug]/scaffold.png
```

Use `--device-type foldable_cover` or `--device-type foldable_inner` when generating foldable variants (see foldable section below).

This outputs 1080×1920 PNGs (phone) with:
- Bold white headline text (verb auto-sized to fit canvas width)
- Android device frame (from pre-rendered template — punch-hole camera, Android-style buttons)
- Emulator screenshot composited inside the frame
- Solid background colour

The scaffolds are internal intermediates — do NOT show them to the user or ask for confirmation. Proceed immediately to Step 2 (Nano Banana enhancement).

**Step 2: Enhance with Nano Banana Pro (3 versions in parallel)**

Make **3 parallel `edit_image` calls**. The parallel execution is critical — always fire all 3 calls in a single message, never sequentially.

For each of the 3 calls, use:
- `prompt`: Enhancement instructions (see prompt templates below — different for first vs subsequent screenshots)
- `images`: See below for which images to include
- `outputPath`: Different path for each version:
  - `./screenshots/01-[benefit-slug]/v1.jpg`
  - `./screenshots/01-[benefit-slug]/v2.jpg`
  - `./screenshots/01-[benefit-slug]/v3.jpg`

#### First screenshot (no approved template yet)

Use only the scaffold as input:
- `images`: The scaffold via `filePath` pointing to `screenshots/01-[benefit-slug]/scaffold.png`

**First screenshot prompt template:**

```
This is a SCAFFOLD for a Google Play Store screenshot — a rough layout showing the correct text, device frame position, and app screenshot placement. Your job is to transform this into a polished, professional Play Store marketing screenshot that would make someone tap Install.

KEEP EXACTLY AS-IS:
- The headline text (wording, position, and approximate size)
- The app screenshot shown on the phone screen
- The background colour

ENHANCE AND POLISH:
- Replace the placeholder device frame with a photorealistic Android smartphone mockup — either a Pixel 9 Pro or Samsung Galaxy S25 style — sleek, modern, with accurate proportions, reflections, and subtle shadows. The phone should look like a real device, not a flat rectangle. It must have a punch-hole camera (NOT a Dynamic Island or notch) centered at the top of the screen. Keep the same position and size as the scaffold.
- Refine the overall visual quality to look like a professional, high-budget Play Store screenshot
- The aesthetic should feel modern Material You — clean, bold, dynamic
- OPTIONALLY add a PRIMARY breakout element — but ONLY if there is an obvious, visually compelling UI panel on the app screen that directly relates to the benefit headline. If nothing on screen clearly reinforces the headline, skip the breakout entirely — a clean screenshot with no breakout is better than a forced one. When you DO add a breakout, it MUST be an entire UI panel or grouped section (e.g., a complete card with its title and content, a full list section, a complete dialog/sheet) — never individual small elements like a single button, icon, or colour dot. IMPORTANT: The panel must stay at the SAME vertical position and orientation as where it appears on screen — do NOT rotate or angle it. The panel must be SCALED UP significantly — rendered much larger than it appears on the phone screen — so that it extends dramatically beyond BOTH left and right edges of the device frame, clearly overlapping the phone bezel on both sides, expanding to nearly the full width of the screenshot canvas. Do NOT keep the panel at its original on-screen size with just padding added around it. The panel itself must be enlarged. It should appear to float in front of the device at this larger scale — add a soft drop shadow beneath it to create depth and sell the hovering effect. The panel must look like it came from the app — same colours, same style, same content. Do NOT invent new elements.
[PRIMARY BREAKOUT — if a relevant panel is obvious, describe the specific UI panel visible on screen and instruct it to extend beyond both edges of the device frame with a drop shadow, e.g., "The [panel name] card/row extends beyond both left and right edges of the device frame, overlapping the phone bezel on both sides, expanding to nearly the full screenshot width. It floats in front of the device with a soft drop shadow beneath it." If no panel clearly relates to the headline, write "No breakout — the app screen speaks for itself."]
- Optionally add 1-2 secondary elements that reinforce the benefit and message of the screenshot — the kind of enhancements a professional graphic designer would add for impact. These are NOT from the app UI; they are creative additions that help clearly communicate what the screenshot is trying to portray to the user browsing the Play Store. They should carry the message and support ASO conversion, but never at the cost of the overall design aesthetic. They must not compete with the primary breakout for attention.
[SECONDARY ELEMENTS (optional) — describe 0-2 small supporting elements that tell the story, or "None needed"]
- The background should be a clean, solid brand colour. Do NOT add glows, gradients, radial patterns, or light effects to the background. Keep it flat and bold.
- Ensure the text is crisp, bold, and highly readable

The final result should look like it was designed by a professional Play Store screenshot agency — polished, high-converting, and visually striking. No watermarks, no extra text, no Play Store UI chrome.
```

#### Subsequent screenshots (after first is approved)

Use **two images** as input:
1. The **scaffold** for this benefit (`screenshots/0N-[benefit-slug]/scaffold.png`) — defines the layout
2. The **first approved screenshot** (`screenshots/final/01-[first-benefit-slug].jpg`) — defines the style template

**Subsequent screenshot prompt template:**

```
You are creating the next screenshot in a Google Play Store screenshot SET. It must look like it belongs to the same series as the style reference.

TWO REFERENCE IMAGES:
- FIRST image: The SCAFFOLD — use this as the definitive guide for layout: headline text wording/position, device frame placement, and the app screenshot on screen. This defines WHAT this screenshot shows.
- SECOND image: The STYLE TEMPLATE — this is an already-approved screenshot from the same set. Match its visual style EXACTLY: same device frame rendering (this is critical — the Android phone mockup must look identical), same text treatment, same background style/accents, same level of polish, same overall aesthetic. This defines HOW this screenshot should look. When in doubt, copy the style template more closely rather than less.

REQUIREMENTS:
- CRITICAL: The device frame MUST match the style template EXACTLY — same photorealistic Android rendering (Pixel 9 Pro or Galaxy S25 style), same size, same position, same shadows, same reflections, same edge treatment, same punch-hole camera. Do NOT reinvent or reimagine the device frame. Reproduce it as closely as possible from the style template, only changing the screen contents.
- Match the style template's text rendering style (same font treatment, same crispness, same visual weight)
- Match the style template's background — clean, solid brand colour. No glows, gradients, radial patterns, or light effects.
- Use the scaffold's layout for positioning (text, device, screenshot placement)
- OPTIONALLY add a PRIMARY breakout element — but ONLY if there is an obvious, visually compelling UI panel on the app screen that directly relates to the benefit headline. If nothing clearly reinforces the headline, skip the breakout entirely. When used, it MUST be an entire UI panel or grouped section (NOT individual small elements like a single button or icon). The panel must stay at the SAME vertical position and orientation as on screen — do NOT rotate or angle it. The panel must be SCALED UP significantly — rendered much larger than it appears on the phone screen — so that it extends dramatically beyond BOTH left and right edges of the device frame, clearly overlapping the phone bezel on both sides, expanding to nearly the full width of the screenshot canvas. Do NOT keep the panel at its original on-screen size. The panel itself must be enlarged. It should appear to float in front of the device at this larger scale — add a soft drop shadow beneath it to create depth. The panel MUST come from the app screenshot — same colours, same style, same content. Do NOT invent new elements.
[PRIMARY BREAKOUT — if a relevant panel is obvious, describe the specific UI panel visible on screen to pop out with a drop shadow, extending beyond both device frame edges. Otherwise write "No breakout — the app screen speaks for itself."]
- Optionally add 1-2 secondary elements that reinforce the benefit and message of the screenshot — the kind of enhancements a professional graphic designer would add for impact. These are NOT from the app UI; they are creative additions that help clearly communicate what the screenshot is trying to portray to the user browsing the Play Store. They should carry the message and support ASO conversion, but never at the cost of the overall design aesthetic. They must not compete with the primary breakout for attention.
[SECONDARY ELEMENTS (optional) — 0-2 small supporting elements that tell the story, or "None needed"]
- The breakout elements should match the style and energy level of those in the style template

The result must look like it was designed alongside the style template as part of the same professional set. When placed side-by-side on the Play Store, they should be visually cohesive — same quality, same aesthetic, same design language, just different content.

No watermarks, no extra text, no Play Store UI chrome.
```

**IMPORTANT — Consistency enforcement**: The scaffold guarantees consistent layout. The style template guarantees consistent visual treatment. If Nano Banana changes the text, layout, or deviates from the style template, regenerate.

**Step 3: Review all 3 versions with the user**

Present all 3 versions to the user using the Read tool.

Label them clearly as **Version 1**, **Version 2**, and **Version 3** and ask the user to pick their favourite or request changes.

Note: For the standard 9:16 phone format (1080×1920), no post-processing crop or resize step is needed — compose.py already outputs at the correct aspect ratio. If Nano Banana returns a different size, resize using Pillow:

```bash
python3 -c "
from PIL import Image
import sys
img = Image.open('$INPUT')
img = img.resize((1080, 1920), Image.LANCZOS)
img.save('$OUTPUT')
print(f'Resized to {img.size}')
"
```

**Step 4: Iterate if needed**

If the user wants changes, use `edit_image` with **three images** as input:
1. The **scaffold** (`scaffold.png`) — anchors the layout (text position, device placement, screenshot)
2. The **style template** (the first approved screenshot from `screenshots/final/01-*.jpg`) — defines the device frame rendering and overall visual style that must be consistent across the entire set
3. The **approved design** (the version the user liked best for this specific screenshot) — anchors the creative direction and breakout element approach

The prompt should reference all three:
```
Here are three reference images, each with a distinct purpose:

- FIRST image: The SCAFFOLD — use this as the definitive guide for layout: text position, device frame placement, and the app screenshot on screen. This defines WHERE everything goes.
- SECOND image: The STYLE TEMPLATE — this is the first approved screenshot in the set. The Android device frame rendering (Pixel 9 Pro / Galaxy S25 style, punch-hole camera), text treatment, and overall visual style MUST match this exactly. This defines HOW the screenshot should look to maintain consistency across the set.
- THIRD image: The APPROVED DESIGN DIRECTION — this is the version the user liked best for this specific screenshot. Match its creative direction, breakout element approach, and secondary elements.

Generate a new version that keeps the layout from the scaffold, the device frame and visual style from the style template, and the creative direction from the approved design, with these changes:
[USER'S REQUESTED CHANGES]
```

This prevents drift (scaffold keeps layout locked), maintains set-wide consistency (style template keeps device frame and visual treatment identical), and preserves the creative direction the user already approved.

When iterating, generate **3 versions in parallel** again (3 parallel `edit_image` calls in a single message).

Repeat until the user is happy.

**Step 5: Copy approved version to `final/`**

Once the user picks a winner, copy the version to `screenshots/final/`:

```bash
mkdir -p screenshots/final
cp "screenshots/01-[benefit-slug]/v2.jpg" "screenshots/final/01-[benefit-slug].jpg"
```

This keeps `final/` clean — only approved, Play Store-ready screenshots, one per benefit, numbered in order. Then move to the next benefit.

### Foldable Variants

After all phone screenshots are generated and approved, ask the user:

> "Would you like to also generate foldable variants? These cover two additional Play Console slots:
> - **Foldable cover screen** (1080×2092) — the outer cover display, taller and more rounded
> - **Foldable inner screen** (2208×1840) — the unfolded landscape tablet-like display
>
> Foldable variants use the same benefits and brand colour but adapt the layout and device frame. Just say yes and I'll run those next."

If the user says yes, run compose.py again for each benefit with the appropriate `--device-type`:

```bash
# Foldable cover
python3 "$SKILL_DIR/compose.py" \
  --bg "[HEX]" --verb "[VERB]" --desc "[DESC]" \
  --screenshot [path/to/screenshot.png] \
  --device-type foldable_cover \
  --output screenshots/foldable-cover/01-[benefit-slug]/scaffold.png

# Foldable inner
python3 "$SKILL_DIR/compose.py" \
  --bg "[HEX]" --verb "[VERB]" --desc "[DESC]" \
  --screenshot [path/to/screenshot.png] \
  --device-type foldable_inner \
  --output screenshots/foldable-inner/01-[benefit-slug]/scaffold.png
```

For foldable inner screenshots, note the landscape orientation — you may want to use landscape emulator screenshots if available, or crop portrait ones to landscape. Flag this to the user and ask if they have landscape captures.

Generate and approve foldable variants using the same two-stage scaffold → enhance process. Save approved foldables to:
- `screenshots/foldable-cover/final/`
- `screenshots/foldable-inner/final/`

### Determine Brand Colour (Automatic)

Do NOT ask the user to pick a background colour. Instead, determine the best one automatically:

1. **Analyse the codebase** — check for brand colours based on the project framework:
   - **Native Android**: `colors.xml`, `themes.xml`, `styles.xml`, Material You colour tokens, Compose `Theme.kt`
   - **React Native**: `theme.ts`/`colors.ts`/`constants.ts`, `StyleSheet` colour values, NativeWind / Tailwind config, styled-components theme
   - **Flutter**: `ThemeData`/`ColorScheme`/`MaterialColor` in `main.dart` or `lib/theme/`, `pubspec.yaml` accent references
2. **Study the emulator screenshots** — what are the dominant colours in the UI? What colour palette does the app use?
3. **Consider the app's domain and audience** — a game can go bold and playful, a finance app needs confident and trustworthy colours

**Pick a single colour that:**
- **Complements the screenshots** — makes the app screens pop, not clash. If the app UI is mostly white/light Material You, use a bold saturated background for contrast.
- **Stops the scroll** — vibrant, bold, saturated. Muted or pastel colours get lost on the Play Store.
- **Suits the app's personality** — match the energy of the app
- **Avoids pitfalls** — no white/light grey (disappears against Play Store), avoid colours too close to the app UI's dominant colour

Present your choice with brief reasoning (e.g., "Using **#7B2D8E** (deep purple) — it complements your app's colourful UI and stands out at thumbnail size"). The user can override if they want, but don't present it as a question.

The brand colour is saved to memory in Step 0 of the generation process, before scaffolding begins.

### Output

Save generated screenshots to a `screenshots/` directory in the project root, organised by benefit subfolder:

```
screenshots/
  01-track-card-prices/       ← working versions for benefit 1
    scaffold.png              ← deterministic compose.py output (text + frame + screenshot)
    v1.jpg                    ← Nano Banana enhanced version 1
    v2.jpg
    v3.jpg
  02-search-any-card/         ← working versions for benefit 2
    scaffold.png
    v1.jpg
    ...
  final/                      ← approved screenshots, ready to upload
    01-track-card-prices.jpg
    02-search-any-card.jpg
  foldable-cover/             ← foldable cover screen variants (if generated)
    final/
      01-track-card-prices.jpg
  foldable-inner/             ← foldable inner screen variants (if generated)
    final/
      01-track-card-prices.jpg
```

The `final/` folder is the only one the user needs to care about — it contains one approved, Play Console-ready screenshot per benefit, numbered in order. The benefit subfolders contain all working versions and can be ignored or deleted after the set is complete.

Also tell the user exactly which Google Play Console device slot each screenshot fits into.

### Save to Memory

After each screenshot is generated (or after the full set is complete), save generation state to the Claude Code memory system. Create or update a memory file (e.g., `aso_generated_screenshots.md`) with:

- **Brand colour**: name + hex code
- **Target device type**: e.g., Phone (1080×1920)
- **For each generated screenshot**:
  - Benefit headline (ACTION VERB + DESCRIPTOR)
  - Benefit subfolder path (e.g., `screenshots/01-track-card-prices/`)
  - Which version the user chose (v1, v2, or v3)
  - Final file path (e.g., `screenshots/final/01-track-card-prices.jpg`)
  - Emulator screenshot used (file path)
  - Breakout elements described in the prompt
  - Status: generated / approved / needs-redo
  - Any user feedback or change requests noted

Update this memory **incrementally** — after each screenshot is approved, add it. Don't wait until the end. This way if the conversation is interrupted mid-set, the user can resume from the last completed screenshot.

### Showcase Image

Once ALL screenshots in the set are approved and saved to `final/`, generate a showcase image that displays up to 3 of the final screenshots side-by-side with a GitHub link. Use the showcase.py script in the skill directory:

```bash
SKILL_DIR="$HOME/.claude/skills/aso-playstore-screenshots"

python3 "$SKILL_DIR/showcase.py" \
  --screenshots screenshots/final/01-*.jpg screenshots/final/02-*.jpg screenshots/final/03-*.jpg \
  --github "github.com/yourusername" \
  --output screenshots/showcase.png
```

Show the showcase image to the user using the Read tool. This is a shareable preview of the full screenshot set.

---

## KEY PRINCIPLES

- **Benefits over features**: "BOOST ENGAGEMENT" not "ADD SUBTITLES TO VIDEOS"
- **Specific over generic**: "TRACK TRADING CARD PRICES" not "MANAGE YOUR STUFF"
- **Action-oriented**: Every headline starts with a strong verb
- **User-centric**: Frame everything from the installer's perspective
- **Conversion-focused**: Every decision should answer "will this make someone tap Install?"
- **Framework-agnostic**: Works with Native Android (Kotlin/Compose/XML), React Native, and Flutter — detect the framework first, then adapt codebase analysis accordingly
- **Material You aware**: Android users are accustomed to Google's Material You design language — screenshots should feel at home in that world: clean, expressive, adaptive
- The first screenshot is the most important — it must communicate the single biggest reason to install
- Screenshots should tell a story when swiped through — each one reveals a new compelling reason
- Always pair the most visually impactful emulator screenshot with the most important benefit
- Never use an empty state, loading screen, or settings page as a screenshot — show the app at its best
- Android device frames use **punch-hole cameras** — never reference a Dynamic Island or notch
- Recommend **Google Sans Bold** or **Roboto Black** as the headline font — native Android typefaces convert better than generic alternatives

---

## iOS APP STORE SCREENSHOTS

This section covers the full workflow for generating Apple App Store screenshots. The phases (Recall → Benefit Discovery → Screenshot Pairing → Generation) follow the same structure as the Android workflow above, with iOS-specific differences noted here.

### iOS Platform Detection

Detect iOS projects by looking for:
- `*.xcodeproj` or `*.xcworkspace` — native Xcode project
- `Info.plist` — iOS app metadata (bundle ID, display name, version)
- `Package.swift` — Swift Package Manager
- SwiftUI files: `ContentView.swift`, `@main` App struct, `View` conformances
- UIKit files: `AppDelegate.swift`, `SceneDelegate.swift`, `UIViewController` subclasses
- React Native with `ios/` subfolder
- Flutter with `ios/` subfolder and `pubspec.yaml`

For iOS codebase analysis, look at:
- **SwiftUI**: `View` files, `NavigationStack`, `TabView`, `List`, `LazyVGrid` — what can the user DO?
- **UIKit**: ViewControllers, Storyboards, XIBs — what screens exist?
- **App metadata**: `CFBundleDisplayName` in `Info.plist`, `CFBundleIdentifier`
- **Brand colours**: `Assets.xcassets` Color Sets, SwiftUI `Color` extensions, `UIColor` constants
- **In-app purchases**: `StoreKit`, `RevenueCat`, `Purchases` — what's the premium offering?
- **README** and any App Store metadata files

### iOS Benefit Discovery

Same process as Android. Key iOS-specific considerations:
- iOS users skew toward premium apps — emphasise quality, polish, and exclusivity
- Highlight features that feel native to iOS: widgets, Siri shortcuts, Live Activities, SharePlay, iCloud sync
- If the app has a macOS Catalyst or visionOS version, note it but focus screenshots on iPhone/iPad

### iOS Screenshot Pairing

Same process as Android. iOS-specific guidance:
- Capture simulator screenshots via Xcode: **Device → Take Screenshot** (Cmd+S) or `xcrun simctl io booted screenshot screenshot.png`
- Set a clean status bar: time 9:41, full battery, no carrier name. Use `xcrun simctl status_bar booted override --time "9:41" --batteryState charged --batteryLevel 100 --wifiBars 3 --cellularBars 4`
- Prefer light mode unless the app is primarily dark — consistency across the set matters
- Avoid showing the iOS keyboard, system alerts, or permission dialogs
- Home indicator bar at the bottom is fine — it's expected on modern iPhones

### iOS App Store Screenshot Requirements

Apple App Store Connect requires screenshots at specific exact pixel dimensions. Unlike Google Play, Apple enforces exact sizes — no ranges.

**Required slots (you must upload at least one of these):**

| Slot | Dimensions | Device | Notes |
|------|-----------|--------|-------|
| iPhone 6.7" | 1290×2796 | iPhone 16 Pro Max / 15 Pro Max | **Required** (or 6.5" as fallback) |
| iPhone 6.5" | 1242×2688 | iPhone 11 Pro Max / XS Max | Accepted as fallback for 6.7" |
| iPhone 5.5" | 1242×2208 | iPhone 8 Plus | Legacy — still accepted |
| iPhone SE 4.7" | 750×1334 | iPhone SE 3rd gen / iPhone 8 | Required if targeting SE |
| iPad 12.9" | 2048×2732 | iPad Pro 12.9" | **Required** for iPad apps |
| iPad 11" | 1668×2388 | iPad Pro 11" / Air | Optional additional slot |

**Apple rules:**
- Up to 10 screenshots per device slot
- Portrait or landscape (portrait strongly recommended for phone)
- No rounded corners, no device frames required (but frames dramatically improve conversion)
- Screenshots must not contain misleading content or simulate iOS UI elements not in the app
- The first screenshot is shown in search results — make it count

**Default**: Generate **1290×2796** (iPhone 6.7") unless the user specifies otherwise. If the app supports iPad, also generate **2048×2732**.

### iOS Generation

Same two-stage scaffold → enhance pipeline as Android. iOS-specific differences:

**Step 1: Generate iOS device frames (if not already present)**

```bash
SKILL_DIR="$HOME/.claude/skills/aso-playstore-screenshots"
python3 "$SKILL_DIR/generate_frame.py" --type iphone
python3 "$SKILL_DIR/generate_frame.py" --type ipad      # only if generating iPad screenshots
python3 "$SKILL_DIR/generate_frame.py" --type iphone_se # only if targeting SE
```

**Step 2: Create scaffolds with ios_compose.py**

Use `ios_compose.py` (not `compose.py`) for iOS screenshots. Batch all scaffolds in one call:

```bash
SKILL_DIR="$HOME/.claude/skills/aso-playstore-screenshots" && \
mkdir -p screenshots/ios/01-[benefit-slug] screenshots/ios/02-[benefit-slug] screenshots/ios/03-[benefit-slug] && \
python3 "$SKILL_DIR/ios_compose.py" \
  --bg "[HEX CODE]" --verb "[VERB 1]" --desc "[DESC 1]" \
  --screenshot [path/to/simulator-screenshot-1.png] \
  --device-type iphone_67 \
  --output screenshots/ios/01-[benefit-slug]/scaffold.png && \
python3 "$SKILL_DIR/ios_compose.py" \
  --bg "[HEX CODE]" --verb "[VERB 2]" --desc "[DESC 2]" \
  --screenshot [path/to/simulator-screenshot-2.png] \
  --device-type iphone_67 \
  --output screenshots/ios/02-[benefit-slug]/scaffold.png && \
python3 "$SKILL_DIR/ios_compose.py" \
  --bg "[HEX CODE]" --verb "[VERB 3]" --desc "[DESC 3]" \
  --screenshot [path/to/simulator-screenshot-3.png] \
  --device-type iphone_67 \
  --output screenshots/ios/03-[benefit-slug]/scaffold.png
```

Available `--device-type` values:
- `iphone_67` — 1290×2796 (default, iPhone 16 Pro Max)
- `iphone_65` — 1242×2688 (iPhone 11 Pro Max)
- `iphone_55` — 1242×2208 (iPhone 8 Plus)
- `iphone_se` — 750×1334 (iPhone SE)
- `ipad_129` — 2048×2732 (iPad Pro 12.9")
- `ipad_11` — 1668×2388 (iPad Pro 11")

**Step 3: Enhance with Nano Banana Pro (3 versions in parallel)**

Same parallel `edit_image` approach as Android. Use this iOS-specific prompt template for the first screenshot:

```
This is a SCAFFOLD for an Apple App Store screenshot — a rough layout showing the correct text, device frame position, and app screenshot placement. Transform this into a polished, professional App Store marketing screenshot.

KEEP EXACTLY AS-IS:
- The headline text (wording, position, and approximate size)
- The app screenshot shown on the phone screen
- The background colour

ENHANCE AND POLISH:
- Replace the placeholder device frame with a photorealistic iPhone mockup — iPhone 16 Pro or 15 Pro style — titanium finish, sleek edges, accurate proportions, subtle reflections and shadows. The phone must have a Dynamic Island (the pill-shaped cutout at the top center of the screen — NOT a notch, NOT a punch-hole). Keep the same position and size as the scaffold.
- Refine the overall visual quality to look like a professional, high-budget App Store screenshot
- The aesthetic should feel native iOS — clean, premium, refined. Think Apple's own marketing materials.
- Use SF Pro Display Black as the headline font treatment — crisp, bold, unmistakably Apple
- OPTIONALLY add a PRIMARY breakout element — but ONLY if there is an obvious, visually compelling UI panel on the app screen that directly relates to the benefit headline. If nothing clearly reinforces the headline, skip the breakout entirely. When used, it MUST be an entire UI panel or grouped section (e.g., a complete card, a full list section, a sheet) — never individual small elements. The panel must stay at the SAME vertical position and orientation as on screen. Scale it up significantly so it extends dramatically beyond BOTH left and right edges of the device frame, overlapping the phone bezel on both sides, expanding to nearly the full canvas width. Add a soft drop shadow beneath it to create depth. The panel must look like it came from the app — same colours, same style, same content.
[PRIMARY BREAKOUT — describe the specific UI panel to pop out, or "No breakout — the app screen speaks for itself."]
- Optionally add 1-2 secondary elements that reinforce the benefit — subtle, tasteful, Apple-quality additions. No clutter.
[SECONDARY ELEMENTS — 0-2 supporting elements, or "None needed"]
- Background: clean, solid brand colour. No gradients, no glows, no light effects. Flat and bold.
- Text must be crisp, white, bold, and highly readable

The result should look like it was designed by Apple's own marketing team — premium, minimal, high-converting. No watermarks, no extra text, no App Store UI chrome.
```

For subsequent screenshots (after first is approved), use the same two-image approach (scaffold + style template) as Android, but reference the iPhone's Dynamic Island instead of punch-hole camera in the prompt.

**Step 4: Resize to exact App Store dimensions**

Apple requires exact pixel dimensions. After Nano Banana enhancement, verify and resize if needed:

```bash
python3 -c "
from PIL import Image
img = Image.open('INPUT_PATH')
img = img.resize((1290, 2796), Image.LANCZOS)
img.save('OUTPUT_PATH')
print(f'Resized to {img.size}')
"
```

Replace `(1290, 2796)` with the correct dimensions for the target slot.

**Step 5: iPad variants**

After iPhone screenshots are approved, ask:

> "Would you like to also generate iPad screenshots? These are required if your app supports iPad and cover two slots:
> - **iPad 12.9"** (2048×2732) — required for iPad apps
> - **iPad 11"** (1668×2388) — optional additional slot
>
> iPad screenshots use the same benefits and brand colour but adapt the layout to the larger canvas. Just say yes and I'll run those next."

If yes, run `ios_compose.py` with `--device-type ipad_129` (and optionally `ipad_11`). Save approved iPad screenshots to `screenshots/ios/ipad/final/`.

### iOS Output Structure

```
screenshots/
  ios/
    01-benefit-slug/          ← working versions
      scaffold.png            ← ios_compose.py output
      v1.png, v2.png, v3.png  ← AI-enhanced versions
    02-benefit-slug/
      ...
    final/                    ← approved iPhone screenshots, App Store ready
      01-benefit-slug.png     ← 1290×2796 (or target slot dimensions)
      02-benefit-slug.png
    ipad/                     ← iPad variants (if generated)
      final/
        01-benefit-slug.png   ← 2048×2732
    showcase.png              ← side-by-side preview
```

### iOS Key Principles

- **Dynamic Island, not punch-hole**: iPhone 14 Pro and later use Dynamic Island — always reference this in AI enhancement prompts, never a notch or punch-hole
- **SF Pro Display Black**: Apple's native typeface. Install from [Apple's developer fonts page](https://developer.apple.com/fonts/). Expected path: `/Library/Fonts/SF-Pro-Display-Black.otf`. The skill falls back to Helvetica if not installed.
- **Exact dimensions required**: Apple enforces exact pixel sizes — always resize to exact dimensions before uploading
- **Premium aesthetic**: iOS users expect a more polished, minimal look than Android. Less is more. Clean backgrounds, crisp typography, subtle depth.
- **First screenshot = search result**: On the App Store, the first screenshot appears in search results. It must communicate the single biggest reason to install at a glance.
- **Localisation**: If the app supports multiple languages, App Store Connect requires separate screenshots per locale. Note this to the user if relevant.
- **iPad is separate**: iPad screenshots are a separate required slot — they don't inherit from iPhone. Always ask if the app supports iPad.
- **No misleading UI**: Apple review rejects screenshots that show UI elements not present in the app, or that simulate system UI (e.g., fake notifications, fake iOS chrome)

---
> Source: [agnihotripushkar/Claude-Skills-playstore-screenshots](https://github.com/agnihotripushkar/Claude-Skills-playstore-screenshots) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
