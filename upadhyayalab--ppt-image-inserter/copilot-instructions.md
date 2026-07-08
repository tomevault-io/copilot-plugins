## ppt-image-inserter

> This is a Python toolkit for programmatically inserting images into PowerPoint presentations at scale. The main use case is batch-generating presentation slides from large collections of images (e.g., scientific plots, data visualizations) using a template-based approach.

# Codex Instructions for PPT Batch Image Inserter

## Project Overview

This is a Python toolkit for programmatically inserting images into PowerPoint presentations at scale. The main use case is batch-generating presentation slides from large collections of images (e.g., scientific plots, data visualizations) using a template-based approach.

**Core value proposition**: Instead of manually inserting hundreds of images into PowerPoint, users define a YAML config and run a script.

## Platform & Environment

- **Platform**: Cross-platform (Windows, macOS, Linux)
  - **Note**: Primarily tested on Windows. Most users are on Windows.
  - Avoid Unix-specific commands in user-facing code
  - **Use forward slashes in paths** (works on all platforms including Windows)
- **Python**: 3.9+ (recommended)
- **Main dependencies**: `python-pptx`, `PyYAML`
- **Package manager**: conda (recommended) or pip

## Setup & Running

### Installation

```bash
# Clone and navigate
git clone https://github.com/UpadhyayaLab/ppt-image-inserter.git
cd ppt-image-inserter

# Create environment
conda create -n ppt_inserter python=3.9
conda activate ppt_inserter

# Install dependencies
pip install -r requirements.txt
```

### Running the Batch Script

```bash
python examples_and_configs/batch_insert_images.py examples_and_configs/configs/example_config.yaml
```

**CRITICAL**: The file must be closed in PowerPoint before running the script. The script needs exclusive file access to modify the presentation.

## Architecture

### Package Structure: `ppt_image_inserter/`

The package is organized into modular components. All functions can be imported directly from the package:

```python
from ppt_image_inserter import insert_image, copy_slide_replace_image
```

**Core Principles:**
- **General-purpose**: No experiment-specific logic
- **Well-documented**: Clear docstrings for all public functions
- **Backwards-compatible**: Don't break existing APIs without good reason
- **Standalone**: Minimal external dependencies (just python-pptx, PyYAML)

**Module Organization:**

- **`core.py`** - Basic image insertion functions
  - `insert_image()` - Insert image at exact position on existing slide
  - `insert_image_preserve_aspect()` - Insert image with aspect ratio preserved
  - `list_slides()` - Get slide information

- **`position.py`** - Position and unit conversion utilities
  - `get_image_position()` - Extract position from template image (auto-detection)
  - `cm_to_inches()` - Unit conversion utility

- **`slide_utils.py`** - Slide manipulation utilities
  - `duplicate_slide()` - Copy a slide
  - `remove_pictures_from_slide()` - Remove all pictures from slide
  - `remove_all_text_from_slide()` - Remove all text elements
  - `delete_slide()` - Delete slide with automatic backup

- **`workflow.py`** - High-level workflow functions
  - `copy_slide_replace_image()` - Main workflow for batch processing (duplicate template, replace image)
  - `replace_image_on_existing_slide()` - Update existing slide with new image

- **`backup.py`** - Backup system
  - `backup_presentation()` - Create timestamped backups with multiple time intervals

- **`metadata.py`** - Metadata extraction
  - `extract_image_metadata()` - Extract image source info from all slides

### User-Facing Components

1. **Config files (YAML)**: Define presentation path, template slide, image list
2. **Scripts**: User-written Python scripts that use the core module
3. **Examples**: Template configs and scripts in `examples_and_configs/`

**Config location rule**: Reusable YAML configs should live inside this
repo under `examples_and_configs/configs/`, grouped by project/domain
(for example, `examples_and_configs/configs/live/nuc_rotation/`). Do not
leave generated or reusable configs beside output `.pptx` files in
`K:/FF/PPT/PPT_autogeneration/...`; those output folders should contain
decks, backups, and other generated presentation artifacts only.

### YAML Configuration Structure

Since you'll frequently help users create and modify configs, here's the structure:

**Minimal working config:**
```yaml
presentation: "path/to/presentation.pptx"
template_slide: 1                    # 0-based index (slide 2 in PowerPoint UI)
preserve_slides: [0, 1]              # Keep title slide (0) and template slide (1)
base_dir: "path/to/images"
images:
  - image1.png                       # Single image = one slide
  - [image2.png, image3.png]         # Two images = one slide (multi-image feature)
  - image4.png                       # Single image = one slide
```

**All available fields:**
```yaml
# Required fields
presentation: "presentations/my_presentation.pptx"
template_slide: 1
base_dir: "data/images"
images:
  - plot1.png
  - plot2.png

# Optional fields
preserve_slides: [0, 1]              # Default: [0, template_slide]
backup_dir: "custom/backup/location" # Default: backups/ folder next to the presentation file
output_path: "output/new.pptx"       # Template preservation mode (see below)
auto_position: true                   # Default: true (auto-detect from template image)

# Manual position override (only used if auto_position is false)
position:
  left: 0.5      # inches from left edge
  top: 1.0       # inches from top edge
  width: 8.0     # image width in inches
  height: 6.0    # image height in inches
```

**Template Preservation Mode:**

Use `output_path` to create a new presentation instead of modifying the original:

```yaml
presentation: "templates/template.pptx"  # Source template (unchanged)
output_path: "output/my_results.pptx"    # New file to create
# ... rest of config
```

This copies the template to `output_path` before processing, leaving the original untouched. Useful for:
- Reusable templates
- Generating multiple variants from one template
- Preserving original presentation files

**Naming convention**: Template files are named `*_template.pptx`; output files drop the `_template` suffix (e.g., `my_presentation_template.pptx` → `my_presentation.pptx`). In standalone scripts, derive the output path automatically:
```python
OUTPUT_PATH = PPT_PATH.replace("_template.pptx", ".pptx")
```

## Important Constraints

### PowerPoint File Handling

**CRITICAL**: This tool is for IMAGE INSERTION ONLY. You are NOT allowed to:
- ❌ Edit existing slide content (text, shapes, formatting)
- ❌ Modify existing elements beyond images
- ❌ Rearrange slides
- ❌ Change slide layouts or templates (beyond copying them)
- ❌ Modify any other aspects of the presentation

**You ARE allowed to:**
- ✅ Create new slides by copying templates
- ✅ Insert/replace images in slides
- ✅ Delete slides (with backup)
- ✅ Store metadata in slide notes
- ✅ Add text labels (image filenames)

**Rationale**: Users manage their PowerPoint files manually. This tool just handles the tedious batch image insertion. Overstepping these boundaries creates confusion and potential data loss.

### Backup System

- **ALWAYS** create backups before destructive operations (slide deletion, image replacement)
- Backups are stored in `.backups/` with timestamps
- Multiple backup tiers: latest, 5min, 10min, 30min, hourly, daily
- Users trust the backup system - don't skip it to "save time"

### Error Handling

- **Graceful degradation**: If one image fails, continue processing others
- **Clear error messages**: Tell users WHAT failed and WHY
- **Path validation**: Check that image files exist before processing
- **Non-zero exit codes**: Scripts should exit with error code if any failures occur

## Development Guidelines

### Code Style

- **PEP 8 compliance**: Follow Python style guidelines
- **Type hints**: Use type hints for function signatures (Python 3.7+ compatible)
- **Docstrings**: Google-style docstrings for all public functions
- **Clear variable names**: `template_slide_index` not `tsi`

### Testing Philosophy

- **Real-world testing**: Test with actual PowerPoint files, not mocks
- **Cross-platform**: Consider Windows path handling (backslashes)
- **Large batches**: Test with 100+ images to catch performance issues
- **Edge cases**: Empty presentations, slides without images, corrupt files

### Documentation

- **Keep README.md updated**: Any new feature should be documented
- **Example-driven**: Show concrete examples, not just API docs
- **Beginner-friendly**: Assume users know Python basics but not python-pptx

### When Adding Features

- **Start with user story**: Why do they need this? What problem does it solve?
- **Maintain simplicity**: Does it fit the core mission (image insertion)?
- **Backwards compatible**: Don't break existing configs or APIs
- **Update docs**: Add examples to README and update AGENTS.md

## Anti-Patterns to Avoid

- **Don't over-engineer**: Keep functions simple and direct. This is a utility library, not a framework.
- **Don't break existing configs**: Maintain backwards compatibility. Support both old and new field names if changing config structure.
- **Don't assume directory structure**: Take paths as arguments or use relative paths from known locations.
- **Don't silently fail**: Always log errors with clear messages. Exit with non-zero code on failures.

## When Users Ask For Help

### Debugging Checklist

1. **Check the file is closed in PowerPoint**: The file must not be open in PowerPoint
2. **Check paths**: Are all file paths correct? (absolute vs relative)
3. **Check indices**: Slide indices are 0-based (slide 1 = index 0)
4. **Check template**: Does template slide have exactly one image?
5. **Check permissions**: Can we write to the PowerPoint file?
6. **Check PowerPoint version**: Is it .pptx (not .ppt)?

### Common Questions & Answers

**Q: "Script fails with permission error or file access error"**
- A: The file must be **closed in PowerPoint** before running the script. PowerPoint locks files when they're open.

**Q: "Images aren't in the right position"**
- A: Check that `auto_position: true` and template slide has one image where you want new images

**Q: "Script is slow with 200 images"**
- A: This is normal - PowerPoint files are complex XML. Consider batching or be patient.

**Q: "Can I modify text in slides?"**
- A: No - outside scope of this tool. Use python-pptx directly for that.

**Q: "Does it work with .ppt files?"**
- A: No - only .pptx (Office Open XML format). Convert old files first.

**Q: "How do I preserve my original template?"**
- A: Use the `output_path` config option to create a new file instead of modifying the original.

**Q: "Can I use multiple images per slide?"**
- A: Yes! Use list format in config: `images: [[img1.png, img2.png], ...]`. Template slide must have matching number of placeholder images.

## Technical Gotchas

### Windows Path Handling

```python
# ✅ GOOD - works on all platforms
base_dir = "C:/Users/Name/Data"  # Forward slashes work on Windows!

# ⚠️ OK - but annoying
base_dir = "C:\\Users\\Name\\Data"  # Need double backslashes

# ❌ BAD - breaks on Windows
base_dir = r"C:\Users\Name\Data"  # Raw string, but backslash issues
```

### Slide Indices (0-based)

```python
# PowerPoint UI: Slide 1, Slide 2, Slide 3
# Python indices:     0,       1,       2

template_slide: 1  # This is "Slide 2" in PowerPoint UI
```

### Image Position Units

```python
# python-pptx uses inches (not pixels, not centimeters)
left = 1.5    # 1.5 inches from left edge
top = 2.0     # 2.0 inches from top edge
width = 6.0   # 6 inches wide
height = 4.0  # 4 inches tall

# Use cm_to_inches() if needed
from ppt_image_inserter import cm_to_inches
width = cm_to_inches(15.24)  # 15.24 cm = 6 inches
```

## Testing Checklist

Before suggesting code to users, verify:

- [ ] **Paths**: Are all paths valid for user's OS?
- [ ] **Error handling**: What if image doesn't exist?
- [ ] **Indices**: Are slide indices correct (0-based)?
- [ ] **Dependencies**: Only python-pptx and PyYAML?
- [ ] **Backups**: Are destructive operations backed up?
- [ ] **Output**: Clear success/failure messages?
- [ ] **Documentation**: Is the code commented?

## Resources

- **python-pptx docs**: https://python-pptx.readthedocs.io/
- **Example configs**: See `examples_and_configs/configs/` directory
- **Target users**: Researchers, data scientists, analysts - prioritize simplicity and clear error messages

---
> Source: [UpadhyayaLab/ppt-image-inserter](https://github.com/UpadhyayaLab/ppt-image-inserter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
