## clerk-auth-admin-user-lookup-avatar

> Clerk authentication middleware configuration, admin role lookup, and satellite/primary sign-in and sign-out patterns for Next.js App Router


# Clerk Authentication and Admin Role Lookup Pattern

## **Overview**
This rule defines the correct pattern for configuring Clerk middleware to enable `auth()` calls in server components, particularly for admin role lookup in the root layout. It explains when routes should be in `ignoredRoutes` vs `publicRoutes`, how to properly check admin status after user login, and **why the admin check must run on all routes (including public pages)** so the Admin menu appears in the Header on every page—not only on `/admin` and sub-pages.

## **Problem Solved**
- **Admin Menu Not Appearing**: Ensures admin role lookup works correctly after user login
- **Admin Menu on Public Pages**: Ensures the Admin menu appears when a logged-in admin visits **any** page (homepage, events, gallery, pricing, etc.), not only `/admin` and sub-pages
- **Clerk auth() Errors**: Prevents "Clerk can't detect usage of authMiddleware()" errors
- **Next.js 15+ Compatibility**: Properly handles async `headers()` calls
- **Route Configuration**: Clarifies when routes need Clerk middleware vs when they can be ignored

---

## **Core Pattern: Middleware Configuration**

### **Routes That Call `auth()` MUST NOT Be in `ignoredRoutes`**

**CRITICAL**: Any route that calls `auth()` or `currentUser()` in server components **MUST** have Clerk middleware running. This means:

- ✅ **DO**: Keep route in `publicRoutes` (if it should be accessible without auth)
- ❌ **DON'T**: Put route in `ignoredRoutes` (this bypasses Clerk middleware completely)

**Example - Homepage Admin Lookup**:
```typescript
// ✅ DO: Homepage in publicRoutes but NOT in ignoredRoutes
export default authMiddleware({
  publicRoutes: [
    '/',  // Homepage - accessible without auth, but middleware runs for auth() calls
    // ... other public routes
  ],

  ignoredRoutes: [
    // ❌ DON'T: '/',  // This would break auth() calls in layout.tsx
    '/events(.*)',  // OK - these pages don't call auth()
    '/gallery(.*)',  // OK - these pages don't call auth()
    // ... other routes that don't need auth()
  ],
});
```

---

## **When to Use `ignoredRoutes` vs `publicRoutes`**

### **Use `ignoredRoutes` When:**
- ✅ Route **does NOT** call `auth()` or `currentUser()` in server components
- ✅ Route is a static page with no authentication needs
- ✅ Route is an API endpoint that handles its own authentication
- ✅ Route is for automated testing (Playwright, etc.) that needs no Clerk middleware

**Examples**:
```typescript
ignoredRoutes: [
  '/events(.*)',      // Static public pages, no auth() calls
  '/gallery(.*)',     // Static public pages, no auth() calls
  '/api/proxy/(.*)',  // API routes handle their own auth
  '/api/webhooks/(.*)', // Webhook routes use service JWT, not Clerk
],
```

### **Use `publicRoutes` (NOT `ignoredRoutes`) When:**
- ✅ Route **DOES** call `auth()` or `currentUser()` in server components
- ✅ Route needs to check user authentication status (even if login not required)
- ✅ Route needs to look up user profile or admin status
- ✅ Route should be accessible without login, but needs Clerk middleware for `auth()` calls

**Examples**:
```typescript
publicRoutes: [
  '/',              // Homepage - layout.tsx calls auth() for admin lookup
  '/polls(.*)',     // Poll pages call auth() to check user participation
  '/pricing(.*)',   // Pricing page calls auth() to check subscription status
  '/profile(.*)',   // Profile pages need auth() to get user data
],
```

---

## **Admin Role Lookup Pattern in Root Layout**

### **Server-Side Admin Check in `src/app/layout.tsx`**

**Pattern**:
```typescript
export default async function RootLayout({ children }: { children: React.ReactNode }) {
  // CRITICAL: Next.js 15+ requires headers() to be awaited first
  const headersList = await headers();
  const hostname = headersList.get('host') || '';

  // ... other setup code ...

  // Determine tenant-scoped admin flag on the server
  let isTenantAdmin = false;
  try {
    // CRITICAL: Call auth() AFTER headers() is awaited
    const authResult = await auth();
    const userId = authResult?.userId || null;

    if (userId) {
      const tenantId = getTenantId();

      // Fetch user profile to check admin role
      const url = `${baseUrl}/api/proxy/user-profiles?userId.equals=${userId}&tenantId.equals=${tenantId}&size=1`;
      const resp = await fetch(url, { cache: 'no-store' });

      if (resp.ok) {
        const data = await resp.json();
        const profile = Array.isArray(data) ? data[0] : data;
        isTenantAdmin = profile?.userRole === 'ADMIN';
      }
    }
  } catch (error) {
    // Fail closed (no admin) on error
    console.error('[Layout] Error determining admin status:', error);
    isTenantAdmin = false;
  }

  return (
    <ClerkProvider {...clerkProps}>
      {/* ... */}
      <Header isTenantAdmin={isTenantAdmin} />
      {/* ... */}
    </ClerkProvider>
  );
}
```

### **Admin Check on ALL Routes (Including Public Pages)**

**CRITICAL**: Run the admin check on **every route** where the root layout renders (i.e., whenever `pathname` is present). Do **not** limit the admin lookup to `/admin` or protected routes only.

**Why**: The Header is rendered by the root layout for all non-MOSC/non-Syro routes. If the admin check is skipped on public routes (e.g. homepage, `/events`, `/gallery`, `/pricing`), then when a logged-in admin visits a public page, `isTenantAdmin` will be `false` or stale, and the Admin menu will not appear until they navigate to `/admin`.

**Correct pattern in root layout**:
```typescript
// ✅ DO: Run admin check whenever pathname is present (all pages that use main layout)
if (pathname) {
  try {
    const authResult = await auth();
    const userId = authResult?.userId || null;
    if (userId) {
      // ... fetch profile, set isTenantAdmin
    }
  } catch (error) {
    isTenantAdmin = false;
  }
}
return (
  <ClerkProvider>
    <Header isTenantAdmin={isTenantAdmin} />
    {children}
  </ClerkProvider>
);
```

**Anti-pattern**:
```typescript
// ❌ DON'T: Only run admin check on /admin routes
if (pathname?.startsWith('/admin')) {
  // ... admin lookup
}
// Result: Admin menu missing on homepage, /events, /gallery, etc.
```

**Reference**: [`src/app/layout.tsx`](mdc:src/app/layout.tsx) - Lines 102-107 ("Run admin check on ALL routes (including public) so the Header shows the Admin menu when a logged-in admin visits any page (e.g. homepage).")

### **Skip-Auth Logic in Root Layout (`skipAuthForRoute`)**

The root layout uses a `skipAuthForRoute` flag to skip the auth + profile lookup block on routes that do not render the main Header (e.g. Syro/MOSC routes with their own layout). **The homepage (`/`) must NOT be skipped** so the Admin menu appears when an admin lands on `/`.

**Implemented pattern in `src/app/layout.tsx`**:

- **`skipAuthForRoute`**
  - **Before (incorrect)**: `pathname === '/' || pathname.startsWith('/mosc-old') || pathname.startsWith('/mosc')` — caused the admin check to be skipped on the homepage, so `isTenantAdmin` stayed `false` and the Admin menu did not appear in production when users landed on `/`.
  - **After (correct)**: `pathname.startsWith('/mosc-old') || pathname.startsWith('/mosc')` — only skip routes where the main app Header is not rendered (ConditionalLayout renders children only for `/mosc-old` and `/mosc`).

- **Comment** (keep in layout):
  - "Run auth + profile lookup on all routes that show the main Header (including /) so Admin menu appears for admins (per clerk_auth rule)."
  - "Only skip /mosc-old and /mosc — ConditionalLayout does not render the main Header there."

**Result**:
- **`/`**: Admin check **runs** → profile lookup by `userId` + `tenantId` → `isTenantAdmin` from `user_profile.user_role === 'ADMIN'` → Header can show Admin menu for admins.
- **`/mosc-old`, `/mosc`**: Admin check **skipped** (no main Header; ConditionalLayout renders children only). No change.
- **All other routes** (e.g. `/events`, `/pricing`, `/admin`): Admin check already runs; no change.

No changes are required to middleware or ConditionalLayout for this behavior. After deploy, if the Admin menu still does not show for a user, confirm in **production** that there is a `user_profile` row for that Clerk user and the correct tenant (e.g. mosc-temp) with `user_role = 'ADMIN'` (via `/admin/manage-usage` → Edit user → Role = ADMIN, or direct DB update). The user may need to sign out and sign back in after a role change.

### **Key Requirements**:
1. ✅ **Await `headers()` first** - Required for Next.js 15+
2. ✅ **Call `auth()` after headers()** - Ensures proper async context
3. ✅ **Run admin check on ALL routes** - Use `if (pathname)` (or equivalent); do not gate by `pathname?.startsWith('/admin')` so the Admin menu shows on public pages too
4. ✅ **Do not skip auth on homepage** - `skipAuthForRoute` must NOT include `pathname === '/'`; only skip `/mosc-old` and `/mosc` where the main Header is not rendered
5. ✅ **Check `userId` exists** - Only fetch profile if user is authenticated
6. ✅ **Query by `userId + tenantId`** - Multi-tenant scoping
7. ✅ **Check `userRole === 'ADMIN'`** - Admin status check
8. ✅ **Pass `isTenantAdmin` to Header** - Client component receives server-verified flag

### **CRITICAL: Admin Role Must Be Set in Database**

**The `userRole` field in the `user_profile` table MUST be set to `'ADMIN'` for a user to have admin access.**

**How Admin Role is Determined**:
- ✅ **Database Field**: `user_profile.user_role` must equal `'ADMIN'` or `'SUPER_ADMIN'` (use `isAdminRole()` from `@/lib/utils`)
- ✅ **Tenant-Scoped**: The check uses `userId + tenantId` to find the correct record
- ✅ **Not from Clerk**: Admin role is NOT determined from Clerk metadata - it comes from the database
- ❌ **Not from userStatus**: `userStatus` (APPROVED, PENDING_APPROVAL, etc.) does NOT grant admin access

**Admin menu and `user_status`**:
- **Current behavior**: The Admin menu is shown based **only** on `user_role` (ADMIN/SUPER_ADMIN). `user_status` (e.g. `PENDING_APPROVAL`, `APPROVED`, `REJECTED`) is **not** used to hide the Admin menu. So a user with `user_role = 'ADMIN'` and `user_status = 'PENDING_APPROVAL'` will still see the Admin menu.
- **If you need to restrict the Admin menu to approved users only**: Require both `isAdminRole(profile.userRole)` and `profile.userStatus === 'APPROVED'` (or your chosen status) in the root layout when setting `isTenantAdmin`. Document that product decision in this rule and in the layout comment.

**How to Grant Admin Access**:

#### **Option 1: Using Admin Dashboard** (Recommended)
1. Log in as an existing admin user
2. Navigate to `/admin/manage-usage`
3. Find the user you want to make admin
4. Click "Edit" on the user row
5. In the "Role" dropdown, select `ADMIN`
6. Click "Save"
7. User must log out and log back in for changes to take effect

**Location**: [`src/app/admin/manage-usage/ManageUsageClient.tsx`](mdc:src/app/admin/manage-usage/ManageUsageClient.tsx) - Lines 199-209

#### **Option 2: Direct Database Update** (Manual)
```sql
-- Update user_role to ADMIN for a specific user and tenant
UPDATE user_profile
SET
  user_role = 'ADMIN',
  updated_at = NOW()
WHERE user_id = 'user_2vVLxhPnsIPGYf6qpfozk383Slr'  -- Replace with actual Clerk userId
  AND tenant_id = 'tenant_demo_002';  -- Replace with actual tenantId

-- Verify the update
SELECT id, user_id, email, user_role, user_status, tenant_id
FROM user_profile
WHERE user_id = 'user_2vVLxhPnsIPGYf6qpfozk383Slr'
  AND tenant_id = 'tenant_demo_002';
```

**Important Notes**:
- ⚠️ **Case-Sensitive**: The value must be exactly `'ADMIN'` (uppercase)
- ⚠️ **Tenant-Scoped**: Each tenant can have different admins - the same user can be ADMIN in one tenant and MEMBER in another
- ⚠️ **Requires Re-login**: After changing `user_role` in the database, the user must log out and log back in for the admin menu to appear
- ⚠️ **Multiple Records**: If a user has multiple records with the same email but different `tenant_id`, only the record matching `NEXT_PUBLIC_TENANT_ID` will be checked

**Available Role Values**:
- `'SUPER_ADMIN'` - Super administrator (highest level)
- `'ADMIN'` - Administrator (can access admin panel)
- `'ORGANIZER'` - Event organizer
- `'VOLUNTEER'` - Volunteer
- `'MEMBER'` - Regular member (default for new users)

**Reference**: [`src/app/admin/manage-usage/ManageUsageClient.tsx`](mdc:src/app/admin/manage-usage/ManageUsageClient.tsx) - Lines 201-208

---

## **Header Component Admin Menu Pattern**

### **Admin Menu Visibility: All Pages (Not Just /admin)**

The Admin menu in the Header must be shown whenever the user is an admin (**`isAdmin` true**), on **every page** that uses the main layout—including public pages (homepage, `/events`, `/gallery`, `/pricing`, etc.). The Header does **not** hide the Admin menu based on the current path; it only checks `isAdmin`. Ensuring the root layout runs the admin check on all routes (see above) guarantees `isTenantAdmin` is set correctly for every page, so the Admin menu appears consistently.

### **Client-Side Admin Check in `src/components/Header.tsx`**

**Pattern**:
```typescript
export default function Header({ isTenantAdmin }: HeaderProps) {
  const [isAdmin, setIsAdmin] = useState(!!isTenantAdmin);

  // Prefer server-verified tenant admin flag when provided
  useEffect(() => {
    if (typeof isTenantAdmin === 'boolean') {
      setIsAdmin(isTenantAdmin);
      return;
    }

    // Fallback to Clerk metadata if server flag not provided
    if (isLoaded && user) {
      const publicRole = user.publicMetadata?.role as string;
      const orgRole = user.organizationMemberships?.[0]?.role;
      const isAdminUser =
        publicRole === 'admin' ||
        publicRole === 'administrator' ||
        orgRole === 'admin' ||
        orgRole === 'org:admin';
      setIsAdmin(isAdminUser);
    } else {
      setIsAdmin(false);
    }
  }, [isLoaded, user, isTenantAdmin]);

  return (
    <nav>
      {/* ... */}
      {isAdmin && (
        <Link href="/admin">Admin Panel</Link>
      )}
    </nav>
  );
}
```

### **Priority Order**:
1. ✅ **Server-verified flag** (`isTenantAdmin` prop) - Most reliable, tenant-scoped
2. ✅ **Clerk metadata fallback** - Used if server flag not available
3. ❌ **Never rely solely on Clerk metadata** - Not tenant-aware

---

## **User Profile Avatar Dropdown Pattern**

### **Overview**
The Header component includes a user profile avatar dropdown that displays the logged-in user's profile image from social login (Google, Facebook, etc.) with a dropdown menu for Profile and Sign Out actions. This replaces the previous Clerk UserButton functionality while maintaining the same user experience.

### **Implementation in `src/components/Header.tsx`**

**Pattern**:
```typescript
/**
 * User Avatar Dropdown Component
 * Shows user's profile image with dropdown menu for Profile and Sign Out
 */
function UserAvatarDropdown({
  user,
  onSignOut,
  isSigningOut
}: {
  user: any;
  onSignOut: () => void;
  isSigningOut: boolean;
}) {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef<HTMLDivElement>(null);
  const pathname = usePathname();

  // Close dropdown when clicking outside
  useEffect(() => {
    const handleClickOutside = (event: MouseEvent) => {
      if (dropdownRef.current && !dropdownRef.current.contains(event.target as Node)) {
        setIsOpen(false);
      }
    };

    if (isOpen) {
      document.addEventListener('mousedown', handleClickOutside);
    }

    return () => {
      document.removeEventListener('mousedown', handleClickOutside);
    };
  }, [isOpen]);

  // Get user's profile image or use default avatar
  const userImageUrl = user?.imageUrl || user?.hasImage ? user?.imageUrl : null;
  const userName = user?.firstName || user?.fullName || user?.emailAddresses?.[0]?.emailAddress || 'User';
  const userEmail = user?.emailAddresses?.[0]?.emailAddress || '';

  return (
    <div className="relative group" ref={dropdownRef}>
      {/* Avatar Button */}
      <button
        onClick={() => setIsOpen(!isOpen)}
        className="relative flex items-center justify-center w-10 h-10 min-w-[40px] min-h-[40px] rounded-full border-2 border-transparent hover:border-blue-400 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 transition-all duration-300 ease-in-out hover:scale-105 active:scale-95 overflow-hidden bg-gray-100"
        aria-label="User menu"
        aria-expanded={isOpen}
        aria-haspopup="true"
      >
        {userImageUrl ? (
          <Image
            src={userImageUrl}
            alt={userName}
            width={40}
            height={40}
            className="w-full h-full object-cover rounded-full"
            unoptimized
          />
        ) : (
          <div className="w-full h-full flex items-center justify-center bg-blue-400 text-white">
            <User size={20} className="text-white" />
          </div>
        )}
      </button>

      {/* Dropdown Menu */}
      {isOpen && (
        <div className="absolute top-full right-0 mt-2 w-64 bg-white rounded-xl shadow-xl border border-gray-100 z-50">
          <div className="py-3">
            {/* User Info Section */}
            <div className="px-4 py-3 border-b border-gray-100">
              <div className="flex items-center space-x-3">
                {userImageUrl ? (
                  <Image src={userImageUrl} alt={userName} width={40} height={40} className="w-10 h-10 rounded-full object-cover" unoptimized />
                ) : (
                  <div className="w-10 h-10 flex items-center justify-center bg-blue-400 text-white rounded-full">
                    <User size={20} className="text-white" />
                  </div>
                )}
                <div className="flex-1 min-w-0">
                  <p className="text-sm font-semibold text-gray-900 truncate">{userName}</p>
                  {userEmail && <p className="text-xs text-gray-500 truncate">{userEmail}</p>}
                </div>
              </div>
            </div>

            {/* Menu Items */}
            <div className="py-2">
              <Link href="/profile" onClick={() => setIsOpen(false)} className="flex items-center space-x-3 px-4 py-2 mx-1 rounded-lg text-sm font-medium tracking-[0.025em] focus:outline-none transition-all duration-300 ease-in-out text-blue-400 hover:text-blue-500 hover:font-semibold hover:bg-blue-50">
                <User size={16} aria-hidden="true" />
                <span>Profile</span>
              </Link>
              <button onClick={() => { setIsOpen(false); onSignOut(); }} disabled={isSigningOut} className="w-full flex items-center space-x-3 px-4 py-2 mx-1 rounded-lg text-sm font-medium tracking-[0.025em] focus:outline-none transition-all duration-300 ease-in-out text-red-400 hover:text-red-500 hover:font-semibold hover:bg-red-50">
                <LogOut size={16} aria-hidden="true" />
                <span>{isSigningOut ? 'Signing Out...' : 'Sign Out'}</span>
              </button>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}
```

### **Key Features**

1. **Profile Image Display**:
   - Uses `user.imageUrl` from Clerk's `useUser()` hook
   - Shows social login profile picture (Google, Facebook, etc.)
   - Falls back to User icon if no image available
   - Circular avatar (40x40px) with hover effects

2. **Dropdown Menu**:
   - User info section with avatar, name, and email
   - Profile link (navigates to `/profile`)
   - Sign Out button (calls `handleSignOut` function)
   - Click-outside to close functionality
   - Proper ARIA labels for accessibility

3. **Positioning**:
   - **Desktop**: Rightmost position in header (after Admin menu if admin)
   - **Mobile**: User profile section at top of mobile menu with larger avatar (48x48px)

4. **Styling**:
   - Matches admin action button pattern
   - Hover effects (scale, border color change)
   - Focus states for keyboard navigation
   - Active state for Profile link when on `/profile` page

### **Integration in Header Component**

**Desktop Header** (when user is logged in):
```typescript
{/* Auth and Admin Menu Items */}
<div className="flex items-center space-x-1">
  {!userId ? (
    // Sign In / Sign Up buttons
  ) : (
    <>
      {/* Admin Menu with Submenu */}
      {isAdmin && (
        <div className="relative group">
          {/* Admin menu dropdown */}
        </div>
      )}

      {/* User Profile Avatar Dropdown - Rightmost */}
      <UserAvatarDropdown
        user={user}
        onSignOut={handleSignOut}
        isSigningOut={isSigningOut}
      />
    </>
  )}
</div>
```

**Mobile Menu**:
```typescript
{/* Mobile User Profile Section */}
<div className="px-6 mb-4 pb-4 border-b border-gray-200">
  <div className="flex items-center space-x-3">
    {user?.imageUrl ? (
      <Image src={user.imageUrl} alt={userName} width={48} height={48} className="w-12 h-12 rounded-full object-cover border-2 border-blue-400" unoptimized />
    ) : (
      <div className="w-12 h-12 flex items-center justify-center bg-blue-400 text-white rounded-full">
        <User size={24} className="text-white" />
      </div>
    )}
    <div className="flex-1 min-w-0">
      <p className="text-sm font-semibold text-gray-900 truncate">{userName}</p>
      {user?.emailAddresses?.[0]?.emailAddress && (
        <p className="text-xs text-gray-500 truncate">{user.emailAddresses[0].emailAddress}</p>
      )}
    </div>
  </div>
</div>

{/* Mobile Profile Link */}
<Link href="/profile" onClick={closeMobileMenu}>
  <User size={18} />
  <span>Profile</span>
</Link>

{/* Mobile Sign Out Button */}
<button onClick={() => { closeMobileMenu(); handleSignOut(); }}>
  <LogOut size={18} />
  <span>Sign Out</span>
</button>
```

### **Required Imports**

```typescript
import React, { useState, useEffect, useRef } from 'react';
import Link from 'next/link';
import { usePathname } from 'next/navigation';
import { User, LogOut } from 'lucide-react';
import { useUser } from '@clerk/nextjs';
import Image from 'next/image';
```

### **Key Requirements**

1. ✅ **Use Clerk's `useUser()` hook** - Provides `user.imageUrl`, `user.firstName`, `user.emailAddresses`
2. ✅ **Handle missing image** - Show User icon fallback if `user.imageUrl` is null
3. ✅ **Click-outside to close** - Use `useRef` and `useEffect` to detect clicks outside dropdown
4. ✅ **Accessibility** - Include `aria-label`, `aria-expanded`, `aria-haspopup` attributes
5. ✅ **Position rightmost** - Place avatar dropdown after Admin menu (if admin) in desktop header
6. ✅ **Mobile support** - Show user profile section at top of mobile menu with larger avatar

### **Order in Header**

**Desktop Header Order** (when logged in):
1. Navigation menu items (Home, About, Events, etc.)
2. Admin Menu (if user is admin) - **Left of avatar**
3. User Profile Avatar Dropdown - **Rightmost**

**Mobile Menu Order** (when logged in):
1. User Profile Section (avatar, name, email) - **Top**
2. Profile Link
3. Sign Out Button
4. Admin Menu (if user is admin)

### **Best Practices**

**DO:**
- ✅ Use `user.imageUrl` from Clerk's `useUser()` hook for profile images
- ✅ Provide fallback User icon if no image available
- ✅ Use `useRef` for dropdown container to detect click-outside
- ✅ Include proper ARIA labels for accessibility
- ✅ Position avatar rightmost in desktop header (after Admin menu)
- ✅ Show user profile section at top of mobile menu
- ✅ Use Next.js `Image` component with `unoptimized` prop for external images

**DON'T:**
- ❌ Don't hardcode avatar URLs - always use `user.imageUrl`
- ❌ Don't skip fallback icon - always show User icon if no image
- ❌ Don't forget click-outside handler - dropdown should close when clicking outside
- ❌ Don't skip ARIA labels - important for screen readers
- ❌ Don't place avatar before Admin menu - should be rightmost

### **Reference Implementation**

- **Header Component**: [`src/components/Header.tsx`](mdc:src/components/Header.tsx) - Lines 256-420 (UserAvatarDropdown component)
- **Desktop Integration**: [`src/components/Header.tsx`](mdc:src/components/Header.tsx) - Lines 872-995 (desktop header with Admin menu and avatar)
- **Mobile Integration**: [`src/components/Header.tsx`](mdc:src/components/Header.tsx) - Lines 1223-1280 (mobile menu with user profile section)

---

## **Satellite vs Primary: Sign-In and Sign-Out (Multi-Domain)**

When using Clerk with a **primary domain** (e.g. `event-site-manager.com`) and **satellite domain(s)** (e.g. `mosc-temp.com`), sign-in and sign-out must detect primary vs satellite correctly. If the primary app is misidentified as a satellite (e.g. due to `NEXT_PUBLIC_CLERK_DOMAIN` set to the primary host), the sign-in page can render blank and sign-out can fail or redirect incorrectly.

### **Core Rule: Primary-First Detection**

- **Always detect primary domain first** using `NEXT_PUBLIC_PRIMARY_DOMAIN` and `window.location.hostname`.
- **Never use `NEXT_PUBLIC_CLERK_DOMAIN`** to decide "am I satellite?" on the primary app—that env can be mis-set and cause the primary to redirect to itself or hide the Clerk UI.
- **Only treat as satellite** when the hostname is a known satellite (e.g. `hostname.includes('mosc-temp.com')`).

### **Sign-In Page** (`src/app/(auth)/sign-in/[[...sign-in]]/page.tsx`)

**Pattern**:
1. Localhost → show Clerk `<SignIn />` (development).
2. **If hostname is the primary domain** (using `NEXT_PUBLIC_PRIMARY_DOMAIN`, normalized with/without `www.`) → **do not redirect**; always render Clerk `<SignIn />` and honor `redirect_url` for return to satellite after sign-in.
3. **Only if hostname is satellite** (e.g. `mosc-temp.com`) → set redirect state and send user to primary’s `/sign-in?redirect_url=<satellite-origin>`.

**Primary detection (use this shape)**:
```typescript
const primaryDomain = process.env.NEXT_PUBLIC_PRIMARY_DOMAIN || 'www.event-site-manager.com';
const primaryHost = primaryDomain.replace(/^https?:\/\//, '').replace(/\/$/, '');
const isPrimary =
  hostname === primaryHost ||
  hostname === primaryDomain ||
  hostname.includes(primaryHost.replace('www.', '')) ||
  hostname.includes(primaryDomain.replace('www.', ''));
if (isPrimary) return; // Do not set shouldRedirect; render <SignIn />
```

**Satellite redirect (only for known satellite)**:
```typescript
const isSatellite = hostname.includes('mosc-temp.com');
if (isSatellite) {
  setShouldRedirect(true);
  window.location.href = `https://${primaryHost}/sign-in?redirect_url=${encodeURIComponent(window.location.origin)}`;
}
```

**Reference**: [`src/app/(auth)/sign-in/[[...sign-in]]/page.tsx`](mdc:src/app/(auth)/sign-in/[[...sign-in]]/page.tsx)

### **Sign-Out in Header** (`src/components/Header.tsx` — `handleSignOut`)

**Pattern**:
1. **If hostname is the primary domain** (same primary detection as above) → call Clerk `signOut()` and redirect to `/`. Do **not** redirect to `/auth/signout-redirect`.
2. **Only if hostname is satellite** (e.g. `mosc-temp.com`) → redirect to primary’s `/auth/signout-redirect?redirect_url=<satellite-origin>`.
3. Fallback (e.g. localhost): call `signOut()` and redirect to `/`.

**Reference**: [`src/components/Header.tsx`](mdc:src/components/Header.tsx) — `handleSignOut`

### **Sign-Out Redirect Page** (`src/app/auth/signout-redirect/page.tsx`)

**Purpose**: On the **primary** domain, this page is the target when users sign out from a **satellite**. The satellite redirects here; this page calls Clerk `signOut()` then redirects the user back to the satellite with `?clerk_signout=true` so the satellite can clear local state.

**Requirements**:
- **Primary app must serve this route** (required for satellite sign-out to work). The file can exist in both repos and be deployed to both; it is only used when the primary serves the request.
- Read `redirect_url` from query; allow only known satellites (e.g. `mosc-temp.com`) or `localhost`.
- Call `signOut()` from `useClerk()`, then `window.location.href = redirect_url + '?clerk_signout=true'` (or `&` if URL already has query).
- If `redirect_url` is invalid or missing, redirect to `/`.

**Reference**: [`src/app/auth/signout-redirect/page.tsx`](mdc:src/app/auth/signout-redirect/page.tsx)

### **Deployment (Same Repo vs Two Repos)**

- **Same repo for both domains**: Add/update the sign-in page, Header, and signout-redirect page once; deploy to both Amplify apps. Both get the same code; behavior is determined by hostname and env (primary vs satellite).
- **Two repos**: Apply the same sign-in and Header logic in both. Add the signout-redirect page at least to the **primary** repo; having it in both repos is fine and keeps codebases in sync.

### **CRITICAL: Primary Domain Must Stay Deployed for Satellite Auth to Work**

The **primary** (e.g. https://www.event-site-manager.com/) must remain deployed. If you remove or undeploy the primary app (e.g. from AWS Amplify), **sign-in and sign-out on the satellite (mosc-temp.com) will break**. This is by design, not a bug.

**Why:**

1. **Sign-in (satellite)**  
   - User on mosc-temp.com goes to `/sign-in`.  
   - The sign-in page detects satellite and **redirects** to the primary:  
     `https://www.event-site-manager.com/sign-in?redirect_url=https://mosc-temp.com`  
   - The **primary** serves the Clerk `<SignIn />` page. After successful sign-in, Clerk redirects the user back to `redirect_url` (the satellite).  
   - If the primary app is gone, that URL returns 404 or an error → sign-in never completes.

2. **Sign-out (satellite)**  
   - User on mosc-temp.com clicks Sign out.  
   - The Header detects satellite and **redirects** to the primary:  
     `https://www.event-site-manager.com/auth/signout-redirect?redirect_url=https://mosc-temp.com`  
   - The **primary’s** `/auth/signout-redirect` page runs: it calls Clerk `signOut()` then redirects the user back to the satellite with `?clerk_signout=true` so the satellite can clear local state.  
   - If the primary app is gone, that URL returns 404 → sign-out never runs and the satellite cannot clear the session properly.

So the authentication mechanism is **not** different on the two domains: the **satellite depends on the primary** to host the actual sign-in UI and the sign-out redirect handler. Same Clerk credentials on both sides only mean they share the same Clerk application; the **primary app must be running** for satellite redirects to succeed.

**Summary:** Do not delete or undeploy the primary (event-site-manager.com). Keep both Amplify apps (primary + satellite) deployed. If you need only one public-facing site, you can point users to the satellite and keep the primary as a “hidden” auth backend, but the primary deployment must stay live.

### **Anti-Pattern: Using NEXT_PUBLIC_CLERK_DOMAIN for "Am I Satellite?"**

```typescript
// ❌ DON'T: Primary can be misidentified as satellite if env is wrong
const satelliteDomain = process.env.NEXT_PUBLIC_CLERK_DOMAIN || 'mosc-temp.com';
if (hostname.includes(satelliteDomain.replace('www.', ''))) {
  setShouldRedirect(true);  // On primary, if CLERK_DOMAIN=event-site-manager.com, this runs!
}
```

**Fix**: Detect primary first via `NEXT_PUBLIC_PRIMARY_DOMAIN`; only then treat as satellite when `hostname.includes('mosc-temp.com')` (or another explicit satellite host).

### **Satellite: “Email Already Taken” and User in Database but Not in Clerk Dashboard**

**Scenario**: Same email exists in your `user_profile` table and you can log in on the **primary** (e.g. event-site-manager.com) with that email, but (1) on the **satellite** (mosc-temp.com) when you try to **register** you see “email already taken”, and (2) that user does not appear in Clerk Dashboard’s user list.

**Is it because the same code runs on both domains?**  
**No.** Running the same Next.js app on both the primary (event-site-manager.com) and the satellite (mosc-temp.com) is the intended Clerk satellite setup. It does not conflict with Clerk.

**Why “email already taken” on the satellite when registering?**

- On the satellite, “Sign up” redirects to the **primary** `/sign-up` (Clerk `<SignUp />`).
- **Clerk** allows only one user per email in a single Clerk application. Both domains use the **same** Clerk application (same API keys).
- If that email **already has an account** in Clerk (e.g. you signed in or signed up earlier on the primary), Clerk will reject a new sign-up with “email already taken”.
- So the message is from **Clerk**, not from your database. The database row exists because the app created/updated `user_profile` when you signed in on the primary (layout bootstrap / ProfileBootstrapper).

**What to do:** For that email, use **Sign in** on the satellite (which redirects to primary sign-in), not Sign up. After signing in on the primary, Clerk redirects back to the satellite and the same user is used.

**Why does the user not show in Clerk Dashboard?**

- If you can **log in** on the primary with that email, Clerk **does** have that user. Login cannot succeed without a Clerk user.
- Typical causes for not seeing them in the Dashboard: (1) viewing a **different** Clerk application (e.g. Development vs Production), (2) viewing a different **environment** (e.g. Production vs Development), or (3) filtering/search in the Dashboard.
- Check the **same** Clerk application and **Production** environment that event-site-manager.com uses; the user should appear there.

**Summary**

- Same code on primary and satellite is **correct**; it does not break Clerk.
- “Email already taken” on satellite sign-up = Clerk rejecting duplicate sign-up; use **Sign in** instead.
- User in DB but “not in Clerk” while able to log in on primary = user **is** in Clerk; check the correct Clerk app and Production instance in the Dashboard.

---

## **Clerk Satellite Sync: First-Load Fix (`?__clerk_synced=true`)**

After redirect from primary to satellite, Clerk appends `?__clerk_synced=true`. On this **first request**, the session cookie has not yet been established — `auth()` on the server would either hang or return null. The client-side ClerkProvider also takes time to process the sync param. This causes a blank/broken page on the first load.

### **Three-Layer Fix (implemented March 2026)**

#### **Layer 1: Middleware — Skip Server Auth During Sync** (`src/middleware.ts`)
When `__clerk_synced` is in the URL, middleware sets an `x-clerk-syncing: true` request header. The root layout reads this header and **skips the entire auth/profile lookup block** (auth(), user profile fetch, admin check) for that request — the session will be available on the *next* request after cookies are set.

```typescript
// In middleware.ts — inside clerkMiddleware callback
if (req.nextUrl.searchParams.has('__clerk_synced')) {
  requestHeaders.set('x-clerk-syncing', 'true');
}

// In layout.tsx — skip expensive auth work during sync
const isClerkSyncing = headersList.get('x-clerk-syncing') === 'true';
if (pathname && !skipAuthForRoute && !isClerkSyncing) {
  // existing auth + profile logic
}
```

**Why**: `auth()` during sync makes a network call to Clerk API (~300–800ms on Lambda cold start) that returns null anyway. Skipping it saves server latency and prevents blank/error states.

#### **Layer 2: Loading Overlay — ClerkSatelliteSyncGate** (`src/components/ClerkSatelliteSyncGate.tsx`)
A client component that detects `__clerk_synced` in the URL on mount and shows a full-screen loading overlay (`position: fixed; z-index: 9998`). Once `useAuth().isLoaded` flips to `true`, it waits a 400ms stabilization period then fades out (500ms CSS transition). A 10s max timeout prevents infinite loading.

- **Placement**: Inside `ClerkProvider` in `src/app/layout.tsx`, alongside `ClerkSyncUrlCleanup`.
- **z-index**: 9998 (below navigation-loading-indicator at 9999).

#### **Layer 3: URL Cleanup — ClerkSyncUrlCleanup** (`src/components/ClerkSyncUrlCleanup.tsx`)
Client component that strips `__clerk_*` query params via `history.replaceState` — **only after** `useAuth().isLoaded === true`. Three-tier cleanup with production-tuned timing:

| Tier | Delay after `isLoaded` | Purpose |
|------|----------------------|---------|
| Primary cleanup | 1000ms | Main param removal — gives Clerk time to persist sync state cookies on Lambda cold starts |
| Safety net | 5000ms | Catches re-appended params on slow Clerk SDK init |
| Periodic watchdog | Every 2500ms for 15s (6 checks) | Final defense against params Clerk re-appends during late sync/retry flows |

- **CRITICAL**: Must wait for `isLoaded === true` before stripping `__clerk_synced`. Removing it early causes Clerk to think sync hasn't happened → infinite redirect loop.
- **sessionStorage guard**: `clerk_satellite_synced` key prevents edge-case re-trigger within the same tab session.
- **Placement**: Must render **inside** `ClerkProvider` (so `useAuth()` works). Do not move outside or reorder relative to other layout children.

### **Reference**
- **Middleware**: [`src/middleware.ts`](mdc:src/middleware.ts) — `x-clerk-syncing` header injection
- **Layout**: [`src/app/layout.tsx`](mdc:src/app/layout.tsx) — `isClerkSyncing` check, `<ClerkSatelliteSyncGate />` and `<ClerkSyncUrlCleanup />` in body
- **Sync Gate**: [`src/components/ClerkSatelliteSyncGate.tsx`](mdc:src/components/ClerkSatelliteSyncGate.tsx) — loading overlay during sync
- **URL Cleanup**: [`src/components/ClerkSyncUrlCleanup.tsx`](mdc:src/components/ClerkSyncUrlCleanup.tsx) — three-tier param removal

---

## **Common Anti-Patterns**

### **❌ DON'T: Put Routes That Call auth() in ignoredRoutes**
```typescript
// ❌ WRONG: Homepage calls auth() but is ignored
ignoredRoutes: [
  '/',  // This breaks auth() calls in layout.tsx
],
```

**Problem**: Clerk middleware doesn't run, so `auth()` throws error: "Clerk can't detect usage of authMiddleware()"

**Fix**: Remove from `ignoredRoutes`, keep in `publicRoutes`

---

### **❌ DON'T: Call auth() Before Awaiting headers()**
```typescript
// ❌ WRONG: Calling auth() before headers() is awaited
export default async function Layout() {
  const authResult = await auth();  // Error: headers() not awaited
  const headersList = await headers();
}
```

**Problem**: Next.js 15+ requires `headers()` to be awaited before any function that uses it

**Fix**: Always await `headers()` first, then call `auth()`

---

### **❌ DON'T: Skip Tenant ID in Admin Lookup**
```typescript
// ❌ WRONG: Not scoping by tenantId
const url = `/api/proxy/user-profiles?userId.equals=${userId}`;
```

**Problem**: Multi-tenant systems need tenant scoping

**Fix**: Always include `tenantId.equals` in query

---

### **❌ DON'T: Check userStatus Instead of userRole**
```typescript
// ❌ WRONG: Checking status instead of role
isTenantAdmin = profile?.userStatus === 'APPROVED';
```

**Problem**: `userStatus` indicates approval state, not admin role

**Fix**: Check `userRole === 'ADMIN'`

---

### **❌ DON'T: Run Admin Check Only on /admin Routes**
```typescript
// ❌ WRONG: Admin menu won't show on public pages
if (pathname?.startsWith('/admin')) {
  const authResult = await auth();
  // ... set isTenantAdmin
}
```

**Problem**: On public pages (homepage, `/events`, `/gallery`, etc.), `isTenantAdmin` stays false, so the Admin menu does not appear in the Header until the user navigates to `/admin`.

**Fix**: Run the admin check whenever `pathname` is present (all routes that use the main layout). See "Admin Check on ALL Routes" above.

---

## **Complete Example: Homepage with Admin Lookup**

### **1. Middleware Configuration** (`src/middleware.ts`)
```typescript
export default authMiddleware({
  publicRoutes: [
    '/',  // ✅ Homepage - accessible without auth, but middleware runs
    '/polls(.*)',  // ✅ Polls - need auth() for user participation check
    '/pricing(.*)',  // ✅ Pricing - need auth() for subscription check
  ],

  ignoredRoutes: [
    '/events(.*)',  // ✅ Static pages, no auth() calls
    '/gallery(.*)',  // ✅ Static pages, no auth() calls
    '/api/proxy/(.*)',  // ✅ API routes handle own auth
    // ❌ DON'T: '/',  // Would break auth() in layout.tsx
  ],
});
```

### **2. Root Layout** (`src/app/layout.tsx`)
```typescript
export default async function RootLayout({ children }: { children: React.ReactNode }) {
  // Step 1: Await headers() first (Next.js 15+ requirement)
  const headersList = await headers();
  const hostname = headersList.get('host') || '';

  // Step 2: Setup Clerk config
  const clerkProps = { /* ... */ };

  // Step 3: Check admin status (requires Clerk middleware to run)
  let isTenantAdmin = false;
  try {
    const authResult = await auth();  // ✅ Works because middleware runs
    const userId = authResult?.userId || null;

    if (userId) {
      const tenantId = getTenantId();
      const url = `${baseUrl}/api/proxy/user-profiles?userId.equals=${userId}&tenantId.equals=${tenantId}&size=1`;
      const resp = await fetch(url, { cache: 'no-store' });

      if (resp.ok) {
        const data = await resp.json();
        const profile = Array.isArray(data) ? data[0] : data;
        isTenantAdmin = profile?.userRole === 'ADMIN';  // ✅ Check role, not status
      }
    }
  } catch (error) {
    console.error('[Layout] Error determining admin status:', error);
    isTenantAdmin = false;
  }

  return (
    <ClerkProvider {...clerkProps}>
      <Header isTenantAdmin={isTenantAdmin} />
      {children}
    </ClerkProvider>
  );
}
```

### **3. Header Component** (`src/components/Header.tsx`)
```typescript
export default function Header({ isTenantAdmin }: HeaderProps) {
  const [isAdmin, setIsAdmin] = useState(!!isTenantAdmin);

  useEffect(() => {
    // ✅ Prefer server-verified flag
    if (typeof isTenantAdmin === 'boolean') {
      setIsAdmin(isTenantAdmin);
      return;
    }
    // Fallback to Clerk metadata...
  }, [isTenantAdmin]);

  return (
    <nav>
      {isAdmin && <Link href="/admin">Admin Panel</Link>}
    </nav>
  );
}
```

---

## **Decision Tree: Where Should This Route Go?**

```
Does the route call auth() or currentUser() in server components?
│
├─ YES → publicRoutes (NOT ignoredRoutes)
│   │
│   └─ Does it require login?
│       ├─ YES → protectedRoutes (not in publicRoutes)
│       └─ NO → publicRoutes ✅
│
└─ NO → ignoredRoutes ✅
    │
    └─ Is it an API route?
        ├─ YES → ignoredRoutes ✅
        └─ NO → ignoredRoutes ✅ (static pages)
```

---

## **Best Practices**

### **DO:**
- ✅ Always await `headers()` before calling `auth()` or `currentUser()`
- ✅ Keep routes that call `auth()` in `publicRoutes`, not `ignoredRoutes`
- ✅ **Run admin check on ALL routes** (whenever `pathname` is present) so the Admin menu shows on public pages (homepage, events, gallery, etc.), not only on `/admin`
- ✅ Use server-verified `isTenantAdmin` flag in Header component
- ✅ Query user profiles with both `userId` and `tenantId` for multi-tenant scoping
- ✅ Check `userRole === 'ADMIN'` for admin status (not `userStatus`)
- ✅ Fail closed (no admin) on errors for security

### **DON'T:**
- ❌ Put routes that call `auth()` in `ignoredRoutes`
- ❌ Call `auth()` before awaiting `headers()` in Next.js 15+
- ❌ **Gate the admin check by pathname** (e.g. `pathname?.startsWith('/admin')`)—this hides the Admin menu on public pages
- ❌ Skip tenant ID in admin lookup queries
- ❌ Check `userStatus` instead of `userRole` for admin
- ❌ Rely solely on Clerk metadata for admin status (not tenant-aware)

---

## **Troubleshooting**

### **Error: "Clerk can't detect usage of authMiddleware()"**
**Cause**: Route is in `ignoredRoutes` but calls `auth()`

**Fix**: Remove route from `ignoredRoutes`, keep in `publicRoutes`

---

### **Error: "headers() should be awaited before using its value"**
**Cause**: Calling `auth()` before `headers()` is awaited

**Fix**: Always await `headers()` first, then call `auth()`

---

### **Admin Menu Not Appearing**
**Checklist**:
1. ✅ Is route in `publicRoutes` (not `ignoredRoutes`)?
2. ✅ Is `auth()` being called after `headers()` is awaited?
3. ✅ **Is the homepage (`/`) NOT skipped?** — In `src/app/layout.tsx`, `skipAuthForRoute` must NOT include `pathname === '/'`; it should only skip `pathname.startsWith('/mosc-old')` and `pathname.startsWith('/mosc')`. If `/` is skipped, the admin check never runs when users land on the homepage and the Admin menu will not appear in production.
4. ✅ Is `userId` not null (user is logged in)?
5. ✅ Is `tenantId` matching the database record?
6. ✅ **Is `user_role = 'ADMIN'` or `'SUPER_ADMIN'` in the `user_profile` table?** (CRITICAL - must be set manually; confirm in **production** DB for production domain, e.g. mosc-temp.com). Use `isAdminRole()` from `@/lib/utils` to check — it accepts both `'ADMIN'` and `'SUPER_ADMIN'`.
7. ✅ Is `isTenantAdmin` prop being passed to Header?
8. ✅ Has the user logged out and logged back in after role change?
9. ✅ **Does the Clerk `userId` match the `user_id` in the `user_profile` table?** (CRITICAL — Clerk can assign a **new userId** after re-login if the Clerk application was recreated, the Clerk environment changed (dev→prod), or the user was deleted and re-provisioned in Clerk Dashboard. The old `user_id` in the database will no longer match. Check browser console for `[Header] Checking admin status ... userId=XXX` and compare against the DB. If mismatched, update the DB: `UPDATE user_profile SET user_id = '<new_clerk_userId>', clerk_user_id = '<new_clerk_userId>' WHERE email = '...' AND tenant_id = '...';`)

**Common Causes**:
- `user_role` is not set to `'ADMIN'` or `'SUPER_ADMIN'` in the database. This must be changed manually:
  - **Via Dashboard**: `/admin/manage-usage` → Edit user → Change Role to `ADMIN` → Save
  - **Via Database**: `UPDATE user_profile SET user_role = 'ADMIN' WHERE user_id = '...' AND tenant_id = '...'`
- **Clerk userId mismatch**: The `user_id` in the `user_profile` table does not match the Clerk userId. This happens when Clerk issues a new userId (e.g., after Clerk app recreation, environment change, or user re-provisioning). The layout.tsx has email-based fallback logic to auto-update the userId, but it is skipped during satellite sync (`__clerk_synced`). Fix by updating the DB manually:
  - `UPDATE user_profile SET user_id = '<new_clerk_userId>', clerk_user_id = '<new_clerk_userId>' WHERE email = '<user_email>' AND tenant_id = '<tenant_id>';`

---

### **Sign-In Page Blank on Primary (event-site-manager.com)**
**Cause**: Primary domain was treated as satellite (e.g. `NEXT_PUBLIC_CLERK_DOMAIN` set to primary host), so the page redirected to itself or showed "Redirecting..." instead of Clerk `<SignIn />`.

**Fix**: Use primary-first detection in sign-in page; only redirect when `hostname.includes('mosc-temp.com')`. See "Satellite vs Primary: Sign-In and Sign-Out" above.

---

### **Sign-Out Error on Satellite (mosc-temp.com)**
**Cause**: Primary app does not serve `/auth/signout-redirect`, or Header on primary was redirecting to signout-redirect instead of calling `signOut()` (primary misidentified as satellite).

**Fix**: (1) Add `src/app/auth/signout-redirect/page.tsx` to the primary app (or both). (2) In Header `handleSignOut`, use primary-first detection; only redirect to primary’s signout-redirect when `hostname.includes('mosc-temp.com')`. See "Satellite vs Primary: Sign-In and Sign-Out" above.

---

## **References**

- **Middleware Config**: [`src/middleware.ts`](mdc:src/middleware.ts) - Lines 29-99
- **Root Layout**: [`src/app/layout.tsx`](mdc:src/app/layout.tsx) - Lines 57-212
- **Header Component**: [`src/components/Header.tsx`](mdc:src/components/Header.tsx) - Lines 256-420 (UserAvatarDropdown), Lines 872-995 (desktop integration), Lines 1223-1280 (mobile integration), `handleSignOut` (satellite/primary sign-out)
- **Sign-In Page**: [`src/app/(auth)/sign-in/[[...sign-in]]/page.tsx`](mdc:src/app/(auth)/sign-in/[[...sign-in]]/page.tsx) - Satellite/primary sign-in (primary-first detection)
- **Sign-Out Redirect Page**: [`src/app/auth/signout-redirect/page.tsx`](mdc:src/app/auth/signout-redirect/page.tsx) - Primary sign-out redirect for satellite users
- **Admin Layout**: [`src/app/admin/layout.tsx`](mdc:src/app/admin/layout.tsx) - Admin route protection pattern
- **Next.js 15+ Headers API**: https://nextjs.org/docs/messages/sync-dynamic-apis
- **Clerk Middleware Docs**: https://clerk.com/docs/references/nextjs/auth-middleware
- **Clerk useUser Hook**: https://clerk.com/docs/references/nextjs/use-user

---

## **Summary**

**Key Rule**: Routes that call `auth()` or `currentUser()` **MUST** have Clerk middleware running. This means:
- ✅ Keep them in `publicRoutes` (if accessible without login)
- ❌ **NEVER** put them in `ignoredRoutes` (bypasses middleware)

**Admin Lookup Pattern**:
1. Await `headers()` first
2. Call `auth()` to get `userId`
3. **Run admin check on ALL routes** (whenever `pathname` is present)—do not limit to `/admin` so the Admin menu shows on public pages (homepage, events, gallery, etc.)
4. **Do not skip auth on homepage** — In layout, `skipAuthForRoute` must NOT include `pathname === '/'`; only skip `pathname.startsWith('/mosc-old')` and `pathname.startsWith('/mosc')` (routes where ConditionalLayout does not render the main Header).
5. Query user profile with `userId + tenantId`
6. Check admin role using `isAdminRole()` from `@/lib/utils` — accepts both `'ADMIN'` and `'SUPER_ADMIN'`
7. Pass `isTenantAdmin` to Header component
8. **Verify Clerk userId matches DB `user_id`** — Clerk can assign new userIds after app recreation or environment changes. If mismatched, update DB or rely on email-based fallback in layout.tsx

**User Profile Avatar Pattern**:
1. Use Clerk's `useUser()` hook to get `user.imageUrl`, `user.firstName`, `user.emailAddresses`
2. Display circular avatar (40x40px desktop, 48x48px mobile) with profile image from social login
3. Show dropdown menu with user info, Profile link, and Sign Out button
4. Position avatar rightmost in desktop header (after Admin menu if admin)
5. Show user profile section at top of mobile menu
6. Handle click-outside to close dropdown and provide fallback User icon if no image

**CRITICAL**: The `user_role` field in the `user_profile` table must be set to `'ADMIN'` for admin access. This can be done:
- Via Admin Dashboard: `/admin/manage-usage` → Edit user → Change Role to `ADMIN`
- Via Direct Database: `UPDATE user_profile SET user_role = 'ADMIN' WHERE ...`

**Satellite vs primary (sign-in and sign-out)**:
- **Primary-first detection**: Always detect primary domain via `NEXT_PUBLIC_PRIMARY_DOMAIN` and hostname before any redirect. Never use `NEXT_PUBLIC_CLERK_DOMAIN` to decide "am I satellite?" on the primary.
- **Sign-in**: Primary always renders Clerk `<SignIn />`; only satellite (e.g. `mosc-temp.com`) redirects to primary’s `/sign-in?redirect_url=...`.
- **Sign-out**: Primary always calls `signOut()` and redirects to `/`; only satellite redirects to primary’s `/auth/signout-redirect?redirect_url=...`. Primary must serve `/auth/signout-redirect` (calls `signOut()` then redirects back to satellite with `clerk_signout=true`).

This ensures:
- Admin menu appears correctly after user login while maintaining proper authentication flow
- **Admin menu appears on public pages** (homepage, events, gallery, pricing, etc.) when a logged-in user is an admin—not only on `/admin` and sub-pages
- User profile avatar displays social login profile picture with dropdown menu for Profile and Sign Out
- **Sign-in and sign-out work on both primary and satellite** without blank pages or redirect errors
- Consistent user experience across desktop and mobile with proper accessibility support

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
