## from-email-select

> This rule defines the validation patterns and error handling for the `FromEmailSelect` component used in event forms and other admin pages. The component provides a searchable dropdown for selecting tenant email addresses with comprehensive validation.

# From Email Select Component - Validation Rules

## **Overview**
This rule defines the validation patterns and error handling for the `FromEmailSelect` component used in event forms and other admin pages. The component provides a searchable dropdown for selecting tenant email addresses with comprehensive validation.

## **Problem Solved**
- **Database Value Validation**: Validates that `fromEmail` from database matches an email in the tenant email addresses list, clearing invalid values
- **Empty List Validation**: Detects when no email addresses are configured and shows appropriate error message
- **Empty Field Validation**: Validates that user has selected an email address before form submission
- **Custom Error Messages**: Provides user-friendly error messages instead of browser default validation
- **Parent Component Integration**: Notifies parent components about validation state for form-level validation

## **Component Location**
- **Component**: `src/components/FromEmailSelect.tsx`
- **Usage**: `src/components/EventForm.tsx` and other admin forms

## **Validation Rules**

### **1. Database Value Validation (Event Loading)**

**CRITICAL**: When loading event data from the database, the `fromEmail` value must be validated against the tenant email addresses list. If the database value doesn't exist in the list, it must be cleared so validation will catch it.

**Problem**: The database may contain an email address (e.g., `'events@example.com'`) that doesn't exist in the tenant email addresses list. This creates a conflict where:
- The form field appears empty (because `FromEmailSelect` can't display a value not in its list)
- But `form.fromEmail` still has the database value
- Validation passes incorrectly because the field has a value (even though it's invalid)

**Solution**: Validate the database value on event load and clear it if it doesn't exist in the email addresses list.

**Implementation Pattern:**
```tsx
// In EventForm useEffect when loading event data
useEffect(() => {
  if (event) {
    // CRITICAL: Validate fromEmail against the email addresses list
    // If the database value doesn't exist in the list, clear it so validation will catch it
    const validateAndSetFromEmail = async () => {
      let validFromEmail = event.fromEmail || '';

      // Only validate if fromEmail has a value
      if (validFromEmail && validFromEmail.trim() !== '') {
        try {
          // Fetch all email addresses (use a large page size to get all)
          const emailAddresses = await fetchTenantEmailAddressesServer(0, 1000);

          // Check if the fromEmail exists in the list and is active
          const emailExists = emailAddresses.some(
            email => email.emailAddress === validFromEmail && email.isActive === true
          );

          if (!emailExists) {
            // Email doesn't exist in the list - clear it so validation will catch it
            console.warn('[EventForm] fromEmail from database does not exist in email addresses list:', {
              fromEmail: validFromEmail,
              availableEmails: emailAddresses.map(e => e.emailAddress),
            });
            validFromEmail = '';
          }
        } catch (error) {
          // If fetching email addresses fails, log error but still clear the field
          // This ensures validation will catch it
          console.error('[EventForm] Failed to validate fromEmail against email addresses list:', error);
          validFromEmail = '';
        }
      }

      // Set form with validated fromEmail
      const formData = { ...defaultEvent, ...event, fromEmail: validFromEmail };
      setForm(formData);

      // ... rest of event loading logic (metadata, etc.)
    };

    // Call the async function to validate and set fromEmail
    void validateAndSetFromEmail();
  }
}, [event]);
```

**Key Requirements:**
- ✅ **Always validate** database `fromEmail` value against the email addresses list when loading event data
- ✅ **Clear invalid values** - if database value doesn't exist in list, set `form.fromEmail = ''`
- ✅ **Check active status** - only consider emails where `isActive === true`
- ✅ **Handle errors gracefully** - if fetching email addresses fails, clear the field to ensure validation catches it
- ✅ **Log warnings** - log when database value doesn't match list for debugging
- ✅ **Run before form initialization** - validation must complete before setting form state

**Why This Is Critical:**
- Prevents silent validation failures where form appears empty but has invalid value
- Ensures users must select a valid email from the dropdown
- Maintains data integrity by only allowing emails from the tenant email addresses list
- Provides clear validation feedback when database contains invalid data

### **2. Empty Email List Validation**

When the email address list is empty (length === 0), the component must:

- **Display Error Message**: Show a prominent error message below the input field
- **Error Message Text**: "The from email list is empty. Please contact Admin to add the list of from email addresses."
- **Error Styling**: Use red error styling (`bg-red-50`, `border-red-300`, `text-red-700`)
- **Error Icon**: Include an error icon (exclamation circle) for visibility
- **Notify Parent**: Call `onEmptyListChange(true)` callback to notify parent component

**Implementation Pattern:**
```tsx
// In FromEmailSelect component
const [isEmptyList, setIsEmptyList] = useState(false);

const loadEmailAddresses = async () => {
  setLoading(true);
  try {
    const addresses = await fetchTenantEmailAddressesServer();
    setEmailAddresses(addresses);
    const isEmpty = Array.isArray(addresses) && addresses.length === 0;
    setIsEmptyList(isEmpty);
    if (onEmptyListChange) {
      onEmptyListChange(isEmpty);
    }
  } catch (err: any) {
    setEmailAddresses([]);
    setIsEmptyList(true);
    if (onEmptyListChange) {
      onEmptyListChange(true);
    }
  } finally {
    setLoading(false);
  }
};

// Display error message
{!loading && isEmptyList && (
  <div className="mt-2 p-3 bg-red-50 border border-red-300 rounded-lg">
    <div className="flex items-start">
      <svg className="w-5 h-5 text-red-600 mr-2 flex-shrink-0 mt-0.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
      </svg>
      <p className="text-sm font-medium text-red-700">
        The from email list is empty. Please contact Admin to add the list of from email addresses.
      </p>
    </div>
  </div>
)}
```

### **3. Empty Field Validation (Form Submission)**

When the user clicks "Save" without selecting an email address (including untouched fields), the parent form must:

- **Check Empty State**: Validate that `form.fromEmail` is not empty or just whitespace
- **Catch Untouched Fields**: Must validate fields that were never interacted with (initialized as empty string)
- **Show Custom Error**: Display custom error message instead of browser default validation
- **Error Message Text**: "Please enter from email address"
- **Prevent Submission**: Block form submission until email is selected
- **Error Display**: Show error in form's error summary and below the field

**Implementation Pattern:**
```tsx
// In EventForm validate() function
function validate(): boolean {
  const errs: Record<string, string> = {};

  // Validate fromEmail
  // CRITICAL: Field is ALWAYS required, regardless of list state
  // First check: Is the fromEmail field empty or just whitespace?
  // This must catch untouched fields (empty string), null, undefined, and whitespace-only
  // The field is initialized as '' (empty string) in defaultEvent, so we need to check for that
  const fromEmailValue = form.fromEmail;
  // CRITICAL: Normalize value to handle null, undefined, and empty string
  const normalizedFromEmail = fromEmailValue === null || fromEmailValue === undefined ? '' : String(fromEmailValue);
  const isFromEmailEmpty = normalizedFromEmail === '' || normalizedFromEmail.trim() === '';

  // Second check: Is the email list empty?
  if (isEmailListEmpty) {
    // If list is empty, always show error (user cannot select from empty list)
    // This error takes priority - field cannot be used when list is empty
    errs.fromEmail = 'The from email list is empty. Please contact Admin to add the list of from email addresses.';
  } else {
    // List is NOT empty - validate field value
    if (isFromEmailEmpty) {
      // Field is empty - require selection
      errs.fromEmail = 'Please enter from email address';
    } else if (normalizedFromEmail.trim() !== '' && !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(normalizedFromEmail.trim())) {
      // Field has value but format is invalid
      errs.fromEmail = 'Please enter a valid email address';
    }
  }

  setErrors(errs);
  return Object.keys(errs).length === 0;
}
```

### **4. Parent Component Integration**

The parent component (e.g., `EventForm`) must:

- **Track Empty List State**: Maintain `isEmailListEmpty` state
- **Handle Callback**: Implement `onEmptyListChange` callback to update state
- **Clear Errors**: Clear validation errors when list becomes non-empty or email is selected

**Implementation Pattern:**
```tsx
// In EventForm component
const [isEmailListEmpty, setIsEmailListEmpty] = useState(false);

<FromEmailSelect
  value={form.fromEmail}
  onChange={(email) => {
    setForm(prev => ({ ...prev, fromEmail: email || '' }));
    // Clear error when user selects an email
    if (errors.fromEmail) {
      setErrors(prev => {
        const newErrors = { ...prev };
        delete newErrors.fromEmail;
        return newErrors;
      });
    }
  }}
  onEmptyListChange={(isEmpty) => {
    setIsEmailListEmpty(isEmpty);
    // Clear error if list becomes non-empty
    if (!isEmpty && errors.fromEmail && errors.fromEmail.includes('empty')) {
      setErrors(prev => {
        const newErrors = { ...prev };
        delete newErrors.fromEmail;
        return newErrors;
      });
    }
  }}
  required
/>
```

## **Component Props**

### **FromEmailSelectProps**
```typescript
interface FromEmailSelectProps {
  value?: string;                                    // Current selected email value
  onChange: (email: string | undefined) => void;    // Callback when email is selected
  required?: boolean;                                // Whether field is required (for UI indicator)
  filterByType?: TenantEmailAddressDTO['emailType']; // Optional: filter by email type
  showDisplayName?: boolean;                         // Show display name in dropdown
  onEmptyListChange?: (isEmpty: boolean) => void;   // Callback to notify parent when list is empty
}
```

## **Validation Priority Order**

**CRITICAL**: The from email field is **ALWAYS required**, regardless of whether the email list is empty or not.

1. **First Check**: Is the from email field empty? (including untouched fields)
   - **ALWAYS runs** - field validation is never skipped
   - Must catch untouched fields initialized as empty string (`''`)
   - Determines if field has a value before checking list state

2. **Second Check**: Is the email list empty?
   - **If list is empty**: Show "The from email list is empty. Please contact Admin to add the list of from email addresses."
   - **If list is NOT empty AND field is empty**: Show "Please enter from email address"
   - Block form submission in both cases
   - **CRITICAL**: List empty error takes priority - user cannot use emails from empty list

3. **Third Check**: Is the email format valid?
   - **Only runs if list is NOT empty AND field has a value**
   - If format is invalid: Show "Please enter a valid email address"
   - Block form submission

## **Error Display Locations**

### **1. Component-Level Error (FromEmailSelect)**
- **Location**: Below the input field, within the component
- **Condition**: Displayed when `!loading && isEmptyList`
- **Styling**: Red error box with icon and message
- **Purpose**: Immediate feedback when list is empty

### **2. Field-Level Error (EventForm)**
- **Location**: Below the FromEmailSelect component
- **Condition**: Displayed when `errors.fromEmail` exists
- **Styling**: Red text (`text-red-600`)
- **Purpose**: Validation error from form submission

### **3. Form Summary Error (EventForm)**
- **Location**: Error summary box above Save button
- **Condition**: Displayed when `showErrors && getErrorCount() > 0`
- **Styling**: Red error box with list of all errors
- **Purpose**: Comprehensive error overview

## **Browser Validation Prevention**

- **DO NOT** use `required` attribute on the input field
- **DO** use custom validation in form's `validate()` function
- **DO** prevent form submission using `e.preventDefault()` and validation check
- **Rationale**: Browser's default validation messages are not user-friendly and don't match design standards

**Implementation:**
```tsx
// ❌ DON'T: Use browser validation
<input
  type="text"
  required={required}  // This triggers browser's default message
  ...
/>

// ✅ DO: Remove required attribute and use custom validation
<input
  type="search"  // Use type="search" to reduce browser autocomplete
  // No required attribute
  ...
/>

// In form validation - must catch untouched fields
const fromEmailValue = form.fromEmail;
const isFromEmailEmpty = !fromEmailValue ||
                         fromEmailValue === '' ||
                         (typeof fromEmailValue === 'string' && fromEmailValue.trim() === '');

if (isFromEmailEmpty) {
  errs.fromEmail = 'Please enter from email address';
}
```

## **Chrome Browser Autocomplete Prevention**

Chrome browser's autocomplete can interfere with custom dropdowns by appearing on top of them. The following techniques must be implemented to prevent this:

### **Required Techniques**

1. **Hidden Fake Input**: Add a hidden input field before the real input to trick Chrome's autocomplete detection
2. **Input Type**: Use `type="search"` instead of `type="text"` (Chrome is less likely to show autocomplete on search inputs)
3. **High Z-Index**: Set container z-index to `10000` and dropdown z-index to `9999` to ensure custom dropdown appears above browser autocomplete
4. **Multiple Autocomplete Attributes**: Use multiple prevention attributes together for maximum effectiveness

**Implementation Pattern:**
```tsx
{/* Search Input */}
<div className="relative" style={{ zIndex: 10000 }}>
  {/* Hidden fake input to trick Chrome autocomplete - must be before the real input */}
  <input
    type="text"
    autoComplete="off"
    tabIndex={-1}
    style={{
      position: 'absolute',
      opacity: 0,
      height: 0,
      width: 0,
      pointerEvents: 'none',
      zIndex: -1
    }}
    aria-hidden="true"
    readOnly
  />
  <svg
    className="absolute left-3 top-1/2 transform -translate-y-1/2 w-5 h-5 text-gray-400"
    fill="none"
    stroke="currentColor"
    viewBox="0 0 24 24"
  >
    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
  </svg>
  <input
    type="search"  // Use search type to reduce Chrome autocomplete
    value={searchTerm}
    onChange={handleSearchChange}
    onFocus={() => setIsOpen(true)}
    placeholder="Search email addresses..."
    className="mt-1 block w-full border border-gray-400 rounded-xl focus:border-blue-500 focus:ring-blue-500 pl-10 pr-10 px-4 py-2 text-base"
    autoComplete="off"
    autoCapitalize="off"
    autoCorrect="off"
    spellCheck="false"
    data-form-type="other"
    data-lpignore="true"
    data-1p-ignore="true"
    data-browser-autocomplete="off"
    name="fromEmailSearch"
    id="fromEmailSearch"
    role="combobox"
    aria-autocomplete="list"
    aria-expanded={isOpen}
    aria-haspopup="listbox"
  />
</div>

{/* Dropdown Results */}
{isOpen && (filteredEmails.length > 0 || loading) && (
  <div
    className="absolute w-full mt-1 bg-white border border-gray-300 rounded-lg shadow-lg"
    style={{
      maxHeight: 'calc(10 * 3.5rem)',
      overflowY: 'auto',
      zIndex: 9999  // High z-index to appear above browser autocomplete
    }}
  >
    {/* Dropdown content */}
  </div>
)}
```

### **Why These Techniques Work**

1. **Hidden Fake Input**: Chrome's autocomplete detection may target the hidden input instead of the real one
2. **Search Input Type**: Chrome is less aggressive with autocomplete on `type="search"` inputs
3. **High Z-Index**: Ensures custom dropdown appears above browser's autocomplete overlay
4. **Multiple Attributes**: Different browsers respect different attributes, so using multiple increases compatibility

### **Browser Compatibility**

- **Chrome**: Requires all techniques (hidden input, search type, high z-index, multiple attributes)
- **Edge**: Works with standard `autoComplete="off"` (less aggressive autocomplete)
- **Firefox/Safari**: Respects `autoComplete="off"` attribute

### **Testing Checklist**

- [ ] Chrome browser does not show autocomplete dropdown on top of custom dropdown
- [ ] Edge browser works correctly (no autocomplete interference)
- [ ] Custom dropdown appears above any browser autocomplete
- [ ] Hidden fake input does not interfere with form submission
- [ ] Search input type does not add unwanted browser UI elements

## **Error Message Styling**

### **Empty List Error**
```tsx
<div className="mt-2 p-3 bg-red-50 border border-red-300 rounded-lg">
  <div className="flex items-start">
    <svg className="w-5 h-5 text-red-600 mr-2 flex-shrink-0 mt-0.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
      <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
    </svg>
    <p className="text-sm font-medium text-red-700">
      The from email list is empty. Please contact Admin to add the list of from email addresses.
    </p>
  </div>
</div>
```

### **Empty Field Error**
```tsx
{errors.fromEmail && (
  <div className="mt-2 p-3 bg-red-50 border border-red-300 rounded-lg">
    <div className="flex items-start">
      <svg className="w-5 h-5 text-red-600 mr-2 flex-shrink-0 mt-0.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
      </svg>
      <p className="text-sm font-medium text-red-700">{errors.fromEmail}</p>
    </div>
  </div>
)}
```

## **State Management**

### **FromEmailSelect Component State**
- `emailAddresses`: Array of all tenant email addresses
- `filteredEmails`: Filtered list based on search term and filters
- `isEmptyList`: Boolean indicating if email list is empty
- `loading`: Boolean indicating if emails are being loaded
- `selectedEmail`: Currently selected email object
- `searchTerm`: Current search input value
- `isOpen`: Boolean indicating if dropdown is open

### **Parent Component State (EventForm)**
- `isEmailListEmpty`: Boolean tracking if email list is empty (from callback)
- `form.fromEmail`: String value of selected email address
- `errors.fromEmail`: String error message if validation fails

## **Error Clearing Logic**

### **When to Clear Errors**

1. **User Selects Email**: Clear error when `onChange` is called with a valid email
2. **List Becomes Non-Empty**: Clear "empty list" error when `onEmptyListChange(false)` is called
3. **User Starts Typing**: Clear error when user interacts with the field (optional, based on UX preference)

**Implementation:**
```tsx
// Clear error when user selects an email
onChange={(email) => {
  setForm(prev => ({ ...prev, fromEmail: email || '' }));
  if (errors.fromEmail) {
    setErrors(prev => {
      const newErrors = { ...prev };
      delete newErrors.fromEmail;
      return newErrors;
    });
  }
}}

// Clear error when list becomes non-empty
onEmptyListChange={(isEmpty) => {
  setIsEmailListEmpty(isEmpty);
  if (!isEmpty && errors.fromEmail && errors.fromEmail.includes('empty')) {
    setErrors(prev => {
      const newErrors = { ...prev };
      delete newErrors.fromEmail;
      return newErrors;
    });
  }
}}
```

## **Complete Validation Flow**

1. **Event Loads**: When event data is loaded from database:
   - **CRITICAL**: Validate `event.fromEmail` against tenant email addresses list
   - If database value doesn't exist in list → Clear `form.fromEmail = ''`
   - If database value exists in list → Keep it and pre-populate field
   - Log warning if database value is invalid
2. **Component Mounts**: `FromEmailSelect` loads email addresses
3. **Check List**: If list is empty, set `isEmptyList = true` and call `onEmptyListChange(true)`
4. **User Interaction**: User types or selects email (or leaves empty)
5. **Form Submission**: User clicks "Save Event"
6. **Validation Runs**: `validate()` function checks (field is ALWAYS validated):
   - **First**: Is field empty? (always checked)
   - **Then**:
     - If list is empty → Show "list is empty" error (user cannot use emails from empty list)
     - If list is NOT empty AND field is empty → Show "please select from email" error
     - If list is NOT empty AND field has value → Validate email format
7. **Error Display**: Errors shown in component, field, and summary
8. **Submission Blocked**: Form does not submit if validation fails (field is always required)
9. **Error Clearing**: Errors cleared when user selects email or list becomes non-empty

## **Best Practices**

### **DO:**
- ✅ **Validate database values** - always check if `fromEmail` from database exists in email addresses list when loading event data
- ✅ **Clear invalid database values** - if database value doesn't exist in list, set `form.fromEmail = ''` so validation will catch it
- ✅ **ALWAYS validate the field** - field is required regardless of list state
- ✅ Check field emptiness first, then check list state to determine appropriate error message
- ✅ Show "list is empty" error if list is empty (user cannot use emails from empty list)
- ✅ Show "please select from email" error if list is NOT empty but field is empty
- ✅ Use custom error messages instead of browser defaults
- ✅ Display errors in multiple locations (component, field, summary)
- ✅ Clear errors when user corrects the issue
- ✅ Use consistent error styling (red colors, icons)
- ✅ Notify parent component about empty list state
- ✅ Handle loading states gracefully
- ✅ Log warnings when database contains invalid email values

### **DON'T:**
- ❌ **Pre-populate invalid database values** - never set `form.fromEmail` to a value that doesn't exist in the email addresses list
- ❌ **Skip database value validation** - always validate `fromEmail` from database against the email addresses list
- ❌ Use `required` attribute on input field (causes browser validation)
- ❌ Show browser's default validation messages
- ❌ Submit form when validation fails
- ❌ **Skip field validation when email list is empty** - field is ALWAYS required
- ❌ Allow form submission without from email value (regardless of list state)
- ❌ Forget to clear errors when state changes
- ❌ Show errors while data is still loading
- ❌ Use generic error messages (be specific about the issue)
- ❌ Ignore database values that don't match the email list (always clear them)

## **Testing Checklist**

- [ ] **Database value validation** - invalid email from database is cleared on event load
- [ ] **Database value logging** - warning is logged when database value doesn't match email list
- [ ] **Valid database value** - valid email from database is pre-populated correctly
- [ ] Empty list shows error message immediately after load
- [ ] Empty field shows error when clicking "Save" without selection
- [ ] **Untouched field** (never focused/interacted) shows error when clicking "Save"
- [ ] Error messages are custom (not browser default)
- [ ] Form submission is blocked when validation fails
- [ ] Errors clear when user selects an email
- [ ] Errors clear when list becomes non-empty
- [ ] Error appears in component, field, and summary locations
- [ ] Error styling is consistent (red colors, icons, styled error boxes)
- [ ] Loading state doesn't show false errors
- [ ] Chrome browser does not show autocomplete on top of custom dropdown
- [ ] Edge browser works correctly (no autocomplete interference)
- [ ] Custom dropdown appears above any browser autocomplete

## **References**

- **Component**: [`src/components/FromEmailSelect.tsx`](mdc:src/components/FromEmailSelect.tsx)
- **Usage**: [`src/components/EventForm.tsx`](mdc:src/components/EventForm.tsx) - Lines 1689-1723
- **Database Value Validation**: [`src/components/EventForm.tsx`](mdc:src/components/EventForm.tsx) - Lines 83-196 (validates database fromEmail against email addresses list)
- **Validation**: [`src/components/EventForm.tsx`](mdc:src/components/EventForm.tsx) - Lines 244-301 (includes untouched field validation)
- **Chrome Autocomplete Prevention**: [`src/components/FromEmailSelect.tsx`](mdc:src/components/FromEmailSelect.tsx) - Lines 163-210 (hidden input, search type, z-index)
- **Server Action**: [`src/app/admin/tenant-email-addresses/ApiServerActions.ts`](mdc:src/app/admin/tenant-email-addresses/ApiServerActions.ts)

## **Related Patterns**

- See [MOSC Styling Standards](mdc:.cursor/rules/mosc_styling_standards.mdc) for error message styling
- See [Icon Standards](mdc:.cursor/rules/icon_standards.mdc) for error icon usage
- See [Form Validation Patterns](mdc:.cursor/rules/cursor_rules.mdc) for general validation guidelines

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
