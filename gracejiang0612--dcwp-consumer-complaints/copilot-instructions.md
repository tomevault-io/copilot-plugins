## dcwp-consumer-complaints

> - **Font**: Georgia, 'Times New Roman', serif

# Newspaper-Style Story Format Guide

## Typography Hierarchy

### Headlines
- **Font**: Georgia, 'Times New Roman', serif
- **Size**: 3.2em (desktop), 2.2em (mobile)
- **Weight**: normal
- **Line Height**: 1.1
- **Alignment**: center
- **Margin**: 30px bottom

### Deck (Subtitle)
- **Font**: Georgia, 'Times New Roman', serif (inherited)
- **Size**: 1.4em (desktop), 1.2em (mobile)
- **Weight**: 300 (light)
- **Color**: #444 (dark gray)
- **Line Height**: 1.4
- **Alignment**: center
- **Margin**: 30px bottom

### Byline
- **Font**: Arial, sans-serif
- **Size**: 1em (desktop), 0.9em (mobile)
- **Color**: #888 (medium gray)
- **Margin**: 30px auto 50px auto (top/bottom spacing)
- **Padding**: 0 60px (desktop), 0 30px (mobile)
- **Max Width**: 800px
- **Alignment**: left

### Byline Author Link
- **Font**: Arial, sans-serif
- **Weight**: bold
- **Color**: #333 (dark gray)
- **Hover**: #1a0dab (blue) with underline

### Publish Date
- **Font**: Arial, sans-serif
- **Size**: 0.8em (desktop), 0.7em (mobile)
- **Color**: #999 (light gray)
- **Margin**: 5px top

### Main Paragraphs
- **Font**: Georgia, 'Times New Roman', serif
- **Size**: 1.1em
- **Line Height**: 1.7
- **Color**: #000 (pure black)
- **Alignment**: justify
- **Margin**: 20px bottom between paragraphs
- **Container Margin**: 40px auto 30px auto (top/bottom spacing)
- **Container Padding**: 0 60px (desktop), 0 30px (mobile)
- **Container Max Width**: 800px

### Image Captions
- **Font**: Georgia, 'Times New Roman', serif
- **Size**: 0.9em
- **Style**: italic
- **Color**: #666 (medium gray)
- **Alignment**: center
- **Margin**: 20px 0
- **Padding**: 0 20px

### Author Bio
- **Font**: Arial, sans-serif
- **Size**: 0.95em
- **Color**: #666 (medium gray)
- **Margin**: 50px top
- **Padding**: 30px top
- **Border**: 1px solid #ddd (top border)
- **Alignment**: center
- **Line Height**: 1.6

### Author Bio Links
- **Color**: #1a0dab (blue)
- **Weight**: bold
- **Hover**: #c0392b (red) with underline

## Layout Structure

### Header Section
- **Display**: flex
- **Alignment**: stretch
- **Min Height**: 80vh
- **Gap**: 0
- **Mobile**: column direction

### Header Content (Left Side)
- **Flex**: 1
- **Padding**: 60px 40px
- **Margin**: 0 20px
- **Display**: flex column
- **Justify**: center

### Header Image (Right Side)
- **Flex**: 0 0 auto
- **Alignment**: flex-start
- **Justify**: flex-end
- **Image Max Width**: 120%
- **Image Max Height**: 100vh
- **Object Fit**: contain

### Story Content Container
- **Max Width**: 800px
- **Centered**: auto margins
- **Responsive**: 100% width on mobile

## Links & Interactive Elements

### General Links
- **Color**: #1a0dab (blue)
- **Decoration**: none
- **Hover**: #c0392b (red) with underline

### Section Breaks
- **Content**: "***"
- **Color**: #999
- **Size**: 1.2em
- **Letter Spacing**: 0.5em
- **Margin**: 40px 0

## Responsive Design

### Mobile Breakpoint
- **Max Width**: 600px
- **Header**: column layout
- **Image Height**: 50vh max
- **Padding**: reduced to 20px/30px
- **Margins**: adjusted for mobile

## SEO & Metadata

### Meta Tags
- **Description**: Compelling summary for search results
- **Keywords**: Relevant terms for search engines
- **Author**: Grace Jiang
- **Robots**: index, follow

### Social Media (Open Graph)
- **Title**: Article headline
- **Description**: Article summary
- **Image**: Hero image
- **Type**: article
- **Site Name**: Grace Jiang - Data Journalism

### Twitter Cards
- **Card Type**: summary_large_image
- **Creator**: @gracejiang0612

### Structured Data (JSON-LD)
- **Type**: NewsArticle
- **Schema**: Complete article metadata
- **Keywords**: Array of relevant terms

## File Structure
- **HTML**: index.html
- **Images**: final img/ folder
- **Data Viz**: dataviz/ folder (SVG format)
- **Favicon**: final img/tab.png
- **SEO Files**: sitemap.xml, robots.txt

## Content Guidelines
- **Quotes**: Use HTML entities (&ldquo; &rdquo; &lsquo; &rsquo;)
- **Images**: Place before captions
- **Data Viz**: Embed as SVG objects for selectable text
- **Links**: Include relevant external sources
- **Author Bio**: Include website and GitHub links

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gracejiang0612) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
