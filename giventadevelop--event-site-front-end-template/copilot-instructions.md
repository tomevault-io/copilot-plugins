## event-site-front-end-template

> Rules for implementing desktop browser payment flow (Stripe Checkout), transaction persistence, and success page handling separate from mobile workflow. Includes membership subscription flow with Stripe Subscription ID persistence.


- **Main Points in Bold**
  - Desktop flow uses Stripe Checkout Sessions (`cs_`) OR Stripe Elements inline (ExpressCheckoutElement + PaymentElement) for payment processing
  - **Stripe Elements Inline Option**: Desktop can use Stripe Elements (`StripeDesktopCheckout` component) to show Apple Pay, Google Pay, Link, and card form directly on the page (no redirect)
  - **Stripe Checkout Session Option**: Desktop can also use Stripe Checkout Sessions (hosted payment page) via redirect
  - Desktop flow persists transactions immediately when payment succeeds via GET endpoint fallback
  - Desktop flow is completely separate from mobile workflow - uses different entry points and detection methods
  - Desktop flow must include both `tenantId` and `paymentMethodDomainId` in all backend requests
  - Desktop flow creates both transaction and transaction items via `createTransactionFromPaymentIntent`
  - **Membership Subscription Flow**: Desktop membership flow creates Stripe Subscription for recurring billing and stores `stripeSubscriptionId`, `stripeCustomerId`, and `stripePaymentIntentId` in database
  - **CRITICAL**: Membership subscriptions must filter out CANCELLED/EXPIRED subscriptions when looking up existing subscriptions
  - Mobile workflow (`/event/ticket-qr` page) remains completely untouched

# Desktop Browser Payment Flow Architecture

This document outlines the comprehensive desktop browser payment flow architecture for the MCEFEE event management application, detailing how desktop payments differ from mobile payments and how transactions are persisted.

## Overview

The desktop payment flow is fundamentally different from the mobile flow due to:
- Different payment methods (Stripe Checkout vs Payment Request Button)
- Different user experience (hosted payment page vs native wallet)
- Different transaction persistence strategy (immediate creation vs polling with fallback)
- Different success page handling (inline display vs redirect to QR page)

## Desktop vs Mobile Flow Comparison

### Desktop Flow (Two Options)

#### Option 1: Stripe Elements Inline (Preferred for Registered Users)
```
User fills form →
Payment Intent created via /api/stripe/payment-intent (or /api/stripe/membership-payment-intent) →
Stripe Elements rendered inline (ExpressCheckoutElement + PaymentElement) →
Apple Pay/Google Pay/Link/Card form shown on same page →
Payment completion →
Redirect to /event/success?pi=pi_xxx (or /membership/success?pi=pi_xxx) →
SuccessClient detects desktop →
Stays on success page →
GET /api/event/success/process?pi=pi_xxx (or /api/membership/success/process?pi=pi_xxx) →
If transaction/subscription not found (webhook delayed) →
Create transaction/subscription immediately via createTransactionFromPaymentIntent/processMembershipSubscriptionFromPaymentIntent →
For events: Create transaction items via createTransactionItemsBulk →
For memberships: Create Stripe Subscription for recurring billing →
Fetch QR code (events) or display subscription confirmation (memberships) →
Display success page with QR inline (events) or subscription details (memberships)
```

#### Option 2: Stripe Checkout Session (Fallback for Visitors)
```
User fills form →
Stripe Checkout Session created →
Hosted payment page →
Payment completion →
Redirect to /event/success?session_id=cs_xxx or ?pi=pi_xxx →
SuccessClient detects desktop →
Stays on success page →
GET /api/event/success/process?pi=pi_xxx →
If transaction not found (webhook delayed) →
Create transaction immediately via createTransactionFromPaymentIntent →
Create transaction items via createTransactionItemsBulk →
Fetch QR code →
Display success page with QR inline
```

### Mobile Flow (Separate - Unchanged)
```
User taps PRB →
Native wallet sheet opens →
Payment Intent created →
Payment confirmed with PI →
Redirect to /event/success?pi=pi_xxx →
SuccessClient detects mobile →
Shows brief success (2 seconds) →
Redirect to /event/ticket-qr?pi=pi_xxx →
Dedicated QR page with POST fallback →
Transaction created via POST endpoint
```

**CRITICAL**: Desktop and mobile flows are completely separate:
- **Desktop**: Uses GET endpoint with transaction creation fallback
- **Mobile**: Uses POST endpoint (via `/event/ticket-qr` page)
- **Entry Points**: Different API routes and client components
- **Mobile workflow files remain untouched**: `/event/ticket-qr` page and related mobile components are not modified

## Desktop Browser Detection Methods

### Client-Side Detection (`SuccessClient.tsx`)

The application uses multiple methods to reliably detect desktop browsers:

#### 1. User Agent Detection (Primary)
```typescript
const mobileRegexMatch = /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini|Mobile|mobile|CriOS|FxiOS|EdgiOS/i.test(userAgent);
const platformMatch = /iPhone|iPad|iPod|Android|BlackBerry|Windows Phone/i.test(platform);
```

#### 2. Screen Width Detection (Secondary)
```typescript
const narrowScreenMatch = window.innerWidth <= 768;
const hasMobileUserAgent = mobileRegexMatch || platformMatch;
```

#### 3. Combined Detection Logic
```typescript
// CRITICAL: Desktop detection - Only consider mobile if:
// 1. User agent indicates mobile (primary method), OR
// 2. User agent data API says mobile, OR
// 3. Narrow screen AND mobile user agent (not just narrow screen alone)
// This prevents desktop browsers with narrow windows from being detected as mobile
const isMobile = mobileRegexMatch || platformMatch || isMobileFromUA || (hasMobileUserAgent && narrowScreenMatch);
const isDesktop = !isMobile; // Desktop is the inverse of mobile detection
```

**Key Points**:
- Desktop detection is the inverse of mobile detection
- Desktop browsers with narrow windows are NOT detected as mobile
- Multiple detection methods prevent false positives
- Detection happens immediately on component mount

### Server-Side Detection (`/api/event/success/process/route.ts`)

Server-side detection for logging and routing decisions:

#### CloudFront Headers (AWS Deployment)
```typescript
const cloudfrontMobile = req.headers.get('cloudfront-is-mobile-viewer') === 'true';
const cloudfrontAndroid = req.headers.get('cloudfront-is-android-viewer') === 'true';
const cloudfrontIOS = req.headers.get('cloudfront-is-ios-viewer') === 'true';
```

#### User Agent Analysis
```typescript
const mobileRegexMatch = /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini|Mobile|mobile|CriOS|FxiOS|EdgiOS/i.test(userAgent);
const platformMatch = /iPhone|iPad|iPod|Android|BlackBerry|Windows Phone/i.test(userAgent);
const isMobile = mobileRegexMatch || platformMatch || cloudfrontMobile || cloudfrontAndroid || cloudfrontIOS;
```

**Server-Side Detection Purpose**:
- CloudWatch logging for debugging
- Route decision making (desktop vs mobile transaction creation)
- Request routing and validation

## Desktop Payment Flow Architecture

### Payment Processing Entry Points

#### 1. Stripe Elements Inline Flow (Preferred for Registered Users)
```
User fills form (email, name, etc.) →
Payment Intent created via /api/stripe/payment-intent (or /api/stripe/membership-payment-intent) →
Stripe Elements rendered inline (ExpressCheckoutElement + PaymentElement) →
Apple Pay/Google Pay/Link/Card form shown on same page (no redirect) →
User selects payment method and completes payment →
Payment confirmed via stripe.confirmPayment() →
Redirect to /event/success?pi=pi_xxx (or /membership/success?pi=pi_xxx) →
SuccessClient detects desktop →
Stays on success page →
GET /api/event/success/process?pi=pi_xxx →
If transaction not found (webhook delayed) →
Create transaction immediately via createTransactionFromPaymentIntent →
Create transaction items via createTransactionItemsBulk →
Fetch QR code →
Display success page with QR inline
```

**When to Use Stripe Elements Inline:**
- User is registered (has user profile with email)
- Better UX - no redirect, payment form on same page
- Shows Apple Pay, Google Pay, Link, and card form inline
- Example: Event checkout page (`/events/[id]/checkout`), Membership subscribe page (`/membership/subscribe/[planId]`)

**Components:**
- `StripeDesktopCheckout` - For event tickets
- `MembershipDesktopCheckout` - For membership subscriptions

**CRITICAL: Membership Subscription Flow Differences:**
- Creates Stripe Subscription for recurring billing (not just one-time Payment Intent)
- Stores `stripeSubscriptionId`, `stripeCustomerId`, and `stripePaymentIntentId` in database
- Creates Stripe Product/Price on the fly if plan doesn't have `stripePriceId`
- Filters out CANCELLED/EXPIRED subscriptions when looking up existing subscriptions
- Uses `payment_behavior: 'default_incomplete'` to allow subscription creation without immediate payment method

#### 2. Stripe Checkout Session Flow (Fallback for Visitors)
```
User fills form →
Stripe Checkout Session created →
Redirect to Stripe hosted payment page →
Payment completion →
Redirect to /event/success?session_id=cs_xxx →
SuccessClient detects desktop →
Stays on success page →
GET /api/event/success/process?session_id=cs_xxx →
Lookup transaction by session_id →
If not found, convert session_id to payment_intent →
Create transaction from payment_intent
```

**When to Use Stripe Checkout Session:**
- User is a visitor (no user profile, no email)
- Fallback when Stripe Elements cannot be enabled
- Simpler implementation (no client-side Stripe Elements setup)

#### 3. Payment Intent Direct Flow
```
User completes payment →
Redirect to /event/success?pi=pi_xxx →
SuccessClient detects desktop →
GET /api/event/success/process?pi=pi_xxx →
Lookup transaction by payment_intent →
If not found, create transaction from payment_intent
```

### Desktop Transaction Persistence Flow

**CRITICAL**: Desktop flow persists transactions immediately when payment succeeds, even if webhook hasn't processed yet.

#### GET Endpoint Transaction Creation (`/api/event/success/process` GET handler)

**When Transaction Creation Triggers**:
1. Transaction lookup returns `null` (webhook hasn't processed yet)
2. Request is from desktop browser (`isMobile === false`)
3. Payment Intent ID (`pi`) is provided
4. Payment Intent status is `succeeded`

**Transaction Creation Process**:
```typescript
// 1. Retrieve Payment Intent from Stripe
const paymentIntent = await stripe.paymentIntents.retrieve(pi, {
  expand: ['charges.data.balance_transaction']
});

// 2. Validate payment succeeded
if (paymentIntent.status !== 'succeeded') {
  return { error: 'Payment not completed yet' };
}

// 3. Extract metadata (cart, eventId, customer info)
const metadata = paymentIntent.metadata || {};
const cartJson = metadata.cart;
const eventIdRaw = metadata.eventId;
const customerEmail = paymentIntent.receipt_email || metadata.customerEmail || '';

// 4. Validate tenant ID and payment method domain ID
const metadataTenantId = metadata.tenantId || metadata.tenant_id;
const metadataPaymentMethodDomainId = metadata.paymentMethodDomainId || metadata.payment_method_domain_id;
const expectedTenantId = getTenantId();
const expectedPaymentMethodDomainId = getPaymentMethodDomainId();

// CRITICAL: Validate metadata matches environment variables
if (metadataTenantId && metadataTenantId !== expectedTenantId) {
  return { error: 'Tenant ID mismatch' };
}
if (metadataPaymentMethodDomainId && metadataPaymentMethodDomainId !== expectedPaymentMethodDomainId) {
  return { error: 'Payment Method Domain ID mismatch' };
}

// 5. Create transaction via createTransactionFromPaymentIntent
const { createTransactionFromPaymentIntent } = await import('@/app/event/success/ApiServerActions');
const createdTransaction = await createTransactionFromPaymentIntent(
  pi,
  eventId,
  customerEmail,
  cartItems,
  amountTotal,
  firstName,
  lastName,
  customerPhone
);
```

**Key Points**:
- Desktop flow creates transactions immediately (no waiting for webhook)
- Uses same `createTransactionFromPaymentIntent` function as mobile (but different entry point)
- Validates tenant ID and payment method domain ID before creation
- Creates both transaction and transaction items in single flow

## Transaction and Transaction Items Persistence

### CRITICAL: Both Transaction and Transaction Items Must Be Created

**Desktop flow MUST create both**:
1. **Event Ticket Transaction** (main transaction record)
2. **Event Ticket Transaction Items** (bulk creation of ticket line items)

### Transaction Creation (`createTransaction` function)

**Location**: `src/app/event/success/ApiServerActions.ts`

**Process**:
```typescript
async function createTransaction(transactionData: Omit<EventTicketTransactionDTO, 'id'>): Promise<EventTicketTransactionDTO> {
  // 1. Get tenant ID and payment method domain ID from environment variables
  const expectedTenantId = getTenantId();
  const expectedPaymentMethodDomainId = getPaymentMethodDomainId();

  // 2. Validate transactionData matches environment variables
  if (transactionData.tenantId !== expectedTenantId) {
    throw new Error('Tenant ID mismatch');
  }
  if (transactionData.paymentMethodDomainId !== expectedPaymentMethodDomainId) {
    throw new Error('Payment Method Domain ID mismatch');
  }

  // 3. Build payload with environment variables (not metadata)
  const payload = {
    ...transactionData,
    tenantId: expectedTenantId, // ALWAYS use environment tenant ID
    paymentMethodDomainId: expectedPaymentMethodDomainId, // ALWAYS use environment Payment Method Domain ID
  };

  // 4. POST to backend via proxy
  const response = await fetchWithJwtRetry(
    `${getAppUrl()}/api/proxy/event-ticket-transactions`,
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload),
    }
  );

  return await response.json();
}
```

**Key Points**:
- Always uses environment variables for `tenantId` and `paymentMethodDomainId`
- Validates metadata matches environment variables before creation
- Includes `X-Tenant-ID` header in request (via `fetchWithJwtRetry`)
- Backend validates triple combination (tenantId, paymentMethodDomainId, webhookSecret)

### Transaction Items Creation (`createTransactionItemsBulk` function)

**Location**: `src/app/event/success/ApiServerActions.ts`

**CRITICAL**: Transaction items MUST be created after transaction creation.

**Process**:
```typescript
async function createTransactionItemsBulk(items: any[]): Promise<any[]> {
  // 1. Get tenant ID and payment method domain ID from environment variables
  const expectedTenantId = getTenantId();
  const expectedPaymentMethodDomainId = getPaymentMethodDomainId();

  // 2. Validate each item's tenant ID and payment method domain ID
  for (const item of items) {
    if (item.tenantId && item.tenantId !== expectedTenantId) {
      throw new Error('Tenant ID mismatch in item');
    }
    if (item.paymentMethodDomainId && item.paymentMethodDomainId !== expectedPaymentMethodDomainId) {
      throw new Error('Payment Method Domain ID mismatch in item');
    }
  }

  // 3. Build payload with environment variables
  const payload = items.map(item => ({
    ...item,
    tenantId: expectedTenantId, // ALWAYS use environment tenant ID
    paymentMethodDomainId: expectedPaymentMethodDomainId, // ALWAYS use environment Payment Method Domain ID
  }));

  // 4. POST to backend via proxy (bulk endpoint)
  const response = await fetchWithJwtRetry(
    `${getAppUrl()}/api/proxy/event-ticket-transaction-items/bulk`,
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload),
    }
  );

  return await response.json();
}
```

**Key Points**:
- Creates transaction items in bulk (single API call)
- Validates each item before creation
- Includes `tenantId` and `paymentMethodDomainId` in each item
- Uses `/api/proxy/event-ticket-transaction-items/bulk` endpoint

### Complete Desktop Transaction Creation Flow

**Function**: `createTransactionFromPaymentIntent` (used by both desktop and mobile, but different entry points)

**Location**: `src/app/event/success/ApiServerActions.ts`

**Complete Process**:
```typescript
export async function createTransactionFromPaymentIntent(
  paymentIntentId: string,
  eventId: number,
  customerEmail: string,
  cart: { ticketTypeId: number; quantity: number }[],
  amountPaid: number,
  firstName?: string,
  lastName?: string,
  phone?: string
): Promise<EventTicketTransactionDTO> {
  // 1. Check if transaction already exists (prevent duplicates)
  const existingTransaction = await findTransactionByPaymentIntentId(paymentIntentId);
  if (existingTransaction) {
    return existingTransaction; // Return existing, don't create duplicate
  }

  // 2. Get tenant ID and payment method domain ID from environment variables
  const expectedTenantId = getTenantId();
  const expectedPaymentMethodDomainId = getPaymentMethodDomainId();

  // 3. Build transaction data with environment variables
  const transactionData: Omit<EventTicketTransactionDTO, 'id'> = withTenantId({
    email: customerEmail,
    firstName: firstName || '',
    lastName: lastName || '',
    phone: phone || '',
    quantity: totalQuantity,
    totalAmount: amountPaid,
    status: 'COMPLETED',
    stripePaymentIntentId: paymentIntentId,
    eventId,
    paymentMethodDomainId: expectedPaymentMethodDomainId, // CRITICAL: Always from environment
    // ... other fields
  });

  // 4. Create transaction
  const transaction = await createTransaction(transactionData);

  // 5. Create transaction items
  const transactionItems = [];
  for (const cartItem of cart) {
    const ticketType = await fetchTicketTypeByIdServer(cartItem.ticketTypeId);
    const itemData = withTenantId({
      transactionId: transaction.id,
      ticketTypeId: cartItem.ticketTypeId,
      quantity: cartItem.quantity,
      pricePerUnit: ticketType.price,
      totalAmount: ticketType.price * cartItem.quantity,
      paymentMethodDomainId: expectedPaymentMethodDomainId, // CRITICAL: Always from environment
    });
    transactionItems.push(itemData);
  }

  // 6. Check if items already exist (prevent duplicates)
  const existingItems = await checkExistingItems(transaction.id);
  const itemsToCreate = transactionItems.filter(item => {
    return !existingItems.find(existing =>
      existing.ticketTypeId === item.ticketTypeId &&
      existing.transactionId === transaction.id
    );
  });

  // 7. Create transaction items in bulk
  if (itemsToCreate.length > 0) {
    await createTransactionItemsBulk(itemsToCreate);
  }

  return transaction;
}
```

**Key Points**:
- Creates transaction first, then transaction items
- Checks for existing transaction and items before creating (idempotency)
- Uses environment variables for `tenantId` and `paymentMethodDomainId`
- Creates transaction items in bulk (single API call)
- Handles duplicate prevention at both transaction and item levels

## Tenant ID and Payment Method Domain ID Handling

### CRITICAL: Both Must Be Passed Together

**Backend Requirement**: Backend requires both `tenantId` and `paymentMethodDomainId` together for:
- Transaction lookup queries
- Transaction creation
- Transaction items creation
- Triple validation (tenantId, paymentMethodDomainId, webhookSecret)

### Environment Variable Sources

**Tenant ID**:
```typescript
// Priority order (AWS Amplify compatible)
const tenantId =
  process.env.AMPLIFY_NEXT_PUBLIC_TENANT_ID ||
  process.env.NEXT_PUBLIC_TENANT_ID;
```

**Payment Method Domain ID**:
```typescript
// Priority order (AWS Amplify compatible)
const paymentMethodDomainId =
  process.env.AMPLIFY_NEXT_PUBLIC_PAYMENT_METHOD_DOMAIN_ID ||
  process.env.NEXT_PUBLIC_PAYMENT_METHOD_DOMAIN_ID;
```

### Request Headers and Query Parameters

**X-Tenant-ID Header** (via `fetchWithJwtRetry`):
```typescript
// CRITICAL: Backend TenantContextFilter expects this header
headers: {
  'Authorization': `Bearer ${token}`,
  'X-Tenant-ID': tenantId, // Backend extracts tenant ID from this header
}
```

**Query Parameters** (for transaction lookup):
```typescript
const params = new URLSearchParams({
  'stripePaymentIntentId.equals': paymentIntentId,
  'tenantId.equals': tenantId, // Filter by tenant
  'paymentMethodDomainId.equals': paymentMethodDomainId, // CRITICAL: Backend requires both
});
```

**Request Body** (for transaction creation):
```typescript
const payload = {
  ...transactionData,
  tenantId: expectedTenantId, // From environment variable
  paymentMethodDomainId: expectedPaymentMethodDomainId, // From environment variable
};
```

### Backend Validation

**Backend Triple Validation**:
- Backend validates that the combination (`tenantId`, `paymentMethodDomainId`, `webhookSecret`) exists in `payment_provider_config` table
- If combination doesn't exist, backend returns `tripleValidationFailed` error
- Frontend must always pass both `tenantId` and `paymentMethodDomainId` together

## Stripe Webhook Listener and Desktop Flow

### Webhook Handler (`src/app/api/webhooks/stripe/route.ts`)

**CRITICAL**: Webhook handler does NOT differentiate between desktop and mobile - it processes all `payment_intent.succeeded` events.

**Webhook Process**:
```typescript
case 'payment_intent.succeeded':
  // 1. Extract Payment Intent ID
  const pi = event.data.object as Stripe.PaymentIntent;
  const paymentIntentId = pi.id;

  // 2. Check if transaction already exists (prevent duplicates)
  const existingTransaction = await findTransactionByPaymentIntentId(paymentIntentId);
  if (existingTransaction) {
    console.log('[STRIPE-WEBHOOK] Transaction already exists - skipping creation');
    break; // Skip if already exists
  }

  // 3. Extract metadata and validate tenant ID and payment method domain ID
  const metadata = pi.metadata || {};
  const metadataTenantId = metadata.tenantId || metadata.tenant_id;
  const metadataPaymentMethodDomainId = metadata.paymentMethodDomainId || metadata.payment_method_domain_id;
  const expectedTenantId = getTenantId();
  const expectedPaymentMethodDomainId = getPaymentMethodDomainId();

  // 4. Validate metadata matches environment variables
  if (metadataTenantId && metadataTenantId !== expectedTenantId) {
    console.error('[STRIPE-WEBHOOK] Tenant ID mismatch - rejecting');
    return new NextResponse('Tenant ID mismatch', { status: 403 });
  }

  // 5. Create transaction with tenant ID and payment method domain ID
  const txPayload = {
    ...transactionData,
    tenantId: expectedTenantId, // From environment variable
    paymentMethodDomainId: expectedPaymentMethodDomainId, // From environment variable
  };

  // 6. Create transaction via backend API
  const created = await createEventTicketTransactionServer(withTenantId(txPayload));
```

**Key Points**:
- Webhook processes all payment intents (desktop and mobile)
- Webhook validates tenant ID and payment method domain ID
- Webhook creates transaction if it doesn't exist
- Desktop flow creates transaction if webhook hasn't processed yet (fallback)

### Desktop Flow Fallback Strategy

**Why Desktop Flow Creates Transactions**:
- Webhook may be delayed (network issues, backend processing time)
- Desktop users expect immediate transaction display
- Desktop flow acts as "client-side webhook listener" - creates transaction immediately when payment succeeds

**Fallback Flow**:
1. **Primary**: Webhook creates transaction automatically
2. **Fallback**: Desktop GET endpoint creates transaction if not found
3. **Idempotency**: Both paths check for existing transaction before creating

**Desktop Flow Transaction Creation Trigger**:
```typescript
// In GET handler (/api/event/success/process)
if (!transaction) {
  // Desktop flow - create transaction immediately if payment succeeded
  if (!isMobile && pi) {
    const paymentIntent = await stripe.paymentIntents.retrieve(pi);
    if (paymentIntent.status === 'succeeded') {
      // Create transaction immediately (webhook fallback)
      const createdTransaction = await createTransactionFromPaymentIntent(...);
      transaction = createdTransaction;
    }
  }
}
```

## Desktop Membership Subscription Flow

### CRITICAL: Stripe Subscription ID Persistence

**Problem**: When processing membership subscriptions from Payment Intents, the database was missing `stripeSubscriptionId` and `stripeCustomerId` fields, even though the payment succeeded.

**Root Cause**: Payment Intents are one-time payments and don't automatically create Stripe Subscriptions for recurring billing. The system needs to explicitly create a Stripe Subscription for recurring charges.

**Solution**: Create Stripe Subscription during Payment Intent processing, even if payment method cannot be attached immediately.

### Membership Subscription Processing Flow

**Function**: `processMembershipSubscriptionFromPaymentIntent`

**Location**: `src/app/membership/success/ApiServerActions.ts`

**Complete Process**:
```typescript
export async function processMembershipSubscriptionFromPaymentIntent(
  paymentIntentId: string,
  userId?: string | null,
): Promise<{ subscription: MembershipSubscriptionDTO | null; plan: MembershipPlanDTO | null; userProfile: UserProfileDTO | null } | null> {
  // 1. Retrieve Payment Intent from Stripe
  const paymentIntent = await stripe().paymentIntents.retrieve(paymentIntentId, {
    expand: ['payment_method'],
  });

  // 2. Check Payment Intent status (accepts 'succeeded' or 'requires_capture')
  if (paymentIntent.status !== 'succeeded' && paymentIntent.status !== 'requires_capture') {
    return null;
  }

  // 3. Extract metadata and user profile
  const metadata = paymentIntent.metadata || {};
  const membershipPlanId = metadata.membershipPlanId;
  const customerEmail = metadata.customerEmail || paymentIntent.receipt_email || '';

  // CRITICAL: Get userId from email if not provided (for public routes)
  let userProfile: UserProfileDTO | null = null;
  if (!userId && customerEmail) {
    userProfile = await fetchUserProfileByEmail(customerEmail);
    userId = userProfile?.userId || null;
  } else if (userId) {
    userProfile = await fetchUserProfileByUserId(userId);
  }

  // 4. Fetch membership plan
  const plan = await fetchMembershipPlanById(parseInt(membershipPlanId, 10));

  // 5. CRITICAL: Check for existing subscription (filter out CANCELLED/EXPIRED)
  let existingSubscription = await findSubscriptionByPaymentIntentId(paymentIntentId);

  // CRITICAL: Only accept ACTIVE or TRIAL subscriptions - ignore CANCELLED ones
  if (existingSubscription && (existingSubscription.subscriptionStatus === 'CANCELLED' || existingSubscription.subscriptionStatus === 'EXPIRED')) {
    console.log('[MEMBERSHIP-SUCCESS] Found cancelled/expired subscription - will create new one');
    existingSubscription = null; // Reset to null so we create a new subscription
  }

  if (existingSubscription && (existingSubscription.subscriptionStatus === 'ACTIVE' || existingSubscription.subscriptionStatus === 'TRIAL')) {
    return { subscription: existingSubscription, plan, userProfile };
  }

  // 6. CRITICAL: Get or create Stripe Customer
  let stripeCustomerId: string | null = null;

  if (paymentIntent.customer) {
    // Customer already attached to Payment Intent
    stripeCustomerId = typeof paymentIntent.customer === 'string'
      ? paymentIntent.customer
      : (paymentIntent.customer as any)?.id || null;
  } else {
    // No customer attached - create/get customer from email
    const existingCustomers = await stripe().customers.list({
      email: customerEmail,
      limit: 1,
    });

    if (existingCustomers.data.length > 0) {
      stripeCustomerId = existingCustomers.data[0].id;
    } else {
      // Create new customer
      const newCustomer = await stripe().customers.create({
        email: customerEmail,
        name: userProfile.firstName && userProfile.lastName
          ? `${userProfile.firstName} ${userProfile.lastName}`
          : undefined,
        phone: userProfile.phone || undefined,
        metadata: {
          userId: userId,
          tenantId: tenantId,
          userProfileId: String(userProfile.id),
        },
      });
      stripeCustomerId = newCustomer.id;
    }
  }

  // 7. CRITICAL: Create Stripe Subscription for recurring billing
  let stripeSubscriptionId: string | null = null;
  let finalStripePriceId: string | null = plan.stripePriceId || null;

  // CRITICAL: If plan doesn't have stripePriceId, create/get Stripe Price on the fly
  if (!finalStripePriceId && stripeCustomerId) {
    // Determine Stripe Price interval based on billing interval
    let priceInterval: 'month' | 'year' = 'month';
    if (plan.billingInterval === 'YEARLY') {
      priceInterval = 'year';
    } else if (plan.billingInterval === 'QUARTERLY') {
      priceInterval = 'month'; // Use monthly with interval_count: 3
    }

    // Get or create Stripe Product
    let stripeProductId = plan.stripeProductId;
    if (!stripeProductId) {
      const stripeProduct = await stripe().products.create({
        name: plan.planName || `Membership Plan ${plan.id}`,
        description: plan.description || undefined,
        metadata: {
          membershipPlanId: String(membershipPlanId),
          tenantId: tenantId,
        },
      });
      stripeProductId = stripeProduct.id;
    }

    // Create Stripe Price
    const stripePrice = await stripe().prices.create({
      product: stripeProductId,
      unit_amount: Math.round((plan.price || 0) * 100), // Convert to cents
      currency: (plan.currency || 'USD').toLowerCase(),
      recurring: {
        interval: priceInterval,
        ...(plan.billingInterval === 'QUARTERLY' ? { interval_count: 3 } : {}),
      },
      metadata: {
        membershipPlanId: String(membershipPlanId),
        tenantId: tenantId,
      },
    });
    finalStripePriceId = stripePrice.id;
  }

  // 8. CRITICAL: Create Stripe Subscription with default_incomplete behavior
  if (stripeCustomerId && finalStripePriceId) {
    const subscriptionParams: any = {
      customer: stripeCustomerId,
      items: [{ price: finalStripePriceId }],
      metadata: {
        membershipPlanId: String(membershipPlanId),
        tenantId: tenantId,
        userId: userId,
        userProfileId: String(userProfile.id),
        paymentIntentId: paymentIntentId,
      },
      // CRITICAL: Use default_incomplete to allow subscription creation without payment method
      // The subscription will be created but may require payment method setup for future invoices
      payment_behavior: 'default_incomplete',
      payment_settings: {
        save_default_payment_method: 'on_subscription',
      },
      expand: ['latest_invoice.payment_intent'],
    };

    // Add trial period if applicable
    if (plan.trialDays && plan.trialDays > 0) {
      subscriptionParams.trial_period_days = plan.trialDays;
    }

    const stripeSubscription = await stripe().subscriptions.create(subscriptionParams);
    stripeSubscriptionId = stripeSubscription.id;

    // Update current period dates from Stripe Subscription (more accurate than calculated dates)
    if (stripeSubscription.current_period_start) {
      currentPeriodStart = new Date(stripeSubscription.current_period_start * 1000).toISOString();
    }
    if (stripeSubscription.current_period_end) {
      currentPeriodEnd = new Date(stripeSubscription.current_period_end * 1000).toISOString();
    }
  }

  // 9. Create subscription payload with all Stripe IDs
  const subscriptionPayload = withTenantId({
    userProfileId: userProfile.id,
    membershipPlanId: plan.id!,
    subscriptionStatus: plan.trialDays && plan.trialDays > 0 ? 'TRIAL' : 'ACTIVE',
    currentPeriodStart,
    currentPeriodEnd,
    trialStart,
    trialEnd,
    cancelAtPeriodEnd: false,
    stripeSubscriptionId, // CRITICAL: Store Stripe Subscription ID
    stripeCustomerId, // CRITICAL: Store Stripe Customer ID
    stripePaymentIntentId: paymentIntentId, // CRITICAL: Store Payment Intent ID for tracking
    createdAt: new Date().toISOString(),
    updatedAt: new Date().toISOString(),
  });

  // 10. Create subscription via backend API
  const createdSubscription = await fetchWithJwtRetry(
    `${baseUrl}/api/proxy/membership-subscriptions`,
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(subscriptionPayload),
      cache: 'no-store',
    }
  );

  return { subscription: await createdSubscription.json(), plan, userProfile };
}
```

### Key Points for Membership Subscription Flow

1. **CRITICAL: Filter Out CANCELLED/EXPIRED Subscriptions**
   - When looking up existing subscriptions, filter out `CANCELLED` or `EXPIRED` subscriptions
   - If only cancelled/expired subscriptions are found, return `null` to force creation of new subscription
   - This prevents returning old cancelled subscriptions when a new payment is made

2. **CRITICAL: Stripe Customer Creation/Retrieval**
   - Check if Payment Intent has customer attached
   - If not, search for existing customer by email
   - If not found, create new Stripe Customer with user profile metadata
   - Store `stripeCustomerId` in database subscription record

3. **CRITICAL: Stripe Subscription Creation**
   - Create Stripe Subscription for recurring billing (not just one-time Payment Intent)
   - Use `payment_behavior: 'default_incomplete'` to allow creation without immediate payment method
   - Use `payment_settings: { save_default_payment_method: 'on_subscription' }` to save payment method when available
   - Store `stripeSubscriptionId` in database subscription record

4. **CRITICAL: Stripe Product/Price Creation on the Fly**
   - If plan doesn't have `stripePriceId`, create Stripe Product and Price automatically
   - Map billing interval to Stripe Price interval:
     - `MONTHLY` → `interval: 'month'`
     - `QUARTERLY` → `interval: 'month', interval_count: 3`
     - `YEARLY` → `interval: 'year'`
   - Store created `stripeProductId` and `stripePriceId` for future use

5. **CRITICAL: Store All Stripe IDs**
   - `stripeSubscriptionId`: For recurring billing and renewal processing
   - `stripeCustomerId`: For customer management and payment method storage
   - `stripePaymentIntentId`: For tracking the initial payment that created the subscription

6. **CRITICAL: Payment Intent Status Check**
   - Accept both `succeeded` and `requires_capture` statuses
   - `requires_capture` indicates payment succeeded but needs manual capture (still valid)

7. **CRITICAL: User Profile Lookup**
   - If `userId` not provided (public routes), look up user profile by email
   - Extract email from Payment Intent metadata or `receipt_email`
   - This allows processing subscriptions from public routes without requiring authentication

### Membership Subscription Success Page Handling

**Location**: `src/app/membership/success/page.tsx` and `src/app/api/membership/success/process/route.ts`

**Desktop Flow**:
```typescript
// GET handler processes Payment Intent and creates subscription
export async function GET(req: NextRequest) {
  const { searchParams } = new URL(req.url);
  const pi = searchParams.get('pi');

  if (pi && pi.startsWith('pi_')) {
    // Process Payment Intent and create subscription
    const result = await processMembershipSubscriptionFromPaymentIntent(pi, undefined);

    if (result?.subscription) {
      return NextResponse.json({
        subscription: result.subscription,
        plan: result.plan,
        userProfile: result.userProfile,
        success: true,
      });
    }
  }

  return NextResponse.json({ error: 'Subscription not found' }, { status: 404 });
}
```

**Key Points**:
- Desktop success page calls `processMembershipSubscriptionFromPaymentIntent` with Payment Intent ID
- Function handles all Stripe Customer, Product, Price, and Subscription creation
- Returns subscription with all Stripe IDs populated
- Success page displays subscription details and confirmation

## Desktop Success Page Flow

### SuccessClient Component (`src/app/event/success/SuccessClient.tsx`)

**Desktop Flow Behavior**:
```typescript
// 1. Mobile detection (happens immediately on mount)
const [isMobileDevice, setIsMobileDevice] = useState<boolean | null>(null);

useEffect(() => {
  const isMobile = /* detection logic */;
  setIsMobileDevice(isMobile);

  if (isMobile) {
    // Mobile: Redirect to /event/ticket-qr
    setTimeout(() => router.replace('/event/ticket-qr?pi=...'), 2000);
    return; // Exit early for mobile
  }

  // Desktop: Continue with success page
}, []);

// 2. Desktop data fetching (only runs if desktop detected)
useEffect(() => {
  if (isMobileDevice === null) return; // Wait for detection
  if (isMobileDevice === true) return; // Skip for mobile

  // Desktop: Fetch transaction data
  async function fetchData() {
    const getUrl = `/api/event/success/process?pi=${payment_intent}`;
    const response = await fetch(getUrl);
    const data = await response.json();

    if (data.transaction) {
      setResult(data); // Display success page with QR
    } else {
      // Start polling if transaction not found
      startPolling();
    }
  }

  fetchData();
}, [isMobileDevice, payment_intent]);
```

**Desktop Polling Logic**:
```typescript
// Desktop polls GET endpoint up to 10 times with 3-second intervals
const MAX_POLL_ATTEMPTS = 10;
const POLL_INTERVAL_MS = 3000;

for (let pollAttempt = 1; pollAttempt <= MAX_POLL_ATTEMPTS; pollAttempt++) {
  const response = await fetch(`/api/event/success/process?pi=${pi}&_poll=${pollAttempt}`);
  const data = await response.json();

  if (data.transaction) {
    setResult(data); // Transaction found - display success page
    break; // Exit polling loop
  }

  // Wait before next poll
  await new Promise(resolve => setTimeout(resolve, POLL_INTERVAL_MS));
}

// After polling completes, show error if transaction still not found
if (!data.transaction) {
  setError('Transaction is still being processed. Please refresh or check your email.');
}
```

**Key Points**:
- Desktop stays on `/event/success` page (no redirect)
- Desktop polls GET endpoint (not POST)
- Desktop creates transaction via GET endpoint fallback
- Desktop displays QR code inline on success page

## API Endpoint Details

### GET `/api/event/success/process` (Desktop Flow)

**Purpose**: Lookup existing transaction or create transaction if not found (desktop only)

**Query Parameters**:
- `session_id` (optional): Stripe Checkout Session ID
- `pi` (optional): Payment Intent ID
- `_poll` (optional): Polling attempt number (for logging)

**Process**:
1. Detect if request is from desktop browser
2. Lookup transaction by `session_id` or `pi`
3. If not found AND desktop AND `pi` provided:
   - Retrieve Payment Intent from Stripe
   - Validate payment succeeded
   - Validate tenant ID and payment method domain ID
   - Create transaction via `createTransactionFromPaymentIntent`
   - Create transaction items via `createTransactionItemsBulk`
4. Return transaction data with QR code

**Response**:
```typescript
{
  transaction: EventTicketTransactionDTO | null,
  eventDetails: EventDetailsDTO | null,
  qrCodeData: { qrCodeImageUrl: string } | null,
  transactionItems: EventTicketTransactionItemDTO[],
  heroImageUrl: string | null,
  hasTransaction: boolean,
  error?: string,
  message?: string
}
```

### POST `/api/event/success/process` (Mobile Flow - Unchanged)

**Purpose**: Mobile flow transaction creation (via `/event/ticket-qr` page)

**CRITICAL**: This endpoint remains unchanged and is used only by mobile workflow.

**Body Parameters**:
- `session_id` (optional): Stripe Checkout Session ID
- `pi` (optional): Payment Intent ID
- `skip_qr` (optional): Skip QR code generation (prevents duplicate emails)

**Process**:
1. Detect if request is from mobile browser
2. Lookup existing transaction
3. If not found, create transaction via `createTransactionFromPaymentIntent`
4. Return transaction data (QR code skipped if `skip_qr=true`)

## Transaction Lookup with Tenant ID and Payment Method Domain ID

### CRITICAL: Both Must Be Included in Lookup Queries

**Function**: `findTransactionByPaymentIntentId`

**Location**: `src/app/event/success/ApiServerActions.ts`

**Implementation**:
```typescript
export async function findTransactionByPaymentIntentId(
  paymentIntentId: string,
): Promise<EventTicketTransactionDTO | null> {
  const tenantId = getTenantId();
  const paymentMethodDomainId = getPaymentMethodDomainId();

  // CRITICAL: Query with both tenantId and paymentMethodDomainId
  const tenantParams = new URLSearchParams({
    'stripePaymentIntentId.equals': paymentIntentId,
    'tenantId.equals': tenantId,
    'paymentMethodDomainId.equals': paymentMethodDomainId, // CRITICAL: Backend requires both
  });

  const tenantUrl = `${getAppUrl()}/api/proxy/event-ticket-transactions?${tenantParams.toString()}`;
  const tenantResponse = await fetchWithJwtRetry(tenantUrl);

  if (tenantResponse.ok) {
    const tenantItems: EventTicketTransactionDTO[] = await tenantResponse.json();
    if (tenantItems.length > 0) {
      return tenantItems[0]; // Found with tenant and payment method domain filter
    }
  }

  // Fallback: Check without tenant filter (for cross-tenant duplicate detection)
  const globalParams = new URLSearchParams({
    'stripePaymentIntentId.equals': paymentIntentId,
    'paymentMethodDomainId.equals': paymentMethodDomainId, // Still include payment method domain ID
  });

  const globalResponse = await fetchWithJwtRetry(
    `${getAppUrl()}/api/proxy/event-ticket-transactions?${globalParams.toString()}`
  );

  if (globalResponse.ok) {
    const globalItems: EventTicketTransactionDTO[] = await globalResponse.json();
    if (globalItems.length > 0) {
      // Found for different tenant - return to prevent duplicate creation
      return globalItems[0];
    }
  }

  return null; // Not found
}
```

**Key Points**:
- Always includes both `tenantId.equals` and `paymentMethodDomainId.equals` in queries
- Uses `X-Tenant-ID` header (via `fetchWithJwtRetry`)
- Falls back to global check (without tenant filter) for cross-tenant duplicate detection
- Returns existing transaction if found (prevents duplicates)

## Proxy Handler Configuration

### X-Tenant-ID Header Injection

**Location**: `src/lib/proxyHandler.ts`

**CRITICAL**: All backend API calls must include `X-Tenant-ID` header for backend `TenantContextFilter`.

**Implementation**:
```typescript
export async function fetchWithJwtRetry(apiUrl: string, options: any = {}, debugLabel = '') {
  const tenantId = getTenantId();

  let response = await fetch(apiUrl, {
    ...options,
    headers: {
      ...options.headers,
      Authorization: `Bearer ${token}`,
      'X-Tenant-ID': tenantId, // CRITICAL: Backend TenantContextFilter expects this header
    },
  });

  // Retry logic with same header...
  return response;
}
```

**Why This Is Required**:
- Backend `TenantContextFilter` extracts tenant ID from `X-Tenant-ID` header
- Without this header, backend defaults to `tenant_demo_001` (wrong tenant)
- Causes transaction lookup failures (wrong tenant filter)
- Causes transaction creation failures (wrong tenant context)

## Desktop Flow Transaction Items Verification

### CRITICAL: Verify Transaction Items Are Created

**Function**: `createTransactionFromPaymentIntent` MUST create transaction items after creating transaction.

**Code Location**: `src/app/event/success/ApiServerActions.ts` lines 1372-1478

**Verification Steps**:
1. Check that `createTransactionFromPaymentIntent` calls `createTransactionItemsBulk` (line 1468)
2. Verify transaction items are created with correct `tenantId` and `paymentMethodDomainId`
3. Verify transaction items include `transactionId` linking to parent transaction
4. Verify transaction items are created in bulk (single API call)

**Current Implementation**:
```typescript
// In createTransactionFromPaymentIntent (line 1372-1478):
// 1. Create transaction first
const transaction = await createTransaction(transactionData);

// 2. Build transaction items array (lines 1373-1396)
const transactionItems = [];
for (const cartItem of cart) {
  const ticketType = await fetchTicketTypeByIdServer(cartItem.ticketTypeId);
  const itemData = withTenantId({
    transactionId: transaction.id, // CRITICAL: Link to parent transaction
    ticketTypeId: cartItem.ticketTypeId,
    quantity: cartItem.quantity,
    pricePerUnit: ticketType.price,
    totalAmount: ticketType.price * cartItem.quantity,
    paymentMethodDomainId: expectedPaymentMethodDomainId, // CRITICAL: Include payment method domain ID
  });
  transactionItems.push(itemData);
}

// 3. Check for existing items (idempotency) (lines 1402-1463)
const existingItems = await checkExistingItems(transaction.id);
const itemsToCreate = transactionItems.filter(item => {
  return !existingItems.find(existing =>
    existing.ticketTypeId === item.ticketTypeId &&
    existing.transactionId === transaction.id
  );
});

// 4. Create transaction items in bulk (line 1468)
if (itemsToCreate.length > 0) {
  await createTransactionItemsBulk(itemsToCreate); // CRITICAL: Must be called
  console.log('[createTransactionFromPaymentIntent] Successfully created transaction items:', itemsToCreate.length);
}
```

**Verification Checklist**:
- [ ] `createTransactionFromPaymentIntent` is called from desktop GET handler
- [ ] Transaction is created successfully (check `transaction.id` exists)
- [ ] `transactionItems` array is built correctly (check `transactionItems.length > 0`)
- [ ] `itemsToCreate` array has items to create (check `itemsToCreate.length > 0`)
- [ ] `createTransactionItemsBulk` is called (check logs for "Successfully created transaction items")
- [ ] Backend API returns success for bulk creation (check response status)
- [ ] Transaction items appear in database (check `event_ticket_transaction_item` table)

**If Transaction Items Are Missing**:
1. Check console logs for `[createTransactionFromPaymentIntent]` messages
2. Verify `itemsToCreate.length > 0` (items exist to create)
3. Check if `createTransactionItemsBulk` is throwing errors
4. Verify backend API endpoint `/api/proxy/event-ticket-transaction-items/bulk` is accessible
5. Check backend logs for transaction items creation errors
6. Verify `transactionId` is correctly set in item data
7. Verify `tenantId` and `paymentMethodDomainId` are included in item payload

**Code Verification**:
```typescript
// In createTransactionFromPaymentIntent:
// 1. Create transaction
const transaction = await createTransaction(transactionData);

// 2. Build transaction items array
const transactionItems = [];
for (const cartItem of cart) {
  const ticketType = await fetchTicketTypeByIdServer(cartItem.ticketTypeId);
  const itemData = withTenantId({
    transactionId: transaction.id, // CRITICAL: Link to parent transaction
    ticketTypeId: cartItem.ticketTypeId,
    quantity: cartItem.quantity,
    pricePerUnit: ticketType.price,
    totalAmount: ticketType.price * cartItem.quantity,
    paymentMethodDomainId: expectedPaymentMethodDomainId, // CRITICAL: Include payment method domain ID
  });
  transactionItems.push(itemData);
}

// 3. Check for existing items (idempotency)
const existingItems = await checkExistingItems(transaction.id);
const itemsToCreate = transactionItems.filter(item => {
  return !existingItems.find(existing =>
    existing.ticketTypeId === item.ticketTypeId &&
    existing.transactionId === transaction.id
  );
});

// 4. Create transaction items in bulk
if (itemsToCreate.length > 0) {
  await createTransactionItemsBulk(itemsToCreate); // CRITICAL: Must be called
  console.log('[DESKTOP FLOW] ✅ Transaction items created:', itemsToCreate.length);
}
```

**If Transaction Items Are Missing**:
- Check that `createTransactionItemsBulk` is being called
- Verify `itemsToCreate.length > 0` (items exist to create)
- Check backend API response for transaction items creation
- Verify `transactionId` is correctly set in item data
- Check backend logs for transaction items creation errors

## Desktop Flow Error Handling

### Common Errors and Solutions

#### 1. "Transaction not found" After Polling

**Cause**: Transaction not created by webhook or desktop fallback

**Solution**:
- Verify desktop flow is calling `createTransactionFromPaymentIntent`
- Check that `paymentIntent.status === 'succeeded'`
- Verify tenant ID and payment method domain ID are correct
- Check backend logs for transaction creation errors

#### 2. "Tenant ID mismatch" Error

**Cause**: Payment Intent metadata tenant ID doesn't match environment variable

**Solution**:
- Verify `NEXT_PUBLIC_TENANT_ID` environment variable is set correctly
- Check Payment Intent metadata includes correct `tenantId`
- Verify `X-Tenant-ID` header is being sent (via `fetchWithJwtRetry`)

#### 3. "Payment Method Domain ID mismatch" Error

**Cause**: Payment Intent metadata payment method domain ID doesn't match environment variable

**Solution**:
- Verify `NEXT_PUBLIC_PAYMENT_METHOD_DOMAIN_ID` environment variable is set correctly
- Check Payment Intent metadata includes correct `paymentMethodDomainId`
- Verify both `tenantId` and `paymentMethodDomainId` are passed together

#### 4. "Transaction items not found" Error

**Cause**: Transaction items not created after transaction creation

**Solution**:
- Verify `createTransactionItemsBulk` is being called
- Check that `itemsToCreate.length > 0` (items exist to create)
- Verify transaction items include `transactionId` linking to parent transaction
- Check backend API response for transaction items creation

## Best Practices for Desktop Implementation

### 1. Always Include Tenant ID and Payment Method Domain ID

- **CRITICAL**: Always pass both `tenantId` and `paymentMethodDomainId` together
- Use environment variables (not metadata) for consistency
- Validate metadata matches environment variables before creation
- Include `X-Tenant-ID` header in all backend requests

### 2. Implement Proper Desktop Detection

- Use multiple detection methods (user agent + screen width)
- Desktop detection is inverse of mobile detection
- Desktop browsers with narrow windows are NOT mobile
- Test on actual desktop browsers, not just browser dev tools

### 3. Handle Transaction Creation Properly

- **ALWAYS** check for existing transaction before creating
- Use `findTransactionByPaymentIntentId()` with both tenant ID and payment method domain ID filters
- Create transaction items after transaction creation
- Verify transaction items are created successfully

### 4. Prevent Duplicate Transactions

- Check for existing transaction before creating
- Check for existing transaction items before creating
- Use idempotency checks at both transaction and item levels
- Handle webhook retries gracefully (skip if transaction exists)

### 5. Optimize for Desktop UX

- Show immediate feedback for payment actions
- Poll GET endpoint if transaction not found (up to 10 attempts)
- Create transaction immediately if webhook delayed
- Display QR code inline on success page

### 6. Debug Desktop Flows Thoroughly

- Test with actual desktop browsers
- Verify transaction creation happens immediately
- Check transaction items are created
- Validate tenant ID and payment method domain ID are correct
- Verify `X-Tenant-ID` header is being sent
- Check backend logs for transaction and item creation

## Security Considerations

### 1. Payment Validation

- All amounts validated server-side against backend prices
- Idempotency keys prevent duplicate payment processing
- JWT authentication for backend API calls
- Tenant ID isolation for multi-tenant security

### 2. Desktop-Specific Security

- HTTPS required for Stripe Checkout
- Payment Intent validation before transaction creation
- Tenant ID and payment method domain ID validation
- XSS prevention in desktop redirect flows
- CSRF protection on API endpoints

### 3. Triple Validation

- Backend validates combination (`tenantId`, `paymentMethodDomainId`, `webhookSecret`)
- Frontend must pass both `tenantId` and `paymentMethodDomainId` together
- Metadata validation ensures correct tenant and payment method domain
- Environment variable validation prevents misconfiguration

## Monitoring and Debugging

### 1. Enhanced Logging

All desktop flows include comprehensive logging:
```typescript
console.log('[DESKTOP FLOW]', {
  userAgent: navigator.userAgent,
  windowWidth: window.innerWidth,
  paymentMethod: 'Checkout',
  identifier: session_id || payment_intent,
  timestamp: new Date().toISOString()
});
```

### 2. Error Tracking

- Specific desktop error codes and messages
- Payment failure reason tracking
- Transaction creation failure logging
- Transaction items creation failure logging
- Performance metric collection

### 3. CloudWatch Logging

- Desktop detection results
- Transaction lookup attempts
- Transaction creation attempts
- Transaction items creation attempts
- Tenant ID and payment method domain ID validation results

## Critical Desktop Flow Rules

### 1. Desktop Flow Must Be Separate from Mobile

- **CRITICAL**: Desktop flow uses GET endpoint with transaction creation fallback
- **CRITICAL**: Mobile flow uses POST endpoint (via `/event/ticket-qr` page)
- **CRITICAL**: Mobile workflow files remain untouched (`/event/ticket-qr` page and related components)
- **CRITICAL**: Both flows use same `createTransactionFromPaymentIntent` function but different entry points

### 2. Desktop Flow Must Create Transaction Items

- **CRITICAL**: Desktop flow MUST create transaction items after creating transaction
- **CRITICAL**: Transaction items MUST include `tenantId` and `paymentMethodDomainId`
- **CRITICAL**: Transaction items MUST include `transactionId` linking to parent transaction
- **CRITICAL**: Transaction items MUST be created in bulk (single API call)

### 3. Desktop Flow Must Include Tenant ID and Payment Method Domain ID

- **CRITICAL**: All backend requests MUST include `X-Tenant-ID` header
- **CRITICAL**: All transaction lookup queries MUST include both `tenantId.equals` and `paymentMethodDomainId.equals`
- **CRITICAL**: All transaction creation payloads MUST include both `tenantId` and `paymentMethodDomainId`
- **CRITICAL**: All transaction items creation payloads MUST include both `tenantId` and `paymentMethodDomainId`

### 4. Desktop Flow Must Handle Webhook Delays

- **CRITICAL**: Desktop flow creates transaction immediately if webhook delayed
- **CRITICAL**: Desktop flow polls GET endpoint if transaction not found
- **CRITICAL**: Desktop flow validates payment succeeded before creating transaction
- **CRITICAL**: Desktop flow checks for existing transaction before creating (idempotency)

## Files Affected by Desktop Flow

### Desktop Flow Files (Modified)
- `src/app/api/event/success/process/route.ts` - GET handler with desktop transaction creation
- `src/app/event/success/SuccessClient.tsx` - Desktop detection and polling logic
- `src/app/event/success/ApiServerActions.ts` - Transaction and transaction items creation
- `src/lib/proxyHandler.ts` - X-Tenant-ID header injection

### Mobile Flow Files (Unchanged)
- `src/app/event/ticket-qr/page.tsx` - Mobile QR page (unchanged)
- `src/app/event/ticket-qr/TicketQrClient.tsx` - Mobile QR client (unchanged)
- `src/app/api/event/success/process/route.ts` - POST handler (unchanged, used by mobile)

### Shared Files (Used by Both)
- `src/app/event/success/ApiServerActions.ts` - `createTransactionFromPaymentIntent` function (shared)
- `src/lib/env.ts` - Environment variable helpers (shared)
- `src/lib/proxyHandler.ts` - Proxy handler with tenant ID header (shared)

## Lessons Learned and Critical Fixes

### 1. X-Tenant-ID Header Missing Fix

**Issue**: Backend `TenantContextFilter` was not finding tenant ID in requests, defaulting to `tenant_demo_001`.

**Root Cause**: `fetchWithJwtRetry` was not including `X-Tenant-ID` header.

**Fix**: Added `X-Tenant-ID` header to all backend requests:
```typescript
headers: {
  'Authorization': `Bearer ${token}`,
  'X-Tenant-ID': tenantId, // CRITICAL: Backend TenantContextFilter expects this header
}
```

**Lesson**: All backend API calls must include `X-Tenant-ID` header for backend tenant context filtering.

### 2. Payment Method Domain ID Missing in Lookup Queries

**Issue**: Transaction lookup queries were not including `paymentMethodDomainId.equals` parameter.

**Root Cause**: `findTransactionByPaymentIntentId` was only filtering by `tenantId.equals`, not `paymentMethodDomainId.equals`.

**Fix**: Added `paymentMethodDomainId.equals` to all transaction lookup queries:
```typescript
const params = new URLSearchParams({
  'stripePaymentIntentId.equals': paymentIntentId,
  'tenantId.equals': tenantId,
  'paymentMethodDomainId.equals': paymentMethodDomainId, // CRITICAL: Backend requires both
});
```

**Lesson**: Backend requires both `tenantId` and `paymentMethodDomainId` together for transaction lookups.

### 3. Desktop Flow Transaction Creation Missing

**Issue**: Desktop flow was polling GET endpoint but never creating transactions, resulting in "Transaction not found" errors.

**Root Cause**: GET handler was only looking up transactions, not creating them for desktop flow.

**Fix**: Added desktop transaction creation fallback in GET handler:
```typescript
if (!transaction && !isMobile && pi) {
  const paymentIntent = await stripe.paymentIntents.retrieve(pi);
  if (paymentIntent.status === 'succeeded') {
    const createdTransaction = await createTransactionFromPaymentIntent(...);
    transaction = createdTransaction;
  }
}
```

**Lesson**: Desktop flow must create transactions immediately if webhook delayed, using GET endpoint fallback.

### 4. Transaction Items Creation Verification

**Issue**: User suspected transaction items were not being created in desktop flow.

**Root Cause**: Need to verify `createTransactionItemsBulk` is being called.

**Verification**: `createTransactionFromPaymentIntent` function (used by desktop flow) DOES call `createTransactionItemsBulk` at line 1468. Transaction items should be created.

**If Transaction Items Are Still Missing**:
- Check that `itemsToCreate.length > 0` (items exist to create)
- Verify `createTransactionItemsBulk` is not throwing errors
- Check backend API response for transaction items creation
- Verify `transactionId` is correctly set in item data

**Lesson**: Always verify transaction items are created after transaction creation. Use logging to track item creation process.

### 5. Membership Subscription Stripe Subscription ID Missing Fix

**Issue**: Database subscription records were missing `stripeSubscriptionId` and `stripeCustomerId` fields, even though payment succeeded.

**Root Cause**: Payment Intents are one-time payments and don't automatically create Stripe Subscriptions for recurring billing. The system was only processing the Payment Intent but not creating the Stripe Subscription needed for recurring charges.

**Fix**: Added logic to create Stripe Subscription during Payment Intent processing:
```typescript
// Create Stripe Subscription with default_incomplete behavior
const stripeSubscription = await stripe().subscriptions.create({
  customer: stripeCustomerId,
  items: [{ price: finalStripePriceId }],
  payment_behavior: 'default_incomplete', // Allows creation without immediate payment method
  payment_settings: {
    save_default_payment_method: 'on_subscription',
  },
  metadata: {
    membershipPlanId: String(membershipPlanId),
    tenantId: tenantId,
    userId: userId,
    userProfileId: String(userProfile.id),
    paymentIntentId: paymentIntentId,
  },
});

// Store Stripe Subscription ID in database
stripeSubscriptionId = stripeSubscription.id;
```

**Lesson**: For membership subscriptions, always create a Stripe Subscription object for recurring billing, even if payment method cannot be attached immediately. Use `payment_behavior: 'default_incomplete'` to allow subscription creation.

### 6. Membership Subscription CANCELLED Filter Fix

**Issue**: When a user made a new payment after cancelling a subscription, the system was returning the old cancelled subscription instead of creating a new one.

**Root Cause**: `findSubscriptionByPaymentIntentId` was returning cancelled subscriptions without filtering them out.

**Fix**: Filter out CANCELLED and EXPIRED subscriptions when looking up existing subscriptions:
```typescript
// CRITICAL: Filter out cancelled/expired subscriptions
const activeSubscriptions = items.filter(sub =>
  sub.subscriptionStatus !== 'CANCELLED' && sub.subscriptionStatus !== 'EXPIRED'
);

if (activeSubscriptions.length === 0) {
  return null; // Force creation of new subscription
}

// Only return ACTIVE or TRIAL subscriptions
if (existingSubscription && (existingSubscription.subscriptionStatus === 'CANCELLED' || existingSubscription.subscriptionStatus === 'EXPIRED')) {
  existingSubscription = null; // Reset to null so we create a new subscription
}
```

**Lesson**: Always filter out CANCELLED and EXPIRED subscriptions when looking up existing subscriptions. If only cancelled subscriptions are found, return `null` to force creation of a new subscription.

### 7. Membership Subscription Stripe Product/Price Creation Fix

**Issue**: If a membership plan didn't have `stripePriceId` configured, subscription creation would fail.

**Root Cause**: System required `stripePriceId` to create Stripe Subscription, but some plans didn't have it configured.

**Fix**: Create Stripe Product and Price on the fly if missing:
```typescript
// CRITICAL: If plan doesn't have stripePriceId, create/get Stripe Price on the fly
if (!finalStripePriceId && stripeCustomerId) {
  // Get or create Stripe Product
  let stripeProductId = plan.stripeProductId;
  if (!stripeProductId) {
    const stripeProduct = await stripe().products.create({
      name: plan.planName || `Membership Plan ${plan.id}`,
      description: plan.description || undefined,
      metadata: {
        membershipPlanId: String(membershipPlanId),
        tenantId: tenantId,
      },
    });
    stripeProductId = stripeProduct.id;
  }

  // Create Stripe Price
  const stripePrice = await stripe().prices.create({
    product: stripeProductId,
    unit_amount: Math.round((plan.price || 0) * 100),
    currency: (plan.currency || 'USD').toLowerCase(),
    recurring: {
      interval: priceInterval,
      ...(plan.billingInterval === 'QUARTERLY' ? { interval_count: 3 } : {}),
    },
  });
  finalStripePriceId = stripePrice.id;
}
```

**Lesson**: Always handle missing Stripe Product/Price configuration by creating them on the fly. Map billing intervals correctly (QUARTERLY uses `interval: 'month'` with `interval_count: 3`).

### 8. Membership Subscription Payment Intent Status Check Fix

**Issue**: Payment Intents in `requires_capture` status were being rejected, even though payment succeeded.

**Root Cause**: Status check only accepted `succeeded` status, but Payment Intents can also be in `requires_capture` status after successful payment.

**Fix**: Accept both `succeeded` and `requires_capture` statuses:
```typescript
// Check if payment succeeded or requires capture (both indicate successful payment)
if (paymentIntent.status !== 'succeeded' && paymentIntent.status !== 'requires_capture') {
  return null;
}
```

**Lesson**: Payment Intents can be in `requires_capture` status after successful payment (requires manual capture). Always accept both `succeeded` and `requires_capture` statuses for subscription processing.

### 9. Membership Subscription User Profile Lookup Fix

**Issue**: Public routes couldn't process subscriptions because `userId` wasn't available.

**Root Cause**: `processMembershipSubscriptionFromPaymentIntent` required `userId` parameter, but public routes don't have authenticated user context.

**Fix**: Look up user profile by email if `userId` not provided:
```typescript
// CRITICAL: Get userId from email if not provided (for public routes)
let finalUserId = userId;
let userProfile: UserProfileDTO | null = null;

if (!finalUserId && customerEmail) {
  userProfile = await fetchUserProfileByEmail(customerEmail);
  if (userProfile?.userId) {
    finalUserId = userProfile.userId;
  }
} else if (finalUserId) {
  userProfile = await fetchUserProfileByUserId(finalUserId);
}
```

**Lesson**: Always support email-based user profile lookup for public routes. Extract email from Payment Intent metadata or `receipt_email` field.

### 10. Payment Method Attachment Fix - CRITICAL for Subscription Functionality

**Issue**: When a user purchases a membership subscription, the payment method from the confirmed Payment Intent cannot be reused for the subscription because it was used **without being attached to the customer first**. This results in:
- Subscription status `INCOMPLETE` instead of `ACTIVE`
- Error: "This PaymentMethod was previously used without being attached to a Customer or was detached from a Customer, and may not be used again."
- Missing `stripeSubscriptionId` in database (subscription creation fails)

**Root Cause**:
1. **Payment Intent Creation**: Payment Intent was created **without** a `customer` parameter, so the payment method wasn't associated with a customer
2. **Payment Confirmation**: When payment is confirmed, the payment method is used but **not attached** to a customer
3. **Stripe Security Model**: Payment methods used without attachment are marked as "one-time use" and cannot be reused for subscriptions
4. **Subscription Creation**: When trying to create a subscription with the payment method, Stripe rejects it because it was already used without attachment

**Why Attachment is Mandatory**:
- **Stripe's Security Model**: Payment methods used without attachment are considered "one-time use" to prevent unauthorized reuse
- **Subscription Requirements**: Subscriptions need a **reusable** payment method for recurring billing
- **Best Practices**: Stripe recommends attaching payment methods to customers for:
  - Recurring billing
  - Payment method management in customer portal
  - Better fraud detection
  - PCI compliance benefits

**Why We Haven't Done It Yet**:
1. **Original Design**: Payment flow was designed for one-time payments (event tickets), not subscriptions
2. **Customer Creation Timing**: Customer was created after payment confirmation, not before
3. **Payment Element Limitation**: Payment Element doesn't expose payment method until after confirmation

**Production Safety**:
- ✅ **YES - It's Safe and Recommended**: Attaching payment methods is a standard Stripe operation with no additional security risks
- ✅ **User Experience**: Users expect their payment method to be saved for subscriptions
- ✅ **Compliance**: PCI DSS compliant (Stripe handles all sensitive data)
- ✅ **No Side Effects**: Doesn't affect one-time payments, doesn't create duplicate charges

**Solution Implemented**: Create Payment Intent with Customer Parameter

**Implementation** (`src/app/api/stripe/membership-payment-intent/route.ts`):
```typescript
// CRITICAL: Create or retrieve customer BEFORE creating Payment Intent
// This ensures payment method is automatically attached when payment is confirmed
let customerId: string | undefined;
if (email) {
  // Search for existing customer by email
  const existingCustomers = await stripe().customers.list({
    email: email,
    limit: 1,
  });

  if (existingCustomers.data.length > 0) {
    customerId = existingCustomers.data[0].id;
  } else {
    // Create new customer if doesn't exist
    const newCustomer = await stripe().customers.create({
      email: email,
      name: customerName,
      phone: customerPhone,
      metadata: {
        tenantId: tenantId,
        membershipPlanId: String(membershipPlanId),
      },
    });
    customerId = newCustomer.id;
  }
}

// CRITICAL: Add customer if available - this ensures payment method is attached when payment is confirmed
if (customerId) {
  piParams.customer = customerId;
  // Enable payment method reuse for subscriptions
  piParams.setup_future_usage = 'off_session';
}
```

**How It Works**:
1. **Before Payment Intent Creation**: Create or retrieve Stripe Customer by email
2. **Payment Intent Creation**: Include `customer` parameter in `PaymentIntent.create()` call
3. **Payment Confirmation**: When payment is confirmed on frontend, Stripe **automatically attaches** the payment method to the specified customer
4. **Subscription Creation**: Payment method is now reusable and can be used for subscription creation

**Key Benefits**:
- ✅ Minimal code changes (only one file modified)
- ✅ Works with existing Payment Element flow
- ✅ Payment method automatically attached when customer is provided
- ✅ No frontend changes required
- ✅ Safe for production

**Expected Result**:
- Payment method is attached to customer automatically
- Subscription can be created with attached payment method
- Subscription status will be `ACTIVE` (not `incomplete`)
- No more "payment method cannot be reused" errors
- `stripeSubscriptionId` is properly stored in database

**Alternative Solutions Considered**:
1. **Setup Intent for Subscriptions** (Long-term recommended): Use Setup Intent instead of Payment Intent for subscription signups - cleaner separation but requires significant refactoring
2. **Attach Payment Method in Frontend**: Attach payment method before confirming Payment Intent - requires frontend changes and more complex flow
3. **Attach Payment Method After Confirmation**: Current attempt failed because payment method is already marked as "used" before attachment

**Testing Checklist**:
- ✅ Test desktop card payment → subscription should be `active`
- ✅ Test mobile Apple Pay/Google Pay → subscription should be `active`
- ✅ Test existing customer → should use existing customer, attach new payment method
- ✅ Test payment failure → should handle gracefully
- ✅ Verify payment method is attached in Stripe dashboard
- ✅ Verify subscription is `active` in both database and Stripe

**Lesson**: For membership subscriptions, **always create/retrieve the Stripe Customer BEFORE creating the Payment Intent** and include the `customer` parameter. This ensures the payment method is automatically attached when payment is confirmed, making it reusable for subscriptions. This is the simplest and most reliable solution that works with the existing Payment Element flow.

### 11. Stripe API Lookup and Cancellation Fix - CRITICAL for Plan Upgrades

**Issue**: When users upgraded to a new plan, the system was creating duplicate subscriptions instead of cancelling the old one first. Additionally, Stripe API was the source of truth but database could be out of sync.

**Root Cause**:
1. System was only checking database for existing subscriptions
2. Stripe API could have active subscriptions that weren't reflected in database
3. No cancellation of existing active subscriptions before creating new ones
4. Plan switch detection wasn't cancelling old subscriptions properly

**Solution**: Check Stripe API for existing active subscriptions BEFORE creating new ones, and cancel any existing active subscriptions (except the current one being created):

**Implementation** (`src/app/membership/success/ApiServerActions.ts`):
```typescript
// CRITICAL: Check Stripe API for existing active subscriptions BEFORE creating new one
// Stripe API is the source of truth - database may be out of sync
if (stripeCustomerId && !existingSubscription) {
  try {
    console.log('[MEMBERSHIP-SUCCESS] 🔍 Checking Stripe API for existing active subscriptions for customer:', stripeCustomerId);
    const stripeSubscriptions = await stripe().subscriptions.list({
      customer: stripeCustomerId,
      status: 'active', // Only look for active subscriptions
      limit: 10, // Limit to a reasonable number
    });

    if (stripeSubscriptions.data.length > 0) {
      console.log('[MEMBERSHIP-SUCCESS] Found existing active Stripe subscriptions:', stripeSubscriptions.data.length);
      for (const sub of stripeSubscriptions.data) {
        if (sub.id !== stripeSubscriptionId) { // Don't cancel the current one if it's already active
          console.log('[MEMBERSHIP-SUCCESS] Cancelling existing active Stripe subscription:', sub.id);
          await stripe().subscriptions.cancel(sub.id);
          console.log('[MEMBERSHIP-SUCCESS] Cancelled Stripe subscription:', sub.id);

          // Also update our database to reflect cancellation
          const existingDbSubscription = await findSubscriptionByStripeSubscriptionId(sub.id);
          if (existingDbSubscription && existingDbSubscription.subscriptionStatus !== 'CANCELLED') {
            console.log('[MEMBERSHIP-SUCCESS] Updating database subscription to CANCELLED:', existingDbSubscription.id);
            await updateSubscription(existingDbSubscription.id, { subscriptionStatus: 'CANCELLED' });
          }
        }
      }
    }
  } catch (error) {
    console.error('[MEMBERSHIP-SUCCESS] Error checking/cancelling Stripe subscriptions (non-fatal):', error);
    // Continue - will still create new subscription
  }
}
```

**Key Points**:
- Stripe API is the source of truth - check it before creating new subscriptions
- Cancel all existing active subscriptions except the current one
- Update database to reflect Stripe cancellations
- Handle errors gracefully (non-fatal) - continue with subscription creation even if cancellation fails

**Lesson**: Always check Stripe API for existing active subscriptions before creating new ones, especially for plan upgrades. Cancel old subscriptions to prevent duplicates and ensure only one active subscription per customer.

### 12. CANCELLED Subscription Filtering - CRITICAL for New Payments After Cancellation

**Issue**: When a user made a new payment after cancelling a subscription, the system was returning the old cancelled subscription instead of creating a new one.

**Root Cause**: Backend queries were returning CANCELLED subscriptions even with `subscriptionStatus.in=ACTIVE,TRIAL` filter, especially when expanded relations (membershipPlan, userProfile) were included.

**Solution**: Filter out CANCELLED/EXPIRED subscriptions IMMEDIATELY after lookup, before any processing:

**Implementation** (`src/app/api/membership/success/process/route.ts`):
```typescript
if (existingSubscription) {
  // CRITICAL: Filter out CANCELLED/EXPIRED subscriptions IMMEDIATELY after lookup
  // Backend queries may return CANCELLED subscriptions with expanded relations (membershipPlan, userProfile)
  // We must reject these and create a new subscription instead
  const subscriptionStatus = existingSubscription.subscriptionStatus;
  if (subscriptionStatus === 'CANCELLED' || subscriptionStatus === 'EXPIRED') {
    console.log('[MEMBERSHIP-PROCESS GET/POST] ⚠️⚠️⚠️ CRITICAL: Found CANCELLED/EXPIRED subscription - REJECTING and will create new one');
    // CRITICAL: Reset to null so we proceed to create a new subscription
    existingSubscription = null;
  } else if (subscriptionStatus !== 'ACTIVE' && subscriptionStatus !== 'TRIAL') {
    console.error('[MEMBERSHIP-PROCESS GET/POST] ⚠️⚠️⚠️ CRITICAL: Subscription found but status is not ACTIVE/TRIAL');
    existingSubscription = null;
  }
}

// CRITICAL: Final safety check before returning response
if (existingSubscription && (existingSubscription.subscriptionStatus === 'CANCELLED' || existingSubscription.subscriptionStatus === 'EXPIRED')) {
  console.error('[MEMBERSHIP-PROCESS GET/POST] ⚠️⚠️⚠️ CRITICAL: Attempted to return CANCELLED/EXPIRED subscription - REJECTING');
  existingSubscription = null;
}
```

**Key Points**:
- Filter CANCELLED/EXPIRED subscriptions immediately after lookup
- Backend filters may not work correctly with expanded relations
- Final safety check before returning response
- Set `existingSubscription = null` to force creation of new subscription

**Lesson**: Always filter out CANCELLED/EXPIRED subscriptions immediately after lookup, even if backend query includes status filter. Backend filters may not work correctly with expanded relations.

### 13. Plan Details in Response - CRITICAL for Success Page Display

**Issue**: Success page was not displaying plan details (features, max events, max attendees, billing info) because plan was not included in API response.

**Root Cause**: GET endpoint was not ensuring plan details were included in response, especially for existing subscriptions.

**Solution**: Ensure plan details are always included in response, with fallback to fetch plan directly if missing:

**Implementation** (`src/app/api/membership/success/process/route.ts`):
```typescript
// CRITICAL: Ensure plan is included in response for desktop flow
let planDetails = details?.plan;
if (!planDetails && existingSubscription.membershipPlanId) {
  // Fallback: Fetch plan directly if not included in details
  try {
    const { fetchMembershipPlanById } = await import('@/app/membership/success/ApiServerActions');
    planDetails = await fetchMembershipPlanById(existingSubscription.membershipPlanId);
    console.log('[MEMBERSHIP-PROCESS GET] Fetched plan details as fallback for existing subscription');
  } catch (planFetchError) {
    console.error('[MEMBERSHIP-PROCESS GET] Error fetching plan details as fallback:', planFetchError);
  }
}

return NextResponse.json({
  subscription: existingSubscription,
  plan: planDetails || null,
  amount: details?.amount || null,
  currency: details?.currency || 'USD',
});
```

**Key Points**:
- Always include plan details in response
- Fallback to fetch plan directly if not included in details
- Handle errors gracefully (non-fatal)
- Ensure success page has all data needed for display

**Lesson**: Always ensure plan details are included in API response for success page display. Provide fallback to fetch plan directly if not included in initial details.

### 14. Desktop Success Page UI - Complete Plan Details and Styled Buttons

**Issue**: Desktop success page was missing plan features, additional plan details (max events, max attendees, billing), Stripe subscription ID display, and styled action buttons.

**Root Cause**: Success page UI was not matching the complete design with all subscription details and action buttons.

**Solution**: Replicate complete desktop success page UI with:
- Plan features display (`PlanFeaturesList` component)
- Additional plan details (max events, max attendees, billing)
- Stripe subscription ID display
- Four styled action buttons (Manage Subscription, View All Plans, My Profile, Go Home)

**Implementation** (`src/app/membership/success/MembershipSuccessClient.tsx`):
- Plan features from `plan.featuresJson` using `PlanFeaturesList` component
- Max Events per month display (with calendar icon)
- Max Attendees per event display (with users icon)
- Billing information display (with currency icon)
- Stripe Subscription ID display (with code icon)
- Four styled action buttons matching admin action button pattern

**Key Points**:
- Use `PlanFeaturesList` component for plan features display
- Display all plan details (max events, max attendees, billing)
- Show Stripe subscription ID for reference
- Use styled action buttons matching admin action button pattern
- Ensure consistent UI across desktop and mobile flows

**Lesson**: Success page must display complete subscription details including plan features, additional plan details, Stripe subscription ID, and styled action buttons for optimal user experience.

This architecture ensures optimal desktop payment experience while maintaining security, reliability, and proper transaction persistence for the MCEFEE event management system. The membership subscription flow now properly creates Stripe Subscriptions for recurring billing, stores all required Stripe IDs in the database, handles plan upgrades correctly, filters out cancelled subscriptions, and displays complete subscription details on the success page.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/giventadevelop) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
