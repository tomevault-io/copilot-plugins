## jurgendn-github-io

> Generates interactive map of talk locations:

# AGENTS.md - Academic Pages Jekyll Site

## Project Overview

This is an **Academic Pages** Jekyll-based GitHub Pages template for academic personal websites. It's a static site generator that transforms Markdown files and YAML data into HTML pages hosted on GitHub Pages.

**Technology Stack:**
- Jekyll (static site generator)
- Ruby/Bundler (dependency management)
- Liquid templates (templating)
- Sass/SCSS (styling)
- Minimal JavaScript (jQuery, plugins)
- Kramdown (Markdown processor)

**Template Origin:** Forked from Minimal Mistakes Jekyll Theme

## Essential Commands

### Local Development

**Using Native Ruby/Jekyll:**
```bash
# Install dependencies (first time setup)
bundle install

# If you get permission errors, install gems locally:
bundle config set --local path 'vendor/bundle'
bundle install

# Serve the site locally (auto-rebuilds on changes)
jekyll serve -l -H localhost

# Alternative (ensures specific dependencies)
bundle exec jekyll serve -l -H localhost
```

The site will be available at `http://localhost:4000`

**Using Docker:**
```bash
# Set permissions and run
chmod -R 777 .
docker compose up
```

**Using Dev Container (VS Code):**
- Open in VS Code
- F1 → "Dev Container: Reopen in Container"
- Site automatically hosted at `http://localhost:4000`

### JavaScript Build

```bash
# Install npm dependencies
npm install

# Build minified JavaScript
npm run build:js
# or
npm run uglify

# Watch for JavaScript changes
npm run watch:js
```

### Content Generation

**Generate publications/talks from TSV:**
```bash
# Using Python scripts
cd markdown_generator
python publications.py
python talks.py

# Using Jupyter notebooks (preferred for documentation)
jupyter notebook publications.ipynb
jupyter notebook talks.ipynb
```

**Generate talk map:**
```bash
# Creates geographic visualization of talk locations
python talkmap.py

# Or use Jupyter notebook
jupyter notebook talkmap.ipynb
```

**Update CV JSON from Markdown:**
```bash
./scripts/update_cv_json.sh
```

## Project Structure

### Directory Organization

```
.
├── _config.yml              # Main site configuration (CRITICAL FILE)
├── _data/                   # YAML data files
│   ├── navigation.yml       # Header navigation menu
│   ├── authors.yml          # Author information
│   ├── ui-text.yml          # UI text strings
│   └── cv.json              # JSON CV data
├── _includes/               # Reusable HTML/Liquid components
│   ├── head.html
│   ├── footer.html
│   ├── masthead.html
│   └── author-profile.html
├── _layouts/                # Page templates
│   ├── default.html         # Base layout
│   ├── single.html          # Single post/page
│   ├── archive.html         # Collection listing
│   ├── talk.html            # Talk template
│   └── cv-layout.html       # CV template
├── _sass/                   # Sass/SCSS stylesheets
│   ├── _themes.scss         # Theme variables
│   └── _syntax.scss         # Code syntax highlighting
├── assets/                  # Compiled assets
│   ├── css/
│   ├── js/
│   └── fonts/
├── images/                  # Image assets
├── files/                   # PDF, slides, papers (publicly accessible)
├── _pages/                  # Standalone pages (about, CV, etc.)
├── _posts/                  # Blog posts
├── _publications/           # Publication entries
├── _talks/                  # Talk/presentation entries
├── _teaching/               # Teaching experience entries
├── _portfolio/              # Portfolio project entries
├── _drafts/                 # Draft content (not published)
├── markdown_generator/      # Scripts to generate markdown from TSV
└── scripts/                 # Utility scripts
```

### Collections

Jekyll collections are defined in `_config.yml` and correspond to directories:
- `_publications/` → `/publication/` URLs
- `_talks/` → `/talks/` URLs  
- `_teaching/` → `/teaching/` URLs
- `_portfolio/` → `/portfolio/` URLs
- `_posts/` → `/posts/` URLs

Each collection item is a markdown file with YAML front matter.

## Content Structure

### YAML Front Matter

All content files start with YAML front matter between `---` markers.

**Blog Posts** (`_posts/YYYY-MM-DD-title.md`):
```yaml
---
title: 'Blog Post Title'
date: 2024-01-15
permalink: /posts/2024/01/blog-post-title/
tags:
  - tag1
  - tag2
---
```

**Publications** (`_publications/YYYY-MM-DD-title.md`):
```yaml
---
title: "Paper Title"
collection: publications
category: conferences  # or manuscripts, books
permalink: /publication/2024-paper-title
excerpt: 'Brief description'
date: 2024-01-15
venue: 'Conference/Journal Name'
paperurl: 'http://example.com/paper.pdf'
citation: 'Your Name. (2024). "Paper Title." <i>Venue</i>. 1(1).'
---
```

**Talks** (`_talks/YYYY-MM-DD-title.md`):
```yaml
---
title: "Talk Title"
collection: talks
type: "Talk"  # or Conference proceedings, Tutorial, etc.
permalink: /talks/2024-talk-title
venue: "Conference/Institution Name"
date: 2024-01-15
location: "City, State, Country"
---
```

**Teaching** (`_teaching/YYYY-semester-title.md`):
```yaml
---
title: "Course Name"
collection: teaching
type: "Undergraduate course"  # or Graduate course
permalink: /teaching/2024-spring-course
venue: "University Name, Department"
date: 2024-01-15
location: "City, Country"
---
```

**Portfolio** (`_portfolio/portfolio-name.md` or `.html`):
```yaml
---
title: "Portfolio Item Name"
excerpt: "Short description<br/><img src='/images/500x300.png'>"
collection: portfolio
---
```

### File Naming Conventions

- **Date-based content:** `YYYY-MM-DD-title.md` (posts, publications, talks, teaching)
- **Portfolio:** `portfolio-name.md` or `portfolio-name.html`
- **Pages:** Descriptive names like `about.md`, `cv.md`
- Use hyphens for spaces in filenames
- Keep filenames lowercase and URL-friendly

## Configuration

### `_config.yml` - Critical Settings

**Basic Site Settings:**
```yaml
locale: "en-US"
title: "Your Name / Site Title"
name: &name "Your Name"
description: &description "personal description"
url: https://username.github.io
baseurl: ""
repository: "username/username.github.io"
```

**Author Profile (sidebar):**
```yaml
author:
  avatar: "profile.png"  # in /images/
  name: "Your Sidebar Name"
  bio: "Short biography"
  location: "Earth"
  email: "email@example.org"
  github: "username"
  googlescholar: "https://scholar.google.com/..."
  orcid: "http://orcid.org/..."
  # ... many more social links available
```

**Publication Categories:**
```yaml
publication_category:
  books:
    title: 'Books'
  manuscripts:
    title: 'Journal Articles'
  conferences:
    title: 'Conference Papers'
```

**Jekyll Configuration:**
```yaml
markdown: kramdown
highlighter: rouge
plugins:
  - jekyll-feed
  - jekyll-sitemap
  - jekyll-redirect-from
  - jemoji
```

**IMPORTANT:** After changing `_config.yml`, you MUST restart Jekyll server (changes are not auto-reloaded).

### `_data/navigation.yml` - Header Menu

Controls the navigation links in the site header:

```yaml
main:
  - title: "Publications"
    url: /publications/
  - title: "Talks"
    url: /talks/
  - title: "CV"
    url: /cv/
```

Order in this file determines order in header. Remove entries to hide menu items.

## Code Patterns and Conventions

### Markdown Files

- Use **Kramdown** markdown syntax
- Code blocks: Use triple backticks with language identifier
- Math: Use MathJax syntax (`$$...$$` for display, `$...$` for inline)
- Diagrams: Mermaid.js supported
- Tables: Standard markdown tables

### Liquid Templates

Common patterns in `_includes/` and `_layouts/`:

```liquid
{% include base_path %}
{% for post in site.posts %}
  {{ post.title }}
{% endfor %}

{% if site.author.email %}
  <a href="mailto:{{ site.author.email }}">Email</a>
{% endif %}
```

### Permalinks

- Set in front matter: `permalink: /posts/2024/01/my-post/`
- Collections have default permalinks: `/:collection/:path/`
- Can use redirect_from for old URLs: `redirect_from: ["/old-url/"]`

### Images and Files

- **Images:** Place in `/images/`, reference as `/images/filename.png`
- **Files (PDFs, etc.):** Place in `/files/`, reference as `/files/paper.pdf`
- Both directories are publicly accessible after deployment
- Images in front matter excerpts: `excerpt: "Text<br/><img src='/images/image.png'>"`

## Development Workflow

### Making Changes

1. **Content changes (Markdown files):**
   - Edit files in `_posts/`, `_publications/`, etc.
   - Changes auto-reload in `jekyll serve -l` mode
   - Commit and push to trigger GitHub Pages rebuild

2. **Configuration changes (`_config.yml`):**
   - Edit `_config.yml`
   - **MUST restart Jekyll server** for changes to take effect
   - Changes take effect on next Jekyll build

3. **Template/layout changes (`_includes/`, `_layouts/`):**
   - Edit HTML/Liquid files
   - Changes auto-reload in watch mode
   - Test thoroughly as they affect multiple pages

4. **Style changes (`_sass/`):**
   - Edit SCSS files
   - Sass recompiles automatically
   - Check compiled CSS in `assets/css/`

5. **JavaScript changes:**
   - Edit source files in `assets/js/`
   - Run `npm run build:js` to minify
   - Minified output: `assets/js/main.min.js`

### Testing Before Commit

```bash
# Always test locally before pushing
bundle exec jekyll serve -l -H localhost

# Build without serving (check for errors)
bundle exec jekyll build

# Check site in browser at localhost:4000
# Test navigation, links, responsive design
```

### Git Workflow

```bash
# Check status
git status

# Stage content changes
git add _posts/ _publications/ _config.yml

# Commit with descriptive message
git commit -m "Add new publication and update bio"

# Push to trigger GitHub Pages build
git push origin main
```

**Note:** GitHub Pages automatically builds and deploys on push to `main` branch. Build status visible in repository settings → Pages.

## Markdown Generator Workflow

For managing many publications/talks via spreadsheet:

1. **Maintain data in TSV files:**
   - `markdown_generator/publications.tsv`
   - `markdown_generator/talks.tsv`

2. **Edit TSV in spreadsheet application** (Excel, Google Sheets)

3. **Run generator:**
   ```bash
   cd markdown_generator
   python publications.py  # Generates _publications/*.md
   python talks.py         # Generates _talks/*.md
   ```

4. **Review generated files** and commit

5. **Alternative:** Use Jupyter notebooks for interactive workflow:
   - `publications.ipynb`
   - `talks.ipynb`
   - Better documentation and step-by-step process

## Styling and Themes

### Theme System

- Theme controlled by `site_theme` in `_config.yml`: `"default"` or `"dark"`
- Dark theme: `site_theme: "dark"` in `_config.yml`
- Theme variables in `_sass/_themes.scss`

### Typography

Font families defined in `_sass/_themes.scss`:
```scss
$sans-serif: -apple-system, "San Francisco", "Roboto", "Segoe UI", ...;
$serif: Georgia, Times, serif;
$monospace: Monaco, Consolas, "Lucida Console", monospace;
```

Type scale: `$type-size-1` through `$type-size-8`

### Customization

- **Colors:** Edit `_sass/_themes.scss`
- **Layout:** Edit `_layouts/*.html`
- **Sidebar:** Edit `_includes/author-profile.html`
- **Footer:** Edit `_includes/footer.html`
- **Custom CSS:** Add to `_sass/` and import in main stylesheet
- **Custom JS:** Add to `assets/js/_main.js`, then run `npm run build:js`

## Important Gotchas

### Jekyll Server Restart Required

**Changes to `_config.yml` do NOT auto-reload.** You must restart `jekyll serve` to see configuration changes.

### Base Path and URL

- `url` in `_config.yml` must match your GitHub Pages URL
- For `username.github.io`, set `baseurl: ""`
- For project pages (`username.github.io/project`), set `baseurl: "/project"`
- Use `{% include base_path %}` in templates for proper URL generation

### Front Matter Dates

- Posts MUST have date in filename: `YYYY-MM-DD-title.md`
- Front matter date overrides filename date
- Future-dated posts only shown if `future: true` in `_config.yml`

### Permalink Conflicts

- Ensure permalinks are unique across all collections
- Permalink in front matter overrides defaults
- Duplicate permalinks cause one page to overwrite another

### Vendor Directory

- `vendor/` directory created if using local gem installation
- This directory should be excluded in `_config.yml` (already configured)
- Add to `.gitignore` if not already present

### File Permissions (Docker)

When using Docker, you may need to set permissions:
```bash
chmod -R 777 .
```
This is mentioned in README but easy to forget.

### Plugin Compatibility

- Only use plugins supported by GitHub Pages (whitelist in `_config.yml`)
- Custom plugins require local build + manual deployment
- Current whitelist: jekyll-feed, jekyll-gist, jekyll-paginate, jekyll-sitemap, jekyll-redirect-from, jemoji

### Image Paths

- Images must be in `/images/` directory
- Reference with leading slash: `/images/photo.png` (not `../images/photo.png`)
- Same for files: `/files/paper.pdf`

### Markdown in YAML

For complex text in front matter (like citations), use quotes:
```yaml
citation: 'Author. (2024). "Title." <i>Venue</i>. 1(1).'
```

### Collection Output

Collections must have `output: true` in `_config.yml` to generate individual pages:
```yaml
collections:
  publications:
    output: true
    permalink: /:collection/:path/
```

## Dependencies

### Ruby Gems (Gemfile)

```ruby
gem 'jekyll'
gem 'jekyll-feed'
gem 'jekyll-sitemap'
gem 'jekyll-redirect-from'
gem 'jemoji'
gem 'webrick', '~> 1.8'
gem 'github-pages'
```

### Python (for markdown generators)

Required for `markdown_generator/` scripts:
- `python-frontmatter` - Parse YAML front matter
- `getorg` - Location mapping
- `geopy` - Geocoding

Install:
```bash
pip install python-frontmatter getorg geopy
```

### Node.js (package.json)

```json
"dependencies": {
  "fitvids": "^2.1.1",
  "jquery": "^3.7.1",
  "jquery-smooth-scroll": "^2.2.0",
  "magnific-popup": "^1.2.0"
}
```

## Advanced Features

### Talk Map

Generates interactive map of talk locations:
- Reads `location` field from `_talks/*.md`
- Geocodes using Nominatim
- Generates `talkmap/org-locations.js` and `talkmap/map.html`
- Enable with `talkmap_link: true` in `_config.yml`

### CV JSON

Alternative CV format using JSON Resume schema:
- Data in `_data/cv.json`
- Template: `_layouts/cv-layout.html`
- Page: `_pages/cv-json.md`
- Convert from markdown CV: `./scripts/update_cv_json.sh`

### Analytics

Supports Google Analytics:
```yaml
analytics:
  provider: "google-analytics-4"
  google:
    tracking_id: "G-XXXXXXXXXX"
```

### Comments

Supports Disqus, Discourse, Facebook comments:
```yaml
comments:
  provider: "disqus"
  disqus:
    shortname: "your-shortname"
```

### Math, Diagrams, Plots

- **MathJax:** For LaTeX equations in markdown
- **Mermaid:** For diagrams (flowcharts, sequence diagrams)
- **Plotly:** For interactive plots

Include these in front matter or site-wide config as needed.

## Deployment

### GitHub Pages (Automatic)

1. Repository name: `username.github.io`
2. Push to `main` branch
3. GitHub Pages builds and deploys automatically
4. Check status: Repository Settings → Pages
5. Site live at `https://username.github.io`

### Manual Build (if needed)

```bash
# Build site to _site/ directory
bundle exec jekyll build

# Deploy _site/ contents to hosting provider
# (only needed if not using GitHub Pages)
```

## Troubleshooting

### Bundle Install Fails

```bash
# Permission errors
bundle config set --local path 'vendor/bundle'
bundle install

# Bundler version mismatch
gem install bundler:2.3.26
bundle install
```

### Jekyll Serve Fails

```bash
# Missing dependencies (Linux)
sudo apt install build-essential gcc make ruby-dev nodejs

# MacOS
brew install ruby node
gem install bundler
```

### Changes Not Appearing

- **Config changes:** Restart Jekyll server
- **Content changes:** Check for YAML syntax errors
- **Cached content:** Clear browser cache or use incognito
- **Future dates:** Set `future: true` in `_config.yml` for testing

### GitHub Pages Not Updating

- Check GitHub Actions tab for build errors
- Verify `_config.yml` syntax
- Ensure using GitHub Pages-compatible plugins only
- Check repository settings → Pages section

### Geocoding Fails (talkmap)

- Check internet connection
- Increase `TIMEOUT` in `talkmap.py`
- Verify location strings are geocodable (e.g., "City, Country")
- Rate limiting: Add delays between requests

## File Exclusions

Files/folders excluded from Jekyll build (in `_config.yml`):
- `.bundle`, `vendor/` (local gems)
- `node_modules/` (npm packages)
- `Dockerfile`, `docker-compose.yaml`
- `Gemfile`, `package.json`
- `.github/` (CI/CD configs)

These files exist in repo but don't appear in built site.

## Performance Tips

- Optimize images before adding to `/images/`
- Use `jekyll build --watch` for faster incremental builds during development
- Minimize custom JavaScript (site is primarily static)
- Leverage GitHub Pages CDN (automatic when using GitHub Pages)

## Additional Resources

- **Official template docs:** https://academicpages.github.io/
- **Template guide page:** https://academicpages.github.io/markdown/
- **GitHub discussions:** https://github.com/academicpages/academicpages.github.io/discussions
- **Minimal Mistakes docs:** https://mmistakes.github.io/minimal-mistakes/docs/configuration/
- **Jekyll docs:** https://jekyllrb.com/docs/
- **Kramdown syntax:** https://kramdown.gettalong.org/syntax.html
- **Liquid templates:** https://shopify.github.io/liquid/

## Quick Reference

**Most common tasks:**
```bash
# Start local development
bundle exec jekyll serve -l -H localhost

# Add a new blog post
# Create: _posts/YYYY-MM-DD-title.md with front matter

# Add a new publication
# Create: _publications/YYYY-MM-DD-title.md with front matter

# Update site config
# Edit: _config.yml
# Then: restart jekyll serve

# Build JavaScript
npm run build:js

# Generate talk map
python talkmap.py

# Deploy (using GitHub Pages)
git add .
git commit -m "Update content"
git push origin main
```

---

**Remember:** This is a Jekyll static site. Content is in Markdown, configuration in YAML, templates in Liquid/HTML, styles in Sass. Local changes preview at localhost:4000, GitHub Pages auto-deploys from `main` branch.

---
> Source: [jurgendn/jurgendn.github.io](https://github.com/jurgendn/jurgendn.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
