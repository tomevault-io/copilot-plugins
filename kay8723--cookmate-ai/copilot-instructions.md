## cookmate-ai

> Before diving into the best practices, please note that you may need to adapt the globs depending on your project's specific file structure and requirements.

Before diving into the best practices, please note that you may need to adapt the globs depending on your project's specific file structure and requirements.

---
name: react-native-best-practices.mdc
description: Best practices for React Native mobile development
globs: **/*.{ts,tsx,js,jsx}
---

- Use `useMemo` and `useCallback` to optimize performance
- Implement proper state management with Context API or Redux
- Utilize `FlatList` for efficient rendering of lists
- Ensure platform-specific code is handled with `Platform.select`
- Use `Dimensions` API for responsive layouts

---
name: typescript-best-practices.mdc
description: TypeScript coding standards and type safety guidelines
globs: **/*.{ts,tsx}
---

- Enable `strict` mode in `tsconfig.json`
- Use type aliases for complex types
- Leverage union types for better type checking
- Implement generics for reusable components
- Use `readonly` for immutable properties

---
name: expo-best-practices.mdc
description: Best practices for using Expo in React Native projects
globs: **/*.{ts,tsx,js,jsx}
---

- Use Expo's managed workflow for easier app deployment
- Leverage Expo's built-in modules for common functionalities
- Implement Expo's push notification service for better user engagement
- Use Expo's asset management for efficient resource handling
- Regularly update Expo SDK for the latest features and security

---
name: supabase-best-practices.mdc
description: Best practices for using Supabase as a backend and database
globs: **/*.{ts,tsx,js,jsx}
---

- Use Supabase's real-time capabilities for live updates
- Implement proper error handling for database operations
- Utilize Supabase's authentication for secure user management
- Optimize queries with Supabase's filtering and sorting options
- Use Supabase's storage for efficient file management

---
name: react-native-paper-best-practices.mdc
description: Best practices for using React Native Paper for UI components
globs: **/*.{ts,tsx,js,jsx}
---

- Use React Native Paper's theming system for consistent styling
- Implement custom themes to match your app's design
- Utilize Paper's built-in components for faster development
- Ensure accessibility by using Paper's accessibility props
- Optimize performance by using `memo` on Paper components

---
name: google-gemini-api-best-practices.mdc
description: Best practices for using Google Gemini API for AI processing
globs: **/*.{ts,tsx,js,jsx}
---

- Implement proper error handling for API requests
- Use caching to reduce API calls and improve performance
- Monitor and handle API rate limits effectively
- Implement fallback mechanisms for when the API is unavailable
- Ensure user data privacy when using the API

---
name: deepseek-best-practices.mdc
description: Best practices for using DeepSeek for AI processing
globs: **/*.{ts,tsx,js,jsx}
---

- Optimize API calls by batching requests where possible
- Implement robust error handling and retries
- Use DeepSeek's capabilities for complex AI tasks
- Ensure compliance with DeepSeek's usage policies
- Monitor API usage to stay within rate limits

---
name: react-native-animations-best-practices.mdc
description: Best practices for using React Native Animations
globs: **/*.{ts,tsx,js,jsx}
---

- Use `Animated` API for smooth animations
- Implement `useNativeDriver` for better performance
- Optimize animations by using `interpolate` for value mapping
- Use `LayoutAnimation` for smooth list updates
- Ensure animations are accessible and don't cause motion sickness

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kay8723) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
