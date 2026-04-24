## obsidian-web-clipper

> This is a Chrome extension that clips web pages and saves them as Markdown files for use with Obsidian. The extension achieves **Notion Web Clipper level precision** using algorithmic content extraction with multi-dimensional scoring and converts HTML to Markdown using the Turndown library.

# Claude Instructions for Obsidian Web Clipper

## Project Overview
This is a Chrome extension that clips web pages and saves them as Markdown files for use with Obsidian. The extension achieves **Notion Web Clipper level precision** using algorithmic content extraction with multi-dimensional scoring and converts HTML to Markdown using the Turndown library.

## Key Files
- `manifest.json` - Chrome extension manifest (v3) 
- `popup.html` - Extension popup UI with filtering controls
- `popup.js` - Main functionality with smart content extraction algorithms
- `turndown.js` - Turndown library for HTML to Markdown conversion
- `README.md` - Comprehensive documentation and feature comparison

## Technology Stack
- Chrome Extension Manifest V3
- JavaScript (vanilla)
- Turndown library for HTML to Markdown conversion
- Chrome Downloads API for file saving
- Algorithmic content scoring (similar to Readability.js)

## Current Version
**v2.2** - Latest features include unified 680px image sizing

## Development Commands
No specific build commands - this is a vanilla JavaScript Chrome extension.

## Testing
- Load unpacked extension in Chrome developer mode
- Test on various websites (especially news/blog sites like Lifehacker, Zenn, Qiita)
- Verify smart content extraction vs raw extraction
- Test image filtering and navigation link removal
- Verify file saving and cancellation handling

## Code Style
- Use vanilla JavaScript (no frameworks)
- Follow Chrome extension best practices
- Keep code simple and maintainable
- Use Japanese comments where appropriate (project is bilingual)

## Key Features

### 🧠 Smart Content Extraction
1. **Multi-dimensional scoring**: HTML structure, text amount, class names
2. **Semantic analysis**: Prioritizes article, main, section elements  
3. **Heuristic noise removal**: Auto-detects navigation, ads, comments
4. **Site-specific optimization**: Zenn, GitHub, Medium, Qiita support

### 🎯 Advanced Filtering
1. **Small image filtering**: Removes images under 100x100px (icons, avatars)
2. **Navigation link removal**: Converts nav links to text-only
3. **Clean vs Raw modes**: User-selectable extraction precision

### 🖼️ Image Processing
1. **Unified 680px sizing**: All images display at consistent 680px width in Obsidian
2. **Obsidian syntax**: Uses `![alt|680](url)` format
3. **External URL preservation**: Images remain as links (no local storage)

### 📊 Content Structure
1. **YAML frontmatter**: Title, URL, date with proper escaping
2. **Table preservation**: Headers, cell content, complex structures
3. **Code block handling**: Language detection and syntax preservation
4. **List structure**: Nested lists, numbered lists

### 🔧 User Experience
1. **Preview functionality**: See extraction results before saving
2. **User-friendly cancellation**: No error on download cancel
3. **Format selection**: Clean (Notion-style) vs Raw (full page)
4. **Error handling**: Robust fallback mechanisms

## Important Notes
- Extension uses Chrome Downloads API (not File System Access API)
- All images are set to 680px width for consistent Obsidian display
- Smart content extraction can fall back to full page if content too filtered
- Download cancellation shows friendly message instead of error
- Supports Japanese UI and error messages
- Site-specific filtering for major platforms (Zenn, GitHub, etc.)

## Architecture
- **Content scoring**: Multi-dimensional algorithm rates elements by value
- **Main content detection**: Finds best content area using scored elements  
- **Noise filtering**: Removes navigation, ads, social elements
- **Fallback safety**: Multiple safety nets prevent empty content extraction

## Testing Strategy
- Test on news sites (large images, navigation noise)
- Test on technical blogs (code blocks, tables)
- Test on documentation sites (structured content)
- Verify both Clean and Raw modes work correctly
- Test download cancellation and error scenarios

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kohsuk3) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
