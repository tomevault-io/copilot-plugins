## numerical-value-display

> Standards for displaying numerical and currency values with consistent decimal places


# Numerical & Currency Value Display Standards

## **Overview**
This rule defines the standard pattern for displaying numerical and currency values across the application, ensuring consistent decimal place formatting (e.g., `$0.80` instead of `$0.8`, `$10.00` instead of `$10`).

## **Problem Solved**
- **Consistent Currency Display**: Ensures all currency values display with exactly 2 decimal places
- **Professional Presentation**: Prevents confusion from inconsistent decimal formatting
- **User Experience**: Users see consistent, professional monetary values throughout the application

## **Core Rule**

### **Currency Display Standard**
- **Rule:** Always display currency values with exactly 2 decimal places.
- **Purpose:** Ensures consistent and professional presentation of monetary values.
- **Implementation:** Use `Intl.NumberFormat` with `minimumFractionDigits: 2` and `maximumFractionDigits: 2`.

## **Currency Formatting Function**

### **Standard Implementation**
```typescript
// ✅ DO: Format currency to 2 decimal places
const formatCurrency = (amount: number, currency: string = 'USD') => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency,
    minimumFractionDigits: 2,  // Always show 2 decimal places
    maximumFractionDigits: 2,  // Never show more than 2
  }).format(amount);
};

// Examples:
formatCurrency(0.8, 'USD');   // Returns: "$0.80"
formatCurrency(10, 'USD');     // Returns: "$10.00"
formatCurrency(99.9, 'USD');   // Returns: "$99.90"
formatCurrency(100.5, 'USD');  // Returns: "$100.50"
```

### **Common Usage Patterns**

#### **Display Price in Cards/Tables**
```tsx
// ✅ DO: Use formatCurrency for all price displays
<span className="text-4xl font-bold">
  {formatCurrency(plan.price, plan.currency)}
</span>

// ❌ DON'T: Display raw price value
<span className="text-4xl font-bold">
  ${plan.price}  // May show "$0.8" instead of "$0.80"
</span>
```

#### **Price Input Field Formatting**
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
  onBlur={(e) => {
    const numValue = parseFloat(e.target.value) || 0;
    setDisplayPrice(numValue.toFixed(2)); // Format to 2 decimal places on blur
  }}
  step="0.01"
  placeholder="0.00"
/>
```

#### **Display Price Values in Tables**
```tsx
// ✅ DO: Format price for display in tables
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

## **Non-Currency Numerical Values**

### **Decimal Numbers (Percentages, Rates, etc.)**
For non-currency numerical values that require decimal precision:

```typescript
// ✅ DO: Use appropriate decimal places for context
const formatPercentage = (value: number) => {
  return new Intl.NumberFormat('en-US', {
    style: 'percent',
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
  }).format(value / 100);
};

// Examples:
formatPercentage(85.5);  // Returns: "85.50%"
formatPercentage(100);   // Returns: "100.00%"
```

### **Whole Numbers (Counts, Quantities)**
For whole numbers that don't require decimal places:

```typescript
// ✅ DO: Use no decimal places for counts
const formatCount = (count: number) => {
  return new Intl.NumberFormat('en-US', {
    minimumFractionDigits: 0,
    maximumFractionDigits: 0,
  }).format(count);
};

// Examples:
formatCount(10);   // Returns: "10"
formatCount(1000); // Returns: "1,000"
```

## **Input Field Best Practices**

### **Price Input Fields**
- **Display Value**: Use string state to allow free typing (e.g., `displayPrice: "0.80"`)
- **Numeric Value**: Store actual numeric value separately (e.g., `formData.price: 0.8`)
- **On Blur Formatting**: Format to 2 decimal places when user leaves the field
- **Step Attribute**: Use `step="0.01"` for price inputs
- **Zero Prefix Validation**: Automatically prefix decimal values starting with `.` with `0` (e.g., `.70` becomes `0.70`)

```tsx
// ✅ DO: Format price input with 2 decimal places and zero prefix validation
const [displayPrice, setDisplayPrice] = useState<string>('');

useEffect(() => {
  if (editingItem) {
    setDisplayPrice(editingItem.price ? editingItem.price.toFixed(2) : '0.00');
  } else {
    setDisplayPrice('');
  }
}, [editingItem]);

<input
  type="text"
  name="price"
  inputMode="decimal"
  value={displayPrice}
  onChange={(e) => {
    const inputValue = e.target.value;

    // Update display value immediately to allow free typing
    setDisplayPrice(inputValue);

    // CRITICAL: If value starts with a decimal point (e.g., ".70"), prefix with "0"
    let processedValue = inputValue;
    if (inputValue.startsWith('.')) {
      processedValue = '0' + inputValue;
      setDisplayPrice(processedValue);
    }

    // Parse and update formData with numeric value
    if (inputValue === '' || inputValue === '.') {
      setFormData(prev => ({ ...prev, price: 0 }));
    } else {
      const numValue = parseFloat(processedValue);
      if (!isNaN(numValue)) {
        setFormData(prev => ({ ...prev, price: numValue }));
      } else {
        setFormData(prev => ({ ...prev, price: 0 }));
      }
    }
  }}
  onBlur={(e) => {
    // Ensure decimal values are properly formatted on blur
    const value = e.target.value;
    let finalValue = value;

    if (value && value.startsWith('.')) {
      finalValue = '0' + value;
    }

    const numValue = parseFloat(finalValue) || 0;
    setFormData(prev => ({ ...prev, price: numValue }));
    setDisplayPrice(numValue.toFixed(2));
  }}
  className="mt-1 block w-full border border-gray-400 rounded-xl focus:border-blue-500 focus:ring-blue-500 px-4 py-3 text-base"
  pattern="[0-9]*\.?[0-9]*"
  placeholder="0.00"
  required
/>
```

### **Zero Prefix Validation Rule**
- **Requirement**: When a user types a decimal value starting with `.` (e.g., `.70`), automatically prefix it with `0` to become `0.70`
- **Purpose**: Prevents invalid decimal input and ensures consistent formatting
- **Implementation**: Check `inputValue.startsWith('.')` in the `onChange` handler and prefix with `'0'`
- **User Experience**: Provides immediate visual feedback by updating the display value as the user types

**Example Behavior**:
- User types `.70` → Automatically becomes `0.70` ✅
- User types `.5` → Automatically becomes `0.5` ✅
- User types `10.50` → Remains `10.50` ✅
- User types `0.70` → Remains `0.70` ✅

## **Common Anti-Patterns**

```typescript
// ❌ DON'T: Use minimumFractionDigits: 0 for currency
const formatPrice = (price: number) => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 0,  // WRONG: Will show "$0.8" instead of "$0.80"
    maximumFractionDigits: 2,
  }).format(price);
};

// ❌ DON'T: Display raw price values
<span>${plan.price}</span>  // May show "$0.8" instead of "$0.80"

// ❌ DON'T: Use toFixed() without Intl.NumberFormat for currency
<span>${plan.price.toFixed(2)}</span>  // Works but doesn't handle currency symbols/locale
```

## **Best Practices**

### **DO:**
- ✅ Always use `minimumFractionDigits: 2` and `maximumFractionDigits: 2` for currency
- ✅ Use `Intl.NumberFormat` for all currency displays
- ✅ Format price inputs to 2 decimal places on blur
- ✅ Use `step="0.01"` for price input fields (or `type="text"` with `inputMode="decimal"`)
- ✅ Store display value as string, numeric value as number
- ✅ Use `toFixed(2)` for initializing display values from numeric data
- ✅ **Automatically prefix decimal values starting with `.` with `0`** (e.g., `.70` → `0.70`)
- ✅ Check `inputValue.startsWith('.')` in `onChange` handler and prefix with `'0'`

### **DON'T:**
- ❌ Use `minimumFractionDigits: 0` for currency values
- ❌ Display raw price values without formatting
- ❌ Mix different decimal place standards across the application
- ❌ Use `toFixed()` alone for currency (use `Intl.NumberFormat` instead)
- ❌ Skip formatting for "whole dollar" amounts (always show 2 decimal places)
- ❌ **Allow decimal values starting with `.` without zero prefix** (e.g., `.70` ❌ should be `0.70` ✅)

## **Reference Implementations**

- **Membership Page**: [`src/app/membership/MembershipClient.tsx`](mdc:src/app/membership/MembershipClient.tsx) - `formatPrice` function
- **Membership Plan Form**: [`src/components/admin/membership/MembershipPlanForm.tsx`](mdc:src/components/admin/membership/MembershipPlanForm.tsx) - Price input formatting with zero prefix validation
- **Ticket Type List**: [`src/app/admin/events/[id]/ticket-types/list/TicketTypeListClient.tsx`](mdc:src/app/admin/events/[id]/ticket-types/list/TicketTypeListClient.tsx) - Price input with zero prefix validation
- **Membership Plan List**: [`src/components/admin/membership/MembershipPlanList.tsx`](mdc:src/components/admin/membership/MembershipPlanList.tsx) - Price display in table
- **Currency Formatting Helper**: [`src/lib/payments/localization.ts`](mdc:src/lib/payments/localization.ts) - `formatCurrency` utility (if available)

## **Summary**

**Key Rules**:
1. **Currency Display**: Always display currency values with exactly 2 decimal places using `Intl.NumberFormat` with `minimumFractionDigits: 2` and `maximumFractionDigits: 2`.
2. **Zero Prefix Validation**: Automatically prefix decimal values starting with `.` with `0` in input fields (e.g., `.70` → `0.70`).

**Display Examples**:
- `$0.80` ✅ (not `$0.8` ❌)
- `$10.00` ✅ (not `$10` ❌)
- `$99.90` ✅ (not `$99.9` ❌)

**Input Validation Examples**:
- User types `.70` → Automatically becomes `0.70` ✅
- User types `.5` → Automatically becomes `0.5` ✅
- User types `10.50` → Remains `10.50` ✅

This ensures consistent, professional presentation of monetary values throughout the application and prevents invalid decimal input.

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
