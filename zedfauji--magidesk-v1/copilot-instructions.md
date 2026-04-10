## magidesk-v1

> - Entities contain behavior, not just data

# Domain Model Rules

## Entity Design Principles

### Rich Domain Model
- Entities contain behavior, not just data
- Business logic lives in entities and domain services
- Entities enforce their own invariants
- Use domain events for important state changes

### Aggregate Roots
- **Ticket**: Aggregate root for order management
- **CashSession**: Aggregate root for cash management
- **Shift**: Aggregate root for shift management
- Only aggregate roots are accessed via repositories
- Child entities accessed through aggregate root

### Value Objects
- **Money**: Immutable, enforces 2 decimal places, currency support
- **UserId**: Strongly-typed identifier
- Value objects are immutable
- Use value objects to prevent primitive obsession

## Ticket Entity Rules

### Properties
- Use `TicketStatus` enum (not multiple booleans)
- Support multiple payments (collection, not single)
- Support multiple order lines
- Support discounts (item and ticket level)
- Support gratuity (optional)
- Version field for optimistic concurrency

### State Transitions
- Draft → Open (when first item added)
- Open → Paid (when PaidAmount >= TotalAmount)
- Paid → Closed (when settled)
- Any → Voided (if no payments)
- Closed → Refunded (via refund workflow)

### Invariants
- Cannot add items to Closed/Voided/Refunded tickets
- Cannot void ticket with payments
- Cannot close ticket with DueAmount > 0
- TotalAmount = Subtotal + Tax + ServiceCharge + DeliveryCharge + Adjustment - Discount + Gratuity
- PaidAmount = Sum of all payment amounts
- DueAmount = TotalAmount - PaidAmount

### Methods
- `CalculateTotals()`: Recalculates all amounts
- `CanAddPayment(Payment)`: Validates payment
- `CanClose()`: Validates closing
- `CanVoid()`: Validates voiding
- `CanRefund(Money)`: Validates refund
- `CanSplit()`: Validates splitting

## OrderLine Entity Rules

### Properties
- Support fractional quantities (for weight-based items)
- Support discrete quantities (ItemCount)
- Support modifiers and add-ons
- Support item-level discounts
- Snapshot menu item data (name, price, tax rate)

### Invariants
- Quantity > 0 OR ItemCount > 0
- UnitPrice >= 0
- DiscountAmount <= SubtotalAmount
- All calculated amounts >= 0
- Cannot modify if parent ticket is finalized

### Methods
- `CalculatePrice()`: Recalculates line totals
- `CanMerge(OrderLine)`: Checks if items can be merged
- `Merge(OrderLine)`: Merges identical items

## Payment Entity Rules

### Base Payment
- Abstract base class or interface
- Common properties: Id, Amount, TransactionType, PaymentType, etc.
- Type-specific properties in derived classes

### Payment Types
- **CashTransaction**: TenderAmount, ChangeAmount
- **CreditCardTransaction**: Card details, AuthCode, etc.
- **DebitCardTransaction**: Card details
- **GiftCertificateTransaction**: CertNumber, FaceValue, CashBack
- **CustomPaymentTransaction**: Custom fields

### Invariants
- Amount > 0
- For cash: TenderAmount >= Amount
- For cash: ChangeAmount = TenderAmount - Amount
- Cannot void if already voided
- Cannot refund if not completed
- Gift cert cash back <= (FaceValue - PaidAmount)

### Split Payments
- Multiple payments per ticket supported
- Partial payments allowed
- Ticket closes when PaidAmount >= TotalAmount
- Each payment is independent

## CashSession Entity Rules

### Properties
- OpeningBalance (cash at start)
- ExpectedCash (calculated)
- ActualCash (counted at close)
- Difference (ActualCash - ExpectedCash)
- Status (Open/Closed)

### Invariants
- OpeningBalance >= 0
- Cannot close with open tickets for user
- ExpectedCash = OpeningBalance + CashReceipts - CashRefunds - Payouts - CashDrops - Bleeds
- Cannot modify once Closed
- Only one open session per user

### Methods
- `CalculateExpectedCash()`: Calculates expected amount
- `CanClose()`: Validates closing
- `Close(actualCash)`: Closes session

## Discount Entity Rules

### Discount Definition
- Reference data (can change over time)
- Multiple discount types supported
- Minimum buy/quantity requirements
- Auto-apply option

### Applied Discounts
- **TicketDiscount**: Snapshot of discount applied to ticket
- **OrderLineDiscount**: Snapshot of discount applied to item
- Immutable once created
- Only one discount applies (max discount selected)

### Invariants
- Discount amount cannot exceed subtotal
- Discount eligibility checked before application
- Applied discounts are immutable snapshots

## Gratuity Entity Rules

### Properties
- Amount (tips/gratuity)
- Paid (separate from ticket payment)
- Refunded (if ticket refunded)
- Belongs to ticket

### Invariants
- Amount >= 0
- Can be paid separately from ticket
- If ticket refunded, gratuity may be refunded

## Domain Events

### Event Design
- Immutable events
- Contain entity state snapshots
- Timestamped
- User attribution
- Correlation ID for related events

### Required Events
- TicketCreated, TicketOpened, TicketClosed, TicketVoided, TicketRefunded
- PaymentProcessed, PaymentAuthorized, PaymentCaptured, PaymentVoided, PaymentRefunded
- CashSessionOpened, CashSessionClosed
- OrderLineAdded, OrderLineRemoved, OrderLineModified
- DiscountApplied

## Domain Services

### TicketDomainService
- Complex ticket operations
- CalculateTotals, CanAddPayment, CanClose, CanVoid, CanRefund, CanSplit
- Stateless service

### PaymentDomainService
- Payment operations
- CalculateChange, CanVoid, CanRefund, CanCapture
- Stateless service

### DiscountDomainService
- Discount calculations
- GetMaxDiscount, CalculateDiscountAmount, IsEligible
- Stateless service

### CashSessionDomainService
- Cash session operations
- CalculateExpectedCash, CanClose
- Stateless service

## Enumerations

### Use Enums, Not Strings
- TicketStatus enum (not string)
- PaymentType enum (not string)
- PaymentStatus enum (not string)
- TransactionType enum (Credit/Debit)
- CashSessionStatus enum
- TableStatus enum
- KitchenStatus enum

### Enum Rules
- Strongly typed
- Cannot have invalid values
- Easy to validate
- IntelliSense support

## Immutability Rules

### Immutable Entities
- AuditEvent: Immutable once created
- Applied discounts: Immutable snapshots
- Closed tickets: Immutable (use reopen if needed)
- Closed cash sessions: Immutable
- Completed payments: Immutable (use refund instead)

### Immutable Value Objects
- Money: Immutable
- UserId: Immutable

## Relationships

### One-to-Many
- Ticket → OrderLines
- Ticket → Payments
- Ticket → TicketDiscounts
- OrderLine → OrderLineModifiers
- OrderLine → OrderLineDiscounts
- CashSession → Payments (cash only)

### Many-to-One
- OrderLine → Ticket
- Payment → Ticket
- Payment → CashSession (if cash)
- Ticket → Shift
- Ticket → OrderType
- Ticket → User (owner)

### Optional Relationships
- Ticket → Customer (optional)
- Ticket → Gratuity (optional)
- Ticket → AssignedDriver (optional)
- OrderLine → SizeModifier (optional)

## Validation Rules

### Construction Validation
- All required properties must be provided
- Invariants checked at construction
- Invalid state cannot be created

### Mutation Validation
- State transitions validated
- Invariants checked before mutation
- Invalid operations throw exceptions

### Business Rule Validation
- Domain services validate complex rules
- Cannot bypass validation
- Validation is part of domain logic

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zedfauji) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
