## ui-style-guide

> This guide provides standards for creating consistent and maintainable UI components across the application UI

# UI Component Style Guide

This guide provides standards for creating consistent and maintainable UI components across the application UI
.

---

## 1. Page & Content Layout

### Page Container

- **Rule:** Use `max-w-5xl mx-auto px-8 py-8` for main page containers.
- **Purpose:** Enforces a consistent 80% width and center alignment on desktop views.
- **Example:**
  ```tsx
  // ✅ DO: Use consistent page layout
  <div className="max-w-5xl mx-auto px-8 py-8">
    {/* Page content goes here */}
  </div>
  ```

### Content Card

- **Rule:** Use `bg-white rounded-lg shadow-md p-6` for containers that wrap main content sections (tables, forms, etc.).
- **Purpose:** Creates a consistent, elevated card-based layout for content.
- **Example:**
  ```tsx
  // ✅ DO: Use a styled container for content sections
  <div className="bg-white rounded-lg shadow-md p-6">
    {/* Table, list, or form content */}
  </div>
  ```

### Centered Card Grid Layout

- **Rule:** Use CSS modules with flexbox for centered card grids (Featured Guests, Contact Information, Program Directors, Gallery thumbnails, Team members).
- **Purpose:** Ensures cards are perfectly centered regardless of the number of items, preventing left-aligned layouts when items don't fill the full width.
- **Pattern:** Create a CSS module file (e.g., `CenteredCardGrid.module.css`) with:
  - Flexbox container with `justify-content: center`
  - Responsive `max-width` calculations based on number of columns
  - Fixed or calculated widths for card items
- **Example:**
  ```css
  /* Centered Card Grid */
  .centeredCardGrid {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
    width: 100%;
    justify-content: center;
    align-items: flex-start;
    margin: 0 auto;
  }

  /* Desktop: 3 columns */
  @media (min-width: 1024px) {
    .centeredCardGrid {
      max-width: calc(3 * 350px + 2 * 1rem);
    }
    .cardItem {
      width: 350px;
      max-width: 350px;
    }
  }

  /* Tablet: 2 columns */
  @media (min-width: 768px) and (max-width: 1023px) {
    .centeredCardGrid {
      max-width: calc(2 * 350px + 1 * 1rem);
    }
    .cardItem {
      width: calc((100% - 1rem) / 2);
      max-width: calc((100% - 1rem) / 2);
    }
  }

  /* Mobile: 1 column */
  @media (max-width: 767px) {
    .centeredCardGrid {
      max-width: 100%;
    }
    .cardItem {
      width: 100%;
      max-width: 100%;
    }
  }
  ```
- **Usage:**
  ```tsx
  // ✅ DO: Use CSS module for centered card grids
  import cardGridStyles from './CenteredCardGrid.module.css';

  <div className={cardGridStyles.centeredCardGrid}>
    {items.map((item) => (
      <div key={item.id} className={cardGridStyles.cardItem}>
        {/* Card content */}
      </div>
    ))}
  </div>

  // ❌ DON'T: Use standard grid without centering
  <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
    {/* Cards will be left-aligned when not filling full width */}
  </div>
  ```
- **References:**
  - See `src/app/events/[id]/CenteredCardGrid.module.css` for implementation
  - See `src/app/events/[id]/GalleryThumbnails.module.css` for gallery pattern
  - See `src/components/TeamSection.module.css` for team member pattern

---

## 2. Forms

### Input Fields

- **Rule:** Use the following classes for consistent input field styling.
- **Example:**
  ```tsx
  // ✅ DO: Use consistent input field styling
  <input
    type="text"
    className="mt-1 block w-full border border-gray-400 rounded-xl focus:border-blue-500 focus:ring-blue-500 px-4 py-3 text-base"
  />
  ```

### Labels

- **Rule:** Use the following classes for consistent label styling.
- **Example:**
  ```tsx
  // ✅ DO: Use consistent label styling
  <label className="block text-sm font-medium text-gray-700">
    Field Label
  </label>
  ```

### Checkboxes

- **Rule:** Use the `custom-checkbox` implementation for a larger, more visible checkbox with a custom tick mark.
- **Click Handling:** Always include `onClick={(e) => e.stopPropagation()}` on the `input` to prevent unintended event bubbling, especially inside clickable table rows or containers.
- **Example:**
  ```tsx
  // ✅ DO: Use consistent checkbox styling with stopPropagation
  <label className="flex flex-col items-center">
    <span className="relative flex items-center justify-center">
      <input
        type="checkbox"
        className="custom-checkbox"
        checked={isChecked}
        onChange={handleChange}
        onClick={(e) => e.stopPropagation()}
      />
      <span className="custom-checkbox-tick">
        {isChecked && (
          <svg className="w-6 h-6 text-black" fill="none" stroke="currentColor" strokeWidth="4" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" d="M5 13l5 5L19 7" />
          </svg>
        )}
      </span>
    </span>
    <span className="mt-2 text-xs text-center select-none break-words max-w-[6rem]">Checkbox Label</span>
  </label>
  ```

- **Checkbox Group Layout:**
  ```tsx
  // ✅ DO: Use a CSS grid for checkbox group layout
  <div className="custom-grid-table mt-4" style={{ display: 'grid', gridTemplateColumns: 'repeat(3, 1fr)', gap: '0.5rem' }}>
    {/* Checkbox items */}
  </div>
  ```

- **Required CSS (`globals.css`):**
  ```css
  .custom-checkbox {
    @apply h-6 w-6 border-2 border-gray-400 rounded-lg cursor-pointer appearance-none relative bg-white;
  }
  .custom-checkbox:checked {
    @apply bg-blue-600 border-blue-600;
  }
  .custom-checkbox-tick {
    @apply absolute inset-0 flex items-center justify-center pointer-events-none;
  }
  ```

---

## 3. Buttons & Icons

### Button Styling

- **Primary Action (Save/Submit):** Blue background.
- **Secondary Action (Cancel):** Light teal background to be non-destructive.
- **Example:**
  ```tsx
  // ✅ DO: Use consistent button styling with icons
  <button type="submit" className="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded-md flex items-center gap-2">
    <FaSave />
    Save Changes
  </button>

  <button type="button" className="bg-teal-100 hover:bg-teal-200 text-teal-800 px-4 py-2 rounded-md flex items-center gap-2">
    <FaBan />
    Cancel
  </button>
  ```

### Standard Action Icons

- **Rule:** Use the following icons from `react-icons/fa` for all common actions to ensure a consistent visual language.
- **Implementation:** Use `.icon-btn` with a modifier (e.g., `.icon-btn-delete`) for standalone icon buttons.

| Action         | Icon           | Usage                                        |
| -------------- | -------------- | -------------------------------------------- |
| **Cancel/Abort**| `<FaBan />`      | Stop a current action (e.g., in a modal).    |
| **Save**       | `<FaFolderOpen />`| Save or update data.                         |
| **Delete**     | `<FaTrashAlt />` | **MANDATORY.** Never use `<FaTrash />`.        |
| **Edit**       | `<FaEdit />`     | Edit an item.                                |
| **Upload**     | `<FaUpload />`   | Upload a file.                               |

---

## 4. Tooltips

- **Rule:** Use the standardized `DetailsTooltip` component, which renders in a React Portal, for all mouse-over popovers that show detailed information. This prevents the tooltip from being clipped by parent containers with scrollbars.
- **Trigger:** The tooltip should be triggered on hover of specific table cells. Use a debounced handler to prevent flickering.
- **User Guidance:** Always place a descriptive note above the table to inform users about the hover behavior.

### Tooltip Implementation

The implementation involves three parts: a portal-based `DetailsTooltip` component, state management, and hover handlers in the parent component.

```typescript
// 1. The DetailsTooltip Component (renders with createPortal)
function UserDetailsTooltip({ user, anchorRect, onClose }: { user: UserProfileDTO, anchorRect: DOMRect | null, onClose: () => void }) {
  if (!anchorRect) return null;

  // Always show tooltip to the right of the anchor cell, never above the columns
  const spacing = 8;
  const tooltipWidth = 320; // px, adjust as needed
  let top = anchorRect.top;
  let left = anchorRect.right + spacing;

  // Clamp position to stay within the viewport
  const estimatedHeight = 300; // Heuristic average height
  if (top + estimatedHeight > window.innerHeight) {
    top = window.innerHeight - estimatedHeight - spacing;
  }
  if (top < spacing) {
    top = spacing;
  }
  if (left + tooltipWidth > window.innerWidth) {
    left = window.innerWidth - tooltipWidth - spacing;
  }

  const style: React.CSSProperties = {
    position: 'fixed',
    top,
    left,
    zIndex: 9999,
    width: tooltipWidth,
    // ... other styles: background, border, shadow, etc.
  };

  return ReactDOM.createPortal(
    <div style={style} tabIndex={-1} className="admin-tooltip">
      {/* Sticky, always-visible close button */}
      <div className="sticky top-0 right-0 z-10 bg-white flex justify-end">
        <button
          onClick={onClose}
          className="w-10 h-10 text-2xl bg-red-500 hover:bg-red-600 text-white rounded-full shadow-lg flex items-center justify-center transition-all"
          aria-label="Close tooltip"
        >
          &times;
        </button>
      </div>
      {/* ... Tooltip content ... */}
    </div>,
    document.body
  );
}
```

#### Tooltip Close Button & Positioning Standards
- All tooltips must include a close button (×) in the top-right corner, always visible and fixed above scrollable content using `sticky top-0 right-0 z-10 bg-white flex justify-end`.
- The close button must be large and visually prominent: `w-10 h-10 text-2xl bg-red-500 hover:bg-red-600 text-white rounded-full shadow-lg flex items-center justify-center transition-all`.
- The tooltip must always appear to the right of the hovered cell for the first two columns, never above or covering them. Use `anchorRect.right + spacing` for left, and `anchorRect.top` for top, clamped to the viewport.
- Remove any logic that places the tooltip to the left of the cell.
- The tooltip should only close when the close button is clicked, not on mouse leave or blur.
- This ensures accessibility, usability, and a consistent experience across all pages.

- **References:**
  - Manage usage page: [`/admin/manage-usage`](mdc:http:/localhost:3000/admin/manage-usage)
  - Example file: `src/app/admin/manage-usage/ManageUsageClient.tsx`

---

## 5. Pagination

- **Rule:** Always fetch the true total count of items from the backend for paginated lists.
  - The backend must return the total count in the `x-total-count` response header for paginated GET requests.
  - The proxy handler (`src/lib/proxyHandler.ts`) must forward the `x-total-count` header from the backend to the frontend for all GET requests.
    ```typescript
    // In createProxyHandler, after fetching from the backend:
    if (method === 'GET') {
      const totalCount = apiRes.headers.get('x-total-count');
      if (totalCount) {
        res.setHeader('x-total-count', totalCount);
      }
      const data = await apiRes.json();
      res.status(apiRes.status).json(data);
      return;
    }
    ```
- **Frontend pagination logic must use the value from the `x-total-count` header** to calculate:
  - Total pages: `totalPages = Math.ceil(totalCount / pageSize)`
  - Enable/disable Previous/Next buttons based on the current page and total pages.
  - Display the correct range: "Showing X to Y of Z items".
- **Do not fallback to `rows.length` for total count unless the header is truly missing.**
- **Example of correct usage:**
  See [`/admin/events/[id]/tickets/list`](http://localhost:3000/admin/events/1/tickets/list) for a working implementation that matches the admin dashboard.

### UI/UX Consistency
- **CRITICAL: Always show pagination controls** - Never conditionally hide pagination based on `totalPages > 1` or `totalCount > 0`
- Pagination controls must be visible in ALL states: loading, empty results, with data
- Pagination controls must use:
  - Previous/Next buttons on the left/right.
  - Page status in the center ("Page X of Y").
  - Range and total count below ("Showing X to Y of Z items").
- Use the same button and layout logic as in `src/components/EventList.tsx` and the admin dashboard.
- **Always show grayed out buttons** when navigation is not possible (first page, last page, loading)

### Example Implementation
```tsx
// ALWAYS show pagination controls - never conditionally hide
const totalPages = Math.ceil(totalCount / pageSize);
const isPrevDisabled = currentPage === 0 || loading;
const isNextDisabled = currentPage >= totalPages - 1 || loading;
const startItem = totalCount > 0 ? currentPage * pageSize + 1 : 0;
const endItem = totalCount > 0 ? currentPage * pageSize + Math.min(pageSize, totalCount - currentPage * pageSize) : 0;

// Always render pagination - show in ALL states (loading, empty, with data)
<div className="mt-8">
  <div className="flex justify-between items-center">
    <button
      disabled={isPrevDisabled}
      onClick={handlePrevPage}
      className="px-4 py-2 bg-blue-600 text-white font-semibold rounded-lg shadow hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed flex items-center gap-2 transition-colors"
    >
      <ChevronLeft className="h-5 w-5" />
      Previous
    </button>
    <div className="text-sm font-semibold text-gray-700">
      Page {currentPage + 1} of {totalPages}
    </div>
    <button
      disabled={isNextDisabled}
      onClick={handleNextPage}
      className="px-4 py-2 bg-blue-600 text-white font-semibold rounded-lg shadow hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed flex items-center gap-2 transition-colors"
    >
      Next
      <ChevronRight className="h-5 w-5" />
    </button>
  </div>
  <div className="text-center text-sm text-gray-600 mt-2">
    {totalCount > 0 ? (
      <>Showing <span className="font-medium">{startItem}</span> to <span className="font-medium">{endItem}</span> of{' '}
      <span className="font-medium">{totalCount}</span> items</>
    ) : (
      <div className="flex items-center justify-center gap-2">
        <span>No items found</span>
        <span className="bg-blue-100 text-blue-800 px-2 py-1 rounded-md text-sm font-medium">
          [No items match your criteria]
        </span>
      </div>
    )}
  </div>
</div>
```

### References
- Proxy handler: [`src/lib/proxyHandler.ts`](mdc:src/lib/proxyHandler.ts)
- Example page: [`/admin/events/[id]/tickets/list`](http://localhost:3000/admin/events/1/tickets/list)
- Admin dashboard: [`/admin`](mdc:http:/localhost:3000/admin)
- Gallery implementation: [`src/app/gallery/components/GalleryPagination.tsx`](mdc:src/app/gallery/components/GalleryPagination.tsx)
- CSS: `src/styles/globals.css`
- Example Components: `src/components/EventList.tsx`, `src/app/admin/events/[id]/media/list/page.tsx`, `src/app/admin/manage-usage/page.tsx`

---

## 6. Responsive Button Group Grid

- **Rule:** Use a responsive grid for button groups at the top of admin pages (e.g., event tickets, ticket types) to ensure all navigation/action buttons are always visible, accessible, and visually centered on all screen sizes.

- **Mobile View (default):**
  - The grid uses `grid-cols-1` and is centered  with `justify-items-center mx-auto`.
  - Each button uses `w-48 max-w-xs mx-auto` to be compact and centered, not full width.
  - Use `p-1` and `text-xs` for the button, and `text-base` for the icon.
  - The button group appears as a perfectly centered, vertically stacked set of compact buttons.

- **Tablet/Desktop (sm and up):**
  - Use `sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-6` for the grid.
  - Button padding increases to `sm:p-2`, icon to `sm:text-lg`.

- **General:**
  - Wrap the grid in a `w-full overflow-x-auto` container to allow horizontal scrolling on very small screens.
  - All buttons are always visible, never cut off, and the group is horizontally scrollable if needed.
  - All buttons must be keyboard accessible and have clear focus/hover states.

- **Example Implementation:**
  ```tsx
  <div className="w-full overflow-x-auto">
    <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-6 gap-2 mb-6 justify-items-center mx-auto">
      <Link href="/admin/manage-usage" className="w-48 max-w-xs mx-auto flex flex-col items-center justify-center bg-blue-50 hover:bg-blue-100 text-blue-800 rounded-md shadow p-1 sm:p-2 text-xs sm:text-xs transition-all">
        <FaUsers className="text-base sm:text-lg mb-1 mx-auto" />
        <span className="font-semibold text-center leading-tight">Manage Usage<br />[Users]</span>
      </Link>
      {/* ...other buttons... */}
    </div>
  </div>
  ```
- **References:**
  - Tickets list page: [`/admin/events/[id]/tickets/list`](http://localhost:3000/admin/events/1/tickets/list)
  - Ticket-types list page: [`/admin/events/[id]/ticket-types/list`](http://localhost:3000/admin/events/1/ticket-types/list)
  - Example file: `src/app/admin/events/[id]/tickets/list/page.tsx`
  - Example file: `src/app/admin/events/[id]/ticket-types/list/page.tsx`

---

## 7. Date & Timezone Formatting

### Date Display Standards

- **Rule:** Always display event dates and times using the event's intended timezone, not the user's local timezone.
- **Purpose:** Prevents off-by-one-day errors and ensures all users see the correct event date as intended by organizers.

#### Implementation

- **Use `date-fns-tz` for formatting:**
  - Install with:
    ```bash
    npm install date-fns date-fns-tz
    ```
  - Import and use in your component:
    ```tsx
    import { formatInTimeZone } from 'date-fns-tz';

    // Example usage:
    <span>
      {formatInTimeZone(eventDetails.startDate, eventDetails.timezone, 'EEEE, MMMM d, yyyy (zzz)')}
    </span>
    ```
    - `eventDetails.startDate` should be a string in `YYYY-MM-DD` format.
    - `eventDetails.timezone` should be an IANA timezone string (e.g., `"America/New_York"`).
    - The format string `'EEEE, MMMM d, yyyy (zzz)'` will display:
      `Wednesday, August 7, 2025 (EDT)`

- **Never use `new Date('YYYY-MM-DD')` for display.**
  - This parses as UTC and can cause the date to appear as the previous day in US timezones.

- **Always store and use the IANA timezone name in the DTO/database.**
  - Example: `"America/New_York"`, not `"EST"` or `"PST"`.

#### UI Example

```tsx
<div className="flex items-center gap-2">
  <FaCalendarAlt />
  <span>
    {formatInTimeZone(eventDetails.startDate, eventDetails.timezone, 'EEEE, MMMM d, yyyy (zzz)')}
  </span>
</div>
```

#### References
- See: `src/app/event/success/page.tsx` for a working implementation.
- DTO: `EventDetailsDTO` in `src/types/index.ts` (must include a `timezone: string` field).

---

## 8. Currency & Numerical Formatting

### Currency Display Standards

- **Rule:** Always display currency values with exactly 2 decimal places using `Intl.NumberFormat`.
- **Purpose:** Ensures consistent currency display (e.g., `$0.80` instead of `$0.8`) and prevents confusion.
- **Implementation:**
  - Use `Intl.NumberFormat` with `minimumFractionDigits: 2` and `maximumFractionDigits: 2`.
  - For input fields, format the display value to show 2 decimal places while allowing user input.
  - Use the `formatCurrency` helper from `src/lib/payments/localization.ts` when available.

#### Currency Formatting Function

```tsx
// ✅ DO: Use Intl.NumberFormat with 2 decimal places
const formatCurrency = (amount: number, currency: string = 'USD') => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency,
    minimumFractionDigits: 2,  // Always show 2 decimal places
    maximumFractionDigits: 2,  // Never show more than 2
  }).format(amount);
};

// Example usage:
formatCurrency(0.8, 'USD');   // Returns: "$0.80"
formatCurrency(10, 'USD');     // Returns: "$10.00"
formatCurrency(99.9, 'USD');   // Returns: "$99.90"
```

#### Price Input Field Formatting

- **Rule:** For price input fields, format the display value to show 2 decimal places.
- **Implementation:**
  ```tsx
  // ✅ DO: Format price input display value
  const [displayPrice, setDisplayPrice] = useState<string>('');

  useEffect(() => {
    if (formData.price !== undefined && formData.price !== null) {
      setDisplayPrice(formData.price.toFixed(2));
    }
  }, [formData.price]);

  <input
    type="number"
    name="price"
    value={displayPrice}
    onChange={(e) => {
      const numValue = parseFloat(e.target.value) || 0;
      handleChange({ target: { name: 'price', value: numValue } });
    }}
    step="0.01"
    placeholder="0.00"
  />
  ```

#### Display Price Values

- **Rule:** Always format price values when displaying them in tables, cards, or text.
- **Example:**
  ```tsx
  // ✅ DO: Format price for display
  <td>
    {new Intl.NumberFormat('en-US', {
      style: 'currency',
      currency: plan.currency,
      minimumFractionDigits: 2,
      maximumFractionDigits: 2,
    }).format(plan.price)}
  </td>

  // ❌ DON'T: Display raw price value
  <td>${plan.price}</td>  // May show "$0.8" instead of "$0.80"
  ```

#### References
- Currency formatting helper: [`src/lib/payments/localization.ts`](mdc:src/lib/payments/localization.ts)
- Example usage: [`src/components/admin/membership/MembershipPlanList.tsx`](mdc:src/components/admin/membership/MembershipPlanList.tsx)

---

## References
- **CSS:** `src/styles/globals.css`
- **Example Pages:**
  - `src/app/admin/events/[id]/media/list/page.tsx`
  - `src/app/admin/manage-usage/page.tsx`
  - `src/components/EventList.tsx`

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
