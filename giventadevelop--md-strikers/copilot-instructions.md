## form-validation-styling

> Standard pattern for form validation styling, error display, and scroll-to-error functionality across all forms (ProfileForm, EventForm, etc.)


# Form Validation Styling Pattern

## **Overview**
This rule defines the standard pattern for form validation styling, error display, and scroll-to-error functionality used across all forms in the application (ProfileForm, EventForm, etc.). This ensures consistent validation UX, error styling, and user feedback patterns.

## **Problem Solved**
- **Consistent Validation UX**: Ensures all forms use the same validation styling and error display patterns
- **Error Visibility**: Red borders, inline error messages, and error summary boxes provide clear feedback
- **Scroll-to-Error**: Automatically navigates users to the first error field on validation failure
- **Real-time Error Clearing**: Errors clear as users type, providing immediate feedback
- **Immediate Field Validation**: Required fields validate on blur (when user clicks outside), showing errors immediately without waiting for form submission
- **Professional Presentation**: Consistent error styling (red borders, red text, error icons) across all forms

## **Core Pattern**

### **1. State Management**

```tsx
// ✅ DO: Add validation state and field refs
import { useState, useRef } from "react";
import { flushSync } from "react-dom";

export default function FormComponent() {
  // Error state management
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [showErrors, setShowErrors] = useState(false);

  // Refs for form fields to enable scroll-to-error functionality
  const fieldRefs = useRef<Record<string, HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>>({});

  // ... rest of component
}
```

### **2. Scroll-to-Error Function**

```tsx
// ✅ DO: Add scroll-to-first-error function
const scrollToFirstError = (errorObj?: Record<string, string>) => {
  // Use provided errors or fall back to state
  const errorsToUse = errorObj || errors;
  const firstErrorField = Object.keys(errorsToUse)[0];
  if (firstErrorField && fieldRefs.current[firstErrorField]) {
    const field = fieldRefs.current[firstErrorField];
    // Scroll to field but DON'T focus it immediately
    // This allows all fields to show red borders before focusing
    field.scrollIntoView({
      behavior: 'smooth',
      block: 'center',
      inline: 'nearest'
    });
    // Delay focus slightly to ensure all fields have rendered with red borders
    setTimeout(() => {
      if (fieldRefs.current[firstErrorField]) {
        fieldRefs.current[firstErrorField]?.focus();
      }
    }, 100);
  }
};
```

### **3. Error Count Helper**

```tsx
// ✅ DO: Add helper function to get error count
const getErrorCount = () => Object.keys(errors).length;
```

### **4. Validation Function**

```tsx
// ✅ DO: Add validate() function with flushSync for immediate state updates
function validate(): boolean {
  const errs: Record<string, string> = {};

  // Required field validations
  if (!formData.firstName || formData.firstName.trim() === '') {
    errs.firstName = 'First name is required';
  }
  if (!formData.lastName || formData.lastName.trim() === '') {
    errs.lastName = 'Last name is required';
  }
  if (!formData.email || formData.email.trim() === '') {
    errs.email = 'Email is required';
  } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(formData.email.trim())) {
    errs.email = 'Please enter a valid email address';
  }

  // Additional validations (length, format, etc.)
  if (formData.title && formData.title.length > 250) {
    errs.title = 'Title must not exceed 250 characters';
  }

  // CRITICAL: Use flushSync to force immediate state update so red borders appear instantly
  const hasErrors = Object.keys(errs).length > 0;

  if (hasErrors) {
    // Force synchronous state updates so fields show red borders immediately
    flushSync(() => {
      setErrors(errs);
      setShowErrors(true);
    });

    // Scroll to first error field
    scrollToFirstError(errs);
  } else {
    setErrors({});
    setShowErrors(false);
  }

  return !hasErrors;
}
```

### **5. HandleChange Pattern (Error Clearing)**

```tsx
// ✅ DO: Clear errors when user starts typing
const handleChange = (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>) => {
  const { name, value, type } = e.target;
  const checked = (e.target as HTMLInputElement).checked;

  // Clear error for this field when user starts typing
  if (errors[name]) {
    setErrors(prev => {
      const newErrors = { ...prev };
      delete newErrors[name];
      return newErrors;
    });
  }

  // Update form data
  setFormData((prev) => ({
    ...prev,
    [name]: type === 'checkbox' ? checked : (value || ''),
  }));
};
```

### **6. Individual Field Validation (onBlur Pattern)**

```tsx
// ✅ DO: Create validateField function for individual field validation on blur
const validateField = (fieldName: keyof ValidationErrors) => {
  const newErrors: ValidationErrors = { ...errors };

  switch (fieldName) {
    case 'fieldName': {
      if (!formData.fieldName?.trim()) {
        newErrors.fieldName = 'Field name is required.';
      } else {
        delete newErrors.fieldName;
      }
      break;
    }

    case 'description': {
      if (!formData.description?.trim()) {
        newErrors.description = 'Description is required.';
      } else {
        delete newErrors.description;
      }
      break;
    }

    case 'price': {
      if (Number(formData.price) <= 0) {
        newErrors.price = 'Price must be greater than zero.';
      } else {
        delete newErrors.price;
      }
      break;
    }

    // Add more field validations as needed
  }

  setErrors(newErrors);
};
```

**Key Points:**
- Use block scope (`{}`) for each case to prevent variable name conflicts
- Validate required fields by checking if value is empty or whitespace
- Clear errors when field becomes valid
- Update errors state immediately for visual feedback

### **7. HandleSubmit Pattern (Validation Before Submission)**

```tsx
// ✅ DO: Validate before submission and prevent default behavior
const handleSubmit = async (e: React.FormEvent) => {
  e.preventDefault();
  e.stopPropagation(); // Prevent browser default validation and scrolling

  // CRITICAL: Always validate before submission
  const isValid = validate();
  if (!isValid) {
    console.log('[FormComponent handleSubmit] Validation failed - preventing submission');
    return; // Validation failed - prevent submission
  }

  console.log('[FormComponent handleSubmit] Validation passed - proceeding with submission');

  // Clear any previous errors and hide error display
  setErrors({});
  setShowErrors(false);

  // Proceed with form submission
  // ... rest of submit logic
};
```

## **Field Styling Pattern**

### **Input Fields with Error Styling**

```tsx
// ✅ DO: Use conditional error styling with refs and onBlur validation
<input
  ref={(el) => { if (el) fieldRefs.current.fieldName = el; }}
  type="text"
  id="fieldName"
  name="fieldName"
  value={formData.fieldName}
  onChange={handleChange}
  onBlur={() => validateField('fieldName')}
  className={`mt-1 block w-full border rounded-xl focus:ring-blue-500 px-4 py-3 text-base ${
    errors.fieldName
      ? 'border-red-500 focus:border-red-500 focus:ring-red-500'
      : 'border-gray-400 focus:border-blue-500'
  }`}
/>
{errors.fieldName && (
  <div className="text-red-500 text-sm mt-1">{errors.fieldName}</div>
)}
```

**Key Points:**
- Add `onBlur={() => validateField('fieldName')}` to all required input fields
- This provides immediate validation feedback when user clicks outside without entering a value
- Red border appears immediately, not waiting for form submission

### **Textarea Fields with Error Styling**

```tsx
// ✅ DO: Apply same error styling pattern to textareas with onBlur validation
<textarea
  ref={(el) => { if (el) fieldRefs.current.description = el; }}
  name="description"
  value={formData.description ?? ""}
  onChange={handleChange}
  onBlur={() => validateField('description')}
  className={`w-full border rounded-xl focus:ring-blue-500 px-4 py-3 text-base ${
    errors.description
      ? 'border-red-500 focus:border-red-500 focus:ring-red-500'
      : 'border-gray-400 focus:border-blue-500'
  }`}
  rows={3}
/>
{errors.description && (
  <div className="text-red-500 text-sm mt-1">{errors.description}</div>
)}
```

**Key Points:**
- Add `onBlur={() => validateField('fieldName')}` to all required fields
- This provides immediate validation feedback when user clicks outside without entering a value
- Red border appears immediately, not waiting for form submission

### **Select Fields with Error Styling**

```tsx
// ✅ DO: Apply same error styling pattern to selects
<select
  ref={(el) => { if (el) fieldRefs.current.eventType = el; }}
  name="eventType"
  value={formData.eventType?.id ?? ''}
  onChange={handleChange}
  className={`w-full border rounded p-2 ${
    errors.eventType
      ? 'border-red-500 focus:border-red-500 focus:ring-red-500'
      : 'border-gray-300 focus:border-blue-500 focus:ring-blue-500'
  }`}
>
  <option value="">Select option</option>
  {/* options */}
</select>
{errors.eventType && (
  <div className="text-red-500 text-sm mt-1">{errors.eventType}</div>
)}
```

## **Error Display Patterns**

### **Inline Error Messages**

```tsx
// ✅ DO: Display inline error messages below fields
{errors.fieldName && (
  <div className="text-red-500 text-sm mt-1">{errors.fieldName}</div>
)}
```

### **Field-Level Error Box (For Complex Validation)**

```tsx
// ✅ DO: Use error box for complex validation messages (optional)
{errors.fieldName && (
  <div className="mt-2 p-3 bg-red-50 border border-red-300 rounded-lg">
    <div className="flex items-start">
      <svg className="w-5 h-5 text-red-600 mr-2 flex-shrink-0 mt-0.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
      </svg>
      <p className="text-sm font-medium text-red-700">{errors.fieldName}</p>
    </div>
  </div>
)}
```

### **Error Summary Box (Above Submit Button)**

```tsx
// ✅ DO: Display error summary box above submit button
{showErrors && getErrorCount() > 0 && (
  <div className="bg-red-50 border border-red-200 rounded-lg p-4 mb-4">
    <div className="flex items-start">
      <div className="flex-shrink-0">
        <svg className="h-5 w-5 text-red-400" viewBox="0 0 20 20" fill="currentColor">
          <path fillRule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clipRule="evenodd" />
        </svg>
      </div>
      <div className="ml-3">
        <h3 className="text-sm font-medium text-red-800">
          Please fix the following {getErrorCount()} error{getErrorCount() !== 1 ? 's' : ''}:
        </h3>
        <div className="mt-2 text-sm text-red-700">
          <ul className="list-disc pl-5 space-y-1">
            {Object.entries(errors).map(([fieldName, errorMessage]) => (
              <li key={fieldName}>
                <span className="font-medium capitalize">{fieldName.replace(/([A-Z])/g, ' $1').trim()}:</span> {errorMessage}
              </li>
            ))}
          </ul>
        </div>
      </div>
    </div>
  </div>
)}
```

## **Tailwind CSS Classes Reference**

### **Error State Classes**

| Element | Normal State | Error State |
|---------|-------------|-------------|
| **Input Border** | `border-gray-400` | `border-red-500` |
| **Input Focus Border** | `focus:border-blue-500` | `focus:border-red-500` |
| **Input Focus Ring** | `focus:ring-blue-500` | `focus:ring-red-500` |
| **Error Message Text** | — | `text-red-500 text-sm mt-1` |
| **Error Box Background** | — | `bg-red-50` |
| **Error Box Border** | — | `border border-red-200` or `border-red-300` |
| **Error Box Text** | — | `text-red-700` or `text-red-800` |
| **Error Icon** | — | `text-red-400` or `text-red-600` |

### **Complete Class Sets**

**Input Field (Normal):**
```tsx
className="mt-1 block w-full border border-gray-400 rounded-xl focus:border-blue-500 focus:ring-blue-500 px-4 py-3 text-base"
```

**Input Field (Error):**
```tsx
className="mt-1 block w-full border border-red-500 rounded-xl focus:border-red-500 focus:ring-red-500 px-4 py-3 text-base"
```

**Textarea Field (Normal):**
```tsx
className="w-full border border-gray-300 rounded p-2 focus:border-blue-500 focus:ring-blue-500"
```

**Textarea Field (Error):**
```tsx
className="w-full border border-red-500 rounded p-2 focus:border-red-500 focus:ring-red-500"
```

**Select Field (Normal):**
```tsx
className="w-full border border-gray-300 rounded p-2 focus:border-blue-500 focus:ring-blue-500"
```

**Select Field (Error):**
```tsx
className="w-full border border-red-500 rounded p-2 focus:border-red-500 focus:ring-red-500"
```

**Error Summary Box:**
```tsx
className="bg-red-50 border border-red-200 rounded-lg p-4 mb-4"
```

**Inline Error Message:**
```tsx
className="text-red-500 text-sm mt-1"
```

**Error Box with Icon:**
```tsx
className="mt-2 p-3 bg-red-50 border border-red-300 rounded-lg"
```

## **Key CSS Properties**

### **Error State Border Colors**
- **Normal**: `border-gray-400` or `border-gray-300`
- **Error**: `border-red-500` (darker red for visibility)
- **Focus Normal**: `focus:border-blue-500`
- **Focus Error**: `focus:border-red-500`

### **Error State Ring Colors**
- **Normal**: `focus:ring-blue-500`
- **Error**: `focus:ring-red-500`

### **Error Message Styling**
- **Text Color**: `text-red-500` (inline) or `text-red-700` (error box)
- **Text Size**: `text-sm` (small, non-intrusive)
- **Margin**: `mt-1` (spacing above message)

### **Error Summary Box Styling**
- **Background**: `bg-red-50` (light red background)
- **Border**: `border border-red-200` (subtle red border)
- **Padding**: `p-4` (comfortable spacing)
- **Margin**: `mb-4` (spacing before submit button)
- **Border Radius**: `rounded-lg` (rounded corners)

### **Error Icon Styling**
- **Size**: `w-5 h-5` (20px × 20px)
- **Color**: `text-red-400` (summary box) or `text-red-600` (field-level error box)
- **Position**: `flex-shrink-0` (prevents icon from shrinking)

## **Complete Example**

### **Full Form Component with Validation**

```tsx
"use client";

import { useState, useRef } from "react";
import { flushSync } from "react-dom";

interface FormData {
  firstName: string;
  lastName: string;
  email: string;
}

export default function FormComponent() {
  // State management
  const [formData, setFormData] = useState<FormData>({
    firstName: '',
    lastName: '',
    email: '',
  });
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [showErrors, setShowErrors] = useState(false);
  const [loading, setLoading] = useState(false);

  // Refs for scroll-to-error
  const fieldRefs = useRef<Record<string, HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>>({});

  // Scroll to first error
  const scrollToFirstError = (errorObj?: Record<string, string>) => {
    const errorsToUse = errorObj || errors;
    const firstErrorField = Object.keys(errorsToUse)[0];
    if (firstErrorField && fieldRefs.current[firstErrorField]) {
      const field = fieldRefs.current[firstErrorField];
      field.scrollIntoView({
        behavior: 'smooth',
        block: 'center',
        inline: 'nearest'
      });
      setTimeout(() => {
        if (fieldRefs.current[firstErrorField]) {
          fieldRefs.current[firstErrorField]?.focus();
        }
      }, 100);
    }
  };

  // Error count helper
  const getErrorCount = () => Object.keys(errors).length;

  // Validation function
  function validate(): boolean {
    const errs: Record<string, string> = {};

    if (!formData.firstName || formData.firstName.trim() === '') {
      errs.firstName = 'First name is required';
    }
    if (!formData.lastName || formData.lastName.trim() === '') {
      errs.lastName = 'Last name is required';
    }
    if (!formData.email || formData.email.trim() === '') {
      errs.email = 'Email is required';
    } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(formData.email.trim())) {
      errs.email = 'Please enter a valid email address';
    }

    const hasErrors = Object.keys(errs).length > 0;

    if (hasErrors) {
      flushSync(() => {
        setErrors(errs);
        setShowErrors(true);
      });
      scrollToFirstError(errs);
    } else {
      setErrors({});
      setShowErrors(false);
    }

    return !hasErrors;
  }

  // Handle change (clear errors)
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;

    if (errors[name]) {
      setErrors(prev => {
        const newErrors = { ...prev };
        delete newErrors[name];
        return newErrors;
      });
    }

    setFormData(prev => ({ ...prev, [name]: value }));
  };

  // Handle submit (validate first)
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    e.stopPropagation();

    const isValid = validate();
    if (!isValid) {
      return;
    }

    setErrors({});
    setShowErrors(false);

    setLoading(true);
    // ... submit logic
    setLoading(false);
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4 max-w-2xl mx-auto p-4">
      {/* First Name Field */}
      <div>
        <label htmlFor="firstName" className="block text-sm font-medium text-gray-700">
          First Name *
        </label>
        <input
          ref={(el) => { if (el) fieldRefs.current.firstName = el; }}
          type="text"
          id="firstName"
          name="firstName"
          value={formData.firstName}
          onChange={handleChange}
          className={`mt-1 block w-full border rounded-xl focus:ring-blue-500 px-4 py-3 text-base ${
            errors.firstName
              ? 'border-red-500 focus:border-red-500 focus:ring-red-500'
              : 'border-gray-400 focus:border-blue-500'
          }`}
        />
        {errors.firstName && (
          <div className="text-red-500 text-sm mt-1">{errors.firstName}</div>
        )}
      </div>

      {/* Last Name Field */}
      <div>
        <label htmlFor="lastName" className="block text-sm font-medium text-gray-700">
          Last Name *
        </label>
        <input
          ref={(el) => { if (el) fieldRefs.current.lastName = el; }}
          type="text"
          id="lastName"
          name="lastName"
          value={formData.lastName}
          onChange={handleChange}
          className={`mt-1 block w-full border rounded-xl focus:ring-blue-500 px-4 py-3 text-base ${
            errors.lastName
              ? 'border-red-500 focus:border-red-500 focus:ring-red-500'
              : 'border-gray-400 focus:border-blue-500'
          }`}
        />
        {errors.lastName && (
          <div className="text-red-500 text-sm mt-1">{errors.lastName}</div>
        )}
      </div>

      {/* Email Field */}
      <div>
        <label htmlFor="email" className="block text-sm font-medium text-gray-700">
          Email *
        </label>
        <input
          ref={(el) => { if (el) fieldRefs.current.email = el; }}
          type="email"
          id="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
          className={`mt-1 block w-full border rounded-xl focus:ring-blue-500 px-4 py-3 text-base ${
            errors.email
              ? 'border-red-500 focus:border-red-500 focus:ring-red-500'
              : 'border-gray-400 focus:border-blue-500'
          }`}
        />
        {errors.email && (
          <div className="text-red-500 text-sm mt-1">{errors.email}</div>
        )}
      </div>

      {/* Error Summary Box */}
      {showErrors && getErrorCount() > 0 && (
        <div className="bg-red-50 border border-red-200 rounded-lg p-4 mb-4">
          <div className="flex items-start">
            <div className="flex-shrink-0">
              <svg className="h-5 w-5 text-red-400" viewBox="0 0 20 20" fill="currentColor">
                <path fillRule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clipRule="evenodd" />
              </svg>
            </div>
            <div className="ml-3">
              <h3 className="text-sm font-medium text-red-800">
                Please fix the following {getErrorCount()} error{getErrorCount() !== 1 ? 's' : ''}:
              </h3>
              <div className="mt-2 text-sm text-red-700">
                <ul className="list-disc pl-5 space-y-1">
                  {Object.entries(errors).map(([fieldName, errorMessage]) => (
                    <li key={fieldName}>
                      <span className="font-medium capitalize">{fieldName.replace(/([A-Z])/g, ' $1').trim()}:</span> {errorMessage}
                    </li>
                  ))}
                </ul>
              </div>
            </div>
          </div>
        </div>
      )}

      {/* Submit Button */}
      <div className="flex justify-end pt-4">
        <button
          type="submit"
          disabled={loading}
          className="bg-blue-600 text-white px-4 py-2 rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed"
        >
          {loading ? "Saving..." : "Save"}
        </button>
      </div>
    </form>
  );
}
```

## **Best Practices**

### **DO:**
- ✅ Always use `flushSync` in `validate()` for immediate state updates
- ✅ Always call `scrollToFirstError()` after setting errors
- ✅ Always clear errors in `handleChange` when user starts typing
- ✅ Always validate before submission in `handleSubmit`
- ✅ **Always add `onBlur={() => validateField('fieldName')}` to required fields** for immediate validation feedback
- ✅ **Create `validateField()` function** to validate individual fields on blur
- ✅ Always add refs to form fields for scroll-to-error
- ✅ Always display inline error messages below fields
- ✅ Always show error summary box when `showErrors && getErrorCount() > 0`
- ✅ Use consistent error styling: `border-red-500`, `text-red-500`
- ✅ Use `e.preventDefault()` and `e.stopPropagation()` in `handleSubmit`
- ✅ Remove `required` attributes (use custom validation instead)
- ✅ Use block scope (`{}`) in switch cases to prevent variable name conflicts

### **DON'T:**
- ❌ Don't use browser default validation (`required` attribute)
- ❌ Don't skip `flushSync` in validation (causes delayed error display)
- ❌ Don't skip `scrollToFirstError` after validation failure
- ❌ **Don't skip `onBlur` validation** on required fields (users should see errors immediately when clicking outside)
- ❌ Don't forget to add refs to form fields
- ❌ Don't use inconsistent error colors (always use `red-500` for borders, `red-500` for text)
- ❌ Don't skip error clearing in `handleChange`
- ❌ Don't skip validation in `handleSubmit`
- ❌ Don't use different error summary box styling
- ❌ Don't declare variables with the same name in different switch cases without block scope (`{}`)

## **Common Validation Patterns**

### **Required Field Validation**
```tsx
if (!formData.fieldName || formData.fieldName.trim() === '') {
  errs.fieldName = 'Field name is required';
}
```

### **Email Format Validation**
```tsx
if (!formData.email || formData.email.trim() === '') {
  errs.email = 'Email is required';
} else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(formData.email.trim())) {
  errs.email = 'Please enter a valid email address';
}
```

### **Length Validation**
```tsx
if (formData.title && formData.title.length > 250) {
  errs.title = 'Title must not exceed 250 characters';
}
```

### **Date Validation (Future Date)**
```tsx
const today = new Date();
const todayStr = today.getFullYear() + '-' +
  String(today.getMonth() + 1).padStart(2, '0') + '-' +
  String(today.getDate()).padStart(2, '0');

if (formData.startDate && formData.startDate < todayStr) {
  errs.startDate = 'Start date must be today or in the future';
}
```

## **Reference Implementations**

- **ProfileForm**: [`src/components/ProfileForm.tsx`](mdc:src/components/ProfileForm.tsx) - Lines 120-234 (validation pattern), Lines 490-550 (field styling), Lines 789-810 (error summary box)
- **EventForm**: [`src/components/EventForm.tsx`](mdc:src/components/EventForm.tsx) - Lines 80-236 (validation pattern), Lines 994-1200 (field styling), Lines 1786-1810 (error summary box)
- **TicketTypeListClient**: [`src/app/admin/events/[id]/ticket-types/list/TicketTypeListClient.tsx`](mdc:src/app/admin/events/[id]/ticket-types/list/TicketTypeListClient.tsx) - Lines 314-391 (validateField function with onBlur validation), Lines 715-725 (textarea with onBlur), Lines 810-867 (input fields with onBlur)

## **Troubleshooting**

### **Errors Not Showing Red Borders?**
- Check that `flushSync` is used in `validate()`
- Verify `errors[fieldName]` is being checked in className
- Ensure error state is set before render

### **Scroll-to-Error Not Working?**
- Verify refs are added to all form fields
- Check that `scrollToFirstError()` is called after `flushSync`
- Ensure field names match between `fieldRefs` and error keys

### **Errors Not Clearing on Type?**
- Verify `handleChange` clears errors when user types
- Check that field `name` matches error key

### **Error Summary Box Not Showing?**
- Verify `showErrors` is set to `true` in `validate()`
- Check that `getErrorCount() > 0` condition is met
- Ensure error summary box is placed above submit button

## **Related Patterns**

- See [Dialog Button Styling](mdc:.cursor/rules/dialog_button_styling.mdc) for button styling patterns
- See [Icon Standards](mdc:.cursor/rules/icon_standards.mdc) for error icon patterns
- See [MOSC Styling Standards](mdc:.cursor/rules/mosc_styling_standards.mdc) for overall design system

## **Summary**

**Key Pattern**: Forms should always:
1. Use `errors`, `showErrors`, and `fieldRefs` state
2. Validate before submission with `validate()` function
3. **Create `validateField()` function** for individual field validation on blur
4. **Add `onBlur={() => validateField('fieldName')}` to all required fields** for immediate feedback
5. Use `flushSync` for immediate error state updates
6. Scroll to first error with `scrollToFirstError()`
7. Display inline errors below fields (`text-red-500 text-sm mt-1`)
8. Show error summary box above submit button
9. Clear errors in `handleChange` when user types
10. Use consistent error styling: `border-red-500`, `text-red-500`, `bg-red-50`
11. Use block scope (`{}`) in switch cases to prevent variable name conflicts

**onBlur Validation Pattern**: When a user clicks on a required field and then clicks outside without entering a value, the validation error should be shown immediately with a red border and error message. This provides instant feedback without waiting for form submission.

This ensures consistent, professional validation UX across all forms in the application.

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
