## chromeproxy

> This is a Chrome Extension (Manifest V3) for proxy management with the following structure:

# Chrome Proxy Manager - Cursor Rules

## Project Context
This is a Chrome Extension (Manifest V3) for proxy management with the following structure:
- `manifest.json` - Extension manifest
- `background.js` - Service worker for proxy management
- `popup.html/css/js` - Extension popup interface
- `icons/` - Extension icons
- `.github/workflows/` - GitHub Actions for CI/CD

## Language & Localization Rules
- **Code comments**: Always in English
- **UI text**: Russian by default, with English localization support via language switcher
- **Commit messages**: English, using conventional commits format (feat:, fix:, chore:, etc.)
- **Variable/function names**: English, camelCase
- **Documentation**: English for technical docs, Russian for user-facing content

## Code Style Guidelines
- Use modern JavaScript (ES6+)
- Prefer async/await over promises
- Use descriptive variable and function names
- Add error handling for all async operations
- Follow Chrome Extension Manifest V3 best practices

## Key Features to Maintain
- Proxy server management (add/edit/delete servers)
- HTTP/HTTPS protocol support
- DNS through proxy option (optional)
- Exclusion lists per server
- Automatic update checking via GitHub API
- Language switcher (Russian/English)
- Repository link in header (🏠 icon)
- Proxy diagnostics functionality (🔍 button)

## File-Specific Rules

### manifest.json
- Always use Manifest V3 format
- Update version following semantic versioning
- Maintain required permissions: proxy, storage, notifications

### background.js
- Use chrome.proxy.settings API for proxy configuration
- Implement proper error handling and logging
- Save state to chrome.storage.sync
- Handle DNS proxy settings correctly with fallbackToDirect option

### popup.js
- Implement proper message passing with background script
- Handle UI state updates immediately for better UX
- Support both Russian and English languages
- Include proper error handling with user-friendly messages

### popup.html/css
- Maintain modern gradient design
- Support responsive layout
- Include language selector and repository link in header
- Use semantic HTML structure

## Development Workflow
1. **Feature Development**: 
   - Update relevant files (popup.html, popup.css, popup.js, background.js)
   - Test functionality thoroughly
   - Ensure both language versions work

2. **Version Updates**:
   - Update `manifest.json` version
   - Add entry to `CHANGELOG.md` (in English)
   - Create git tag in format `vX.Y.Z`

3. **Release Process**:
   - Git tag triggers GitHub Actions
   - Changelog content auto-extracted for release notes
   - Release notes generated in English

## Common Patterns

### Adding New Features
- Update popup UI (HTML/CSS)
- Add event handlers in popup.js
- Implement backend logic in background.js
- Add message passing between popup and background
- Update language translations for both Russian and English

### Fixing Proxy Issues
- Check `applyProxySettings()` function in background.js
- Ensure proxy config includes all protocols (HTTP/HTTPS/FTP)
- Verify DNS settings and fallbackToDirect option
- Test with diagnostics functionality

### UI Updates
- Maintain gradient design theme
- Update both language versions
- Ensure responsive layout
- Test language switcher functionality

## Testing Checklist
- [ ] Proxy connection works for HTTP/HTTPS traffic
- [ ] Server CRUD operations function properly
- [ ] Language switching works correctly
- [ ] Update checking mechanism works
- [ ] Exclusion lists are respected
- [ ] DNS proxy option functions as expected
- [ ] Diagnostics button shows correct information

## Error Handling
- Always wrap async operations in try-catch
- Provide user-friendly error messages in current language
- Log detailed errors to console for debugging
- Gracefully handle network failures and API errors

## Performance Considerations
- Minimize background script activity
- Use efficient storage operations
- Implement proper cleanup for timeouts and intervals
- Avoid unnecessary DOM manipulations

## Security Best Practices
- Validate all user inputs
- Sanitize data before storage
- Use secure communication with GitHub API
- Follow Chrome Extension security guidelines 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kalkalch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
