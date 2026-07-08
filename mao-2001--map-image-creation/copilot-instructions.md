## map-image-creation

> A desktop GIS application built with Qt6 and Leaflet.js, integrating China's Tianditu (天地图) map service with GeoTIFF and Shapefile visualization capabilities.

# AGENTS.md

## Project: MAP_v1.3 - 地理信息系统 (GIS)

A desktop GIS application built with Qt6 and Leaflet.js, integrating China's Tianditu (天地图) map service with GeoTIFF and Shapefile visualization capabilities.

## Tech Stack

- **Backend**: C++17, Qt6 (Widgets, WebEngineWidgets, WebChannel)
- **Frontend**: Leaflet.js (v1.9.4), Tianditu WMTS API (EPSG:3857)
- **GIS Library**: GDAL 3.12.3 (GeoTIFF RasterIO, OGR Shapefile, coordinate transforms)
- **Build**: CMake (>=3.16)

## Key Features

### 1. Base Map Display
- Tianditu vector basemap (`vec_w`) with annotation overlay (`cva_w`), both EPSG:3857
- Map centered on Xi'an (34.34, 108.94)
- Custom right-click context menu (only "Reload" via QWebChannel)
- Interactive zoom and pan controls

### 2. GeoTIFF Loading & Visualization
- **File → Open TIFF**: Load single-band GeoTIFF with CRS-aware bounds
- **7 color schemes**: Default, Terrain, Cool-Warm, Viridis, Plasma, Spectral, RdBu
- Real-time preview in color map selection dialog
- Automatic min/max with NoData handling
- Debug PNG output to `/tmp/debug_tiff.png`

### 3. Shapefile Loading & Visualization
- **File → Open SHP**: Load Shapefile with custom color picker
- GDAL OGR → GeoJSON conversion, rendered via Leaflet
- Supports Polygon, LineString, Point geometries

### 4. Single File Downsampling
- **Tool → 单文件降采样**: Resample TIFF to target resolution via `gdalwarp -tr`
- Configurable method: bilinear / near / cubic / average
- Output EPSG:3857 for pixel-perfect Leaflet alignment

### 5. Batch TIFF Import & Downsampling
- **File → Open TIFFs (批量导入)**: Import multiple TIFFs
- **Tool → 批量降采样**: Batch resample → merge into multi-band EPSG:3857 TIFF

### 6. Multi-band TIFF Playback
- **File → 打开多波段 TIFF**: 10fps animation via `L.imageOverlay.setUrl()`
- CRS auto-detection (4326 / 3857) with WGS84 bounds transform
- NoData regions rendered as white

### 7. Export Current View
- **Tool → 导出图片 (PNG/GIF)**: Auto-adapts:
  - Single TIFF loaded → export PNG with current colormap
  - Animation playing → export GIF via ImageMagick

### 8. API Key Management
- First launch prompts for Tianditu key (persisted via QSettings)

## Build Instructions

### Prerequisites
- Qt6 (Widgets, WebEngineWidgets, WebChannel)
- GDAL >= 3.0 with dev headers
- CMake >= 3.16, C++17 compiler
- ImageMagick (for GIF export)

### Build (Linux)
```bash
cd /home/mao/Documents/Map
rm -rf build && mkdir build && cd build
cmake ..
make -j$(nproc)
```

### Build (Windows)
```bash
# Prerequisites: VS2022, Qt 6.8 (msvc2022_64), OSGeo4W GDAL, ImageMagick
# Edit paths in package_windows.bat first, then run:
package_windows.bat
```

### Run
```bash
cd /home/mao/Documents/Map/build
./MAP_v1.3
```

## File Structure

```
Map/
├── src/
│   ├── main.cpp                          # Application entry point
│   ├── mainwindow.h/cpp                  # UI, menus, export state machine
│   ├── dialogs/
│   │   ├── keydialog.h/cpp               # Tianditu API key input
│   │   ├── colormapdialog.h/cpp          # Color map selection + preview
│   │   └── resampledialog.h/cpp          # Resolution/method input
│   ├── core/
│   │   ├── geo_tiff_loader.h/cpp         # GeoTIFF reading + CRS-aware bounds
│   │   ├── shapefile_loader.h/cpp        # SHP → GeoJSON conversion
│   │   ├── raster_renderer.h/cpp         # Colormap definitions + frame rendering
│   │   └── tiff_utils.h/cpp             # Multi-band merge + CRS utils + gdalwarp wrapper
│   └── export/
│       └── gif_exporter.h/cpp            # Multi-band TIFF → GIF via ImageMagick
├── map.html                              # Leaflet.js frontend (QWebChannel bridge)
├── CMakeLists.txt
├── AGENTS.md
└── build/
```

## Module Responsibilities

| Module | Role | Depends On |
|--------|------|-----------|
| `mainwindow` | Menu setup, user interaction, export state machine | All others |
| `keydialog` | Tianditu API key input UI | — |
| `colormapdialog` | Color map list + gradient preview | `raster_renderer` |
| `resampledialog` | Resolution spinbox + method combo | — |
| `geo_tiff_loader` | GDAL RasterIO read, min/max/nodata calc, WGS84 bounds (with 3857 fallback) | GDAL, `raster_renderer` |
| `shapefile_loader` | OGR vector read, GeoJSON string output | GDAL |
| `raster_renderer` | 7 colormap definitions, `renderFrame(data → QImage)` with nodata handling | Qt GUI |
| `tiff_utils` | `createMultiBandTiff()`, `computeWGS84Bounds()`, `runGdalwarp()` | GDAL, QProcess |
| `gif_exporter` | Multi-band TIFF read → per-band `renderFrame()` → PNG frames → ImageMagick | GDAL, `raster_renderer` |

## Key Design Decisions

- **RasterRenderer** pooling: single `renderFrame()` eliminates 3x duplicated render logic (geo_tiff_loader, onPlayAnimation, gif_exporter)
- **TiffUtils** extraction: CRS detection and gdalwarp calls moved out of mainwindow for testability
- **Colormaps** live in `RasterRenderer`, not `GeoBIFFLoader` — data is separate from loading
- **Export state machine** (`ExportNone/SingleTiff/Animation`) keeps a single menu item adaptive
- **Downsampling output** EPSG:3857: meters for `-tr`, pixel grid matches Leaflet basemap
- **L.imageOverlay distortion** accepted for quick open; resampled 3857 files give perfect alignment

## Coordinate System Notes

- Basemap: Tianditu `vec_w`/`cva_w` → `TILEMATRIXSET=w` → EPSG:3857 Web Mercator
- Source TIFFs: typically EPSG:4326 (WGS84 geographic)
- GDAL 3.x axis order fix: `SetAxisMappingStrategy(OAMS_TRADITIONAL_GIS_ORDER)` on every OGRSpatialReference before transform
- CRS fallback: if geoTransform values exceed ±180° / ±90°, assume EPSG:3857

## Known Issues

1. **Large TIFF files**: Memory grows with raster dimensions (full Float64 buffer in RAM)
2. **4326 TIFF on 3857 map**: Slight pixel distortion visible at high zoom (use downsampled 3857)
3. **Qt6 QWebChannel warnings**: Property notify-signal warnings at startup are harmless

## Version History

- **v1.0**: Initial release — GeoTIFF/SHP loading, color mapping
- **v1.1**: Batch import, GDAL-based downsampling, multi-band TIFF, GIF export via ImageMagick
- **v1.2**: Single file downsampling, colormap dialog simplified (Heat/Grayscale removed), all downsampling → EPSG:3857
- **v1.3**: GDAL 3.x axis order fix, CRS auto-detection fallback, context menu (only Reload), adaptive export, QWebChannel bridge
- **v1.3-refactor**: Dead code removal, RasterRenderer (unified rendering), TiffUtils (CRS + gdalwarp), directory restructure, -Wall -Wextra cleanup

---
> Source: [Mao-2001/MAP_Image-Creation](https://github.com/Mao-2001/MAP_Image-Creation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
