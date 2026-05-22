## desktop-payment-flow

> Rules for implementing desktop browser payment flow (Stripe Checkout), transaction persistence, and success page handling separate from mobile workflow


- **Main Points in Bold**
  - Desktop flow uses Stripe Checkout Sessions (`cs_`) OR Stripe Elements inline (ExpressCheckoutElement + PaymentElement) for payment processing
  - **Stripe Elements Inline Option**: Desktop can use Stripe Elements (`StripeDesktopCheckout` component) to show Apple Pay, Google Pay, Link, and card form directly on the page (no redirect)
  - **Stripe Checkout Session Option**: Desktop can also use Stripe Checkout Sessions (hosted payment page) via redirect
  - Desktop flow persists transactions immediately when payment succeeds via GET endpoint fallback
  - Desktop flow is completely separate from mobile workflow - uses different entry points and detection methods
  - Desktop flow must include both `tenantId` and `paymentMethodDomainId` in all backend requests
  - Desktop flow creates both transaction and transaction items via `createTransactionFromPaymentIntent`
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
Payment Intent created via /api/stripe/payment-intent →
Stripe Elements rendered inline (ExpressCheckoutElement + PaymentElement) →
Apple Pay/Google Pay/Link/Card form shown on same page →
Payment completion →
Redirect to /event/success?pi=pi_xxx →
SuccessClient detects desktop →
Stays on success page →
GET /api/event/success/process?pi=pi_xxx →
If transaction not found (webhook delayed) →
Create transaction immediately via createTransactionFromPaymentIntent →
Create transaction items via createTransactionItemsBulk →
Fetch QR code →
Display success page with QR inline
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

This architecture ensures optimal desktop payment experience while maintaining security, reliability, and proper transaction persistence for the MCEFEE event management system.

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
