## figma-restoration-mcp-vue-tools

> Centralized MCP tool setup, configuration, and usage guidance for all Figma restoration tools including screenshots, comparisons, and asset optimization.

# Tool Configuration

## Purpose
Centralized MCP tool setup, configuration, and usage guidance for all Figma restoration tools including screenshots, comparisons, and asset optimization.

## Core Concepts
- **MCP Tools**: Model Context Protocol tools for Figma restoration workflow
- **Tool Parameters**: Configuration settings and usage parameters
- **Path Requirements**: Absolute path specifications for tool operations
- **Usage Patterns**: Common tool usage scenarios and examples
- **Integration**: Tool integration within the restoration workflow

## Essential MCP Tools

### Figma Data Extraction
- **Tool**: `mcp_figma-context_get_figma_data`
- **Purpose**: Extract Figma design data and node information
- **Parameters**:
  - `fileKey`: Figma file identifier from URL
  - `nodeId`: Specific node ID (optional)
  - `savePath`: Absolute path for saving JSON data
  - `depth`: Tree traversal depth (optional)

### Asset Downloads
- **Tool**: `mcp_figma-context_download_figma_images`
- **Purpose**: Download SVG and PNG assets from Figma
- **Parameters**:
  - `fileKey`: Figma file identifier
  - `nodes`: Array of node objects with nodeId and fileName
  - `localPath`: Absolute path for saving assets
  - `pngScale`: Export scale for PNG images (default: 2, recommended: 3)
  - `svgOptions`: SVG export configuration

### Component Screenshots
- **Tool**: `mcp_figma_restoration_mcp_vue_tools_snapdom_screenshot`
- **Purpose**: Take high-quality 3x scale screenshots using snapDOM technology
- **Parameters**:
  - `componentName`: Name of component to screenshot
  - `projectPath`: Path to Vue project
  - `outputPath`: Custom output path for screenshot
  - `selector`: Custom CSS selector (optional)
  - `snapDOMOptions`: Screenshot configuration
  - `viewport`: Viewport size configuration

### Pixel Comparison
- **Tool**: `mcp_figma_restoration_mcp_vue_tools_figma_compare`
- **Purpose**: Compare screenshots and generate diff.png
- **Parameters**:
  - `componentName`: Name of component to compare
  - `projectPath`: Path to Vue project
  - `outputPath`: Custom output directory (optional)
  - `threshold`: Comparison threshold (0-1, lower is more strict)

### SVG Optimization
- **Tool**: `mcp_figma_restoration_mcp_vue_tools_optimize_svg`
- **Purpose**: Optimize SVG files using SVGO
- **Parameters**:
  - `inputPath`: Path to input SVG file
  - `outputPath`: Path to output SVG file (optional)
  - `svgoConfig`: Custom SVGO configuration (optional)

### Image Optimization (PNG/JPEG/WebP)
- **Tool**: `mcp_figma_restoration_mcp_vue_tools_optimize_image`
- **Purpose**: Optimize PNG, JPEG, and WebP image files using Sharp
- **Parameters**:
  - `inputPath`: Path to input image file (required)
  - `outputPath`: Path to output image file (optional, overwrites original if not specified)
  - `quality`: Image quality 0-100 (default: 85 for JPEG/WebP)
  - `compressionLevel`: PNG compression level 0-9 (default: 9)
  - `format`: Output format "png", "jpeg", "webp", or "auto" (default: "auto")
  - `progressive`: Enable progressive encoding for JPEG (default: true)
  - `mozjpeg`: Use MozJPEG encoder (default: true)
  - `resize`: Optional resize settings with width, height, fit

### Visual Image Analysis (Critical for Visual-Driven Development)
- **Primary Method**: Use Claude's built-in image recognition capabilities
- **Purpose**: Direct visual analysis of images for structure identification and debugging
- **Critical Use Cases**:
  - Analyze expected.png for visual structure identification using Claude's vision
  - Analyze diff.png for difference pattern recognition
  - Identify visual elements, layout hierarchy, and structural patterns
  - Debug restoration issues through direct visual inspection
- **Implementation**: 
  - Read image files directly using readFile tool
  - Provide images to Claude for visual analysis
  - Use Claude's image recognition for element identification and layout analysis
- **Advantages**: Direct visual understanding, no conversion overhead, immediate analysis

## Tool Configuration Standards

### Path Requirements
**Always use absolute paths for MCP tool calls**
- **Figma data**: Save to `{current_directory}/figma-restoration-mcp-vue-tools/src/figma-data/`
- **Expected images**: Save to `{current_directory}/figma-restoration-mcp-vue-tools/src/components/{ComponentName}/results/expected.png`
- **Assets**: Save to `{current_directory}/figma-restoration-mcp-vue-tools/src/components/{ComponentName}/images/`
- **Screenshots**: Save to `{current_directory}/figma-restoration-mcp-vue-tools/src/components/{ComponentName}/results/actual.png`
- **Knowledge base**: Store in `{current_directory}/figma-restoration-mcp-vue-tools/src/restoration-tips/`

### Key Tool Parameters

#### Screenshot Configuration
- **Scale**: Always use 3x for high resolution
- **Padding**: Set to 0 for shadowless components (precise cropping)
- **Background**: "transparent"
- **Viewport**: Default 1440x800, adjust based on component needs

#### Comparison Settings
- **Threshold**: 0.02 (98% accuracy requirement)
- **Output**: diff.png for visual analysis

#### Development Server
- **Port**: 1932 (default for component testing)
- **Hot reload**: Enable for rapid development iteration
- **Error overlay**: Display compilation errors in browser
- **Source maps**: Enable for debugging

#### SVG Optimization
- **Configuration**: Use default SVGO configuration for optimal file size and performance
- **Processing**: Always optimize SVG assets after download
- **Sizing**: Original SVG size must equal CSS size (no scaling)

#### Image Optimization
- **Format Preservation**: Keep original format by default (format: "auto" or don't specify)
- **Quality Settings**: Use quality=85 for JPEG/WebP, compressionLevel=9 for PNG
- **Target Reduction**: Aim for 60-70% file size reduction while maintaining web-quality visuals
- **Processing**: Always optimize all downloaded PNG/JPEG assets after download

## Asset Identification Rules

### Name Patterns
Look for these keywords when identifying assets:
- `ic`, `ic_` - Icon prefixes
- `素材`, `material` - Material/asset indicators
- `asset`, `icon`, `img`, `image` - Direct asset references

### Node Type Priority
When selecting nodes for asset download:
1. **RECTANGLE/VECTOR/PATH** - Highest priority (actual shapes)
2. **INSTANCE/COMPONENT** - Medium priority (component instances)
3. **GROUP/FRAME** - Lowest priority (containers)

### Complex Element Strategy
Download complete UI sections as single assets for:
- Mobile status bars with detailed icons and precise layouts
- Multi-layer navigation bars with complex controls
- 3D effect cards and complex shadows/gradients
- Complex icon combinations and nested groups
- Elements with imageRef fills or complex boolean operations

## Usage Examples

### Basic Figma Data Extraction
```javascript
mcp_figma-context_get_figma_data({
  fileKey: "abc123def456",
  savePath: "/absolute/path/to/figma-restoration-mcp-vue-tools/src/figma-data/"
})
```

### Asset Download with Optimization
```javascript
// Download assets
mcp_figma-context_download_figma_images({
  fileKey: "abc123def456",
  nodes: [
    { nodeId: "1234:5678", fileName: "icon.svg" },
    { nodeId: "9876:5432", fileName: "background.png" }
  ],
  localPath: "/absolute/path/to/component/images/",
  pngScale: 3,
  svgOptions: {
    outlineText: true,
    simplifyStroke: true,
    includeId: false
  }
})

// Optimize SVG assets
mcp_figma_restoration_mcp_vue_tools_optimize_svg({
  inputPath: "/absolute/path/to/component/images/icon.svg"
})

// Optimize PNG/JPEG assets (保持原格式)
mcp_figma_restoration_mcp_vue_tools_optimize_image({
  inputPath: "/absolute/path/to/component/images/background.png",
  quality: 85,
  compressionLevel: 9
})
```

### Component Screenshot and Comparison
```javascript
// Take screenshot
mcp_figma_restoration_mcp_vue_tools_snapdom_screenshot({
  componentName: "MyComponent",
  projectPath: "/absolute/path/to/figma-restoration-mcp-vue-tools",
  outputPath: "/absolute/path/to/component/results/actual.png",
  snapDOMOptions: {
    backgroundColor: "transparent",
    compress: true,
    embedFonts: true
  },
  viewport: { width: 1440, height: 800 }
})

// Compare with expected
mcp_figma_restoration_mcp_vue_tools_figma_compare({
  componentName: "MyComponent",
  projectPath: "/absolute/path/to/figma-restoration-mcp-vue-tools",
  threshold: 0.02
})
```

### Visual Image Analysis for Debugging
```javascript
// Direct visual analysis using Claude's image recognition
// Read diff.png for visual difference pattern analysis
readFile({
  path: "src/components/ComponentName/results/diff.png",
  explanation: "Analyzing diff image for restoration debugging"
})

// Read expected.png for visual structure analysis
readFile({
  path: "src/components/ComponentName/results/expected.png", 
  explanation: "Analyzing expected image for layout structure identification"
})
```

## Integration Patterns

### Workflow Integration
1. **Phase 1**: Use `mcp_figma-context_get_figma_data` and `mcp_figma-context_download_figma_images`
2. **Phase 2**: 
   - Download assets with `mcp_figma-context_download_figma_images`
   - Optimize SVG assets with `mcp_figma_restoration_mcp_vue_tools_optimize_svg`
   - Optimize PNG/JPEG assets with `mcp_figma_restoration_mcp_vue_tools_optimize_image` (保持原格式)
   - Take screenshots with `mcp_figma_restoration_mcp_vue_tools_snapdom_screenshot`
   - Compare with `mcp_figma_restoration_mcp_vue_tools_figma_compare`
3. **Optimization**: Use Claude's visual analysis capabilities for image analysis if accuracy < 98%

### Error Handling
- **Path Validation**: Verify absolute paths before tool calls
- **Directory Management**: Ensure target directories exist
- **URL Validation**: Validate Figma URLs and node IDs
- **Server Availability**: Check dev server port availability

## Reference

### Tool Quick Reference
| Tool | Purpose | Key Parameters |
|------|---------|----------------|
| `get_figma_data` | Extract Figma data | fileKey, savePath |
| `download_figma_images` | Download assets | fileKey, nodes, localPath, pngScale |
| `snapdom_screenshot` | Take screenshots | componentName, projectPath, outputPath |
| `figma_compare` | Compare images | componentName, projectPath, threshold |
| `optimize_svg` | Optimize SVG | inputPath, outputPath |
| `optimize_image` | Optimize PNG/JPEG/WebP | inputPath, quality, compressionLevel |
| `readFile` | Visual image analysis | path, explanation |

### Configuration Checklist
- [ ] Always use absolute paths
- [ ] Set screenshot scale to 3x
- [ ] Use 0.02 threshold for comparisons
- [ ] Optimize SVG assets after download
- [ ] Optimize PNG/JPEG assets after download (保持原格式)
- [ ] Validate paths before tool calls
- [ ] Check dev server on port 1932
- [ ] Use transparent background for screenshots
- [ ] Apply proper viewport settings

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tianmuji)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tianmuji)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
