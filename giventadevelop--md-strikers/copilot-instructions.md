## loading-animation-pattern

> This rule defines the standard pattern for loading animations used across admin pages and success pages. The pattern provides consistent visual feedback with animated loading images and wavy overlay effects, matching the design aesthetic used in EventList and manage-events pages.

# Loading Animation Pattern

## **Overview**
This rule defines the standard pattern for loading animations used across admin pages and success pages. The pattern provides consistent visual feedback with animated loading images and wavy overlay effects, matching the design aesthetic used in EventList and manage-events pages.

## **Problem Solved**
- **Consistent Loading UI**: Ensures all loading states use the same visual pattern across admin and success pages
- **Visual Feedback**: Provides engaging animations (pulse, zoom, wavy overlay) to indicate processing
- **Professional Appearance**: Uses high-quality loading images with smooth animations
- **User Experience**: Clear visual indication that content is being loaded

## **Core Pattern**

### **Loading Container Structure**
```tsx
// ✅ DO: Use the standard loading animation pattern
if (loading) {
  return (
    <div className="flex justify-center items-center min-h-[600px] w-full">
      <div className="relative w-full max-w-6xl">
        <Image
          src="/images/loading_events.jpg"
          alt="Loading..."
          width={800}
          height={600}
          className="w-full h-auto rounded-lg shadow-2xl animate-pulse zoom-loading"
          priority
        />
        <div className="absolute inset-0 rounded-lg overflow-hidden">
          <div className="wavy-animation"></div>
        </div>
      </div>
    </div>
  );
}
```

## **Key CSS Properties**

### **Container Requirements**
- **`flex justify-center items-center`**: Centers loading content horizontally and vertically
- **`min-h-[600px]`**: Minimum height (600px) for consistent loading area
- **`w-full`**: Full width of parent container
- **`relative`**: Enables absolute positioning of overlay
- **`w-full max-w-6xl`**: Full width with maximum width constraint (1152px)

### **Image Requirements**
- **`src="/images/loading_events.jpg"`**: Standard loading image path
- **`width={800} height={600}`**: Image dimensions for Next.js Image optimization
- **`w-full h-auto`**: Full width with automatic height (maintains aspect ratio)
- **`rounded-lg`**: Medium border radius (8px) for rounded corners
- **`shadow-2xl`**: Large shadow for depth
- **`animate-pulse`**: Tailwind pulse animation (opacity fade in/out)
- **`zoom-loading`**: Custom zoom animation (scale 0.8 → 1.1 → 1.0)
- **`priority`**: Next.js Image priority loading (loads immediately)

### **Wavy Overlay Requirements**
- **`absolute inset-0`**: Covers entire container (absolute positioned, all sides at 0)
- **`rounded-lg overflow-hidden`**: Matches container border radius, clips overlay
- **`wavy-animation`**: Custom CSS class for wavy shimmer effect

## **CSS Animation Definitions**

### **Zoom Loading Animation** (defined in `src/app/globals.css`)
```css
@keyframes zoomInOut {
  0% {
    transform: scale(0.8);
    opacity: 0.7;
  }
  50% {
    transform: scale(1.1);
    opacity: 1;
  }
  100% {
    transform: scale(1);
    opacity: 0.9;
  }
}

.zoom-loading {
  animation: zoomInOut 2s ease-in-out infinite;
}
```

### **Wavy Animation** (defined in `src/app/globals.css`)
```css
@keyframes wavy {
  0%, 100% {
    transform: translateY(0px) scale(1);
    opacity: 0.3;
  }
  50% {
    transform: translateY(-15px) scale(1.05);
    opacity: 0.7;
  }
}

@keyframes shimmer {
  0% {
    background-position: -200% 0;
  }
  100% {
    background-position: 200% 0;
  }
}

.wavy-animation {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(90deg,
    transparent 0%,
    rgba(255, 193, 7, 0.2) 25%,
    rgba(255, 193, 7, 0.4) 50%,
    rgba(255, 193, 7, 0.2) 75%,
    transparent 100%);
  background-size: 200% 100%;
  animation:
    wavy 3s ease-in-out infinite,
    shimmer 2s ease-in-out infinite;
  border-radius: inherit;
  pointer-events: none;
}
```

## **Complete Example**

### **EventList Loading Pattern**
```tsx
// ✅ DO: Use the standard loading animation pattern (EventList component)
if (loading) {
  return (
    <div className="flex justify-center items-center min-h-[600px] w-full">
      <div className="relative w-full max-w-6xl">
        <Image
          src="/images/loading_events.jpg"
          alt="Loading events..."
          width={800}
          height={600}
          className="w-full h-auto rounded-lg shadow-2xl animate-pulse zoom-loading"
          priority
        />
        <div className="absolute inset-0 rounded-lg overflow-hidden">
          <div className="wavy-animation"></div>
        </div>
      </div>
    </div>
  );
}
```

### **Membership Success Loading Pattern**
```tsx
// ✅ DO: Use the standard loading animation pattern (MembershipSuccessClient component)
if (loading) {
  return (
    <div className="flex justify-center items-center min-h-[600px] w-full">
      <div className="relative w-full max-w-6xl">
        <Image
          src="/images/loading_events.jpg"
          alt="Loading membership subscription..."
          width={800}
          height={600}
          className="w-full h-auto rounded-lg shadow-2xl animate-pulse zoom-loading"
          priority
        />
        <div className="absolute inset-0 rounded-lg overflow-hidden">
          <div className="wavy-animation"></div>
        </div>
      </div>
    </div>
  );
}
```

## **Required Imports**

```tsx
import Image from 'next/image';
```

## **Animation Behavior**

### **Combined Animations**
1. **Pulse Animation** (`animate-pulse`): Tailwind CSS built-in animation
   - Fades opacity between 1.0 and 0.5
   - Duration: 2s
   - Infinite loop

2. **Zoom Animation** (`zoom-loading`): Custom CSS animation
   - Scales from 0.8 → 1.1 → 1.0
   - Opacity changes: 0.7 → 1.0 → 0.9
   - Duration: 2s
   - Easing: ease-in-out
   - Infinite loop

3. **Wavy Animation** (`wavy-animation`): Custom CSS overlay
   - Vertical movement: translateY(0px) → translateY(-15px) → translateY(0px)
   - Scale: 1.0 → 1.05 → 1.0
   - Opacity: 0.3 → 0.7 → 0.3
   - Shimmer effect: horizontal gradient movement
   - Duration: 3s (wavy) + 2s (shimmer)
   - Infinite loop

## **Best Practices**

### **DO:**
- ✅ Use Next.js `Image` component (not `<img>` tag)
- ✅ Always include `priority` prop for loading images
- ✅ Use `/images/loading_events.jpg` as the standard loading image
- ✅ Include both `animate-pulse` and `zoom-loading` classes
- ✅ Always include the `wavy-animation` overlay div
- ✅ Use `min-h-[600px]` for consistent loading area height
- ✅ Use `max-w-6xl` for maximum width constraint
- ✅ Use `relative` container with `absolute inset-0` overlay
- ✅ Include descriptive `alt` text for accessibility

### **DON'T:**
- ❌ Use different loading images (always use `/images/loading_events.jpg`)
- ❌ Skip the `wavy-animation` overlay (required for visual consistency)
- ❌ Use `<img>` tag instead of Next.js `Image` component
- ❌ Omit `priority` prop (causes loading delay)
- ❌ Use different animation classes or custom animations
- ❌ Skip `rounded-lg` border radius (required for overlay clipping)
- ❌ Use different container dimensions (always use `min-h-[600px]`)

## **Accessibility Requirements**

### **Required Attributes**
- **`alt`**: Descriptive text for screen readers (e.g., "Loading events...", "Loading membership subscription...")
- **`priority`**: Next.js Image optimization (loads immediately)

### **Example with Accessibility**
```tsx
<Image
  src="/images/loading_events.jpg"
  alt="Loading events..."
  width={800}
  height={600}
  className="w-full h-auto rounded-lg shadow-2xl animate-pulse zoom-loading"
  priority
/>
```

## **Reference Implementations**

- **EventList Component**: [`src/components/EventList.tsx`](mdc:src/components/EventList.tsx) - Lines 202-219
- **Membership Success Client**: [`src/app/membership/success/MembershipSuccessClient.tsx`](mdc:src/app/membership/success/MembershipSuccessClient.tsx) - Lines 422-437
- **CSS Animations**: [`src/app/globals.css`](mdc:src/app/globals.css) - Lines 513-594
- **Manage Events Page**: [`src/app/admin/manage-events/page.tsx`](mdc:src/app/admin/manage-events/page.tsx) - Uses EventList component

## **Troubleshooting**

### **Animation Not Working?**
- Check that CSS classes are defined in `src/app/globals.css`
- Verify `wavy-animation` and `zoom-loading` classes are present
- Ensure `animate-pulse` is available (Tailwind CSS built-in)
- Check browser console for CSS errors

### **Image Not Loading?**
- Verify `/images/loading_events.jpg` exists in `public/images/`
- Check Next.js Image optimization is working (check Network tab)
- Ensure `priority` prop is included
- Verify `width` and `height` props match image dimensions

### **Overlay Not Visible?**
- Check that container has `relative` positioning
- Verify overlay div has `absolute inset-0`
- Ensure `rounded-lg overflow-hidden` is on overlay container
- Check that `wavy-animation` class is applied

### **Layout Issues?**
- Verify `min-h-[600px]` is set on container
- Check that `flex justify-center items-center` is applied
- Ensure `w-full max-w-6xl` is on image container
- Verify responsive classes work on mobile

## **Related Patterns**

- See [MOSC Styling Standards](mdc:.cursor/rules/mosc_styling_standards.mdc) for overall design system
- See [`src/components/EventList.tsx`](mdc:src/components/EventList.tsx) for complete implementation
- See [`src/app/membership/success/MembershipSuccessClient.tsx`](mdc:src/app/membership/success/MembershipSuccessClient.tsx) for success page implementation

## **Summary**

**Key Pattern**: Loading states should use:
- Next.js `Image` component with `/images/loading_events.jpg`
- Classes: `animate-pulse zoom-loading`
- Wavy overlay: `<div className="wavy-animation"></div>`
- Container: `flex justify-center items-center min-h-[600px] w-full`
- Image container: `relative w-full max-w-6xl`

This ensures consistent, professional loading animations across all admin and success pages.

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
