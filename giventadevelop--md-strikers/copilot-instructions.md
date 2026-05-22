## hero-section-image-rotation

> This rule defines the standard pattern for hero section image rotation on the home page. The pattern provides automatic rotation through event flyer images with synchronized overlay buttons, interactive navigation controls (Previous/Next/Play/Pause), default image display, and proper event selection criteria.

# Hero Section Image Rotation Pattern

## **Overview**
This rule defines the standard pattern for hero section image rotation on the home page. The pattern provides automatic rotation through event flyer images with synchronized overlay buttons, interactive navigation controls (Previous/Next/Play/Pause), default image display, and proper event selection criteria.

## **Problem Solved**
- **Automatic Image Rotation**: Rotates through event hero images automatically every 8 seconds
- **Interactive Controls**: Provides Previous/Next/Play/Pause buttons for manual navigation (shown on hover/touch)
- **Overlay Synchronization**: Keeps overlay buttons (Buy Tickets Click Here images) synchronized with current event image
- **Event Selection**: Only shows events that meet specific criteria (future dates, isHomePageHeroImage flag)
- **Recurring Event Handling**: Shows only the next occurrence for recurring events
- **Default Image Display**: Shows default image for first 2 seconds before rotation begins
- **User Experience**: Smooth transitions between event images with proper navigation links and manual control
- **Homepage cache**: Hero section images and data are cached in sessionStorage under `homepage_hero_section_cache` (see `src/lib/homepageCacheKeys.ts` and `documentation/cloud_front_amplify_cache/HOMEPAGE_CACHE_IMPLEMENTATION_PLAN.html`). On refresh or repeat visit, a `useLayoutEffect` reads the cache before paint so the hero shows immediately without waiting for `useFilteredEvents('hero')` or standalone media fetch.

## **Hero Image Display Dimensions (Upload Spec)**

The rotating hero uses **object-contain** in a wide container (~65% viewport width, min-height 280–480px). The `<Image>` component uses **width={1200} height={800}** (3:2 landscape). **Recommended upload dimensions** so the hero section does not become excessively tall:

- **Desktop**: **1200×800px (3:2 landscape)**
- **Mobile**: **900×600px (3:2)** or same file scales down
- **Do not use portrait 800×1200 (2:3)** — portrait images make the hero block too tall.

Keep admin/upload copy and docs (e.g. `HERO_SECTION_IMAGE_SPECIFICATIONS.md`, MediaClientPage hero tip) aligned with 1200×800 (3:2).

## **Core Pattern**

### **Component Structure**
```tsx
// ✅ DO: Use DynamicHeroImage component for hero section rotation
const DynamicHeroImage: React.FC = () => {
  const [currentImageIndex, setCurrentImageIndex] = useState(0);
  const [isShowingDefault, setIsShowingDefault] = useState(true);
  const [dynamicImages, setDynamicImages] = useState<string[]>([]);
  const [currentEvent, setCurrentEvent] = useState<EventWithMediaExtended | null>(null);
  const [upcomingEvents, setUpcomingEvents] = useState<EventWithMediaExtended[]>([]);

  // Use shared data hook for consistent date parsing
  const { filteredEvents, isLoading: eventsLoading, error } = useFilteredEvents('hero');

  // Default image path
  const defaultImage = "/images/hero_section/default_hero_section_second_column_poster.jpeg";

  // Rotation logic with useEffect
  useEffect(() => {
    // Start with default image for 2 seconds
    const defaultTimer = setTimeout(() => {
      setIsShowingDefault(false);
    }, 2000);

    // If we have dynamic images, start rotating them
    if (dynamicImages.length > 0) {
      const dynamicTimer = setTimeout(() => {
        const interval = setInterval(() => {
          setCurrentImageIndex((prev) => {
            const newIndex = (prev + 1) % dynamicImages.length;
            // Update current event when image changes
            if (newIndex < dynamicImages.length - 1 && newIndex < upcomingEvents.length) {
              setCurrentEvent(upcomingEvents[newIndex]);
            } else {
              setCurrentEvent(null);
            }
            return newIndex;
          });
        }, 15000); // Change every 15 seconds

        return () => clearInterval(interval);
      }, 2000); // Start after 2 seconds

      return () => {
        clearTimeout(defaultTimer);
        clearTimeout(dynamicTimer);
      };
    }

    return () => clearTimeout(defaultTimer);
  }, [dynamicImages.length, upcomingEvents]);

  // Render logic...
};
```

## **Interactive Slider Controls (Play/Pause/Previous/Next)**

### **Feature Overview**
The hero section slider includes interactive controls that allow users to manually navigate through images and control auto-rotation. These controls appear on hover (desktop) or touch (mobile) and provide:
- **Previous/Next Navigation**: Manual browsing through event images
- **Play/Pause Control**: Toggle auto-rotation on/off
- **Event Synchronization**: Current event state updates when navigating manually

### **Control Visibility**
- **Show on Hover**: Controls appear when mouse hovers over hero image
- **Show on Touch**: Controls appear when user touches the hero image on mobile devices
- **Auto-Hide on Touch**: Controls automatically hide after 3 seconds of no interaction on touch devices
- **Only for Multiple Images**: Controls only appear when there are 2+ images in the rotation

### **Control Buttons**

**1. Previous Button (Left)**
- **Position**: Left side of hero image
- **Action**: Navigate to previous image in rotation
- **Icon**: `ChevronLeft` from lucide-react
- **Styling**: Circular white button with backdrop blur

**2. Play/Pause Button (Center)** ⭐ **KEY FEATURE**
- **Position**: Center of hero image
- **Action**: Toggle auto-rotation on/off
- **Icon**: `Play` when paused, `Pause` when playing
- **Styling**: Circular white button with backdrop blur
- **State Management**: Uses `isPaused` state to control rotation interval
- **Behavior**: 
  - When **paused**: Shows `Play` icon, auto-rotation stops completely
  - When **playing**: Shows `Pause` icon, auto-rotation continues every 8 seconds
  - Clicking toggles between states and immediately affects rotation behavior
- **Implementation**: 
  - `isPaused` state is included in rotation effect dependencies
  - Rotation interval is only created when `isPaused === false`
  - When paused, the interval is cleared and rotation stops immediately
  - When resumed, the interval is recreated and rotation continues from current image

**3. Next Button (Right)**
- **Position**: Right side of hero image
- **Action**: Navigate to next image in rotation
- **Icon**: `ChevronRight` from lucide-react
- **Styling**: Circular white button with backdrop blur

### **Control Implementation**
```tsx
const [isPaused, setIsPaused] = useState(false);
const [isHovered, setIsHovered] = useState(false);
const [isTouched, setIsTouched] = useState(false);
const touchTimeoutRef = React.useRef<NodeJS.Timeout | null>(null);

// Show controls on hover or touch
const showControls = isHovered || isTouched;
const hasMultipleImages = dynamicImages.length > 1;

// Pause rotation when isPaused is true
useEffect(() => {
  if (!isInitialized || dynamicImages.length < 2 || isPaused) {
    return; // Don't rotate if paused
  }
  // ... rotation interval logic
}, [isInitialized, dynamicImages.length, upcomingEvents.length, isPaused]);

// Navigation functions
const goToPrevious = () => {
  setIsTransitioning(true);
  setTimeout(() => {
    setCurrentImageIndex((prevIndex) => {
      const newIndex = (prevIndex - 1 + dynamicImages.length) % dynamicImages.length;
      // Update current event
      if (newIndex < upcomingEvents.length && onEventChangeRef.current) {
        onEventChangeRef.current(upcomingEvents[newIndex]);
      }
      return newIndex;
    });
    setTimeout(() => setIsTransitioning(false), 50);
  }, 400);
};

const goToNext = () => {
  setIsTransitioning(true);
  setTimeout(() => {
    setCurrentImageIndex((prevIndex) => {
      const nextIndex = (prevIndex + 1) % dynamicImages.length;
      // Update current event
      if (nextIndex < upcomingEvents.length && onEventChangeRef.current) {
        onEventChangeRef.current(upcomingEvents[nextIndex]);
      }
      return nextIndex;
    });
    setTimeout(() => setIsTransitioning(false), 50);
  }, 400);
};

const togglePlayPause = () => {
  setIsPaused((prev) => !prev);
  // Rotation effect automatically respects isPaused state
  // When isPaused is true, the rotation interval is not created/cleared
};

// Touch handling
const handleTouchStart = () => {
  setIsTouched(true);
  if (touchTimeoutRef.current) {
    clearTimeout(touchTimeoutRef.current);
  }
  touchTimeoutRef.current = setTimeout(() => {
    setIsTouched(false);
    touchTimeoutRef.current = null;
  }, 3000);
};

// Render controls
{hasMultipleImages && showControls && (
  <div 
    className="absolute inset-0 flex items-center justify-between px-4 z-20 pointer-events-none"
    onTouchStart={handleTouchInteraction}
  >
    <button onClick={goToPrevious} className="...">
      <ChevronLeft className="w-6 h-6" />
    </button>
    <button onClick={togglePlayPause} className="...">
      {isPaused ? <Play /> : <Pause />}
    </button>
    <button onClick={goToNext} className="...">
      <ChevronRight className="w-6 h-6" />
    </button>
  </div>
)}
```

### **Control Styling**

**Button Container:**
- **Position**: `absolute inset-0` (covers entire hero image)
- **Layout**: `flex items-center justify-between` (buttons on left, center, right)
- **Padding**: `px-4` (16px horizontal padding)
- **Z-Index**: `z-20` (above image, below modals)
- **Pointer Events**: `pointer-events-none` on container, `pointer-events-auto` on buttons

**Individual Buttons:**
- **Size**: `w-12 h-12` (48px × 48px circular buttons)
- **Background**: `bg-white/90` (90% opacity white with backdrop blur)
- **Hover**: `hover:bg-white` (full white on hover)
- **Border**: `border-2 border-gray-300 hover:border-blue-500` (2px border, blue on hover)
- **Shadow**: `shadow-lg` (large shadow for depth)
- **Backdrop**: `backdrop-blur-sm` (subtle blur effect)
- **Hover Effect**: `hover:scale-110` (10% scale increase)
- **Active Effect**: `active:scale-95` (5% scale decrease on press)
- **Transition**: `transition-all duration-300` (smooth animations)

**Icon Styling:**
- **Size**: `w-6 h-6` (24px × 24px)
- **Color**: `text-gray-700` (dark gray)
- **Play Icon**: `ml-0.5` (slight left margin for visual centering)

### **Touch Device Behavior**

**Touch Detection:**
- **Trigger**: `onTouchStart` on hero image container
- **Visibility**: Controls show immediately on touch
- **Auto-Hide**: Controls hide after 3 seconds of no interaction
- **Reset**: Any touch interaction resets the 3-second timer

**Touch Interaction:**
- **Button Press**: Each button press resets the 3-second auto-hide timer
- **Swipe Support**: Can be extended with swipe gestures (optional)
- **Active State**: Buttons show `active:scale-95` for tactile feedback

### **Hover Behavior (Desktop)**

**Mouse Enter:**
- **Trigger**: `onMouseEnter` on hero image container
- **Visibility**: Controls show immediately on hover
- **Persistence**: Controls remain visible while mouse is over image

**Mouse Leave:**
- **Trigger**: `onMouseLeave` on hero image container
- **Visibility**: Controls hide immediately when mouse leaves

## **Key Implementation Details**

### **Event Selection Criteria**
- **Primary Criteria (BOTH required)**:
  1. Event must be in the future (`startDate >= today`)
  2. Media must have `isHomePageHeroImage = true`
- **Additional Filtering**:
  - Events within 1 year (`startDate <= oneYearFromNow`)
  - Event must be active (`isActive = true`)
- **Recurring Events**:
  - Show only next occurrence (calculated via `getNextOccurrenceDate()`)
  - Group by `recurrenceSeriesId` or `parentEventId`
  - Keep only earliest next occurrence per series
  - Skip child events (have `parentEventId` but `isRecurring === false`)

### **Image Array Construction**
```typescript
// Build dynamic images array
const imageUrls: string[] = [];

// Add filtered upcoming events (only next occurrence for recurring events)
processedEvents.forEach((event) => {
  if (event.thumbnailUrl) {
    imageUrls.push(event.thumbnailUrl);
  }
});

// Add fallback image at the end
imageUrls.push("https://cdn.builder.io/api/v1/image/assets%2Fa70a28525f6f491aaa751610252a199c%2F67c8b636de774dd2bb5d7097f5fcc176?format=webp&width=800");

setDynamicImages(imageUrls);
```

### **Rotation Timing**
- **Default Image Duration**: 2000ms (2 seconds) - shown before rotation starts
- **Default Rotation Interval**: 8000ms (8 seconds) - default time between image changes (used when `home_page_hero_display_duration_seconds` is NULL)
- **Rotation Start Delay**: 2000ms (2 seconds) - delay before first rotation after default image
- **Pause State**: When `isPaused === true`, rotation timeout is not created (auto-rotation stops)
- **Resume State**: When `isPaused === false`, rotation timeout resumes automatically
- **Per-Image Duration (CRITICAL)**: `event_media.home_page_hero_display_duration_seconds` (int4, nullable). 
  - **When set (1–600 seconds)**: Use the specified duration for that specific hero image
  - **When `NULL`**: Use default 8 seconds (8000ms)
  - **Implementation**: Each image in the rotation array has a corresponding duration in `imageDurations` array
  - **Access Pattern**: Always access durations from `imageDurationsRef.current[imageIndex]` to avoid stale closures
  - **See**: [`documentation/INTEGRATION_PROMPT_HOME_PAGE_HERO_DISPLAY_DURATION.md`](mdc:documentation/INTEGRATION_PROMPT_HOME_PAGE_HERO_DISPLAY_DURATION.md)

### **State Management**
- **`currentImageIndex`**: Tracks which image in the array is currently displayed (0-based)
- **`isShowingDefault`**: Boolean flag for showing default image (first 2 seconds)
- **`dynamicImages`**: Array of image URLs (event images + fallback at end)
- **`currentEvent`**: Current event object (null for default/fallback images)
- **`upcomingEvents`**: Array of processed event objects (sorted by startDate)

## **Rotation Logic**

### **Image Index Calculation**
```typescript
setCurrentImageIndex((prev) => {
  const newIndex = (prev + 1) % dynamicImages.length;
  
  // Calculate number of event images (excluding fallback image at the end)
  const eventImageCount = dynamicImages.length - 1; // Subtract 1 for fallback image
  
  // Update current event when image changes - key implementation for overlay sync
  if (newIndex < eventImageCount && newIndex < upcomingEvents.length) {
    // Show event-specific overlay for event images
    setCurrentEvent(upcomingEvents[newIndex]);
  } else {
    // No overlay for fallback/default images
    setCurrentEvent(null);
  }
  
  return newIndex;
});
```

### **Event Synchronization**
- **Event Images**: When `currentImageIndex < eventImageCount`, set `currentEvent` to `upcomingEvents[newIndex]`
- **Fallback Image**: When showing fallback (last image in array), set `currentEvent` to `null`
- **Overlay Display**: Overlay buttons are shown/hidden based on `currentEvent` state

## **Preventing Duplicate Rotations and Stale Closures**

### **Problem Solved**
- **Duplicate Rotations**: Prevents multiple timeouts from being created simultaneously, causing images to flash or skip
- **Stale Closures**: Ensures rotation function always accesses the latest state values, not outdated closure values
- **Missing Images**: Prevents images from being skipped due to premature effect restarts
- **Flickering**: Eliminates visual flickering caused by duplicate state updates

### **Critical Implementation Pattern**

#### **1. Use Refs for Latest State Values**
```typescript
// ✅ DO: Store latest state values in refs to avoid stale closures
const imageDurationsRef = React.useRef<number[]>([]);
const dynamicImagesRef = React.useRef<string[]>([]);
const upcomingEventsRef = React.useRef<EventWithMediaExtended[]>([]);
const isPausedRef = React.useRef<boolean>(false);
const rotationTimeoutRef = React.useRef<NodeJS.Timeout | null>(null);
const lastScheduledIndexRef = React.useRef<number | null>(null);

// Update refs whenever state changes
useEffect(() => {
  imageDurationsRef.current = imageDurations;
}, [imageDurations]);

useEffect(() => {
  dynamicImagesRef.current = dynamicImages;
}, [dynamicImages]);

useEffect(() => {
  upcomingEventsRef.current = upcomingEvents;
}, [upcomingEvents]);

useEffect(() => {
  isPausedRef.current = isPaused;
}, [isPaused]);
```

#### **2. Store Rotation Function in Ref**
```typescript
// ✅ DO: Store scheduleNextRotation in a ref to avoid dependency issues
const scheduleNextRotationRef = React.useRef<((imageIndex: number) => void) | null>(null);

const scheduleNextRotation = React.useCallback((imageIndex: number) => {
  // Function implementation...
}, [isInitialized]);

// Update ref whenever function changes
useEffect(() => {
  scheduleNextRotationRef.current = scheduleNextRotation;
}, [scheduleNextRotation]);
```

#### **3. Prevent Duplicate Scheduling**
```typescript
// ✅ DO: Prevent duplicate scheduling for the same image index
const scheduleNextRotation = React.useCallback((imageIndex: number) => {
  // CRITICAL: Prevent duplicate scheduling for the same image index
  if (lastScheduledIndexRef.current === imageIndex && rotationTimeoutRef.current !== null) {
    console.log('[HeroSection] Duplicate schedule prevented for index', imageIndex);
    return;
  }

  // CRITICAL: Clear any existing timeout before scheduling a new one
  if (rotationTimeoutRef.current) {
    clearTimeout(rotationTimeoutRef.current);
    rotationTimeoutRef.current = null;
  }

  // Mark this index as scheduled
  lastScheduledIndexRef.current = imageIndex;

  // Don't schedule if paused or not initialized - use refs to get latest values
  if (isPausedRef.current || !isInitialized) {
    lastScheduledIndexRef.current = null;
    return;
  }

  // Access latest arrays from refs (not from closure)
  const currentDurations = imageDurationsRef.current;
  const currentImages = dynamicImagesRef.current;
  const currentEvents = upcomingEventsRef.current;

  // Get duration for the specified image (default to 8 seconds if not available)
  const imageDuration = (currentDurations && currentDurations[imageIndex]) 
    ? currentDurations[imageIndex] 
    : 8000;

  // Schedule timeout with per-image duration
  rotationTimeoutRef.current = setTimeout(() => {
    // Clear the scheduled index ref when timeout executes
    lastScheduledIndexRef.current = null;
    
    setIsTransitioning(true);

    setTimeout(() => {
      setCurrentImageIndex((prevIndex) => {
        // Access latest arrays from refs (not from closure)
        const latestDurations = imageDurationsRef.current;
        const latestImages = dynamicImagesRef.current;
        const latestEvents = upcomingEventsRef.current;
        
        const nextIndex = (prevIndex + 1) % latestImages.length;
        const nextDuration = (latestDurations && latestDurations[nextIndex]) 
          ? latestDurations[nextIndex] 
          : 8000;

        // Update current event based on new index
        if (nextIndex < latestEvents.length && onEventChangeRef.current) {
          onEventChangeRef.current(latestEvents[nextIndex]);
        } else if (onEventChangeRef.current) {
          onEventChangeRef.current(null);
        }

        // Schedule next rotation with the new image's duration
        // Use setTimeout to ensure state update completes before scheduling
        setTimeout(() => {
          if (!isPausedRef.current && scheduleNextRotationRef.current) {
            scheduleNextRotationRef.current(nextIndex);
          }
        }, 10);

        return nextIndex;
      });

      setTimeout(() => setIsTransitioning(false), 50);
    }, 400);
  }, imageDuration);
}, [isInitialized]);
```

#### **4. Rotation Effect Without scheduleNextRotation in Dependencies**
```typescript
// ✅ DO: Remove scheduleNextRotation from effect dependencies
useEffect(() => {
  // Don't start rotation until initialized and we have at least 2 images
  if (!isInitialized || dynamicImages.length < 2 || isPaused) {
    // Clear any existing timeout when paused
    if (rotationTimeoutRef.current) {
      clearTimeout(rotationTimeoutRef.current);
      rotationTimeoutRef.current = null;
    }
    lastScheduledIndexRef.current = null;
    return;
  }

  // CRITICAL: Clear any existing timeout before starting new rotation
  if (rotationTimeoutRef.current) {
    clearTimeout(rotationTimeoutRef.current);
    rotationTimeoutRef.current = null;
  }

  // Reset the scheduled index guard when starting fresh rotation
  lastScheduledIndexRef.current = null;

  // Start the rotation cycle with the current image index
  // Use the ref to call the function to avoid dependency issues
  setTimeout(() => {
    if (scheduleNextRotationRef.current && !isPausedRef.current) {
      scheduleNextRotationRef.current(currentImageIndex);
    }
  }, 0);

  return () => {
    if (rotationTimeoutRef.current) {
      clearTimeout(rotationTimeoutRef.current);
      rotationTimeoutRef.current = null;
    }
    lastScheduledIndexRef.current = null;
  };
}, [isInitialized, dynamicImages.length, upcomingEvents.length, isPaused]);
// ❌ DON'T: Include scheduleNextRotation in dependencies - causes duplicate rotations
```

#### **5. Manual Navigation Functions**
```typescript
// ✅ DO: Clear timeout and reset guard when manually navigating
const goToPrevious = () => {
  // Clear existing rotation timeout when manually navigating
  if (rotationTimeoutRef.current) {
    clearTimeout(rotationTimeoutRef.current);
    rotationTimeoutRef.current = null;
  }
  lastScheduledIndexRef.current = null;

  setIsTransitioning(true);
  setTimeout(() => {
    setCurrentImageIndex((prevIndex) => {
      const latestImages = dynamicImagesRef.current;
      const latestEvents = upcomingEventsRef.current;
      const newIndex = (prevIndex - 1 + latestImages.length) % latestImages.length;

      if (newIndex < latestEvents.length && onEventChangeRef.current) {
        onEventChangeRef.current(latestEvents[newIndex]);
      } else if (onEventChangeRef.current) {
        onEventChangeRef.current(null);
      }

      // Restart rotation from new index after navigation completes
      setTimeout(() => {
        if (scheduleNextRotationRef.current) {
          scheduleNextRotationRef.current(newIndex);
        }
      }, 100);

      return newIndex;
    });
    setTimeout(() => setIsTransitioning(false), 50);
  }, 400);
};

const goToNext = () => {
  // Clear existing rotation timeout when manually navigating
  if (rotationTimeoutRef.current) {
    clearTimeout(rotationTimeoutRef.current);
    rotationTimeoutRef.current = null;
  }
  lastScheduledIndexRef.current = null;

  setIsTransitioning(true);
  setTimeout(() => {
    setCurrentImageIndex((prevIndex) => {
      const latestImages = dynamicImagesRef.current;
      const latestEvents = upcomingEventsRef.current;
      const nextIndex = (prevIndex + 1) % latestImages.length;

      if (nextIndex < latestEvents.length && onEventChangeRef.current) {
        onEventChangeRef.current(latestEvents[nextIndex]);
      } else if (onEventChangeRef.current) {
        onEventChangeRef.current(null);
      }

      // Restart rotation from new index after navigation completes
      setTimeout(() => {
        if (scheduleNextRotationRef.current) {
          scheduleNextRotationRef.current(nextIndex);
        }
      }, 100);

      return nextIndex;
    });
    setTimeout(() => setIsTransitioning(false), 50);
  }, 400);
};
```

### **Key Requirements**

#### **Required Refs**
- **`imageDurationsRef`**: Stores latest `imageDurations` array to avoid stale closures
- **`dynamicImagesRef`**: Stores latest `dynamicImages` array to avoid stale closures
- **`upcomingEventsRef`**: Stores latest `upcomingEvents` array to avoid stale closures
- **`isPausedRef`**: Stores latest `isPaused` state to avoid stale closures
- **`rotationTimeoutRef`**: Stores current timeout ID for cleanup
- **`lastScheduledIndexRef`**: Tracks last scheduled image index to prevent duplicates
- **`scheduleNextRotationRef`**: Stores rotation function to avoid dependency issues

#### **Required useEffect Hooks for Ref Synchronization**
```typescript
// ✅ DO: Keep refs synchronized with state
useEffect(() => {
  imageDurationsRef.current = imageDurations;
}, [imageDurations]);

useEffect(() => {
  dynamicImagesRef.current = dynamicImages;
}, [dynamicImages]);

useEffect(() => {
  upcomingEventsRef.current = upcomingEvents;
}, [upcomingEvents]);

useEffect(() => {
  isPausedRef.current = isPaused;
}, [isPaused]);

useEffect(() => {
  scheduleNextRotationRef.current = scheduleNextRotation;
}, [scheduleNextRotation]);
```

#### **Critical Rules**
1. **Always access arrays/states from refs inside `scheduleNextRotation`**: Use `imageDurationsRef.current`, `dynamicImagesRef.current`, `upcomingEventsRef.current`, `isPausedRef.current` instead of direct state values
2. **Always clear timeout before scheduling new one**: Check `rotationTimeoutRef.current` and clear it before creating a new timeout
3. **Prevent duplicate scheduling**: Check `lastScheduledIndexRef.current === imageIndex` before scheduling
4. **Don't include `scheduleNextRotation` in effect dependencies**: Store it in a ref and call via `scheduleNextRotationRef.current`
5. **Reset guards on cleanup**: Set `lastScheduledIndexRef.current = null` in cleanup functions
6. **Defer next rotation scheduling**: Use `setTimeout(..., 10)` after state update to ensure it completes

### **Common Anti-Patterns**

```typescript
// ❌ DON'T: Access state directly in recursive function (causes stale closures)
const scheduleNextRotation = React.useCallback((imageIndex: number) => {
  const duration = imageDurations[imageIndex]; // WRONG: Stale closure
  // ...
}, [imageDurations]); // This causes function to be recreated on every change

// ❌ DON'T: Include scheduleNextRotation in effect dependencies
useEffect(() => {
  scheduleNextRotation(currentImageIndex);
}, [scheduleNextRotation, currentImageIndex]); // WRONG: Causes duplicate rotations

// ❌ DON'T: Schedule without clearing existing timeout
rotationTimeoutRef.current = setTimeout(() => {
  // WRONG: May create duplicate timeouts
}, duration);

// ❌ DON'T: Schedule immediately from state updater
setCurrentImageIndex((prev) => {
  scheduleNextRotation(nextIndex); // WRONG: May cause duplicate scheduling
  return nextIndex;
});
```

### **Best Practices**

#### **DO:**
- ✅ Always use refs (`imageDurationsRef.current`, `dynamicImagesRef.current`, etc.) inside `scheduleNextRotation`
- ✅ Always clear existing timeout before scheduling new one
- ✅ Check `lastScheduledIndexRef.current` to prevent duplicate scheduling
- ✅ Store `scheduleNextRotation` in a ref and call via `scheduleNextRotationRef.current`
- ✅ Use `setTimeout` with small delay (10ms) after state update before scheduling next rotation
- ✅ Reset `lastScheduledIndexRef.current = null` in cleanup functions
- ✅ Keep refs synchronized with state using dedicated `useEffect` hooks
- ✅ Clear timeout and reset guard in manual navigation functions

#### **DON'T:**
- ❌ Access state directly in `scheduleNextRotation` (use refs instead)
- ❌ Include `scheduleNextRotation` in effect dependency array
- ❌ Schedule new timeout without clearing existing one
- ❌ Skip duplicate prevention guard (`lastScheduledIndexRef`)
- ❌ Call `scheduleNextRotation` directly from state updater (use `setTimeout` with delay)
- ❌ Forget to reset guards in cleanup functions
- ❌ Skip ref synchronization `useEffect` hooks

### **Troubleshooting**

#### **Duplicate Rotations (Images Flashing/Skipping)**
- **Check**: Is `scheduleNextRotation` in the effect dependency array? (Should NOT be)
- **Check**: Is `lastScheduledIndexRef` being checked before scheduling? (Should be)
- **Check**: Is timeout being cleared before scheduling new one? (Should be)
- **Check**: Is `scheduleNextRotationRef.current` being used instead of direct function call? (Should be)

#### **Missing Images in Rotation**
- **Check**: Are refs being updated when state changes? (Should have `useEffect` hooks)
- **Check**: Is `scheduleNextRotation` accessing arrays from refs? (Should use `.current`)
- **Check**: Is guard being reset on cleanup? (Should set `lastScheduledIndexRef.current = null`)

#### **Stale Duration Values (All Images Using 8 Seconds)**
- **Check**: Is `imageDurationsRef.current` being used instead of `imageDurations`? (Should use ref)
- **Check**: Is ref being updated when `imageDurations` changes? (Should have `useEffect`)

#### **Rotation Not Continuing After Manual Navigation**
- **Check**: Is `scheduleNextRotationRef.current` being called after navigation? (Should be)
- **Check**: Is guard being reset before calling? (Should set `lastScheduledIndexRef.current = null`)

## **Overlay Logic (Buy Tickets Click Here Image Pattern)**

### **Event Type Detection (Matching Events Page Logic)**

The hero section overlay uses the same event type detection logic as the events page (`/events`), ensuring consistency across the application.

### **Overlay Display Rules**

**1. Ticketed Fundraiser/Charity Events** (Highest Priority)
   - **Condition**: `isTicketedFundraiserEvent(event) === true`
   - **Image**: `/images/buy_tickets_click_here_fundraiser.png`
   - **Action**: `/events/${event.id}/donation-checkout`
   - **Detection**: Event must be `admissionType === 'TICKETED'` AND `isDonationBasedEvent(event) === true`

**2. Regular Ticketed Events**
   - **Condition**: `event.admissionType?.toUpperCase() === 'TICKETED'` AND NOT a ticketed fundraiser
   - **Image**: `/images/buy_tickets_click_here_red.webp`
   - **Action**: 
     - `/events/${event.id}/manual-checkout` (if `manualPaymentEnabled === true` AND `paymentFlowMode === 'MANUAL_ONLY'` or `'HYBRID'`)
     - `/events/${event.id}/checkout` (default Stripe checkout)

**3. No Overlay**
   - **Condition**: Event is not ticketed OR event is in the past
   - **Action**: No overlay displayed

### **Upcoming Events Only**

**Critical**: Overlay is only shown for **upcoming events** (today or future):
```typescript
// Check if event is upcoming (today or future)
const today = new Date();
const todayStr = `${today.getFullYear()}-${String(today.getMonth() + 1).padStart(2, '0')}-${String(today.getDate()).padStart(2, '0')}`;
const eventDateStr = event.startDate ? event.startDate.split('T')[0] : null;

if (!eventDateStr) return null; // No date = no overlay

const isToday = eventDateStr === todayStr;
const isFuture = eventDateStr > todayStr;
const isUpcomingLocal = isToday || isFuture;

if (!isUpcomingLocal) return null; // Don't show overlay for past events
```

### **Overlay Implementation**
```tsx
import { isTicketedFundraiserEvent } from '@/lib/donation/utils';

const getOverlayInfo = (event: EventWithMediaExtended | null) => {
  if (!event || !event.id) return null;

  // Check if event is upcoming (today or future)
  const today = new Date();
  const todayStr = `${today.getFullYear()}-${String(today.getMonth() + 1).padStart(2, '0')}-${String(today.getDate()).padStart(2, '0')}`;
  const eventDateStr = event.startDate ? event.startDate.split('T')[0] : null;
  
  if (!eventDateStr) return null;
  
  const isToday = eventDateStr === todayStr;
  const isFuture = eventDateStr > todayStr;
  const isUpcomingLocal = isToday || isFuture;

  if (!isUpcomingLocal) return null; // Don't show overlay for past events

  // Priority 1: Ticketed fundraiser/charity (shows special fundraiser image)
  const isTicketedFundraiser = isTicketedFundraiserEvent(event);
  
  if (isTicketedFundraiser) {
    return {
      image: '/images/buy_tickets_click_here_fundraiser.png',
      href: `/events/${event.id}/donation-checkout`,
      alt: 'Buy Tickets'
    };
  }

  // Priority 2: Regular ticketed event
  if (event.admissionType?.toUpperCase() === 'TICKETED') {
    // Route to manual checkout if manual payment is enabled, otherwise Stripe checkout
    const checkoutRoute =
      event.manualPaymentEnabled === true &&
      (event.paymentFlowMode === 'MANUAL_ONLY' || event.paymentFlowMode === 'HYBRID')
        ? `/events/${event.id}/manual-checkout`
        : `/events/${event.id}/checkout`;

    return {
      image: '/images/buy_tickets_click_here_red.webp',
      href: checkoutRoute,
      alt: 'Buy Tickets'
    };
  }

  return null; // No overlay for non-ticketed events
};

// Render overlay in hero section
const overlayInfo = getOverlayInfo(currentEvent);

{overlayInfo && (
  <div className="absolute bottom-4 right-4 z-10">
    <Link
      href={overlayInfo.href}
      className="block cursor-pointer hover:scale-105 transition-transform duration-300"
      onClick={(e) => e.stopPropagation()}
      title={overlayInfo.alt}
      aria-label={overlayInfo.alt}
    >
      <img
        src={overlayInfo.image}
        alt={overlayInfo.alt}
        className="object-contain w-[150px] h-[52px] sm:w-[200px] sm:h-[70px] cursor-pointer hover:scale-105 transition-transform duration-300"
      />
    </Link>
  </div>
)}
```

### **Image Specifications**

**Regular Ticketed Events:**
- **Image Path**: `/images/buy_tickets_click_here_red.webp`
- **Responsive Sizing**: 
  - Mobile: `w-[150px] h-[52px]` (150px × 52px)
  - Desktop: `sm:w-[200px] sm:h-[70px]` (200px × 70px)
- **Styling**: `object-contain` (maintains aspect ratio, no cropping)

**Fundraiser/Charity Events:**
- **Image Path**: `/images/buy_tickets_click_here_fundraiser.png`
- **Responsive Sizing**: 
  - Mobile: `w-[150px] h-[52px]` (150px × 52px)
  - Desktop: `sm:w-[200px] sm:h-[70px]` (200px × 70px)
- **Styling**: `object-contain` (maintains aspect ratio, no cropping)

### **Positioning and Styling**

**Overlay Container:**
- **Position**: `absolute bottom-4 right-4 z-10` (bottom-right corner, 16px from edges)
- **Z-Index**: `z-10` (above hero image, below modals)

**Link Styling:**
- **Display**: `block` (full clickable area)
- **Hover Effect**: `hover:scale-105 transition-transform duration-300` (5% scale increase)
- **Click Handling**: `onClick={(e) => e.stopPropagation()}` (prevents event bubbling)

**Image Styling:**
- **Object Fit**: `object-contain` (maintains aspect ratio, no cropping)
- **Hover Effect**: `hover:scale-105 transition-transform duration-300` (matches link hover)
- **Cursor**: `cursor-pointer` (shows pointer on hover)

### **Routing Logic**

**Manual Payment Checkout Route:**
- **Condition**: `event.manualPaymentEnabled === true` AND (`event.paymentFlowMode === 'MANUAL_ONLY'` OR `event.paymentFlowMode === 'HYBRID'`)
- **Route**: `/events/${event.id}/manual-checkout`
- **Use Case**: Events configured for fee-free manual payments (Zelle, Venmo, etc.)

**Stripe Checkout Route (Default):**
- **Condition**: All other ticketed events
- **Route**: `/events/${event.id}/checkout`
- **Use Case**: Standard Stripe payment flow

**Fundraiser Donation Checkout Route:**
- **Condition**: `isTicketedFundraiserEvent(event) === true`
- **Route**: `/events/${event.id}/donation-checkout`
- **Use Case**: Ticketed fundraiser/charity events with Givebutter integration

## **Image Display Requirements**

### **Default Image (First 2 Seconds)**
```tsx
if (isShowingDefault) {
  return (
    <div className="relative w-full h-full">
      <Link href="/events" className="block w-full h-full">
        <Image
          src={defaultImage}
          alt="Default Hero Image"
          fill
          className="object-fill w-full h-full cursor-pointer"
          style={{
            filter: 'contrast(1.1) saturate(0.9)'
          }}
          sizes="(max-width: 1024px) 100vw, 50vw"
        />
      </Link>
      {/* No overlay for default image */}
    </div>
  );
}
```

### **Dynamic Images (After 2 Seconds)**
```tsx
if (dynamicImages.length > 0) {
  const isShowingEventFlyer = currentImageIndex < dynamicImages.length - 1; // Skip the fallback image

  return (
    <div className="relative w-full h-full">
      <Link
        href={isShowingEventFlyer && currentEvent && currentEvent.id ? `/events/${currentEvent.id}` : '/events'}
        className="block w-full h-full"
      >
        <Image
          src={dynamicImages[currentImageIndex]}
          alt="Dynamic Hero Image"
          fill
          className="object-fill w-full h-full cursor-pointer"
          style={{
            filter: 'contrast(1.1) saturate(0.9)'
          }}
          sizes="(max-width: 1024px) 100vw, 50vw"
        />
      </Link>
      {/* Overlay logic based on currentEvent */}
    </div>
  );
}
```

## **Key CSS Properties**

### **Image Container**
- **`relative w-full h-full`**: Full-width, full-height container with relative positioning
- **`object-fill`**: Image fills container (may crop, but maintains aspect ratio)
- **`cursor-pointer`**: Shows pointer cursor on hover
- **Filter**: `contrast(1.1) saturate(0.9)` - subtle image enhancement

### **Overlay Container**
- **`absolute bottom-4 right-4 z-10`**: Positioned at bottom-right corner
- **`hover:scale-105`**: 5% scale increase on hover
- **`transition-transform duration-300`**: Smooth hover animation

### **Slider Controls Container**
- **`absolute inset-0`**: Covers entire hero image area
- **`flex items-center justify-between`**: Horizontal layout with buttons on left, center, right
- **`px-4`**: Horizontal padding (16px) for button spacing
- **`z-20`**: Above image and Buy Tickets overlay (controls are interactive)
- **`pointer-events-none`**: Container doesn't block clicks (buttons have `pointer-events-auto`)

### **Control Button Styling**
- **Size**: `w-12 h-12` (48px × 48px circular buttons)
- **Background**: `bg-white/90` (90% opacity white)
- **Backdrop**: `backdrop-blur-sm` (subtle blur effect)
- **Border**: `border-2 border-gray-300 hover:border-blue-500` (2px border, blue on hover)
- **Shadow**: `shadow-lg` (large shadow for depth)
- **Hover**: `hover:bg-white hover:scale-110` (full white, 10% scale increase)
- **Active**: `active:bg-white active:scale-95 active:border-blue-600` (tactile feedback on press)
- **Transition**: `transition-all duration-300` (smooth animations)

### **See All Events Button** (Bottom Left)
- **Position**: `absolute left-4` with `bottom: '-36px'` (outside image container)
- **Styling**: `h-14 rounded-xl bg-indigo-100 hover:bg-indigo-200`
- **Icon Container**: `w-10 h-10 rounded-lg bg-indigo-200`
- **Text**: `font-semibold text-indigo-700`

## **Data Hook Requirements**

### **useFilteredEvents Hook**
- **Filter Type**: `'hero'` - filters events for hero section display
- **Returns**: `{ filteredEvents, isLoading, error }`
- **Filter Criteria**: 
  - Events with `isHomePageHeroImage = true`
  - Future events (`startDate >= today`)
  - Active events (`isActive = true`)
  - Within time window (typically 1 year)

### **Event Processing**
```typescript
// Process events and filter recurring events to show only next occurrence
const processedEvents: EventWithMediaExtended[] = [];
const recurringSeriesMap = new Map<number, EventWithMediaExtended>();

filteredEvents.forEach(({ event, media }) => {
  const eventWithMedia: EventWithMediaExtended = {
    ...event,
    thumbnailUrl: media.fileUrl,
    media: [media]
  };

  // Handle recurring events
  if (isRecurringEvent(event)) {
    const seriesId = event.recurrenceSeriesId || event.parentEventId || event.id;
    const nextOccurrence = getNextOccurrenceDate(event, today);

    if (!nextOccurrence || nextOccurrence > oneYearFromNow) {
      return; // Skip if no next occurrence or beyond 1 year
    }

    // Update event startDate to next occurrence for display
    eventWithMedia.startDate = nextOccurrence.toISOString().split('T')[0];

    // Keep only earliest next occurrence per series
    const existingSeriesEvent = recurringSeriesMap.get(seriesId);
    if (!existingSeriesEvent || nextOccurrence < new Date(existingSeriesEvent.startDate!)) {
      recurringSeriesMap.set(seriesId, eventWithMedia);
    }
  } else {
    // Non-recurring event - add directly
    processedEvents.push(eventWithMedia);
  }
});

// Add recurring events (only one per series)
recurringSeriesMap.forEach((event) => {
  processedEvents.push(event);
});

// Sort by startDate to show earliest events first
processedEvents.sort((a, b) => {
  if (!a.startDate || !b.startDate) return 0;
  return new Date(a.startDate).getTime() - new Date(b.startDate).getTime();
});
```

## **Complete Example**

### **Full DynamicHeroImage Component Pattern**
```tsx
'use client';

import React, { useState, useEffect } from 'react';
import Image from 'next/image';
import Link from 'next/link';
import { useFilteredEvents } from '@/hooks/useFilteredEvents';
import { isRecurringEvent, getNextOccurrenceDate } from '@/lib/eventUtils';

const DynamicHeroImage: React.FC = () => {
  const [currentImageIndex, setCurrentImageIndex] = useState(0);
  const [isShowingDefault, setIsShowingDefault] = useState(true);
  const [dynamicImages, setDynamicImages] = useState<string[]>([]);
  const [currentEvent, setCurrentEvent] = useState<EventWithMediaExtended | null>(null);
  const [upcomingEvents, setUpcomingEvents] = useState<EventWithMediaExtended[]>([]);

  const { filteredEvents, isLoading: eventsLoading, error } = useFilteredEvents('hero');
  const defaultImage = "/images/hero_section/default_hero_section_second_column_poster.jpeg";

  // Initialize images
  useEffect(() => {
    const initializeHeroImages = async () => {
      if (filteredEvents && filteredEvents.length > 0) {
        // Process events (recurring handling, sorting, etc.)
        // Build imageUrls array
        // Set dynamicImages and upcomingEvents
      }
    };

    if (!eventsLoading && !error) {
      initializeHeroImages();
    }
  }, [filteredEvents, eventsLoading, error]);

  // Rotation logic
  useEffect(() => {
    const defaultTimer = setTimeout(() => {
      setIsShowingDefault(false);
    }, 2000);

    if (dynamicImages.length > 0) {
      const dynamicTimer = setTimeout(() => {
        const interval = setInterval(() => {
          setCurrentImageIndex((prev) => {
            const newIndex = (prev + 1) % dynamicImages.length;
            const eventImageCount = dynamicImages.length - 1;
            
            if (newIndex < eventImageCount && newIndex < upcomingEvents.length) {
              setCurrentEvent(upcomingEvents[newIndex]);
            } else {
              setCurrentEvent(null);
            }
            
            return newIndex;
          });
        }, 15000);

        return () => clearInterval(interval);
      }, 2000);

      return () => {
        clearTimeout(defaultTimer);
        clearTimeout(dynamicTimer);
      };
    }

    return () => clearTimeout(defaultTimer);
  }, [dynamicImages.length, upcomingEvents]);

  // Render default image
  if (isShowingDefault) {
    return (
      <div className="relative w-full h-full">
        <Link href="/events" className="block w-full h-full">
          <Image
            src={defaultImage}
            alt="Default Hero Image"
            fill
            className="object-fill w-full h-full cursor-pointer"
            style={{ filter: 'contrast(1.1) saturate(0.9)' }}
            sizes="(max-width: 1024px) 100vw, 50vw"
          />
        </Link>
      </div>
    );
  }

  // Render dynamic images
  if (dynamicImages.length > 0) {
    const isShowingEventFlyer = currentImageIndex < dynamicImages.length - 1;
    const overlay = getOverlayForEvent(currentEvent);

    return (
      <div className="relative w-full h-full">
        <Link
          href={isShowingEventFlyer && currentEvent?.id ? `/events/${currentEvent.id}` : '/events'}
          className="block w-full h-full"
        >
          <Image
            src={dynamicImages[currentImageIndex]}
            alt="Dynamic Hero Image"
            fill
            className="object-fill w-full h-full cursor-pointer"
            style={{ filter: 'contrast(1.1) saturate(0.9)' }}
            sizes="(max-width: 1024px) 100vw, 50vw"
          />
        </Link>

        {/* Buy Tickets Overlay Image - Bottom Right Corner (matching events page style) */}
        {overlayInfo && (
          <div className="absolute bottom-4 right-4 z-10">
            <Link
              href={overlayInfo.href}
              className="block cursor-pointer hover:scale-105 transition-transform duration-300"
              onClick={(e) => e.stopPropagation()}
              title={overlayInfo.alt}
              aria-label={overlayInfo.alt}
            >
              <img
                src={overlayInfo.image}
                alt={overlayInfo.alt}
                className="object-contain w-[150px] h-[52px] sm:w-[200px] sm:h-[70px] cursor-pointer hover:scale-105 transition-transform duration-300"
              />
            </Link>
          </div>
        )}
      </div>
    );
  }

  // Fallback
  return (
    <div className="relative w-full h-full">
      <Image
        src={defaultImage}
        alt="Default Hero Image"
        fill
        className="object-fill w-full h-full cursor-pointer"
        style={{ filter: 'contrast(1.1) saturate(0.9)' }}
        sizes="(max-width: 1024px) 100vw, 50vw"
      />
    </div>
  );
};
```

## **Timing Constants**

### **Critical Timing Values**
- **Default Image Duration**: `2000` (2 seconds) - `setTimeout(() => setIsShowingDefault(false), 2000)`
- **Rotation Interval**: `8000` (8 seconds) - `setInterval(() => { ... }, 8000)` (updated from 15 seconds)
- **Rotation Start Delay**: `2000` (2 seconds) - `setTimeout(() => { setInterval(...) }, 2000)`
- **Pause Control**: `isPaused` state controls whether rotation interval is active

### **Timing Flow**
1. **0-2 seconds**: Show default image (`isShowingDefault = true`)
2. **2 seconds**: Hide default image, start rotation timer (if `isPaused === false`)
3. **2-10 seconds**: Show first event image (if not paused)
4. **10 seconds**: Rotate to second event image (if not paused)
5. **18 seconds**: Rotate to third event image (if not paused)
6. **...continues every 8 seconds...** (if not paused)
7. **Last image**: Fallback image (no overlay)
8. **Pause State**: When user clicks pause, rotation stops immediately and interval is cleared
9. **Resume State**: When user clicks play, rotation resumes and interval is recreated

## **Hero Image Source by 4-Month Window**

Hero image sources depend on whether there are **events in the next 4 months** that qualify for hero display (same event/hero criteria, with `startDate` within the next 4 months).

### **Definition: Events in Next 4 Months**
- **Events in next 4 months**: Among the hero-qualifying processed events (after recurring handling and sorting), at least one has `startDate` between **today** (inclusive) and **today + 4 months** (inclusive).
- **Four-month boundary**: `fourMonthsFromNow = new Date(); fourMonthsFromNow.setMonth(fourMonthsFromNow.getMonth() + 4);`

### **When There Are NO Events in the Next 4 Months**
- **Source**: Use **standalone** hero images from the **event_media** table (**event_id** IS NULL).
- **Filter**: **Either** `is_home_page_hero_image = true` **or** `is_hero_image = true`; `startDisplayingFromDate` valid (null or <= today).
- **Limit**: Select **at most 6** images for the hero loop.
- **Display**: Same rotation, per-image duration (`home_page_hero_display_duration_seconds`), and default image at end. **No event overlay** (no event object for these slides; `currentEvent` remains null for all these images).
- **API**: Fetch by hero flags then filter client-side (do **not** rely on `eventId.specified=false` — many backends don't support it or return empty). (1) `isHeroImage.equals=true&size=50`, (2) if needed `isHomePageHeroImage.equals=true&size=50`; merge/dedupe by id, filter by `event_id` null + `isStandaloneHeroEligible` + `startDisplayingFromDate`, sort by `displayOrder`, take first 6.

### **When There ARE Events in the Next 4 Months**
- **Primary source**: Keep current behavior — hero loop is built from hero-qualifying events (event-based images with overlay where applicable).
- **Additional source**: Add **at most 2** hero images from **event_media** that are **not associated with any event** (`event_id` IS NULL).
- **Filter**: **Either** `is_home_page_hero_image = true` **or** `is_hero_image = true`; `startDisplayingFromDate` valid (null or <= today).
- **Limit**: **Maximum 2** standalone images, appended to the loop after event-based images (before the default image).
- **Display**: Same animation and rotation; for these 2 slides **no event overlay** (do not add them to `upcomingEvents`; `currentEvent` is null when showing these indices).
- **API**: Fetch by hero flags then filter client-side. (1) `isHeroImage.equals=true&size=30`, (2) if needed `isHomePageHeroImage.equals=true&size=30`; merge/dedupe by id, filter by `event_id` null + `isStandaloneHeroEligible` + `startDisplayingFromDate`, sort by `displayOrder`, take first 2.

### **Preserved Behavior**
- **Animation and display**: No change to transition, refs, per-image duration, or default image (first 2s, then rotation).
- **Overlay**: Event overlay only for slides that have an associated event; standalone or “no events in 4 months” slides have no overlay.
- **Play/Pause/Prev/Next**: Unchanged; controls and rotation logic use the same `dynamicImages` / `imageDurations` / `upcomingEvents` pattern.

### **event_media Fields Used**
- `event_id`: NULL = standalone hero image; NOT NULL = assigned to an event.
- `is_home_page_hero_image` / `isHomePageHeroImage`: For event-based hero, must be true. For **standalone** only, either this **or** `is_hero_image` can be true.
- `is_hero_image` / `isHeroImage`: For **standalone** selection only, accepted as alternative to `is_home_page_hero_image` (either flag true).
- `start_displaying_from_date` / `startDisplayingFromDate`: If set, only show when <= today.
- `file_url` / `fileUrl`: Image URL.
- `home_page_hero_display_duration_seconds` / `homePageHeroDisplayDurationSeconds`: Per-slide duration (default 8s if null).

### **Standalone Hero Image Data Requirement (event_id NULL only)**
For **standalone** hero images (no event) to be selected and shown on the home page hero section:

1. **`event_id` IS NULL** — Media must not be tied to any event (standalone).
2. **Either** **`is_home_page_hero_image = true`** **or** **`is_hero_image = true`** — For standalone selection only, the frontend accepts either flag. Event-based hero image retrieval still requires `is_home_page_hero_image = true` only.

- **API for standalone (implementation)**: Do **not** rely on `eventId.specified=false` — many backends don't support it or return no rows. Instead: (1) Fetch `isHeroImage.equals=true&size=30` (or 50 for no-events-in-4-months), (2) optionally fetch `isHomePageHeroImage.equals=true&size=30` (or 50) and merge; (3) filter client-side with `isStandaloneHeroMedia(m)` (event_id/eventId null), `isStandaloneHeroEligible(m)` (is_home_page_hero_image OR is_hero_image), and `isHeroMediaDisplayDateValid(m)`; (4) dedupe by id, sort by displayOrder, take first 2 (or 6 for no-events case).
- **Admin / inserts**: When creating standalone hero media, set either **`is_home_page_hero_image = true`** or **`is_hero_image = true`** (or both).

## **Event Selection Rules**

### **Primary Selection Criteria (BOTH Required)**
1. **Future Event**: `startDate >= today` (event must be today or in the future)
2. **Hero Image Flag**: Media must have `isHomePageHeroImage = true`

### **Additional Filters**
- **Time Window**: Events within 1 year (`startDate <= oneYearFromNow`)
- **Active Status**: `isActive = true`
- **Recurring Events**: Show only next occurrence (calculated via `getNextOccurrenceDate()`)
- **Recurring Series**: One event per series (earliest next occurrence)

### **Recurring Event Handling**
- **Series Grouping**: Group by `recurrenceSeriesId` or `parentEventId`
- **Next Occurrence**: Calculate using `getNextOccurrenceDate(event, today)`
- **Date Update**: Update `event.startDate` to next occurrence date for display
- **Child Events**: Skip child events (have `parentEventId` but `isRecurring === false`)

## **Overlay Display Rules**

### **Overlay Priority System**
1. **Ticketed Fundraiser/Charity** (Highest Priority)
   - Condition: `isTicketedFundraiserEvent(event) === true`
   - Image: `/images/buy_tickets_click_here_fundraiser.png`
   - Route: `/events/${event.id}/donation-checkout`

2. **Regular Ticketed Events**
   - Condition: `event.admissionType?.toUpperCase() === 'TICKETED'` AND NOT a ticketed fundraiser
   - Image: `/images/buy_tickets_click_here_red.webp`
   - Route: `/events/${event.id}/checkout` or `/events/${event.id}/manual-checkout` (if manual payment enabled)

3. **No Overlay**
   - Condition: Event is not ticketed OR event is in the past OR no event is associated with current image

### **Overlay Display Conditions**
- **Show Overlay**: 
  - Event is upcoming (today or future)
  - Event is ticketed (regular or fundraiser)
  - Current image is an event flyer (not default/fallback image)
- **Hide Overlay**: 
  - When showing default image or fallback image (`currentEvent === null`)
  - When event is in the past
  - When event is not ticketed

## **Best Practices**

### **DO:**
- ✅ Use `useFilteredEvents('hero')` hook for consistent event filtering
- ✅ Always show default image for first 2 seconds
- ✅ Use per-image durations from `home_page_hero_display_duration_seconds` field (default 8 seconds if NULL)
- ✅ **Always use refs** (`imageDurationsRef.current`, `dynamicImagesRef.current`, etc.) inside `scheduleNextRotation` to avoid stale closures
- ✅ **Always clear timeout** before scheduling new one to prevent duplicates
- ✅ **Prevent duplicate scheduling** using `lastScheduledIndexRef` guard
- ✅ **Store rotation function in ref** (`scheduleNextRotationRef`) and call via ref to avoid dependency issues
- ✅ **Defer next rotation scheduling** with `setTimeout(..., 10)` after state update
- ✅ Update `currentEvent` state when image index changes
- ✅ Sort events by `startDate` (earliest first)
- ✅ Handle recurring events (show only next occurrence)
- ✅ Add fallback image at end of `dynamicImages` array
- ✅ Use `getOverlayInfo()` function for overlay logic (matching events page)
- ✅ Import `isTicketedFundraiserEvent` from `@/lib/donation/utils` for event type detection
- ✅ Check if event is upcoming (today or future) before showing overlay
- ✅ Use image overlays (`<img>`) instead of Next.js `Image` component for overlay buttons
- ✅ Use responsive sizing: `w-[150px] h-[52px] sm:w-[200px] sm:h-[70px]` (matches events page)
- ✅ Clean up timeouts in useEffect cleanup
- ✅ Use `onClick={(e) => e.stopPropagation()}` on overlay links to prevent event bubbling
- ✅ Keep refs synchronized with state using dedicated `useEffect` hooks
- ✅ Reset guards (`lastScheduledIndexRef.current = null`) in cleanup functions

### **DON'T:**
- ❌ Skip default image display (always show for 2 seconds)
- ❌ Use hardcoded rotation intervals (always use per-image durations from `home_page_hero_display_duration_seconds`)
- ❌ **Access state directly in `scheduleNextRotation`** (always use refs: `imageDurationsRef.current`, `dynamicImagesRef.current`, etc.)
- ❌ **Include `scheduleNextRotation` in effect dependency array** (store in ref instead)
- ❌ **Schedule timeout without clearing existing one** (always clear `rotationTimeoutRef.current` first)
- ❌ **Skip duplicate prevention guard** (always check `lastScheduledIndexRef.current`)
- ❌ **Call `scheduleNextRotation` directly from state updater** (use `setTimeout` with delay)
- ❌ Show multiple occurrences of same recurring event series
- ❌ Show overlays on default or fallback images
- ❌ Show overlays for past events (only upcoming events)
- ❌ Forget to clean up timeouts
- ❌ Use hardcoded event selection (always use `useFilteredEvents` hook)
- ❌ Skip event sorting (always sort by startDate)
- ❌ Use different overlay logic than events page (must match for consistency)
- ❌ Use Next.js `Image` component for overlay buttons (use `<img>` tag instead)
- ❌ Skip upcoming event check (always verify event date is today or future)
- ❌ Use different image paths than events page (must match exactly)
- ❌ Show controls when there's only one image (only show for multiple images)
- ❌ Forget to update `currentEvent` when manually navigating (Previous/Next)
- ❌ Skip pause state in rotation effect dependencies (rotation must respect pause)
- ❌ Forget to clean up touch timeout on unmount (prevent memory leaks)
- ❌ Show controls permanently (only show on hover/touch)
- ❌ Forget to reset guards in cleanup functions

## **Error Handling**

### **Loading States**
```tsx
if (eventsLoading) {
  // Show loading state or default image
  return <DefaultImageDisplay />;
}

if (error) {
  // Show default image on error
  return <DefaultImageDisplay />;
}
```

### **Empty States**
```tsx
if (!filteredEvents || filteredEvents.length === 0) {
  // Show default image only (no rotation)
  setDynamicImages([defaultImage]);
}
```

### **Image Loading Errors**
```tsx
<Image
  src={overlay.image}
  onError={(e) => {
    // Fallback to buy tickets image if overlay image is missing
    const img = e.target as HTMLImageElement;
    img.src = '/images/buy_tickets_click_here_red.webp';
  }}
/>
```

## **Accessibility Requirements**

### **Image Alt Text**
- **Default Image**: `alt="Default Hero Image"`
- **Dynamic Images**: `alt="Dynamic Hero Image"` (or event-specific alt text if available)
- **Overlay Images**: `alt="{overlay.type} overlay"` (e.g., "tickets overlay")

### **Link Navigation**
- **Default Image**: Links to `/events`
- **Event Images**: Links to `/events/${event.id}`
- **Fallback Image**: Links to `/events`
- **Overlay Buttons**: Links to appropriate action route

## **Reference Implementations**

- **DynamicHeroImage Component (Rotation Logic)**: [`src/components/HeroSection.tsx`](mdc:src/components/HeroSection.tsx) - Lines 53-575 (Complete rotation implementation with per-image durations, refs, and duplicate prevention)
  - **Refs Declaration**: Lines 56-70 (All refs for avoiding stale closures)
  - **Ref Synchronization**: Lines 201-216 (useEffect hooks to keep refs updated)
  - **scheduleNextRotation Function**: Lines 221-296 (Recursive rotation with per-image durations and duplicate prevention)
  - **Rotation Effect**: Lines 299-324 (Main rotation effect without scheduleNextRotation in dependencies)
  - **Manual Navigation**: Lines 327-409 (goToPrevious and goToNext with proper cleanup)
- **HeroSection Component (Overlay Logic)**: [`src/components/HeroSection.tsx`](mdc:src/components/HeroSection.tsx) - Lines 221-323 (HeroSection component with overlay logic)
- **Events Page Overlay Pattern**: [`src/app/events/page.tsx`](mdc:src/app/events/page.tsx) - Lines 1409-1474 (Overlay image logic - matches hero section)
- **Event Type Detection**: [`src/lib/donation/utils.ts`](mdc:src/lib/donation/utils.ts) - `isTicketedFundraiserEvent()` function
- **useFilteredEvents Hook**: [`src/hooks/useFilteredEvents.ts`](mdc:src/hooks/useFilteredEvents.ts) - Hero filter implementation
- **Event Utils**: [`src/lib/eventUtils.ts`](mdc:src/lib/eventUtils.ts) - Recurring event handling (`isRecurringEvent`, `getNextOccurrenceDate`)
- **Home Page**: [`src/app/page.tsx`](mdc:src/app/page.tsx) - Uses `<HeroSection />` component
- **Per-Image Duration Documentation**: [`documentation/INTEGRATION_PROMPT_HOME_PAGE_HERO_DISPLAY_DURATION.md`](mdc:documentation/INTEGRATION_PROMPT_HOME_PAGE_HERO_DISPLAY_DURATION.md) - Complete specification for per-image duration feature
- **Documentation**: [`documentation/hero-image-selection-overlay-logic.md`](mdc:documentation/hero-image-selection-overlay-logic.md) - Complete specification

## **Troubleshooting**

### **Images Not Rotating?**
- Check that `dynamicImages.length > 0`
- Verify `scheduleNextRotation` is being called correctly
- Check that `useEffect` dependencies include `isInitialized`, `dynamicImages.length`, `upcomingEvents.length`, and `isPaused` (but NOT `scheduleNextRotation`)
- Verify `isPaused` is `false` (rotation stops when paused)
- Check that cleanup functions are properly clearing timeouts
- Verify rotation effect checks `isPaused` before scheduling timeout
- Check that refs are being updated when state changes (verify `useEffect` hooks for ref synchronization)

### **Duplicate Rotations (Images Flashing/Skipping)?**
- **Check**: Is `scheduleNextRotation` in the effect dependency array? (Should NOT be - causes duplicate rotations)
- **Check**: Is `lastScheduledIndexRef` being checked before scheduling? (Should prevent duplicates)
- **Check**: Is timeout being cleared before scheduling new one? (Should always clear `rotationTimeoutRef.current` first)
- **Check**: Is `scheduleNextRotationRef.current` being used instead of direct function call? (Should use ref)
- **Check**: Are multiple timeouts being created? (Should only have one active timeout at a time)

### **Missing Images in Rotation?**
- **Check**: Are refs being updated when state changes? (Should have `useEffect` hooks for each ref)
- **Check**: Is `scheduleNextRotation` accessing arrays from refs? (Should use `.current` not direct state)
- **Check**: Is guard being reset on cleanup? (Should set `lastScheduledIndexRef.current = null`)
- **Check**: Is effect restarting prematurely? (Should NOT include `currentImageIndex` in dependencies)

### **Standalone Hero Images Not Showing?**
- **Check**: In the database, standalone rows must have `event_id` IS NULL and **either** `is_home_page_hero_image = true` **or** `is_hero_image = true`. The frontend filters standalone media client-side by this rule (see **Standalone Hero Image Data Requirement** above).
- **Check**: Admin/INSERTs for standalone hero media should set at least one of `is_home_page_hero_image = true` or `is_hero_image = true`.
- **Backend**: The implementation does **not** use `eventId.specified=false`; it fetches by `isHeroImage.equals=true` and optionally `isHomePageHeroImage.equals=true`, then filters client-side for `event_id` null. If standalone still don't appear, confirm the backend returns media with `event_id` NULL when querying by `isHeroImage.equals=true` (or `isHomePageHeroImage.equals=true`), and that `size` is large enough (30 or 50).

### **All Images Using 8 Seconds (Per-Image Duration Not Working)?**
- **Check**: Is `imageDurationsRef.current` being used instead of `imageDurations`? (Should use ref to avoid stale closure)
- **Check**: Is ref being updated when `imageDurations` changes? (Should have `useEffect` hook)
- **Check**: Is `imageDurations` array being populated correctly during initialization? (Should match `dynamicImages` array length)
- **Check**: Are durations being calculated correctly from `home_page_hero_display_duration_seconds`? (Should convert seconds to milliseconds: `duration * 1000`)

### **Overlay Not Showing?**
- Check that `currentEvent` is set correctly when image changes
- Verify event is upcoming (today or future) - past events don't show overlay
- Check that `getOverlayInfo()` returns correct overlay for event type
- Verify event is ticketed (`admissionType === 'TICKETED'`)
- Check that `isTicketedFundraiserEvent()` is imported and working correctly
- Verify image paths are correct (`/images/buy_tickets_click_here_red.webp` or `/images/buy_tickets_click_here_fundraiser.png`)
- Check browser console for image loading errors

### **Wrong Events Showing?**
- Verify `useFilteredEvents('hero')` is filtering correctly
- Check that events have `isHomePageHeroImage = true` in media
- Verify events are future events (`startDate >= today`)
- Check recurring event handling (should show only next occurrence)

### **Recurring Events Showing Multiple Times?**
- Verify `recurringSeriesMap` is grouping by `recurrenceSeriesId` or `parentEventId`
- Check that only earliest next occurrence is kept per series
- Verify child events are being skipped

### **Rotation Timing Issues?**
- Verify default image timer is 2000ms (2 seconds)
- Check per-image durations are being used (from `home_page_hero_display_duration_seconds`, default 8000ms if NULL)
- Verify rotation start delay is 2000ms (2 seconds)
- Check that cleanup functions are clearing timeouts properly
- Verify `isPaused` state is included in rotation effect dependencies
- Check that pause button correctly toggles `isPaused` state
- Verify durations are being accessed from `imageDurationsRef.current` (not from closure)
- Check that `imageDurations` array matches `dynamicImages` array length

## **Related Patterns**

- See [Image Containment Prevention](mdc:.cursor/rules/image_containment_prevention.mdc) for hero image display patterns
- See [Loading Animation Pattern](mdc:.cursor/rules/loading_animation_pattern.mdc) for loading states (different from rotation)
- See [`src/components/HeroSection.tsx`](mdc:src/components/HeroSection.tsx) for complete implementation
- See [`documentation/hero-image-selection-overlay-logic.md`](mdc:documentation/hero-image-selection-overlay-logic.md) for detailed specification

## **Summary**

**Key Pattern**: Hero section image rotation should:
- Show default image for 2 seconds
- Rotate through event images with **per-image durations** from `home_page_hero_display_duration_seconds` (default 8 seconds if NULL)
- **Use refs** (`imageDurationsRef.current`, `dynamicImagesRef.current`, etc.) to avoid stale closures
- **Prevent duplicate rotations** using `lastScheduledIndexRef` guard and clearing timeouts before scheduling
- **Store rotation function in ref** (`scheduleNextRotationRef`) to avoid dependency issues
- Update `currentEvent` state when image changes
- Display "Buy Tickets Click Here" image overlays based on current event type (matching events page)
- Handle recurring events (show only next occurrence)
- Sort events by startDate (earliest first)
- Include fallback image at end of rotation array
- Use `useFilteredEvents('hero')` hook for event selection
- Clean up timeouts properly
- Only show overlays for upcoming events (today or future)
- Provide interactive controls (Previous/Next/Play/Pause) for manual navigation
- Show controls only on hover (desktop) or touch (mobile)
- Pause auto-rotation when user clicks pause button
- Update `currentEvent` when navigating manually (Previous/Next buttons)

**Critical Timing Values**:
- Default image: 2000ms (2 seconds)
- Default rotation interval: 8000ms (8 seconds) - used when `home_page_hero_display_duration_seconds` is NULL
- Per-image duration: From `event_media.home_page_hero_display_duration_seconds` (1-600 seconds, converted to milliseconds)
- Transition duration: 400ms (fade transition)
- Next rotation scheduling delay: 10ms (deferred after state update)

**Event Selection**:
- Primary: `isHomePageHeroImage = true` AND `startDate >= today`
- Additional: `isActive = true` AND `startDate <= oneYearFromNow`
- Recurring: Show only next occurrence, one per series

**Overlay Image Pattern**:
- **Ticketed Fundraiser/Charity**: `/images/buy_tickets_click_here_fundraiser.png` → `/events/${event.id}/donation-checkout`
- **Regular Ticketed**: `/images/buy_tickets_click_here_red.webp` → `/events/${event.id}/checkout` or `/events/${event.id}/manual-checkout`
- **Position**: Bottom-right corner (`absolute bottom-4 right-4 z-10`)
- **Sizing**: Responsive (`w-[150px] h-[52px] sm:w-[200px] sm:h-[70px]`)
- **Styling**: `object-contain` with hover scale effect
- **Only for Upcoming Events**: Past events don't show overlay

This ensures consistent, automatic hero image rotation with synchronized overlay buttons across all event types.

## **Video-Like Animated Images (Movie Trailer Effect)**

### **Overview**
The hero section currently supports static image uploads only. To achieve video-like cinematic effects (movie trailer animations), several options are available, each with different trade-offs.

### **Current System Limitations**
- **Image Uploads Only**: System currently supports image file uploads
- **Filtering**: Images filtered by `isHomePageHeroImage = true` flag
- **Display**: Uses Next.js `Image` component for all hero images
- **Media Type**: `thumbnailUrl` from event media is used for display

### **Recommended Options (In Order of Preference)**

#### **1. Animated WebP (Best for Current System)** ⭐ **RECOMMENDED**

**Advantages:**
- ✅ Works with existing image upload system (no code changes needed)
- ✅ Better compression than GIF (smaller file sizes)
- ✅ Good browser support (Chrome, Firefox, Edge, Safari 14+)
- ✅ Maintains image quality while adding animation
- ✅ Can be uploaded immediately using current media upload interface

**Implementation:**
1. Convert video/movie trailer to animated WebP format using tools like:
   - FFmpeg: `ffmpeg -i input.mp4 -vf "fps=10,scale=800:-1:flags=lanczos" -loop 0 output.webp`
   - Online converters (CloudConvert, EZGIF)
   - Photoshop/GIMP with WebP export plugin
2. Upload animated WebP file through existing media upload interface
3. Mark as `isHomePageHeroImage = true` in media settings
4. System treats it as regular image - works immediately

**File Specifications:**
- **Format**: Animated WebP (`.webp`)
- **Frame Rate**: 10-15 fps (balance between smoothness and file size)
- **Duration**: 3-10 seconds (loops automatically)
- **Dimensions**: Match hero image specs (800×1200px desktop, 600×900px mobile)
- **File Size**: Target under 500KB (animated WebP compresses well)

**Browser Support:**
- Chrome/Edge: Full support
- Firefox: Full support
- Safari: 14+ (full support), older versions fallback to static image

---

#### **2. Animated GIF (Universal Fallback)**

**Advantages:**
- ✅ Universal browser support (works everywhere)
- ✅ No code changes needed
- ✅ Simple upload process (same as WebP)

**Disadvantages:**
- ❌ Larger file sizes than WebP (typically 2-3x larger)
- ❌ Limited color palette (256 colors max)
- ❌ No transparency support (or limited)

**Implementation:**
1. Convert video to animated GIF using:
   - FFmpeg: `ffmpeg -i input.mp4 -vf "fps=10,scale=800:-1:flags=lanczos" -loop 0 output.gif`
   - Online converters (EZGIF, CloudConvert)
   - Photoshop/GIMP
2. Upload GIF file through existing media upload interface
3. Mark as `isHomePageHeroImage = true`
4. Works immediately (no code changes)

**File Specifications:**
- **Format**: Animated GIF (`.gif`)
- **Frame Rate**: 8-12 fps (to keep file size manageable)
- **Duration**: 3-8 seconds (loops automatically)
- **Dimensions**: Match hero image specs
- **File Size**: Target under 1MB (GIFs are larger than WebP)

---

#### **3. CSS Animations on Static Images (Lightweight)**

**Advantages:**
- ✅ No file size increase (uses existing static images)
- ✅ Smooth, performant animations (GPU-accelerated)
- ✅ Flexible animation effects (Ken Burns, fade, scale, pan)
- ✅ Works with all browsers

**Disadvantages:**
- ❌ Requires code changes to detect and apply animations
- ❌ Limited to CSS-based effects (no complex video-like motion)
- ❌ Requires adding animation flag to media metadata

**Implementation Steps:**

**Step 1: Add Animation Flag to Media**
```typescript
// In media upload interface, add checkbox:
// "Enable CSS Animation" → sets hasAnimation: true in media metadata
```

**Step 2: Update Hero Component**
```tsx
// Detect animated images
const hasAnimation = currentEvent?.media?.[0]?.hasAnimation || false;

// Apply CSS animation class
<Image
  src={currentImage}
  alt="Featured Event"
  fill
  className={`object-contain hero-image-transition ${
    isTransitioning ? 'transitioning' : ''
  } ${hasAnimation ? 'hero-image-animated' : ''}`}
  sizes="100vw"
  priority
/>
```

**Step 3: Add CSS Animations**
```css
/* Ken Burns Effect (zoom + pan) */
.hero-image-animated {
  animation: kenBurns 15s ease-in-out infinite;
}

@keyframes kenBurns {
  0% {
    transform: scale(1) translate(0, 0);
  }
  50% {
    transform: scale(1.1) translate(-2%, -2%);
  }
  100% {
    transform: scale(1) translate(0, 0);
  }
}

/* Alternative: Fade pulse effect */
.hero-image-animated-pulse {
  animation: fadePulse 3s ease-in-out infinite;
}

@keyframes fadePulse {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0.9;
  }
}
```

**Animation Options:**
- **Ken Burns Effect**: Slow zoom + pan (cinematic)
- **Fade Pulse**: Subtle brightness/opacity changes
- **Scale Breathe**: Gentle scale in/out
- **Pan Left/Right**: Horizontal movement

---

#### **4. Future: Video Upload Support (Most Flexible)**

**Advantages:**
- ✅ Full video quality and control
- ✅ Supports audio (if needed)
- ✅ Native video playback controls
- ✅ Best for true movie trailer experience

**Disadvantages:**
- ❌ Requires significant backend and frontend changes
- ❌ Larger file sizes (requires video hosting/CDN)
- ❌ More complex implementation

**Implementation Requirements:**

**Backend Changes:**
1. Extend media upload API to accept video files (MP4, WebM)
2. Add `eventMediaType` field to distinguish video from image
3. Store video files in media storage (S3/CDN)
4. Add video metadata (duration, format, resolution)

**Frontend Changes:**
1. Update media upload component to accept video files
2. Add video preview in upload interface
3. Update hero component to detect video vs image:
```tsx
const isVideo = currentEvent?.media?.[0]?.eventMediaType?.startsWith('video/');
const mediaUrl = currentEvent?.thumbnailUrl || currentEvent?.media?.[0]?.fileUrl;

{isVideo ? (
  <video
    src={mediaUrl}
    autoPlay
    loop
    muted
    playsInline
    className="object-contain hero-image-transition"
    style={{ width: '100%', height: '100%' }}
  />
) : (
  <Image
    src={mediaUrl}
    alt="Featured Event"
    fill
    className="object-contain hero-image-transition"
    sizes="100vw"
    priority
  />
)}
```

**Video Specifications:**
- **Format**: MP4 (H.264) or WebM (VP9)
- **Resolution**: 1920×1080 (Full HD) or 1280×720 (HD)
- **Frame Rate**: 24-30 fps
- **Duration**: 10-30 seconds (loops automatically)
- **File Size**: Target under 5MB (compressed)
- **Audio**: Muted (autoplay requires muted for browser compatibility)

---

### **Quick Implementation Path**

#### **Option A: Animated WebP (No Code Changes)** ⭐ **START HERE**

**Steps:**
1. Convert video/movie trailer to animated WebP:
   ```bash
   # Using FFmpeg
   ffmpeg -i trailer.mp4 -vf "fps=12,scale=800:-1:flags=lanczos" \
     -loop 0 -quality 85 hero-trailer.webp
   ```
2. Upload through existing media upload interface
3. Mark as `isHomePageHeroImage = true`
4. **Works immediately** - no code changes needed

**Tools for Conversion:**
- **FFmpeg** (command-line, most control)
- **CloudConvert** (online, easy to use)
- **EZGIF** (online, GIF/WebP converter)
- **Photoshop** (with WebP export plugin)

---

#### **Option B: CSS Animations (Code Changes Required)**

**Steps:**
1. Add `hasAnimation` boolean field to media metadata
2. Update hero component to detect animation flag
3. Add CSS animation classes to `globals.css`
4. Apply animation class conditionally in hero component
5. Upload static images with animation flag enabled

**Code Changes:**
- Update `HeroSection.tsx` to check `hasAnimation` flag
- Add CSS animation keyframes to `globals.css`
- Update media upload interface to include animation checkbox

---

#### **Option C: Video Support (Larger Implementation)**

**Steps:**
1. **Backend**: Extend media upload API to accept video files
2. **Backend**: Add video storage and metadata handling
3. **Frontend**: Update media upload component for video files
4. **Frontend**: Update hero component to render `<video>` for videos
5. **Frontend**: Add video autoplay, loop, and muted attributes
6. **Testing**: Test video playback across browsers and devices

**Estimated Effort:**
- Backend changes: 2-4 hours
- Frontend changes: 2-3 hours
- Testing and refinement: 1-2 hours
- **Total: 5-9 hours**

---

### **Recommendation Summary**

**For Immediate Results:**
- ✅ **Start with Animated WebP (Option A)**
  - No code changes needed
  - Works with existing system
  - Good quality and file size
  - Can be implemented today

**For Enhanced Flexibility:**
- ✅ **Add Video Support (Option C) Later**
  - Full video quality and control
  - Best for true movie trailer experience
  - Requires development time but provides most flexibility

**For Lightweight Alternative:**
- ✅ **CSS Animations (Option B)**
  - No file size increase
  - Smooth, performant effects
  - Good for subtle cinematic effects

### **File Format Comparison**

| Format | File Size | Quality | Browser Support | Code Changes | Best For |
|--------|-----------|---------|-----------------|--------------|----------|
| **Animated WebP** | Small (200-500KB) | High | Good (Safari 14+) | None | ⭐ **Recommended** |
| **Animated GIF** | Medium (500KB-2MB) | Medium | Universal | None | Universal fallback |
| **CSS Animations** | None (uses existing) | High | Universal | Required | Lightweight effects |
| **Video (MP4/WebM)** | Large (2-10MB) | Highest | Universal | Required | Full video experience |

### **Best Practices**

**For Animated WebP/GIF:**
- ✅ Keep duration short (3-10 seconds) for smooth looping
- ✅ Optimize frame rate (10-15 fps for WebP, 8-12 fps for GIF)
- ✅ Compress files to target sizes (WebP: <500KB, GIF: <1MB)
- ✅ Test on multiple browsers before deploying
- ✅ Provide fallback static image for older browsers

**For CSS Animations:**
- ✅ Use GPU-accelerated properties (`transform`, `opacity`)
- ✅ Keep animations subtle and smooth
- ✅ Test performance on mobile devices
- ✅ Provide option to disable animations for accessibility

**For Video Support:**
- ✅ Always include `muted` attribute (required for autoplay)
- ✅ Use `playsInline` for mobile compatibility
- ✅ Compress videos to reasonable file sizes (<5MB)
- ✅ Provide fallback image if video fails to load
- ✅ Consider video CDN for large files

### **Accessibility Considerations**

- **Motion Sensitivity**: Provide option to disable animations for users with motion sensitivity
- **Performance**: Monitor performance impact, especially on mobile devices
- **Bandwidth**: Consider users on slow connections - provide loading states
- **Fallbacks**: Always provide static image fallback for animated content

### **Testing Checklist**

- [ ] Animated WebP/GIF loops smoothly without stuttering
- [ ] File sizes are within acceptable limits (<500KB WebP, <1MB GIF)
- [ ] Animations work on mobile devices (iOS Safari, Android Chrome)
- [ ] Fallback static image displays on older browsers
- [ ] Performance is acceptable (no frame drops, smooth transitions)
- [ ] Animations don't interfere with overlay buttons or controls
- [ ] Video autoplay works (muted, loop, playsInline attributes)
- [ ] CSS animations are GPU-accelerated and performant

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
