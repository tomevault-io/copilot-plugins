## focal-point

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Standard Workflow
1. First think through the problem, read the codebase for relevant files, and write a plan to todo.md.
2. The plan should have a list of todo items that you can check off as you complete them
3. Before you begin working, check in with me and I will verify the plan.
4. Then, begin working on the todo items, marking them as complete as you go.
5. Please every step of the way just give me a high level explanation of what changes you made
6. Make every task and code change you do as simple as possible. We want to avoid making any massive or complex changes. Every change should impact as little code as possible. Everything is about simplicity.
7. Finally, add a review section to the todo.md file with a summary of the changes you made and any other relevant information.

## Development Commands

```bash
# Start development server
npm start

# Build for production
npm run build

# Run tests
npm test

# Run linting
npm run lint
npm run lint:fix
```

## Architecture Overview

### Core Technology Stack
- **React 18** with functional components and hooks
- **Firebase** for authentication, Firestore database, and file storage
- **React Router** for client-side routing
- **Lucide React** for icons
- **Bootstrap** for base styling with custom CSS variables

### Application Structure

**Authentication Flow:**
- `AuthProvider` manages global auth state via `AuthContext`
- Firebase Authentication with organization-based multi-tenancy
- Role-based access control (admin, photographer, etc.)
- Protected routes require authentication

**Layout Architecture:**
- `Layout` component contains fixed `Sidebar` and `Header`
- Main content area dynamically loads page components
- Sidebar navigation items defined in `/src/components/layout/Sidebar.js`

**Context Providers:**
- `AuthContext` - user authentication, profile, and organization data
- `JobsContext` - sports job management and data operations  
- `ChatContext` - real-time messaging with cache-first loading (REFERENCE IMPLEMENTATION)
- `ToastContext` - global notification system
- `DataCacheContext` - centralized cache management and read tracking

**Modal Rendering:**
- Modals use `ReactDOM.createPortal` to render at document.body level
- Critical: Use inline styles for overlay positioning to avoid CSS conflicts
- Modal overlay must have `position: fixed`, `zIndex: 10001`, and flexbox centering
- Inner modal container needs `position: relative`, `margin: 0`, `transform: none`

### Key Modules

**Sports Module (`/src/components/sports/`):**
- Comprehensive sports photography job management
- File upload and roster management with Excel/CSV support
- Real-time job tracking and statistics
- Player search across jobs and rosters

**Firebase Integration:**
- Configuration in `/src/firebase/config.js` with environment variable fallbacks and offline persistence
- Firestore collections: users, organizations, schools, sportsJobs, conversations, messages
- File storage for user photos and roster uploads  
- Real-time listeners with cache optimization (see `ChatContext` for reference implementation)
- ReadCounter system for monitoring Firebase usage and costs (`/src/services/readCounter.js`)
- Cache services for efficient data management (`/src/services/chatCacheService.js` as reference)

**Styling System:**
- CSS custom properties defined in `/src/styles/variables.css`
- Component-specific CSS files follow naming convention: `ComponentName.css`
- Global styles in `/src/styles/globals.css`
- Sports module has dedicated styling in `/src/styles/sports/`

### Data Flow Patterns

**Cache-First Data Loading (STANDARD PATTERN):**
- Load from cache immediately for instant display
- Fetch updates from Firestore in background  
- Use optimized real-time listeners for incremental updates
- Track all operations with ReadCounter for cost monitoring
- Reference implementation: `ChatContext` and `ChatCacheService`

**User Profile Management:**
- User profile linked to organization via `organizationID`
- Profile photos with crop functionality and multiple image formats
- Settings modals for both user profile and studio/organization settings
- **REQUIRED**: Implement caching for user profile data

**Job Management (Sports):**
- Jobs associated with schools and organizations
- Roster data uploaded via Excel/CSV with real-time preview
- Job status tracking (active, completed, archived)
- Search functionality across jobs and player rosters
- **REQUIRED**: Implement cache-first loading for job lists and search results

**Real-Time Messaging (REFERENCE IMPLEMENTATION):**
- Cache-first message loading with instant display
- Optimized listeners that only fetch new messages
- Local storage persistence with automatic cleanup
- ReadCounter integration for cost tracking
- Cross-platform data synchronization ready

**File Upload Patterns:**
- User profile photos: original + cropped versions stored
- Roster files: Excel/CSV processing with validation and preview
- All uploads use Firebase Storage with organized folder structure

### CSS Architecture Notes

**Modal Positioning:**
- Always import both `../shared/Modal.css` and component-specific CSS
- Use React portals with `ReactDOM.createPortal(content, document.body)`
- Inline styles required for overlay to override layout constraints

**Layout Constraints:**
- Sidebar has fixed positioning with `--sidebar-width` CSS variable
- Main content area has left margin equal to sidebar width
- Header is fixed height using `--header-height` variable

### Environment Configuration

Firebase configuration supports environment variables:
- `REACT_APP_FIREBASE_API_KEY`
- `REACT_APP_FIREBASE_AUTH_DOMAIN`
- `REACT_APP_FIREBASE_PROJECT_ID`
- `REACT_APP_FIREBASE_STORAGE_BUCKET`
- `REACT_APP_FIREBASE_MESSAGING_SENDER_ID`
- `REACT_APP_FIREBASE_APP_ID`
- `REACT_APP_FIREBASE_MEASUREMENT_ID`

Fallback values are provided for development use.

## Firebase Cost Optimization & Caching

### Critical Context
This application previously experienced a **58 million Firebase read incident** that resulted in significant unexpected costs. To prevent this from happening again, **ALL** new development must prioritize Firebase read efficiency and implement comprehensive caching.

### Mandatory Requirements

**1. Cache-First Data Loading (REQUIRED)**
- Every new component that reads Firestore data MUST implement cache-first loading
- Load from cache immediately for instant display, then fetch updates from Firestore
- Use `chatCacheService` pattern as the standard implementation example
- Store data in localStorage with proper serialization/deserialization

**2. ReadCounter Integration (REQUIRED)**
- ALL Firestore operations must be tracked in the ReadCounter system
- Record cache hits: `readCounter.recordCacheHit(collection, component, savedReads)`
- Record cache misses: `readCounter.recordCacheMiss(collection, component)`
- Record actual reads: `readCounter.recordRead(operation, collection, component, count)`

**3. Real-Time Listener Optimization (REQUIRED)**
- Use optimized listeners that only fetch new data after cached timestamps
- Implement `startAfter()` queries to avoid re-reading cached data
- Follow the pattern in `chatService.subscribeToNewMessages()`
- Always clean up listeners properly to prevent memory leaks

### Implementation Patterns

**Cache Service Template:**
```javascript
// Required imports
import { readCounter } from '../services/readCounter';

// Cache-first loading pattern
const cachedData = cacheService.getCachedData(key);
if (cachedData) {
  setData(cachedData);
  readCounter.recordCacheHit('collection', 'Component', cachedData.length);
} else {
  readCounter.recordCacheMiss('collection', 'Component');
}

// Optimized real-time listener
const latestTimestamp = cacheService.getLatestTimestamp(key);
const unsubscribe = service.subscribeToNewData(
  key,
  (newData, isIncremental) => {
    if (isIncremental) {
      const updatedData = cacheService.appendNewData(key, newData);
      setData(updatedData);
    } else {
      setData(newData);
      cacheService.setCachedData(key, newData);
    }
  },
  latestTimestamp
);
```

**Prohibited Patterns:**
- ❌ Direct Firestore queries without caching layer
- ❌ Real-time listeners without cache optimization
- ❌ Pagination without cache management
- ❌ Any data fetching without ReadCounter tracking

### Development Rules

**Before Writing Any Firebase Code:**
1. Plan the caching strategy first
2. Identify what data can be cached and for how long
3. Design cache invalidation strategy
4. Implement ReadCounter tracking from the start

**Code Review Requirements:**
- Firebase read efficiency check is mandatory
- Cache implementation must be verified
- ReadCounter integration must be validated
- Cost impact assessment required for new features

**Monitoring:**
- Use ReadCounterWidget to monitor Firebase usage in real-time
- Aim for >80% cache hit rates on returning user sessions
- Track cost savings and projected monthly costs
- Set up alerts if daily read counts exceed expected thresholds

### Caching Implementation Guidelines

**Cache Service Architecture:**
Every data type should have a dedicated cache service following the `ChatCacheService` pattern:

```javascript
class DataCacheService {
  constructor() {
    this.CACHE_PREFIX = 'focal_data_';
    this.CACHE_VERSION = '1.0';
    this.MAX_CACHE_AGE = 7 * 24 * 60 * 60 * 1000; // 7 days
  }

  // Required methods for all cache services:
  getCachedData(key) { /* implementation */ }
  setCachedData(key, data) { /* implementation */ }
  clearCache(key) { /* implementation */ }
  getLatestTimestamp(key) { /* implementation */ }
  appendNewData(key, newData) { /* implementation */ }
}
```

**LocalStorage Management:**
- Use consistent cache key naming: `focal_{dataType}_{identifier}`
- Implement cache versioning to handle data structure changes
- Set reasonable expiration times (1-7 days depending on data type)
- Limit cache size to prevent localStorage overflow
- Handle serialization/deserialization of Firebase Timestamps properly

**Cache Invalidation Strategies:**
- **Time-based**: Automatic expiration after set duration
- **Event-based**: Clear cache when relevant data changes
- **Version-based**: Clear incompatible cache when data structure changes
- **Manual**: Provide cache clear functions for debugging

**Offline Persistence:**
- Enable Firestore offline persistence: `enableNetwork(firestore)`
- Combine with localStorage caching for maximum efficiency
- Handle network state changes gracefully
- Sync cached data when connection restored

### Firestore Development Rules

**Data Fetching Rules:**

1. **NO Direct Queries Without Caching**
   ```javascript
   // ❌ PROHIBITED - Direct query without caching
   const docs = await getDocs(collection(firestore, 'users'));
   
   // ✅ REQUIRED - Cache-first approach
   const cachedUsers = userCacheService.getCachedUsers();
   if (cachedUsers) {
     setUsers(cachedUsers);
     readCounter.recordCacheHit('users', 'UserList', cachedUsers.length);
   } else {
     readCounter.recordCacheMiss('users', 'UserList');
     const docs = await getDocs(collection(firestore, 'users'));
     // Process and cache results
   }
   ```

2. **Real-Time Listener Optimization**
   ```javascript
   // ❌ PROHIBITED - Full collection listener
   onSnapshot(collection(firestore, 'messages'), callback);
   
   // ✅ REQUIRED - Optimized listener with cache integration
   const latestTimestamp = cacheService.getLatestTimestamp();
   onSnapshot(
     query(collection(firestore, 'messages'), 
           where('timestamp', '>', latestTimestamp)),
     callback
   );
   ```

3. **Pagination Requirements**
   - Always check cache before fetching next page
   - Use `startAfter()` with cached document references
   - Implement cache-aware pagination logic
   - Track pagination reads separately in ReadCounter

**Context and Service Layer Rules:**

1. **Context Implementation**
   - All data contexts MUST implement cache-first loading
   - Include ReadCounter tracking in all data operations
   - Use optimized real-time listeners
   - Implement proper cleanup to prevent memory leaks

2. **Service Layer Standards**
   - Create dedicated cache services for each data type
   - Implement both cached and live data methods
   - Include ReadCounter integration in all service methods
   - Design for offline-first functionality

**Performance Standards:**

- **Initial Load**: <2 seconds for cached data display
- **Cache Hit Rate**: >80% for returning users within 24 hours
- **Read Efficiency**: <10 reads per user session for cached content
- **Memory Usage**: localStorage cache <10MB per user
- **Network Requests**: Minimize redundant Firestore calls

**Error Handling:**
- Graceful fallback when cache is corrupted
- Network error recovery with cached data
- ReadCounter error tracking for debugging
- User feedback for cache-related issues

### Code Review Checklist

**Firebase Efficiency Review (MANDATORY):**

Before approving any code that interacts with Firestore, verify:

✅ **Caching Implementation**
- [ ] Cache-first data loading implemented
- [ ] LocalStorage caching with proper serialization
- [ ] Cache expiration and versioning strategy
- [ ] Cache invalidation logic present

✅ **ReadCounter Integration**
- [ ] All Firestore reads tracked with `readCounter.recordRead()`
- [ ] Cache hits recorded with `readCounter.recordCacheHit()`
- [ ] Cache misses recorded with `readCounter.recordCacheMiss()`
- [ ] Proper collection and component names provided

✅ **Real-Time Listener Optimization**
- [ ] Listeners use optimized queries (no full collection scans)
- [ ] Cache-aware listener implementation
- [ ] Proper cleanup in useEffect return or component unmount
- [ ] Timestamp-based filtering for incremental updates

✅ **Data Loading Patterns**
- [ ] No direct Firestore queries without caching layer
- [ ] Instant cached data display before live data fetch
- [ ] Proper error handling for cache failures
- [ ] Loading states managed correctly

✅ **Performance Validation**
- [ ] ReadCounterWidget shows acceptable read counts
- [ ] Cache hit rate >80% for returning users
- [ ] No redundant Firestore operations
- [ ] Memory usage within acceptable limits

✅ **Cost Impact Assessment**
- [ ] Estimated reads per user calculated
- [ ] Monthly cost projection acceptable
- [ ] Comparison with previous implementation
- [ ] Alternative optimization strategies considered

**Code Quality Standards:**
- [ ] Follows existing caching patterns (`ChatCacheService` example)
- [ ] Proper TypeScript/JSDoc documentation
- [ ] Error boundaries for cache-related issues
- [ ] Unit tests for cache functionality
- [ ] Integration tests for read optimization

**Red Flags (REJECT CODE):**
- ❌ Direct `getDocs()` or `onSnapshot()` without caching
- ❌ Missing ReadCounter tracking
- ❌ Full collection real-time listeners
- ❌ No cache invalidation strategy
- ❌ Estimated reads >100 per user session for new features

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jsunwilke) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
