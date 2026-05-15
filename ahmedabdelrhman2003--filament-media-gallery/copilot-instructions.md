## filament-media-gallery

> **Package:** `ahmed-abdelrhman/filament-media-gallery`

# filament-media-gallery — Claude Code Reference

## Project Overview

**Package:** `ahmed-abdelrhman/filament-media-gallery`
**Type:** Filament plugin (library)
**Purpose:** Renders a [Spatie Laravel Media Library](https://spatie.be/docs/laravel-medialibrary) collection as a grid gallery inside a Filament **Infolist** entry, with a built-in Alpine.js lightbox for images and direct-link cards for PDFs/other files.

- **PHP:** ^8.2
- **Filament:** ^3.0 || ^4.0 || ^5.0
- **Spatie Media Library:** ^10.0 | ^11.0
- **No extra JS or CSS** — relies entirely on Alpine.js and Tailwind already bundled with Filament.

---

## Directory Structure

```
filament-media-gallery/
├── composer.json                                    # Package metadata & autoloading
├── README.md                                        # End-user documentation
├── LICENSE
├── src/
│   ├── FilamentMediaGalleryPlugin.php               # Filament Plugin contract implementation
│   ├── FilamentMediaGalleryServiceProvider.php      # View + component registration
│   └── Infolists/
│       └── Components/
│           └── MediaGalleryEntry.php                # Main infolist component (public API)
└── resources/
    └── views/
        ├── infolist-entries/
        │   └── media-gallery.blade.php              # Thin wrapper rendered by Filament
        └── components/
            └── media-viewer.blade.php               # Full gallery UI + Alpine.js lightbox
```

---

## Architecture & Data Flow

```
Filament Infolist schema
  → MediaGalleryEntry::make('field')        [src/Infolists/Components/MediaGalleryEntry.php]
      → renders view: media-gallery.blade.php
          → calls $entry->getMediaItems()   [loads via Spatie getMedia($collection)]
          → passes items + config to <x-media-gallery-viewer>
              → media-viewer.blade.php      [renders grid, lightbox, PDF cards]
                  → Alpine.js handles all client-side interactions (no server calls)
```

---

## Key Files & Roles

### `src/FilamentMediaGalleryPlugin.php`
Implements `Filament\Contracts\Plugin`. Required for explicit panel registration.
- `make()` — static factory
- `getId()` — returns `'filament-media-gallery'`
- `register(Panel $panel)` / `boot(Panel $panel)` — empty hooks, available for future panel-level config

### `src/FilamentMediaGalleryServiceProvider.php`
Standard Laravel service provider, auto-discovered.
- Loads views from `resources/views` under namespace `media-gallery`
- Registers Blade component alias: `<x-media-gallery-viewer>` → `media-gallery::components.media-viewer`
- Publishes views via `--tag=media-gallery-views`

### `src/Infolists/Components/MediaGalleryEntry.php`
The **main API class** users interact with. Extends `Filament\Infolists\Components\Entry`.

| Property | Type | Default | Description |
|---|---|---|---|
| `$collection` | string | — | Spatie collection name (required) |
| `$relation` | string\|null | null | Related model name for media loading |
| `$size` | int\|null | 250 | Fixed card size in px; null = responsive |
| `$rounded` | bool | false | `rounded-full` vs `rounded-lg` |

**`getMediaItems()`** — core loader:
1. Gets the infolist record
2. If `fromRelation()` set, navigates to that relation first
3. Calls `$model->getMedia($collection)` (Spatie)
4. Returns `Collection` of media items (empty if none)

### `resources/views/infolist-entries/media-gallery.blade.php`
Thin Filament wrapper using `<x-dynamic-component :component="$getEntryWrapperView()">`. Calls `$entry->getMediaItems()` and delegates rendering to `<x-media-gallery-viewer>`. Do not add logic here.

### `resources/views/components/media-viewer.blade.php`
The full UI component — the most complex file in the project.

**Props:** `$media` (Collection), `$size` (int|null), `$rounded` (bool)

Rendering logic:
1. Separates images (MIME starts with `image/`) from other files (e.g. PDFs)
2. Builds `$imageUrls[]` for lightbox and `$imageItems[]` for indexed rendering
3. Computes `$cardStyle` (inline fixed dimensions), `$cardClass` (aspect-square if responsive), `$roundedClass`
4. Images: rendered with thumbnail → preview → full URL priority; magnifier icon on hover; click opens lightbox
5. PDFs/other: red doc icon card, "PDF" badge, opens in new tab on click
6. Shows "No files uploaded." if collection empty

**Alpine.js lightbox state** (defined in `x-data`):
```js
{
  lightboxOpen: false,
  lightboxSrc: '',
  lightboxImages: [],
  lightboxIndex: 0,
  openLightbox(images, index),
  closeLightbox(),
  nextImage(),    // circular
  prevImage(),    // circular
}
```
- ESC key closes lightbox
- Clicking image or backdrop closes lightbox
- Prev/Next buttons and counter badge only appear when 2+ images

---

## Public API

```php
MediaGalleryEntry::make('gallery')          // field name (for label)
    ->collection('photos')                  // REQUIRED: Spatie collection name
    ->fromRelation('user')                  // optional: load from $record->user->getMedia(...)
    ->size(200)                             // optional: fixed px size (null = responsive)
    ->rounded()                             // optional: circular cards
    ->label('Gallery')                      // inherited from Filament Entry
```

---

## Adding New Features — Where to Touch

| What you want to add | Files to change |
|---|---|
| New config option (e.g. `->columns(3)`) | 1. Add property + getter/setter to `MediaGalleryEntry.php` <br>2. Pass as prop in `media-gallery.blade.php` <br>3. Use prop in `media-viewer.blade.php` |
| Support a new media type (e.g. video) | Edit MIME-type detection block in `media-viewer.blade.php` |
| Change lightbox behaviour | Edit Alpine `x-data` object in `media-viewer.blade.php` |
| Panel-level plugin config | Add to `FilamentMediaGalleryPlugin.php` (register/boot) |
| Change how media is loaded | Edit `getMediaItems()` in `MediaGalleryEntry.php` |

---

## Development Notes

- **No routes, no config files, no migrations** — purely view/component based.
- **Publish views for customisation:**
  ```bash
  php artisan vendor:publish --tag=media-gallery-views
  ```
  Views land in `resources/views/vendor/media-gallery/`.
- **Version compatibility** is handled by Filament's stable `Entry` and `Plugin` contracts — zero version-switch code is needed. The composer constraint `^3.0 || ^4.0 || ^5.0` covers all.
- **Image URL priority** in `media-viewer.blade.php`: `thumbnail` conversion → `preview` conversion → full URL. If you add new Spatie conversions, update this priority chain.
- **Dark mode** is supported throughout via Tailwind `dark:` variants — keep this in mind when editing blade files.

---
> Source: [ahmedabdelrhman2003/filament-media-gallery](https://github.com/ahmedabdelrhman2003/filament-media-gallery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
