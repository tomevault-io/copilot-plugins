## user-profile-operations

> Handles cases where existing user profiles (especially those created via mobile payments) need to be updated with current Clerk user data during sign-in and profile page load.


---
description: Comprehensive rules for user profile operations including fetch, creation, and update patterns
globs: src/app/profile/**/*.ts, src/pages/api/proxy/user-profiles/**/*.ts
alwaysApply: true
---

### **User Profile Fetch Operations (4-Step Fallback)**

- **Step 1: Primary Lookup by User ID**
  - Always attempt to fetch profile using `/api/proxy/user-profiles/by-user/{userId}`
  - Use Clerk `currentUser()` to get authenticated user context
  - Return profile immediately if found

- **Step 2: Email-Based Fallback Lookup with Reconciliation**
  - If Step 1 fails (404), extract email from Clerk user object
  - Query using `/api/proxy/user-profiles?email.equals={email}`
  - **NEW: Profile Reconciliation Logic**
    - If profile found by email but `userId` differs from Clerk user ID
    - OR if `firstName`/`lastName` are empty/placeholder values
    - Update profile with current Clerk user data
    - This handles mobile payment profiles and incomplete profiles
  - Validate profile exists and has valid ID before returning
  - Log reconciliation attempts for debugging

- **Step 3: Automatic Profile Creation**
  - If no profile exists, create automatically using Clerk user data
  - Use placeholder values for required fields (no null values)
  - Include all required fields: `userId`, `email`, `firstName`, `lastName`, `tenantId`, `createdAt`, `updatedAt`
  - Use proxy endpoint `/api/proxy/user-profiles` for creation
  - Handle race conditions gracefully (profile might be created by another request)

- **Step 4: Final Fallback**
  - Return `null` if all steps fail
  - This triggers profile form display for manual creation
  - Log comprehensive failure information for debugging

## Profile Reconciliation Logic (New Section)

### Purpose
Handles cases where existing user profiles (especially those created via mobile payments) need to be updated with current Clerk user data during sign-in and profile page load.

### Trigger Points
- **Clerk Sign-In**: Direct client-side reconciliation after successful sign-in (PRIMARY METHOD)
- **Profile Page Load**: When user visits profile page after sign-in (FALLBACK METHOD)
- ~~**Clerk Webhook**: `session.created` webhook (DEPRECATED - unreliable in development)~~

### Scenarios Covered
1. **Mobile Payment Profiles**: Guest profiles with empty names get proper Clerk user data
2. **Incomplete Profiles**: Profiles with placeholder names ('Pending', 'User') get real names
3. **User ID Mismatches**: Profiles with old/guest user IDs get current Clerk user ID

### Reconciliation Conditions
Profile needs reconciliation if ANY of these are true:
- `profile.userId !== currentClerkUserId` (different user ID)
- `profile.firstName` is empty, null, or 'Pending'
- `profile.lastName` is empty, null, or 'User'

### Implementation Patterns

#### Clerk Sign-In Flow (Client-Side Integration) - PRIMARY METHOD
```typescript
// src/components/SignInWithReconciliation.tsx - Custom sign-in wrapper
'use client';
import { SignIn } from "@clerk/nextjs";
import { useUser } from "@clerk/nextjs";
import { useEffect, useState } from "react";

export function SignInWithReconciliation() {
  const { isSignedIn, user } = useUser();
  const [hasTriggeredReconciliation, setHasTriggeredReconciliation] = useState(false);

  useEffect(() => {
    // Trigger profile reconciliation immediately after successful sign-in
    if (isSignedIn && user && !hasTriggeredReconciliation) {
      setHasTriggeredReconciliation(true);

      // Call existing profile reconciliation API endpoint
      fetch('/api/auth/profile-reconciliation', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ triggerSource: 'sign_in_flow' })
      }).then(response => {
        if (response.ok) {
          console.log('Profile reconciliation completed after sign-in');
          setTimeout(() => window.location.href = '/', 1000);
        }
      }).catch(error => {
        console.error('Profile reconciliation failed:', error);
      });
    }
  }, [isSignedIn, user, hasTriggeredReconciliation]);

  return <SignIn redirectUrl="/" />;
}
```

#### ~~Clerk Sign-In Flow (Webhook) - DEPRECATED~~
```typescript
// DEPRECATED: Webhook approach was unreliable in development environments
// Use client-side integration instead (see above)
```

#### Profile Page Load Flow (Server Action)
```typescript
// src/app/profile/ApiServerActions.ts - fetchUserProfileServer Step 2
// Step 2: Fallback to email lookup with reconciliation
const profile = await findProfileByEmail(email);
if (profile && needsReconciliation(profile, userId, currentUser)) {
  const reconciledProfile = await reconcileProfileWithClerkData(profile, userId, currentUser);
  return reconciledProfile;
}
```

### Profile Update Helper Function
```typescript
async function reconcileProfileWithClerkData(
  profile: UserProfileDTO,
  currentClerkUserId: string,
  currentUser: any
): Promise<UserProfileDTO> {
  const updatePayload = {
    id: profile.id,
    userId: currentClerkUserId, // Always update to current Clerk user ID
    updatedAt: new Date().toISOString()
  };

  // Update names if they're empty or different
  if (currentUser?.firstName && (!profile.firstName || profile.firstName === 'Pending')) {
    updatePayload.firstName = currentUser.firstName;
  }
  if (currentUser?.lastName && (!profile.lastName || profile.lastName === 'User')) {
    updatePayload.lastName = currentUser.lastName;
  }

  return await updateUserProfileServer(profile.id, updatePayload);
}
```

### Reconciliation Benefits
1. **Seamless User Experience**: No manual profile completion needed
2. **Data Consistency**: Ensures profiles match current Clerk users
3. **Mobile Payment Integration**: Guest profiles get proper user data automatically
4. **Automatic Cleanup**: Removes placeholder/guest data

### Integration Points
- **Sign-In Page**: Custom `SignInWithReconciliation` component handles post-sign-in reconciliation
- **Profile Reconciliation API**: `/api/auth/profile-reconciliation` endpoint processes reconciliation requests
- **Profile Page**: `fetchUserProfileServer` includes reconciliation in Step 2 fallback (secondary method)
- **Mobile Payment Flow**: Profiles created during Stripe webhooks get reconciled on sign-in

### References
- **Primary Sign-In Flow**: `src/components/SignInWithReconciliation.tsx` for client-side reconciliation
- **Sign-In Page Integration**: `src/app/(auth)/sign-in/[[...sign-in]]/page.tsx` uses custom component
- **Reconciliation API**: `src/app/api/auth/profile-reconciliation/route.ts` for API endpoint
- **Profile Page Fallback**: `src/app/profile/ApiServerActions.ts` for server-side reconciliation
- ~~**Webhook Implementation**: `src/app/api/webhooks/clerk/route.ts` (deprecated due to reliability issues)~~
- **Mobile Payment Integration**: `src/app/api/webhooks/stripe/route.ts` for mobile payment profile creation

### **User Profile Update Operations (Direct Backend Pattern)**

- **Use Direct Backend Calls for PATCH Operations**
  - Never use proxy endpoints for PATCH operations
  - Follow cursor rules: `PATCH/PUT Server Actions: Direct Backend Update Pattern`
  - Use service JWT authentication, not Clerk session

- **Required Implementation Pattern**
  ```typescript
  export async function updateUserProfileServer(profileId: number, payload: Partial<UserProfileDTO>): Promise<UserProfileDTO | null> {
    try {
      // Direct backend call with service JWT
      const API_BASE_URL = process.env.NEXT_PUBLIC_API_BASE_URL;
      const url = `${API_BASE_URL}/api/user-profiles/${profileId}`;

      // Get service JWT
      let token = await getCachedApiJwt();
      if (!token) token = await generateApiJwt();

      // Include ID field in payload (required by backend)
      const finalPayload = { ...payload, id: profileId };

      const response = await fetch(url, {
        method: 'PATCH',
        headers: {
          'Content-Type': 'application/merge-patch+json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify(finalPayload),
      });

      // Handle response...
    } catch (error) {
      // Handle errors...
    }
  }
  ```

- **Required Fields for PATCH Operations**
  - Always include `id: profileId` in the payload
  - Use `Content-Type: application/merge-patch+json` for PATCH
  - Include `Authorization: Bearer <token>` header
  - Never send null values for required fields

### **Profile Creation Data Requirements**

- **Required Fields with Placeholders**
  - `userId`: Clerk user ID (required)
  - `email`: Clerk email or placeholder (no null)
  - `firstName`: Clerk firstName or 'Pending' (no null)
  - `lastName`: Clerk lastName or 'User' (no null)
  - `tenantId`: From environment (required)
  - `userRole`: 'ROLE_USER' (default)
  - `userStatus`: 'ACTIVE' (default)
  - `status`: 'PENDING' (default)
  - `createdAt`: Current timestamp (required)
  - `updatedAt`: Current timestamp (required)

- **Optional Fields**
  - Set empty strings `''` for text fields (not null)
  - Set `false` for boolean fields (not null)
  - Set `null` only for explicitly nullable fields

### **Error Handling and Logging**

- **Comprehensive Logging**
  - Log each step of the 4-step fallback process
  - Include payload details for debugging
  - Log race condition detection and resolution
  - Track profile creation success/failure

- **Graceful Error Handling**
  - Never let profile operations crash the application
  - Provide meaningful error messages
  - Fall back to profile form when automatic creation fails
  - Handle network errors and backend validation failures

### **Race Condition Prevention**

- **Duplicate Profile Creation**
  - Handle backend constraint violations gracefully
  - Implement retry logic for race conditions
  - Use singleton patterns to prevent duplicate API calls
  - Log race condition detection for monitoring

### **Proxy vs Direct Backend Usage**

- **Use Proxy For**
  - Profile creation (POST operations)
  - Profile lookup (GET operations)
  - List operations with filtering

- **Use Direct Backend For**
  - Profile updates (PATCH operations)
  - Profile deletions (DELETE operations)
  - Any operation requiring service JWT

### **References and Examples**

- **Working Implementation**: See `fetchUserProfileServer` for 4-step fallback pattern
- **PATCH Pattern**: Follow cursor rules for `PATCH/PUT Server Actions: Direct Backend Update Pattern`
- **Error Handling**: See current implementation for comprehensive error logging
- **Race Condition Handling**: See profile creation error handling for constraint violations

## **SOLUTION: Client-Side Sign-In Profile Reconciliation (IMPLEMENTED)**

### **Problem Solved**
The original webhook-based approach (`session.created` event) was unreliable in development environments and required external webhook configuration. Users signing in with existing mobile payment profiles weren't getting their profiles updated with proper Clerk user data.

### **Solution Implemented**
Direct client-side profile reconciliation integrated into the Clerk sign-in flow using a custom wrapper component.

### **Implementation Steps Taken**

**1. Created Custom Sign-In Wrapper Component**
- File: `src/components/SignInWithReconciliation.tsx`
- Uses `useUser` hook to detect successful sign-in
- Automatically triggers profile reconciliation API call
- Shows user feedback and handles errors gracefully
- Redirects to home page after completion

**2. Updated Sign-In Page**
- File: `src/app/(auth)/sign-in/[[...sign-in]]/page.tsx`
- Replaced default `<SignIn>` component with `<SignInWithReconciliation>`
- Maintains all existing Clerk sign-in functionality
- Adds automatic profile reconciliation behavior

**3. Leveraged Existing Infrastructure**
- Reuses existing `/api/auth/profile-reconciliation` endpoint
- Uses same reconciliation logic as profile page fallback
- Maintains consistency with existing error handling and logging
- No changes needed to backend reconciliation logic

### **Flow Architecture**
```
User Sign-In → Clerk Authentication → useUser Hook Detects Success →
API Call to /api/auth/profile-reconciliation → Profile Lookup by Email →
Reconciliation Check → Profile Update with Clerk Data → Success Feedback →
Redirect to Home Page
```

### **Advantages of This Approach**
1. **No Webhook Dependencies**: Works without external webhook configuration
2. **Immediate Execution**: Reconciliation happens immediately after sign-in
3. **User Feedback**: Users see confirmation of the process
4. **Reliable**: Client-side execution is more predictable than webhook timing
5. **Reuses Existing Code**: Leverages proven reconciliation API endpoint
6. **Development Friendly**: Works consistently in local development environments

### **Key Components**
- **`SignInWithReconciliation`**: Custom client component that wraps Clerk SignIn
- **`useUser` Hook**: Detects successful authentication state changes
- **`/api/auth/profile-reconciliation`**: Existing API endpoint for reconciliation logic
- **Profile Reconciliation Logic**: Existing server-side functions in `ApiServerActions.ts`

### **Testing Verification**
1. Manually modify profile data to incorrect values in database
2. Sign out and sign in again using the updated sign-in page
3. Observe console logs for `[SignInWithReconciliation]` messages
4. Verify profile gets updated with correct Clerk user data
5. Confirm user is redirected to home page after success

### **Logging Tags for Debugging**
```typescript
'[SignInWithReconciliation]' // Client-side sign-in reconciliation process
'[PROFILE-RECONCILIATION-API]' // Server-side API endpoint processing
'[Profile Reconciliation]' // Server-side reconciliation logic execution
```

### **Integration Status**
✅ **COMPLETED**: Client-side sign-in profile reconciliation is fully integrated and tested
✅ **VERIFIED**: Mobile payment profiles get updated with Clerk data on sign-in
✅ **DOCUMENTED**: Cursor rules updated with implementation details
⚠️ **DEPRECATED**: Webhook-based approach marked as unreliable for development

### **Future Considerations**
- Keep webhook implementation for production environments where webhooks are properly configured
- Consider adding user notification/toast messages for better UX feedback
- Monitor reconciliation success rates and add analytics if needed
- Extend pattern to other authentication flows (sign-up, OAuth providers) if required

### **Mobile Payment Flow Integration (Stripe Webhook Pattern)**

- **Integration Point**: User profile creation/update happens in Stripe webhook `payment_intent.succeeded` event
- **Timing**: Asynchronous execution after critical payment operations to avoid blocking webhook response
- **Data Source**: Stripe Payment Intent metadata and customer information

#### **Mobile Payment Profile Creation Flow**

**1. Webhook Event Processing**
```typescript
// In Stripe webhook payment_intent.succeeded handler
case 'payment_intent.succeeded':
  // Extract user data from Stripe
  const customerEmail = pi.receipt_email || md.customerEmail;
  const customerName = await getStripeCustomerName(pi.customer);
  const { firstName, lastName } = extractNameFromStripe(customerName);

  // Schedule profile operation asynchronously
  setTimeout(async () => {
    await createOrUpdateUserProfileFromStripe(
      customerEmail, firstName, lastName, phone, baseUrl
    );
  }, 1000);
```

**2. Stripe Data Extraction**
```typescript
function extractNameFromStripe(stripeName: string | null | undefined): {
  firstName: string;
  lastName: string
} {
  if (!stripeName || stripeName.trim().length === 0) {
    return { firstName: '', lastName: '' };
  }

  const nameParts = stripeName.trim().split(' ');
  if (nameParts.length === 1) {
    return { firstName: nameParts[0], lastName: '' };
  }

  const firstName = nameParts[0];
  const lastName = nameParts.slice(1).join(' ');
  return { firstName, lastName };
}
```

**3. Profile Creation/Update Logic**
```typescript
async function createOrUpdateUserProfileFromStripe(
  email: string,
  firstName: string,
  lastName: string,
  phone: string,
  baseUrl: string
): Promise<void> {
  // Step 1: Lookup existing profile by email
  const existingProfile = await findProfileByEmail(email);

  if (existingProfile) {
    // Update existing profile with Stripe data
    await updateProfileWithStripeData(existingProfile, firstName, lastName, phone);
  } else {
    // Create new profile with guest userId
    const guestUserId = `guest_${email}_${Date.now()}`;
    await createProfileWithStripeData(guestUserId, email, firstName, lastName, phone);
  }
}
```

#### **Mobile Payment Profile Data Requirements**

- **Required Fields for Mobile Profiles**
  - `userId`: Generated guest ID (`guest_{email}_{timestamp}`)
  - `email`: From Stripe `receipt_email` or metadata
  - `firstName`: From Stripe customer name (may be empty)
  - `lastName`: From Stripe customer name (may be empty)
  - `phone`: From Stripe customer phone (may be empty)
  - `tenantId`: From environment variable
  - `userStatus`: 'ACTIVE' (default)
  - `userRole`: 'MEMBER' (default)
  - `status`: 'PENDING_COMPLETION' (indicates incomplete profile)

- **Data Limitations in Mobile Flow**
  - **Stripe PRB provides minimal data**: Only payment method, amount, email
  - **Names often empty**: Mobile wallets don't collect customer names
  - **Phone often empty**: Mobile wallets don't collect phone numbers
  - **Email is primary identifier**: Used for profile lookup and creation

#### **Asynchronous Execution Pattern**

**Why Asynchronous?**
- Webhook must respond quickly to Stripe (prevents timeout)
- Profile operations shouldn't block payment processing
- Allows for comprehensive error handling and retry logic

**Implementation Pattern**
```typescript
// Schedule profile operation after webhook response
setTimeout(async () => {
  try {
    console.log('[STRIPE-WEBHOOK] [USER-PROFILE-ASYNC] Starting profile operation');

    // Validate required data
    if (!customerEmail || customerEmail.trim().length === 0) {
      console.warn('[STRIPE-WEBHOOK] [USER-PROFILE-ASYNC] No valid email, skipping profile operation');
      return;
    }

    // Execute profile creation/update
    await createOrUpdateUserProfileFromStripe(
      customerEmail, firstName, lastName, phone, baseUrl
    );

    console.log('[STRIPE-WEBHOOK] [USER-PROFILE-ASYNC] Profile operation completed');
  } catch (error) {
    console.error('[STRIPE-WEBHOOK] [USER-PROFILE-ASYNC] Profile operation failed:', error);
  }
}, 1000); // 1 second delay
```

#### **Mobile Profile Completion Strategy**

**1. Accept Limited Initial Profiles**
- Create profiles with available data (email + empty name/phone)
- Mark as `PENDING_COMPLETION` status
- Don't require immediate completion

**2. Deferred Profile Completion**
- Users can complete profiles when they visit the site
- Provide profile completion forms on dashboard/profile pages
- Use existing profile update patterns for completion

**3. Profile Status Tracking**
```typescript
// Profile status values for mobile flow
enum ProfileStatus {
  PENDING_COMPLETION = 'PENDING_COMPLETION',  // Mobile payment profile
  COMPLETED = 'COMPLETED',                    // Full profile data
  VERIFIED = 'VERIFIED'                       // Verified user
}
```

#### **Error Handling for Mobile Profiles**

**1. Stripe Data Validation**
```typescript
// Validate extracted data before profile operations
if (!customerEmail || customerEmail.trim().length === 0) {
  console.warn('[STRIPE-WEBHOOK] [USER-PROFILE-ASYNC] Invalid email data');
  return; // Skip profile operation
}

// Accept empty names/phone (normal for mobile PRB)
const hasValidData = customerEmail && customerEmail.trim().length > 0;
if (!hasValidData) {
  console.warn('[STRIPE-WEBHOOK] [USER-PROFILE-ASYNC] No valid data for profile');
  return;
}
```

**2. Profile Operation Failures**
```typescript
try {
  await createOrUpdateUserProfileFromStripe(email, firstName, lastName, phone, baseUrl);
} catch (error) {
  console.error('[STRIPE-WEBHOOK] [USER-PROFILE-ASYNC] Profile operation failed:', {
    error: String(error),
    email,
    firstName,
    lastName,
    phone,
    timestamp: new Date().toISOString()
  });

  // Don't retry - let user complete profile manually later
  // Payment was successful, profile is secondary
}
```

#### **Logging and Debugging for Mobile Profiles**

**Required Log Tags**
```typescript
// Use consistent log prefixes for easy filtering
'[STRIPE-WEBHOOK] [USER-PROFILE-ASYNC]'     // Async operation start
'[STRIPE-WEBHOOK] [USER-PROFILE]'           // Profile operation details
'[STRIPE-WEBHOOK] [STRIPE-DATA-INSPECTION]' // Raw Stripe data analysis
'[STRIPE-WEBHOOK] [USER-DATA-EXTRACTION]'   // Data extraction process
'[STRIPE-WEBHOOK] [PROFILE-DATA-VALIDATION]' // Data validation results
'[STRIPE-WEBHOOK] [MOBILE-PAYMENT-SUMMARY]' // Complete flow summary
```

**Debug Information to Log**
- Stripe Payment Intent metadata and customer data
- Extracted user information (email, name, phone)
- Profile lookup results (existing vs. new)
- Database operation results (create/update success/failure)
- Timing information for async operations

#### **Integration with Existing Profile Patterns**

**1. Follow 4-Step Fallback for Lookups**
- Use email-based lookup in mobile flow
- Fall back to profile creation if none exists
- Maintain consistency with authenticated profile patterns

**2. Use Proxy Endpoints for Creation**
- Use `/api/proxy/user-profiles` for profile creation
- Follow existing DTO patterns and validation
- Maintain tenant isolation and JWT authentication

**3. Extend Profile Update Patterns**
- Use existing PATCH patterns for profile completion
- Maintain audit trail and change tracking
- Follow security and validation requirements

#### **Best Practices for Mobile Profile Integration**

**1. Never Block Payment Processing**
- Profile operations must be asynchronous
- Payment success is primary, profile is secondary
- Use timeouts and error boundaries

**2. Accept Incomplete Data**
- Mobile PRB limitations are normal
- Empty names/phone are acceptable initial values
- Focus on email-based identification

**3. Provide Completion Paths**
- Clear profile completion forms
- Incentivize profile completion
- Track completion rates and user engagement

**4. Maintain Data Consistency**
- Use consistent field names and types
- Follow existing validation patterns
- Maintain audit trails and change history

#### **References for Mobile Profile Integration**

- **Stripe Webhook Handler**: `src/app/api/webhooks/stripe/route.ts`
- **Profile Creation Logic**: `createOrUpdateUserProfileFromStripe` function
- **Name Extraction**: `extractNameFromStripe` function
- **Asynchronous Pattern**: `setTimeout` with comprehensive logging
- **Mobile Payment Flow**: See `mobile_payment_flow.mdc` for complete architecture

- **Working Implementation**: See `fetchUserProfileServer` for 4-step fallback pattern
- **PATCH Pattern**: Follow cursor rules for `PATCH/PUT Server Actions: Direct Backend Update Pattern`
- **Error Handling**: See current implementation for comprehensive error logging
- **Race Condition Handling**: See profile creation error handling for constraint violations

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
