## adobe-spectrum-usage

> Styling and Design System Guidance

 # Adobe Spectrum Usage Guidelines

This rule provides standards for using Adobe React Spectrum components and Spectrum design tokens throughout the codebase.

<rule>
name: adobe_spectrum_usage
description: Guidelines for using Adobe React Spectrum components and Spectrum design tokens
filters:
  # Match React component files
  - type: file_extension
    pattern: "\\.(jsx|tsx)$"
  # Match CSS/SCSS files
  - type: file_extension
    pattern: "\\.(css|scss|less)$"
  # Match style objects in JS/TS files
  - type: content
    pattern: "(?:style|styles|css|className)\\s*=\\s*[{'\"]"

actions:
  - type: suggest
    conditions:
      # Detect hardcoded color values
      - pattern: "(?:color|background|border)\\s*:\\s*['\"#][a-fA-F0-9]{3,8}['\"]"
        message: "Use Spectrum color tokens instead of hardcoded colors"
      # Detect hardcoded spacing values
      - pattern: "(?:margin|padding|gap)\\s*:\\s*['\"]\\d+(?:px|rem|em)['\"]"
        message: "Use Spectrum spacing tokens instead of hardcoded spacing values"
      # Detect non-Spectrum components that have Spectrum equivalents
      - pattern: "<(?:Button|Input|Checkbox|Radio|Select|Slider|Switch|Textarea|Dialog|Modal|Tooltip|Menu|Tabs)\\s"
        message: "Use Adobe React Spectrum components instead of custom or third-party components"

    message: |
      ## Adobe React Spectrum Usage Guidelines

      ### Component Usage
      - Import and use React Spectrum components instead of custom implementations:
        ```jsx
        import { Button, TextField, Checkbox } from '@adobe/react-spectrum';
        
        // Instead of <button> or other custom buttons
        <Button variant="primary">Submit</Button>
        
        // Instead of <input>
        <TextField label="Name" />
        
        // Instead of custom checkboxes
        <Checkbox>Enable feature</Checkbox>
        ```

      ### Design Token Usage
      - Use Spectrum design tokens for consistent styling:
        ```jsx
        // For colors
        import { colorValue } from '@react-spectrum/color';
        
        // In styles
        const styles = {
          backgroundColor: 'var(--spectrum-global-color-gray-100)',
          color: 'var(--spectrum-semantic-negative-color-text)',
        };
        
        // For spacing
        const containerStyles = {
          padding: 'var(--spectrum-global-dimension-size-200)',
          gap: 'var(--spectrum-global-dimension-size-100)',
        };
        ```

      ### CSS Variables Pattern
      - Use CSS variables for Spectrum tokens in CSS files:
        ```css
        .container {
          background-color: var(--spectrum-global-color-gray-50);
          color: var(--spectrum-global-color-gray-900);
          padding: var(--spectrum-global-dimension-size-200);
        }
        ```

      ### Theme Awareness
      - Ensure components and styles respect the current Spectrum theme:
        ```jsx
        import { Provider, defaultTheme } from '@adobe/react-spectrum';
        
        <Provider theme={defaultTheme}>
          <YourApp />
        </Provider>
        ```

examples:
  - input: |
      // Bad: Hardcoded colors and custom components
      const styles = {
        backgroundColor: '#f5f5f5',
        color: '#333',
        padding: '16px',
      };
      
      return (
        <div style={styles}>
          <button className="custom-button">Submit</button>
          <input type="text" placeholder="Enter name" />
        </div>
      );
    output: |
      // Good: Using Spectrum tokens and components
      import { Flex, Button, TextField } from '@adobe/react-spectrum';
      
      const styles = {
        backgroundColor: 'var(--spectrum-global-color-gray-100)',
        color: 'var(--spectrum-global-color-gray-800)',
        padding: 'var(--spectrum-global-dimension-size-200)',
      };
      
      return (
        <Flex direction="column" gap="size-100" style={styles}>
          <Button variant="primary">Submit</Button>
          <TextField label="Enter name" />
        </Flex>
      );

  - input: |
      /* Bad: CSS with hardcoded values */
      .container {
        background-color: #ffffff;
        color: #333333;
        padding: 16px;
        margin: 8px;
        border: 1px solid #cccccc;
      }
    output: |
      /* Good: CSS with Spectrum tokens */
      .container {
        background-color: var(--spectrum-global-color-gray-50);
        color: var(--spectrum-global-color-gray-800);
        padding: var(--spectrum-global-dimension-size-200);
        margin: var(--spectrum-global-dimension-size-100);
        border: var(--spectrum-alias-border-size-thin) solid var(--spectrum-global-color-gray-300);
      }

metadata:
  priority: high
  version: 1.0
  tags:
    - design-system
    - styling
    - components
</rule>

---
> Source: [alexawilcox1120/TestingCursor](https://github.com/alexawilcox1120/TestingCursor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
