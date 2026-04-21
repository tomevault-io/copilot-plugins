## premium-design-web

> Image optimization standards for performance


# Image Optimization

## ALWAYS Use next/image

```tsx
// ✅ GOOD
import Image from "next/image";

<Image
  src="/images/hero.webp"
  alt="Descriptive alt text"
  width={1920}
  height={1080}
  sizes="100vw"
  priority // Only for above-the-fold images
/>

// ❌ BAD - Never use raw img tags
<img src="/images/hero.png" alt="" />
```

## Local Images Over Remote

```tsx
// ✅ GOOD - Local optimized images
src="/images/project-hero.webp"
src={localImportedImage}

// ❌ BAD - Remote URLs (no optimization)
src="https://images.unsplash.com/photo-xxx"
```

## Blur Placeholders

Generate unique blur data URLs for each image:

```tsx
// ✅ GOOD - Unique blur per image
placeholder="blur"
blurDataURL={IMAGES.projectHero.blurDataURL}

// ❌ BAD - Same blur for all images
blurDataURL="data:image/jpeg;base64,/9j/4AAQ..." // Generic
```

## Sizes Attribute

Always specify `sizes` for responsive images:

```tsx
// Hero images (full width)
sizes="100vw"

// Grid images (2 columns on desktop)
sizes="(max-width: 768px) 100vw, 50vw"

// Grid images (3 columns on desktop)
sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
```

## Loading Strategy

```tsx
// Above the fold - eager loading
const isAboveFold = index < 3;

<Image
  loading={isAboveFold ? "eager" : "lazy"}
  priority={isAboveFold}
  // ...
/>
```

## WebP Format

Store images as WebP in `/public/images/`:

```
/public/images/
  hero.webp          ✅ Optimized
  hero.png           ❌ Only as fallback
  project-1.webp     ✅ Optimized
```

## Image Data Pattern

Centralize image data in `src/lib/image-data.ts`:

```tsx
export const IMAGES = {
  hero: {
    src: "/images/hero.webp",
    width: 1920,
    height: 1080,
    alt: "Luxury home exterior at twilight",
    blurDataURL: "data:image/jpeg;base64,..." // Unique per image
  },
};
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Hecpac) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
