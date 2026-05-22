## media-gallery-grid-style

> Media gallery grid styling pattern with medium dark gradient backgrounds for admin pages and conservative gradients for public pages


# Media Gallery Grid Styling Pattern

## **Overview**
This rule defines the standard pattern for displaying media/image grids with gradient backgrounds, matching the design aesthetic of both admin pages (medium dark gradients) and public-facing gallery pages (conservative warm tones). The pattern provides consistent styling, subtle visual depth, and proper icon presentation.

## **Problem Solved**
- **Consistent Grid Styling**: Ensures all media grids use the same visual pattern across admin and public pages
- **Medium Dark Gradient Backgrounds**: Provides professional medium dark gradient backgrounds for admin pages that enhance without overwhelming
- **Conservative Gradient Backgrounds**: Provides subtle, professional gradient backgrounds for public pages
- **Icon Standardization**: Provides consistent icon container and sizing for action buttons (references Icon Standards and Admin Action Buttons rules)
- **Visual Hierarchy**: Creates clear separation between grid container and individual media items
- **Responsive Design**: Works consistently across different screen sizes

## **Core Pattern**

### **Grid Container Structure**
```tsx
// ✅ DO: Use the standard media grid container pattern
// Admin Pages (Medium Dark Gradient)
<div className="relative overflow-hidden rounded-3xl bg-gradient-to-br from-gray-700 via-gray-800 to-gray-700 border border-gray-600/30 shadow-2xl mb-8">
  {/* Medium Dark Radial Gradient Overlay */}
  <div className="absolute inset-0 pointer-events-none opacity-60" style={{ backgroundImage: 'radial-gradient(circle at top left, rgba(255, 255, 255, 0.12), transparent 55%)' }} />

  {/* Grid Content */}
  <div className="relative px-6 py-10 sm:px-10 lg:px-14">
    <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6">
      {/* Media items */}
    </div>
  </div>
</div>
```

## **Key CSS Properties**

### **Container Requirements**
- **`relative`**: Enables absolute positioning of overlay
- **`overflow-hidden`**: Clips content to rounded corners
- **`rounded-3xl`**: Large border radius (24px) for modern appearance
- **`bg-gradient-to-br`**: Diagonal gradient from top-left to bottom-right
- **Admin Pages**: `from-gray-700 via-gray-800 to-gray-700` (medium dark gradient)
- **Public Pages**: `from-background via-muted to-background` (conservative warm tones)
- **Event Pages**: `from-gray-900 via-purple-900 to-indigo-900` (bold dark gradient)
- **Border**:
  - Admin: `border-gray-600/30` (medium dark border)
  - Public: `border-border/30` (subtle border)
  - Event: `border-white/10` (light border on dark)
- **Shadow**:
  - Admin: `shadow-2xl` (large shadow for depth)
  - Public: `sacred-shadow-lg` (MOSC shadow)
  - Event: `shadow-2xl` (large shadow)
- **`mb-8`**: Bottom margin (32px) for spacing

### **Radial Gradient Overlay Requirements**
- **`absolute inset-0`**: Covers entire container
- **`pointer-events-none`**: Doesn't interfere with interactions
- **Opacity**:
  - **Admin Pages (Medium Dark)**: `opacity-60` (60% for medium dark backgrounds)
  - **Public Pages (Conservative)**: `opacity-30` (30% for subtlety)
  - **Event Pages (Bold Dark)**: `opacity-70` (70% for bold dark backgrounds)
- **Radial Gradient**:
  - **Admin Pages**: `radial-gradient(circle at top left, rgba(255, 255, 255, 0.12), transparent 55%)` (white at 12% opacity)
  - **Public Pages**: `radial-gradient(circle at top left, rgba(139, 125, 107, 0.08), transparent 55%)` (primary color at 8% opacity)
  - **Event Pages**: `radial-gradient(circle at top left, rgba(255,255,255,0.18), transparent 55%)` (white at 18% opacity)
  - Starts at top-left corner
  - Fades to transparent at 55% radius

### **Grid Content Requirements**
- **`relative`**: Positions content above overlay
- **`px-6 py-10 sm:px-10 lg:px-14`**: Responsive padding
  - Mobile: 24px horizontal, 40px vertical
  - Small screens: 40px horizontal
  - Large screens: 56px horizontal

### **Grid Requirements**
- **`grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4`**: Responsive grid
  - Mobile: 1 column
  - Small screens (640px+): 2 columns
  - Medium screens (768px+): 3 columns
  - Large screens (1024px+): 4 columns
- **`gap-6`**: Consistent gap between items (24px)

## **Color Coding System**

### **Admin Pages** (Medium Dark Gradient)
- **Background**: `from-gray-700 via-gray-800 to-gray-700`
- **Border**: `border-gray-600/30`
- **Radial Gradient**: `rgba(255, 255, 255, 0.12)` (white at 12% opacity)
- **Overlay Opacity**: `opacity-60` (60% for medium dark backgrounds)

### **Public Pages** (MOSC Warm Earth Tones)
- **Background**: `from-background via-muted to-background`
- **Border**: `border-border/30`
- **Radial Gradient**: `rgba(139, 125, 107, 0.08)` (primary color at 8% opacity)

### **Event Pages** (Bold Dark Gradient)
- **Background**: `from-gray-900 via-purple-900 to-indigo-900`
- **Border**: `border-white/10` (light border on dark background)
- **Radial Gradient**: `rgba(255,255,255,0.18)` (white at 18% opacity)
- **Overlay Opacity**: `opacity-70` (70% for bold dark backgrounds)
- **Inner Thumbnail Container**: `bg-white/10 backdrop-blur-sm rounded-2xl border border-white/20 p-6 shadow-inner` (glassmorphism effect)

## **Icon Button Styling Pattern**

**CRITICAL**: All buttons and icons within grid tiles must follow the styling patterns defined in [Icon and Button Styles](mdc:.cursor/rules/icons_buttons_styles.mdc). This ensures consistency across all media gallery pages.

### **Action Button Icons in Grid Tiles**
```tsx
// ✅ DO: Use the standard icon button pattern from icons_buttons_styles.mdc
<button
  onClick={handleAction}
  className="flex-shrink-0 w-10 h-10 rounded-lg bg-{color}-100 hover:bg-{color}-200 flex items-center justify-center transition-all duration-300 hover:scale-110"
  title="Action Label"
  aria-label="Action Label"
  type="button"
>
  <IconComponent className="w-5 h-5 text-{color}-600" />
</button>
```

### **Icon Button Requirements** (from icons_buttons_styles.mdc)
- **`flex-shrink-0`**: Prevents icon container from shrinking
- **`w-10 h-10`**: Fixed icon container size (40px × 40px) - Medium Action Icons pattern
- **`rounded-lg`**: Medium border radius (8px)
- **`bg-{color}-100`**: Light background color matching action type
- **`hover:bg-{color}-200`**: Darker background on hover
- **`flex items-center justify-center`**: Centers icon within container
- **`transition-all duration-300`**: Smooth transitions
- **`hover:scale-110`**: 10% scale increase on hover
- **Icon Size**: `w-5 h-5` (20px × 20px) - Medium Action Icons pattern
- **Icon Color**: `text-{color}-600` (matching action type)

### **Color Coding for Icons** (from icons_buttons_styles.mdc)
- **Edit Actions**: `bg-blue-100 hover:bg-blue-200 text-blue-600`
- **Delete Actions**: `bg-red-100 hover:bg-red-200 text-red-600`
- **View Actions**: `bg-green-100 hover:bg-green-200 text-green-600`
- **Upload Actions**: `bg-purple-100 hover:bg-purple-200 text-purple-600`

### **References for Icon and Button Styling**
- **PRIMARY REFERENCE**: See [Icon and Button Styles](mdc:.cursor/rules/icons_buttons_styles.mdc) for complete icon sizing, styling, color palette, button patterns, and SVG guidelines
- See [Icon Standards](mdc:.cursor/rules/icon_standards.mdc) for complete icon sizing, styling, and color palette guidelines
- See [Admin Action Buttons Styling](mdc:.cursor/rules/admin_action_buttons_styling.mdc) for full-width vertical action buttons and icon container patterns

## **Complete Example**

### **Admin Media Grid** (Medium Dark Gradient)
```tsx
{!loading && sortedMedia.length > 0 && (
  <div className="relative overflow-hidden rounded-3xl bg-gradient-to-br from-gray-700 via-gray-800 to-gray-700 border border-gray-600/30 shadow-2xl mb-8">
    {/* Medium Dark Radial Gradient Overlay */}
    <div className="absolute inset-0 pointer-events-none opacity-60" style={{ backgroundImage: 'radial-gradient(circle at top left, rgba(255, 255, 255, 0.12), transparent 55%)' }} />

    {/* Grid Content */}
    <div className="relative px-6 py-10 sm:px-10 lg:px-14">
      <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6">
        {mediaItems.map((item) => (
          <div key={item.id} className="bg-white rounded-lg shadow-md overflow-hidden group flex flex-col justify-between">
            {/* Media Image */}
            <div className="relative h-48 bg-gray-200">
              <img src={item.fileUrl} alt={item.title} className="w-full h-full object-cover" />
            </div>

            {/* Media Info */}
            <div className="p-4">
              <h3 className="font-semibold text-lg truncate">{item.title}</h3>
              <p className="text-gray-600 text-sm h-10 overflow-hidden">{item.description}</p>
            </div>

            {/* Action Buttons */}
            <div className="p-4 pt-0 flex justify-end gap-2">
              <button
                onClick={() => handleEdit(item)}
                className="flex-shrink-0 w-10 h-10 rounded-lg bg-blue-100 hover:bg-blue-200 flex items-center justify-center transition-all duration-300 hover:scale-110"
                title="Edit Media"
                aria-label="Edit Media"
                type="button"
              >
                <FaEdit className="w-5 h-5 text-blue-600" />
              </button>
              <button
                onClick={() => handleDelete(item)}
                className="flex-shrink-0 w-10 h-10 rounded-lg bg-red-100 hover:bg-red-200 flex items-center justify-center transition-all duration-300 hover:scale-110"
                title="Delete Media"
                aria-label="Delete Media"
                type="button"
              >
                <FaTrashAlt className="w-5 h-5 text-red-600" />
              </button>
            </div>
          </div>
        ))}
      </div>
    </div>
  </div>
)}
```

### **MOSC Gallery Grid** (Conservative Warm Tones)
```tsx
<section className="py-16 bg-muted">
  <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    {/* Grid Container with Conservative Gradient Background */}
    <div className="relative overflow-hidden rounded-3xl bg-gradient-to-br from-background via-muted to-background border border-border/30 sacred-shadow-lg">
      {/* Subtle Radial Gradient Overlay */}
      <div className="absolute inset-0 pointer-events-none opacity-40" style={{ backgroundImage: 'radial-gradient(circle at top left, rgba(139, 125, 107, 0.08), transparent 55%)' }} />

      {/* Grid Content */}
      <div className="relative px-6 py-10 sm:px-10 lg:px-14">
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
          {/* Gallery items */}
        </div>
      </div>
    </div>
  </div>
</section>
```

### **Event Pages Gallery Grid** (Bold Dark Gradient with Inner Container)
```tsx
{/* Gallery Section - Event Page Style */}
{gallery.length > 0 && (
  <div className="mb-12 mt-12">
    {/* Outer Gradient Container */}
    <div className="relative overflow-hidden rounded-3xl bg-gradient-to-br from-gray-900 via-purple-900 to-indigo-900 border border-white/10 shadow-2xl">
      {/* Bold Dark Radial Gradient Overlay */}
      <div className="absolute inset-0 pointer-events-none opacity-70" style={{ backgroundImage: 'radial-gradient(circle at top left, rgba(255,255,255,0.18), transparent 55%)' }} />

      {/* Grid Content */}
      <div className="relative px-6 py-10 sm:px-10 lg:px-14">
        {/* Header Section (optional) */}
        <div className="flex flex-col lg:flex-row items-start lg:items-center justify-between gap-8 text-white mb-10">
          {/* Header content */}
        </div>

        {/* Inner Thumbnail Container - Glassmorphism Effect */}
        {previewMedia.length > 0 && (
          <div className="bg-white/10 backdrop-blur-sm rounded-2xl border border-white/20 p-6 shadow-inner">
            {/* Thumbnail Grid - Uses flexbox with centered alignment */}
            <div className="flex flex-wrap gap-3 justify-center items-start">
              {previewMedia.map((mediaItem) => (
                <button
                  key={mediaItem.id}
                  className="relative w-[220px] h-[220px] rounded-2xl overflow-hidden bg-white/8 border border-white/22 backdrop-blur-[10px] shadow-[0_18px_30px_-18px_rgba(15,23,42,0.55)] transition-all duration-300 hover:-translate-y-1.5 hover:shadow-[0_22px_40px_-20px_rgba(15,23,42,0.65)] hover:border-white/35 group"
                >
                  {/* Image */}
                  <Image
                    src={mediaItem.fileUrl}
                    alt={mediaItem.altText || mediaItem.title}
                    fill
                    className="object-cover transition-transform duration-500 group-hover:scale-110"
                    sizes="(min-width: 1024px) 220px, (min-width: 640px) 200px, 160px"
                  />
                  {/* Hover Overlay */}
                  <div className="absolute inset-0 bg-gradient-to-t from-black/60 via-transparent to-transparent opacity-0 group-hover:opacity-100 transition-opacity duration-300" />
                </button>
              ))}
            </div>
          </div>
        )}
      </div>
    </div>
  </div>
)}
```

**Event Page Inner Container Requirements:**
- **`bg-white/10`**: Semi-transparent white background (10% opacity)
- **`backdrop-blur-sm`**: Small backdrop blur for glassmorphism effect
- **`rounded-2xl`**: Large border radius (16px) for inner container
- **`border border-white/20`**: Light border (20% opacity white)
- **`p-6`**: Padding (24px) for inner container
- **`shadow-inner`**: Inner shadow for depth

**Event Page Thumbnail Requirements:**
- **Size**: `w-[220px] h-[220px]` (220px × 220px) - Fixed square thumbnails
- **Border Radius**: `rounded-2xl` (16px)
- **Background**: `bg-white/8` (8% opacity white)
- **Border**: `border border-white/22` (22% opacity white)
- **Backdrop Blur**: `backdrop-blur-[10px]` (10px blur)
- **Shadow**: `shadow-[0_18px_30px_-18px_rgba(15,23,42,0.55)]` (custom shadow)
- **Hover Effect**: `hover:-translate-y-1.5` (lifts up 6px) with enhanced shadow
- **Image Hover**: `group-hover:scale-110` (10% scale increase)
- **Overlay**: Gradient overlay on hover (`from-black/60 via-transparent to-transparent`)

## **Media Card Styling**

### **Card Requirements**
- **`bg-white`**: White background for cards
- **`rounded-lg`**: Medium border radius (8px)
- **`shadow-md`**: Medium shadow for depth
- **`overflow-hidden`**: Clips content to rounded corners
- **`flex flex-col`**: Vertical flex layout
- **`justify-between`**: Spaces content evenly

### **Image Container**
- **`relative h-48`**: Fixed height (192px) for consistent card size
- **`bg-gray-200`**: Placeholder background color
- **`object-cover`**: Maintains aspect ratio while filling container

## **Responsive Breakpoints**

### **Grid Column Behavior**
- **`grid-cols-1`** (mobile, default): 1 column, full width cards
- **`sm:grid-cols-2`** (640px+): 2 columns, side-by-side on small tablets
- **`md:grid-cols-3`** (768px+): 3 columns, better use of medium screens
- **`lg:grid-cols-4`** (1024px+): 4 columns, optimal for desktop (admin pages)

### **Padding Scaling**
- **Mobile**: `px-6 py-10` (24px horizontal, 40px vertical)
- **Small screens**: `sm:px-10` (40px horizontal)
- **Large screens**: `lg:px-14` (56px horizontal)

## **Accessibility Requirements**

### **Required Attributes**
- **`title`**: Tooltip text for hover state
- **`aria-label`**: Screen reader accessible label (should match button text)
- **`type="button"`**: Explicit button type
- **Semantic HTML**: Use `<button>` for actions, not `<div>`

### **Example with Accessibility**
```tsx
<button
  onClick={handleAction}
  className="flex-shrink-0 w-10 h-10 rounded-lg bg-blue-100 hover:bg-blue-200 flex items-center justify-center transition-all duration-300 hover:scale-110"
  title="Edit Media"
  aria-label="Edit Media"
  type="button"
>
  <FaEdit className="w-5 h-5 text-blue-600" />
</button>
```

## **Best Practices**

### **DO:**
- ✅ Use conservative gradient backgrounds (`from-{color}-50 via-gray-50 to-{color}-50`)
- ✅ Include subtle radial gradient overlay at low opacity (30-40%)
- ✅ Use consistent grid layout: `grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6`
- ✅ Always include `title` and `aria-label` attributes on icon buttons
- ✅ Use `w-10 h-10` for icon containers, `w-5 h-5` for icons
- ✅ Include hover effects: `hover:scale-110 transition-all duration-300`
- ✅ Use semantic color choices (blue for edit, red for delete, green for view)
- ✅ Match border colors to gradient colors (`border-{color}-200/50`)

### **DON'T:**
- ❌ Use light conservative gradients on admin pages (use medium dark gradients)
- ❌ Skip radial gradient overlay (adds visual depth)
- ❌ Use different grid layouts on different pages
- ❌ Mix icon container sizes (always use `w-10 h-10`)
- ❌ Skip accessibility attributes (`title`, `aria-label`)
- ❌ Use different gap values (always use `gap-6`)
- ❌ Omit hover effects on icon buttons
- ❌ Use arbitrary colors without semantic meaning

## **Color Reference Table**

| Page Type | Background Gradient | Border | Radial Gradient Color | Overlay Opacity | Inner Container |
|----|----|----|----|----|----|
| **Admin Pages** | `from-gray-700 via-gray-800 to-gray-700` | `border-gray-600/30` | `rgba(255, 255, 255, 0.12)` | 60% | N/A |
| **MOSC Public** | `from-background via-muted to-background` | `border-border/30` | `rgba(139, 125, 107, 0.08)` | 40% | N/A |
| **Event Pages** | `from-gray-900 via-purple-900 to-indigo-900` | `border-white/10` | `rgba(255,255,255,0.18)` | 70% | `bg-white/10 backdrop-blur-sm rounded-2xl border border-white/20 p-6 shadow-inner` |

## **Reference Implementations**

- **Admin Media Page**: [`src/app/admin/media/page.tsx`](mdc:src/app/admin/media/page.tsx) - Lines 764-824 (media grid with gradient background)
- **MOSC Gallery Page**: [`src/app/mosc/gallery/page.tsx`](mdc:src/app/mosc/gallery/page.tsx) - Lines 309-376 (gallery grid with conservative gradient)
- **Events Page Gallery**: [`src/app/events/[id]/page.tsx`](mdc:src/app/events/[id]/page.tsx) - Lines 1129-1203 (event gallery with dark gradient)

## **Troubleshooting**

### **Gradient Not Visible?**
- Check that gradient colors are light enough (`-50` shades)
- Verify opacity is set correctly (30-40% for conservative, 70% for bold)
- Ensure radial gradient overlay is present

### **Icons Not Scaling on Hover?**
- Check that `hover:scale-110` is included
- Verify `transition-all duration-300` is present
- Ensure parent container doesn't have `overflow-hidden` that clips the scale

### **Grid Not Responsive?**
- Verify grid classes: `grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4`
- Check that Tailwind responsive breakpoints are configured correctly
- Ensure container has sufficient width for columns to display

### **Border Not Visible?**
- Check border opacity (`/50` or `/30` for subtle borders)
- Verify border color matches gradient color family
- Ensure border width is set (`border` = 1px)

## **Related Patterns**

- **PRIMARY**: See [Icon and Button Styles](mdc:.cursor/rules/icons_buttons_styles.mdc) for complete button and icon styling patterns used within grid tiles
- See [Admin Action Buttons Styling](mdc:.cursor/rules/admin_action_buttons_styling.mdc) for full-width vertical action buttons and icon container patterns
- See [Icon Standards](mdc:.cursor/rules/icon_standards.mdc) for complete icon sizing, styling, color palette, and SVG patterns
- See [MOSC Styling Standards](mdc:.cursor/rules/mosc_styling_standards.mdc) for overall design system
- See [Pagination Footer](mdc:.cursor/rules/pagination_footer_styling_pattern.mdc) for pagination controls

## **Event Page Gallery Structure**

### **Two-Level Container Pattern**
Event pages use a two-level container structure for enhanced visual depth:

1. **Outer Container** (Gradient Background):
   - Background: `from-gray-900 via-purple-900 to-indigo-900` (bold dark gradient)
   - Border: `border-white/10` (light border on dark)
   - Shadow: `shadow-2xl` (large shadow)
   - Radial overlay: `opacity-70` with `rgba(255,255,255,0.18)`

2. **Inner Container** (Glassmorphism Thumbnail Container):
   - Background: `bg-white/10 backdrop-blur-sm` (semi-transparent with blur)
   - Border: `border border-white/20` (light border)
   - Border radius: `rounded-2xl` (16px)
   - Padding: `p-6` (24px)
   - Shadow: `shadow-inner` (inner shadow for depth)

### **Thumbnail Grid Styling**
- **Layout**: Flexbox with `flex-wrap` and `justify-center` for centered alignment
- **Gap**: `gap-3` (12px) between thumbnails
- **Thumbnail Size**: `w-[220px] h-[220px]` (220px × 220px) - Fixed square
- **Thumbnail Styling**: Glassmorphism effect with backdrop blur, semi-transparent background, and hover lift effect
- **Responsive**: Thumbnails adapt on mobile (see CSS module for breakpoints)

## **Summary**

**Key Pattern**: Media grids should use gradient backgrounds with subtle radial overlays:
- **Admin Pages**: Medium dark gradients (`from-gray-700 via-gray-800 to-gray-700`) with 60% overlay opacity
- **Public Pages**: Warm earth tones (`from-background via-muted to-background`) with 40% overlay opacity
- **Event Pages**: Bold dark gradients (`from-gray-900 via-purple-900 to-indigo-900`) with 70% overlay opacity and inner glassmorphism container

**Icon Buttons**: All buttons and icons within grid tiles must follow the patterns defined in [Icon and Button Styles](mdc:.cursor/rules/icons_buttons_styles.mdc):
- Fixed container size (`w-10 h-10` for medium action icons)
- Icon size (`w-5 h-5` for medium action icons)
- Semantic colors (blue for edit, red for delete, green for view)
- Hover effects (`hover:scale-110` for square buttons)
- Proper accessibility attributes (`title`, `aria-label`)

This ensures consistent, professional appearance across all media gallery pages while maintaining appropriate styling for each context.

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
