## github-folders

> This document contains development-specific information for working on the GitHub Actions Folder Organizer extension.

# Developer Guide

This document contains development-specific information for working on the GitHub Actions Folder Organizer extension.

## Project Structure

```
github-actions-folders/
├── manifest.json                 # Extension manifest (Manifest V3)
├── src/
│   ├── content/
│   │   ├── constants/
│   │   │   ├── selectors.js     # DOM selectors and CSS class names
│   │   │   └── config.js        # Extension configuration constants
│   │   ├── core/
│   │   │   ├── repository-info.js    # URL parsing and repo detection
│   │   │   └── workflow-organizer.js # Workflow grouping logic
│   │   ├── services/
│   │   │   ├── token-service.js        # GitHub token storage and management
│   │   │   ├── repo-detector.js        # Public/private repository detection
│   │   │   ├── messaging-service.js    # Service worker communication
│   │   │   ├── permissions-service.js  # Permission checking (API + HTML fallback)
│   │   │   └── storage-service.js      # Chrome storage abstraction
│   │   ├── ui/
│   │   │   ├── dom-selector.js            # DOM element finding
│   │   │   ├── css-injector.js            # CSS injection/removal
│   │   │   ├── loading-state.js           # Loading skeleton UI
│   │   │   ├── folder-renderer.js         # Folder UI generation
│   │   │   ├── toggle-button.js           # Toggle and config buttons
│   │   │   ├── notification-banner.js     # Token feature notification (private repos)
│   │   │   └── notification-banner.css    # Notification styling
│   │   ├── main.js              # Main entry point and orchestration
│   │   └── content.css          # Folder styles (Primer CSS variables)
│   ├── background/
│   │   └── service-worker.js    # Background service worker - caching, API, token auth
│   ├── popup/
│   │   ├── popup.html           # Extension popup UI
│   │   ├── popup.js             # Popup logic and token status
│   │   └── popup.css            # Popup styles
│   └── options/
│       ├── options.html         # Options page (cache + token management)
│       ├── options.js           # Options page logic
│       └── options.css          # Options page styles
└── icons/                        # Extension icons
```

## Technologies

- **Manifest V3**: Latest Chrome extension standard
- **Vanilla JavaScript**: No frameworks - lightweight and fast
- **Primer CSS Variables**: GitHub's design system for native look and feel
- **Flexible Authentication**:
  - Optional GitHub API token for better performance and private repos
  - Falls back to session-based auth when no token is configured
  - Smart detection: uses best available method automatically

## Development Setup

1. Clone the repository
2. Make changes to the code
3. Navigate to `chrome://extensions/`
4. Enable "Developer mode" (top right)
5. Click "Load unpacked" and select the project directory
6. Make changes and click the refresh icon on the extension card to reload

## Architecture

### Modular Design

The extension follows SOLID principles with a modular architecture:

**Content Scripts** (loaded in order per manifest.json):
1. **Constants** - Configuration and selectors
2. **Core** - Business logic (URL parsing, workflow organization)
3. **Services** - External interactions (storage, messaging, permissions)
4. **UI** - DOM manipulation and rendering
5. **Main** - Entry point and orchestration

### Content Script (`main.js`)

- Entry point that orchestrates all modules
- Runs on GitHub Actions pages
- Detects URL changes (GitHub is a SPA)
- Coordinates initialization workflow
- Handles cleanup on navigation
- Manages enable/disable state per repository

### Service Worker (`service-worker.js`)

- Coordinates GitHub API requests (workflows endpoint)
- Implements caching layer for config files (5-minute cache)
- Handles chrome.storage management
- Provides messaging interface to content scripts
- Tries multiple default branches (main, master) for config files

### Module Responsibilities

**Constants**: Configuration values, DOM selectors, CSS class names
**Core**: Repository info parsing, workflow-to-folder grouping logic
**Services**: Chrome API abstractions (storage, messaging), permission checks
**UI**: DOM selection, CSS injection, loading states, folder rendering

### Config Format

The extension looks for `.github/actions-folders.json`:

```json
{
  "folders": [
    {
      "name": "Folder Name",
      "workflows": ["workflow1.yml", "workflow2.yml"]
    }
  ]
}
```

- Workflow filenames only (no paths)
- Uncategorized workflows automatically grouped
- Case-sensitive matching

## Testing

### Manual Testing

1. Make changes
2. Reload extension at `chrome://extensions/`
3. Navigate to GitHub Actions page
4. Check browser console for `[GitHub Actions Folders]` logs

### Test Repository

Use [AntoineGagnon/miryoku_zmk](https://github.com/AntoineGagnon/miryoku_zmk) for testing:
- Has a working config file
- Multiple workflows for folder testing
- Public repository (no auth issues)

## Debugging

### Content Script Logs

- Open DevTools on any GitHub Actions page
- Look for `[GitHub Actions Folders]` prefixed messages
- Shows config loading, folder creation, and errors

### Service Worker Logs

- Go to `chrome://extensions/`
- Find the extension card
- Click "service worker" link
- View background script logs and errors

### Common Issues

**Config not loading:**
- Check network tab for 404s on `raw.githubusercontent.com`
- Verify file path is exactly `.github/actions-folders.json`
- Ensure file is on default branch (`main`/`master`)

**Folders not appearing:**
- Verify GitHub's workflow sidebar is present
- Check console for DOM manipulation errors
- Ensure workflow filenames match exactly (case-sensitive)

**Styles not matching:**
- Primer CSS variables might have changed
- Check GitHub's source for updated variable names
- Test in both light and dark mode

## Performance Considerations

### Caching Strategy

- Configs cached per repository
- Cache expires based on user preference (options page)
- Service worker coordinates all cache operations
- Avoids rate limiting by GitHub API

### DOM Manipulation

- Waits for GitHub's sidebar to load
- Uses MutationObserver for SPA navigation
- Minimal DOM queries (cached selectors)
- CSS-based animations (no JavaScript animations)

## Extension Permissions

**Why each permission is needed:**

- `github.com`: Detect Actions pages and inject content script
- `raw.githubusercontent.com`: Fetch config files from repositories
- `api.github.com`: Fetch workflow data via GitHub REST API
- `storage`: Cache configs and folder states locally for performance

**Optional GitHub token:**
- Token is completely optional - extension works without it for public repos
- Token stored in `chrome.storage.sync` (encrypted by Chrome)
- Token only used for GitHub API requests (api.github.com, raw.githubusercontent.com)
- Provides better rate limits (5,000/hour vs 60/hour) and reliable private repo access

## Publishing Checklist

- [ ] Update version in `manifest.json`
- [ ] Update CHANGELOG in README
- [ ] Test in clean browser profile
- [ ] Test with public and private repos
- [ ] Verify all permissions are justified
- [ ] Create promotional images (1280x800, 640x400, 440x280, 128x128)
- [ ] Write Chrome Web Store description
- [ ] Test on different screen sizes
- [ ] Verify dark mode support

## Code Style

- **SOLID Principles**: Each module has a single responsibility
- **Dependency Inversion**: Modules depend on abstractions, not implementations
- **Modular Architecture**: Features separated into logical modules
- Use `const` for all variables unless mutation is required
- Prefer functional patterns over classes
- Keep functions small and single-purpose
- Comment module purpose and complex logic
- Use descriptive variable names
- Follow existing code formatting

## Key Features

### Repository-Specific Settings
- **Per-repo enable/disable**: Toggle extension on/off for each repository
- **Folder state persistence**: Remembers which folders are expanded/collapsed
- **Settings survive navigation**: State persists across page reloads

### Smart Fallbacks
- **Intelligent auth**: Token (if configured) → Session-based → DOM extraction
- **Multi-branch config**: Tries both `main` and `master` branches for config files
- **Graceful degradation**: Shows original GitHub UI if config is missing or invalid
- **Rate limit awareness**: Tracks API rate limits, shows warnings when approaching limit

### Permission Detection
- **Write access checking**: Shows "Create Config" button only for users with write access
- **Multi-method detection**: API endpoint (with token) → HTML endpoint → DOM fallback
- **Flexible auth**: Supports both optional API tokens and session-based authentication

### Performance
- **5-minute config cache**: Reduces API calls and improves load times
- **Early loading state**: Shows skeleton UI immediately to prevent layout shift
- **CSS-based hiding**: Prevents flash of unstyled content during initialization
- **Better rate limits with token**: 5,000/hour with token vs 60/hour without

## Future Enhancements

Potential features to consider:

- Drag-and-drop folder reordering
- Workflow search/filter
- Custom folder icons/colors
- Export/import folder configurations
- Workflow status indicators in folders
- Keyboard shortcuts for folder navigation
- Multi-repository config sharing

## Contributing

When contributing:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly (see Testing section)
5. Submit a pull request with clear description

## Resources

- [Chrome Extension Manifest V3 Docs](https://developer.chrome.com/docs/extensions/mv3/)
- [GitHub Primer CSS](https://primer.style/css/)
- [GitHub REST API](https://docs.github.com/en/rest)
- [Chrome Storage API](https://developer.chrome.com/docs/extensions/reference/storage/)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kurigi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Kurigi)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
