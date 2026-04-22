## tetra

> Always respond in English


Always respond in English
 
1. HTML Rules
 
- Do not write CSS inside HTML files.
- Do not write JavaScript inside HTML files (avoid  and  tags in HTML).
- Maintain separation of concerns:
 
2. HTML for structure.
 
- CSS for styling.
- JavaScript for behavior.
- CSS Rules
 
3. Write CSS in separate .css files.
 
- Avoid inline CSS (e.g., style="color: red;").
- Use class names and IDs properly (semantic naming).
 
4. JavaScript Rules
 
- Write JavaScript in separate .js files.
- Use modern JavaScript (ES6+) features whenever possible.
- Follow clean code principles (functions, naming, etc.).
 
5. Language Rules
 
- All text content in HTML, CSS, and JavaScript must be written in English.
- Comments should also be in English.


You are an expert in PHP development.
  Code Style and Structure
  - Write concise, technical PHP code with accurate examples.
  - Use functional and declarative programming patterns; avoid classes.
  - Prefer iteration and modularization over code duplication.
  - Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError).
  - Structure files: exported component, subcomponents, helpers, static content, types.
  - Follow PHP's official documentation for setting up and configuring your projects: https://docs.php.net/
  Naming Conventions
  - Use lowercase with dashes for directories (e.g., controllers/auth.php).
  - Favor named exports for controllers.
  PHP Usage
  - Use PHP for all code; prefer functions over classes.
  - Avoid enums; use maps instead.
  - Use functional programming patterns.
  - Use strict mode in PHP for better type safety.
  Syntax and Formatting
  - Use the "function" keyword for pure functions.
  - Avoid unnecessary curly braces in conditionals; use concise syntax for simple statements.
  - Use declarative PHP.
  - Use Prettier for consistent code formatting.
  UI and Styling
  - Use PHP's built-in components for common UI patterns and layouts.
  - Implement responsive design with Flexbox and PHP's useWindowDimensions for screen size adjustments.
  - Use styled-components or Tailwind CSS for component styling.
  - Implement dark mode support using PHP's useColorScheme.
  - Ensure high accessibility (a11y) standards using ARIA roles and native accessibility props.
  - Leverage react-native-reanimated and react-native-gesture-handler for performant animations and gestures.
  Performance Optimization
  - Optimize images: use WebP format where supported, include size data, implement lazy loading with PHP-image.
  - Profile and monitor performance using PHP's built-in tools and debugging features.
  Error Handling and Validation
  - Implement proper error logging using Sentry or a similar service.
  - Prioritize error handling and edge cases:
    - Handle errors at the beginning of functions.
    - Use early returns for error conditions to avoid deeply nested if statements.
    - Avoid unnecessary else statements; use if-return pattern instead.
    - Implement global error boundaries to catch and handle unexpected errors.
  - Use expo-error-reporter for logging and reporting errors in production.
  Testing
  - Write unit tests using PHPUnit.
  - Implement integration tests for critical user flows using Detox.
  - Use Expo's testing tools for running tests in different environments.
  - Consider snapshot testing for components to ensure UI consistency.
  Security
  - Sanitize user inputs to prevent XSS attacks.
  - Use react-native-encrypted-storage for secure storage of sensitive data.
  - Ensure secure communication with APIs using HTTPS and proper authentication.
  - Use PHP's Security guidelines to protect your app: https://docs.php.net/
  Internationalization (i18n)
  - Use react-native-i18n or expo-localization for internationalization and localization.
  - Support multiple languages and RTL layouts.
  - Ensure text scaling and font adjustments for accessibility.
  Key Conventions
  1. Rely on PHP's managed workflow for streamlined development and deployment.
  2. Prioritize Mobile Web Vitals (Load Time, Jank, and Responsiveness).
  3. Use PHP's constants for managing environment variables and configuration.
  4. Use PHP's permissions to handle device permissions gracefully.
  5. Implement PHP's updates for over-the-air (OTA) updates.
  6. Follow PHP's best practices for app deployment and publishing: https://docs.php.net/
  7. Ensure compatibility with iOS and Android by testing extensively on both platforms.
  API Documentation
  - Use PHP's official documentation for setting up and configuring your projects: https://docs.php.net/
  Refer to PHP's documentation for detailed information on Views, Blueprints, and Extensions for best practices.
    

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IuriGuerreiro) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
