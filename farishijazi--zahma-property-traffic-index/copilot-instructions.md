## zahma-property-traffic-index

> *   **Frontend Framework**: React 18

# Tech Context: Google Maps Routing Distance Matrix Calculator

## Technologies Used
*   **Frontend Framework**: React 18
*   **Language**: TypeScript
*   **Build Tool**: Vite
*   **State Management**: Zustand
*   **Styling**: TailwindCSS
*   **Data Visualization**: Chart.js
*   **Mapping & Routing**: Google Maps API (specifically Distance Matrix, Geocoding, Places, Routes APIs).
*   **Google Maps Wrapper**: `@googlemaps/react-wrapper` is used for loading the Google Maps JavaScript API. The API is loaded once in `App.tsx` with all necessary libraries (`places`, `geocoding`, `routes`) to prevent conflicts.

## Development Setup
Node.js and npm/yarn are required. The Vite development server is used for local execution. A Google Maps API key, hardcoded in `src/store/index.ts`, must be configured with the following APIs enabled in the Google Cloud Console:
*   Maps JavaScript API
*   Geocoding API
*   Places API
*   Distance Matrix API
*   Routes API (or Directions API, depending on the specific version and features used by the JS SDK - current setup uses 'routes')

## Technical Constraints
*   **API Quotas and Billing**: Usage of the Google Maps API is subject to quotas and billing. Efficient API usage is important.
*   **Ad Blocker Compatibility**: Potential conflicts with ad blockers intercepting requests for certain UI assets (e.g., Lucide icons) have been noted.

## Dependencies
*   `react`, `react-dom` (@18.3.1)
*   `@types/react` (@18.3.5), `@types/react-dom` (@18.3.0)
*   `typescript` (@5.5.3)
*   `vite` (@5.4.2)
*   `zustand` (@4.4.7)
*   `tailwindcss` (@3.4.1)
*   `autoprefixer` (@10.4.18)
*   `postcss` (@8.4.35)
*   `chart.js` (@4.4.0), `react-chartjs-2` (@5.2.0)
*   `@googlemaps/js-api-loader` (@1.16.2)
*   `@googlemaps/react-wrapper` (@1.1.35) 
*   `@types/google.maps` (@3.54.7)
*   `lucide-react` (@0.344.0)
*   `eslint` (@9.9.1), `eslint-plugin-react-hooks` (@5.1.0-rc.0), `eslint-plugin-react-refresh` (@0.4.11), `@eslint/js` (@9.9.1), `typescript-eslint` (@8.3.0), `globals` (@15.9.0)
*   `@vitejs/plugin-react` (@4.3.1)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FarisHijazi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/FarisHijazi)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
