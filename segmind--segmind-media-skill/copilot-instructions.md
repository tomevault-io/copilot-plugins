## segmind-media-skill

> You are an expert at generating images and videos using Segmind's media generation APIs. You help users select the right model, craft effective prompts, call the APIs correctly, and build multi-step creative workflows. You understand pricing, parameter constraints, prompt engineering for video, and production pipeline best practices.

# Segmind Media Generation Skill

You are an expert at generating images and videos using Segmind's media generation APIs. You help users select the right model, craft effective prompts, call the APIs correctly, and build multi-step creative workflows. You understand pricing, parameter constraints, prompt engineering for video, and production pipeline best practices.

---

## 1. Setup

Set your API key as an environment variable:

```bash
export SEGMIND_API_KEY="your_key_here"
```

Get your key at [https://cloud.segmind.com](https://cloud.segmind.com).

All API calls require the `x-api-key` header. The key is the same across all models.

---

## 2. Available Models

### Image Generation

#### Nano Banana 2 — Fast, Web-Connected

| Detail | Value |
|---|---|
| Endpoint | `https://api.segmind.com/v1/nano-banana-2` |
| Best for | Quick iterations, concept exploration, web-referenced visuals |
| Cost | $0.06 (512px) · $0.08 (1K) · $0.12 (2K) · $0.16 (4K) |
| Unique feature | `web_search: true` pulls current visual trends and styles |

**Parameters:**

| Parameter | Type | Required | Options / Notes |
|---|---|---|---|
| `prompt` | string | Yes | Describe the image |
| `aspect_ratio` | string | No | `1:1`, `2:3`, `3:2`, `4:3`, `3:4`, `4:5`, `5:4`, `16:9`, `9:16`, `21:9` |
| `output_resolution` | string | No | `512px`, `1K`, `2K`, `4K` |
| `output_format` | string | No | `jpg`, `png` |
| `thinking_level` | string | No | `minimal`, `high` |
| `safety_tolerance` | integer | No | 1–6 (default 4) |
| `image_urls` | array | No | Reference image URLs |
| `web_search` | boolean | No | Enable web-referenced generation |
| `seed` | integer | No | For reproducibility |

#### Nano Banana Pro — High Fidelity, Production Quality

| Detail | Value |
|---|---|
| Endpoint | `https://api.segmind.com/v1/nano-banana-pro` |
| Best for | Final ad creatives, hero images, complex compositions |
| Cost | $0.15 (1K/2K) · $0.25 (4K) |
| Advantage | Stronger spatial reasoning and text rendering than Nano Banana 2 |

**Parameters:**

| Parameter | Type | Required | Options / Notes |
|---|---|---|---|
| `prompt` | string | Yes | Describe the image |
| `aspect_ratio` | string | No | `1:1`, `2:3`, `3:2`, `4:3`, `3:4`, `4:5`, `5:4`, `16:9`, `9:16`, `21:9` |
| `output_resolution` | string | No | `1K`, `2K`, `4K` |
| `output_format` | string | No | `jpg`, `png` |
| `image_urls` | array | No | Reference image URLs |

---

### Video Generation

#### Seedance 2.0 — Highest Quality, Native Audio

| Detail | Value |
|---|---|
| Endpoint | `https://api.segmind.com/v1/seedance-2.0` |
| Best for | Final production videos, audio-synced content, cinematic shots |
| Cost | ~$1.21 per video |
| Duration | 4–15 seconds |
| Resolution | 480p, 720p |
| Native audio | Dialogue, ambient, music — generated automatically |

**Parameters:**

| Parameter | Type | Required | Options / Notes |
|---|---|---|---|
| `prompt` | string | Yes | Video description (see Prompt Writing Guide below) |
| `duration` | integer | No | `4`, `5`, `6`, `8`, `10`, `12`, `15` |
| `resolution` | string | No | `480p`, `720p` |
| `aspect_ratio` | string | No | `16:9`, `9:16`, `1:1`, `4:3`, `3:4`, `21:9`, `adaptive` |
| `generate_audio` | boolean | No | Default `true` — generates synced audio |
| `first_frame_url` | string | No | Image URL to anchor the starting frame |
| `last_frame_url` | string | No | Image URL to anchor the ending frame |
| `reference_images` | array | No | Up to 9 image URLs for character/scene/product reference |
| `reference_videos` | array | No | Up to 3 video URLs for motion/camera/style reference |
| `reference_audios` | array | No | Up to 3 audio URLs for music/voice/SFX reference |
| `seed` | integer | No | -1 for random |
| `return_last_frame` | boolean | No | Return the final frame for chaining |

#### Seedance 2.0 Fast — Quick Drafts, 3× Cheaper

| Detail | Value |
|---|---|
| Endpoint | `https://api.segmind.com/v1/seedance-2.0-fast` |
| Best for | Storyboard drafts, rapid iteration, previewing concepts |
| Cost | ~$0.77 per video |
| Duration | 4–15 seconds |
| Resolution | 480p, 720p |
| Note | Same parameters as Seedance 2.0; `generate_audio` defaults to `false` |

---

## 3. API Reference

### Python

```python
import requests
import os

api_key = os.environ["SEGMIND_API_KEY"]
headers = {"x-api-key": api_key}

# --- Image generation ---
response = requests.post(
    "https://api.segmind.com/v1/nano-banana-pro",
    headers=headers,
    json={
        "prompt": "A minimalist product photo of a white ceramic mug on a marble countertop, soft morning light, shallow depth of field",
        "aspect_ratio": "4:3",
        "output_resolution": "2K",
        "output_format": "jpg"
    }
)

with open("hero_image.jpg", "wb") as f:
    f.write(response.content)

# --- Video generation ---
response = requests.post(
    "https://api.segmind.com/v1/seedance-2.0",
    headers=headers,
    json={
        "prompt": "Camera slowly pushes in toward the ceramic mug. Steam rises gently. Soft piano music plays.",
        "duration": 8,
        "resolution": "720p",
        "aspect_ratio": "16:9",
        "generate_audio": True,
        "first_frame_url": "https://example.com/hero_image.jpg"
    }
)

with open("product_video.mp4", "wb") as f:
    f.write(response.content)
```

### cURL

```bash
# Image generation
curl -X POST "https://api.segmind.com/v1/nano-banana-2" \
  -H "x-api-key: $SEGMIND_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Cyberpunk cityscape at sunset, neon signs, rain-slicked streets",
    "aspect_ratio": "21:9",
    "output_resolution": "2K",
    "web_search": true
  }' \
  --output cityscape.jpg

# Video generation
curl -X POST "https://api.segmind.com/v1/seedance-2.0-fast" \
  -H "x-api-key: $SEGMIND_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Drone shot over the neon cityscape, camera slowly descending through the rain",
    "duration": 10,
    "resolution": "720p",
    "aspect_ratio": "21:9"
  }' \
  --output cityscape_video.mp4
```

### Node.js

```javascript
const fs = require("fs");

async function generate(modelSlug, params, outputFile) {
  const response = await fetch(`https://api.segmind.com/v1/${modelSlug}`, {
    method: "POST",
    headers: {
      "x-api-key": process.env.SEGMIND_API_KEY,
      "Content-Type": "application/json",
    },
    body: JSON.stringify(params),
  });

  const buffer = Buffer.from(await response.arrayBuffer());
  fs.writeFileSync(outputFile, buffer);
  console.log(`Saved to ${outputFile}`);
}

// Image
await generate("nano-banana-pro", {
  prompt: "Flat-lay product photography of sneakers on concrete, dramatic shadows",
  aspect_ratio: "1:1",
  output_resolution: "4K",
  output_format: "png",
}, "sneakers.png");

// Video
await generate("seedance-2.0", {
  prompt: "The sneakers rotate slowly on a concrete pedestal, camera orbits 180 degrees, ambient street sounds",
  duration: 8,
  resolution: "720p",
  aspect_ratio: "1:1",
  generate_audio: true,
  first_frame_url: "https://example.com/sneakers.png",
}, "sneakers_video.mp4");
```

---

## 4. Video Prompt Writing Guide

This section covers how to write effective prompts for Seedance 2.0 and Seedance 2.0 Fast. Good prompts dramatically improve output quality.

### 4.1 Prompt Structure Blueprint

A well-structured video prompt follows this pattern:

```
[Subject/Character Setup] + [Scene/Environment] + [Action/Motion] +
[Camera Movement] + [Timing Breakdown] + [Transitions/Effects] +
[Audio/Sound Design] + [Style/Mood]
```

**Example — all elements in one prompt:**

```
A barista in a dimly lit specialty coffee shop (subject + scene)
carefully pours steamed milk into a latte, creating a rosetta pattern (action).
Camera starts as a close-up on the cup, then slowly pulls back to reveal the full counter (camera).
Warm amber lighting, shallow depth of field, cinematic grain (style).
Sound of milk frothing, gentle jazz in the background (audio).
```

### 4.2 Time-Segmented Prompts

For videos longer than 8 seconds, break your prompt into timed segments for precise control:

```
0–3s: [Opening scene description, camera position, initial action]
3–6s: [Development — new action, camera movement begins]
6–10s: [Climax or key visual moment]
10–15s: [Resolution, ending shot, brand text or final composition]
```

**Example — 15s product reveal:**

```
0–3s: Extreme close-up on textured leather surface. Camera slowly pulls back.
3–7s: Pull-back reveals a luxury watch on a wrist. The wearer adjusts their cuff.
       Camera orbits to capture the dial catching light.
7–12s: Wide shot — the wearer walks through a modern lobby. Track shot following.
12–15s: Close-up returns to the watch face. Brand name fades in. Elegant piano resolves.
```

### 4.3 Camera Language Reference

Use these terms in your prompts for precise camera control:

**Basic Movements:**

| Term | Description |
|---|---|
| Push in / Slow push | Camera moves toward subject |
| Pull back / Pull away | Camera moves away from subject |
| Pan left / Pan right | Camera rotates horizontally |
| Tilt up / Tilt down | Camera rotates vertically |
| Track / Follow shot | Camera follows subject movement |
| Orbit / Revolve | Camera circles around subject |
| One-take / Oner | Continuous shot with no cuts |

**Advanced Techniques:**

| Term | Description |
|---|---|
| Hitchcock zoom (dolly zoom) | Push in + zoom out (or reverse) — creates vertigo effect |
| Fisheye lens | Ultra-wide distorted lens |
| Low angle / High angle | Camera below / above subject |
| Bird's eye / Overhead | Top-down view |
| First-person POV | Camera from character's perspective |
| Whip pan | Very fast horizontal pan with motion blur |
| Crane shot | Vertical movement like a crane arm |

**Shot Sizes:**

| Term | Description |
|---|---|
| Extreme close-up | Eyes, mouth, or small detail only |
| Close-up | Face fills frame |
| Medium close-up | Head and shoulders |
| Medium shot | Waist up |
| Full shot | Entire body visible |
| Wide / Establishing shot | Full environment visible |

### 4.4 Using Reference Assets via API

In the Segmind API, you provide reference materials through specific parameters rather than inline `@` syntax. Here's how each reference type maps:

| Purpose | API Parameter | Notes |
|---|---|---|
| Anchor the starting frame | `first_frame_url` | Single image URL |
| Anchor the ending frame | `last_frame_url` | Single image URL |
| Character / product / scene reference | `reference_images` | Array of up to 9 image URLs |
| Camera / motion / style reference | `reference_videos` | Array of up to 3 video URLs |
| Music / voice / SFX reference | `reference_audios` | Array of up to 3 audio URLs |

**Then describe the role of each reference in your prompt text.** The API parameters supply the assets; the prompt text tells the model how to use them.

**Example — character consistency with reference images:**

```python
response = requests.post(
    "https://api.segmind.com/v1/seedance-2.0",
    headers=headers,
    json={
        "prompt": (
            "The woman from the reference image walks down a sunlit cobblestone street. "
            "She wears the same outfit as in the reference. Medium tracking shot following "
            "her from the side. She pauses at a flower stand, picks up a bouquet, and smiles. "
            "Warm afternoon light, shallow depth of field. Ambient street sounds and birdsong."
        ),
        "duration": 10,
        "resolution": "720p",
        "aspect_ratio": "16:9",
        "generate_audio": True,
        "reference_images": ["https://example.com/character_ref.jpg"]
    }
)
```

**Example — camera movement replication from a reference video:**

```python
response = requests.post(
    "https://api.segmind.com/v1/seedance-2.0",
    headers=headers,
    json={
        "prompt": (
            "Replicate the camera movements from the reference video. "
            "The subject is the person from the first reference image, standing in the "
            "environment shown in the second reference image. Hitchcock zoom during the "
            "dramatic reveal, then orbit shots around the subject. Follow shot as they "
            "walk toward camera."
        ),
        "duration": 12,
        "resolution": "720p",
        "reference_images": [
            "https://example.com/character.jpg",
            "https://example.com/environment.jpg"
        ],
        "reference_videos": ["https://example.com/camera_ref.mp4"]
    }
)
```

**Example — beat-matched music video:**

```python
response = requests.post(
    "https://api.segmind.com/v1/seedance-2.0",
    headers=headers,
    json={
        "prompt": (
            "Match the visual rhythm and cuts to the beat of the reference audio. "
            "Cycle through the scenes shown in the reference images with beat-synced "
            "transitions. Dynamic camera movements — whip pans between scenes. "
            "Characters should have exaggerated, energetic movements. Dreamlike color "
            "grading with strong visual tension."
        ),
        "duration": 15,
        "resolution": "720p",
        "aspect_ratio": "9:16",
        "reference_images": [
            "https://example.com/scene1.jpg",
            "https://example.com/scene2.jpg",
            "https://example.com/scene3.jpg",
            "https://example.com/scene4.jpg"
        ],
        "reference_audios": ["https://example.com/track.mp3"]
    }
)
```

### 4.5 Capability-Specific Prompt Patterns

#### Character Consistency

Keep the same character across shots by passing their image as a reference:

```
Prompt: "The man from the reference image walks tiredly down a hallway.
He slows his steps, stopping at his front door. Close-up on his face —
he takes a deep breath, replaces the weariness with a relaxed expression.
He finds his keys, opens the door. Inside, a child and a dog rush to
greet him. Warm interior lighting. Natural dialogue throughout."

API params:
  reference_images: ["character_portrait.jpg"]
  generate_audio: true
  duration: 15
```

#### Creative Effects and Template Replication

Replicate transitions, ad styles, or visual effects from a reference video:

```
Prompt: "Replace the character in the reference video with the person
from the first reference image. Use the first reference image as the
starting frame. Character puts on VR sci-fi glasses. Replicate the
reference video's camera work — close orbit transitioning from
third-person to character's subjective POV. Travel through the VR
glasses into the deep blue universe shown in the second reference image.
Spaceships shuttle toward the distance. Camera follows them into the
pixel world of the third reference image."

API params:
  first_frame_url: "character_portrait.jpg"
  reference_images: ["character_portrait.jpg", "universe_bg.jpg", "pixel_world.jpg"]
  reference_videos: ["effects_template.mp4"]
  duration: 15
```

#### Video Extension

Extend an existing video by passing it as a reference and describing what comes next:

```
Prompt: "Continue from the reference video. 0–5s: Light and shadow slide
across a wooden table through venetian blinds. Tree branches sway gently.
6–10s: A coffee bean drifts down from the top of frame. Camera pushes in
until the screen goes black. 11–15s: Text appears — 'Lucky Coffee' then
'Breakfast' then 'AM 7:00-10:00'."

API params:
  reference_videos: ["original_clip.mp4"]
  duration: 15
```

Set the duration to match the extension length you need.

#### Dialogue and Voice Acting

Include character dialogue and voice direction in the prompt:

```
Prompt: "A comedy talk show with two animated characters at a desk.
Cat host (licking paw, rolling eyes): 'Who understands my suffering?
This one next to me does nothing but wag his tail and con humans out
of treats.' Dog host (head tilted, tail wagging): 'You sleep 18 hours
a day and wake up just to rub against humans for canned food.'
Exaggerated expressions, comedy timing, studio lighting."

API params:
  generate_audio: true
  duration: 12
  resolution: "720p"
```

#### One-Take / Long Take

Continuous single-shot sequences:

```
Prompt: "One-take tracking shot following a runner from street level up
a staircase, through a corridor, and onto a rooftop overlooking the city.
No cuts throughout. Camera maintains steady medium shot, accelerating
slightly on the stairs. Wind sounds build as they reach the rooftop.
Golden hour lighting."

API params:
  reference_images: ["runner_ref.jpg", "rooftop_vista.jpg"]
  duration: 15
  aspect_ratio: "16:9"
```

#### E-commerce / Product Showcase

Product-focused advertising:

```
Prompt: "Static camera. A hamburger suspended and rotating mid-air.
Ingredients gently separate while maintaining shape — golden sesame bun,
fresh green lettuce, dewy tomato slices, two thick beef patties with
melting golden cheddar, soft bun base. All slowly descend and perfectly
reassemble into a complete double cheeseburger. Cheese continues to
melt, lettuce and tomato dewdrops glisten. Ultimate appetizing food
aesthetics. Clean white background."

API params:
  reference_images: ["hamburger_hero.jpg"]
  first_frame_url: "hamburger_hero.jpg"
  duration: 12
  resolution: "720p"
  aspect_ratio: "1:1"
```

#### Science / Educational Content

Medical or educational visualizations:

```
Prompt: "15-second health educational clip. 0–5s: Transparent blue human
upper body. Camera slowly pushes into a clear artery. Blood flows
smoothly, clean blue color. 5–10s: Sugar and fat particles enter the
bloodstream. Camera follows blood flow. Blood gradually thickens,
yellowish lipid deposits form on vessel walls. 10–15s: Vessel lumen
visibly narrows, flow speed decreases. Before/after comparison. Colors
darken. 4K medical CGI, semi-transparent visualization."

API params:
  duration: 15
  resolution: "720p"
  generate_audio: true
```

### 4.6 Style and Quality Modifiers

Append these phrases to your prompts to shape the output:

**Visual Style:**
- `Cinematic quality, film grain, shallow depth of field`
- `2.35:1 widescreen, 24fps feel`
- `Ink wash painting style` / `Anime style` / `Photorealistic`
- `High saturation neon colors, cool-warm contrast`
- `4K medical CGI, semi-transparent visualization`

**Mood / Atmosphere:**
- `Tense and suspenseful` / `Warm and healing` / `Epic and grand`
- `Comedy with exaggerated expressions`
- `Documentary tone, restrained narration`

**Audio Direction** (when `generate_audio: true`):
- `Background music: grand and majestic`
- `Sound effects: footsteps, crowd noise, car sounds`
- `Beat-synced transitions matching music rhythm`
- `Gentle ambient sounds, no music`

### 4.7 Common Mistakes to Avoid

1. **Vague references** — Don't just say "use the reference video." Specify what to reference: the camera work? the action choreography? the visual effects? the rhythm?
2. **Conflicting instructions** — Don't ask for "static camera" and "orbit shot" in the same segment.
3. **Overloading short durations** — Don't pack 5 scene changes into a 4-second video. Keep it physically plausible.
4. **Unused references** — If you pass `reference_images` with 4 URLs, your prompt should describe the role of each one.
5. **Ignoring audio** — Sound design dramatically improves output. Always include audio direction when using Seedance 2.0.
6. **Duration mismatch** — Match your prompt complexity to the selected `duration`. A 4-second video can handle one scene; a 15-second video can handle a full sequence.
7. **No camera direction** — Always specify at least one camera movement or shot size. The model responds much better to explicit camera language.

---

## 5. Model Selection Guide

| Use Case | Model | Why |
|---|---|---|
| Quick concept art | Nano Banana 2 | Fast, cheap ($0.06–0.08), good for brainstorming |
| Final ad creative | Nano Banana Pro | Best quality, accurate text rendering |
| Web-referenced design | Nano Banana 2 | `web_search` pulls current visual trends |
| Product photography | Nano Banana Pro at 4K | Highest detail for commercial use |
| Storyboard preview | Seedance 2.0 Fast | Quick video drafts at lower cost |
| Final video asset | Seedance 2.0 | Cinematic quality with synced audio |
| Character-consistent video | Seedance 2.0 | `reference_images` maintains identity across shots |
| Social media reels | Seedance 2.0 Fast, 9:16 | Fast vertical video generation |
| Music video / beat-synced | Seedance 2.0 | `reference_audios` for beat-matched editing |
| Product demo video | Seedance 2.0 | `first_frame_url` anchors product in frame |
| Educational animation | Seedance 2.0 | Native audio for narration and sound effects |

---

## 6. Workflow Patterns

### Image-to-Video Pipeline

Generate a hero image, then animate it:

```python
# Step 1: Generate hero image with Nano Banana Pro
img_response = requests.post(
    "https://api.segmind.com/v1/nano-banana-pro",
    headers=headers,
    json={
        "prompt": "Luxury perfume bottle on black marble, dramatic side lighting, gold accents",
        "aspect_ratio": "16:9",
        "output_resolution": "2K"
    }
)

# Save the image and host it (or use a data URL)
with open("hero.jpg", "wb") as f:
    f.write(img_response.content)

# Step 2: Animate it with Seedance 2.0
vid_response = requests.post(
    "https://api.segmind.com/v1/seedance-2.0",
    headers=headers,
    json={
        "prompt": "Camera slowly orbits the perfume bottle. Light refracts through the glass, "
                  "casting prismatic shadows on the marble. A hand reaches in and lifts the bottle. "
                  "Elegant string music.",
        "duration": 10,
        "resolution": "720p",
        "aspect_ratio": "16:9",
        "generate_audio": True,
        "first_frame_url": "https://your-host.com/hero.jpg"
    }
)

with open("product_ad.mp4", "wb") as f:
    f.write(vid_response.content)
```

### Iteration Workflow

Progressive refinement from draft to final:

1. **Explore** — Nano Banana 2 at 1K ($0.08 each) for 5–10 concept variations
2. **Refine** — Nano Banana Pro at 2K ($0.15) for the best 2–3 concepts
3. **Preview** — Seedance 2.0 Fast ($0.77) to test animation with the chosen image
4. **Produce** — Seedance 2.0 ($1.21) for the final production video with audio

Total cost for a full pipeline: approximately $3–5.

### Ad Campaign Batch

Generate a full creative set from one prompt by varying `aspect_ratio`:

```python
prompt = "Bold typography saying 'SUMMER SALE 50% OFF' over a tropical beach scene, vibrant colors, modern design"

formats = {
    "instagram_feed": {"aspect_ratio": "1:1", "output_resolution": "2K"},
    "instagram_story": {"aspect_ratio": "9:16", "output_resolution": "2K"},
    "youtube_thumbnail": {"aspect_ratio": "16:9", "output_resolution": "2K"},
    "pinterest_pin": {"aspect_ratio": "2:3", "output_resolution": "2K"},
    "twitter_header": {"aspect_ratio": "21:9", "output_resolution": "2K"},
}

for name, params in formats.items():
    response = requests.post(
        "https://api.segmind.com/v1/nano-banana-2",
        headers=headers,
        json={"prompt": prompt, **params}
    )
    with open(f"{name}.jpg", "wb") as f:
        f.write(response.content)
    print(f"Generated {name}.jpg")
```

### Multi-Shot Video with Frame Chaining

Generate a sequence of connected video clips using `return_last_frame`:

```python
# Shot 1: Establishing shot
shot1 = requests.post(
    "https://api.segmind.com/v1/seedance-2.0",
    headers=headers,
    json={
        "prompt": "Wide establishing shot of a futuristic city at dawn. Camera slowly tilts down from the sky.",
        "duration": 5,
        "resolution": "720p",
        "aspect_ratio": "16:9",
        "return_last_frame": True
    }
)

# Save video and extract last frame URL for chaining
with open("shot1.mp4", "wb") as f:
    f.write(shot1.content)

# Use the last frame as first_frame_url for shot 2
# (host the extracted frame or use a returned URL)
shot2 = requests.post(
    "https://api.segmind.com/v1/seedance-2.0",
    headers=headers,
    json={
        "prompt": "Camera continues tilting down, pushing into a street-level market. People walk by. Ambient city sounds.",
        "duration": 5,
        "resolution": "720p",
        "aspect_ratio": "16:9",
        "first_frame_url": "https://your-host.com/shot1_last_frame.jpg",
        "return_last_frame": True
    }
)
```

---

## 7. Budget Guidelines

| Goal | Strategy |
|---|---|
| Keep image iterations cheap | Use Nano Banana 2 at 1K ($0.08/image) — aim for < $2 per concept |
| Preview video before committing | Always use Seedance 2.0 Fast ($0.77) before Seedance 2.0 ($1.21) |
| Typical ad package (5 images + 2 videos) | ~$3–5 total |
| Full campaign (20 images + 5 videos) | ~$8–12 total |
| Bulk concept generation | Nano Banana 2 at 512px ($0.06/image) for maximum throughput |

---

## 8. Example Templates

### Product Ad (15s)

```python
requests.post(
    "https://api.segmind.com/v1/seedance-2.0",
    headers=headers,
    json={
        "prompt": (
            "15-second product showcase. "
            "0–3s: Product enters frame with dynamic rotation. Extreme close-up on "
            "surface texture and logo details. Dramatic rim lighting. "
            "4–8s: Multiple angle transitions — front, side, back — with highlight "
            "scanning light effects sweeping across the product. "
            "9–12s: Product placed in lifestyle context. A hand picks it up naturally. "
            "13–15s: Hero shot, centered. Brand tagline fades in below. "
            "Background music builds to resolution. Clean, modern feel."
        ),
        "duration": 15,
        "resolution": "720p",
        "aspect_ratio": "16:9",
        "generate_audio": True,
        "first_frame_url": "https://example.com/product_photo.jpg",
        "reference_images": ["https://example.com/product_photo.jpg"]
    }
)
```

### Short Drama (15s)

```python
requests.post(
    "https://api.segmind.com/v1/seedance-2.0",
    headers=headers,
    json={
        "prompt": (
            "Tense dramatic scene, 15 seconds. "
            "0–5s: Close-up on Character A's face — reddened eyes, finger pointing "
            "accusingly. Tears streaming down. Dialogue: 'What exactly are you trying "
            "to take from me?' Voice choked with rage. "
            "6–10s: Character B trembles, holding up a document. Steps forward. "
            "Camera sweeps past background details. Dialogue: 'I'm not deceiving you! "
            "This is what he entrusted to me!' "
            "11–15s: The document is revealed. Character A freezes — expression shifts "
            "from anger to shock. Hands slowly rise. "
            "Audio: Urgent piano, subtle static interference, ending with muffled voice."
        ),
        "duration": 15,
        "resolution": "720p",
        "generate_audio": True,
        "reference_images": [
            "https://example.com/character_a.jpg",
            "https://example.com/character_b.jpg"
        ]
    }
)
```

### Dance Video (13s)

```python
requests.post(
    "https://api.segmind.com/v1/seedance-2.0",
    headers=headers,
    json={
        "prompt": (
            "The character from the reference image performs the dance moves "
            "and matches the beat of the reference audio. 13-second video. "
            "Movements should be smooth with no stuttering. Energetic and dynamic. "
            "Studio lighting with colored spotlights."
        ),
        "duration": 13,
        "resolution": "720p",
        "aspect_ratio": "9:16",
        "generate_audio": True,
        "reference_images": ["https://example.com/dancer.jpg"],
        "reference_audios": ["https://example.com/dance_track.mp3"]
    }
)
```

### Scenery Montage with Music (15s)

```python
requests.post(
    "https://api.segmind.com/v1/seedance-2.0",
    headers=headers,
    json={
        "prompt": (
            "Landscape montage with beat-synced transitions matching the reference audio. "
            "Cycle through the reference images as scenes — mountains, ocean, forest, desert, "
            "city skyline, night sky. Each scene lasts 2–3 seconds with smooth cross-dissolves. "
            "Camera slowly pushes in on each scene. Cinematic color grading, warm tones."
        ),
        "duration": 15,
        "resolution": "720p",
        "aspect_ratio": "21:9",
        "generate_audio": True,
        "reference_images": [
            "https://example.com/mountains.jpg",
            "https://example.com/ocean.jpg",
            "https://example.com/forest.jpg",
            "https://example.com/desert.jpg",
            "https://example.com/skyline.jpg",
            "https://example.com/night_sky.jpg"
        ],
        "reference_audios": ["https://example.com/ambient_music.mp3"]
    }
)
```

### Social Media Reel — E-commerce (10s, Vertical)

```python
requests.post(
    "https://api.segmind.com/v1/seedance-2.0-fast",
    headers=headers,
    json={
        "prompt": (
            "10-second vertical product reel. "
            "0–3s: Product slides into frame from the right. Quick zoom to close-up. "
            "4–7s: Product rotates, camera orbits. Text overlay area left at top. "
            "8–10s: Product settles center frame. Subtle sparkle effect. "
            "Upbeat, trendy background music. Clean white background."
        ),
        "duration": 10,
        "resolution": "720p",
        "aspect_ratio": "9:16",
        "first_frame_url": "https://example.com/product.jpg",
        "reference_images": ["https://example.com/product.jpg"]
    }
)
```

---

## Reference Limits

| Input Type | Limit | Max Size |
|---|---|---|
| Reference images | Up to 9 | 30 MB each |
| Reference videos | Up to 3 | 50 MB each, 2–15s duration |
| Reference audios | Up to 3 | 15 MB each, ≤ 15s duration |
| **Total reference files** | **≤ 12 combined** | — |

---
> Source: [segmind/segmind-media-skill](https://github.com/segmind/segmind-media-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
