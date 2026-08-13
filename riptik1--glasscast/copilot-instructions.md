## glasscast

> **Glasscast** is a production-ready minimal weather app built with SwiftUI for iOS 26, featuring:

# CLAUDE.md - Glasscast Weather App

## Project Overview

**Glasscast** is a production-ready minimal weather app built with SwiftUI for iOS 26, featuring:
- Email/password authentication via Supabase
- Current weather and 5-day forecast
- City search with favorites synced to Supabase
- Recently viewed cities (local storage)
- Temperature unit toggle (°C/°F)
- iOS 26 Liquid Glass design system
- Location-based weather
- Offline caching with SwiftData

## Architecture

### MVVM + Coordinator Pattern
- **Models**: Data structures in `Models/` directory
- **ViewModels**: Business logic and state management
  - `AuthViewModel` - Authentication state
  - `WeatherViewModel` - Weather data and forecast
  - `CitySearchViewModel` - City search, favorites, and recents
- **Views**: SwiftUI views organized by feature
- **Coordinator**: `AppCoordinator` handles navigation and routing
- **DependencyFactory**: Centralized dependency injection

### Key Design Principles

1. **Dependency Injection**: All dependencies injected via `DependencyFactory`
2. **Protocol-Oriented**: Services and use cases use protocols for testability
3. **Separation of Concerns**: Clear boundaries between layers
4. **Memory Safety**: All closures use `[weak self]` to prevent retain cycles
5. **Performance**: Request deduplication, caching, and offline support

## File Structure

```
Glasscast/
├── App/
│   ├── GlasscastApp.swift          # App entry point
│   └── ContentView.swift            # Root view with coordinator
├── Core/
│   └── DependencyFactory.swift     # Dependency injection container
├── Models/
│   ├── WeatherModels.swift          # Weather data models
│   └── CachedWeatherModels.swift    # SwiftData cache models
├── ViewModels/
│   ├── AuthViewModel.swift
│   ├── WeatherViewModel.swift
│   └── CitySearchViewModel.swift
├── Views/
│   ├── Auth/                        # Authentication screens
│   ├── Home/                        # Weather display
│   ├── Search/                      # City search
│   ├── Settings/                    # App settings
│   └── MainTabView.swift            # Tab navigation
├── Services/
│   ├── WeatherService.swift         # OpenWeatherMap API
│   ├── SupabaseService.swift        # Supabase auth & database
│   ├── CacheService.swift           # SwiftData caching
│   └── LocationService.swift        # CoreLocation wrapper
├── UseCases/
│   ├── WeatherUseCase.swift         # Weather business logic
│   └── FavoriteCitiesUseCase.swift # Favorites management
├── Network/
│   ├── HTTPClient.swift             # Network layer
│   ├── Endpoint.swift               # API endpoints
│   ├── HTTPMethod.swift             # HTTP methods
│   ├── NetworkError.swift           # Error types
│   └── Keychain.swift               # Secure storage
├── Config/
│   ├── AppConfig.swift              # Configuration management
│   └── Config.xcconfig              # Build settings
├── Utilities/
│   ├── Views/
│   │   ├── Coordinator/             # Navigation coordinator
│   │   └── CommonGlassView.swift    # Reusable glass components
│   └── ViewExtensions.swift         # Custom modifiers
└── Info.plist                       # App configuration
```

## API Integration

### OpenWeatherMap API
- **Base URL**: `https://api.openweathermap.org/data/2.5`
- **Endpoints**:
  - `/weather` - Current weather by lat/lon
  - `/forecast` - 5-day forecast by lat/lon
  - `/find` - City search with `type=like`
- **API Key**: Stored securely in Keychain
- **Units**: Metric (converted in UI)
- **Caching**: SwiftData with 24-hour TTL
- **Request Deduplication**: Prevents duplicate simultaneous requests

### Supabase
- **Authentication**: Email/password with session management
- **Database Table**: `favorite_cities`
  - Columns: `id`, `user_id`, `city_name`, `lat`, `lon`, `created_at`
  - RLS enabled: Users can only access their own favorites
- **Sync**: Favorites automatically sync to user account
- **Credentials**: Stored securely in Keychain

## Performance Optimizations

### 1. Request Deduplication
- `WeatherUseCase` tracks active requests by city+unit
- Multiple simultaneous requests for same data share the same task
- Prevents API rate limit issues and reduces network usage

### 2. Caching Strategy
- **SwiftData**: Persistent cache for weather data
- **Cache TTL**: 24 hours for weather, 6 hours for forecast
- **Offline Support**: App works offline using cached data
- **Cache Invalidation**: Automatic cleanup of old cache

### 3. Memory Management
- All closures use `[weak self]` to prevent retain cycles
- Tasks are properly cancelled when views disappear
- Location service continuations are cleaned up

### 4. Network Optimization
- Network status monitoring to avoid unnecessary requests
- Automatic fallback to cache when offline
- Request cancellation on view dismissal

## Configuration

### Required Environment Variables
Set these in Xcode Build Settings or `Info.plist`:
- `SUPABASE_URL` - Your Supabase project URL
- `SUPABASE_ANON_KEY` - Your Supabase anonymous key
- `WEATHER_API_KEY` - Your OpenWeatherMap API key

### Supabase Setup

1. Create `favorite_cities` table:
```sql
CREATE TABLE favorite_cities (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    city_name TEXT NOT NULL,
    lat DOUBLE PRECISION NOT NULL,
    lon DOUBLE PRECISION NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

ALTER TABLE favorite_cities ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view their own favorites"
    ON favorite_cities FOR SELECT
    USING (auth.uid() = user_id);

CREATE POLICY "Users can insert their own favorites"
    ON favorite_cities FOR INSERT
    WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can delete their own favorites"
    ON favorite_cities FOR DELETE
    USING (auth.uid() = user_id);
```

## Code Patterns

### Error Handling
```swift
do {
    let result = try await service.fetch()
    // Handle success
} catch {
    errorMessage = error.localizedDescription
    // Fallback to cache if available
}
```

### Loading States
```swift
@Published var isLoading: Bool = false

func loadData() async {
    await MainActor.run { isLoading = true }
    defer { Task { @MainActor in isLoading = false } }
    // Load data
}
```

### Memory Safety
```swift
Task { [weak self] in
    guard let self = self else { return }
    await self.doSomething()
}
```

### Dependency Injection
```swift
let viewModel = DependencyFactory.shared.makeWeatherViewModel()
```

## iOS 26 Liquid Glass Design

### Glass Effects
- Use `.glassCard()` modifier for card backgrounds
- `.ultraThinMaterial` with glass effects
- Gradient overlays for depth
- Smooth spring animations

### Color Scheme
- Background: Dark gradient (deep blue/purple)
- Glass cards: Ultra-thin material
- Accent: Blue to purple gradient (#0FA3A8)
- Text: White with varying opacity

## Testing

### Test Structure
- Unit tests for ViewModels
- Integration tests for UseCases
- Mock services for network layer
- Test coverage: 100%

### Running Tests
```bash
# Run all tests
xcodebuild test -scheme Glasscast -destination 'platform=iOS Simulator,name=iPhone 15'

# Run specific test
xcodebuild test -scheme Glasscast -only-testing:GlasscastTests/WeatherViewModelTests
```

## Development Workflow

### When Adding Features
1. Follow MVVM + Coordinator pattern
2. Use `@MainActor` for UI-related code
3. Implement proper error handling
4. Add loading states
5. Use dependency injection
6. Add `[weak self]` in closures
7. Write tests

### When Debugging
- Check `AppConfig` for missing API keys
- Verify Supabase RLS policies
- Check network connectivity
- Review cache state in SwiftData
- Check for retain cycles with Instruments

### Code Review Checklist
- [ ] No retain cycles (all closures use `[weak self]`)
- [ ] Proper error handling
- [ ] Loading states implemented
- [ ] Dependencies injected via factory
- [ ] Navigation uses coordinator
- [ ] Tests written and passing
- [ ] Memory leaks checked

---
> Source: [Riptik1/Glasscast](https://github.com/Riptik1/Glasscast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
