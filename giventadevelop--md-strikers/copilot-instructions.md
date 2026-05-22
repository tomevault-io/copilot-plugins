## admin-home-button-groups-styling

> Standard pattern for admin home button groups with grid layout for navigation buttons in admin pages and sub-pages


# Admin Home Button Groups Pattern

## **Overview**
This rule defines the standard pattern for navigation button groups displayed in a grid layout across admin pages and sub-pages. These button groups provide consistent styling, responsive grid layout, and hover effects matching the admin home page design.

## **Problem Solved**
- **Consistent Grid Layout**: Ensures all admin navigation button groups use the same responsive grid system
- **Card-Style Buttons**: Provides consistent card appearance with shadows and rounded corners
- **Icon Standardization**: Provides consistent icon container and sizing (14x14 container, 10x10 icon)
- **Hover Effects**: Standardized hover states with scale transforms and background color changes
- **Responsive Design**: Ensures buttons adapt properly across mobile, tablet, and desktop breakpoints
- **Accessibility**: Proper ARIA labels and titles for screen readers

## **Core Pattern**

### **Grid Container**
```tsx
// ✅ DO: Use responsive grid layout matching admin home page
<div className="w-full mb-8">
  <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
    {/* Button cards */}
  </div>
</div>
```

### **Button Card Structure**
```tsx
// ✅ DO: Use card-style button pattern
<Link
  href="/admin/path/to/resource"
  className="flex flex-col items-center justify-center bg-{color}-50 hover:bg-{color}-100 text-{color}-800 rounded-lg shadow-md p-4 text-xs transition-all group"
  title="Button Label"
  aria-label="Button Label"
>
  <div className="flex-shrink-0 w-14 h-14 rounded-xl bg-{color}-100 flex items-center justify-center mb-3 group-hover:scale-110 transition-transform duration-300">
    <IconComponent className="w-10 h-10 text-{color}-500" />
  </div>
  <span className="font-semibold text-center leading-tight">Button Label</span>
</Link>
```

## **Key CSS Properties**

### **Grid Container Requirements**
- **`w-full`**: Full width of parent container
- **`mb-8`**: Standard margin bottom (32px) for spacing
- **`grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4`**: Responsive grid
  - Mobile: 1 column
  - Small screens (640px+): 2 columns
  - Medium screens (768px+): 3 columns
  - Large screens (1024px+): 4 columns
- **`gap-4`**: Consistent gap between grid items (16px)

### **Button Card Requirements**
- **`flex flex-col items-center justify-center`**: Centers content vertically and horizontally
- **`bg-{color}-50`**: Light background color matching action type
- **`hover:bg-{color}-100`**: Darker background on hover
- **`text-{color}-800`**: Text color matching action type
- **`rounded-lg`**: Medium border radius (8px) for card appearance
- **`shadow-md`**: Medium shadow for depth
- **`p-3`**: Padding (12px) for card content - reduced by 25% from original p-4
- **`text-xs`**: Small text size for labels
- **`transition-all`**: Smooth transitions for all properties
- **`group`**: Enables group hover effects on child elements

### **Icon Container Requirements**
- **`flex-shrink-0`**: Prevents icon container from shrinking
- **`w-11 h-11`**: Fixed icon container size (44px × 44px) - reduced by 25% from original w-14 h-14
- **`rounded-xl`**: Large border radius (12px) for icon container
- **`bg-{color}-100`**: Background color matching button hover state
- **`flex items-center justify-center`**: Centers icon within container
- **`mb-2`**: Margin bottom (8px) for spacing between icon and text - reduced by 25% from original mb-3
- **`group-hover:scale-110`**: Scales up 10% when parent card is hovered
- **`transition-transform duration-300`**: Smooth scale animation

### **Icon Requirements**
- **`w-8 h-8`**: Icon size (32px × 32px) - reduced by 25% from original w-10 h-10
- **`text-{color}-500`**: Icon color matching action type (medium shade)

### **Text Requirements**
- **`font-semibold`**: Bold text for emphasis
- **`text-center`**: Centers text horizontally
- **`leading-tight`**: Tighter line height for compact display

## **Color Coding System**

### **CRITICAL: Unique Color Requirement**
- **NO TWO BUTTONS IN THE SAME BUTTON GROUP CAN HAVE THE SAME BACKGROUND COLOR**
- **ALL GRAY COLORS ARE PROHIBITED** - Never use `gray`, `slate`, `stone`, `zinc`, or `neutral` colors for button backgrounds
- Each button in a button group must have a unique, vibrant color to ensure visual distinction
- This requirement applies to all admin pages and subpages that display button groups
- When adding new buttons to a group, ensure the color is not already used by another button in that group
- Standard color assignments:
  - **Admin Home**: Always use `blue` (not gray)
  - **Manage Usage**: Always use `indigo` (not blue, to differentiate from Admin Home)
  - All other buttons: Use unique colors from available palette (green, teal, purple, violet, orange, pink, rose, lime, yellow, fuchsia, cyan, amber, emerald, sky, red, or custom colors)

### **Semantic Colors for Navigation**
- **Blue** (`blue-50/100/500/800`): Admin Home (standard - replaces gray)
- **Indigo** (`indigo-50/100/500/800`): Manage Usage, User Management (standard - replaces blue for Manage Usage)
- **Green** (`green-50/100/500/800`): Manage Events, Calendar Actions
- **Yellow** (`yellow-50/100/500/800`): Media Files, Media Management
- **Purple** (`purple-50/100/500/800`): Ticket Types, Tags, Categories
- **Teal** (`teal-50/100/500/800`): Tickets, Transactions, Analytics
- **Pink** (`pink-50/100/500/800`): Discount Codes, Promotions
- **Orange** (`orange-50/100/500/800`): Warnings, Attention Items
- **Indigo** (`indigo-50/100/500/800`): Settings, Configuration
- **Cyan** (`cyan-50/100/500/800`): Media Management, Organizations
- **Rose** (`rose-50/100/500/800`): Membership Subscriptions
- **Lime** (`lime-50/100/500/800`): Email Addresses
- **Amber** (`amber-50/100/500/800`): Event Sponsors, Executive Committee
- **Emerald** (`emerald-50/100/500/800`): Global Contacts, Global Performers
- **Violet** (`violet-50/100/500/800`): Test Stripe
- **Fuchsia** (`fuchsia-50/100/500/800`): Event Sponsors
- **Sky** (`sky-50/100/500/800`): Tenant Settings
- **Slate** (`slate-50/100/500/800`): Organizations
- **Stone** (`stone-50/100/500/800`): Global Contacts
- **Zinc** (`zinc-50/100/500/800`): Global Emails
- **Neutral** (`neutral-50/100/500/800`): Global Directors
- **Red** (`red-50/100/500/800`): Test CRUD, Destructive Actions

## **Complete Example**

### **Navigation Button Group**
```tsx
{/* Responsive Button Group */}
<div className="w-full mb-8">
  <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
    <Link
      href="/admin"
      className="flex flex-col items-center justify-center bg-gray-50 hover:bg-gray-100 text-gray-800 rounded-lg shadow-md p-4 text-xs transition-all group"
      title="Admin Home"
      aria-label="Admin Home"
    >
      <div className="flex-shrink-0 w-14 h-14 rounded-xl bg-gray-100 flex items-center justify-center mb-3 group-hover:scale-110 transition-transform duration-300">
        <FaHome className="w-10 h-10 text-gray-500" />
      </div>
      <span className="font-semibold text-center leading-tight">Admin Home</span>
    </Link>
    <Link
      href="/admin/manage-usage"
      className="flex flex-col items-center justify-center bg-blue-50 hover:bg-blue-100 text-blue-800 rounded-lg shadow-md p-4 text-xs transition-all group"
      title="Manage Usage"
      aria-label="Manage Usage"
    >
      <div className="flex-shrink-0 w-14 h-14 rounded-xl bg-blue-100 flex items-center justify-center mb-3 group-hover:scale-110 transition-transform duration-300">
        <FaUsers className="w-10 h-10 text-blue-500" />
      </div>
      <span className="font-semibold text-center leading-tight">Manage Usage</span>
    </Link>
    <Link
      href="/admin/manage-events"
      className="flex flex-col items-center justify-center bg-green-50 hover:bg-green-100 text-green-800 rounded-lg shadow-md p-4 text-xs transition-all group"
      title="Manage Events"
      aria-label="Manage Events"
    >
      <div className="flex-shrink-0 w-14 h-14 rounded-xl bg-green-100 flex items-center justify-center mb-3 group-hover:scale-110 transition-transform duration-300">
        <FaCalendarAlt className="w-10 h-10 text-green-500" />
      </div>
      <span className="font-semibold text-center leading-tight">Manage Events</span>
    </Link>
  </div>
</div>
```

## **Layout Patterns**

### **Full Button Group Example**
```tsx
{/* Responsive Button Group */}
<div className="w-full mb-8">
  <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
    {/* Admin Home */}
    <Link href="/admin" className="flex flex-col items-center justify-center bg-gray-50 hover:bg-gray-100 text-gray-800 rounded-lg shadow-md p-4 text-xs transition-all group" title="Admin Home" aria-label="Admin Home">
      <div className="flex-shrink-0 w-14 h-14 rounded-xl bg-gray-100 flex items-center justify-center mb-3 group-hover:scale-110 transition-transform duration-300">
        <FaHome className="w-10 h-10 text-gray-500" />
      </div>
      <span className="font-semibold text-center leading-tight">Admin Home</span>
    </Link>

    {/* Manage Usage */}
    <Link href="/admin/manage-usage" className="flex flex-col items-center justify-center bg-blue-50 hover:bg-blue-100 text-blue-800 rounded-lg shadow-md p-4 text-xs transition-all group" title="Manage Usage" aria-label="Manage Usage">
      <div className="flex-shrink-0 w-14 h-14 rounded-xl bg-blue-100 flex items-center justify-center mb-3 group-hover:scale-110 transition-transform duration-300">
        <FaUsers className="w-10 h-10 text-blue-500" />
      </div>
      <span className="font-semibold text-center leading-tight">Manage Usage</span>
    </Link>

    {/* Manage Media Files */}
    <Link href={`/admin/events/${eventId}/media/list`} className="flex flex-col items-center justify-center bg-yellow-50 hover:bg-yellow-100 text-yellow-800 rounded-lg shadow-md p-4 text-xs transition-all group" title="Manage Media Files" aria-label="Manage Media Files">
      <div className="flex-shrink-0 w-14 h-14 rounded-xl bg-yellow-100 flex items-center justify-center mb-3 group-hover:scale-110 transition-transform duration-300">
        <FaPhotoVideo className="w-10 h-10 text-yellow-500" />
      </div>
      <span className="font-semibold text-center leading-tight">Manage Media Files</span>
    </Link>

    {/* Additional buttons... */}
  </div>
</div>
```

## **Common Icon Components**

### **Using React Icons (react-icons/fa)**
```tsx
import { FaHome, FaUsers, FaCalendarAlt, FaPhotoVideo, FaTags, FaTicketAlt, FaPercent } from 'react-icons/fa';

// Example usage
<FaHome className="w-10 h-10 text-gray-500" />
<FaUsers className="w-10 h-10 text-blue-500" />
<FaCalendarAlt className="w-10 h-10 text-green-500" />
<FaPhotoVideo className="w-10 h-10 text-yellow-500" />
<FaTags className="w-10 h-10 text-purple-500" />
<FaTicketAlt className="w-10 h-10 text-teal-500" />
<FaPercent className="w-10 h-10 text-pink-500" />
```

## **Responsive Breakpoints**

### **Grid Column Behavior**
- **`grid-cols-1`** (mobile, default): 1 column, full width buttons
- **`sm:grid-cols-2`** (640px+): 2 columns, side-by-side on small tablets
- **`md:grid-cols-3`** (768px+): 3 columns, better use of medium screens
- **`lg:grid-cols-4`** (1024px+): 4 columns, optimal for desktop

### **Mobile Considerations**
- Buttons remain fully functional on mobile
- Touch targets are large enough (56px icon container + padding)
- Text remains readable at `text-xs` size
- Hover effects work on touch devices via `group-hover`

## **Accessibility Requirements**

### **Required Attributes**
- **`title`**: Tooltip text for hover state (desktop)
- **`aria-label`**: Screen reader accessible label (should match button text)
- **Semantic HTML**: Use `<Link>` for navigation

### **Example with Accessibility**
```tsx
<Link
  href="/admin/path"
  className="flex flex-col items-center justify-center bg-gray-50 hover:bg-gray-100 text-gray-800 rounded-lg shadow-md p-4 text-xs transition-all group"
  title="Admin Home"
  aria-label="Admin Home"
>
  <div className="flex-shrink-0 w-14 h-14 rounded-xl bg-gray-100 flex items-center justify-center mb-3 group-hover:scale-110 transition-transform duration-300">
    <FaHome className="w-10 h-10 text-gray-500" />
  </div>
  <span className="font-semibold text-center leading-tight">Admin Home</span>
</Link>
```

## **Best Practices**

### **DO:**
- ✅ Use consistent grid layout: `grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4`
- ✅ Always include `title` and `aria-label` attributes
- ✅ Use card-style buttons with `rounded-lg shadow-md p-4`
- ✅ Maintain consistent icon container size (`w-14 h-14`)
- ✅ Use semantic color choices (gray for home, blue for users, green for events, etc.)
- ✅ Include group hover effects (`group` and `group-hover:scale-110`)
- ✅ Use smooth transitions (`transition-all` and `transition-transform duration-300`)
- ✅ Match icon container background to button hover state (`bg-{color}-100`)

### **DON'T:**
- ❌ Use different grid layouts on different pages
- ❌ Mix card-style buttons with full-width vertical buttons
- ❌ Use arbitrary colors without semantic meaning
- ❌ **Use duplicate colors within the same button group** (CRITICAL: Each button must have a unique color)
- ❌ **Use gray colors** (`gray`, `slate`, `stone`, `zinc`, `neutral`) for button backgrounds (CRITICAL: All gray colors are prohibited)
- ❌ Skip accessibility attributes (`title`, `aria-label`)
- ❌ Use different icon container sizes (always `w-14 h-14`)
- ❌ Omit hover effects or use different hover animations
- ❌ Mix different gap values (always use `gap-4`)
- ❌ Use different padding values (always use `p-4`)

## **Color Reference Table**

| Action Type | Background | Hover | Icon Container | Icon Color | Text Color |
|-------------|------------|-------|----------------|------------|------------|
| Admin Home | `bg-gray-50` | `hover:bg-gray-100` | `bg-gray-100` | `text-gray-500` | `text-gray-800` |
| Manage Usage | `bg-blue-50` | `hover:bg-blue-100` | `bg-blue-100` | `text-blue-500` | `text-blue-800` |
| Manage Events | `bg-green-50` | `hover:bg-green-100` | `bg-green-100` | `text-green-500` | `text-green-800` |
| Media Files | `bg-yellow-50` | `hover:bg-yellow-100` | `bg-yellow-100` | `text-yellow-500` | `text-yellow-800` |
| Ticket Types | `bg-purple-50` | `hover:bg-purple-100` | `bg-purple-100` | `text-purple-500` | `text-purple-800` |
| Tickets | `bg-teal-50` | `hover:bg-teal-100` | `bg-teal-100` | `text-teal-500` | `text-teal-800` |
| Discount Codes | `bg-pink-50` | `hover:bg-pink-100` | `bg-pink-100` | `text-pink-500` | `text-pink-800` |

## **Reference Implementations**

- **Admin Home Page**: [`src/app/admin/page.tsx`](mdc:src/app/admin/page.tsx) - Lines 291-312
- **Events List Page**: [`src/app/admin/manage-events/page.tsx`](mdc:src/app/admin/manage-events/page.tsx) - Lines 381-415
- **Tickets List Page**: [`src/app/admin/events/[id]/tickets/list/page.tsx`](mdc:src/app/admin/events/[id]/tickets/list/page.tsx) - Lines 189-220
- **Ticket Types List Page**: [`src/app/admin/events/[id]/ticket-types/list/page.tsx`](mdc:src/app/admin/events/[id]/ticket-types/list/page.tsx) - Lines 46-78
- **Media List Page**: [`src/app/admin/events/[id]/media/list/page.tsx`](mdc:src/app/admin/events/[id]/media/list/page.tsx) - Lines 668-693
- **Discount Codes List**: [`src/app/admin/events/[id]/discount-codes/list/DiscountCodeListClient.tsx`](mdc:src/app/admin/events/[id]/discount-codes/list/DiscountCodeListClient.tsx) - Lines 144-169

## **Troubleshooting**

### **Buttons Not Scaling on Hover?**
- Check that `group` class is on the Link element
- Verify `group-hover:scale-110` is on the icon container
- Ensure `transition-transform duration-300` is present on icon container
- Check that parent container doesn't have `overflow-hidden` that clips the scale

### **Grid Not Responsive?**
- Verify grid classes: `grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4`
- Check that Tailwind responsive breakpoints are configured correctly
- Ensure container has sufficient width for columns to display

### **Icons Not Centered?**
- Verify `flex items-center justify-center` on both Link and icon container
- Check that icon container has `flex-shrink-0` to prevent shrinking
- Ensure icon component has correct size classes (`w-10 h-10`)

### **Colors Not Matching?**
- Ensure color values follow the pattern: `{color}-50` for button, `{color}-100` for icon container, `{color}-500` for icon, `{color}-800` for text
- Use the color reference table above for consistency

## **Related Patterns**

- See [Admin Action Buttons](mdc:.cursor/rules/admin_action_buttons.mdc) for full-width vertical action buttons
- See [Icon Standards](mdc:.cursor/rules/icon_standards.mdc) for icon sizing and styling
- See [MOSC Styling Standards](mdc:.cursor/rules/mosc_styling_standards.mdc) for overall design system
- See [`src/app/admin/page.tsx`](mdc:src/app/admin/page.tsx) for the canonical implementation

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
