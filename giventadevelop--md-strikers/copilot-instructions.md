## admin-toggle-switch-styling

> Standard pattern for toggle switch components used in admin pages for filtering and state toggling


# Admin Toggle Switch Styling Pattern

## **Overview**
This rule defines the standard pattern for toggle switch components used in admin pages for filtering and state toggling. The pattern provides consistent styling, smooth animations, and clear visual feedback for binary state changes.

## **Problem Solved**
- **Consistent Toggle UI**: Ensures all toggle switches use the same visual pattern across admin pages
- **Smooth Animations**: Standardized transition effects and state changes
- **Clear Visual Feedback**: Color-coded states and icons for immediate understanding
- **Accessibility**: Proper ARIA labels and focus states
- **Responsive Design**: Works consistently across different screen sizes

## **Core Pattern**

### **Toggle Switch Container**
```tsx
// ✅ DO: Use the standard toggle switch container pattern
<div className="flex justify-center items-center gap-4 mt-6">
  {/* Left Label */}
  {/* Toggle Switch Button */}
  {/* Right Label */}
</div>
```

### **Toggle Switch Button Structure**
```tsx
// ✅ DO: Use the standard toggle switch button pattern
<button
  onClick={() => setState(!state)}
  className={`relative inline-flex h-10 w-16 items-center rounded-full transition-all duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 hover:scale-105 ${
    state
      ? 'bg-blue-500 focus:ring-blue-500'
      : 'bg-purple-500 focus:ring-purple-500'
    }`}
  title={state ? 'Switch to Off State' : 'Switch to On State'}
  aria-label={state ? 'Switch to Off State' : 'Switch to On State'}
>
  <span
    className={`inline-flex items-center justify-center h-8 w-8 transform rounded-full bg-white transition-transform duration-300 shadow-md ${
      state ? 'translate-x-7' : 'translate-x-1'
    }`}
  >
    {/* Icon based on state */}
  </span>
</button>
```

## **Key CSS Properties**

### **Container Requirements**
- **`flex justify-center items-center`**: Centers toggle switch horizontally and vertically
- **`gap-4`**: Spacing between labels and switch (16px)
- **`mt-6`**: Top margin (24px) for spacing above toggle

### **Button Requirements**
- **`relative`**: Enables absolute positioning of thumb
- **`inline-flex`**: Inline flex layout
- **`h-10`**: Fixed height (40px) for toggle track
- **`w-16`**: Fixed width (64px) for toggle track
- **`items-center`**: Centers thumb vertically
- **`rounded-full`**: Fully rounded track (pill shape)
- **`transition-all duration-300`**: Smooth transitions for all properties
- **`focus:outline-none focus:ring-2 focus:ring-offset-2`**: Focus ring for accessibility
- **`hover:scale-105`**: Subtle scale effect on hover (5% increase)
- **State Colors**:
  - **Active/On State**: `bg-blue-500 focus:ring-blue-500`
  - **Inactive/Off State**: `bg-purple-500 focus:ring-purple-500`

### **Thumb (Slider) Requirements**
- **`inline-flex items-center justify-center`**: Centers icon within thumb
- **`h-8 w-8`**: Fixed thumb size (32px × 32px)
- **`transform rounded-full`**: Fully rounded thumb
- **`bg-white`**: White background for thumb
- **`transition-transform duration-300`**: Smooth slide animation
- **`shadow-md`**: Medium shadow for depth
- **Position**:
  - **Active/On State**: `translate-x-7` (moves to right, 28px)
  - **Inactive/Off State**: `translate-x-1` (moves to left, 4px)

### **Label Requirements**
- **`text-lg font-semibold`**: Large, bold text
- **`transition-colors duration-300`**: Smooth color transitions
- **Active State**: `text-purple-600` or `text-blue-600` (vibrant color)
- **Inactive State**: `text-purple-300` or `text-blue-300` (muted color)

### **Icon Requirements**
- **`w-5 h-5`**: Icon size (20px × 20px)
- **Color**: Matches button state color (`text-blue-600` or `text-purple-600`)
- **SVG**: Use Heroicons pattern with `fill="none" stroke="currentColor"`

## **Complete Example**

### **Future/Past Events Toggle**
```tsx
{/* Event Filter Toggle */}
<div className="flex justify-center items-center gap-4 mt-6">
  <span className={`text-lg font-semibold transition-colors duration-300 ${!showPastEvents ? 'text-purple-600' : 'text-purple-300'}`}>
    Future Events
  </span>
  <button
    onClick={() => setShowPastEvents(!showPastEvents)}
    className={`relative inline-flex h-10 w-16 items-center rounded-full transition-all duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 hover:scale-105 ${
      showPastEvents
        ? 'bg-blue-500 focus:ring-blue-500'
        : 'bg-purple-500 focus:ring-purple-500'
    }`}
    title={showPastEvents ? 'Show Future Events' : 'Show Past Events'}
    aria-label={showPastEvents ? 'Show Future Events' : 'Show Past Events'}
  >
    <span
      className={`inline-flex items-center justify-center h-8 w-8 transform rounded-full bg-white transition-transform duration-300 shadow-md ${showPastEvents ? 'translate-x-7' : 'translate-x-1'}`}
    >
      {showPastEvents ? (
        <svg className="w-5 h-5 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
      ) : (
        <svg className="w-5 h-5 text-purple-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
        </svg>
      )}
    </span>
  </button>
  <span className={`text-lg font-semibold transition-colors duration-300 ${showPastEvents ? 'text-blue-600' : 'text-blue-300'}`}>
    Past Events
  </span>
</div>
```

## **Color Coding System**

### **Standard Color Pairs**
- **Purple/Blue**: Future Events (purple) ↔ Past Events (blue)
- **Green/Red**: Active (green) ↔ Inactive (red)
- **Blue/Gray**: Enabled (blue) ↔ Disabled (gray)
- **Teal/Orange**: On (teal) ↔ Off (orange)

### **Color Usage Pattern**
- **Left Label (Off State)**: Uses primary color when inactive, muted when active
- **Right Label (On State)**: Uses secondary color when active, muted when inactive
- **Toggle Track**: Uses secondary color when active, primary color when inactive
- **Thumb Icon**: Matches track color (blue-600 or purple-600)

## **Common Icon Patterns**

### **Clock Icon (Past/Time-based)**
```tsx
<svg className="w-5 h-5 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
</svg>
```

### **Calendar Icon (Future/Scheduled)**
```tsx
<svg className="w-5 h-5 text-purple-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
</svg>
```

### **Checkmark Icon (Active/Enabled)**
```tsx
<svg className="w-5 h-5 text-green-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M5 13l4 4L19 7" />
</svg>
```

### **X Icon (Inactive/Disabled)**
```tsx
<svg className="w-5 h-5 text-red-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M6 18L18 6M6 6l12 12" />
</svg>
```

## **State Management Pattern**

### **React State Hook**
```tsx
const [toggleState, setToggleState] = useState(false);

// Toggle handler
const handleToggle = () => {
  setToggleState(!toggleState);
};
```

### **Usage in useEffect**
```tsx
useEffect(() => {
  // Reload data when toggle state changes
  loadData();
}, [toggleState]);
```

## **Accessibility Requirements**

### **Required Attributes**
- **`title`**: Tooltip text for hover state (should change based on current state)
- **`aria-label`**: Screen reader accessible label (should change based on current state)
- **`focus:ring-2 focus:ring-offset-2`**: Visible focus indicator
- **Semantic HTML**: Use `<button>` element, not `<div>` or `<span>`

### **Example with Accessibility**
```tsx
<button
  onClick={() => setToggleState(!toggleState)}
  className={`... ${toggleState ? 'bg-blue-500' : 'bg-purple-500'}`}
  title={toggleState ? 'Switch to Off State' : 'Switch to On State'}
  aria-label={toggleState ? 'Switch to Off State' : 'Switch to On State'}
  aria-checked={toggleState}
>
  {/* Toggle content */}
</button>
```

## **Best Practices**

### **DO:**
- ✅ Use consistent toggle dimensions: `h-10 w-16` for track, `h-8 w-8` for thumb
- ✅ Always include `title` and `aria-label` attributes
- ✅ Use smooth transitions: `transition-all duration-300`
- ✅ Include hover effects: `hover:scale-105`
- ✅ Use semantic color pairs (purple/blue, green/red, etc.)
- ✅ Match thumb icon color to track color
- ✅ Use `translate-x-7` for active state, `translate-x-1` for inactive state
- ✅ Include focus ring for keyboard navigation
- ✅ Use `rounded-full` for both track and thumb

### **DON'T:**
- ❌ Mix different toggle sizes or styles
- ❌ Skip accessibility attributes (`title`, `aria-label`)
- ❌ Use arbitrary colors without semantic meaning
- ❌ Omit hover effects or transitions
- ❌ Use different transition durations
- ❌ Skip focus states
- ❌ Use `<div>` or `<span>` instead of `<button>` for toggle
- ❌ Use different thumb sizes or positions

## **Color Reference Table**

| State | Track Color | Thumb Icon Color | Left Label Color | Right Label Color |
|----|----|----|----|----|
| **Off/Inactive** | `bg-purple-500` | `text-purple-600` | `text-purple-600` (active) | `text-blue-300` (muted) |
| **On/Active** | `bg-blue-500` | `text-blue-600` | `text-purple-300` (muted) | `text-blue-600` (active) |

## **Reference Implementation**

- **Manage Events Page**: [`src/app/admin/manage-events/page.tsx`](mdc:src/app/admin/manage-events/page.tsx) - Lines 500-532

## **Troubleshooting**

### **Toggle Not Sliding Smoothly?**
- Check that `transition-transform duration-300` is on the thumb span
- Verify `transform` class is included
- Ensure `translate-x-7` (active) and `translate-x-1` (inactive) are correct

### **Colors Not Changing?**
- Verify state is being toggled correctly (`onClick` handler)
- Check that conditional classes use template literals with `${state ? ... : ...}`
- Ensure both track and label colors update based on state

### **Thumb Not Centered?**
- Verify `items-center` is on the button
- Check that thumb has `inline-flex items-center justify-center`
- Ensure thumb size (`h-8 w-8`) matches container height (`h-10`)

### **Focus Ring Not Visible?**
- Check that `focus:ring-2 focus:ring-offset-2` is included
- Verify `focus:ring-{color}-500` matches track color
- Ensure button is focusable (not disabled)

## **Related Patterns**

- See [Admin Action Buttons](mdc:.cursor/rules/admin_action_buttons.mdc) for button styling patterns
- See [Icon Standards](mdc:.cursor/rules/icon_standards.mdc) for icon sizing and styling
- See [MOSC Styling Standards](mdc:.cursor/rules/mosc_styling_standards.mdc) for overall design system
- See [`src/app/admin/manage-events/page.tsx`](mdc:src/app/admin/manage-events/page.tsx) for complete implementation

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
