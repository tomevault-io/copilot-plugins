## admin-action-buttons-styling

> Standard pattern for admin action buttons with icons in admin pages and sub-pages


# Admin Action Buttons Pattern

## **Overview**
This rule defines the standard pattern for action buttons with icons used in admin pages and sub-pages. These buttons provide consistent styling, hover effects, and icon presentation across all admin interfaces.

## **Problem Solved**
- **Consistent Styling**: Ensures all admin action buttons follow the same visual pattern
- **Icon Standardization**: Provides consistent icon container and sizing
- **Hover Effects**: Standardized hover states and transitions
- **Color Coding**: Semantic color usage for different action types
- **Accessibility**: Proper ARIA labels and titles for screen readers

## **Core Pattern**

### **Button Structure**
```tsx
// ✅ DO: Use the standard admin action button pattern
<Link
  href="/admin/path/to/resource"
  className="w-full flex-shrink-0 h-14 rounded-xl bg-{color}-100 hover:bg-{color}-200 flex items-center justify-center gap-3 transition-all duration-300 hover:scale-105"
  title="Button Label"
  aria-label="Button Label"
>
  <div className="flex-shrink-0 w-10 h-10 rounded-lg bg-{color}-200 flex items-center justify-center">
    <svg className="w-6 h-6 text-{color}-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
      <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="..." />
    </svg>
  </div>
  <span className="font-semibold text-{color}-700">Button Label</span>
</Link>
```

## **Key CSS Properties**

### **Button Container Requirements**
- **`w-full`**: Full width of parent container
- **`flex-shrink-0`**: Prevents button from shrinking
- **`h-14`**: Fixed height (56px) for consistent button size
- **`rounded-xl`**: Large border radius (12px) for modern appearance
- **`bg-{color}-100`**: Light background color matching action type
- **`hover:bg-{color}-200`**: Darker background on hover
- **`flex items-center justify-center`**: Centers content horizontally and vertically
- **`gap-3`**: Spacing between icon and text (12px)
- **`transition-all duration-300`**: Smooth transitions for all properties
- **`hover:scale-105`**: Subtle scale effect on hover (5% increase)

### **Icon Container Requirements**
- **`flex-shrink-0`**: Prevents icon container from shrinking
- **`w-10 h-10`**: Fixed icon container size (40px × 40px)
- **`rounded-lg`**: Medium border radius (8px) for icon container
- **`bg-{color}-200`**: Darker background than button (creates depth)
- **`flex items-center justify-center`**: Centers icon within container

### **Icon Requirements**
- **`w-6 h-6`**: Icon size (24px × 24px)
- **`text-{color}-600`**: Icon color matching action type
- **`fill="none" stroke="currentColor"`**: Standard SVG styling
- **`viewBox="0 0 24 24"`**: Standard viewBox for Heroicons
- **`strokeWidth={2}`**: Standard stroke width

### **Text Requirements**
- **`font-semibold`**: Bold text for emphasis
- **`text-{color}-700`**: Text color matching action type (darker than icon)

## **Color Coding System**

### **Semantic Colors for Actions**
- **Blue** (`blue-100/200/600/700`): Edit, Update, Modify actions
- **Green** (`green-100/200/600/700`): View, View Details, View Organization actions
- **Gray** (`gray-100/200/600/700`): Settings, Configuration actions
- **Purple** (`purple-100/200/600/700`): Special, Advanced actions
- **Orange** (`orange-100/200/600/700`): Warning, Attention actions
- **Red** (`red-100/200/600/700`): Delete, Remove, Destructive actions
- **Indigo** (`indigo-100/200/600/700`): Navigation, Link actions
- **Teal** (`teal-100/200/600/700`): Analytics, Reports actions

## **Complete Examples**

### **View Organization Button (Green)**
```tsx
<Link
  href={`/admin/tenant-management/organizations/${organization.id}`}
  className="w-full flex-shrink-0 h-14 rounded-xl bg-green-100 hover:bg-green-200 flex items-center justify-center gap-3 transition-all duration-300 hover:scale-105"
  title="View Organization"
  aria-label="View Organization"
>
  <div className="flex-shrink-0 w-10 h-10 rounded-lg bg-green-200 flex items-center justify-center">
    <svg className="w-6 h-6 text-green-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
      <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 21V5a2 2 0 00-2-2H7a2 2 0 00-2 2v16m14 0h2m-2 0h-5m-9 0H3m2 0h5M9 7h1m-1 4h1m4-4h1m-1 4h1m-5 10v-5a1 1 0 011-1h2a1 1 0 011 1v5m-4 0h4" />
    </svg>
  </div>
  <span className="font-semibold text-green-700">View Organization</span>
</Link>
```

### **Edit Settings Button (Blue)**
```tsx
<Link
  href={`/admin/tenant-management/settings/${id}/edit`}
  className="w-full flex-shrink-0 h-14 rounded-xl bg-blue-100 hover:bg-blue-200 flex items-center justify-center gap-3 transition-all duration-300 hover:scale-105"
  title="Edit Settings"
  aria-label="Edit Settings"
>
  <div className="flex-shrink-0 w-10 h-10 rounded-lg bg-blue-200 flex items-center justify-center">
    <svg className="w-6 h-6 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
      <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
    </svg>
  </div>
  <span className="font-semibold text-blue-700">Edit Settings</span>
</Link>
```

### **View Settings Button (Gray)**
```tsx
<Link
  href={`/admin/tenant-management/settings/${settings.id}`}
  className="w-full flex-shrink-0 h-14 rounded-xl bg-gray-100 hover:bg-gray-200 flex items-center justify-center gap-3 transition-all duration-300 hover:scale-105"
  title="View Settings"
  aria-label="View Settings"
>
  <div className="flex-shrink-0 w-10 h-10 rounded-lg bg-gray-200 flex items-center justify-center">
    <svg className="w-6 h-6 text-gray-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
      <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z" />
      <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
    </svg>
  </div>
  <span className="font-semibold text-gray-700">View Settings</span>
</Link>
```

## **Common Icon SVGs for Admin Actions**

### **View/View Details (Building Icon)**
```tsx
<svg className="w-6 h-6 text-green-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 21V5a2 2 0 00-2-2H7a2 2 0 00-2 2v16m14 0h2m-2 0h-5m-9 0H3m2 0h5M9 7h1m-1 4h1m4-4h1m-1 4h1m-5 10v-5a1 1 0 011-1h2a1 1 0 011 1v5m-4 0h4" />
</svg>
```

### **Edit (Pencil Icon)**
```tsx
<svg className="w-6 h-6 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
</svg>
```

### **Settings (Cog Icon)**
```tsx
<svg className="w-6 h-6 text-gray-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z" />
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
</svg>
```

### **Delete (Trash Icon)**
```tsx
<svg className="w-6 h-6 text-red-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
</svg>
```

### **External Link (Arrow Icon)**
```tsx
<svg className="w-6 h-6 text-indigo-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M10 6H6a2 2 0 00-2 2v10a2 2 0 002 2h10a2 2 0 002-2v-4M14 4h6m0 0v6m0-6L10 14" />
</svg>
```

## **Layout Patterns**

### **Button Group Container**
```tsx
// ✅ DO: Use flex container for button groups
<div className="flex flex-col gap-3">
  <Link className="w-full flex-shrink-0 h-14 rounded-xl bg-blue-100 hover:bg-blue-200 flex items-center justify-center gap-3 transition-all duration-300 hover:scale-105" href="...">
    {/* Button content */}
  </Link>
  <Link className="w-full flex-shrink-0 h-14 rounded-xl bg-green-100 hover:bg-green-200 flex items-center justify-center gap-3 transition-all duration-300 hover:scale-105" href="...">
    {/* Button content */}
  </Link>
</div>
```

### **Sidebar Button Placement**
```tsx
// ✅ DO: Place action buttons in sidebar or action panel
<div className="bg-white shadow rounded-lg p-6">
  <h3 className="text-lg font-semibold text-gray-900 mb-4">Actions</h3>
  <div className="flex flex-col gap-3">
    {/* Action buttons */}
  </div>
</div>
```

## **Accessibility Requirements**

### **Required Attributes**
- **`title`**: Tooltip text for hover state
- **`aria-label`**: Screen reader accessible label (should match button text)
- **Semantic HTML**: Use `<Link>` for navigation, `<button>` for actions

### **Example with Accessibility**
```tsx
<Link
  href="/admin/path"
  className="w-full flex-shrink-0 h-14 rounded-xl bg-green-100 hover:bg-green-200 flex items-center justify-center gap-3 transition-all duration-300 hover:scale-105"
  title="View Organization"
  aria-label="View Organization"
>
  {/* Icon and text */}
</Link>
```

## **Best Practices**

### **DO:**
- ✅ Use consistent color coding for action types
- ✅ Always include `title` and `aria-label` attributes
- ✅ Use full width (`w-full`) for sidebar/panel buttons
- ✅ Maintain fixed height (`h-14`) for consistency
- ✅ Use semantic color choices (green for view, blue for edit, etc.)
- ✅ Include hover scale effect (`hover:scale-105`)
- ✅ Use smooth transitions (`transition-all duration-300`)
- ✅ Match icon container background to button hover state

### **DON'T:**
- ❌ Mix different button styles on the same page
- ❌ Use arbitrary colors without semantic meaning
- ❌ Skip accessibility attributes (`title`, `aria-label`)
- ❌ Use different heights or border radius values
- ❌ Omit hover effects
- ❌ Use different icon sizes or container sizes
- ❌ Mix inline styles with Tailwind classes

## **Color Reference Table**

| Action Type | Color | Background | Hover | Icon Container | Icon Color | Text Color |
|-------------|-------|------------|-------|----------------|------------|------------|
| View/Details | Green | `bg-green-100` | `hover:bg-green-200` | `bg-green-200` | `text-green-600` | `text-green-700` |
| Edit/Update | Blue | `bg-blue-100` | `hover:bg-blue-200` | `bg-blue-200` | `text-blue-600` | `text-blue-700` |
| Settings/Config | Gray | `bg-gray-100` | `hover:bg-gray-200` | `bg-gray-200` | `text-gray-600` | `text-gray-700` |
| Delete/Remove | Red | `bg-red-100` | `hover:bg-red-200` | `bg-red-200` | `text-red-600` | `text-red-700` |
| Navigation/Link | Indigo | `bg-indigo-100` | `hover:bg-indigo-200` | `bg-indigo-200` | `text-indigo-600` | `text-indigo-700` |
| Analytics/Reports | Teal | `bg-teal-100` | `hover:bg-teal-200` | `bg-teal-200` | `text-teal-600` | `text-teal-700` |
| Special/Advanced | Purple | `bg-purple-100` | `hover:bg-purple-200` | `bg-purple-200` | `text-purple-600` | `text-purple-700` |
| Warning/Attention | Orange | `bg-orange-100` | `hover:bg-orange-200` | `bg-orange-200` | `text-orange-600` | `text-orange-700` |

## **Reference Implementations**

- **View Organization Button**: [`src/app/admin/tenant-management/settings/[id]/page.tsx`](mdc:src/app/admin/tenant-management/settings/[id]/page.tsx) - Lines 222-234
- **Edit Settings Button**: [`src/app/admin/tenant-management/settings/[id]/page.tsx`](mdc:src/app/admin/tenant-management/settings/[id]/page.tsx) - Lines 207-220

## **Troubleshooting**

### **Button Not Scaling on Hover?**
- Check that `hover:scale-105` is included
- Verify `transition-all duration-300` is present
- Ensure parent container doesn't have `overflow-hidden` that clips the scale

### **Icon Not Centered?**
- Verify `flex items-center justify-center` on both button and icon container
- Check that icon container has `flex-shrink-0` to prevent shrinking

### **Colors Not Matching?**
- Ensure color values follow the pattern: `{color}-100` for button, `{color}-200` for icon container, `{color}-600` for icon, `{color}-700` for text
- Use the color reference table above for consistency

## **Related Patterns**

- See [Icon Standards](mdc:.cursor/rules/icon_standards.mdc) for icon sizing and styling
- See [MOSC Styling Standards](mdc:.cursor/rules/mosc_styling_standards.mdc) for overall design system
- See [`src/app/admin/tenant-management/settings/[id]/page.tsx`](mdc:src/app/admin/tenant-management/settings/[id]/page.tsx) for complete implementation examples

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
