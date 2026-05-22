## icons-buttons-styles

> Icon styling standards and patterns for consistent iconography across the application


# Icon Standards and Styling Guide

This guide defines the standard patterns for displaying icons across the application, ensuring visual consistency and proper sizing.

---

## **Icon Library**

- **Primary Library:** Inline SVG icons using Heroicons pattern (Tailwind CSS compatible)
- **ViewBox:** Always use `viewBox="0 0 24 24"` for consistent scaling
- **Stroke Style:** Use `fill="none" stroke="currentColor"` with `strokeWidth={2}` for outline icons
- **Fill Style:** Use `fill="currentColor"` for solid icons when needed

---

## **Standard Icon Sizes**

### **Large Feature Icons (Event Cards, Feature Sections)**
- **Container:** `w-14 h-14` (56px × 56px)
- **Icon:** `w-10 h-10` (40px × 40px)
- **Container Style:** `rounded-xl` (12px border radius)
- **Background:** Colored backgrounds (e.g., `bg-blue-100`, `bg-green-100`, `bg-purple-100`)
- **Icon Color:** Matching colored text (e.g., `text-blue-500`, `text-green-500`, `text-purple-500`)
- **Hover Effect:** `group-hover:scale-110 transition-transform duration-300`

**Example:**
```tsx
// ✅ DO: Use large feature icon pattern
<div className="flex-shrink-0 w-14 h-14 rounded-xl bg-blue-100 flex items-center justify-center group-hover:scale-110 transition-transform duration-300">
  <svg className="w-10 h-10 text-blue-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
  </svg>
</div>
```

### **Medium Action Icons (Buttons, Action Items)**
- **Container:** `w-10 h-10` (40px × 40px)
- **Icon:** `w-6 h-6` (24px × 24px) - **CRITICAL: Always use inline SVG, never react-icons**
- **Container Style:** `rounded-lg` (8px border radius)
- **Background:** Colored backgrounds with hover states
- **Icon Color:** Matching colored text (e.g., `text-blue-600` for edit, `text-red-600` for delete)
- **SVG Attributes:** `fill="none" stroke="currentColor" viewBox="0 0 24 24"` with `strokeWidth={2}`
- **Hover Effect:** `hover:scale-110 transition-all duration-300`

**Example:**
```tsx
// ✅ DO: Use medium action icon pattern with inline SVG
<button
  className="flex-shrink-0 w-10 h-10 rounded-lg bg-blue-100 hover:bg-blue-200 flex items-center justify-center transition-all duration-300 hover:scale-110"
  title="Edit Item"
  aria-label="Edit Item"
  type="button"
>
  <svg className="w-6 h-6 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
  </svg>
</button>
```

**Media Gallery Grid Buttons (Medium Action Icons):**
```tsx
// ✅ DO: Use inline SVG icons in media gallery grid tiles
<div className="p-4 pt-0 flex justify-end gap-2">
  {/* Edit Button */}
  <button
    onClick={() => handleEditClick(item)}
    className="flex-shrink-0 w-10 h-10 rounded-lg bg-blue-100 hover:bg-blue-200 flex items-center justify-center transition-all duration-300 hover:scale-110"
    title="Edit Media"
    aria-label="Edit Media"
    type="button"
  >
    <svg className="w-6 h-6 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
      <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
    </svg>
  </button>
  {/* Delete Button */}
  <button
    onClick={() => handleDelete(item)}
    className="flex-shrink-0 w-10 h-10 rounded-lg bg-red-100 hover:bg-red-200 flex items-center justify-center transition-all duration-300 hover:scale-110"
    title="Delete Media"
    aria-label="Delete Media"
    type="button"
  >
    <svg className="w-6 h-6 text-red-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
      <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
    </svg>
  </button>
</div>
```

### **Small Inline Icons (Text, Lists)**
- **Icon:** `w-4 h-4` (16px × 16px) or `w-5 h-5` (20px × 20px)
- **No container needed** - inline with text
- **Color:** Inherit text color or use semantic colors

**Example:**
```tsx
// ✅ DO: Use small inline icon pattern
<button className="flex items-center gap-2">
  <svg className="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 9l-7 7-7-7" />
  </svg>
  <span>Read More</span>
</button>
```

---

## **Color Palette for Icons**

### **Semantic Colors**
- **Date/Calendar:** `bg-blue-100` / `text-blue-500`
- **Time/Clock:** `bg-green-100` / `text-green-500`
- **Location/Map:** `bg-purple-100` / `text-purple-500`
- **Calendar/Add to Calendar:** `bg-orange-100` / `text-orange-500`
- **View/Details:** `bg-green-100` / `text-green-700` (table buttons) or `bg-white/20` / `text-white` (on colored backgrounds)
- **Copy:** `bg-blue-100` / `text-blue-700`
- **Navigate/External Link:** `bg-green-100` / `text-green-700`
- **Edit:** `bg-blue-100` / `text-blue-500` (table buttons) or `bg-blue-100` / `text-blue-600` (full-width buttons)
- **Delete:** `bg-red-100` / `text-red-500`
- **Create/Add:** `bg-blue-100` / `text-blue-600` (icon) / `text-blue-700` (text)
- **Toggle Active:** `bg-green-100` / `text-green-500` (active) or `bg-gray-100` / `text-gray-500` (inactive)

---

## **Icon Patterns by Context**

### **Event Card Icons**
```tsx
// Date icon
<div className="flex-shrink-0 w-14 h-14 rounded-xl bg-blue-100 flex items-center justify-center group-hover:scale-110 transition-transform duration-300">
  <svg className="w-10 h-10 text-blue-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
  </svg>
</div>

// Time icon
<div className="flex-shrink-0 w-14 h-14 rounded-xl bg-green-100 flex items-center justify-center group-hover:scale-110 transition-transform duration-300">
  <svg className="w-10 h-10 text-green-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
  </svg>
</div>

// Location icon
<div className="flex-shrink-0 w-14 h-14 rounded-xl bg-purple-100 flex items-center justify-center group-hover:scale-110 transition-transform duration-300">
  <svg className="w-10 h-10 text-purple-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z" />
    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" />
  </svg>
</div>
```

### **Action Button Icons (Admin Tables)**
```tsx
// Edit icon (Table Action Button)
<Link
  href={`/admin/tenant-management/settings/${id}/edit`}
  className="flex-shrink-0 w-14 h-14 rounded-xl bg-blue-100 hover:bg-blue-200 flex items-center justify-center transition-all duration-300 hover:scale-110"
  title="Edit settings"
  aria-label="Edit settings"
>
  <svg className="w-10 h-10 text-blue-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
  </svg>
</Link>

// View icon (Table Action Button)
<Link
  href={`/admin/tenant-management/settings/${id}`}
  className="flex-shrink-0 w-14 h-14 rounded-xl bg-green-100 hover:bg-green-200 flex items-center justify-center transition-all duration-300 hover:scale-110"
  title="View details"
  aria-label="View details"
>
  <svg className="w-10 h-10 text-green-700" fill="none" stroke="currentColor" viewBox="0 0 24 24">
    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z" />
  </svg>
</Link>

// Delete icon (Table Action Button)
<button
  onClick={() => handleDelete(id)}
  className="flex-shrink-0 w-14 h-14 rounded-xl bg-red-100 hover:bg-red-200 flex items-center justify-center transition-all duration-300 hover:scale-110"
  title="Delete settings"
  aria-label="Delete settings"
>
  <svg className="w-10 h-10 text-red-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
  </svg>
</button>

// Toggle active icon
<button className={`flex-shrink-0 w-14 h-14 rounded-xl flex items-center justify-center transition-all duration-300 hover:scale-110 ${
  isActive ? 'bg-green-100 hover:bg-green-200' : 'bg-gray-100 hover:bg-gray-200'
}`}>
  {isActive ? (
    <svg className="w-10 h-10 text-green-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
      <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M5 13l4 4L19 7" />
    </svg>
  ) : (
    <svg className="w-10 h-10 text-gray-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
      <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M6 18L18 6M6 6l12 12" />
    </svg>
  )}
</button>
```

### **Full-Width Admin Action Buttons (With Icon + Text)**
```tsx
// ✅ DO: Use full-width button with icon container and text label
<Link
  href="/admin/tenant-management/settings/new"
  className="flex-shrink-0 h-14 rounded-xl bg-blue-100 hover:bg-blue-200 flex items-center justify-center gap-3 transition-all duration-300 hover:scale-105 px-6"
  title="Create New Settings"
  aria-label="Create New Settings"
>
  <div className="flex-shrink-0 w-10 h-10 rounded-lg bg-blue-200 flex items-center justify-center">
    <svg className="w-6 h-6 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
      <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 4v16m8-8H4" />
    </svg>
  </div>
  <span className="font-semibold text-blue-700">Create New Settings</span>
</Link>
```

**Full-Width Button Requirements:**
- **`flex-shrink-0`**: Prevents button from shrinking
- **`h-14`**: Fixed height (56px) for consistent button size
- **`rounded-xl`**: Large border radius (12px) for modern appearance
- **`bg-{color}-100 hover:bg-{color}-200`**: Light background with darker hover state
- **`flex items-center justify-center`**: Centers content horizontally and vertically
- **`gap-3`**: Spacing between icon container and text (12px)
- **`transition-all duration-300`**: Smooth transitions for all properties
- **`hover:scale-105`**: Subtle scale effect on hover (5% increase)
- **`px-6`**: Horizontal padding (24px) for button content
- **Icon Container**: `w-10 h-10 rounded-lg bg-{color}-200` (40px × 40px, darker than button background)
- **Icon**: `w-6 h-6 text-{color}-600` (24px × 24px)
- **Text**: `font-semibold text-{color}-700` (bold, darker than icon)

---

## **Common Icon SVGs**

### **Calendar/Date**
```tsx
<svg className="w-10 h-10 text-blue-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
</svg>
```

### **Clock/Time**
```tsx
<svg className="w-10 h-10 text-green-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
</svg>
```

### **Location/Map Pin**
```tsx
<svg className="w-10 h-10 text-purple-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z" />
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" />
</svg>
```

### **Edit/Pencil**
```tsx
// Large icon (Table Action Buttons - w-14 h-14 container)
<svg className="w-10 h-10 text-blue-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
</svg>

// Medium icon (Media Gallery Grid - w-10 h-10 container)
<svg className="w-6 h-6 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
</svg>
```

### **Delete/Trash**
```tsx
// Large icon (Table Action Buttons - w-14 h-14 container)
<svg className="w-10 h-10 text-red-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
</svg>

// Medium icon (Media Gallery Grid - w-10 h-10 container)
<svg className="w-6 h-6 text-red-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
</svg>
```

### **Checkmark/Active**
```tsx
<svg className="w-10 h-10 text-green-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M5 13l4 4L19 7" />
</svg>
```

### **X/Close/Inactive**
```tsx
<svg className="w-10 h-10 text-gray-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M6 18L18 6M6 6l12 12" />
</svg>
```

### **View/Eye (Details)**
```tsx
<svg className="w-10 h-10 text-green-700" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z" />
</svg>
```

### **Add/Plus (Create)**
```tsx
<svg className="w-6 h-6 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 4v16m8-8H4" />
</svg>
```

### **Copy**
```tsx
<svg className="w-6 h-6 text-blue-700" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z" />
</svg>
```

### **External Link/Navigate**
```tsx
<svg className="w-6 h-6 text-green-700" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M9 20l-5.447-2.724A1 1 0 013 16.382V5.618a1 1 0 011.447-.894L9 7m0 13l6-3m-6 3V7m6 10l4.553 2.276A1 1 0 0021 18.382V7.618a1 1 0 00-.553-.894L15 4m0 13V4m0 0L9 7" />
</svg>
```

---

## **Best Practices**

### **DO:**
- ✅ Use consistent container sizes (`w-14 h-14` for large square buttons, `h-14` for full-width buttons, `w-10 h-10` for medium)
- ✅ Use matching icon sizes (`w-10 h-10` for large square buttons, `w-6 h-6` for full-width buttons and medium buttons)
- ✅ **Always use inline SVG icons** - Never use react-icons or other icon libraries for Medium Action Icons
- ✅ Apply semantic colors consistently (blue for date/edit/create, green for time/view, purple for location, red for delete)
- ✅ Include hover effects (`hover:scale-110` for square buttons, `hover:scale-105` for full-width buttons)
- ✅ Use `rounded-xl` for large containers and buttons, `rounded-lg` for icon containers within buttons
- ✅ Always include `viewBox="0 0 24 24"` for proper scaling
- ✅ Use `strokeWidth={2}` for consistent stroke thickness
- ✅ Include `title` and `aria-label` attributes on all interactive buttons/links
- ✅ Use `transition-all duration-300` for smooth animations
- ✅ Use `fill="none" stroke="currentColor"` for outline icons

### **DON'T:**
- ❌ **Never use react-icons for Medium Action Icons** - Always use inline SVG
- ❌ Mix icon libraries (don't use react-icons alongside inline SVGs)
- ❌ Use inconsistent sizes (don't mix `w-8 h-8` with `w-10 h-10`)
- ❌ Skip hover effects on interactive icons
- ❌ Use different border radius values inconsistently
- ❌ Forget to include `onClick={(e) => e.stopPropagation()}` on checkboxes/inputs inside clickable containers

---

## **Responsive Considerations**

- Icons maintain their size across breakpoints
- Container sizes remain consistent (`w-14 h-14`, `w-10 h-10`)
- Hover effects work on both desktop and touch devices
- Icons scale proportionally with their containers

---

## **Accessibility**

- Always include `aria-label` or `title` attributes on icon-only buttons
- Ensure sufficient color contrast (WCAG AA minimum)
- Icons should have visible focus states when keyboard navigable
- Use semantic HTML (buttons, links) rather than divs for interactive icons

---

## **Admin Page Button Patterns**

### **Table Action Buttons (Square Icon-Only)**
- **Container**: `w-14 h-14` (56px × 56px) - Square button
- **Border Radius**: `rounded-xl` (12px)
- **Background**: `bg-{color}-100 hover:bg-{color}-200`
- **Icon Size**: `w-10 h-10` (40px × 40px)
- **Icon Color**: `text-{color}-500` or `text-{color}-700` (depending on action)
- **Hover Effect**: `hover:scale-110 transition-all duration-300`
- **Use Cases**: Edit, View, Delete actions in admin tables

### **Full-Width Action Buttons (Icon + Text)**
- **Height**: `h-14` (56px) - Fixed height
- **Border Radius**: `rounded-xl` (12px)
- **Background**: `bg-{color}-100 hover:bg-{color}-200`
- **Layout**: `flex items-center justify-center gap-3`
- **Padding**: `px-6` (horizontal padding)
- **Icon Container**: `w-10 h-10 rounded-lg bg-{color}-200` (darker than button background)
- **Icon Size**: `w-6 h-6` (24px × 24px)
- **Icon Color**: `text-{color}-600`
- **Text**: `font-semibold text-{color}-700`
- **Hover Effect**: `hover:scale-105 transition-all duration-300`
- **Use Cases**: "Create New", "Add", primary actions in page headers

## **References**

- Events page: [`src/app/events/page.tsx`](mdc:src/app/events/page.tsx) - Lines 979-1044 for event card icons
- Membership plans: [`src/components/admin/membership/MembershipPlanList.tsx`](mdc:src/components/admin/membership/MembershipPlanList.tsx) - Action button icons
- Tenant Settings Page: [`src/app/admin/tenant-management/settings/page.tsx`](mdc:src/app/admin/tenant-management/settings/page.tsx) - Lines 94-106 for full-width button, Lines 278-308 for table action buttons
- Tenant Settings List: [`src/app/admin/tenant-management/components/TenantSettingsList.tsx`](mdc:src/app/admin/tenant-management/components/TenantSettingsList.tsx) - Lines 278-308 for table action buttons
- Admin Media Page: [`src/app/admin/media/page.tsx`](mdc:src/app/admin/media/page.tsx) - Lines 810-832 for media gallery grid buttons with inline SVG icons
- Heroicons: https://heroicons.com/ (source for SVG paths)

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
