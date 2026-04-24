## sloview

> Non-obvious debugging guidelines for the SLOView project


# Debug Rules (Non-Obvious Only)

## Map-Specific Debugging
- Map tile caching issues may cause silent failures in offline viewing
- Check browser console for Leaflet.js errors when map doesn't load
- Verify local storage quota when implementing tile caching
- Test map drag and zoom functionality manually (cannot be fully automated)
- Cross-browser compatibility issues may occur with Leaflet.js implementation

## Deployment-Specific Debugging
- Frontend deployment issues specifically relate to Google Cloud Storage configuration
- Backend deployment issues specifically relate to containerization for Google Cloud Run
- Check Google Cloud Run logs for backend container startup issues
- Verify static file serving configuration for Google Cloud Storage

## Network and Performance Issues
- Network connectivity issues may not be immediately apparent with local tile caching
- Local storage limitations may affect map tile caching performance
- Check for CORS issues when frontend calls backend APIs
- Monitor memory usage during map tile loading and caching

## Cursor-Specific Debug Tips
- Use browser dev tools to inspect network requests for tile loading
- Check both frontend and backend logs when debugging API issues
- Test offline functionality by disabling network in browser dev tools
- Verify environment variables are properly set for Google Cloud deployment

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MathewTomberlin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
