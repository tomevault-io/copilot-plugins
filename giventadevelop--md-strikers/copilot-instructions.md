## image-containment-prevention-hero-images-cards

> CSS rules and patterns for displaying images without cropping or truncation


# Image Containment Prevention Pattern

## **Overview**
This rule defines the correct CSS patterns for displaying banner, hero, and sponsor images that must be fully visible without cropping or truncation on any side. This pattern ensures images maintain their aspect ratio and are contained within their containers.

## **Problem Solved**
- **Image Cropping**: Prevents images from being cut off on left, right, top, or bottom edges
- **Aspect Ratio Preservation**: Maintains original image proportions
- **Consistent Display**: Ensures images display the same way across different screen sizes
- **User Experience**: Users can see complete images without important content being hidden

## **Core Pattern**

### **Container Styling**
```tsx
// ✅ DO: Use flexible height container
<div className="relative w-full h-auto rounded-t-2xl overflow-hidden">
  {/* Image content */}
</div>

// ❌ DON'T: Use fixed height container
<div className="relative w-full h-[448px] rounded-t-2xl overflow-hidden">
  {/* This causes cropping */}
</div>
```

### **Image Styling**
```tsx
// ✅ DO: Use object-contain with flexible dimensions
<Image
  src={imageUrl}
  alt="Description"
  width={800}
  height={600}
  className="w-full h-auto object-contain group-hover:scale-105 transition-transform duration-300"
  style={{
    backgroundColor: 'transparent',
    borderRadius: '1rem 1rem 0 0'
  }}
/>

// ❌ DON'T: Use object-cover with fixed dimensions
<Image
  src={imageUrl}
  alt="Description"
  width={800}
  height={600}
  className="w-full h-full object-cover group-hover:scale-105 transition-transform duration-300"
  style={{
    borderRadius: '1rem 1rem 0 0'
  }}
/>
```

## **Key CSS Properties**

### **Container Requirements**
- **`relative`**: Enables absolute positioning of child elements (badges, overlays)
- **`w-full`**: Full width of parent container
- **`h-auto`**: **CRITICAL**: Flexible height allows image to maintain aspect ratio
- **`rounded-t-2xl`**: Rounded top corners (1rem radius)
- **`overflow-hidden`**: Clips content to rounded corners

### **Image Requirements**
- **`w-full`**: Full width of container
- **`h-auto`**: **CRITICAL**: Flexible height maintains aspect ratio
- **`object-contain`**: **CRITICAL**: Shows complete image without cropping (alternative to `object-cover`)
- **`group-hover:scale-105`**: Optional hover effect for interactivity
- **`transition-transform duration-300`**: Smooth hover animation
- **`backgroundColor: 'transparent'`**: Allows card gradient to show through
- **`borderRadius: '1rem 1rem 0 0'`**: Matches container's rounded corners

## **When to Use This Pattern**

### **Use `object-contain` When:**
- ✅ Banner images (sponsor banners, event banners)
- ✅ Hero images that must show complete content
- ✅ Logo images that need full visibility
- ✅ Images where cropping would hide important content
- ✅ Images in cards or containers where aspect ratio matters

### **Use `object-cover` When:**
- ✅ Background images where cropping is acceptable
- ✅ Thumbnail images where partial visibility is fine
- ✅ Decorative images where content isn't critical
- ✅ Images in fixed-size containers where filling space is priority

## **Complete Example Pattern**

```tsx
{/* Image Container - Matching events page style */}
<div className="relative w-full h-auto rounded-t-2xl overflow-hidden">
  {imageUrl ? (
    <Image
      src={imageUrl}
      alt="Image description"
      width={800}
      height={600}
      className="w-full h-auto object-contain group-hover:scale-105 transition-transform duration-300"
      style={{
        backgroundColor: 'transparent',
        borderRadius: '1rem 1rem 0 0'
      }}
    />
  ) : (
    <div
      className="w-full h-80 flex items-center justify-center"
      style={{
        backgroundColor: 'transparent',
        borderRadius: '1rem 1rem 0 0'
      }}
    >
      <span className="text-gray-400 text-5xl">🏢</span>
    </div>
  )}
</div>
```

## **Reference Implementations**

### **Events Page Pattern**
See [`src/app/events/page.tsx`](mdc:src/app/events/page.tsx) lines 771-783 for the canonical implementation:
```tsx
<div className="relative w-full h-auto rounded-t-2xl overflow-hidden">
  {event.thumbnailUrl ? (
    <Image
      src={event.thumbnailUrl}
      alt={event.title}
      width={800}
      height={600}
      className="w-full h-auto object-contain group-hover:scale-105 transition-transform duration-300"
      style={{
        backgroundColor: 'transparent',
        borderRadius: '1rem 1rem 0 0'
      }}
    />
  ) : (
    {/* Placeholder */}
  )}
</div>
```

### **Sponsor Images Pattern**
See [`src/app/events/[id]/page.tsx`](mdc:src/app/events/[id]/page.tsx) lines 964-991 for sponsor banner images:
```tsx
<div className="relative w-full h-auto rounded-t-2xl overflow-hidden">
  {displayImageUrl ? (
    <Image
      src={displayImageUrl}
      alt={sponsor.name}
      width={800}
      height={600}
      className="w-full h-auto object-contain group-hover:scale-105 transition-transform duration-300"
      style={{
        backgroundColor: 'transparent',
        borderRadius: '1rem 1rem 0 0'
      }}
    />
  ) : (
    {/* Placeholder */}
  )}
</div>
```

### **OurSponsorsSection Pattern**
See [`src/components/OurSponsorsSection.tsx`](mdc:src/components/OurSponsorsSection.tsx) lines 201-224 for sponsor section images.

## **Common Anti-Patterns to Avoid**

```tsx
// ❌ DON'T: Use fixed height with object-cover
<div className="relative w-full h-[448px] rounded-t-2xl overflow-hidden">
  <Image
    className="w-full h-full object-cover"
    {/* This crops images */}
  />
</div>

// ❌ DON'T: Use object-cover for images that must be fully visible
<Image
  className="w-full h-full object-cover"
  {/* This will crop important content */}
/>

// ❌ DON'T: Mix fixed height container with object-contain
<div className="relative w-full h-[448px]">
  <Image
    className="w-full h-auto object-contain"
    {/* Container height conflicts with image sizing */}
  />
</div>
```

## **Troubleshooting**

### **Image Still Getting Cropped?**
1. Check container height: Must be `h-auto`, not fixed height (`h-[XXXpx]`)
2. Check image class: Must use `object-contain`, not `object-cover`
3. Check image dimensions: Use `w-full h-auto`, not `w-full h-full`
4. Verify overflow: Container should have `overflow-hidden` for rounded corners

### **Image Too Small?**
- Ensure `width` and `height` props are set appropriately (e.g., `width={800} height={600}`)
- Check if container has sufficient width (`w-full`)
- Verify no conflicting CSS is overriding the styles

### **Image Not Centered?**
- Add `flex items-center justify-center` to container if needed
- Use `object-position: center` in style if required

## **Hero Section Mobile Layout — Aspect-Ratio Containment (March 2026)**

The homepage hero section uses a 4-section split layout with images of **varying aspect ratios** (5:6 portrait, 5:2 landscape, 5:1 ultra-wide). On mobile, each requires a different containment strategy. The full technique is documented in the dedicated rule:

**See [Hero Mobile Image Containment](mdc:.cursor/rules/hero_mobile_image_containment_aspect_ratio.mdc)** for:
- The **aspect-ratio matching technique** (container ratio = image ratio + `object-contain` = zero crop, zero gaps)
- Section-by-section CSS with rationale for each approach
- Decision tree: when to use `aspect-ratio` vs `max-height` cap vs `object-cover`
- Common pitfalls and lessons learned from iterative fixes
- Desktop (768px+) strategy differences

### **Quick Summary**

| Section | Image Ratio | Mobile Technique | Container Key CSS |
|---------|-------------|-----------------|-------------------|
| 1 (Kerala) | 5:6 portrait | `object-contain` + `max-height: 22vh` + dark bg | `background: #1a0a2e` blends side bars |
| 2 (Slideshow) | 5:2 landscape | `aspect-ratio: 5/2` + `object-contain` | Zero crop, zero gaps |
| 3 (Mission) | 5:1 ultra-wide | `aspect-ratio: 5/1` + `object-contain` + text overlay | `position: absolute` text + gradient |

---

## **Related Patterns**

- See [Hero Mobile Image Containment](mdc:.cursor/rules/hero_mobile_image_containment_aspect_ratio.mdc) for the complete mobile technique
- See [MOSC Styling Standards](mdc:.cursor/rules/mosc_styling_standards.mdc) for overall design system
- See [`src/app/events/page.tsx`](mdc:src/app/events/page.tsx) for reference implementation
- See [`src/components/OurSponsorsSection.tsx`](mdc:src/components/OurSponsorsSection.tsx) for sponsor image pattern

## **Summary Checklist**

Before submitting images that must not be cropped:
- [ ] Container uses `h-auto` (not fixed height)
- [ ] Image uses `object-contain` (not `object-cover`)
- [ ] Image uses `w-full h-auto` (not `w-full h-full`)
- [ ] Container has `overflow-hidden` for rounded corners
- [ ] Image has `backgroundColor: 'transparent'` in style
- [ ] Image has matching `borderRadius` in style
- [ ] Placeholder div matches image container styling

**Hero section mobile** (see [dedicated rule](mdc:.cursor/rules/hero_mobile_image_containment_aspect_ratio.mdc)):
- [ ] Section 1: `object-contain` + `max-height: 22vh` + `background: #1a0a2e` (portrait → side gaps blend with dark bg)
- [ ] Section 2: `aspect-ratio: 5/2` + `object-contain` (landscape → ratio match = zero crop)
- [ ] Section 3: `aspect-ratio: 5/1` + `object-contain` + `position: absolute` text overlay (ultra-wide → ratio match + gradient text)
- [ ] All sections: `background: #1a0a2e` on containers, no `min-height`, no decorative styles

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
