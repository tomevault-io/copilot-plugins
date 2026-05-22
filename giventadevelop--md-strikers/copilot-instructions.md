## dialog-button-styling

> Standard pattern for dialog buttons (AlertDialog Action/Cancel) matching admin action buttons styling


# Dialog Button Styling Pattern

## **Overview**
This rule defines the standard pattern for dialog buttons (AlertDialog Action and Cancel buttons) that match the admin action buttons styling pattern. These buttons provide consistent styling, hover effects, and icon presentation across all dialogs in the application.

## **Problem Solved**
- **Consistent Dialog Button Styling**: Ensures all dialog buttons follow the same visual pattern as admin action buttons
- **Icon Standardization**: Provides consistent icon container and sizing for dialog buttons
- **Hover Effects**: Standardized hover states and transitions matching admin buttons
- **Color Coding**: Semantic color usage for different action types (blue for cancel/keep, red for destructive actions)
- **Accessibility**: Proper styling and visual feedback for user interactions

## **Core Pattern**

### **Dialog Footer Structure**
```tsx
// ✅ DO: Use the standard dialog button pattern
<AlertDialogFooter className="flex flex-row gap-3 sm:gap-4">
  {/* Cancel/Secondary Button */}
  <AlertDialogCancel
    onClick={handleClose}
    className="flex-1 flex-shrink-0 h-14 rounded-xl bg-blue-100 hover:bg-blue-200 flex items-center justify-center gap-3 transition-all duration-300 hover:scale-105"
  >
    <div className="flex-shrink-0 w-10 h-10 rounded-lg bg-blue-200 flex items-center justify-center">
      <svg className="w-6 h-6 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M6 18L18 6M6 6l12 12" />
      </svg>
    </div>
    <span className="font-semibold text-blue-700">Cancel/Keep</span>
  </AlertDialogCancel>

  {/* Confirm/Primary Button */}
  <AlertDialogAction
    onClick={handleConfirm}
    disabled={isLoading}
    className="flex-1 flex-shrink-0 h-14 rounded-xl bg-red-100 hover:bg-red-200 flex items-center justify-center gap-3 transition-all duration-300 hover:scale-105 disabled:opacity-50 disabled:cursor-not-allowed disabled:hover:scale-100"
  >
    <div className="flex-shrink-0 w-10 h-10 rounded-lg bg-red-200 flex items-center justify-center">
      {isLoading ? (
        <svg className="animate-spin w-6 h-6 text-red-600" fill="none" viewBox="0 0 24 24">
          <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4"></circle>
          <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
        </svg>
      ) : (
        <svg className="w-6 h-6 text-red-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
        </svg>
      )}
    </div>
    <span className="font-semibold text-red-700">{isLoading ? 'Processing...' : 'Confirm'}</span>
  </AlertDialogAction>
</AlertDialogFooter>
```

## **Key CSS Properties**

### **Dialog Footer Requirements**
- **`flex flex-row`**: Horizontal layout for buttons
- **`gap-3 sm:gap-4`**: Spacing between buttons (12px on mobile, 16px on desktop)

### **Button Container Requirements**
- **`flex-1`**: Equal width buttons
- **`flex-shrink-0`**: Prevents button from shrinking
- **`h-14`**: Fixed height (56px) for consistent button size
- **`rounded-xl`**: Large border radius (12px) for modern appearance
- **`bg-{color}-100`**: Light background color matching action type
- **`hover:bg-{color}-200`**: Darker background on hover
- **`flex items-center justify-center`**: Centers content horizontally and vertically
- **`gap-3`**: Spacing between icon and text (12px)
- **`transition-all duration-300`**: Smooth transitions for all properties
- **`hover:scale-105`**: Subtle scale effect on hover (5% increase)
- **`disabled:opacity-50 disabled:cursor-not-allowed disabled:hover:scale-100`**: Disabled state styling

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
- **`animate-spin`**: For loading state icons

### **Text Requirements**
- **`font-semibold`**: Bold text for emphasis
- **`text-{color}-700`**: Text color matching action type (darker than icon)

## **Color Coding System**

### **Semantic Colors for Dialog Actions**
- **Blue** (`blue-100/200/600/700`): Cancel, Keep, Secondary actions
- **Red** (`red-100/200/600/700`): Confirm Delete, Destructive actions
- **Green** (`green-100/200/600/700`): Confirm Save, Positive actions
- **Gray** (`gray-100/200/600/700`): Neutral actions

## **Complete Examples**

### **Cancel Subscription Dialog**
```tsx
<AlertDialog open={showCancelDialog} onOpenChange={setShowCancelDialog}>
  <AlertDialogContent>
    <AlertDialogHeader>
      <AlertDialogTitle>Cancel Subscription</AlertDialogTitle>
      <AlertDialogDescription>
        Are you sure you want to cancel your subscription? Your subscription will remain active until the end of the current billing period, after which you will lose access to premium features.
      </AlertDialogDescription>
    </AlertDialogHeader>
    <AlertDialogFooter className="flex flex-row gap-3 sm:gap-4">
      <AlertDialogCancel
        onClick={handleCancelDialogClose}
        className="flex-1 flex-shrink-0 h-14 rounded-xl bg-blue-100 hover:bg-blue-200 flex items-center justify-center gap-3 transition-all duration-300 hover:scale-105"
      >
        <div className="flex-shrink-0 w-10 h-10 rounded-lg bg-blue-200 flex items-center justify-center">
          <svg className="w-6 h-6 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M6 18L18 6M6 6l12 12" />
          </svg>
        </div>
        <span className="font-semibold text-blue-700">Keep Subscription</span>
      </AlertDialogCancel>
      <AlertDialogAction
        onClick={handleCancelConfirm}
        disabled={cancelingSubscriptionId !== null}
        className="flex-1 flex-shrink-0 h-14 rounded-xl bg-red-100 hover:bg-red-200 flex items-center justify-center gap-3 transition-all duration-300 hover:scale-105 disabled:opacity-50 disabled:cursor-not-allowed disabled:hover:scale-100"
      >
        <div className="flex-shrink-0 w-10 h-10 rounded-lg bg-red-200 flex items-center justify-center">
          {cancelingSubscriptionId ? (
            <svg className="animate-spin w-6 h-6 text-red-600" fill="none" viewBox="0 0 24 24">
              <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4"></circle>
              <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
            </svg>
          ) : (
            <svg className="w-6 h-6 text-red-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
            </svg>
          )}
        </div>
        <span className="font-semibold text-red-700">{cancelingSubscriptionId ? 'Cancelling...' : 'Cancel Subscription'}</span>
      </AlertDialogAction>
    </AlertDialogFooter>
  </AlertDialogContent>
</AlertDialog>
```

## **Common Icon SVGs for Dialog Buttons**

### **Cancel/Close Icon (X)**
```tsx
<svg className="w-6 h-6 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M6 18L18 6M6 6l12 12" />
</svg>
```

### **Delete/Trash Icon (Destructive Actions)**
```tsx
<svg className="w-6 h-6 text-red-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
</svg>
```

### **Checkmark Icon (Confirm/Save Actions)**
```tsx
<svg className="w-6 h-6 text-green-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M5 13l4 4L19 7" />
</svg>
```

### **Loading Spinner Icon**
```tsx
<svg className="animate-spin w-6 h-6 text-red-600" fill="none" viewBox="0 0 24 24">
  <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4"></circle>
  <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
</svg>
```

## **Best Practices**

### **DO:**
- ✅ Use consistent button styling: `h-14 rounded-xl bg-{color}-100 hover:bg-{color}-200`
- ✅ Always include icon containers with `w-10 h-10 rounded-lg bg-{color}-200`
- ✅ Use semantic color choices (blue for cancel, red for destructive, green for confirm)
- ✅ Include hover effects: `hover:scale-105 transition-all duration-300`
- ✅ Use `flex-1` for equal-width buttons in footer
- ✅ Include loading state with spinner icon
- ✅ Use `gap-3 sm:gap-4` for footer spacing
- ✅ Match icon container background to button hover state
- ✅ Use `disabled:opacity-50 disabled:cursor-not-allowed disabled:hover:scale-100` for disabled state

### **DON'T:**
- ❌ Use default button styling from buttonVariants (override with custom classes)
- ❌ Skip icon containers (always include icon container div)
- ❌ Mix different button styles in the same dialog
- ❌ Use arbitrary colors without semantic meaning
- ❌ Omit hover effects
- ❌ Use different heights or border radius values
- ❌ Skip loading state indicators
- ❌ Use different icon sizes or container sizes

## **Color Reference Table**

| Action Type | Background | Hover | Icon Container | Icon Color | Text Color |
|-------------|------------|-------|----------------|------------|------------|
| Cancel/Keep | `bg-blue-100` | `hover:bg-blue-200` | `bg-blue-200` | `text-blue-600` | `text-blue-700` |
| Delete/Destructive | `bg-red-100` | `hover:bg-red-200` | `bg-red-200` | `text-red-600` | `text-red-700` |
| Confirm/Save | `bg-green-100` | `hover:bg-green-200` | `bg-green-200` | `text-green-600` | `text-green-700` |
| Neutral | `bg-gray-100` | `hover:bg-gray-200` | `bg-gray-200` | `text-gray-600` | `text-gray-700` |

## **Reference Implementations**

- **Cancel Subscription Dialog**: [`src/app/membership/MembershipClient.tsx`](mdc:src/app/membership/MembershipClient.tsx) - Lines 672-690
- **Cancel Subscription Dialog (Plans)**: [`src/app/membership/plans/MembershipPlansClient.tsx`](mdc:src/app/membership/plans/MembershipPlansClient.tsx) - Lines 691-709
- **Admin Action Buttons**: See [Admin Action Buttons Pattern](mdc:.cursor/rules/admin_action_buttons_styling.mdc) for button styling reference

## **Troubleshooting**

### **Buttons Not Scaling on Hover?**
- Check that `hover:scale-105` is included
- Verify `transition-all duration-300` is present
- Ensure parent container doesn't have `overflow-hidden` that clips the scale

### **Icons Not Centered?**
- Verify `flex items-center justify-center` on both button and icon container
- Check that icon container has `flex-shrink-0` to prevent shrinking

### **Colors Not Matching?**
- Ensure color values follow the pattern: `{color}-100` for button, `{color}-200` for icon container, `{color}-600` for icon, `{color}-700` for text
- Use the color reference table above for consistency

### **Buttons Not Equal Width?**
- Verify `flex-1` is included on both buttons
- Check that footer has `flex flex-row` layout

## **Related Patterns**

- See [Admin Action Buttons](mdc:.cursor/rules/admin_action_buttons_styling.mdc) for button styling patterns
- See [Icon Standards](mdc:.cursor/rules/icon_standards.mdc) for icon sizing and styling
- See [MOSC Styling Standards](mdc:.cursor/rules/mosc_styling_standards.mdc) for overall design system

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
