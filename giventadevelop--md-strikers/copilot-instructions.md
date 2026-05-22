## events-page-filtering-display-rules

> This rule defines the filtering logic, display rules, and "Buy Tickets" button/image display logic for the events listing page (`/events`). It ensures consistent event filtering, recurring event handling, and proper routing for ticket purchases.

# Events Page Filtering and Display Rules

## **Overview**
This rule defines the filtering logic, display rules, and "Buy Tickets" button/image display logic for the events listing page (`/events`). It ensures consistent event filtering, recurring event handling, and proper routing for ticket purchases.

## **Problem Solved**
- **Consistent Event Filtering**: Ensures all events displayed meet specific criteria (active, date-based, recurring event handling)
- **Buy Tickets Display Logic**: Determines when and how to show the "Buy Tickets" image/button
- **Payment Flow Routing**: Routes users to the correct checkout page based on event payment configuration
- **Recurring Event Handling**: Properly filters and displays recurring events showing only the next occurrence

---

## **Event Filtering Rules**

### **1. Active Events Only**
- **Rule**: Only events with `isActive === true` are displayed
- **Backend Query**: `isActive.equals=true`
- **Rationale**: Inactive events should not appear in public listings

### **2. Date-Based Filtering**

#### **Future Events (Default View)**
- **Rule**: Show events where `startDate >= today` (including today)
- **Backend Query**: `startDate.greaterThanOrEqual=today` (YYYY-MM-DD format)
- **Sort Order**: `sort=startDate,asc` (earliest first)
- **Display Logic**: Events happening today or in the future

#### **Past Events (Toggle View)**
- **Rule**: Show events where `endDate < today`
- **Backend Query**: `endDate.lessThan=today` (YYYY-MM-DD format)
- **Sort Order**: `sort=startDate,desc` (most recent first)
- **Display Logic**: Events that have already ended

#### **Date Range Search (Overrides Toggle)**
- **Rule**: If user specifies date range, it overrides Future/Past toggle
- **Backend Query**:
  - `startDate.greaterThanOrEqual=searchDateFrom` (if provided)
  - `startDate.lessThanOrEqual=searchDateTo` (if provided)
- **Priority**: Date range search takes precedence over Future/Past toggle

### **3. Title Search Filter**
- **Rule**: Filter events by title containing search term (case-insensitive)
- **Backend Query**: `title.contains=searchTitle` (trimmed, case-insensitive)
- **Combines With**: Date filtering (both filters apply simultaneously)

### **4. Recurring Event Handling**

#### **Recurring Event Detection**
- **Rule**: Event is considered recurring if `isRecurring === true`
- **Series Identification**: Uses `recurrenceSeriesId` or `parentEventId` or `event.id` as series identifier

#### **Next Occurrence Calculation**
- **Rule**: Calculate next occurrence date using `getNextOccurrenceDate(event, todayDate)`
- **Time Window**: Only show next occurrence if it's within 1 year from today
- **Date Update**: Update event's `startDate` to next occurrence date for display purposes

#### **Series Deduplication**
- **Rule**: Only show one event per recurring series (the one with earliest next occurrence)
- **Logic**:
  - First event from series: Add to map
  - Subsequent events from same series: Compare dates, keep earlier occurrence
- **Skip Child Events**: Skip events with `parentEventId` or `recurrenceSeriesId` but `isRecurring === false`

#### **Recurring Event Filtering Flow**
```typescript
// Process events and filter recurring events to show only next occurrence
eventList.forEach((event) => {
  if (isRecurringEvent(event)) {
    const seriesId = event.recurrenceSeriesId || event.parentEventId || event.id;
    const nextOccurrence = getNextOccurrenceDate(event, todayDate);

    if (!nextOccurrence || nextOccurrence > oneYearFromNow) {
      return; // Skip if no next occurrence or beyond 1 year
    }

    // Update event startDate to next occurrence
    const eventWithNextOccurrence = { ...event, startDate: nextOccurrenceStr };

    // Keep only earliest occurrence per series
    const existingSeriesEvent = recurringSeriesMap.get(seriesId);
    if (!existingSeriesEvent || nextOccurrence < new Date(existingSeriesEvent.startDate!)) {
      recurringSeriesMap.set(seriesId, eventWithNextOccurrence);
    }
  } else {
    // Skip child events (have parentEventId/recurrenceSeriesId but not recurring)
    const seriesId = event.recurrenceSeriesId || event.parentEventId;
    if (seriesId) {
      return; // Skip child event
    }
    // Non-recurring event - add directly
    processedEvents.push(event);
  }
});
```

### **5. Pagination**
- **Backend Fetch Size**: `BACKEND_FETCH_SIZE = 50` (fetch more to account for filtering)
- **Display Size**: `EVENTS_PAGE_SIZE = 20` (display 20 events per page after filtering)
- **Rationale**: Fetch more events than displayed because recurring event filtering reduces count

---

## **Buy Tickets Image/Button Display Rules**

### **Display Conditions**

#### **1. Event Must Be Ticketed**
- **Rule**: `event.admissionType?.toUpperCase() === 'TICKETED'`
- **Case Handling**: Case-insensitive check (handles 'TICKETED', 'ticketed', etc.)
- **Rationale**: Only ticketed events should show Buy Tickets option

#### **2. Event Must Be Upcoming**
- **Rule**: Event date must be today or in the future
- **Date Calculation**:
  ```typescript
  const today = new Date();
  const todayStr = `${today.getFullYear()}-${String(today.getMonth() + 1).padStart(2, '0')}-${String(today.getDate()).padStart(2, '0')}`;
  const eventDateStr = event.startDate?.split('T')[0]; // YYYY-MM-DD
  const isToday = eventDateStr === todayStr;
  const isFuture = eventDateStr > todayStr;
  const isUpcomingLocal = isToday || isFuture;
  ```
- **Rationale**: Past events should not show Buy Tickets option

#### **3. Combined Display Rule**
```typescript
const showBuyTicketsButton =
  event.admissionType?.toUpperCase() === 'TICKETED' &&
  isUpcomingLocal;
```

### **Routing Logic**

#### **Manual Payment Checkout Route**
- **Condition**: Route to manual checkout if:
  - `event.manualPaymentEnabled === true` AND
  - (`event.paymentFlowMode === 'MANUAL_ONLY'` OR `event.paymentFlowMode === 'HYBRID'`)
- **Route**: `/events/${eventId}/manual-checkout`
- **Use Case**: Events configured for fee-free manual payments (Zelle, Venmo, etc.)

#### **Stripe Checkout Route (Default)**
- **Condition**: All other ticketed events
- **Route**: `/events/${eventId}/checkout`
- **Use Case**: Standard Stripe payment flow

#### **Routing Implementation**
```typescript
const checkoutRoute =
  event.manualPaymentEnabled === true &&
  (event.paymentFlowMode === 'MANUAL_ONLY' || event.paymentFlowMode === 'HYBRID')
    ? `/events/${eventId}/manual-checkout`
    : `/events/${eventId}/checkout`;
```

### **Image Specifications**
- **Source**: `/images/buy_tickets_click_here_red.webp`
- **Sizing**:
  - Mobile: `w-[150px] h-[52px]`
  - Desktop: `sm:w-[200px] sm:h-[70px]`
- **Styling**: `object-contain` (maintains aspect ratio, no cropping)
- **Hover Effect**: `hover:scale-105 transition-transform`
- **Accessibility**: `title="Buy Tickets"` and `aria-label="Buy Tickets"`

### **Display Locations**

#### **1. Top Right Corner (Overlay on Event Image)**
- **Position**: `absolute top-4 right-4 lg:top-6 lg:right-6 z-10`
- **Layout**: Vertical stack if both Register and Buy Tickets buttons shown
- **Condition**: Only shown if event is ticketed and upcoming

#### **2. Action Buttons Section (Bottom of Event Card)**
- **Position**: After "See Event Details" button in action buttons flex container
- **Layout**: Horizontal flex layout with other action buttons
- **Condition**: Same as top-right (ticketed and upcoming)

---

## **Complete Display Logic Flow**

### **Step 1: Fetch Events**
```typescript
// Backend query parameters
const queryParams = new URLSearchParams({
  sort: showPastEvents ? 'startDate,desc' : 'startDate,asc',
  page: page.toString(),
  size: BACKEND_FETCH_SIZE.toString(), // 50
  'isActive.equals': 'true',
  // Date filter based on toggle or search
  // Title filter if searchTitle provided
});
```

### **Step 2: Process Recurring Events**
```typescript
// Filter recurring events to show only next occurrence
// Group by series, keep earliest next occurrence
// Skip child events
```

### **Step 3: Limit Display**
```typescript
// Limit to EVENTS_PAGE_SIZE (20) events after filtering
const limitedProcessedEvents = processedEvents.slice(0, EVENTS_PAGE_SIZE);
```

### **Step 4: Render Buy Tickets Button/Image**
```typescript
// For each event:
// 1. Check if ticketed: event.admissionType?.toUpperCase() === 'TICKETED'
// 2. Check if upcoming: isUpcomingLocal (today or future)
// 3. Determine route: manual checkout or Stripe checkout
// 4. Render image with appropriate route
```

---

## **EventDetailsDTO Fields Used**

### **Filtering Fields**
- `isActive?: boolean` - Must be `true` for event to display
- `startDate: string` - Used for date filtering (YYYY-MM-DD format)
- `endDate: string` - Used for past event filtering
- `title: string` - Used for title search filtering
- `isRecurring?: boolean` - Determines if event is recurring
- `recurrenceSeriesId?: number` - Groups recurring events into series
- `parentEventId?: number` - Identifies child events in recurring series

### **Buy Tickets Display Fields**
- `admissionType?: string` - Must be 'TICKETED' (case-insensitive)
- `startDate: string` - Used to determine if event is upcoming
- `manualPaymentEnabled?: boolean` - Determines if manual payment is enabled
- `paymentFlowMode?: 'STRIPE_ONLY' | 'MANUAL_ONLY' | 'HYBRID'` - Determines payment flow type

---

## **Backend Query Parameters Reference**

### **Standard Filters**
- `isActive.equals=true` - Only active events
- `startDate.greaterThanOrEqual=YYYY-MM-DD` - Future events (including today)
- `endDate.lessThan=YYYY-MM-DD` - Past events
- `startDate.lessThanOrEqual=YYYY-MM-DD` - Date range end
- `title.contains=searchTerm` - Title search (case-insensitive)

### **Sorting**
- `sort=startDate,asc` - Future events (earliest first)
- `sort=startDate,desc` - Past events (most recent first)

### **Pagination**
- `page=0` - Page number (0-based)
- `size=50` - Backend fetch size (fetch more than displayed)

---

## **Auto-Switch Logic**

### **Initial Load Behavior**
- **Rule**: On initial load, check both future and past event counts
- **Auto-Switch**: If `futureEventCount === 0` and `pastEventCount > 0`, automatically switch to past events view
- **Rationale**: Better UX - show available events instead of empty future events list

---

## **Best Practices**

### **DO:**
- ✅ Always check `isActive === true` before displaying events
- ✅ Use case-insensitive check for `admissionType` (`toUpperCase()`)
- ✅ Compare dates as strings (YYYY-MM-DD) to avoid timezone issues
- ✅ Handle recurring events by showing only next occurrence
- ✅ Route to manual checkout when `manualPaymentEnabled === true` AND (`MANUAL_ONLY` or `HYBRID`)
- ✅ Fetch more events from backend (`BACKEND_FETCH_SIZE = 50`) than displayed (`EVENTS_PAGE_SIZE = 20`) to account for filtering
- ✅ Use `getNextOccurrenceDate()` utility for recurring event date calculation
- ✅ Skip child events (have `parentEventId`/`recurrenceSeriesId` but `isRecurring === false`)

### **DON'T:**
- ❌ Don't display inactive events (`isActive === false`)
- ❌ Don't show Buy Tickets for non-ticketed events
- ❌ Don't show Buy Tickets for past events
- ❌ Don't parse dates with `new Date()` for comparison (use string comparison)
- ❌ Don't show multiple events from the same recurring series
- ❌ Don't show child events in recurring series
- ❌ Don't route to manual checkout if `paymentFlowMode === 'STRIPE_ONLY'`
- ❌ Don't route to manual checkout if `manualPaymentEnabled === false`

---

## **Reference Implementations**

- **Events Listing Page**: [`src/app/events/page.tsx`](mdc:src/app/events/page.tsx) - Lines 88-327 (fetching/filtering), Lines 1049-1116 (top-right Buy Tickets), Lines 1352-1396 (action buttons Buy Tickets)
- **Event Details Page**: [`src/app/events/[id]/page.tsx`](mdc:src/app/events/[id]/page.tsx) - Lines 513-577 (Buy Tickets display logic)
- **Recurring Event Utilities**: [`src/lib/eventUtils.ts`](mdc:src/lib/eventUtils.ts) - `isRecurringEvent()` and `getNextOccurrenceDate()` functions
- **EventDetailsDTO**: [`src/types/index.ts`](mdc:src/types/index.ts) - EventDetailsDTO interface definition

---

## **Troubleshooting**

### **Events Not Showing?**
- Check `isActive` field - must be `true`
- Check date filtering - verify `startDate`/`endDate` values
- Check recurring event logic - ensure next occurrence is calculated correctly
- Verify backend query parameters match filtering rules

### **Buy Tickets Not Showing?**
- Verify `admissionType === 'TICKETED'` (case-insensitive)
- Check event date - must be today or future (`isUpcomingLocal === true`)
- Verify `startDate` is present and valid

### **Wrong Checkout Route?**
- Check `manualPaymentEnabled` - must be `true` for manual checkout
- Check `paymentFlowMode` - must be `'MANUAL_ONLY'` or `'HYBRID'` for manual checkout
- Verify routing logic matches the rules above

### **Recurring Events Showing Multiple Times?**
- Check `recurrenceSeriesId` or `parentEventId` grouping logic
- Verify `getNextOccurrenceDate()` is calculating correctly
- Ensure child events are being skipped

---

## **Summary**

**Key Rules**:
1. **Active Events Only**: `isActive === true`
2. **Date Filtering**: Future (`startDate >= today`) or Past (`endDate < today`)
3. **Recurring Events**: Show only next occurrence within 1 year, one per series
4. **Buy Tickets Display**: `admissionType === 'TICKETED'` AND event is upcoming (today or future)
5. **Buy Tickets Routing**: Manual checkout if `manualPaymentEnabled === true` AND (`MANUAL_ONLY` or `HYBRID`), otherwise Stripe checkout

**Display Locations**:
- Top-right corner overlay on event image
- Action buttons section at bottom of event card

This ensures consistent, predictable event filtering and Buy Tickets functionality across the application.

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
