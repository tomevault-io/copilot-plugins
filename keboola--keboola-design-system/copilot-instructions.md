## keboola-design-system

> Typography forms the foundation of our design system, providing consistent text styles that enhance readability and establish visual hierarchy. This guide provides comprehensive instructions for implementing typography in your applications.

# Typography Usage Guide

Typography forms the foundation of our design system, providing consistent text styles that enhance readability and establish visual hierarchy. This guide provides comprehensive instructions for implementing typography in your applications.

## Core Typography Principles

1. **Consistency**: Use defined typography classes rather than custom styles
2. **Hierarchy**: Establish clear content hierarchy through appropriate heading levels
3. **Readability**: Ensure text has sufficient contrast and appropriate sizing
4. **Spacing**: Maintain consistent spacing between text elements

## Heading Styles

Headings define the information hierarchy of your content. Use the appropriate heading level based on the content structure, not for styling purposes.

### Heading 1 (`.heading-1`)

The primary heading, used for page titles and major sections.

```html
<h1 class="heading-1">Page Title</h1>
```

**Specifications**:
- Font: Inter
- Weight: 500 (Medium)
- Size: 32px
- Line height: 40px

**Best Practices**:
- Use only once per page for the main title
- Keep concise (ideally under 60 characters)
- Do not use for decorative purposes

### Heading 2 (`.heading-2`)

Used for major section headings.

```html
<h2 class="heading-2">Section Title</h2>
```

**Specifications**:
- Font: Inter
- Weight: 500 (Medium)
- Size: 24px
- Line height: 32px
- Includes bottom padding: 16px (var(--space-4))

**Best Practices**:
- Use to divide content into major sections
- Maintain clear hierarchy with H1
- Use in card headers with the `card__title` class

### Heading 3 (`.heading-3`)

Used for subsection headings.

```html
<h3 class="heading-3">Subsection Title</h3>
```

**Specifications**:
- Font: Inter
- Weight: 500 (Medium)
- Size: 20px
- Line height: 28px

**Best Practices**:
- Use for subsections within major sections
- Can be used for card subheadings
- Effective for grouping related content

### Heading 4 (`.heading-4`)

Used for minor subsections or emphasized content.

```html
<h4 class="heading-4">Minor Subsection Title</h4>
```

**Specifications**:
- Font: Inter
- Weight: 500 (Medium)
- Size: 16px
- Line height: 24px

**Best Practices**:
- Use sparingly for lower-level headings
- Good for emphasizing content within subsections
- Avoid using more than 4 levels of headings in a single page

## Body Text Styles

Body text is used for the main content of your application. Choose the appropriate style based on the context and importance of the content.

### Standard Body Text (`.body-text`)

The default text style for most content.

```html
<p class="body-text">This is a standard paragraph of body text.</p>
```

**Specifications**:
- Font: Inter
- Weight: 400 (Regular)
- Size: 16px
- Line height: 24px

**Best Practices**:
- Use for most paragraph content
- Maintain proper spacing between paragraphs (var(--space-4))
- Avoid long paragraphs (aim for 3-5 sentences)

### Body Text Medium (`.body-text-medium`)

Medium weight variant for slightly emphasized body text.

```html
<p class="body-text-medium">This text has slightly more emphasis.</p>
```

**Specifications**:
- Font: Inter
- Weight: 500 (Medium)
- Size: 16px
- Line height: 24px

**Best Practices**:
- Use for medium emphasis without changing size
- Effective for section introductions
- Don't overuse - keep emphasis meaningful

### Small Body Text (`.small-body-text`)

Smaller text for secondary content.

```html
<p class="small-body-text">This is smaller text for less important information.</p>
```

**Specifications**:
- Font: Inter
- Weight: 400 (Regular)
- Size: 14px
- Line height: 20px

**Best Practices**:
- Use for secondary information, captions, or metadata
- Good for timestamps, helper text, or auxiliary information
- Maintain sufficient contrast for readability

### Small Body Text Medium (`.small-body-text-medium`)

Medium weight small text for emphasized secondary content.

```html
<p class="small-body-text-medium">This is emphasized smaller text.</p>
```

**Specifications**:
- Font: Inter
- Weight: 500 (Medium)
- Size: 14px
- Line height: 20px

**Best Practices**:
- Use for emphasized secondary content
- Good for small subheadings or emphasized metadata
- Combine with semantic elements when appropriate (e.g., `<strong>`)

### Badge Text (`.badge`)

Compact text style specifically designed for badges.

```html
<span class="badge badge-primary">
    <span class="badge-text">Badge Example</span>
</span>
```

**Specifications**:
- Font: Inter
- Weight: 500 (Medium)
- Size: 11px
- Line height: 16px
- Letter spacing: 1px

**Best Practices**:
- Use for status indicators, labels, and tags
- Keep text short and concise (1-2 words ideally)
- Combine with appropriate badge color variants (`badge-primary`, `badge-success`, etc.)
- Always wrap the text content in a `badge-text` span

## Link Styles

Links should be clearly distinguishable from surrounding text while maintaining design harmony. Choose the appropriate link class based on the context and content type.

### Common Link Characteristics

All links in the design system share these features:
- Underlined by default
- Blue color (var(--secondary-blue-500-base))
- Remove underline on hover
- Darker blue on hover (var(--secondary-blue-600))
- Smooth transition (0.2s ease)

### Link H3 (`.link-h3`)

Large link style for heading-level links.

```html
<a href="#" class="link-h3">Large Link Heading</a>
```

**Best Practices**:
- Use for navigation to major sections
- Effective for featured links
- Don't use for inline text links

### Link Body (`.link-body`)

Standard link style for body text content.

```html
<p class="body-text">This paragraph contains a <a href="#" class="link-body">standard link</a> within the text.</p>
```

**Best Practices**:
- Use for most inline text links
- Maintain context with surrounding text
- Don't modify the default styling

### Link Body Medium (`.link-body-medium`)

Medium weight link for body text that needs more emphasis.

```html
<p class="body-text">For important actions, use a <a href="#" class="link-body-medium">medium weight link</a>.</p>
```

**Best Practices**:
- Use for emphasized links or important actions
- Good for calls-to-action within body text
- Use sparingly to maintain impact

### Link Small Body Medium (`.link-small-body-medium`)

Medium weight link for small body text.

```html
<p class="small-body-text">See <a href="#" class="link-small-body-medium">more details</a> about this topic.</p>
```

**Best Practices**:
- Use in small body text contexts
- Great for "read more" or secondary actions
- Ensure sufficient contrast against background

## Text Utilities

Additional utility classes to modify text styling without changing semantic meaning.

### Font Weight Utilities

```html
<p class="body-text font-medium">Medium weight text (500)</p>
```

### Text Size Utilities

```html
<p class="text-small">Small text (14px)</p>
<p class="text-medium">Medium text (16px)</p>
<p class="text-large">Large text (20px)</p>
```

## Implementation Best Practices

### Semantic Structure

1. **Use Semantic HTML**: Match HTML elements to their semantic meaning:
   ```html
   <!-- CORRECT -->
   <h1 class="heading-1">Page Title</h1>
   <p class="body-text">Content paragraph</p>
   
   <!-- INCORRECT -->
   <div class="heading-1">Page Title</div>
   <div class="body-text">Content paragraph</div>
   ```

2. **Maintain Heading Hierarchy**: Don't skip heading levels:
   ```html
   <!-- CORRECT -->
   <h1 class="heading-1">Page Title</h1>
   <h2 class="heading-2">Section Title</h2>
   <h3 class="heading-3">Subsection Title</h3>
   
   <!-- INCORRECT -->
   <h1 class="heading-1">Page Title</h1>
   <h3 class="heading-3">Subsection Title</h3> <!-- Skipped h2 -->
   ```

### Handling Edge Cases

1. **Missing Typography Classes**: When you can't find the right typography class for your specific use case:
   ```html
   <!-- CORRECT -->
   <h2 style="font-size: 20px; font-weight: 500; color: var(--neutral-grey-800);">Custom Title</h2>
   
   <!-- INCORRECT -->
   <h2 class="heading-3">Custom Title</h2> <!-- Don't misuse existing classes -->
   ```

2. **Apply Styling to the Element**: Apply custom styling directly to the text element itself, not by misusing existing classes:
   ```html
   <!-- CORRECT -->
   <div style="font-family: Inter; font-size: 18px; font-weight: 500; line-height: 26px;">Custom Text</div>
   
   <!-- INCORRECT -->
   <div class="heading-4" style="font-size: 18px;">Custom Text</div> <!-- Don't mix classes with overrides -->
   ```

3. **Card Title Example**: For card titles, always use the `card__title` class with the h2 element:
   ```html
   <!-- CORRECT -->
   <h2 class="card__title">Card Title</h2>
   
   <!-- INCORRECT -->
   <h2 class="heading-2 card__title">Card Title</h2> <!-- Don't combine typography classes with component classes -->
   
   <!-- INCORRECT -->
   <h2 class="heading-2" style="font-size: 24px;">Card Title</h2> <!-- Don't use heading class when card__title is needed -->
   ```
   
   If you need a custom card title style that differs from the standard:
   ```html
   <!-- CORRECT -->
   <h2 class="card__title" style="color: var(--primary-blue-600);">Custom Colored Card Title</h2> <!-- Minor styling adjustment to card__title -->
   
   <!-- CORRECT - For completely custom card title -->
   <h2 style="font-size: 22px; font-weight: 500; margin-bottom: var(--space-4);">Completely Custom Card Title</h2>
   
   <!-- INCORRECT -->
   <h3 class="heading-2">Card Title</h3> <!-- Don't use wrong semantic element -->
   ```

4. **Preserve Semantic Meaning**: Maintain semantic correctness even when custom styling is needed:
   ```html
   <!-- CORRECT -->
   <h2 style="font-size: 22px; font-weight: 500;">Section Title</h2> <!-- Still using h2 for a section title -->
   
   <!-- INCORRECT -->
   <h3 class="heading-2">Section Title</h3> <!-- Don't use wrong semantic element to get styling -->
   ```

5. **Document Custom Implementations**: When creating custom typography styles for edge cases, document them for team reference.

### Spacing and Layout

1. **Consistent Paragraph Spacing**: Maintain consistent spacing between paragraphs:
   ```html
   <div style="display: flex; flex-direction: column; gap: var(--space-4);">
     <p class="body-text">First paragraph</p>
     <p class="body-text">Second paragraph</p>
   </div>
   ```

2. **Heading Bottom Spacing**: The `.heading-2` class already includes bottom padding of 16px (var(--space-4)). Don't add additional margin/padding:
   ```html
   <!-- CORRECT -->
   <h2 class="heading-2">Section Title</h2>
   <p class="body-text">Content follows immediately</p>
   
   <!-- INCORRECT -->
   <h2 class="heading-2" style="margin-bottom: var(--space-4);">Section Title</h2> <!-- Redundant spacing -->
   ```

3. **Text Container Width**: Ensure optimal reading width for body text (ideally 50-75 characters per line):
   ```html
   <div style="max-width: 680px;">
     <p class="body-text">Long-form content should be contained to ensure optimal line length for reading...</p>
   </div>
   ```

### Styling Considerations

1. **Color Contrast**: Ensure text meets accessibility contrast standards:
   - Use default text colors when possible
   - When using custom colors, verify 4.5:1 contrast ratio for normal text
   - Avoid placing text on busy backgrounds

2. **Font Customization**: Don't modify the core typography styles:
   ```html
   <!-- INCORRECT -->
   <h2 class="heading-2" style="font-size: 28px; font-weight: 500;">Custom Heading</h2> <!-- Breaking system -->
   ```

3. **Text Alignment**: Default to left alignment for most content (better readability):
   ```html
   <!-- Center alignment for specific elements only -->
   <h1 class="heading-1" style="text-align: center;">Centered Page Title</h1>
   ```

4. **Long Content**: Break long content into manageable sections:
   ```html
   <div class="card">
     <h2 class="card__title">Section Title</h2>
     <div class="card__content">
       <h3 class="heading-3">Subsection 1</h3>
       <p class="body-text">Content for subsection 1...</p>
       
       <h3 class="heading-3" style="margin-top: var(--space-6);">Subsection 2</h3>
       <p class="body-text">Content for subsection 2...</p>
     </div>
   </div>
   ```

## Common Anti-Patterns to Avoid

1. **Inconsistent Heading Hierarchy**: Using heading levels for visual styling rather than semantic structure
   
2. **Custom Typography Styles**: Creating custom text styles instead of using the design system classes
   
3. **Mixing Typography Systems**: Combining the design system typography with other font families or systems
   
4. **Text Formatting In HTML**: Using `<b>` or `<i>` tags instead of semantic `<strong>` and `<em>` with appropriate classes
   
5. **Small Text for Important Content**: Using small text for primary information that should be standard body text

6. **Overcomplicated Text Hierarchy**: Creating too many levels of visual hierarchy, making the page hard to scan

## Accessibility Considerations

1. **Proper Heading Structure**: Use headings to create a logical document outline for screen readers
   
2. **Sufficient Text Contrast**: Ensure text meets WCAG AA contrast requirements (4.5:1 for normal text)
   
3. **Responsive Text**: Verify text remains readable on all device sizes
   
4. **Line Height**: The design system's line heights are set for optimal readability
   
5. **Font Size**: Minimum 14px for body text to ensure readability
   
6. **Link Purpose**: Make link text descriptive of its destination or purpose

## Example Implementation

A complete example demonstrating proper typography implementation:

```html
<div class="page-container">
  <h1 class="heading-1">Product Documentation</h1>
  
  <div style="display: flex; flex-direction: column; gap: var(--space-6);">
    <div class="card">
      <h2 class="card__title">Getting Started</h2>
      <div class="card__content">
        <p class="body-text">Welcome to our product documentation. This guide will help you get started with our platform.</p>
        
        <h3 class="heading-3" style="margin-top: var(--space-6);">Prerequisites</h3>
        <p class="body-text">Before you begin, ensure you have the following:</p>
        <ul style="margin-top: var(--space-2);">
          <li class="body-text">An active account</li>
          <li class="body-text">Admin access to your organization</li>
          <li class="body-text">Latest browser version</li>
        </ul>
        
        <h3 class="heading-3" style="margin-top: var(--space-6);">Installation</h3>
        <p class="body-text">Follow these steps to install our software:</p>
        <p class="small-body-text" style="color: var(--neutral-grey-600); margin-top: var(--space-2);">Note: These instructions are for version 2.0 and above.</p>
        
        <h4 class="heading-4" style="margin-top: var(--space-4);">Step 1: Download</h4>
        <p class="body-text">Visit our <a href="#" class="link-body">downloads page</a> and select the appropriate version for your system.</p>
      </div>
    </div>
    
    <div class="card">
      <h2 class="card__title">Support</h2>
      <div class="card__content">
        <p class="body-text">Need help? Our support team is available 24/7.</p>
        <p class="body-text-medium" style="margin-top: var(--space-4);">Contact us via:</p>
        <p class="body-text" style="margin-top: var(--space-2);">Email: <a href="mailto:support@example.com" class="link-body">support@example.com</a></p>
        <p class="body-text">Phone: <a href="tel:+123456789" class="link-body">+1 (234) 567-89</a></p>
        <p class="small-body-text" style="color: var(--neutral-grey-600); margin-top: var(--space-4);">Response time: Within 24 hours</p>
      </div>
    </div>
  </div>
</div>
```

By following these guidelines, you'll create consistent, readable, and accessible typography throughout your application that aligns with the design system principles.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/keboola)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/keboola)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
