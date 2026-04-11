## gps2photos

> This project consists of three related WordPress plugins designed to add and display geolocation data for images managed by the NextGEN Gallery plugin. The plugins are:

# Project Overview

This project consists of three related WordPress plugins designed to add and display geolocation data for images managed by the NextGEN Gallery plugin. The plugins are:

*   **`nextgen-gallery-geo`**: The core free plugin for displaying maps with geolocated photos.
*   **`ngg-geo2-maps-plus`**: The premium version of `nextgen-gallery-geo` with extended features.
*   **`gps-2-photos`**: A helper plugin for adding or editing GPS data on photos.

The plugins use the Azure Maps API for map tiles and geocoding, with an option to use MapQuest for geocoding as well.

## Plugin Details

### `nextgen-gallery-geo` (Free version)

*   **Purpose**: Displays maps with photos, galleries, or albums from NextGEN Gallery based on their EXIF GPS data. If no GPS data is present, it can attempt to geocode the gallery title.
*   **Key Features**:
    *   Displays photos as pushpins or thumbnails on a map.
    *   Photo preview in a lightbox (Fancybox, Fancybox 3, Slimbox 2) or an infobox.
    *   "Worldmap" mode to show an overview of multiple galleries/albums.
    *   "Route mode" to display a path from a GPX, KML, or other geospatial file, along with photos taken along the route.
    *   Customization of map appearance, including map type, dimensions, and controls.
    *   Shortcode `[geo2]` for embedding maps.

### `ngg-geo2-maps-plus` (Premium version)

*   **Purpose**: An extended version of `nextgen-gallery-geo` with more advanced features and customization options.
*   **Key Features (in addition to the free version)**:
    *   More advanced customization for lightboxes, including side-panel captions with embedded maps.
    *   Video support in lightboxes.
    *   More thumbnail and pushpin styling options.
    *   Override NextGEN Gallery's default lightbox.
    *   Freemius integration for licensing and sales.

### `gps-2-photos`

*   **Purpose**: An add-on for NextGEN Gallery that allows users to add or edit GPS coordinates in a photo's EXIF data by picking a location on a map.
*   **Key Features**:
    *   Integrates with the WordPress Media Library.
    *   Uses Azure Maps for the map interface.
    *   Allows setting GPS coordinates for photos that don't have them.

## Development Conventions

*   **Language**: PHP, following WordPress coding standards.
*   **Framework**: WordPress plugin architecture.
*   **Dependencies**:
    *   NextGEN Gallery plugin.
    *   Azure Maps API (requires an API key).
    *   MapQuest API (optional, for geocoding).
*   **User-facing integration**: Primarily through the `[geo2]` shortcode and an administration panel under the NextGEN Gallery menu in WordPress.

## Building and Running

These are WordPress plugins and are not "built" in the traditional sense of compiling code.

1.  **Installation**:
    *   Install the NextGEN Gallery plugin.
    *   Install the desired Geo2 Maps plugins (`nextgen-gallery-geo`, `ngg-geo2-maps-plus`, `gps-2-photos`) through the WordPress plugin installer or by uploading the plugin files to `wp-content/plugins`.
2.  **Activation**: Activate the plugins in the WordPress admin panel.
3.  **Configuration**:
    *   Obtain an API key from Microsoft Azure for Azure Maps.
    *   Enter the API key in the Geo2 Maps settings page (NextGEN Gallery -> Geo2 Maps).
    *   Optionally, obtain and enter a MapQuest API key for alternative geocoding.
4.  **Usage**:
    *   Use the `[geo2]` shortcode in posts or pages to display maps.
    *   Configure the plugin's settings in the admin panel to control default behavior.
    *   Use the `gps-2-photos` interface to add GPS data to images.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/2pablo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/2pablo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
