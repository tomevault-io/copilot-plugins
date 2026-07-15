## cosckoya

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a **GitHub profile repository** - the README.md is automatically displayed at https://github.com/cosckoya.

**Structure:**
```
cosckoya/
├── README.md       # Profile page (auto-displayed on GitHub)
├── img/           # Profile images (technical/fantasy aesthetic)
└── .gitignore     # Comprehensive (includes Claude Code files)
```

**Profile Identity:** DevOps & Cloud Architect, Platform Engineer, AI Infrastructure Specialist focused on multi-cloud AI infrastructure, MCP servers, multi-agent systems, and GitOps.

## Key Requirements

### Critical GitHub Profile Rules
1. Repository name MUST match username exactly (`cosckoya`)
2. Repository MUST be public
3. README.md MUST be in root directory
4. README.md MUST not be empty

### Profile Style & Content
- **Sections:** About, Core Expertise, Tech Stack, GitHub Stats, Featured Projects, Beyond Code
- **Visual Elements:** Animated typing header, stat widgets, centered layouts, tables, badges
- **Theme:** Transparent background (`theme=transparent&hide_border=true`) for all widgets
- **Tone:** Professional + personal (DevOps expertise + fantasy/gaming interests)

## Pre-Commit Validation Checklist

**Before committing changes to README.md, verify:**

1. **No Merge Conflict Markers**
   ```bash
   grep -n "<<<<<<< HEAD\|=======\|>>>>>>>" README.md
   # Should return nothing
   ```

2. **No Placeholder Content**
   ```bash
   grep -E "(your-.*-repo|your\.email|your-portfolio\.com|lorem ipsum)" README.md
   # Should return nothing
   ```

3. **Widget URLs Are Valid**
   - Test each widget URL in browser before committing
   - Ensure username parameter is correct (`username=cosckoya`)
   - Verify theme parameters work (`theme=transparent&hide_border=true`)

4. **Image Links Work**
   ```bash
   ls -1 img/*.png img/*.jpg 2>/dev/null
   # Verify all images referenced in README exist
   ```

5. **HTML Comment Markers Intact**
   - Do not remove `<!--START_SECTION:activity-->` / `<!--END_SECTION:activity-->`
   - These enable GitHub Actions auto-updates

6. **Markdown Syntax Valid**
   - Check for unclosed HTML tags (`<div>` must have `</div>`)
   - Verify table formatting
   - Test badge image URLs

## Dynamic Widgets Reference

The profile uses auto-updating stat widgets:

```markdown
# GitHub Stats
https://github-readme-stats.vercel.app/api?username=USERNAME&params

# Streak Stats
https://github-readme-streak-stats.herokuapp.com/?user=USERNAME&params

# Activity Graph
https://github-readme-activity-graph.vercel.app/graph?username=USERNAME&params

# Profile Trophy
https://github-profile-trophy.vercel.app/?username=USERNAME&params

# Typing Animation
https://readme-typing-svg.herokuapp.com?params
```

**Common Parameters:**
- `theme=transparent` - Transparent background
- `hide_border=true` - Remove widget borders
- `count_private=true` - Include private repos in stats
- `show_icons=true` - Display icons in stats

## Editing Guidelines

**When updating README.md:**

1. **Maintain Structure** - Keep section hierarchy intact
2. **Preserve Automation** - Never remove HTML comment markers for GitHub Actions
3. **Test Widgets** - Verify all external URLs render before committing
4. **Consistent Theme** - All widgets use `transparent` theme with `hide_border=true`
5. **Real Data Only** - No placeholder text or fake repository links
6. **Mobile Friendly** - Test layout responsiveness (especially tables)
7. **Professional Balance** - Combine technical expertise with authentic personality

**When adding images to `/img/`:**
- Maintain technical/fantasy aesthetic
- Use PNG for icons/logos, JPG for photos
- Reference in README as `img/filename.ext`

## Common Issues & Troubleshooting

### Profile Not Displaying
**Cause:** Repository is private, README is empty, or name doesn't match username
**Fix:** Ensure repo is public, README has content, repo name is exactly `cosckoya`

### Widgets Not Loading
**Cause:** Service down, incorrect username, or invalid parameters
**Fix:** Test URL directly in browser, verify `username=cosckoya`, check service status

### Merge Conflict Committed
**Cause:** Conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`) were committed
**Fix:** Remove markers, choose one version, commit clean file

### Broken Image Links
**Cause:** Image file moved, renamed, or doesn't exist
**Fix:** Verify file exists in `img/` directory with correct name/extension

### Stale Activity Section
**Cause:** GitHub Actions workflow not running or misconfigured
**Fix:** Check `.github/workflows/` for activity update workflow, verify schedule

## GitHub Actions Automation

**Potential Workflows** (add to `.github/workflows/` to automate):

```yaml
# Update Recent Activity
# Uses: gautamkrishnar/blog-post-workflow
# Updates: <!--START_SECTION:activity--> section

# Blog Post Integration
# Uses: gautamkrishnar/blog-post-workflow (RSS feed)
# Updates: <!-- BLOG-POST-LIST:START --> section

# WakaTime Coding Stats
# Uses: anmol098/waka-readme-stats
# Adds: Weekly coding activity breakdown
```

## Quick Reference

**Featured Repos:** Currently showing `.dotfiles` (7⭐) and `toolbox`
**Contact Links:** LinkedIn, GitHub, GitHub Pages (cosckoya.github.io)
**Theme Colors:** Blue (#58A6FF) for primary accent
**Widget Services:** Vercel Apps (stats, typing), Herokuapp (streak)

**Useful Commands:**
```bash
# Check what's tracked
git ls-files

# Verify .gitignore patterns
git check-ignore -v .claude/

# Preview README changes
# Use GitHub's markdown preview or push to test branch

# List public repos with stars
gh repo list cosckoya --source --json name,stargazerCount,isPrivate
```

## 2026 Profile Best Practices Summary

**Essential Elements:**
- ✅ Dynamic auto-updating widgets (not static bio)
- ✅ Clear professional identity statement
- ✅ Real projects and metrics (not self-assessed skills)
- ✅ GitHub Actions automation (not manual updates)
- ✅ Responsive design (dark/light mode compatible)

**Avoid:**
- ❌ Wall of badges without context
- ❌ Every technology ever touched
- ❌ Broken widget links or placeholders
- ❌ Excessive GIFs/animations
- ❌ Generic template content
- ❌ Manual sections that become stale

**Balance:** Technical credibility + authentic personal voice + strategic CTAs

---

**Last Updated:** 2026-02-12
**Profile Status:** ✅ Active, ⚠️ Featured repos set, ℹ️ Activity section ready for automation

---
> Source: [cosckoya/cosckoya](https://github.com/cosckoya/cosckoya) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
