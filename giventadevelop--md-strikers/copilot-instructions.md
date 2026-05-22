## portrait-image-card-layout-centering

> Standard patterns for displaying portrait and member images without cropping, with proper centering and aspect ratio preservation


# Portrait Image Display Pattern

## **Overview**
This rule defines the correct patterns for displaying portrait and member images (people photos) that must be fully visible without cropping heads or other important parts. This pattern ensures images are properly centered, maintain their aspect ratio, and display consistently across all screen sizes.

## **Problem Solved**
- **Head Cropping**: Prevents portrait images from cutting off people's heads or faces
- **Improper Centering**: Ensures images are centered both horizontally and vertically
- **Aspect Ratio Issues**: Maintains original image proportions without distortion
- **Inconsistent Display**: Provides uniform display across different portrait orientations
- **User Experience**: Shows complete portraits so viewers can see the full person

## **When to Use This Pattern**

### **Use for:**
- ✅ Team member profiles and portraits
- ✅ Leadership/executive team displays
- ✅ Clergy and religious leader portraits
- ✅ Board member and staff photos
- ✅ Speaker or guest profiles
- ✅ Any people-centric imagery where the full person must be visible

### **Don't Use for:**
- ❌ Banner/hero images (use image_containment_prevention.mdc instead)
- ❌ Background images where cropping is acceptable
- ❌ Decorative images where partial visibility is fine
- ❌ Product or object photos that can be cropped

## **Core Pattern: Portrait Grid Cards**

### **Container Setup**
```tsx
// ✅ DO: Use aspect ratio container with flexible height
<div className="relative w-full h-auto aspect-[3/4] mx-auto mb-4 rounded-lg overflow-hidden sacred-shadow-sm group-hover:sacred-shadow reverent-transition bg-muted/20">
  <div className="relative w-full h-full flex items-center justify-center p-2">
    {/* Image goes here */}
  </div>
</div>

// ❌ DON'T: Use fixed height that crops images
<div className="w-full h-48 mx-auto mb-4 rounded-lg overflow-hidden">
  {/* This will crop images */}
</div>
```

### **Image Configuration**
```tsx
// ✅ DO: Use object-contain with fill and proper sizing
<Image
  src={member.image}
  alt={member.name}
  fill
  sizes="(max-width: 768px) 100vw, (max-width: 1024px) 50vw, 33vw"
  className="object-contain group-hover:scale-105 reverent-transition"
  style={{
    objectPosition: 'center center'
  }}
/>

// ❌ DON'T: Use object-cover which crops images
<Image
  src={member.image}
  alt={member.name}
  width={242}
  height={156}
  className="w-full h-full object-cover"
/>
```

## **Pattern Variations**

### **1. Grid Layout (Team Members, Synod Members)**

**Use Case**: Multiple portraits in a responsive grid (3-4 columns)

**Important**: Use **flexbox with `justify-content: center`** instead of CSS Grid to automatically center the last row when there are fewer cards than columns. This ensures cards expand from the center outward, and the last row is always centered.

```tsx
// ✅ DO: Use flexbox for automatic last-row centering
<div className="flex flex-wrap gap-6 justify-center items-start max-w-7xl mx-auto">
  {members.map((member) => (
    <Link
      key={member.id}
      href={member.href}
      className="bg-card rounded-lg sacred-shadow p-6 hover:sacred-shadow-lg reverent-transition group w-full sm:w-[calc(50%-0.75rem)] lg:w-[calc(33.333%-1rem)] flex-shrink-0"
      style={{ maxWidth: '400px' }}
    >
      <div className="text-center">
        {/* Image Container */}
        <div className="relative w-full h-auto aspect-[3/4] mx-auto mb-4 rounded-lg overflow-hidden sacred-shadow-sm group-hover:sacred-shadow reverent-transition bg-muted/20">
          <div className="relative w-full h-full flex items-center justify-center p-2">
            <Image
              src={member.image}
              alt={member.name}
              fill
              sizes="(max-width: 768px) 100vw, (max-width: 1024px) 50vw, 33vw"
              className="object-contain group-hover:scale-105 reverent-transition"
              style={{ objectPosition: 'center center' }}
            />
          </div>
        </div>
        
        {/* Member Info */}
        <h3 className="font-heading font-semibold text-lg text-foreground mb-2">
          {member.name}
        </h3>
        <p className="font-body text-sm text-primary font-medium mb-3">
          {member.title}
        </p>
      </div>
    </Link>
  ))}
</div>

// ❌ DON'T: Use CSS Grid - last row won't center with fewer items
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
  {/* Last row items will align left, not center */}
</div>
```

**Key Implementation Details**:
- **Flexbox Container**: `flex flex-wrap justify-center` - automatically centers all rows, including the last row
- **Card Widths**: `w-full sm:w-[calc(50%-0.75rem)] lg:w-[calc(33.333%-1rem)]` - responsive widths with gap compensation
- **Prevent Shrinking**: `flex-shrink-0` - maintains card size, prevents compression
- **Max Width**: `style={{ maxWidth: '400px' }}` - consistent card size limit across screen sizes
- **Gap**: `gap-6` - consistent spacing between cards (1.5rem / 24px)
- **Center Alignment**: `justify-center items-start` - centers cards horizontally, aligns tops
- **Result**: Cards expand from center outward; last row automatically centers even with 1-2 cards

### **2. Featured Portrait - Detail Page Layout (Leaders, Individual Member Pages)**

**Use Case**: Large portrait on individual detail pages with accompanying biographical content

**Pattern A: Large Responsive Portrait (Holy Synod Member Detail Pages)**
```tsx
<div className="flex flex-col md:flex-row gap-8">
  {/* Featured Portrait - Left Side - Large Display */}
  <div className="flex-shrink-0 flex justify-center md:justify-start">
    <div className="relative w-72 h-[28rem] md:w-80 md:h-[32rem] lg:w-96 lg:h-[36rem] rounded-lg overflow-hidden sacred-shadow-lg">
      <Image
        src={member.image}
        alt={member.name}
        fill
        sizes="(max-width: 768px) 288px, (max-width: 1024px) 320px, 384px"
        className="object-cover object-top"
        style={{
          objectPosition: 'center 15%'
        }}
        priority
      />
    </div>
  </div>

  {/* Content - Right Side of Image */}
  <div className="flex-1">
    <h3 className="font-heading font-semibold text-2xl text-foreground mb-6">
      {member.name}
    </h3>
    <div className="prose prose-lg max-w-none">
      <p className="font-body text-muted-foreground leading-relaxed">
        {member.biography}
      </p>
    </div>
  </div>
</div>
```

**Pattern B: Medium Featured Portrait (Smaller Layout)**
```tsx
<div className="grid grid-cols-1 md:grid-cols-3 gap-8">
  {/* Image - Fixed dimensions with object-contain */}
  <div className="flex justify-center md:justify-start">
    <div className="relative w-48 h-64 md:w-56 md:h-80 rounded-lg overflow-hidden sacred-shadow bg-muted/20 flex items-center justify-center p-2">
      <Image
        src={member.image}
        alt={member.name}
        fill
        sizes="(max-width: 768px) 192px, 224px"
        className="object-contain"
        style={{ objectPosition: 'center center' }}
        priority
      />
    </div>
  </div>

  {/* Content */}
  <div className="md:col-span-2">
    <h3 className="font-heading font-semibold text-2xl text-foreground mb-2">
      {member.name}
    </h3>
    <p className="font-body text-lg text-primary mb-4">
      {member.title}
    </p>
    <p className="font-body text-muted-foreground leading-relaxed">
      {member.description}
    </p>
  </div>
</div>
```

**Key Differences**:
- **Pattern A (Large)**: Uses responsive sizing (`w-72 md:w-80 lg:w-96`), taller heights, `object-cover` for portraits
- **Pattern B (Medium)**: Uses fixed small size (`w-48 md:w-56`), `object-contain` for complete visibility

### **3. Tall Portrait Cards (Executive Team)**

**Use Case**: Taller cards for full-body or formal portraits

```tsx
<div className="relative h-[400px] lg:h-[450px] overflow-hidden p-4">
  <div className="relative w-full h-full rounded-xl overflow-hidden bg-muted/20 flex items-center justify-center">
    <Image
      src={member.image}
      alt={member.name}
      fill
      sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
      className="object-contain group-hover:scale-105 transition-transform duration-700"
      style={{ objectPosition: 'center center' }}
    />
  </div>
</div>
```

## **Key Implementation Details**

### **Container Requirements**
1. **Aspect Ratio**: Use `aspect-[3/4]` for standard portraits (or `aspect-[2/3]` for taller)
2. **Flexible Height**: Use `h-auto` to let the container adapt to content
3. **Background**: Add `bg-muted/20` or similar for visual polish when images have transparency
4. **Padding**: Include `p-2` or `p-4` on inner container for breathing room
5. **Flexbox Centering**: Use `flex items-center justify-center` for perfect centering

### **Image Requirements**
1. **Fill Property**: Use `fill` instead of fixed width/height for responsive behavior
2. **Object-Contain**: Always use `object-contain` (never `object-cover` for portraits)
3. **Object Position**: Set to `center center` for proper centering
4. **Sizes Attribute**: Provide appropriate sizes for responsive optimization
5. **Hover Effects**: Optional `group-hover:scale-105` for interactivity

### **Responsive Sizing Pattern**
```tsx
// ✅ DO: Progressive sizing based on viewport
sizes="(max-width: 768px) 100vw, (max-width: 1024px) 50vw, 33vw"

// For fixed-width containers:
sizes="(max-width: 768px) 192px, 224px"

// For large hero images:
sizes="(max-width: 768px) 100vw, 50vw"
```

## **Centered Grid Layout Pattern**

### **Overview**
For card grids where the **last row must be centered** (e.g., if you have 10 cards in a 3-column grid, the last row with 1 card should be centered, not left-aligned), use **flexbox with `justify-content: center`** instead of CSS Grid.

### **Why Flexbox Over CSS Grid?**
- **CSS Grid**: Last row items align to the start (left) when fewer than column count
- **Flexbox**: Automatically centers all rows, including the last row with fewer items
- **Result**: Cards expand from center outward, maintaining visual balance

### **Implementation**
```tsx
// ✅ DO: Flexbox with centered alignment
<div className="flex flex-wrap gap-6 justify-center items-start max-w-7xl mx-auto">
  {members.map((member) => (
    <div 
      className="bg-card rounded-lg sacred-shadow p-6 w-full sm:w-[calc(50%-0.75rem)] lg:w-[calc(33.333%-1rem)] flex-shrink-0"
      style={{ maxWidth: '400px' }}
    >
      {/* Card content */}
    </div>
  ))}
</div>

// ❌ DON'T: CSS Grid - last row items won't center
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
  {/* Last row items align left */}
</div>
```

### **Key Classes Explained**
- **`flex flex-wrap`**: Creates wrapping flexbox layout
- **`justify-center`**: Centers cards horizontally (this is the magic!)
- **`items-start`**: Aligns card tops (prevents stretching)
- **`gap-6`**: Consistent spacing (1.5rem / 24px) between cards
- **`w-full sm:w-[calc(50%-0.75rem)] lg:w-[calc(33.333%-1rem)]`**: Responsive widths
  - Mobile: Full width (1 column)
  - Tablet: 50% minus half gap (2 columns)
  - Desktop: 33.333% minus gap compensation (3 columns)
- **`flex-shrink-0`**: Prevents cards from shrinking below their width
- **`maxWidth: '400px'`**: Limits card size for consistency

### **Result**
- **1 card in last row**: Centered
- **2 cards in last row**: Centered together
- **3 cards (full row)**: Normal row layout
- **All rows**: Expand from center outward

## **Common Aspect Ratios**

| Use Case | Aspect Ratio | Class | Description |
|----------|--------------|-------|-------------|
| Standard Portrait | 3:4 | `aspect-[3/4]` | Most common, headshots and bust portraits |
| Tall Portrait | 2:3 | `aspect-[2/3]` | Full body or formal portraits |
| Square | 1:1 | `aspect-square` | Profile pictures, avatars |
| Wide Portrait | 4:5 | `aspect-[4/5]` | Landscape-oriented portraits |

## **Error Handling & Fallbacks**

```tsx
// ✅ DO: Provide fallback and error handling
<Image
  src={member.image || '/images/placeholder-portrait.jpg'}
  alt={member.name}
  fill
  sizes="(max-width: 768px) 100vw, (max-width: 1024px) 50vw, 33vw"
  className="object-contain"
  style={{ objectPosition: 'center center' }}
  onError={(e) => {
    const target = e.target as HTMLImageElement;
    target.src = '/images/placeholder-portrait.jpg';
  }}
/>

// ✅ DO: Show loading state
{isLoading ? (
  <div className="w-full h-full bg-gradient-to-br from-gray-200 to-gray-300 animate-pulse" />
) : (
  <Image {...imageProps} />
)}
```

## **Accessibility Considerations**

```tsx
// ✅ DO: Provide meaningful alt text
<Image
  src={member.image}
  alt={`Portrait of ${member.name}, ${member.title}`}
  // ... other props
/>

// ✅ DO: Use proper heading hierarchy
<h3 className="font-heading font-semibold text-lg">
  {member.name}
</h3>
<p className="font-body text-sm text-primary">
  {member.title}
</p>

// ✅ DO: Make cards keyboard accessible
<Link
  href={member.href}
  className="bg-card rounded-lg sacred-shadow p-6 hover:sacred-shadow-lg focus:outline-none focus:ring-2 focus:ring-primary focus:ring-offset-2"
  aria-label={`View profile of ${member.name}`}
>
  {/* Card content */}
</Link>
```

## **Performance Optimization**

```tsx
// ✅ DO: Use priority for above-the-fold images
<Image
  src={member.image}
  alt={member.name}
  fill
  priority // For first 2-3 visible portraits
  sizes="(max-width: 768px) 100vw, 50vw"
  className="object-contain"
/>

// ✅ DO: Use lazy loading for below-the-fold
<Image
  src={member.image}
  alt={member.name}
  fill
  loading="lazy" // For portraits further down the page
  sizes="(max-width: 768px) 100vw, 33vw"
  className="object-contain"
/>
```

## **Complete Example: Holy Synod Members**

Reference: `src/app/mosc/holy-synod/page.tsx` lines 302-340

```tsx
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
  {synodMembers.filter(member => !member.special).map((member) => (
    <Link
      key={member.name}
      href={member.href}
      className="bg-card rounded-lg sacred-shadow p-6 hover:sacred-shadow-lg reverent-transition group"
    >
      <div className="text-center">
        {/* Image Container - Full image display without cropping */}
        <div className="relative w-full h-auto aspect-[3/4] mx-auto mb-4 rounded-lg overflow-hidden sacred-shadow-sm group-hover:sacred-shadow reverent-transition bg-muted/20">
          <div className="relative w-full h-full flex items-center justify-center p-2">
            <Image
              src={member.image || '/images/holy-synod/placeholder.jpg'}
              alt={member.name}
              fill
              sizes="(max-width: 768px) 100vw, (max-width: 1024px) 50vw, 33vw"
              className="object-contain group-hover:scale-105 reverent-transition"
              style={{
                objectPosition: 'center center'
              }}
            />
          </div>
        </div>
        <h3 className="font-heading font-semibold text-lg text-foreground mb-2 group-hover:text-primary reverent-transition">
          {member.name}
        </h3>
        <p className="font-body text-sm text-primary font-medium mb-3">
          {member.title}
        </p>
        <p className="font-body text-xs text-muted-foreground leading-relaxed line-clamp-3">
          {member.description}
        </p>
      </div>
    </Link>
  ))}
</div>
```

## **Troubleshooting**

### **Image Still Getting Cropped?**
1. ✅ Verify container uses `aspect-[3/4]` or `h-auto`, not fixed `h-48`
2. ✅ Check image uses `object-contain`, not `object-cover`
3. ✅ Ensure inner wrapper has `flex items-center justify-center`
4. ✅ Confirm padding is present on inner container (`p-2` or `p-4`)

### **Image Not Centered?**
1. ✅ Add `flex items-center justify-center` to inner container
2. ✅ Set `objectPosition: 'center center'` in style prop
3. ✅ Verify container has proper `relative` positioning

### **Image Too Small?**
1. ✅ Check source image resolution (should be 600px+ wide)
2. ✅ Adjust `sizes` attribute for larger display
3. ✅ Increase container dimensions if needed
4. ✅ Reduce padding if too much white space

### **Image Distorted?**
1. ✅ Ensure using `object-contain` (preserves aspect ratio)
2. ✅ Remove any explicit `width` or `height` from image className
3. ✅ Let container aspect ratio control dimensions

## **Testing Checklist**

Before deploying portrait image implementations:
- [ ] Images display completely without cropping (check heads, feet, sides)
- [ ] Images are centered horizontally and vertically
- [ ] Aspect ratio is preserved (no stretching/squashing)
- [ ] Layout is responsive across mobile, tablet, desktop
- [ ] Hover effects work smoothly
- [ ] Loading states are handled
- [ ] Error states show fallback images
- [ ] Alt text is meaningful and descriptive
- [ ] Performance is optimized (lazy loading, sizes attribute)
- [ ] Keyboard navigation works for linked portraits

## **Quick Reference: Copy-Paste Ready**

```tsx
// Portrait Card Pattern (Copy-Paste Ready)
<div className="bg-card rounded-lg sacred-shadow p-6 hover:sacred-shadow-lg reverent-transition group">
  <div className="text-center">
    {/* Image Container */}
    <div className="relative w-full h-auto aspect-[3/4] mx-auto mb-4 rounded-lg overflow-hidden sacred-shadow-sm group-hover:sacred-shadow reverent-transition bg-muted/20">
      <div className="relative w-full h-full flex items-center justify-center p-2">
        <Image
          src={imageUrl}
          alt={name}
          fill
          sizes="(max-width: 768px) 100vw, (max-width: 1024px) 50vw, 33vw"
          className="object-contain group-hover:scale-105 reverent-transition"
          style={{ objectPosition: 'center center' }}
        />
      </div>
    </div>
    
    {/* Content */}
    <h3 className="font-heading font-semibold text-lg text-foreground mb-2 group-hover:text-primary reverent-transition">
      {name}
    </h3>
    <p className="font-body text-sm text-primary font-medium mb-3">
      {title}
    </p>
    <p className="font-body text-xs text-muted-foreground leading-relaxed line-clamp-3">
      {description}
    </p>
  </div>
</div>
```

## **Related Patterns**

- **Banner Images**: See `image_containment_prevention.mdc`
- **MOSC Design System**: See `mosc_styling_standards.mdc`
- **Team Section**: See `src/components/TeamSection.tsx`

---

**Last Updated**: December 2024
**Reference**: Holy Synod page implementation (`src/app/mosc/holy-synod/page.tsx`)

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
