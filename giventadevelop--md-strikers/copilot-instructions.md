## pagination-footer-styling

> Standard pagination footer styling pattern with Previous/Next buttons and page information display


# Pagination Footer Styling Pattern

## **Overview**
This rule defines the standard pattern for pagination footer controls used across admin pages and list components. The pattern includes Previous/Next navigation buttons, page information display, and item count text, all styled consistently with the admin interface design system.

## **Problem Solved**
- **Consistent Pagination UI**: Ensures all pagination footers use the same visual pattern
- **Button Styling**: Standardized Previous/Next button appearance with hover effects
- **Page Information Display**: Consistent page number and item count presentation
- **Accessibility**: Proper ARIA labels and disabled states
- **Responsive Design**: Works across different screen sizes

## **Core Pattern**

### **Pagination Container**
```tsx
// ✅ DO: Use the standard pagination footer container
<div className="mt-8">
  <div className="flex justify-between items-center">
    {/* Previous Button */}
    {/* Page Info */}
    {/* Next Button */}
  </div>
  <div className="text-center mt-3">
    {/* Item Count Text */}
  </div>
</div>
```

### **Previous/Next Button Structure**
```tsx
// ✅ DO: Use the standard pagination button pattern
<button
  onClick={onPrevPage}
  disabled={isPrevDisabled}
  className="px-5 py-2.5 bg-blue-100 hover:bg-blue-200 text-blue-700 font-semibold rounded-lg shadow-sm border-2 border-blue-400 hover:border-blue-500 disabled:bg-blue-100 disabled:border-blue-300 disabled:text-blue-500 disabled:cursor-not-allowed flex items-center gap-2 transition-all duration-300 hover:scale-105 hover:shadow-md"
  title="Previous Page"
  aria-label="Previous Page"
  type="button"
>
  <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2.5} d="M15 19l-7-7 7-7" />
  </svg>
  <span>Previous</span>
</button>
```

## **Key CSS Properties**

### **Pagination Container Requirements**
- **`mt-8`**: Top margin (32px) for spacing above pagination
- **`flex justify-between items-center`**: Horizontal layout with buttons on sides, page info in center
- **`text-center mt-3`**: Centered item count text with spacing (12px)

### **Button Requirements**
- **`px-5 py-2.5`**: Padding (20px horizontal, 10px vertical)
- **`bg-blue-100`**: Light blue background
- **`hover:bg-blue-200`**: Darker blue on hover
- **`text-blue-700`**: Dark blue text color
- **`font-semibold`**: Bold text weight
- **`rounded-lg`**: Large border radius (8px)
- **`shadow-sm`**: Small shadow for depth
- **`border-2 border-blue-400`**: 2px border with medium blue color
- **`hover:border-blue-500`**: Darker border on hover
- **`disabled:bg-blue-100 disabled:border-blue-300 disabled:text-blue-500 disabled:cursor-not-allowed`**: Disabled state styling
- **`flex items-center gap-2`**: Flex layout with icon and text, 8px gap
- **`transition-all duration-300`**: Smooth transitions
- **`hover:scale-105`**: 5% scale increase on hover
- **`hover:shadow-md`**: Medium shadow on hover

### **Page Info Display Requirements**
- **`px-4 py-2`**: Padding (16px horizontal, 8px vertical)
- **`bg-blue-50`**: Very light blue background
- **`border-2 border-blue-300`**: 2px border with light blue color
- **`rounded-lg`**: Large border radius (8px)
- **`shadow-sm`**: Small shadow
- **`text-sm font-bold text-blue-700`**: Small, bold, dark blue text
- **`text-blue-600`**: Medium blue for page numbers

### **Item Count Text Requirements**
- **`inline-flex items-center`**: Inline flex layout
- **`px-4 py-2`**: Padding (16px horizontal, 8px vertical)
- **`bg-blue-50`**: Very light blue background
- **`border-2 border-blue-300`**: 2px border with light blue color
- **`rounded-lg`**: Large border radius (8px)
- **`shadow-sm`**: Small shadow
- **`text-sm text-gray-700`**: Small, gray text
- **`font-bold text-blue-600`**: Bold, medium blue for numbers

## **Icon Specifications**

### **Previous Button Icon (Left Arrow)**
```tsx
<svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2.5} d="M15 19l-7-7 7-7" />
</svg>
```
- **Size**: `w-5 h-5` (20px × 20px)
- **Stroke Width**: `2.5` (thicker for visibility)
- **Color**: Inherits from button text color (`text-blue-700`)

### **Next Button Icon (Right Arrow)**
```tsx
<svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2.5} d="M9 5l7 7-7 7" />
</svg>
```
- **Size**: `w-5 h-5` (20px × 20px)
- **Stroke Width**: `2.5` (thicker for visibility)
- **Color**: Inherits from button text color (`text-blue-700`)

## **Complete Example**

### **Full Pagination Footer**
```tsx
{/* Pagination Controls - Always visible, matching admin page style */}
<div className="mt-8">
  <div className="flex justify-between items-center">
    {/* Previous Button */}
    <button
      onClick={onPrevPage}
      disabled={isPrevDisabled}
      className="px-5 py-2.5 bg-blue-100 hover:bg-blue-200 text-blue-700 font-semibold rounded-lg shadow-sm border-2 border-blue-400 hover:border-blue-500 disabled:bg-blue-100 disabled:border-blue-300 disabled:text-blue-500 disabled:cursor-not-allowed flex items-center gap-2 transition-all duration-300 hover:scale-105 hover:shadow-md"
      title="Previous Page"
      aria-label="Previous Page"
      type="button"
    >
      <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2.5} d="M15 19l-7-7 7-7" />
      </svg>
      <span>Previous</span>
    </button>

    {/* Page Info */}
    <div className="px-4 py-2 bg-blue-50 border-2 border-blue-300 rounded-lg shadow-sm">
      <span className="text-sm font-bold text-blue-700">
        Page <span className="text-blue-600">{displayPage}</span> of <span className="text-blue-600">{totalPages}</span>
      </span>
    </div>

    {/* Next Button */}
    <button
      onClick={onNextPage}
      disabled={isNextDisabled}
      className="px-5 py-2.5 bg-blue-100 hover:bg-blue-200 text-blue-700 font-semibold rounded-lg shadow-sm border-2 border-blue-400 hover:border-blue-500 disabled:bg-blue-100 disabled:border-blue-300 disabled:text-blue-500 disabled:cursor-not-allowed flex items-center gap-2 transition-all duration-300 hover:scale-105 hover:shadow-md"
      title="Next Page"
      aria-label="Next Page"
      type="button"
    >
      <span>Next</span>
      <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2.5} d="M9 5l7 7-7 7" />
      </svg>
    </button>
  </div>

  {/* Item Count Text */}
  <div className="text-center mt-3">
    {totalCount > 0 ? (
      <div className="inline-flex items-center px-4 py-2 bg-blue-50 border-2 border-blue-300 rounded-lg shadow-sm">
        <span className="text-sm text-gray-700">
          Showing <span className="font-bold text-blue-600">{startItem}</span> to <span className="font-bold text-blue-600">{endItem}</span> of <span className="font-bold text-blue-600">{totalCount}</span> events
        </span>
      </div>
    ) : (
      <div className="inline-flex items-center gap-2 px-4 py-2 bg-orange-50 border-2 border-orange-300 rounded-lg shadow-sm">
        <svg className="w-5 h-5 text-orange-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
        <span className="text-sm font-medium text-orange-700">No events found</span>
        <span className="text-sm text-orange-600">[No events match your criteria]</span>
      </div>
    )}
  </div>
</div>
```

## **Disabled State Handling**

### **Button Disabled Logic**
```tsx
// Calculate disabled states
const totalPages = Math.ceil(totalCount / pageSize) || 1;
const isPrevDisabled = currentPageZeroBased === 0 || loading;
const isNextDisabled = currentPageZeroBased >= totalPages - 1 || loading;
```

### **Disabled State Styling**
- **Background**: `disabled:bg-blue-100` (same as normal, no hover change)
- **Border**: `disabled:border-blue-300` (lighter border)
- **Text**: `disabled:text-blue-500` (lighter text)
- **Cursor**: `disabled:cursor-not-allowed` (not-allowed cursor)
- **Hover Effects**: Disabled automatically by browser (no scale, no shadow change)

## **Page Number Calculation**

### **Zero-Based vs One-Based Indexing**
```tsx
// EventList uses 1-based page indexing by default, but manage-events uses 0-based
// Convert to 0-based for calculations if page is 0 (indicating 0-based indexing)
const isZeroBased = page === 0;
const currentPageZeroBased = isZeroBased ? page : page - 1;
const displayPage = isZeroBased ? page + 1 : page; // Display as 1-based

const totalPages = Math.ceil(totalCount / pageSize) || 1;
const startItem = totalCount > 0 ? currentPageZeroBased * pageSize + 1 : 0;
const endItem = totalCount > 0 ? currentPageZeroBased * pageSize + Math.min(pageSize, totalCount - currentPageZeroBased * pageSize) : 0;
```

## **Empty State Pattern**

### **No Items Found Display**
```tsx
<div className="inline-flex items-center gap-2 px-4 py-2 bg-orange-50 border-2 border-orange-300 rounded-lg shadow-sm">
  <svg className="w-5 h-5 text-orange-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
  </svg>
  <span className="text-sm font-medium text-orange-700">No events found</span>
  <span className="text-sm text-orange-600">[No events match your criteria]</span>
</div>
```

- **Background**: `bg-orange-50` (light orange for warning/empty state)
- **Border**: `border-2 border-orange-300` (orange border)
- **Icon**: Info icon in orange (`text-orange-500`)
- **Text**: Orange text colors (`text-orange-700`, `text-orange-600`)

## **Accessibility Requirements**

### **Required Attributes**
- **`title`**: Tooltip text for hover state
- **`aria-label`**: Screen reader accessible label (should match button text)
- **`type="button"`**: Explicit button type
- **`disabled`**: Proper disabled state handling

### **Example with Accessibility**
```tsx
<button
  onClick={onPrevPage}
  disabled={isPrevDisabled}
  className="..."
  title="Previous Page"
  aria-label="Previous Page"
  type="button"
>
  {/* Button content */}
</button>
```

## **Best Practices**

### **DO:**
- ✅ Use consistent button styling: `px-5 py-2.5 bg-blue-100 hover:bg-blue-200 text-blue-700`
- ✅ Always include `title` and `aria-label` attributes
- ✅ Use `border-2` for visible borders
- ✅ Include hover effects: `hover:scale-105 hover:shadow-md`
- ✅ Use `disabled:` variants for disabled states
- ✅ Display page numbers in medium blue (`text-blue-600`)
- ✅ Use `inline-flex` for item count text container
- ✅ Include empty state with orange styling when no items

### **DON'T:**
- ❌ Mix different button styles on the same page
- ❌ Skip accessibility attributes (`title`, `aria-label`)
- ❌ Use different border widths (always use `border-2`)
- ❌ Omit hover effects
- ❌ Use different color schemes for pagination
- ❌ Skip disabled state styling
- ❌ Use different spacing values

## **Color Reference Table**

| Element | Background | Border | Text | Hover Background | Hover Border |
|----|----|----|----|----|----|
| Button (Normal) | `bg-blue-100` | `border-blue-400` | `text-blue-700` | `hover:bg-blue-200` | `hover:border-blue-500` |
| Button (Disabled) | `bg-blue-100` | `border-blue-300` | `text-blue-500` | — | — |
| Page Info | `bg-blue-50` | `border-blue-300` | `text-blue-700` | — | — |
| Item Count | `bg-blue-50` | `border-blue-300` | `text-gray-700` | — | — |
| Empty State | `bg-orange-50` | `border-orange-300` | `text-orange-700` | — | — |

## **Reference Implementation**

- **EventList Component**: [`src/components/EventList.tsx`](mdc:src/components/EventList.tsx) - Lines 601-653
- **Manage Events Page**: [`src/app/admin/manage-events/page.tsx`](mdc:src/app/admin/manage-events/page.tsx) - Uses EventList with pagination

## **Troubleshooting**

### **Buttons Not Scaling on Hover?**
- Check that `hover:scale-105` is included
- Verify `transition-all duration-300` is present
- Ensure parent container doesn't have `overflow-hidden` that clips the scale

### **Disabled State Not Working?**
- Verify `disabled` prop is set correctly
- Check that `disabled:` variants are included in className
- Ensure disabled state logic calculates correctly

### **Page Numbers Not Displaying?**
- Verify page calculation logic (zero-based vs one-based)
- Check that `displayPage` and `totalPages` are calculated correctly
- Ensure `totalCount` and `pageSize` are provided

### **Item Count Text Not Centered?**
- Verify `text-center` is on the container div
- Check that `inline-flex` is used for the item count box
- Ensure parent container has proper width

## **Related Patterns**

- See [Admin Action Buttons](mdc:.cursor/rules/admin_action_buttons.mdc) for button styling patterns
- See [Icon Standards](mdc:.cursor/rules/icon_standards.mdc) for icon sizing and styling
- See [MOSC Styling Standards](mdc:.cursor/rules/mosc_styling_standards.mdc) for overall design system
- See [`src/components/EventList.tsx`](mdc:src/components/EventList.tsx) for complete implementation

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
