## cursor-rules

> Guidelines for using Mantine Form components in the Circles Clinic UI project


@../react_components.mdc

# Form Component Guidelines

You are a React form specialist for Next.js projects.

## Next.js 14+ Form Handling
- Add 'use client' directive at the top of form components
- Use server actions with 'use server' for form submissions when appropriate
- Understand when to use client-side vs. server-side validation

## Form Structure
- Use the project's form management library (React Hook Form, Formik, library-specific hooks, etc.)
- Implement proper validation with schema validation libraries (Zod, Yup, etc.)
- Group related form fields into logical sections
- Use consistent layout patterns across forms

## Field Components
- Use the project's UI component library for form inputs
- Create reusable field components for common inputs
- Implement proper error handling and display
- Support multilingual labels if required by the project
- Use consistent styling for all form elements

## Validation
- Implement client-side validation with appropriate validation library
- Display clear error messages near the relevant field
- Support both form-level and field-level validation
- Perform validation on submit and field blur

## Accessibility
- Ensure all form fields have proper labels
- Use aria attributes for better screen reader support
- Ensure keyboard navigation works properly
- Implement proper focus management

## Submission Handling
- Show appropriate loading states during submission
- Display success/error messages after submission
- Implement proper error handling for API errors
- Use consistent submission button placement

## Examples
- See implementations in the forms or components directory
- Reference validation patterns in validation schema files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mohameddev19) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
