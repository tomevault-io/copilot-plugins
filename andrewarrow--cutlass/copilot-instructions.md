## cutlass

> **There are extensive validation checks in `fcpxml_lib/validation/`. Never bypass these. Better to let an error stop generation because validation failed than to ever produce invalid FCPXML.**

# Project Context for AI Assistance - Python FCPXML Library

## 🚨 CRITICAL: use ../FCPXMLv1_13.dtd for DTD validation 🚨 ##

## 🚨 CRITICAL: ALL CODE MUST USE VALIDATION 🚨
**There are extensive validation checks in `fcpxml_lib/validation/`. Never bypass these. Better to let an error stop generation because validation failed than to ever produce invalid FCPXML.**

## 🚨 CRITICAL: CHANGE CODE NOT XML 🚨
**NEVER EVER only change problem XML in an XML file, always change the code that generates it too**

## 🚨 CRITICAL: NO XML STRING TEMPLATES 🚨
**NEVER EVER generate XML from hardcoded string templates with f-strings or % formatting, use dataclasses**

❌ BAD: `xml = f"<video ref=\"{video_ref}\">{content}</video>"`
❌ BAD: `"<asset-clip ref=\"%s\" name=\"%s\"/>" % (ref, name)`
✅ GOOD: `Video(ref=video_ref, duration=duration)` → `serialize_to_xml()`

**All FCPXML generation MUST use the dataclasses in `fcpxml_lib/models/elements.py`.**

## 🚨 CRITICAL: Conform-Rate Elements Must Include srcFrameRate 🚨

**The #1 cause of FCP import warnings: Missing srcFrameRate attribute on conform-rate elements**

### ✅ CORRECT conform-rate Structure:
```xml
<clip offset="0s" name="VideoClip" duration="240240/24000s" format="r2" tcFormat="NDF">
    <conform-rate scaleEnabled="0" srcFrameRate="24"/>
    <adjust-transform><!-- keyframe animations --></adjust-transform>
    <video ref="r3" offset="0s" duration="240240/24000s"/>
</clip>
```

### ❌ INCORRECT conform-rate (causes FCP warnings):
```xml
<clip offset="0s" name="VideoClip" duration="240240/24000s" format="r2" tcFormat="NDF">
    <conform-rate scaleEnabled="0"/>  <!-- Missing srcFrameRate! -->
    <adjust-transform><!-- keyframe animations --></adjust-transform>
    <video ref="r3" offset="0s" duration="240240/24000s"/>
</clip>
```

### 🚨 FCP Import Error Without srcFrameRate:
```
Encountered an unexpected value. (conform-rate: /fcpxml[1]/library[1]/event[1]/project[1]/sequence[1]/spine[1]/clip[1])
```

**CRITICAL: Always include srcFrameRate attribute matching media frame rate (24, 25, 29.97, 30, etc.)**

## 🚨 CRITICAL: Multi-Lane Video Audio Implementation 🚨

**The #1 cause of silent videos: Missing audio elements and asset properties**

### ✅ CORRECT Audio Implementation for Complex Clips:

For multi-lane video effects with audio, you need **BOTH** asset-level audio properties AND timeline audio elements:

```xml
<!-- 1. Assets MUST have audio properties -->
<asset id="r2" name="video" hasVideo="1" hasAudio="1" audioSources="1" audioChannels="2" audioRate="48000">
    <media-rep kind="original-media" src="file:///path/to/video.mov"/>
</asset>

<!-- 2. Timeline MUST have both video AND audio elements -->
<clip offset="0s" name="Video Clip" duration="240240/24000s" format="r3" tcFormat="NDF">
    <conform-rate scaleEnabled="0" srcFrameRate="24"/>
    <adjust-transform><!-- keyframe animations --></adjust-transform>
    <video ref="r2" offset="0s" duration="240240/24000s"/>
    <audio ref="r2" offset="0s" duration="240240/24000s" role="dialogue"/>
</clip>
```

### ❌ INCORRECT Audio Implementation (silent in FCP):

```xml
<!-- Missing audio properties on asset -->
<asset id="r2" name="video" hasVideo="1" videoSources="1">  <!-- NO hasAudio! -->
    <media-rep kind="original-media" src="file:///path/to/video.mov"/>
</asset>

<!-- Missing audio element on timeline -->
<clip offset="0s" name="Video Clip" duration="240240/24000s" format="r3" tcFormat="NDF">
    <conform-rate scaleEnabled="0" srcFrameRate="24"/>
    <video ref="r2" offset="0s" duration="240240/24000s"/>  <!-- Only video, no audio! -->
</clip>
```

### 🚨 CRITICAL Audio Implementation Rules:

1. **Asset Audio Properties Required**: Assets MUST have `hasAudio="1"`, `audioSources="1"`, `audioChannels="2"`, `audioRate="48000"`
2. **Timeline Audio Elements Required**: Complex clips need separate `<audio ref="...">` elements alongside `<video ref="...">` elements
3. **DTD Compliance**: Use `role="dialogue"` on `<audio>` elements, NOT `audioRole` on `<clip>` elements
4. **Both Required**: You need BOTH asset properties AND timeline elements for audio to work

### ✅ Implementation Pattern:

```python
# 1. Create assets with audio properties when include_audio=True
asset, format_obj = create_media_asset(
    video_path, asset_id, format_id, include_audio=True
)

# 2. Add both video and audio elements to timeline
clip_elements = [
    {"type": "video", "ref": asset_id, "duration": duration},
    {"type": "audio", "ref": asset_id, "duration": duration, "role": "dialogue"}
]
```

### 🚨 Common Audio Failures:

- **Asset without audio properties**: `hasAudio` missing → silent video
- **Timeline without audio elements**: Only `<video>` elements → silent video  
- **Wrong DTD attributes**: `audioRole` on clips → validation failure
- **Missing role attribute**: Audio elements without `role="dialogue"` → routing issues

## 🚨 CRITICAL: Images vs Videos Architecture 🚨

**The #2 cause of crashes: Using wrong element types for images vs videos**

### ✅ IMAGES (PNG/JPG files):
```xml
<!-- Asset: duration="0s" (timeless) -->
<asset id="r2" duration="0s" hasVideo="1" format="r3" videoSources="1"/>
<!-- Format: NO frameDuration (timeless) -->
<format id="r3" name="FFVideoFormatRateUndefined" width="1280" height="720" colorSpace="1-13-1"/>
<!-- Spine: Video element (NOT AssetClip) -->
<video ref="r2" duration="240240/24000s">
    <adjust-transform><!-- Simple animations work --></adjust-transform>
</video>
```

### ✅ VIDEOS (MOV/MP4 files):
```xml
<!-- Asset: has duration, audio properties -->
<asset id="r2" duration="14122857/100000s" hasVideo="1" hasAudio="1" audioSources="1"/>
<!-- Format: has frameDuration -->
<format id="r3" frameDuration="13335/400000s" width="1920" height="1080"/>
<!-- Spine: AssetClip element -->
<asset-clip ref="r2" duration="373400/3000s">
    <adjust-transform><!-- Complex animations work --></adjust-transform>
    <filter-video><!-- Effects work --></filter-video>
</asset-clip>
```

### ❌ CRASH PATTERNS:
1. **AssetClip for images** → `addAssetClip:toObject:parentFormatID` crash
2. **frameDuration on image formats** → `performAudioPreflightCheckForObject` crash  
3. **Complex effects on images** → Various import crashes

## 🚨 CRITICAL: Title Elements and Resource References 🚨

**The #3 cause of DTD validation failures: Incorrect title structure and invalid resource references**

### ✅ CORRECT Title Structure:
```xml
<!-- Resources: Title effect (NOT title asset) -->
<effect id="r4" name="Text" uid=".../Titles.localized/Basic Text.localized/Text.localized/Text.moti"/>

<!-- Spine: Video element references ASSET (NOT format) -->
<video ref="r2" duration="240240/24000s">  <!-- r2 = asset ID -->
    <title ref="r4" lane="1" duration="120120/24000s" offset="0s" start="0s" name="My Text">
        <text>
            <text-style ref="ts1">Hello World</text-style>
        </text>
        <text-style-def id="ts1">
            <text-style font="Helvetica" fontSize="290" fontColor="1 1 1 1"/>
        </text-style-def>
    </title>
</video>
```

### ❌ INVALID Title Patterns:
1. **Title assets in resources** → DTD validation failure (assets need media-rep)
   ```xml
   ❌ <asset id="r2" name="Title: Hello" duration="300s"/> <!-- NO media-rep = DTD error -->
   ```

2. **Video refs pointing to formats** → "Resource format element invalid" error
   ```xml
   ❌ <video ref="r1" ...>  <!-- r1 = format ID, should be asset ID -->
   ```

3. **Standalone title elements on spine** → Missing parent structure
   ```xml
   ❌ <spine>
   ❌     <title ref="r4" ...>  <!-- Titles must be NESTED in video/asset-clip -->
   ```

### ✅ CRITICAL Rules for Titles:
1. **Titles are NESTED elements** within video/asset-clip elements, not standalone spine elements
2. **Title effects go in resources** (not title assets)
3. **Video elements MUST reference assets** (with media-rep), never formats
4. **Text-style IDs are locally scoped** within each title, not global resources
5. **Always use real media files** from ../assets/ as backgrounds

### ✅ Title Implementation Pattern:
```python
# 1. Create background asset from real media file
background_asset, background_format = create_media_asset(
    str(media_file), asset_id, format_id
)

# 2. Create title effect in resources
title_effect = {
    "id": effect_id,
    "name": "Text", 
    "uid": ".../Titles.localized/Basic Text.localized/Text.localized/Text.moti"
}

# 3. Create video element referencing the ASSET
video_element = {
    "type": "video",
    "ref": asset_id,  # ✅ References asset, not format
    "nested_elements": [title_elements]  # ✅ Titles nested inside
}
```

### 🚨 DTD Validation Checklist:
- [ ] All video/@ref point to asset IDs (not format IDs)
- [ ] All assets have media-rep elements with file:// URLs
- [ ] Titles are nested within video/asset-clip elements
- [ ] Text-style definitions exist for all text-style references
- [ ] No title assets in resources (only title effects)

## 🚨 CRITICAL: Python Dataclass Usage 🚨

**Use dataclasses from `fcpxml_lib/models/elements.py`:**

```python
from fcpxml_lib.models.elements import Video, AssetClip, Asset, Format

# ✅ Images use Video elements
video_element = Video(
    ref="r2",
    duration="240240/24000s",
    start="0s"  # Required for images
)

# ✅ Videos use AssetClip elements  
asset_clip = AssetClip(
    ref="r2", 
    duration="373400/3000s"
    # NO start attribute for videos
)
```

## MANDATORY: Testing and Validation

### Required Tests (ALWAYS run):
1. **Python Tests**: `python -m pytest tests/` - MUST pass
2. **XML Validation**: `xmllint output.fcpxml --noout` - MUST pass  
3. **FCP Import Test**: Import into actual Final Cut Pro

### Study tests/* Package Patterns
**Before writing FCPXML code, review the logic in `tests/` files:**
- `tests/test_crash_prevention.py` - Shows critical crash prevention patterns
- `tests/test_timeline_elements.py` - Shows correct element type usage
- `tests/test_media_detection.py` - Shows proper media property detection
- These tests contain proven patterns that prevent crashes

### Common Error Patterns to Check:
1. **ID collisions** - Use proper ID generation from `fcpxml_lib/utils/ids.py`
2. **Missing resources** - Every `ref=` needs matching `id=`
3. **Wrong element types** - Images use Video, videos use AssetClip
4. **Missing smart collections** - All 5 required collections must be present

## 🏗️ Required Architecture Pattern

**ALWAYS follow this pattern (from `fcpxml_lib/core/fcpxml.py`):**

```python
from fcpxml_lib.core.fcpxml import create_empty_project, add_media_to_timeline, save_fcpxml

def generate_my_feature(input_files: list, output_file: str) -> bool:
    # 1. Create base FCPXML structure
    fcpxml = create_empty_project()
    
    # 2. Add media using validated functions
    success = add_media_to_timeline(
        fcpxml, 
        input_files, 
        clip_duration_seconds=3.0
    )
    if not success:
        raise RuntimeError("Failed to add media to timeline")
    
    # 3. Apply modifications to timeline elements
    # Access via: fcpxml.library.events[0].projects[0].sequences[0].spine
    
    # 4. Save with validation
    return save_fcpxml(fcpxml, output_file)
```

## Frame Boundary Alignment

**All durations MUST use `fcpxml_lib/utils/timing.py`:**
```python
from fcpxml_lib.utils.timing import convert_seconds_to_fcp_duration

# ✅ Proper frame alignment
duration = convert_seconds_to_fcp_duration(3.5)  # "84084/24000s"

# ❌ Manual calculations cause "not on edit frame boundary" errors
duration = "3.5s"  # Wrong!
```

## Resource ID Management

**Use proper ID generation from `fcpxml_lib/utils/ids.py`:**
```python
from fcpxml_lib.utils.ids import generate_resource_id

# ✅ Thread-safe ID generation
asset_id = generate_resource_id()  # "r1", "r2", etc.

# ❌ Manual ID generation causes collisions
asset_id = f"r{count + 1}"  # Race conditions!
```

## 🚨 CRITICAL: Media Property Detection 🚨

**ALWAYS use `fcpxml_lib/core/fcpxml.py` detection functions:**

```python
from fcpxml_lib.core.fcpxml import detect_video_properties

# ✅ Automatic property detection
props = detect_video_properties("video.mp4")
# Returns: width, height, duration_seconds, has_audio, frame_rate

# ❌ Hardcoded properties cause import failures  
width = 1920  # Video might be 1080×1920 portrait!
has_audio = True  # Video might not have audio!
```

## 🚨 CRITICAL: Media Asset Creation 🚨

**Use `fcpxml_lib/core/fcpxml.py` functions for proper media handling:**

```python
from fcpxml_lib.core.fcpxml import create_media_asset

# ✅ Automatic media type detection and proper asset creation
asset, format_obj = create_media_asset(
    file_path="video.mp4",
    asset_id="r2", 
    format_id="r3"
)
# Handles: absolute paths, media type detection, audio properties

# ❌ Manual asset creation bypasses validation
asset = Asset(id="r2", src="./video.mp4")  # Relative path fails!
```

## 🚨 CRITICAL: Smart Collections Requirement 🚨

**All FCPXML must include 5 required smart collections (see `fcpxml_lib/constants.py`):**

```python
# ✅ Automatically included by create_empty_project()
fcpxml = create_empty_project()
# Includes: Projects, All Video, Audio Only, Stills, Favorites

# ❌ Missing smart collections cause FCP crashes
# Never create FCPXML without all 5 collections
```

## 🚨 CRITICAL: Timeline Element Rules 🚨

**Images vs Videos have different timeline element requirements:**

```python
from fcpxml_lib.models.elements import Video, AssetClip

# ✅ Images: Use Video elements with start attribute
if file_path.lower().endswith(('.jpg', '.png')):
    element = Video(
        ref=asset_id,
        duration=duration,
        start="0s"  # Required for images
    )

# ✅ Videos: Use AssetClip elements without start attribute  
elif file_path.lower().endswith(('.mp4', '.mov')):
    element = AssetClip(
        ref=asset_id,
        duration=duration
        # NO start attribute for videos
    )
```

## 🚨 CRITICAL: Multi-Lane Structure Patterns 🚨

**There are two distinct FCPXML patterns for creating multiple lanes, each serving different purposes:**

### ✅ Pattern A: Nested Elements (Recommended for Multi-Lane Visibility)
**Use for creating multiple visible lanes in Final Cut Pro timeline:**

```xml
<!-- Background AssetClip with nested elements -->
<asset-clip ref="r2" duration="240240/24000s" offset="0s">
    <adjust-transform scale="3.27127 3.27127"/>
    <!-- PNGs nested INSIDE the background AssetClip -->
    <video ref="r4" lane="1" offset="0s" start="3600s" duration="96096/24000s">
        <adjust-transform position="62.5 0"/>
    </video>
    <video ref="r7" lane="2" offset="0s" start="3600s" duration="96096/24000s">
        <adjust-transform position="-62.5 0"/>
    </video>
</asset-clip>
```

**Pattern A Implementation:**
```python
# Create background AssetClip
bg_element = {
    "type": "asset-clip",
    "ref": asset_id,
    "duration": duration,
    "nested_elements": []  # Key: nested_elements array
}

# Add PNGs as nested elements INSIDE background
for i, png_file in enumerate(png_files):
    png_element = {
        "type": "video",
        "ref": png_asset_id,
        "lane": i + 1,  # Sequential lanes: 1, 2, 3...
        "start": "3600s",  # Timing like Info.fcpxml
        "duration": duration
    }
    bg_element["nested_elements"].append(png_element)

# Add background to spine (single element)
sequence.spine.ordered_elements.append(bg_element)
```

### ✅ Pattern B: Separate Spine Elements (Fallback Pattern)
**Use when no background video or for separate timeline elements:**

```xml
<spine>
    <!-- Each element is separate on the spine -->
    <video ref="r4" lane="1" offset="0s" start="0s" duration="240240/24000s">
        <adjust-transform position="30 20"/>
    </video>
    <video ref="r7" lane="2" offset="240240/24000s" start="0s" duration="240240/24000s">
        <adjust-transform position="-30 -20"/>
    </video>
</spine>
```

**Pattern B Implementation:**
```python
# Add each PNG as separate spine element
for i, png_file in enumerate(png_files):
    png_element = {
        "type": "video", 
        "ref": png_asset_id,
        "lane": i + 1,
        "start": "0s",  # Different timing
        "duration": duration
    }
    # Each element added separately to spine
    sequence.spine.videos.append(png_element)
    sequence.spine.ordered_elements.append(png_element)
```

### 🎯 When to Use Each Pattern:

**Use Pattern A (Nested) when:**
- Creating multi-lane content with background video
- Want multiple elements visible simultaneously in FCP
- Following Go implementation patterns (like `png-pile` command)
- Need elements to move together as a group

**Use Pattern B (Separate) when:**
- No background video available
- Creating sequential timeline elements
- Each element should be independently positioned in timeline
- Fallback when Pattern A isn't applicable

### 🚨 Key Differences:

| Aspect | Pattern A (Nested) | Pattern B (Separate) |
|--------|-------------------|---------------------|
| **FCP Appearance** | Multiple visible lanes | Separate timeline elements |
| **Structure** | Nested inside AssetClip | Independent spine elements |
| **Timing** | `start="3600s"` (Info.fcpxml style) | `start="0s"` |
| **Use Case** | Multi-lane with background | Sequential or no background |
| **Go Equivalent** | `png-pile` command | Basic timeline |

**Critical Discovery:** The regression was caused by using Pattern B when Pattern A was needed for multi-lane visibility.

## Validation Integration

**All dataclasses have built-in validation (see `fcpxml_lib/validation/`):**

```python
from fcpxml_lib.validation.validators import validate_frame_alignment

# ✅ Validation is automatic in dataclass __post_init__
asset = Asset(id="r2", duration="invalid")  # Raises ValueError

# ✅ Manual validation for complex cases
is_valid = validate_frame_alignment("84084/24000s")  # True
```

## XML Generation

**Use structured serialization from `fcpxml_lib/serialization/`:**

```python
from fcpxml_lib.serialization.xml_serializer import serialize_to_xml

# ✅ Structured XML generation
xml_content = serialize_to_xml(fcpxml)

# ❌ Never manipulate XML strings directly
xml_content = xml_content.replace("<video", "<asset-clip")  # Wrong!
```

## Testing Integration

**Run tests to verify crash prevention patterns:**

```python
# Required test commands:
python -m pytest tests/test_crash_prevention.py -v
python -m pytest tests/test_timeline_elements.py -v  
python -m pytest tests/test_xml_structure.py -v
python -m pytest tests/test_video_at_edge.py -v  # Pattern A vs B validation
python -m pytest tests/test_conform_rate_validation.py -v  # FCP import warnings prevention

# XML validation:
xmllint output.fcpxml --noout
```

## 🚨 CRITICAL: File Path Requirements 🚨

**Final Cut Pro requires absolute file paths:**

```python
import os

# ✅ Always convert to absolute paths
abs_path = os.path.abspath(file_path)
media_rep = MediaRep(src=f"file://{abs_path}")

# ❌ Relative paths cause "missing media" errors
media_rep = MediaRep(src="file://./video.mp4")  # Fails!
```

## Effect and Animation Safety

**Prefer built-in transforms over effects:**

```python
from fcpxml_lib.models.elements import AdjustTransform

# ✅ Built-in transforms are crash-safe
transform = AdjustTransform(
    scale="2.0 2.0",
    rotation="15.0"
)

# ❌ Avoid fictional effect UIDs - causes crashes
# Only use verified UIDs from samples/
```

## 🚨 CRITICAL: File Size and Organization Rules 🚨

**Keep code well-organized and maintainable:**

### File Size Limits:
- **`main.py`**: Keep minimal - CLI argument parsing and command dispatching ONLY
- **`fcpxml_lib/` files**: Maximum 600 lines per file
- **No file exceptions**: If a file exceeds 600 lines, split into multiple modules

### Code Organization Rules:
- **CLI Layer (`main.py`)**: Argument parsing, command routing, basic validation only
- **Core Logic**: All business logic must be in `fcpxml_lib/` packages  
- **Package Structure**: Use appropriate subpackages (`core/`, `models/`, `utils/`, etc.)
- **Single Responsibility**: Each module should have one clear purpose

### When to Split Files:
```python
# 🚨 File getting too long? Split by:
# 1. Related functionality into separate modules
# 2. Different data models into separate files  
# 3. Different command implementations into separate modules

# ✅ Example split:
fcpxml_lib/
├── commands/
│   ├── pile_commands.py      # png-pile, video-pile logic
│   ├── edge_commands.py      # edge, video-at-edge logic  
│   └── timeline_commands.py  # basic timeline operations
├── core/
│   └── fcpxml.py            # Core FCPXML operations
└── models/
    └── elements.py          # Dataclass definitions
```

### Refactoring Guidelines:
- **Move command logic** from `main.py` to `fcpxml_lib/cmd/`
- **Extract reusable functions** to appropriate utility modules
- **Keep imports clean** - no circular dependencies
- **Maintain backwards compatibility** when moving functions

## 🚨 CRITICAL: Command Package Organization 🚨

**CLI commands are organized for scalability with hundreds of commands:**

### Command Package Structure:
```
fcpxml_lib/cmd/
├── __init__.py                  # Package exports all commands
├── create_empty_project.py      # create-empty-project command
├── create_random_video.py       # create-random-video command
├── video_at_edge.py            # video-at-edge command
├── stress_test.py              # stress-test command
└── [future_command].py         # New commands go here
```

### Command Implementation Rules:
- **One command per file**: Each CLI command gets its own module
- **File naming**: Use underscores (create_empty_project.py)
- **Function naming**: End with `_cmd` (create_empty_project_cmd)
- **CLI argument names**: Use hyphens (create-empty-project)

### Required Function Signature:
```python
def command_name_cmd(args):
    """Command implementation that takes parsed arguments"""
    # Handle CLI concerns: validation, output, error messages
    # Delegate business logic to other fcpxml_lib modules
```

### Command File Template:
```python
"""
Command Name Implementation

Brief description of what this command does.
"""

import sys
from pathlib import Path

# Import required fcpxml_lib modules
from fcpxml_lib import create_empty_project, save_fcpxml
from fcpxml_lib.generators.timeline_generators import some_generator

def command_name_cmd(args):
    """CLI implementation for command-name"""
    # Argument validation and processing
    # Business logic delegation
    # Output and error handling
```

### Adding New Commands:
1. **Create new file**: `fcpxml_lib/cmd/new_command.py`
2. **Implement function**: `new_command_cmd(args)`
3. **Update `__init__.py`**: Add import and export
4. **Update `main.py`**: Add argument parser and dispatcher
5. **Keep main.py minimal**: Only argument parsing and command routing

### Scalability Benefits:
- **Easy to add commands**: Just create new file and update imports
- **Parallel development**: Multiple developers can work on different commands
- **Clear separation**: CLI logic separated from business logic
- **Maintainable**: Each command is self-contained and testable

---

**Key Principle: Use `fcpxml_lib/` functions and validation. If FCPXML generation requires more than 1 iteration to work, you're using the wrong approach.**

**Critical File References:**
- **Core Functions**: `fcpxml_lib/core/fcpxml.py`
- **Data Models**: `fcpxml_lib/models/elements.py` 
- **XML Serialization**: `fcpxml_lib/serialization/xml_serializer.py` (conform-rate fix)
- **Validation**: `fcpxml_lib/validation/validators.py`
- **Testing Patterns**: `tests/test_crash_prevention.py`
- **Multi-Lane Testing**: `tests/test_video_at_edge.py`
- **Multi-Lane Audio Testing**: `tests/test_multilane_audio.py`
- **FCP Import Testing**: `tests/test_conform_rate_validation.py`
- **Info.fcpxml Recreation**: `tests/test_info_recreation.py`
- **Timeline Generators**: `fcpxml_lib/generators/timeline_generators.py`
- **Command Implementations**: `fcpxml_lib/cmd/`
- **Utilities**: `fcpxml_lib/utils/timing.py`, `fcpxml_lib/utils/ids.py`

---
> Source: [andrewarrow/cutlass](https://github.com/andrewarrow/cutlass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
