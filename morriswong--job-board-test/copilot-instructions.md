## job-board-test

> This document provides guidance for Claude AI when working on this project.

# Claude AI Development Guide

This document provides guidance for Claude AI when working on this project.

## Project Overview

This is a Hugo-based job board application inspired by Code4Lib Jobs. It's designed to be simple, fast, and easy to maintain while providing essential job board functionality.

## Key Commands

### Development
```bash
# Start local server (with drafts)
hugo server -D

# Start server and auto-navigate to changes
hugo server -D --navigateToChanged

# Start server on specific port
hugo server -D -p 8080
```

### Content Management
```bash
# Create new job posting
hugo new jobs/job-title.md

# Create new content page
hugo new page-name.md
```

### Building
```bash
# Build for production
hugo

# Build with drafts (for testing)
hugo -D

# Build with future-dated posts
hugo -F

# Clean generated files
rm -rf public/ resources/
```

## Architecture

### Content Types

**Jobs** (`content/jobs/`)
- Primary content type
- Uses TOML front matter (+++...+++)
- Required fields: title, date, employer, location, jobType
- Optional: tags array for categorization

### Taxonomies

1. **Tags**: Skills, technologies, categories
2. **Employers**: Automatic organization by employer name

### Layouts Structure

```
layouts/
├── _default/
│   └── baseof.html          # Base template with header/footer
├── index.html               # Homepage (job listings)
├── jobs/
│   ├── list.html           # /jobs/ - all jobs page
│   └── single.html         # Individual job detail
└── taxonomy/
    ├── tag.html            # Jobs by tag
    ├── tag.terms.html      # All tags list
    └── employer.terms.html # All employers list
```

### Styling

- Single CSS file: `static/css/style.css`
- Mobile-first responsive design
- Accessible and semantic HTML
- No JavaScript dependencies

## Common Tasks

### Adding a New Job

1. Create the file:
   ```bash
   hugo new jobs/position-name.md
   ```

2. Edit front matter:
   ```toml
   +++
   title = 'Position Title'
   date = 2025-10-22T10:00:00-00:00
   draft = false
   employer = 'Organization Name'
   location = 'City, State/Country'
   jobType = 'Full time'
   tags = ['skill1', 'skill2']
   +++
   ```

3. Add job description in Markdown

### Modifying Templates

When editing templates:
- Maintain semantic HTML structure
- Keep accessibility features (skip links, ARIA labels)
- Preserve responsive design
- Test on mobile and desktop

### Styling Changes

CSS is organized by:
1. Reset and base styles
2. Layout components (header, footer)
3. Content components (job cards, tags)
4. Responsive overrides

## Hugo Features Used

### Template Functions
- `range`: Iterate over collections
- `where`: Filter content
- `first`: Limit results
- `.Summary`: Auto-generated excerpt
- `.Date.Format`: Date formatting

### Taxonomies
- Automatic tag and employer organization
- `.Data.Terms`: Access taxonomy data

### Content Management
- Front matter: TOML format
- Markdown content with HTML support
- Automatic summaries and excerpts

## Development Guidelines

1. **Content First**: Always prioritize content structure and semantics
2. **Progressive Enhancement**: Base functionality should work without JS
3. **Accessibility**: Maintain WCAG 2.1 AA compliance
4. **Performance**: Keep bundle size minimal, optimize images
5. **SEO**: Use semantic HTML and proper meta tags

## Future Enhancements

Potential additions:
- RSS feed for jobs
- JSON feed for API consumption
- Search functionality (client-side with Fuse.js or server-side)
- Job application form integration
- Email notifications for new jobs
- Admin interface for job management
- Employer profiles with rich content
- Job expiration dates
- Salary range fields

## Testing Checklist

Before deployment:
- [ ] All pages render correctly
- [ ] Navigation works on all pages
- [ ] Tags and employers link properly
- [ ] Mobile responsive design works
- [ ] Job dates display correctly
- [ ] Summary text generates properly
- [ ] No broken links
- [ ] Accessibility scan passes
- [ ] Build completes without errors

## Troubleshooting

### Common Issues

**Build Errors**
- Check TOML front matter syntax
- Verify all required fields are present
- Look for unclosed template tags

**Content Not Showing**
- Check `draft` status (should be `false`)
- Verify date is not in the future
- Ensure file is in correct directory

**Styling Issues**
- Clear browser cache
- Check CSS file path in baseof.html
- Verify `static/` directory structure

## Resources

- [Hugo Documentation](https://gohugo.io/documentation/)
- [Hugo Templates](https://gohugo.io/templates/)
- [Hugo Functions](https://gohugo.io/functions/)
- [Code4Lib Jobs](https://jobs.code4lib.org/) - Original inspiration

---
> Source: [morriswong/job-board-test](https://github.com/morriswong/job-board-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
