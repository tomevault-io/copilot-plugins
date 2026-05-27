## 3shake-marp-templates

> This is a Marp-based presentation template repository for 3-SHAKE Inc. providing branded templates, themes, and tools for creating consistent, professional presentations using Markdown.

# Cursor IDE Rules for 3-SHAKE Marp Templates

## Repository Overview
This is a Marp-based presentation template repository for 3-SHAKE Inc. providing branded templates, themes, and tools for creating consistent, professional presentations using Markdown.

## File Structure Guidelines
- `templates/` - Base presentation templates to copy for new presentations
- `themes/` - CSS themes defining visual styling and layout rules
- `slides/` - Actual presentations, organized by year
- `assets/images/` - Shared images, organized by year and presentation
- `examples/` - Sample presentations and usage examples

## Presentation Development Best Practices

### Content Strategy
- Follow narrative arc: Introduction → Problem → Solution → Conclusion
- One message per slide: Keep each slide focused on a single concept
- Use concrete examples: Replace abstract concepts with specific data and real-world examples
- Audience-appropriate content: Adjust technical depth based on audience expertise

### Visual Design Principles
- Leverage contrast: Use color, size, and positioning to highlight key information
- Embrace white space: Avoid cramming content; prioritize readability
- Maintain consistency: Standardize fonts, colors, and layouts throughout
- Strategic image use: Combine text with relevant visuals for better retention

### Technical Implementation

#### Standard Marp Layouts
```markdown
<!-- Lead (title) slides -->
<!-- _class: lead -->
# Presentation Title

<!-- Two-column layout -->
<!-- _class: split -->
## Left Content    ## Right Content

<!-- Dark theme slides -->
<!-- _class: invert -->
<!-- _backgroundColor: #1a1a1a -->
```

#### Advanced Image and Text Layouts
```markdown
<!-- Image left, text right layout -->
## <span class="highlight-blue">Section Title</span>

<div style="display: flex; gap: 40px;">
<div style="width: 35%;">
<img src="../../assets/images/2025/presentation-name/image.jpg" alt="description" style="width: 100%; height: fit-content;">
<div style="font-size: 0.7em; text-align: left; margin-top: 5px;">
Source: Image attribution
</div>
</div>

<div style="flex: 1;">
Main content goes here.</br></br>

1. **Point 1**
2. **Point 2**  
3. **Point 3**
</div>
</div>

<!-- Centered image layout -->
<div style="text-align: center;">
<img src="diagram.png" alt="diagram" style="max-width: 80%; height: auto;">
</div>

**Figure 1: Diagram description**

<!-- Multiple images side by side -->
<div style="display: flex; gap: 20px; justify-content: center;">
<div style="text-align: center;">
<img src="image1.png" alt="desc1" style="width: 300px;">
<div style="font-size: 0.8em;">Caption 1</div>
</div>
<div style="text-align: center;">
<img src="image2.png" alt="desc2" style="width: 300px;">
<div style="font-size: 0.8em;">Caption 2</div>
</div>
</div>
```

#### Advanced Styling
```markdown
<!-- Highlight colors -->
<span class="highlight-blue">Important keywords</span>
<span class="highlight-yellow">Warning text</span>

<!-- Custom blockquotes -->
<blockquote style="border-left: 4px solid #4AADDD; padding-left: 1em; font-style: italic;">
Important quotation or emphasis
</blockquote>

<!-- Responsive images -->
<img src="image.png" style="max-width: 100%; height: auto; border-radius: 8px;">
```

### Development Workflow

#### Creating New Presentations
1. Copy appropriate template from `templates/`
2. Place in `slides/[year]/[presentation-name].md`
3. Create corresponding image directory: `assets/images/[year]/[presentation-name]/`
4. Reference theme using frontmatter: `theme: ./themes/3shake-theme.css`

#### Development Commands
```bash
# Start local server with live reload
npm start

# Start with preview mode
npm run preview

# Convert to PDF
npx @marp-team/marp-cli@latest slides/[presentation].md --pdf --allow-local-files

# Convert to PowerPoint
npx @marp-team/marp-cli@latest slides/[presentation].md --pptx --allow-local-files

# Convert to HTML
npx @marp-team/marp-cli@latest slides/[presentation].md --html --allow-local-files
```

#### Neovim Integration (marp.nvim)
```lua
-- Setup marp.nvim plugin (lazy.nvim)
{
  'nwiizo/marp.nvim',
  ft = "markdown",
  config = function()
    require("marp").setup {
      marp_command = "/path/to/marp",
      server_mode = false
    }
  end,
}

-- Key commands:
-- :MarpWatch - Start live preview
-- :MarpStop - Stop watching
-- :MarpExport [format] - Export to various formats
-- :MarpTheme [theme] - Switch themes
```

### Performance and Organization
- Optimize image sizes: Use appropriately sized images for web delivery
- Maintain file structure: Follow `assets/images/year/presentation-name/` convention
- Choose appropriate themes: Select themes that match presentation context
- Test across formats: Verify appearance in HTML, PDF, and PPTX outputs
- HTML/CSS flexibility: Leverage direct HTML and CSS for advanced layouts

### Theme System
- Primary theme: `3shake-theme.css` with brand colors (#4AADDD, #0a1929, #ECBE30)
- Auto-inject company logo via CSS `::after` pseudo-elements
- Support custom classes for different slide layouts (`lead`, `split`, etc.)
- Theme inheritance: Custom themes extend Marp's default theme

### Citation System
Use superscript notation with automatic formatting:
```markdown
Important claim^[1]^

> [1] https://example.com
```

### Time Management Guidelines
- 1-2 minutes per slide: Calculate total slides based on presentation duration
- Include buffer time: Account for Q&A and potential technical issues
- Practice with timing: Rehearse presentation to validate pacing

### Engagement Strategies
- Repeat key messages: Reinforce important points multiple times
- Interactive elements: Include polls, questions, or discussion prompts
- Backup materials: Prepare appendix slides for detailed Q&A responses

## Code Quality Standards
- Always include alt text for images
- Use semantic HTML structure when leveraging custom layouts
- Maintain consistent styling across presentations
- Follow accessibility guidelines for color contrast and font sizes
- Test presentations across different export formats before delivery

## Editor Integration
- VS Code: Supports Marp preview with HTML enabled
- Neovim: Use marp.nvim for live preview and export functionality
- Both editors support the file structure and theme system seamlessly

---
> Source: [nwiizo/3shake-marp-templates](https://github.com/nwiizo/3shake-marp-templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
