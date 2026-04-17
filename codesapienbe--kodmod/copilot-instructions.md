## kodmod

> These rules apply when developing the React Native mobile companion app for KodMod, extending the web IDE experience to native mobile platforms.

# React Native Rules for KodMod Mobile App (Cursor IDE)

## Overview
These rules apply when developing the React Native mobile companion app for KodMod, extending the web IDE experience to native mobile platforms.

## Project Structure for React Native

kodmod-mobile/
├── src/
│ ├── components/ # Reusable UI components
│ ├── screens/ # Screen components
│ ├── navigation/ # Navigation setup
│ ├── services/ # API calls and external services
│ ├── store/ # State management (Zustand)
│ ├── utils/ # Helper functions
│ ├── hooks/ # Custom React hooks
│ ├── types/ # TypeScript type definitions
│ └── constants/ # App constants and config
├── assets/ # Images, fonts, etc.
├── ios/ # iOS-specific code
├── android/ # Android-specific code
└── tests/ # Test files

## React Native Specific Rules

### Performance Optimization
- Use `React.memo` for expensive list items
- Implement `FlatList` for large datasets with `getItemLayout` when possible
- Use `InteractionManager.runAfterInteractions()` for heavy operations
- Lazy load screens and components
- Optimize images with appropriate formats and sizes
- Use Flipper for performance profiling

### Platform-Specific Code
- Use `Platform.select()` for platform-specific styling
- Prefer `Platform.OS` checks for logic differences
- Keep platform files organized: `component.ios.tsx`, `component.android.tsx`
- Test on both platforms regularly
- Handle safe area insets properly

### Navigation
- Use React Navigation 6+ with TypeScript
- Define navigation types strictly
- Use deep linking for editor states
- Implement proper back button handling on Android
- Cache navigation state for persistence

### State Management
- Use Zustand for global state (consistent with web version)
- Keep navigation state separate from business logic
- Implement optimistic updates for better UX
- Use React Query for server state management
- Persist critical state to AsyncStorage

### Code Editor Integration
- Use react-native-monaco-editor or similar
- Implement custom syntax highlighting if needed
- Handle keyboard events properly
- Support external keyboards
- Implement code completion and IntelliSense

### Mobile-Specific Features
- Implement haptic feedback for interactions
- Use biometric authentication when available
- Support device rotation and different screen sizes
- Implement proper keyboard avoiding behavior
- Add pull-to-refresh functionality

### Styling Guidelines
- Use StyleSheet.create() for all styles
- Implement consistent spacing using a design system
- Use flexbox for layouts (avoid absolute positioning)
- Support both light and dark themes
- Test on various screen densities

### File System Integration
- Use react-native-fs for file operations
- Implement proper file type associations
- Support importing files from device storage
- Handle file permissions correctly
- Cache files locally for offline editing

### AI Integration on Mobile
- Implement proper loading states for AI requests
- Handle network interruptions gracefully
- Cache AI responses for offline viewing
- Optimize API calls for mobile networks
- Implement request queuing for poor connections

### Testing Rules
- Write unit tests for utilities and pure functions
- Use React Native Testing Library for component tests
- Implement E2E tests with Detox
- Test on real devices, not just simulators
- Test offline functionality thoroughly

### Security Considerations
- Store sensitive data in Keychain/Keystore
- Validate all inputs from external sources
- Implement certificate pinning for API calls
- Use ProGuard/R8 for Android release builds
- Regular security audits of dependencies

### Accessibility
- Add accessibilityLabel to all interactive elements
- Support screen readers properly
- Implement proper focus management
- Test with TalkBack and VoiceOver
- Provide alternative text for images

### Code Quality
- Use TypeScript strict mode
- Implement proper error boundaries
- Log errors to crash reporting service
- Use ESLint with React Native specific rules
- Format code with Prettier consistently

### Build and Deployment
- Use Fastlane for iOS deployment
- Implement proper environment configurations
- Use CodePush for OTA updates
- Automate testing in CI/CD pipeline
- Monitor app performance with analytics

## Component Guidelines

### Screen Components
interface ScreenProps {
navigation: NavigationProp<any>;
route: RouteProp<any>;
}

const EditorScreen: React.FC<ScreenProps> = ({ navigation, route }) => {
// Screen implementation
};

### Custom Hooks
- Prefix custom hooks with "use"
- Keep hooks focused on single responsibility
- Return objects for multiple values
- Use TypeScript for hook parameters and returns
- Test hooks in isolation

### Service Integration
- Create service classes for API interactions
- Implement proper error handling
- Use interceptors for common operations
- Mock services for testing
- Handle authentication token refresh

### Constants and Configuration
- Group related constants together
- Use enums for related string constants
- Make configuration environment-specific
- Document all configuration options
- Validate configuration at startup

## Mobile App Features Roadmap

### Phase 1: Core Editor
- [ ] Basic code editor with syntax highlighting
- [ ] File browser and management
- [ ] Project creation and templates
- [ ] Basic AI integration
- [ ] Offline file editing

### Phase 2: Advanced Features
- [ ] Git integration
- [ ] Advanced AI features
- [ ] Collaborative editing
- [ ] Plugin system
- [ ] Cloud sync

### Phase 3: Platform Integration
- [ ] Share extension for importing files
- [ ] Shortcuts app integration (iOS)
- [ ] Widget support
- [ ] Apple Pencil support (iPad)
- [ ] Samsung S Pen support (Android)

## Performance Targets
- App startup time < 2 seconds
- Screen transitions < 300ms
- Code highlighting response < 100ms
- Memory usage < 150MB for typical projects
- Bundle size < 50MB for initial download

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/codesapienbe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
