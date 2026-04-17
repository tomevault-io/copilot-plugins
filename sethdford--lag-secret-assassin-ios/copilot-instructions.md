## lag-secret-assassin-ios

> // ✅ Model: Data structures and business entities

# iOS Architecture & Design Patterns

## **MVVM-C Architecture Implementation**

### Core Components Structure
```swift
// ✅ Model: Data structures and business entities
struct Game: Codable, Identifiable {
    let id: UUID
    let name: String
    let status: GameStatus
    let players: [Player]
    let boundaries: GameBoundary
}

// ✅ View: SwiftUI views focused on presentation
struct GameDetailView: View {
    @StateObject private var viewModel: GameDetailViewModel
    let coordinator: GameCoordinator
    
    var body: some View {
        // UI implementation only
    }
}

// ✅ ViewModel: Business logic with @MainActor
@MainActor
final class GameDetailViewModel: ObservableObject {
    @Published var game: Game?
    @Published var isLoading = false
    @Published var error: Error?
    
    private let gameService: GameServiceProtocol
    
    func loadGame() async {
        // Business logic implementation
    }
}

// ✅ Coordinator: Navigation and flow control
final class GameCoordinator: ObservableObject {
    @Published var path = NavigationPath()
    
    func showGameDetail(gameId: String) {
        path.append(GameDestination.detail(gameId: gameId))
    }
}
```

## **Dependency Injection Pattern**

### Singleton Services with DI Registration
```swift
// ✅ DO: Implement singleton pattern for services
final class AuthenticationService: AuthenticationServiceProtocol {
    static let shared = AuthenticationService()
    private init() {}
    
    // DI Container registration
    static func register(in container: Container) {
        container.register(AuthenticationServiceProtocol.self) { _ in
            AuthenticationService.shared
        }.inObjectScope(.container)
    }
}

// ✅ DO: Use protocol-based dependency injection
protocol AuthenticationServiceProtocol {
    func authenticate(credentials: LoginCredentials) async throws -> User
    func logout() async
}

// ✅ DO: Inject dependencies through initializers
final class LoginViewModel: ObservableObject {
    private let authService: AuthenticationServiceProtocol
    
    init(authService: AuthenticationServiceProtocol = AuthenticationService.shared) {
        self.authService = authService
    }
}
```

### DI Container Setup
```swift
final class DIContainer {
    static let shared = DIContainer()
    private let container = Container()
    
    init() {
        registerServices()
    }
    
    private func registerServices() {
        // Core services
        AuthenticationService.register(in: container)
        LocationService.register(in: container)
        GameService.register(in: container)
        
        // ViewModels
        container.register(LoginViewModel.self) { resolver in
            LoginViewModel(authService: resolver.resolve(AuthenticationServiceProtocol.self)!)
        }
    }
    
    func resolve<T>(_ type: T.Type) -> T {
        return container.resolve(type)!
    }
}
```

## **SwiftUI Best Practices**

### State Management
```swift
// ✅ DO: Choose appropriate property wrappers
struct PlayerProfileView: View {
    @State private var isEditing = false              // Local view state
    @Binding var player: Player                       // Two-way binding
    @StateObject private var viewModel = ProfileViewModel() // Owned object
    @ObservedObject var gameState: GameStateManager   // External object
    @EnvironmentObject var auth: AuthenticationService // Injected dependency
    @AppStorage("theme") var theme = "dark"           // Persistent storage
}

// ✅ DO: Use @MainActor for ViewModels
@MainActor
final class ProfileViewModel: ObservableObject {
    @Published var isLoading = false
    @Published var error: Error?
    
    func updateProfile() async {
        // Automatically runs on main thread
    }
}
```

### View Composition
```swift
// ✅ DO: Break down complex views
struct GameMapView: View {
    @StateObject private var viewModel = GameMapViewModel()
    
    var body: some View {
        ZStack {
            MapBackgroundView()
            PlayerAnnotationsView(players: viewModel.players)
            SafeZoneOverlayView(zones: viewModel.safeZones)
            GameHUDView(gameState: viewModel.gameState)
        }
        .task {
            await viewModel.startTracking()
        }
    }
}

// ✅ DO: Create reusable components
struct PlayerAnnotationView: View {
    let player: Player
    let isTarget: Bool
    
    var body: some View {
        ZStack {
            Circle()
                .fill(isTarget ? Color.red : Color.blue)
                .frame(width: 40, height: 40)
            
            Image(systemName: "person.fill")
                .foregroundColor(.white)
        }
        .scaleEffect(isTarget ? 1.2 : 1.0)
        .animation(.spring(), value: isTarget)
    }
}
```

## **Error Handling Architecture**

### Structured Error Types
```swift
// ✅ DO: Create comprehensive error hierarchies
protocol AppError: LocalizedError {
    var userMessage: String { get }
    var isRetryable: Bool { get }
}

enum AuthenticationError: AppError {
    case invalidCredentials
    case networkError(Error)
    case serverError(statusCode: Int)
    
    var userMessage: String {
        switch self {
        case .invalidCredentials:
            return "Invalid email or password"
        case .networkError:
            return "Network connection error"
        case .serverError(let code):
            return "Server error (\(code))"
        }
    }
    
    var isRetryable: Bool {
        switch self {
        case .networkError, .serverError:
            return true
        case .invalidCredentials:
            return false
        }
    }
}
```

### Global Error Handling
```swift
@MainActor
final class ErrorHandler: ObservableObject {
    @Published var currentError: AppError?
    @Published var showError = false
    
    func handle(_ error: Error) {
        if let appError = error as? AppError {
            currentError = appError
        } else {
            currentError = UnknownError(underlying: error)
        }
        showError = true
    }
}
```

## **Async/Await Patterns**

### Modern Concurrency
```swift
// ✅ DO: Use async/await for asynchronous operations
@MainActor
class GameViewModel: ObservableObject {
    @Published var gameState: GameState = .loading
    
    func loadGame(id: String) async {
        do {
            let game = try await gameService.fetchGame(id: id)
            self.gameState = .loaded(game)
        } catch {
            self.gameState = .error(error)
        }
    }
}

// ✅ DO: Use TaskGroup for parallel operations
func fetchGameData() async throws -> GameData {
    try await withThrowingTaskGroup(of: Any.self) { group in
        group.addTask { try await self.fetchPlayers() }
        group.addTask { try await self.fetchBoundaries() }
        group.addTask { try await self.fetchSafeZones() }
        
        // Collect and combine results
        var players: [Player] = []
        var boundaries: GameBoundary?
        var safeZones: [SafeZone] = []
        
        for try await result in group {
            // Process results
        }
        
        return GameData(players: players, boundaries: boundaries!, safeZones: safeZones)
    }
}
```

## **Memory Management**

### Preventing Retain Cycles
```swift
// ✅ DO: Use weak references in closures
class MapViewModel: ObservableObject {
    func startLocationUpdates() {
        locationService.startUpdates { [weak self] location in
            self?.processLocation(location)
        }
    }
}

// ✅ DO: Implement proper cleanup
class LocationTracker {
    private var timer: Timer?
    
    deinit {
        timer?.invalidate()
        NotificationCenter.default.removeObserver(self)
    }
}
```

## **Protocol-Oriented Design**

### Service Protocols
```swift
// ✅ DO: Design with protocols for flexibility
protocol LocationServiceProtocol {
    var currentLocation: CLLocation? { get }
    func startTracking() async
    func stopTracking()
    func requestPermission() async -> LocationPermissionResult
}

protocol GameServiceProtocol {
    func fetchGame(id: String) async throws -> Game
    func createGame(_ game: Game) async throws -> Game
    func joinGame(id: String) async throws
}

// ✅ DO: Use protocol composition
protocol Trackable {
    var id: UUID { get }
    var location: CLLocationCoordinate2D { get }
    var lastUpdated: Date { get }
}

protocol GameParticipant: Trackable {
    var name: String { get }
    var status: PlayerStatus { get }
}
```

## **Repository Pattern**

### Data Layer Abstraction
```swift
// ✅ DO: Implement repository pattern
protocol GameRepositoryProtocol {
    func fetchGame(id: String) async throws -> Game
    func saveGame(_ game: Game) async throws
    func deleteGame(id: String) async throws
}

final class GameRepository: GameRepositoryProtocol {
    private let remoteDataSource: GameRemoteDataSourceProtocol
    private let localDataSource: GameLocalDataSourceProtocol
    
    func fetchGame(id: String) async throws -> Game {
        // Try local first, fallback to remote
        if let localGame = try? await localDataSource.fetchGame(id: id) {
            return localGame
        }
        
        let remoteGame = try await remoteDataSource.fetchGame(id: id)
        try? await localDataSource.saveGame(remoteGame)
        return remoteGame
    }
}
```

## **Testing Architecture**

### Testable Design
```swift
// ✅ DO: Design for testability
final class MockGameService: GameServiceProtocol {
    var stubbedGame: Game?
    var stubbedError: Error?
    
    func fetchGame(id: String) async throws -> Game {
        if let error = stubbedError {
            throw error
        }
        return stubbedGame ?? Game.fixture()
    }
}

// ✅ DO: Use dependency injection for testing
final class GameViewModelTests: XCTestCase {
    private var sut: GameViewModel!
    private var mockGameService: MockGameService!
    
    override func setUp() {
        mockGameService = MockGameService()
        sut = GameViewModel(gameService: mockGameService)
    }
}
```

## **Performance Considerations**

### Lazy Loading
```swift
// ✅ DO: Use lazy properties for expensive operations
lazy var dateFormatter: DateFormatter = {
    let formatter = DateFormatter()
    formatter.dateStyle = .medium
    formatter.timeStyle = .short
    return formatter
}()

// ✅ DO: Implement lazy collections in SwiftUI
struct GameListView: View {
    var body: some View {
        LazyVStack {
            ForEach(games) { game in
                GameRowView(game: game)
            }
        }
    }
}
```

## **Development Mode Protection**

### Debug-Only Features
```swift
// ✅ DO: Add development mode safeguards
#if DEBUG
extension AuthenticationService {
    func checkStoredCredentials() async {
        // Use mock user in development
        currentUser = User.mockUser
        isAuthenticated = true
        return
    }
}
#endif

// ✅ DO: Conditional compilation for debug features
#if DEBUG
struct DebugMenuView: View {
    var body: some View {
        VStack {
            Button("Reset User Data") { /* debug action */ }
            Button("Simulate Network Error") { /* debug action */ }
        }
    }
}
#endif
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sethdford) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
