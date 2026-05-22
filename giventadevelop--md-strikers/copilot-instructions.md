## future-past-events-toggle-switch

> This rule defines the business logic and UI patterns for the future/past events toggle switch implemented on both admin and public events pages. The toggle allows users to switch between viewing future events (upcoming) and past events (historical), with intelligent auto-switching and informative messaging.

# Future / Past Events Toggle Switch Business Rules

## **Overview**
This rule defines the business logic and UI patterns for the future/past events toggle switch implemented on both admin and public events pages. The toggle allows users to switch between viewing future events (upcoming) and past events (historical), with intelligent auto-switching and informative messaging.

## **Problem Solved**
- **User Experience**: Automatically shows past events when no future events exist, preventing empty "No events found" messages
- **Information Clarity**: Provides clear messaging about event availability and how to use the toggle switch
- **Consistent Behavior**: Ensures the same logic works on both admin and public pages (with appropriate customization)
- **Empty State Handling**: Gracefully handles all scenarios (no events, no future events, no past events)

## **Core Business Rules**

### **1. Auto-Switch on Page Load**
- **Rule**: On initial page load, if there are no future events but past events exist, automatically switch to showing past events
- **When**: Only on initial load (page 0, no search filters applied)
- **Purpose**: Prevents showing "No events found" when past events are available
- **Implementation**: Check both future and past event counts on initial load, then auto-switch if needed

### **2. Event Count Tracking**
- **Rule**: Track both future and past event counts separately to determine appropriate messaging
- **State Variables**: 
  - `futureEventCount`: Number of future events (events with `startDate >= today`)
  - `pastEventCount`: Number of past events (events with `endDate < today`)
  - `hasCheckedInitialLoad`: Flag to ensure initial check happens only once
  - `isAutoSwitching`: Flag to prevent double-loading when auto-switching

### **3. Info Box Messages**

#### **No Events at All (Both Future and Past)**
- **Condition**: `futureEventCount === 0 && pastEventCount === 0`
- **Message**: 
  - **Title**: "There are no events listed yet."
  - **Body**: "Please check back again. New events will appear here once they are created. Please use future / past events switch above."
- **Styling**: Blue info box (`bg-blue-50 border border-blue-200`)
- **Icon**: Info icon (blue)

#### **Showing Past Events (No Future Events Exist)**
- **Condition**: `showPastEvents === true && futureEventCount === 0 && pastEventCount > 0`
- **Message**: "Here is the list of recent events. New future events will be added soon. Please use future / past events switch above."
- **Styling**: Amber info box (`bg-amber-50 border border-amber-200`)
- **Icon**: Info icon (amber)
- **Position**: Above the events table

#### **Showing Future Events (No Future Events Exist)**
- **Condition**: `showPastEvents === false && futureEventCount === 0`
- **Message**:
  - **Title**: "No future events created."
  - **Body**: "Please use future / past events switch above."
- **Styling**: Blue info box (`bg-blue-50 border border-blue-200`)
- **Icon**: Info icon (blue)
- **Admin Pages**: Include "Create New Event" button
- **Public Pages**: No action button (public users cannot create events)

### **4. Date Filtering Logic**

#### **Future Events Filter**
- **Query Parameter**: `startDate.greaterThanOrEqual=${today}`
- **Sort**: `startDate,asc` (earliest first)
- **Definition**: Events that start today or in the future (including today)

#### **Past Events Filter**
- **Query Parameter**: `endDate.lessThan=${today}`
- **Sort**: `startDate,desc` (newest first)
- **Definition**: Events that have already ended (ended before today)

### **5. Date Format**
- **Format**: `YYYY-MM-DD` (ISO date format)
- **Calculation**: `new Date().toISOString().split('T')[0]`
- **Usage**: Used for all date comparisons and API queries

## **Implementation Pattern**

### **State Management**
```tsx
// Track event counts for both future and past
const [futureEventCount, setFutureEventCount] = useState<number | null>(null);
const [pastEventCount, setPastEventCount] = useState<number | null>(null);
const [hasCheckedInitialLoad, setHasCheckedInitialLoad] = useState(false);
const [isAutoSwitching, setIsAutoSwitching] = useState(false);
const [showPastEvents, setShowPastEvents] = useState(false);
```

### **Initial Load Check**
```tsx
// On initial load, check both future and past event counts
if (!hasCheckedInitialLoad && page === 0 && !searchTitle && !searchDateFrom && !searchDateTo) {
  // Check future events count
  const futureParams = {
    sort: 'startDate,asc',
    pageNum: 0,
    pageSize: 1, // Just need count, not data
    startDate: today,
  };
  const { totalCount: futureCount } = await fetchEventsFilteredServer(futureParams);
  setFutureEventCount(futureCount);

  // Check past events count
  const pastParams = {
    sort: 'startDate,desc',
    pageNum: 0,
    pageSize: 1, // Just need count, not data
    endDate: today,
  };
  const { totalCount: pastCount } = await fetchEventsFilteredServer(pastParams);
  setPastEventCount(pastCount);

  setHasCheckedInitialLoad(true);

  // Auto-switch to past events if no future events but past events exist
  if (futureCount === 0 && pastCount > 0) {
    setIsAutoSwitching(true);
    setShowPastEvents(true);
    loadingPastEvents = true; // Load past events data in this same call
  }
}
```

### **Auto-Switch Prevention**
```tsx
// Skip reload if we're currently auto-switching (prevents double-load)
useEffect(() => {
  if (isAutoSwitching) {
    setIsAutoSwitching(false); // Reset flag after skipping
    return;
  }
  // ... rest of load logic
}, [showPastEvents, isAutoSwitching, /* other dependencies */]);
```

## **Info Box Components**

### **No Events Info Box**
```tsx
{/* Info box when there are no events at all (both future and past) */}
{!loading && hasCheckedInitialLoad && futureEventCount === 0 && pastEventCount === 0 && (
  <div className="mb-6 bg-blue-50 border border-blue-200 rounded-lg p-6">
    <div className="flex items-start">
      <div className="flex-shrink-0">
        <svg className="h-6 w-6 text-blue-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
      </div>
      <div className="ml-3">
        <h3 className="text-base font-medium text-blue-800 mb-1">
          There are no events listed yet.
        </h3>
        <p className="text-sm text-blue-700">
          Please check back again. New events will appear here once they are created. Please use future / past events switch above.
        </p>
      </div>
    </div>
  </div>
)}
```

### **Past Events Message Box**
```tsx
{/* Message above table when showing past events because no future events exist */}
{!loading && hasCheckedInitialLoad && showPastEvents && futureEventCount === 0 && pastEventCount > 0 && (
  <div className="mb-4 bg-amber-50 border border-amber-200 rounded-lg p-4">
    <div className="flex items-start">
      <div className="flex-shrink-0">
        <svg className="h-5 w-5 text-amber-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
      </div>
      <div className="ml-3">
        <p className="text-sm font-medium text-amber-800">
          Here is the list of recent events. New future events will be added soon. Please use future / past events switch above.
        </p>
      </div>
    </div>
  </div>
)}
```

### **No Future Events Info Box (Admin)**
```tsx
{/* Info box when showing future events but there are no future events - Admin Page */}
{!loading && hasCheckedInitialLoad && !showPastEvents && futureEventCount === 0 && (
  <div className="mb-6 bg-blue-50 border border-blue-200 rounded-lg p-6">
    <div className="flex items-start">
      <div className="flex-shrink-0">
        <svg className="h-6 w-6 text-blue-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
      </div>
      <div className="ml-3 flex-1">
        <h3 className="text-base font-medium text-blue-800 mb-1">
          No future events created.
        </h3>
        <p className="text-sm text-blue-700 mb-4">
          Please use future / past events switch above.
        </p>
        {/* Create New Event Button - Admin Only */}
        <Link
          href="/admin/events/new"
          className="inline-flex items-center gap-2 h-14 rounded-xl bg-blue-100 hover:bg-blue-200 flex items-center justify-center gap-3 transition-all duration-300 hover:scale-105 px-6"
          title="Create Event"
          aria-label="Create Event"
        >
          <div className="flex-shrink-0 w-10 h-10 rounded-lg bg-blue-200 flex items-center justify-center">
            <svg className="w-6 h-6 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 4v16m8-8H4" />
            </svg>
          </div>
          <span className="font-semibold text-blue-700">Create New Event</span>
        </Link>
      </div>
    </div>
  </div>
)}
```

### **No Future Events Info Box (Public)**
```tsx
{/* Info box when showing future events but there are no future events - Public Page */}
{!loading && hasCheckedInitialLoad && !showPastEvents && futureEventCount === 0 && (
  <div className="mb-6 bg-blue-50 border border-blue-200 rounded-lg p-6">
    <div className="flex items-start">
      <div className="flex-shrink-0">
        <svg className="h-6 w-6 text-blue-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
      </div>
      <div className="ml-3">
        <h3 className="text-base font-medium text-blue-800 mb-1">
          No future events created.
        </h3>
        <p className="text-sm text-blue-700">
          Please use future / past events switch above.
        </p>
        {/* No action button for public pages */}
      </div>
    </div>
  </div>
)}
```

## **Key CSS Properties**

### **Info Box Styling**
- **Container**: `mb-6 bg-blue-50 border border-blue-200 rounded-lg p-6` (blue info box)
- **Container**: `mb-4 bg-amber-50 border border-amber-200 rounded-lg p-4` (amber message box)
- **Icon Size**: `h-6 w-6` (24px × 24px) for main boxes, `h-5 w-5` (20px × 20px) for message boxes
- **Icon Color**: `text-blue-400` (blue boxes), `text-amber-400` (amber boxes)
- **Title**: `text-base font-medium text-blue-800 mb-1` (16px, bold, dark blue)
- **Body**: `text-sm text-blue-700` (14px, medium blue)

### **Button Styling (Admin Only)**
- **Container**: `inline-flex items-center gap-2 h-14 rounded-xl bg-blue-100 hover:bg-blue-200`
- **Icon Container**: `w-10 h-10 rounded-lg bg-blue-200`
- **Icon**: `w-6 h-6 text-blue-600`
- **Text**: `font-semibold text-blue-700`

## **Complete Example: Admin Page Pattern**

```tsx
export default function ManageEventsPage() {
  const [showPastEvents, setShowPastEvents] = useState(false);
  const [futureEventCount, setFutureEventCount] = useState<number | null>(null);
  const [pastEventCount, setPastEventCount] = useState<number | null>(null);
  const [hasCheckedInitialLoad, setHasCheckedInitialLoad] = useState(false);
  const [isAutoSwitching, setIsAutoSwitching] = useState(false);

  async function loadAll(pageNum = 0, checkInitialLoad = false) {
    const today = new Date().toISOString().split('T')[0];
    let loadingPastEvents = showPastEvents;

    // On initial load, check both future and past event counts
    if (checkInitialLoad && !hasCheckedInitialLoad && pageNum === 0 && !searchTitle) {
      // Check future events count
      const { totalCount: futureCount } = await fetchEventsFilteredServer({
        sort: 'startDate,asc',
        pageNum: 0,
        pageSize: 1,
        startDate: today,
      });
      setFutureEventCount(futureCount);

      // Check past events count
      const { totalCount: pastCount } = await fetchEventsFilteredServer({
        sort: 'startDate,desc',
        pageNum: 0,
        pageSize: 1,
        endDate: today,
      });
      setPastEventCount(pastCount);

      setHasCheckedInitialLoad(true);

      // Auto-switch to past events if no future events but past events exist
      if (futureCount === 0 && pastCount > 0) {
        setIsAutoSwitching(true);
        setShowPastEvents(true);
        loadingPastEvents = true;
      }
    }

    // Build filter params using loadingPastEvents
    const filterParams = {
      sort: loadingPastEvents ? 'startDate,desc' : 'startDate,asc',
      pageNum,
      pageSize,
    };

    if (loadingPastEvents) {
      filterParams.endDate = today;
    } else {
      filterParams.startDate = today;
    }

    // ... fetch and set events
  }

  useEffect(() => {
    if (userId) {
      if (isAutoSwitching) {
        setIsAutoSwitching(false);
        return;
      }
      const isInitialLoad = !hasCheckedInitialLoad && page === 0;
      loadAll(page, isInitialLoad);
    }
  }, [page, showPastEvents, /* other dependencies */]);

  return (
    <>
      {/* Info boxes */}
      {!loading && hasCheckedInitialLoad && futureEventCount === 0 && pastEventCount === 0 && (
        <div className="mb-6 bg-blue-50 border border-blue-200 rounded-lg p-6">
          {/* No events message */}
        </div>
      )}

      {!loading && hasCheckedInitialLoad && showPastEvents && futureEventCount === 0 && pastEventCount > 0 && (
        <div className="mb-4 bg-amber-50 border border-amber-200 rounded-lg p-4">
          {/* Past events message */}
        </div>
      )}

      {!loading && hasCheckedInitialLoad && !showPastEvents && futureEventCount === 0 && (
        <div className="mb-6 bg-blue-50 border border-blue-200 rounded-lg p-6">
          {/* No future events message with Create button */}
        </div>
      )}

      {/* Events list */}
    </>
  );
}
```

## **Complete Example: Public Page Pattern**

```tsx
export default function EventsPage() {
  const [showPastEvents, setShowPastEvents] = useState(false);
  const [futureEventCount, setFutureEventCount] = useState<number | null>(null);
  const [pastEventCount, setPastEventCount] = useState<number | null>(null);
  const [hasCheckedInitialLoad, setHasCheckedInitialLoad] = useState(false);
  const [isAutoSwitching, setIsAutoSwitching] = useState(false);

  useEffect(() => {
    async function fetchEvents() {
      const today = new Date().toISOString().split('T')[0];
      let loadingPastEvents = showPastEvents;

      // On initial load, check both future and past event counts
      if (!hasCheckedInitialLoad && page === 0 && !searchTitle && !searchDateFrom && !searchDateTo) {
        // Check future events count
        const futureRes = await fetch(`/api/proxy/event-details?sort=startDate,asc&page=0&size=1&isActive.equals=true&startDate.greaterThanOrEqual=${today}`);
        const finalFutureCount = futureRes.ok ? parseInt(futureRes.headers.get('x-total-count') || '0', 10) : 0;
        setFutureEventCount(finalFutureCount);

        // Check past events count
        const pastRes = await fetch(`/api/proxy/event-details?sort=startDate,desc&page=0&size=1&isActive.equals=true&endDate.lessThan=${today}`);
        const finalPastCount = pastRes.ok ? parseInt(pastRes.headers.get('x-total-count') || '0', 10) : 0;
        setPastEventCount(finalPastCount);

        setHasCheckedInitialLoad(true);

        // Auto-switch to past events if no future events but past events exist
        if (finalFutureCount === 0 && finalPastCount > 0) {
          setIsAutoSwitching(true);
          setShowPastEvents(true);
          loadingPastEvents = true;
        }
      }

      // Build query params using loadingPastEvents
      const queryParams = new URLSearchParams({
        sort: loadingPastEvents ? 'startDate,desc' : 'startDate,asc',
        page: page.toString(),
        size: BACKEND_FETCH_SIZE.toString(),
        'isActive.equals': 'true',
      });

      if (loadingPastEvents) {
        queryParams.append('endDate.lessThan', today);
      } else {
        queryParams.append('startDate.greaterThanOrEqual', today);
      }

      // ... fetch and process events
    }
    fetchEvents();
  }, [page, showPastEvents, searchTitle, searchDateFrom, searchDateTo]);

  // Skip reload if auto-switching
  useEffect(() => {
    if (isAutoSwitching) {
      const timer = setTimeout(() => setIsAutoSwitching(false), 100);
      return () => clearTimeout(timer);
    }
  }, [isAutoSwitching]);

  return (
    <>
      {/* Info boxes (same as admin, but no Create button) */}
      {/* Events list */}
    </>
  );
}
```

## **Difference Between Admin and Public Pages**

### **Admin Pages** (`/admin/manage-events`)
- **Create Button**: Include "Create New Event" button in "No future events" info box
- **Event Management**: Full CRUD operations available
- **Action Buttons**: Edit, Delete, Activate buttons for each event

### **Public Pages** (`/events`)
- **No Create Button**: Public users cannot create events
- **Read-Only**: View-only access to events
- **Event Actions**: Register, Buy Tickets, Add to Calendar buttons for future events only

## **Conditions Checklist**

### **When to Show Each Message**

| Condition | Future Count | Past Count | Show Past | Message Type | Message |
|----|----|----|----|----|----|
| No events at all | 0 | 0 | N/A | Blue Info Box | "There are no events listed yet..." |
| No future, has past (auto-switched) | 0 | > 0 | true | Amber Message | "Here is the list of recent events..." |
| No future, user switched to future | 0 | >= 0 | false | Blue Info Box | "No future events created..." |
| Has future events | > 0 | >= 0 | false | None | Show events normally |
| Has past events (user switched) | >= 0 | > 0 | true | None | Show events normally |

## **Best Practices**

### **DO:**
- ✅ Always check both future and past counts on initial load
- ✅ Auto-switch to past events if no future events but past events exist
- ✅ Show appropriate info boxes based on event counts
- ✅ Include "Please use future / past events switch above" in all messages
- ✅ Use `isAutoSwitching` flag to prevent double-loading
- ✅ Only check counts on initial load (page 0, no search filters)
- ✅ Use `loadingPastEvents` variable to determine which events to load when auto-switching

### **DON'T:**
- ❌ Don't check counts on every load (only on initial load)
- ❌ Don't auto-switch when search filters are active
- ❌ Don't show "Create New Event" button on public pages
- ❌ Don't forget to reset `isAutoSwitching` flag after auto-switch
- ❌ Don't check counts when page > 0 or search filters are applied
- ❌ Don't show info boxes while loading (`loading === true`)

## **Testing Scenarios**

### **Scenario 1: No Events at All**
- **Setup**: Database has no events (future or past)
- **Expected**: Blue info box with "There are no events listed yet" message
- **Toggle**: Should remain on future events (default)

### **Scenario 2: No Future Events, Has Past Events**
- **Setup**: Database has past events but no future events
- **Expected**: 
  - Auto-switch to past events on page load
  - Amber message box: "Here is the list of recent events..."
  - Past events displayed in table

### **Scenario 3: Has Future Events, No Past Events**
- **Setup**: Database has future events but no past events
- **Expected**: 
  - Show future events normally (default)
  - No info boxes displayed
  - Toggle works to switch to past events (shows empty state)

### **Scenario 4: User Switches to Future Events When None Exist**
- **Setup**: User manually switches toggle to future events when `futureEventCount === 0`
- **Expected**: 
  - Blue info box: "No future events created..."
  - Admin: Includes "Create New Event" button
  - Public: No action button

### **Scenario 5: Search Filters Active**
- **Setup**: User has applied search filters (title, date range)
- **Expected**: 
  - No auto-switch logic runs
  - No info boxes shown (handled by search results)
  - Toggle still works but may show filtered results

## **Reference Implementations**

- **Admin Page**: [`src/app/admin/manage-events/page.tsx`](mdc:src/app/admin/manage-events/page.tsx) - Lines 44-48 (state), Lines 71-107 (auto-switch logic), Lines 605-668 (info boxes)
- **Public Page**: [`src/app/events/page.tsx`](mdc:src/app/events/page.tsx) - Lines 76-81 (state), Lines 115-169 (auto-switch logic), Lines 912-978 (info boxes)

## **Troubleshooting**

### **Auto-Switch Not Working?**
- Check that `hasCheckedInitialLoad` is `false` on initial load
- Verify `page === 0` and no search filters are applied
- Ensure `futureEventCount` and `pastEventCount` are being set correctly
- Check that `isAutoSwitching` flag is preventing double-load

### **Info Boxes Not Showing?**
- Verify `hasCheckedInitialLoad === true` (only shows after initial check)
- Check that `loading === false` (not shown while loading)
- Ensure event counts are set correctly (not `null`)
- Verify conditions match exactly (futureCount === 0, etc.)

### **Double-Loading on Auto-Switch?**
- Check that `isAutoSwitching` flag is set before `setShowPastEvents(true)`
- Verify `useEffect` checks `isAutoSwitching` and returns early if true
- Ensure flag is reset after auto-switch completes

## **Related Patterns**

- See [Admin Toggle Switch Styling](mdc:.cursor/rules/admin_toggle_switch_styling.mdc) for toggle switch UI patterns
- See [Admin Action Buttons](mdc:.cursor/rules/admin_action_buttons_styling.mdc) for "Create New Event" button styling
- See [Form Validation Styling](mdc:.cursor/rules/form_validation_styling.mdc) for error/info box patterns

## **Summary**

**Key Business Rules:**
1. Auto-switch to past events on page load if no future events but past events exist
2. Track both future and past event counts separately
3. Show appropriate info boxes based on event availability
4. Include "Please use future / past events switch above" in all messages
5. Admin pages include "Create New Event" button when no future events
6. Public pages do not include action buttons (read-only)
7. Only check counts on initial load (page 0, no search filters)

This ensures consistent, user-friendly behavior across both admin and public events pages while maintaining appropriate access controls.

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
