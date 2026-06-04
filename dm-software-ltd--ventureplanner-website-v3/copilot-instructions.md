## ventureplanner-website-v3

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Venture Planner 3 is a Django-based business consulting website with an integrated blog platform. The site showcases consulting services, hosts educational blog content, and generates leads through contact forms.

## Technology Stack

- **Framework**: Django 4.2.18
- **Database**: PostgreSQL
- **File Storage**: AWS S3 (for media files)
- **CAPTCHA**: Cloudflare Turnstile
- **Python Version**: 3.x (check your local environment)

## Environment Setup

### Required Environment Variables (.env)

The project requires a `.env` file in the root directory with:

```
SECRET_KEY=<django-secret-key>
DEBUG=True

# Database
DB_NAME=<database-name>
DB_USER=<database-user>
DB_PASSWORD=<database-password>
DB_HOST=localhost
DB_PORT=5432

# AWS S3
AWS_ACCESS_KEY_ID=<aws-access-key>
AWS_SECRET_ACCESS_KEY=<aws-secret-key>
AWS_STORAGE_BUCKET_NAME=<s3-bucket-name>
AWS_S3_REGION_NAME=<aws-region>

# Cloudflare Turnstile
TURNSTILE_SITE_KEY=<site-key>
TURNSTILE_SECRET_KEY=<secret-key>

# Email (add these if not present)
DEFAULT_FROM_EMAIL=<sender-email>
CONTACT_US_EMAIL=<recipient-email>
```

### Initial Setup

```bash
# Install dependencies
pip install -r requirements.txt  # Create if needed from pip freeze

# Run migrations
python manage.py migrate

# Create superuser for admin access
python manage.py createsuperuser

# Run development server
python manage.py runserver
```

## Common Commands

### Development Server
```bash
python manage.py runserver
```

### Database Operations
```bash
# Create migrations after model changes
python manage.py makemigrations

# Apply migrations
python manage.py migrate

# Access database shell
python manage.py dbshell
```

### Static Files
```bash
# Collect static files for production
python manage.py collectstatic
```

### Admin & Shell
```bash
# Django shell
python manage.py shell

# Create admin user
python manage.py createsuperuser
```

## Architecture

### Static vs Media File Separation

**IMPORTANT**: The project has a specific architecture for file handling:

- **Static files** (CSS, JS, fonts): Served locally from `/static/` directory during development
- **Media files** (blog images, user uploads): Served from AWS S3 via `django-storages`

This is configured in `settings.py`:
```python
STATIC_URL = '/static/'
STATICFILES_DIRS = [BASE_DIR / "static"]

MEDIA_URL = f"https://{AWS_STORAGE_BUCKET_NAME}.s3.amazonaws.com/"
DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
```

### Template Structure

Templates use a component-based architecture with reusable partials:

- **Base templates**: `base.html`, `base2.html` - Define overall page structure
- **Page templates**: `pages/*.html` - Individual page content
- **Partials**: `partials/sitewide/*.html` and `partials/*-partial.html` - Reusable components

Components include: header, footer, preloader, hero sections, services, testimonials, pricing, blog listings.

### Context Processors

Two custom context processors inject data into all templates:

1. **`breadcrumbs`** (`pages/context_processors.py:4-57`): Generates page titles and breadcrumb navigation based on URL routing
2. **`turnstile_keys`** (`pages/context_processors.py:59-65`): Injects Cloudflare Turnstile site key globally

These are registered in `settings.py` TEMPLATES configuration.

### Blog System Architecture

The blog uses a relational model structure:

- **Blog** - Main blog post model with many-to-many relationships and SEO fields
- **BlogCategory** - Categorization (many-to-many with Blog)
- **BlogTag** - Tagging system (many-to-many with Blog)
- **BlogAuthor** - Author profiles (foreign key from Blog)
- **BlogComment** - Comment system with approval workflow

**Key behaviors**:
- Blog posts have a `draft` flag (only `draft=False` posts shown to users)
- Blog pagination shows 8 posts per page with SEO-friendly prev/next tags
- Blog images stored in S3 at `static/blog/images/` and `static/blog/thumbnails/`
- **Blog posts use SEO-friendly slug URLs**: `/blog/slug-goes-here/` (auto-generated from title)
- Slugs are automatically generated on save if not provided
- **SEO Fields**: `meta_description` (160 chars max), `meta_keywords`, and `slug` are available per post

### URL Routing

All page routes are defined in `pages/urls.py` (26 total routes). The project uses named URLs for internal linking and breadcrumb generation.

### Forms & CAPTCHA

Contact forms use Django Forms with Bootstrap styling:
- `ContactForm` in `pages/forms.py` handles contact page submissions
- Support submissions handled by `support_submit` view with Cloudflare Turnstile verification
- Email notifications sent via `django.core.mail.send_mail`

### Database Schema

Custom table names are defined via Meta classes:
- `blog_blog` - Blog posts
- `blog_blogcategory` - Categories
- `blog_blogtag` - Tags
- `blog_blogauthor` - Authors
- `blog_blogcomment` - Comments

This prevents Django's default `pages_*` naming convention.

### Email Configuration

The project sends emails for:
1. Contact form submissions (`contact` view)
2. Support requests (`support_submit` view)

Email settings should be configured in `settings.py` (currently using Django defaults). Add SMTP settings for production use.

### Image Data

`image_data.json` contains mappings for 36 blog posts with their associated images. This may be used for data migration or seeding.

## Development Notes

### Current State (from git status)

- `.env` file is staged but untracked in `.gitignore`
- Working branch: `claude-code-initial`
- Multiple `__pycache__/` directories present (should be in `.gitignore`)

### Recent SEO Improvements (2025)

The project has been fully optimized for SEO with the following implementations:

1. **Blog Slugs**: Blog posts now use SEO-friendly slug URLs (`/blog/how-to-build-business-plan/`) instead of IDs
2. **Meta Tags**: All pages have proper meta descriptions, keywords, Open Graph, and Twitter Card tags
3. **Canonical URLs**: All pages include canonical URL tags to prevent duplicate content issues
4. **Structured Data**:
   - BlogPosting schema on blog single pages
   - BreadcrumbList schema on all pages with breadcrumbs
   - Organization schema ready for homepage
5. **Sitemap**: XML sitemap at `/sitemap.xml` with both static pages and blog posts
6. **Robots.txt**: Dynamic robots.txt at `/robots.txt` with sitemap reference
7. **Image Optimization**:
   - 226 alt text attributes added across 40 template files
   - 225 images now use `loading="lazy"` for better Core Web Vitals
8. **Pagination SEO**: Blog pagination includes rel="prev" and rel="next" tags

### Known Issues

1. **Email settings incomplete**: `DEFAULT_FROM_EMAIL` and `CONTACT_US_EMAIL` should be added to `settings.py`
2. **OG Image**: Replace placeholder `{% static 'images/og-image.jpg' %}` with actual social sharing image
3. **Migration Required**: Run `python manage.py migrate` to apply Blog model changes (slug, meta fields)

### Testing Considerations

When making changes to:
- **Models**: Always create and run migrations
- **Templates**: Check both base templates if changes affect layout
- **Static files**: May need `collectstatic` for production
- **Forms**: Test both validation and email sending functionality
- **Blog**: Remember pagination (8 posts/page) when testing with multiple posts

## SEO Maintenance

### Adding New Blog Posts

When creating blog posts via Django admin:
1. **Slug**: Auto-generated from title, but can be manually set for optimization
2. **Meta Description**: Fill this field (160 chars max) - used for search results and social sharing
3. **Meta Keywords**: Add relevant keywords, comma-separated
4. **Draft**: Set to `False` to publish (only published posts appear on site and in sitemap)

### Updating Sitemap

The sitemap auto-updates when blog posts are published. Access at:
- `/sitemap.xml` - Main sitemap index
- Includes both static pages and published blog posts

### Monitoring SEO

Check these regularly:
- **Google Search Console**: Monitor crawl errors, sitemap status, search performance
- **PageSpeed Insights**: Verify lazy loading is working (Core Web Vitals)
- **Rich Results Test**: Verify structured data (BlogPosting, BreadcrumbList schemas)
- **Social Media Debuggers**: Test OG/Twitter cards (Facebook Sharing Debugger, Twitter Card Validator)

## Deployment Considerations

- PostgreSQL database required (not SQLite)
- AWS S3 bucket must be configured for media storage
- Cloudflare Turnstile keys needed for contact/support forms
- Email backend must be configured for notifications
- Set `DEBUG=False` and configure `ALLOWED_HOSTS` for production
- Run `python manage.py migrate` to apply database migrations
- Run `collectstatic` before deploying
- Update robots.txt sitemap URL with production domain
- Replace OG image placeholder with actual social sharing image (1200x630px recommended)

---
> Source: [DM-Software-Ltd/ventureplanner-website-v3](https://github.com/DM-Software-Ltd/ventureplanner-website-v3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
