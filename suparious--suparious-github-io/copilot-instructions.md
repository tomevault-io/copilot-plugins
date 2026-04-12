## suparious-github-io

> **Attach this to every suparious.github.io conversation. Contains everything needed to help effectively.**

# CLAUDE.md - AI Assistant Context

**Attach this to every suparious.github.io conversation. Contains everything needed to help effectively.**

---

## ⚡ QUICK START

**Job**: Help with suparious.com portfolio website (GitHub Pages)
**Rule**: No workarounds, no temp fixes, complete solutions only
**User**: Shaun Prince - Expert developer, intolerant of cruft
**Preference**: Direct command execution with immediate verification

**Process**:
1. Read prompt fully
2. Check project structure and content requirements
3. Use decision trees below
4. Execute commands directly when possible
5. Validate end-to-end in browser

---

## 🌐 PROJECT OVERVIEW

**Vision**: Professional portfolio and technical publication platform showcasing 20+ years of experience in Cloud Architecture, AI Engineering, and DevOps.

### What is suparious.com?
- **Public Website**: https://suparious.com (GitHub Pages)
- **Primary Purpose**: Portfolio and technical publications
- **Tech Stack**: HTML5, CSS3, Vanilla JavaScript, Particles.js, Typed.js, AOS
- **Deployment**: Automatic via GitHub Pages on push to main
- **Type**: Static site (no build process required)

### Key Features
1. **Modern Portfolio**: Interactive resume with animations and effects
2. **Technical Publications**: 8+ in-depth technical articles and guides
3. **Dark/Light Mode**: Automatic theme detection with manual toggle
4. **Responsive Design**: Optimized for all devices
5. **Progressive Web Features**: Service Worker for offline support
6. **SEO Optimized**: Meta tags, sitemap, structured data

### Core Sections
- **Hero**: Animated introduction with particle effects
- **About**: Professional bio with code display
- **Skills**: Categorized expertise with progress bars
- **Experience**: Interactive timeline of work history
- **Projects**: Featured projects and GitHub integration (31+ open source projects)
- **Publications**: Technical articles and guides
- **Contact**: Contact form with validation

---

## 📊 PROJECT STATUS

**Current Version**: 2.0.0 (December 2024)
**Status**: Production - Active portfolio site

### ✅ Version 2.0.0 Features (COMPLETE)
- Modern glassmorphism UI design
- Dark/light mode with automatic detection
- Interactive particle.js background
- Typed.js dynamic role animation
- AOS scroll-triggered animations
- 8+ technical publication articles
- Responsive design for all devices
- Service Worker for offline support
- SEO optimized with sitemap

### 📝 Content Management
**Publications Directory Structure**:
- Each article in own subdirectory at root level (e.g., `/article-name/index.html`)
- Articles inherit parent site styles and themes via relative paths
- Standard navbar and footer maintained across all articles
- Dark/light mode works automatically when parent scripts included

**Current Publications**:
1. Corporate Marketing Failures - Analysis of corporate communication blunders
2. Nixon vs Carter Economic Analysis - Historical economic policy comparison
3. Debian Audio Production Guide - Complete Linux audio workstation setup
4. Self-Hosting AI Stack - Guide to self-hosted AI infrastructure
5. Sci-Fi Tech Predictions - Analysis of technology predictions in science fiction
6. Shell Scripts Collection - Curated collection of useful shell scripts
7. 2012 Chevy Volt Highway Guide - Long-distance EV travel tips
8. Automotive AC Repair Guide - DIY automotive air conditioning repair

---

## 📚 PLATFORM INTEGRATION

**Related Repositories**:
- **solidrust.github.io**: `/Users/shaun/repos/solidrust.github.io/` - SolidRusT main site (AI chat example)
- **srt-inference-proxy**: `/Users/shaun/repos/srt-inference-proxy/` - Artemis proxy (if AI features added)

**Potential Future Integration**:
- **Artemis Proxy**: `https://artemis.hq.solidrust.net/v1/chat/completions` for AI chat
- **Pattern**: See solidrust.net for working AI chat integration

**When NOT to Query**:
- ❌ HTML/CSS/JavaScript development (use project README)
- ❌ Static site design patterns (standard web development)
- ❌ GitHub Pages configuration (see GitHub docs)
- ❌ Content writing and article creation (use AI_STYLE_GUIDE.md)

---

## 🏗️ ARCHITECTURE

### Site Structure
```
User (suparious.com)
    ↓
GitHub Pages (static hosting)
    ↓
index.html (main portfolio)
    ├── assets/css/ (styles, themes)
    ├── assets/js/ (animations, interactivity)
    └── publications/ (technical articles)
```

### Content Inheritance Pattern
```
Root Site (index.html)
    ├── assets/css/main.css (theme variables, core styles)
    ├── assets/css/animations.css (animation library)
    ├── assets/css/responsive.css (mobile optimizations)
    ├── assets/js/main.js (core functionality)
    ├── assets/js/theme.js (dark/light mode system)
    └── assets/js/animations.js (scroll and hover animations)
        ↓ (inherited by all articles via ../)
Articles (/publication-name/index.html)
    └── Links: ../assets/css/*, ../assets/js/*
```

### Key Design Patterns
- **CSS Variables**: Centralized theming via CSS custom properties
- **Progressive Enhancement**: Core functionality works without JavaScript
- **Mobile-First**: Responsive design starts with mobile, enhances for desktop
- **Accessibility**: WCAG 2.1 A compliant, semantic HTML
- **Performance**: Lazy loading, optimized assets, service worker caching

---

## 🗂️ LOCATIONS

**Repository**:
- **Parent Repo**: `/mnt/c/Users/shaun/repos/srt-hq-k8s` (platform documentation)
- **Submodule**: `infrastructure/suparious-site/` (this repository as submodule)
- **Standalone Clone**: `/mnt/c/Users/shaun/repos/suparious.github.io` (independent development)
- **GitHub Remote**: https://github.com/suparious/suparious.github.io

**Deployment**:
- **Production URL**: https://suparious.com
- **GitHub Pages**: https://suparious.github.io (canonical)
- **Deploy Method**: Automatic on push to main branch
- **DNS**: CNAME record pointing suparious.com → suparious.github.io

**Key Files**:
- `index.html` - Main portfolio page
- `CNAME` - Custom domain configuration (suparious.com)
- `sitemap.xml` - SEO sitemap
- `robots.txt` - Crawler instructions
- `sw.js` - Service Worker for offline support
- `AI_STYLE_GUIDE.md` - Content writing guidelines
- `README.md` - Project documentation
- `ROADMAP.md` - Development roadmap

---

## 🛠️ TECH STACK

### Frontend
- **HTML**: HTML5 semantic markup
- **CSS**: CSS3 with custom properties, Flexbox, Grid
- **JavaScript**: Vanilla ES6+, no framework dependencies
- **Fonts**: Inter, Space Grotesk (Google Fonts)
- **Icons**: Font Awesome 6, Devicon

### Libraries (CDN)
- **Particles.js**: Interactive particle network background
- **Typed.js**: Dynamic typing animation for hero section
- **AOS**: Scroll-triggered animations (Animate On Scroll)

### Infrastructure
- **Hosting**: GitHub Pages (free, SSL included)
- **Domain**: suparious.com (custom domain via CNAME)
- **CDN**: GitHub's global CDN for fast delivery
- **SSL**: Automatic via GitHub Pages (Let's Encrypt)

### Development Tools
- **Version Control**: Git (GitHub)
- **Editor**: VS Code / Cursor
- **Browser DevTools**: Chrome/Firefox for testing
- **Lighthouse**: Performance and SEO auditing

---

## 📁 PROJECT STRUCTURE

```
suparious.github.io/
├── index.html                  # Main portfolio page
├── 404.html                   # Custom 404 error page
├── 401.html, 403.html, etc.   # Custom HTTP error pages
├── sw.js                      # Service Worker for offline support
├── robots.txt                 # SEO crawler instructions
├── sitemap.xml                # SEO sitemap
├── CNAME                      # Custom domain (suparious.com)
├── README.md                  # Project documentation
├── ROADMAP.md                 # Development roadmap
├── LICENSE                    # MIT License
├── AI_STYLE_GUIDE.md          # Content writing guidelines
├── assets/
│   ├── css/
│   │   ├── main.css           # Core styles, theme variables
│   │   ├── animations.css     # Animation library
│   │   ├── responsive.css     # Mobile optimizations
│   │   └── chat.css           # AI chat widget styles
│   └── js/
│       ├── main.js            # Core functionality
│       ├── theme.js           # Dark/light mode system
│       ├── animations.js      # Scroll and hover effects
│       └── chat.js            # AI chat assistant widget
├── publications/              # Technical articles (root-level subdirectories)
│   ├── corporate-marketing-failures/
│   │   └── index.html
│   ├── nixon-carter-economic-analysis/
│   │   └── index.html
│   ├── debian-audio-guide/
│   │   └── index.html
│   ├── self-hosting-ai-stack/
│   │   └── index.html
│   ├── scifi-tech-predictions/
│   │   └── index.html
│   ├── shell-scripts/
│   │   └── index.html
│   ├── 2012-chevy-volt-highway-guide/
│   │   └── index.html
│   └── automotive-ac-repair-guide/
│       └── index.html
└── backup/                    # Backup of previous version
```

### Directory Roles
- **Root Level**: Main portfolio page, configuration files, publications
- **assets/**: Shared styles, scripts, images (inherited by all pages)
- **publications/**: Individual article directories (each inherits parent assets)
- **backup/**: Archived previous version for reference

---

## 🚀 DEVELOPMENT WORKFLOW

### Local Development
```bash
# Clone repository
git clone git@github.com:suparious/suparious.github.io.git
cd suparious.github.io

# Start local server (choose one)
python3 -m http.server 8000    # Python 3
npx serve                      # Node.js
php -S localhost:8000          # PHP

# Open browser
open http://localhost:8000
```

### Making Changes
```bash
# Edit files
# - index.html for portfolio content
# - assets/css/*.css for styles
# - assets/js/*.js for functionality
# - Create new /article-name/index.html for publications

# Test locally (verify dark/light mode, responsiveness)

# Commit and push
git add .
git commit -m "Description of changes"
git push origin main

# Site automatically deploys to suparious.com within 1-2 minutes
```

### Adding New Publications
1. **Create directory**: `/article-name/`
2. **Create index.html** with structure:
   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <!-- Meta tags -->
       <title>Article Title - Shaun Prince</title>

       <!-- CRITICAL: Inherit parent styles -->
       <link rel="stylesheet" href="../assets/css/main.css">
       <link rel="stylesheet" href="../assets/css/animations.css">
       <link rel="stylesheet" href="../assets/css/responsive.css">

       <!-- CDN resources -->
       <link href="https://fonts.googleapis.com/..." rel="stylesheet">
       <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
   </head>
   <body>
       <!-- Standard navbar (copy from existing article) -->
       <nav id="navbar" class="navbar">...</nav>

       <!-- Article content -->
       <article class="article-content">...</article>

       <!-- Footer -->
       <footer class="footer">...</footer>

       <!-- CRITICAL: Inherit parent scripts -->
       <script src="../assets/js/main.js"></script>
       <script src="../assets/js/theme.js"></script>
   </body>
   </html>
   ```
3. **Update index.html** publications section with link to new article
4. **Update sitemap.xml** for SEO
5. **Test dark/light mode** to ensure theme works

---

## 📋 COMMON TASKS

### View Site Locally
```bash
cd /mnt/c/Users/shaun/repos/suparious.github.io
python3 -m http.server 8000
# Access: http://localhost:8000
```

### Update Portfolio Content
```bash
# Edit index.html sections
# - About: Professional bio
# - Skills: Expertise areas
# - Experience: Work history
# - Projects: Featured work
# - Contact: Contact information

# Test changes locally
# Commit and push to deploy
git add index.html
git commit -m "Update portfolio content"
git push origin main
```

### Update Styles
```bash
# Edit theme colors
vim assets/css/main.css  # Update CSS variables in :root

# Edit animations
vim assets/css/animations.css

# Edit responsive breakpoints
vim assets/css/responsive.css

# Test on multiple screen sizes
# Commit and push
```

### Troubleshooting

**Issue**: Dark/light mode not working in articles
**Solution**: Ensure article includes:
```html
<link rel="stylesheet" href="../assets/css/main.css">
<script src="../assets/js/theme.js"></script>
```

**Issue**: Navbar not showing on article pages
**Solution**: Copy navbar HTML from working article, ensure parent scripts loaded

**Issue**: Particles not showing on hero
**Solution**: Check particles.js CDN loaded, verify `<div id="particles-js">` exists

**Issue**: Custom domain not working
**Solution**: Verify CNAME file contains `suparious.com`, check DNS settings

**Issue**: AI Chat returning 401 errors
**Solution**: The PAM API key needs to be rotated. See [AI Chat API Key Management](#ai-chat-api-key-management) below.

---

## 🤖 AI CHAT API KEY MANAGEMENT

The AI chat widget (`assets/js/chat.js`) uses a PAM-validated API key to connect to the SolidRusT AI platform.

### Configuration
- **API Endpoint**: `https://api.solidrust.ai/v1/chat/completions`
- **Model**: `vllm-primary` (maps to current production model)
- **Auth Header**: `X-API-Key`
- **Key Storage**: Base64 encoded in `chat.js` → `CONFIG.DEFAULT_API_KEY_ENCODED`
- **PAM Account**: `demo-suparious@suparious.com` / `demo123!`
- **Stripe Customer**: `cus_Tq7olARUK3O94m`

### Rotating the API Key

When the chat returns 401 errors, rotate the key via PAM GraphQL:

```bash
# 1. Login to get access token
cat > /tmp/pam-login.json << 'EOF'
{"query":"mutation { login(input: { email: \"demo-suparious@suparious.com\", password: \"demo123!\" }) { accessToken player { id } } }"}
EOF
TOKEN=$(curl -s -X POST 'https://console.solidrust.ai/graphql' \
  -H 'Content-Type: application/json' \
  -d @/tmp/pam-login.json | jq -r '.data.login.accessToken')

# 2. List current keys to get the key ID
cat > /tmp/pam-list.json << 'EOF'
{"query":"query { myApiKeys { id name lastFour } }"}
EOF
curl -s -X POST 'https://console.solidrust.ai/graphql' \
  -H 'Content-Type: application/json' \
  -H "Authorization: Bearer $TOKEN" \
  -d @/tmp/pam-list.json | jq .

# 3. Revoke old key (replace KEY_ID with actual ID from step 2)
cat > /tmp/pam-revoke.json << 'EOF'
{"query":"mutation { revokeApiKey(keyId: \"KEY_ID\") }"}
EOF
curl -s -X POST 'https://console.solidrust.ai/graphql' \
  -H 'Content-Type: application/json' \
  -H "Authorization: Bearer $TOKEN" \
  -d @/tmp/pam-revoke.json

# 4. Create new key
cat > /tmp/pam-create.json << 'EOF'
{"query":"mutation { createApiKey(input: { name: \"Suparious Demo Key\", scopes: [\"inference\", \"embeddings\"], rateLimit: 100 }) { apiKey { id lastFour } secretKey } }"}
EOF
curl -s -X POST 'https://console.solidrust.ai/graphql' \
  -H 'Content-Type: application/json' \
  -H "Authorization: Bearer $TOKEN" \
  -d @/tmp/pam-create.json | jq .

# 5. Base64 encode the new secretKey and update chat.js
echo -n "srt_prod_xxx..." | base64
```

### Validating the Key

```bash
curl "https://console.solidrust.ai/v1/keys/validate?key=YOUR_KEY" | jq .
# Should return: {"valid": true, "customer_id": "cus_Tq7olARUK3O94m", ...}
```

### Testing Chat Completion

```bash
cat > /tmp/chat-test.json << 'EOF'
{"model":"vllm-primary","messages":[{"role":"user","content":"Hello"}],"max_tokens":50}
EOF
curl -s -X POST 'https://api.solidrust.ai/v1/chat/completions' \
  -H 'Content-Type: application/json' \
  -H 'X-API-Key: YOUR_KEY' \
  -d @/tmp/chat-test.json | jq .
```

### Related Issues
- [srt-pam-platform#31](https://github.com/SolidRusT/srt-pam-platform/issues/31) - rotateApiKey mutation not in production
- [srt-pam-platform#32](https://github.com/SolidRusT/srt-pam-platform/issues/32) - GitHub/Gitea migration incomplete

---

## 🎯 USER PREFERENCES (CRITICAL)

**Context**:
- **Background**: 20+ years in Cloud Architecture, AI Engineering, DevOps
- **Purpose**: Professional portfolio and technical publication platform
- **Philosophy**: Clean, modern, accessible design with no cruft

**Solutions Must Be**:
- ✅ Complete and immediately functional
- ✅ Cross-browser compatible (Chrome, Firefox, Safari)
- ✅ Mobile-responsive and accessible
- ✅ SEO optimized with proper meta tags
- ✅ Performance optimized (< 3s load time)
- ❌ NO JavaScript framework dependencies
- ❌ NO build process requirements (keep it static)
- ❌ NO workarounds or temp fixes

**Quality Standards**:
- Works on first deploy
- Maintains theme inheritance across all articles
- Validates with Lighthouse (90+ scores)
- Semantic HTML and accessibility compliant
- Git = source of truth

**Workflow Preferences**:
- **Direct Editing**: Edit HTML/CSS/JS directly, no build step
- **Browser Testing**: Test in multiple browsers and screen sizes
- **Progressive Enhancement**: Core functionality works without JavaScript

---

## 💡 KEY DECISIONS

### Why Static Site (No Build Process)?
**Decision**: Keep as pure HTML/CSS/JS without build tooling
**Rationale**:
- Simplicity and maintainability
- No dependency management or version conflicts
- Instant deployment (push to GitHub = live)
- Easy for future developers to understand
- GitHub Pages native support

### Why GitHub Pages?
**Decision**: Use GitHub Pages for hosting
**Rationale**:
- Free hosting with SSL
- Automatic deployment on push
- Global CDN for fast delivery
- Custom domain support
- 100% uptime SLA

### Why Vanilla JavaScript?
**Decision**: No React, Vue, or framework dependencies
**Rationale**:
- Reduces bundle size and improves performance
- Eliminates framework lock-in
- Easier maintenance and updates
- No build tooling complexity
- Modern browsers support ES6+ natively

### Why Article Inheritance Pattern?
**Decision**: Articles inherit parent site styles via relative paths
**Rationale**:
- Consistent theming across all content
- Single source of truth for styles
- Easy global updates (change parent, all articles update)
- Reduces code duplication
- Dark/light mode works automatically

---

## 🔍 VALIDATION

### Post-Update Checklist
```bash
# Test locally
python3 -m http.server 8000
# Verify:
# - [ ] Dark/light mode toggle works (Ctrl+Shift+L)
# - [ ] All sections load correctly
# - [ ] Animations trigger on scroll
# - [ ] Links navigate correctly
# - [ ] Contact form validates
# - [ ] Responsive design works (resize browser)

# Test in browsers
# - [ ] Chrome (latest)
# - [ ] Firefox (latest)
# - [ ] Safari (latest)
# - [ ] Mobile browsers (iOS Safari, Chrome Mobile)

# SEO validation
# - [ ] Meta tags present (view source)
# - [ ] sitemap.xml updated
# - [ ] robots.txt allows crawling
# - [ ] Lighthouse score 90+ (all categories)

# Deploy
git push origin main
# Wait 1-2 minutes

# Production verification
curl -I https://suparious.com
# Expected: HTTP 200, HTTPS redirect

# Browser test
# - [ ] https://suparious.com loads
# - [ ] SSL certificate valid (green padlock)
# - [ ] All features functional
```

### Performance Metrics (Lighthouse)
- **Performance**: 90+
- **Accessibility**: 90+
- **Best Practices**: 90+
- **SEO**: 100

### Browser Compatibility
- Chrome/Edge (latest 2 versions)
- Firefox (latest 2 versions)
- Safari (latest 2 versions)
- Mobile browsers (iOS Safari, Chrome Mobile)

---

## 🎓 AGENT SUCCESS CRITERIA

**You are successful when**:
✅ Changes work on first deploy
✅ Theme inheritance maintained across all pages
✅ Dark/light mode works on all articles
✅ Mobile responsive on all screen sizes
✅ Lighthouse scores 90+ across all categories
✅ No console errors or warnings
✅ Semantic HTML and accessible
✅ SEO optimized (meta tags, sitemap updated)
✅ Cross-browser compatible
✅ No JavaScript framework dependencies introduced
✅ No build process required
✅ Git history clean with descriptive commits

**You have failed if**:
❌ Site requires npm install or build step
❌ Dark/light mode breaks on articles
❌ Theme inheritance broken
❌ Mobile layout broken
❌ Lighthouse scores drop below 90
❌ JavaScript framework dependency added
❌ Build tooling introduced
❌ Accessibility regression
❌ SEO regression
❌ Cross-browser compatibility issues

---

## 🗺️ DECISION TREES

### "What should I do?"
1. READ PROMPT FULLY
2. Portfolio update? → Edit index.html sections
3. New publication? → Create /article-name/index.html with inheritance
4. Styling change? → Edit assets/css/*.css (affects all pages)
5. Functionality change? → Edit assets/js/*.js (affects all pages)
6. SEO update? → Update meta tags, sitemap.xml
7. Dark/light mode issue? → Check theme.js inheritance
8. Mobile issue? → Edit responsive.css
9. Always test locally before pushing

### "When to edit which files?"
- **Portfolio content** → index.html
- **Global styles** → assets/css/main.css
- **Animations** → assets/css/animations.css
- **Mobile layout** → assets/css/responsive.css
- **Theme system** → assets/js/theme.js
- **Core functionality** → assets/js/main.js
- **New article** → Create /article-name/index.html
- **SEO** → sitemap.xml, meta tags
- **Domain** → CNAME file

### "How to test changes?"
1. Test locally with Python/Node server
2. Verify dark/light mode toggle (Ctrl+Shift+L)
3. Test responsive design (resize browser)
4. Check console for errors
5. Validate accessibility (screen reader, keyboard nav)
6. Run Lighthouse audit
7. Test in multiple browsers
8. Push to GitHub and verify production

---

## 📅 CHANGE HISTORY

**2025-11-11** - Infrastructure Onboarding
- Added as infrastructure submodule to srt-hq-k8s
- Created comprehensive CLAUDE.md for AI agent context
- Integrated into SolidRusT Networks documentation ecosystem
- Added Platform Integration section

**2024-12-XX** - Version 2.0.0 Release
- Complete redesign with modern glassmorphism UI
- Added dark/light mode with automatic detection
- Integrated Particles.js interactive background
- Added 8+ technical publication articles
- Service Worker for offline support
- SEO optimization with sitemap

---

## 🎫 ISSUE TRACKING

**Platform**: GitHub
**Issues**: https://github.com/suparious/suparious.github.io/issues

Backlog, bugs, and enhancements are tracked as GitHub issues.
This CLAUDE.md contains domain knowledge and patterns, not task tracking.

**To check current work**: Review open issues
**To add new work**: Create an issue with acceptance criteria

**Note**: ROADMAP.md contains historical documentation of completed work (v2.0, v2.1) and future enhancement ideas. Active work items have been migrated to GitHub issues.

---

**Last Updated**: 2026-01-22
**Status**: Production - Active Portfolio Site
**Maintained By**: Shaun Prince
**Used With**: Claude Code

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/suparious)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/suparious)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
