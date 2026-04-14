## maxim

> - **Framework**: Expo SDK 50+ (Managed Workflow)


# Ultra-Premium Architecture Standards

## Core Requirements
- **Framework**: Expo SDK 50+ (Managed Workflow)
- **Language**: TypeScript (Strict)
- **Navigation**: Expo Router v3 (File-based routing)
- **Animations**: React Native Reanimated 3 + Moti
- **3D Globe**: @react-three/fiber + @react-three/drei/native
- **Maps**: @rnmapbox/maps
- **Blur**: expo-blur (for glassmorphism)
- **State**: Zustand (Global Store)
- **Icons**: Custom react-native-svg (NO generic icon libraries)

## Component Standards

### GlassCard Usage
- **Always use GlassCard instead of GlassView** for all glassmorphism effects
- **Props**: `elevated?: boolean`, `interactive?: boolean`, `intensity?: number`, `tint?: 'light' | 'dark' | 'default'`
- **Default intensity**: 30 for optimal blur effect
- **Default tint**: 'dark' for ultra-premium aesthetic
- **Accessibility**: Extends ViewProps - supports all accessibility props

### PremiumButton Usage
- **Always use PremiumButton instead of NeonButton** for all interactive elements
- **Props**: `variant?: 'primary' | 'secondary' | 'ghost'`, `size?: 'sm' | 'md' | 'lg'`
- **Micro-interactions**: Built-in scale animations (1 → 0.98 → 1)
- **Multisensory**: Automatic haptics + sound feedback
- **Accessibility**: Extends PressableProps - supports all accessibility props

### CustomIcon Usage
- **Always use CustomIcon instead of Ionicons or generic libraries**
- **Built with react-native-svg** using custom SVG paths
- **Active states**: Automatic glow effects and color changes
- **Available icons**: 'home', 'activity', 'location', 'profile', 'search', 'menu', 'chevronRight', 'settings'

## Design System Compliance

### Colors
- **Always use ds.colors tokens** - NO hardcoded colors
- **Primary**: ds.colors.primary (cyber blue)
- **Secondary**: ds.colors.secondary (neon green)
- **Background**: ds.colors.backgroundDeep (true black)

### Typography
- **Always use ds.typography tokens** for font sizes, weights, and families
- **Family**: ds.typography.family (Poppins)
- **Available sizes**: micro, caption, body, bodyLg, title, display, hero

### Spacing & Layout
- **Always use ds.spacing tokens** - NO magic numbers
- **Available**: xxs, xs, sm, md, lg, xl, xxl, xxxl
- **Border radius**: Always use ds.radius tokens

### Motion & Animation
- **Always use ds.motion tokens** for durations and easing
- **Entrance**: ds.motion.duration.entrance (360ms)
- **Exit**: ds.motion.duration.exit (220ms)
- **Micro**: ds.motion.duration.micro (140ms)

## Ultra-Premium Features

### Glassmorphism
- **All panels use GlassCard** with expo-blur intensity 30
- **Consistent border**: ds.colors.glassBorder
- **Elevated shadows**: ds.shadow.modern for elevated surfaces

### Micro-interactions
- **Every tappable element** must have scale animation
- **Haptic feedback**: tap() for selections, confirm() for actions
- **Sound feedback**: play('tapSoft') for micro, play('success') for confirm

### Noise Texture
- **Always include NoiseOverlay** with opacity ds.effects.noiseOpacity (0.03)
- **Position**: Absolute fill with pointerEvents="none"

## Code Quality Standards

### TypeScript
- **Strict mode enabled** - NO any types
- **Proper interfaces** for all component props
- **Accessibility props** supported on all interactive components

### ESLint
- **Zero warnings policy** - All warnings must be fixed
- **No unused imports** - Clean imports only
- **Proper prop spreading** for accessibility support

### File Structure
- **Components**: /src/components/ui/ for reusable components
- **3D**: /src/components/3d/ for globe and 3D elements
- **Maps**: /src/components/map/ for map components
- **Store**: /src/store/ for Zustand state management

## Migration Rules

### From Old Components
- **GlassView → GlassCard**: Update imports and props
- **NeonButton → PremiumButton**: Update variants and accessibility
- **Ionicons → CustomIcon**: Replace with custom SVG icons

### State Management
- **Use Zustand selectors** for optimized re-renders
- **DevTools enabled** for debugging
- **Type-safe interfaces** for all state slices

## Visual Requirements

### Globe & Background
- **Globe renders behind glassmorphism panels** with proper z-index
- **Background uses ds.colors.backgroundDeep** with noise overlay
- **Smooth cross-fade transitions** between globe and map states
- **Maintain 60+ FPS** on all interactions

### Layout Consistency
- **GlassCard styling** consistent across all screens
- **PremiumButton variants** used appropriately
- **CustomIcon active states** working correctly
- **Noise overlay** visible but not intrusive

## Testing Requirements

### Build Verification
- **TypeScript compilation**: tsc --noEmit must pass
- **ESLint compliance**: npx eslint with --max-warnings 0
- **Metro bundling**: Successful build with no errors

### Visual Testing
- **Web build**: Test at http://localhost:8082
- **Mobile testing**: Verify on iOS/Android if possible
- **Performance**: Globe maintains 60+ FPS
- **Accessibility**: All interactive elements properly labeled

## Enforcement

### Code Reviews
- **Check for GlassView usage** - should be replaced with GlassCard
- **Verify PremiumButton usage** - no NeonButton remnants
- **Validate ds token usage** - no hardcoded values
- **Confirm CustomIcon usage** - no generic icon libraries

### Automated Checks
- **TypeScript strict mode** prevents type errors
- **ESLint rules** enforce code quality
- **Build pipeline** ensures compilation success

## Future Considerations

### Performance Optimization
- **Monitor bundle size** with new components
- **Optimize re-renders** with proper selectors
- **Test on low-end devices** for smooth animations

### Accessibility Compliance
- **Screen reader support** with proper labels
- **High contrast mode** compatibility
- **Voice control navigation** support

These standards ensure the ultra-premium aesthetic is maintained consistently across the entire application while preserving performance and accessibility.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/deramatamara-lab) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
