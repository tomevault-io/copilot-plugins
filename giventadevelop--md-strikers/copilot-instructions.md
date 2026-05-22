## admin-page-responsive-button-group

> This rule defines the standard pattern for responsive button groups displayed in a grid layout across admin pages. The pattern ensures consistent 2-column layout on mobile devices, proper spacing, and responsive sizing for icons, text, and containers.

# Admin Page Responsive Button Group Pattern

## **Overview**
This rule defines the standard pattern for responsive button groups displayed in a grid layout across admin pages. The pattern ensures consistent 2-column layout on mobile devices, proper spacing, and responsive sizing for icons, text, and containers.

## **Problem Solved**
- **Mobile Layout**: Ensures 2 items per row on mobile instead of 1 (better space utilization)
- **Responsive Alignment**: Proper spacing and padding across all screen sizes
- **Icon & Text Scaling**: Icons and text scale appropriately for mobile, tablet, and desktop
- **Consistent Spacing**: Standardized gaps and padding that work across breakpoints
- **Touch-Friendly**: Adequate button sizes for mobile touch interactions

## **Core Pattern**

### **Grid Container**
```tsx
// ✅ DO: Use responsive grid layout with 2 columns on mobile
<div className="bg-white rounded-xl shadow-lg p-4 sm:p-6 lg:p-8">
  <div className="grid grid-cols-2 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-3 sm:gap-4 lg:gap-6">
    {/* Button cards */}
  </div>
</div>
```

### **Button Card Structure**
```tsx
// ✅ DO: Use responsive button card pattern
<Link
  href="/admin/path/to/resource"
  className={`flex flex-col items-center justify-center rounded-lg border-2 p-2.5 sm:p-3 lg:p-4 transition-all duration-300 hover:scale-105 hover:shadow-md group ${colorClasses}`}
  title="Button Label"
  aria-label="Button Label"
>
  <div className={`flex-shrink-0 w-10 h-10 sm:w-11 sm:h-11 rounded-xl ${iconBgColor} flex items-center justify-center mb-1.5 sm:mb-2 group-hover:scale-110 transition-transform duration-300`}>
    <IconComponent className={`w-6 h-6 sm:w-8 sm:h-8 ${iconTextColor}`} />
  </div>
  <span className="font-semibold text-center text-xs sm:text-sm lg:text-base leading-tight px-1">
    Button Label
  </span>
</Link>
```

## **Key CSS Properties**

### **Grid Container Requirements**
- **`grid grid-cols-2`**: **CRITICAL**: 2 columns on mobile (not 1)
- **`sm:grid-cols-2`**: 2 columns on small screens (640px+)
- **`md:grid-cols-3`**: 3 columns on medium screens (768px+)
- **`lg:grid-cols-4`**: 4 columns on large screens (1024px+)
- **`gap-3 sm:gap-4 lg:gap-6`**: Responsive gaps
  - Mobile: 12px (0.75rem)
  - Small screens: 16px (1rem)
  - Large screens: 24px (1.5rem)

### **Card Container Requirements**
- **`p-4 sm:p-6 lg:p-8`**: Responsive padding
  - Mobile: 16px (1rem)
  - Small screens: 24px (1.5rem)
  - Large screens: 32px (2rem)

### **Button Card Requirements**
- **`flex flex-col items-center justify-center`**: Centers content vertically and horizontally
- **`rounded-lg`**: Medium border radius (8px) for card appearance
- **`border-2`**: 2px border for definition
- **`p-2.5 sm:p-3 lg:p-4`**: Responsive padding
  - Mobile: 10px (0.625rem)
  - Small screens: 12px (0.75rem)
  - Large screens: 16px (1rem)
- **`transition-all duration-300`**: Smooth transitions
- **`hover:scale-105`**: 5% scale increase on hover
- **`hover:shadow-md`**: Medium shadow on hover
- **`group`**: Enables group hover effects on child elements

### **Icon Container Requirements**
- **`flex-shrink-0`**: Prevents icon container from shrinking
- **`w-10 h-10 sm:w-11 sm:h-11`**: Responsive icon container size
  - Mobile: 40px × 40px
  - Small screens+: 44px × 44px
- **`rounded-xl`**: Large border radius (12px) for icon container
- **`flex items-center justify-center`**: Centers icon within container
- **`mb-1.5 sm:mb-2`**: Responsive margin bottom
  - Mobile: 6px (0.375rem)
  - Small screens+: 8px (0.5rem)
- **`group-hover:scale-110`**: Scales up 10% when parent card is hovered
- **`transition-transform duration-300`**: Smooth scale animation

### **Icon Requirements**
- **`w-6 h-6 sm:w-8 sm:h-8`**: Responsive icon size
  - Mobile: 24px × 24px
  - Small screens+: 32px × 32px
- **Color**: Use semantic color matching action type (e.g., `text-blue-600`)

### **Text Requirements**
- **`font-semibold`**: Bold text for emphasis
- **`text-center`**: Centers text horizontally
- **`text-xs sm:text-sm lg:text-base`**: Responsive text size
  - Mobile: 12px (0.75rem)
  - Small screens: 14px (0.875rem)
  - Large screens: 16px (1rem)
- **`leading-tight`**: Tighter line height for compact display
- **`px-1`**: **CRITICAL**: Small horizontal padding prevents text overflow on mobile

## **Complete Example**

### **Full Responsive Button Group**
```tsx
export default function AdminPage() {
  const adminButtons = [
    {
      href: '/admin',
      icon: 'home',
      label: 'Admin Home',
      color: 'blue',
      key: 'admin-home'
    },
    // ... more buttons
  ];

  const getColorClasses = (color: string) => {
    const colorMap: Record<string, string> = {
      blue: 'bg-blue-50 hover:bg-blue-100 text-blue-700 border-blue-200',
      green: 'bg-green-50 hover:bg-green-100 text-green-700 border-green-200',
      // ... other colors
    };
    return colorMap[color] || colorMap.blue;
  };

  const getIconBgColor = (color: string) => {
    const colorMap: Record<string, string> = {
      blue: 'bg-blue-100',
      green: 'bg-green-100',
      // ... other colors
    };
    return colorMap[color] || colorMap.blue;
  };

  const getIconTextColor = (color: string) => {
    const colorMap: Record<string, string> = {
      blue: 'text-blue-500',
      green: 'text-green-500',
      // ... other colors
    };
    return colorMap[color] || colorMap.blue;
  };

  return (
    <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-4 sm:py-8" style={{ paddingTop: '120px' }}>
      <h1 className="text-xl sm:text-2xl lg:text-3xl font-bold text-indigo-800 mb-4 sm:mb-8 flex flex-wrap items-center justify-center gap-2 text-center">
        Page Title
      </h1>

      <div className="bg-white rounded-xl shadow-lg p-4 sm:p-6 lg:p-8">
        <div className="grid grid-cols-2 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-3 sm:gap-4 lg:gap-6">
          {adminButtons.map((button) => {
            const colorClasses = getColorClasses(button.color);
            const iconBgColor = getIconBgColor(button.color);
            const iconTextColor = getIconTextColor(button.color);

            return (
              <Link
                key={button.key}
                href={button.href}
                className={`flex flex-col items-center justify-center rounded-lg border-2 p-2.5 sm:p-3 lg:p-4 transition-all duration-300 hover:scale-105 hover:shadow-md group ${colorClasses}`}
                title={button.label}
                aria-label={button.label}
              >
                <div className={`flex-shrink-0 w-10 h-10 sm:w-11 sm:h-11 rounded-xl ${iconBgColor} flex items-center justify-center mb-1.5 sm:mb-2 group-hover:scale-110 transition-transform duration-300`}>
                  {renderIcon(button.icon, `w-6 h-6 sm:w-8 sm:h-8 ${iconTextColor}`)}
                </div>
                <span className="font-semibold text-center text-xs sm:text-sm lg:text-base leading-tight px-1">
                  {button.label}
                </span>
              </Link>
            );
          })}
        </div>
      </div>
    </div>
  );
}
```

## **Responsive Breakpoints**

### **Grid Column Behavior**
- **`grid-cols-2`** (mobile, default): **2 columns** - Better space utilization on mobile
- **`sm:grid-cols-2`** (640px+): 2 columns, maintained for small tablets
- **`md:grid-cols-3`** (768px+): 3 columns, better use of medium screens
- **`lg:grid-cols-4`** (1024px+): 4 columns, optimal for desktop

### **Spacing Scaling**
- **Gaps**: `gap-3` (12px) → `sm:gap-4` (16px) → `lg:gap-6` (24px)
- **Card Padding**: `p-4` (16px) → `sm:p-6` (24px) → `lg:p-8` (32px)
- **Button Padding**: `p-2.5` (10px) → `sm:p-3` (12px) → `lg:p-4` (16px)
- **Icon Margin**: `mb-1.5` (6px) → `sm:mb-2` (8px)

### **Typography Scaling**
- **Text Size**: `text-xs` (12px) → `sm:text-sm` (14px) → `lg:text-base` (16px)
- **Icon Size**: `w-6 h-6` (24px) → `sm:w-8 sm:h-8` (32px)
- **Icon Container**: `w-10 h-10` (40px) → `sm:w-11 sm:h-11` (44px)

## **Header Responsiveness**

### **Page Title Pattern**
```tsx
// ✅ DO: Use responsive header styling
<h1 className="text-xl sm:text-2xl lg:text-3xl font-bold text-indigo-800 mb-4 sm:mb-8 flex flex-wrap items-center justify-center gap-2 text-center">
  Page Title
</h1>
```

### **Header Requirements**
- **`text-xl sm:text-2xl lg:text-3xl`**: Responsive text size
  - Mobile: 20px (1.25rem)
  - Small screens: 24px (1.5rem)
  - Large screens: 30px (1.875rem)
- **`mb-4 sm:mb-8`**: Responsive margin bottom
  - Mobile: 16px (1rem)
  - Small screens+: 32px (2rem)
- **`flex flex-wrap`**: Allows wrapping on mobile
- **`items-center justify-center`**: Centers content
- **`text-center`**: Centers text for mobile

## **Container Responsiveness**

### **Separated Container Pattern (Recommended for Pages with Navigation + Content)**

**CRITICAL**: For pages that have both navigation buttons and data tables/content, use a **separated container pattern** to ensure navigation buttons are full-width with independent responsive padding, while main content remains constrained.

```tsx
// ✅ DO: Use separated container pattern for navigation + content pages
<div className="w-full overflow-x-hidden box-border" style={{ paddingTop: '120px' }}>
  {/* Navigation Section - Full Width, Separate Responsive Container */}
  <div className="w-full px-2 sm:px-3 md:px-4 lg:px-6 xl:px-8 mb-6 sm:mb-8">
    <AdminNavigation />
  </div>
  {/* Main Content Section - Constrained Width */}
  <div className="max-w-7xl mx-auto px-3 sm:px-4 md:px-6 lg:px-8 py-4 sm:py-8">
    {/* Main content (tables, forms, etc.) */}
  </div>
</div>
```

### **Separated Container Requirements**
- **Outer Container**: `w-full overflow-x-hidden box-border` - Full width, prevents horizontal overflow
- **Navigation Container**: 
  - `w-full` - Full width
  - `px-2 sm:px-3 md:px-4 lg:px-6 xl:px-8` - Independent responsive padding (prevents cutoff on mobile)
  - `mb-6 sm:mb-8` - Responsive margin bottom
- **Main Content Container**: 
  - `max-w-7xl mx-auto` - Constrained width, centered
  - `px-3 sm:px-4 md:px-6 lg:px-8` - Responsive horizontal padding
  - `py-4 sm:py-8` - Responsive vertical padding

### **Why Use Separated Containers?**
- **Prevents Cutoff**: Navigation buttons get full width with proper edge padding
- **Independent Responsiveness**: Navigation and content can have different padding strategies
- **Better Mobile Experience**: Navigation buttons don't get constrained by content container max-width
- **Flexibility**: Allows navigation to break out of content constraints

### **Simple Container Pattern (For Navigation-Only Pages)**

```tsx
// ✅ DO: Use simple container for navigation-only pages
<div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-4 sm:py-8" style={{ paddingTop: '120px' }}>
  {/* Navigation buttons */}
</div>
```

### **Container Requirements**
- **`max-w-7xl mx-auto`**: Centers container with max width
- **`px-4 sm:px-6 lg:px-8`**: Responsive horizontal padding
  - Mobile: 16px (1rem)
  - Small screens: 24px (1.5rem)
  - Large screens: 32px (2rem)
- **`py-4 sm:py-8`**: Responsive vertical padding
  - Mobile: 16px (1rem)
  - Small screens+: 32px (2rem)
- **`paddingTop: '120px'`**: Reduced top padding for mobile (was 180px)

## **Accessibility Requirements**

### **Required Attributes**
- **`title`**: Tooltip text for hover state
- **`aria-label`**: Screen reader accessible label (should match button text)
- **Semantic HTML**: Use `<Link>` for navigation

### **Example with Accessibility**
```tsx
<Link
  href="/admin/path"
  className={`... ${colorClasses}`}
  title="Button Label"
  aria-label="Button Label"
>
  {/* Icon and text */}
</Link>
```

## **Best Practices**

### **DO:**
- ✅ Use `grid-cols-2` for mobile (not `grid-cols-1`)
- ✅ Always include responsive breakpoints: `sm:`, `md:`, `lg:`
- ✅ Use responsive padding: `p-2.5 sm:p-3 lg:p-4`
- ✅ Use responsive text sizes: `text-xs sm:text-sm lg:text-base`
- ✅ Use responsive icon sizes: `w-6 h-6 sm:w-8 sm:h-8`
- ✅ Include `px-1` on text to prevent overflow
- ✅ Use `flex-wrap` on headers for mobile
- ✅ Reduce top padding on mobile (`120px` instead of `180px`)
- ✅ Always include `title` and `aria-label` attributes
- ✅ Use consistent gap values: `gap-3 sm:gap-4 lg:gap-6`

### **DON'T:**
- ❌ Use `grid-cols-1` on mobile (wastes space)
- ❌ Skip responsive breakpoints
- ❌ Use fixed sizes without responsive variants
- ❌ Skip `px-1` on text (causes overflow on mobile)
- ❌ Use large top padding on mobile
- ❌ Mix different gap values inconsistently
- ❌ Skip accessibility attributes (`title`, `aria-label`)
- ❌ Use different padding values without responsive variants

## **Responsive Breakpoint Reference**

| Breakpoint | Min Width | Grid Columns | Gap | Card Padding | Button Padding | Text Size | Icon Size |
|----|----|----|----|----|----|----|----|
| **Mobile** (default) | 0px | 2 | 12px | 16px | 10px | 12px | 24px |
| **Small** (`sm:`) | 640px | 2 | 16px | 24px | 12px | 14px | 32px |
| **Medium** (`md:`) | 768px | 3 | 16px | 24px | 12px | 14px | 32px |
| **Large** (`lg:`) | 1024px | 4 | 24px | 32px | 16px | 16px | 32px |

## **Reference Implementation**

- **Admin Home Page**: [`src/app/admin/page.tsx`](mdc:src/app/admin/page.tsx) - Lines 356-393
  - **Grid Layout**: Lines 370-391
  - **Button Cards**: Lines 377-388
  - **Responsive Styling**: Throughout the component
- **Manage Usage Page** (Separated Container Pattern): [`src/app/admin/manage-usage/ManageUsageClient.tsx`](mdc:src/app/admin/manage-usage/ManageUsageClient.tsx)
  - **Separated Container Pattern**: Lines 546-552
  - **Navigation Section** (full-width with independent padding): Lines 547-550
  - **Main Content Section** (constrained width): Line 552
  - **Table Scrollbar Implementation**: Line 642 (user-table-scroll-container)
- **Scrollbar CSS**: [`src/app/globals.css`](mdc:src/app/globals.css) - Lines 130-150
  - **Custom Scrollbar Styling**: `.user-table-scroll-container` class with WebKit and Firefox support

## **Troubleshooting**

### **Buttons Not Aligned on Mobile?**
- Check that `grid-cols-2` is used (not `grid-cols-1`)
- Verify responsive padding: `p-2.5 sm:p-3 lg:p-4`
- Ensure `px-1` is on text span to prevent overflow
- Check container padding: `p-4 sm:p-6 lg:p-8`

### **Text Overflowing on Mobile?**
- Add `px-1` to text span: `className="... px-1"`
- Reduce text size: `text-xs sm:text-sm lg:text-base`
- Check button padding: `p-2.5 sm:p-3 lg:p-4`

### **Icons Too Small/Large on Mobile?**
- Verify responsive icon sizes: `w-6 h-6 sm:w-8 sm:h-8`
- Check icon container: `w-10 h-10 sm:w-11 sm:h-11`
- Ensure proper margin: `mb-1.5 sm:mb-2`

### **Spacing Issues?**
- Verify responsive gaps: `gap-3 sm:gap-4 lg:gap-6`
- Check responsive padding on cards: `p-4 sm:p-6 lg:p-8`
- Ensure consistent spacing across breakpoints

### **Navigation Buttons Cut Off on Right Side?**
- Use **separated container pattern** instead of single container
- Ensure navigation section has independent padding: `px-2 sm:px-3 md:px-4 lg:px-6 xl:px-8`
- Check that navigation container is `w-full` (not constrained by `max-w-7xl`)
- Verify outer container has `overflow-x-hidden` to prevent horizontal scroll

### **Table Not Scrolling on Mobile?**
- Ensure table has `minWidth: '800px'` inline style
- Verify container has `user-table-scroll-container` class
- Check that CSS is added to `globals.css` for scrollbar styling
- Test on actual mobile device (not just browser dev tools)
- Ensure `-webkit-overflow-scrolling: touch` is included for smooth scrolling

## **Related Patterns**

- See [Admin Home Button Groups Pattern](mdc:.cursor/rules/admin_home_button_groups.mdc) for button group styling
- See [Icon Standards](mdc:.cursor/rules/icon_standards.mdc) for icon sizing and styling
- See [MOSC Styling Standards](mdc:.cursor/rules/mosc_styling_standards.mdc) for overall design system
- See [`src/app/admin/page.tsx`](mdc:src/app/admin/page.tsx) for complete implementation

## **Table Scrollbar Pattern (For Data Tables)**

### **Horizontal Scrollbar for Data Tables**

When pages have data tables that need horizontal scrolling on mobile, use the following pattern:

```tsx
// ✅ DO: Use scrollable table container with custom scrollbar styling
<div className="bg-white dark:bg-gray-800 shadow-md rounded-lg overflow-hidden">
  <div className="user-table-scroll-container">
    <table className="min-w-full divide-y divide-gray-300 dark:divide-gray-600 border border-gray-300 dark:border-gray-600" style={{ minWidth: '800px', width: '100%' }}>
      {/* Table content */}
    </table>
  </div>
</div>
```

### **Scrollbar CSS (Add to globals.css)**

```css
/* User Table Scrollbar Styling */
.user-table-scroll-container {
  overflow-x: auto;
  overflow-y: visible;
  scrollbar-width: auto;
  scrollbar-color: #9CA3AF #F3F4F6;
  -webkit-overflow-scrolling: touch;
}

.user-table-scroll-container::-webkit-scrollbar {
  height: 12px;
}

.user-table-scroll-container::-webkit-scrollbar-track {
  background: #F3F4F6;
  border-radius: 6px;
}

.user-table-scroll-container::-webkit-scrollbar-thumb {
  background: #9CA3AF;
  border-radius: 6px;
  border: 2px solid #F3F4F6;
}

.user-table-scroll-container::-webkit-scrollbar-thumb:hover {
  background: #6B7280;
}
```

### **Table Scrollbar Requirements**
- **Container Class**: `user-table-scroll-container` - Custom scrollbar styling
- **Table Min Width**: `minWidth: '800px'` - Ensures scrolling on mobile
- **Touch Scrolling**: `-webkit-overflow-scrolling: touch` - Smooth mobile scrolling
- **Scrollbar Height**: `12px` - Visible but not intrusive
- **Scrollbar Colors**: Gray track (#F3F4F6) with darker thumb (#9CA3AF)
- **Hover Effect**: Darker thumb on hover (#6B7280)

### **Why Use Custom Scrollbar?**
- **Visibility**: Standard scrollbars can be hard to see on mobile
- **Consistent Styling**: Matches design system colors
- **Touch-Friendly**: Proper height for touch interactions
- **Cross-Browser**: Works on both WebKit (Chrome, Safari) and Firefox

## **Complete Example: Navigation + Table Page**

```tsx
export default function ManageUsagePage() {
  return (
    <div className="w-full overflow-x-hidden box-border" style={{ paddingTop: '120px' }}>
      {/* Navigation Section - Full Width, Separate Responsive Container */}
      <div className="w-full px-2 sm:px-3 md:px-4 lg:px-6 xl:px-8 mb-6 sm:mb-8">
        <AdminNavigation />
      </div>
      
      {/* Main Content Section - Constrained Width */}
      <div className="max-w-7xl mx-auto px-3 sm:px-4 md:px-6 lg:px-8 py-4 sm:py-8">
        <h1 className="text-xl sm:text-2xl lg:text-3xl font-bold mb-4 sm:mb-6">
          Page Title
        </h1>
        
        {/* Data Table with Scrollbar */}
        <div className="bg-white dark:bg-gray-800 shadow-md rounded-lg overflow-hidden">
          <div className="user-table-scroll-container">
            <table className="min-w-full divide-y divide-gray-300 dark:divide-gray-600 border border-gray-300 dark:border-gray-600" style={{ minWidth: '800px', width: '100%' }}>
              <thead>
                {/* Table headers */}
              </thead>
              <tbody>
                {/* Table rows */}
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  );
}
```

## **Summary**

**Key Pattern**: Responsive button groups should use:
- **2 columns on mobile** (`grid-cols-2`) - not 1
- **Responsive sizing** for all elements (icons, text, padding, gaps)
- **Proper spacing** with responsive breakpoints
- **Text overflow prevention** with `px-1` on text spans
- **Reduced top padding** on mobile (`120px` instead of `180px`)
- **Separated containers** for pages with navigation + content (prevents cutoff)
- **Custom scrollbar** for data tables (visible and touch-friendly)

**Container Strategy**:
- **Navigation-only pages**: Use simple `max-w-7xl` container
- **Navigation + content pages**: Use separated container pattern (navigation full-width, content constrained)

This ensures optimal space utilization, proper alignment, touch-friendly interactions, and prevents cutoff issues across all device sizes.

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
