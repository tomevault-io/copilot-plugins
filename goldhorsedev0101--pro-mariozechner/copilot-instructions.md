## pro-mariozechner

> A minimalist personal blog and portfolio website built with **blargh**, a custom static site generator using JavaScript/EJS templating.

# mariozechner.at - Personal Blog & Portfolio

A minimalist personal blog and portfolio website built with **blargh**, a custom static site generator using JavaScript/EJS templating.

## Project Structure

```
.
├── src/                    # Source files
│   ├── posts/             # Blog posts (YYYY-MM-DD-title format)
│   │   └── */             # Each post directory contains:
│   │       ├── index.md   # Post content in Markdown
│   │       ├── meta.json  # Post metadata
│   │       └── media/     # Images, videos, and other assets
│   ├── _css/              # Modular CSS files
│   ├── _icons/            # SVG icons
│   ├── _partials/         # Reusable HTML templates
│   ├── components.js      # Web components (theme toggle, color band)
│   ├── index.html         # Homepage template
│   └── style.css          # Main stylesheet (imports _css files)
├── html/                  # Generated output (git-ignored)
├── publish.sh             # Build & deployment script
└── docker/                # Server-side Docker configuration
```

## Common Tasks

### Add a New Blog Post
1. Check today's date first: `date +%Y-%m-%d`
2. Create directory: `src/posts/YYYY-MM-DD-post-title/` (use short slugs)
3. Add `meta.json`:
   ```json
   {
       "title": "Post Title",
       "date": "YYYY-MM-DD",
       "image": "media/header.png",
       "caption": "Header image caption",
       "description": "SEO description",
       "published": true
   }
   ```
4. Write content in `index.md` with EJS front matter:
   ```ejs
   <%
   	meta("../../meta.json")
   	meta()
   	const path = require('path');
   	url = url + "/posts/" + path.basename(path.dirname(outputPath)) + "/";
   %>
   <%= render("../../_partials/post-header.html", { title, image, url }) %>

   <h1 class="toc-header">Table of contents</h1>
   <div class="toc">
   %%toc%%
   </div>

   Your post content here...

   <%= render("../../_partials/post-footer.html", { title, url }) %>
   ```
5. Add header image (png or jpg) to `media/` subdirectory

### Code blocks with height limit
For long code blocks, wrap them in a scrollable container with fade effect:

```html
<div class="code-preview">

```typescript
// your code here
```

</div>
```

### Figures and videos
If you are asked to add an image with caption, use:

<figure>
<img src="...">
<figcaption></figcaption>
</figure>

If you are asked to add a video, you must put the closing tag on its own line:

<video src="...">
</video>

### Development Mode
```bash
# Run blargh with watch mode and local server
./dev.sh

# The site will be available at http://localhost:8080
# Changes to source files will automatically rebuild
# User can see live-updates in browser
```

### Build for Production
```bash
blargh --in src --out html
```

### Deploy to Production
```bash
./publish.sh         # Client only
./publish.sh server  # Client + restart server. Only use this if things in docker/ changed
```

## Key Features

- **Templating**: EJS-style syntax with `<%= %>` for expressions and `<% %>` for logic
- **Markdown**: Full support with HTML, code highlighting, and KaTeX math
- **Media**: Lazy-loaded images and HTML5 video support
- **RSS**: Auto-generated feed at `/rss.xml`
- **Themes**: Dark/light mode toggle with localStorage persistence
- **Responsive**: Max width 720px for optimal readability

## Template Functions

- `meta()` - Include page metadata
- `render(path, data)` - Render partial templates
- `metas(path)` - Get metadata for posts
- `rss(path, channel, items)` - Generate RSS feed

## Styling
- Atom One Dark syntax highlighting
- Custom color palette in `_css/colors.css`
- Utility classes in `_css/utilities.css`

## Special Markdown Features

- `%%toc%%` - Auto-generate table of contents (wrap in `<div class="toc">%%toc%%</div>`)
- `<q-l href="..."></q-l>` - Styled external links, only used on landing page!
- Standard GitHub Flavored Markdown support

## Writing Guidelines

- NEVER use em-dashes. Start a new sentence, or if you must, use a colon.
- Always add <img> and <video> with lazy loading

## Dictation Workflow

When user dictates paragraphs:
- Fix grammar and spelling only
- Do NOT change tone or content
- If user asks for options or to "tighten it up": output suggestions in chat first, wait for confirmation before applying
- Apply directly to file only after confirmation

---
> Source: [goldhorsedev0101/pro-mariozechner](https://github.com/goldhorsedev0101/pro-mariozechner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
