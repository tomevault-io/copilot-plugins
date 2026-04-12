## nickhuangcyh-github-io

> All posts and pages must include proper YAML front matter:

# Jekyll Content Guidelines

## Front Matter Structure
All posts and pages must include proper YAML front matter:

```yaml
---
layout: post
title: "Your Post Title"
date: YYYY-MM-DD
categories: [Category1, Category2]
tags: [tag1, tag2]
author: Nick Huang
last_modified_at: YYYY-MM-DD
canonical_url: https://nickhuangcyh.github.io/zh/blog/your-post/
keywords: keyword1, keyword2, keyword3
---
```

## Content Organization
- **English posts**: `_posts/` directory
- **Chinese posts**: `_i18n/zh/_posts/` directory
- **Static pages**: `_pages/` directory
- **Projects**: `_projects/` directory

## File Naming Convention
- Posts: `YYYY-MM-DD-title-with-hyphens.md`
- Pages: `page-name.md`
- Projects: `project-name.md`

## Image Handling
- Store images in `assets/img/`
- Use relative paths: `![Alt text](/assets/img/image-name.jpg)`
- Optimize images for web (compress, appropriate dimensions)

## Code Blocks
Use proper syntax highlighting:
```markdown
```python
def example():
    return "Hello World"
```
```

## Links and References
- Internal links: `[Link text](/relative/path/)`
- External links: `[Link text](https://external-url.com)`
- Academic citations: Use the citation system in `_includes/citation.liquid`

## SEO Best Practices
- Include descriptive titles and meta descriptions
- Use relevant keywords in front matter
- Add canonical URLs for duplicate content
- Include proper heading hierarchy (H1, H2, H3)
description:
globs:
alwaysApply: false
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nickhuangcyh)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nickhuangcyh)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
