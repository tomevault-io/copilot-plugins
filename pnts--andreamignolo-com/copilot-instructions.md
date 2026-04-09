## andreamignolo-com

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
```bash
# This site requires Ruby 3.3.6
# Switch to Ruby 3.3.6 if using RVM
rvm use 3.3.6

# Install dependencies
bundle install

# Start development server
bundle exec middleman server

# Build the site
bundle exec middleman build
```

### Deployment
The site appears to be configured for static deployment (build output goes to `/build` directory).

## Architecture

This is a simple personal bio site built with Middleman, a Ruby-based static site generator.

**IMPORTANT: Always follow Middleman conventions and project structure. New files should fit within the existing Middleman architecture (layouts in `source/layouts/`, etc.).**

### Project Structure
- `source/` - All source content and templates
  - `index.html.erb` - Main page content with frontmatter (title, description, keywords)
  - `layouts/layout.erb` - HTML layout template with meta tags and OpenGraph properties
  - `stylesheets/site.css.scss` - Sass stylesheet with custom styling
  - `javascripts/site.js` - JavaScript (minimal)
  - `images/` - Static images
- `config.rb` - Middleman configuration with autoprefixer
- `Gemfile` - Ruby dependencies (Middleman 4.4.2, autoprefixer, haml)

### Key Features
- ERB templating with frontmatter for page metadata
- SCSS preprocessing for styles
- Autoprefixer for CSS vendor prefixes
- OpenGraph meta tags for social sharing
- Single-page personal bio with external links

### Content Management
The main bio content is in `source/index.html.erb` with a "last updated" date that should be manually updated when content changes.

## Email Newsletter System ("the becoming")

A Ruby script generates HTML email previews from markdown files and can send them to Buttondown.

### Key Files
- `scripts/preview_email.rb` - Main script for generating previews and sending to Buttondown
- `source/layouts/the-becoming-email.erb` - HTML email template (table-based for email client compatibility)
- `source/stylesheets/email.scss` - Email-specific styles
- `source/email_preview.html` - Generated preview output (not committed)
- `source/the-becoming/*.html.md` - Newsletter content in markdown with frontmatter

### Usage
```bash
# Preview locally (presents menu to select a newsletter)
ruby scripts/preview_email.rb

# Preview a specific file
ruby scripts/preview_email.rb source/the-becoming/newsletter-name.html.md

# Send to Buttondown as draft
ruby scripts/preview_email.rb --send
```

### How It Works
1. Parses frontmatter (title, issue, date) from markdown files
2. Converts markdown to HTML using Redcarpet (with footnotes enabled)
3. Processes Kramdown-style attributes (`{: .figcaption}`, `{: .center}`)
4. Compiles SCSS from `email.scss`
5. Applies the email template (`the-becoming-email.erb`)
6. Inlines CSS using Premailer for email client compatibility
7. Outputs to `email_preview.html` or sends to Buttondown API

### Important Notes
- Images use relative paths for local preview, absolute URLs (`https://andreamignolo.com/...`) for Buttondown
- Images must be deployed to the live site before they'll work in sent emails
- Requires `.env` file with `BUTTONDOWN_API_KEY` for sending to Buttondown
- The `--send` flag creates a **draft** in Buttondown (does not send to subscribers)

## Code Editing Guidelines

When making edits to HTML, CSS, or JavaScript files, always add a comment indicating that the change was made by Claude Code. Use appropriate comment syntax for each file type:

- HTML/ERB: `<!-- Added by Claude Code -->`
- CSS/SCSS: `/* Added by Claude Code */`
- JavaScript: `// Added by Claude Code`

Additionally, add explanatory comments throughout any code you write to help explain what the code does, especially for complex logic or non-obvious implementations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pnts)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pnts)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
