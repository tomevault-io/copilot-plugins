## coordinatortemplate

> Generate a new iOS app project with Coordinator + MVVM + DI architecture. Creates full project skeleton with navigation, networking, auth, and DI container.


# ios-coordinator

Generate a complete iOS project skeleton using Coordinator + MVVM + DI architecture.

## Usage

`$ARGUMENTS` is the **project name**. Use it as:
- Root directory name
- Xcode target name and sources directory name
- Swift module name (struct name for `@main` App: `${ARGUMENTS}App`)
- Bundle identifier prefix: `com.$ARGUMENTS_LOWER` (lowercased)
- UserDefaults suite key: `com.$ARGUMENTS_LOWER.user-defaults` (lowercased)
- File headers use `$ARGUMENTS` as the project name
- Copyright: `Copyright © Yevhenii Kalashnikov. All rights reserved.`

If `$ARGUMENTS` is empty, ask the user for a project name before proceeding.

Replace all occurrences of `$ARGUMENTS` in the generated code with the actual project name. Replace `$ARGUMENTS_LOWER` with the lowercased version.

---

## Step 1: Create Directory Structure

```
$ARGUMENTS/
├── project.yml
├── Packages/
│   └── Coordinator/
│       ├── Package.swift
│       └── Sources/
│           ├── Coordinator.swift
│           ├── CoordinatorRoot.swift
│           ├── CoordinatorBranch.swift
│           └── Destination.swift
└── $ARGUMENTS/
    ├── ${ARGUMENTS}App.swift
    ├── Core/
    │   ├── Context.swift
    │   ├── ServiceProvider.swift
    │   ├── Session/
    │   │   ├── Session.swift
    │   │   └── SessionProvider.swift
    │   ├── Networking/
    │   │   ├── NetworkingService.swift
    │   │   ├── AnyMutation.swift
    │   │   ├── NetworkingServiceError.swift
    │   │   └── HttpMethod.swift
    │   ├── Helpers/
    │   │   ├── Keychain.swift
    │   │   ├── UserDefault.swift
    │   │   ├── UserDefaults+.swift
    │   │   ├── DataCache.swift
    │   │   ├── Codable+.swift
    │   │   ├── Error+.swift
    │   │   └── URL+.swift
    │   ├── Services/
    │   │   └── UserSettings.swift
    │   └── Models/
    │       ├── User.swift
    │       ├── APISession.swift
    │       ├── APIError.swift
    │       └── Mutations.swift
    └── UI/
        └── Interfaces/
            ├── MainView.swift
            ├── Tabs/
            │   └── TabsView.swift
            ├── Auth/
            │   ├── Login/
            │   │   ├── LoginView.swift
            │   │   ├── LoginViewModel.swift
            │   │   └── LoginViewBuilder.swift
            │   └── SignUp/
            │       ├── SignUpView.swift
            │       ├── SignUpViewModel.swift
            │       └── SignUpViewBuilder.swift
            ├── Home/
            │   ├── HomeView.swift
            │   ├── HomeViewModel.swift
            │   └── HomeViewBuilder.swift
            ├── Detail/
            │   ├── DetailView.swift
            │   ├── DetailViewModel.swift
            │   └── DetailViewBuilder.swift
            ├── Profile/
            │   ├── ProfileView.swift
            │   ├── ProfileViewModel.swift
            │   └── ProfileViewBuilder.swift
            └── Settings/
                ├── SettingsView.swift
                ├── SettingsViewModel.swift
                └── SettingsViewBuilder.swift
```

---

## Step 2: Generate Files

Every Swift file starts with this header:
```swift
//
//  FileName.swift
//  $ARGUMENTS
//
//  Copyright © Yevhenii Kalashnikov. All rights reserved.
//
```

Generate each file below with **exact** content. Replace `$ARGUMENTS` with the project name.

---

### 2.1 Coordinator SPM Package

#### `Packages/Coordinator/Package.swift`

```swift
// swift-tools-version: 6.0
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "Coordinator",
    platforms: [.iOS(.v17)],
    products: [
        .library(
            name: "Coordinator",
            targets: ["Coordinator"]
        ),
    ],
    dependencies: [
        // Dependencies can be added here if needed
    ],
    targets: [
        .target(
            name: "Coordinator",
            dependencies: [
                // Dependencies can be added here if needed
            ],
            path: "Sources",
            resources: [
                // Resources can be added here if needed
            ]
        ),
    ]
)
```

#### `Packages/Coordinator/Sources/Coordinator.swift`

```swift
import SwiftUI

@MainActor
final public class Coordinator<Context: Sendable>: ObservableObject {
    public let context: Context
    private weak var rootCoordinator: Coordinator?

    @Published public var path = NavigationPath()
    @Published public var sheet: Destination<Context>?
    @Published public var fullScreenCover: Destination<Context>?

    public init(context: Context) {
        self.context = context
    }

    public init(rootCoordinator: Coordinator) {
        context = rootCoordinator.context
        self.rootCoordinator = rootCoordinator
    }

    public func push(_ destination: Destination<Context>) {
        path.append(destination)
    }

    public func present(sheet destination: Destination<Context>) {
        sheet = destination
    }

    public func present(fullScreenCover destination: Destination<Context>) {
        fullScreenCover = destination
    }

    public func pop() {
        guard !path.isEmpty else { return }
        path.removeLast()
    }

    public func popToRoot() {
        path.removeLast(path.count)
    }

    /// Dismisses the sheet or fullScreenCover that presented this coordinator's CoordinatorBranch.
    /// Also clears navigation path and any nested presentations on this coordinator.
    public func dismiss() {
        popToRoot()
        sheet = nil
        fullScreenCover = nil

        if rootCoordinator?.sheet != nil {
            rootCoordinator?.sheet = nil
        } else if rootCoordinator?.fullScreenCover != nil {
            rootCoordinator?.fullScreenCover = nil
        }
    }

    /// Dismisses the sheet on this coordinator.
    /// Pass `withRoot: true` to also dismiss the root coordinator's sheet.
    public func dismissSheet(withRoot: Bool = false) {
        if sheet != nil {
            sheet = nil
        }
        if withRoot {
            rootCoordinator?.sheet = nil
        }
    }

    /// Dismisses the fullScreenCover on this coordinator.
    /// Pass `withRoot: true` to also dismiss the root coordinator's fullScreenCover.
    public func dismissFullScreenCover(withRoot: Bool = false) {
        if fullScreenCover != nil {
            fullScreenCover = nil
        }
        if withRoot {
            rootCoordinator?.fullScreenCover = nil
        }
    }

    @ViewBuilder public func build(destination: Destination<Context>) -> some View {
        destination.view(for: self)
    }
}
```

#### `Packages/Coordinator/Sources/CoordinatorRoot.swift`

```swift
import SwiftUI

/// Creates an independent coordinator and its full navigation stack.
///
/// Use this as the **top-level** entry point for any navigation tree:
/// tabs, login, onboarding — anything that owns its own navigation.
///
///     CoordinatorRoot(context: context, destination: .home)
///
public struct CoordinatorRoot<Context: AnyObject & Sendable>: View, @MainActor Equatable {

    public static func == (lhs: Self, rhs: Self) -> Bool {
        lhs.context === rhs.context
    }

    let context: Context
    @StateObject private var coordinator: Coordinator<Context>
    private let destination: @MainActor () -> Destination<Context>

    public init(context: Context,
                @_implicitSelfCapture destination: @MainActor @autoclosure @escaping () -> Destination<Context>) {

        self.context = context
        self.destination = destination
        _coordinator = StateObject(wrappedValue: .init(context: context))
    }

    public var body: some View {
        NavigationStack(path: $coordinator.path) {
            coordinator.build(destination: destination())
                .navigationDestination(for: Destination<Context>.self) { destination in
                    coordinator.build(destination: destination)
                        .toolbarRole(.editor)
                }
                .sheet(item: $coordinator.sheet) { destination in
                    coordinator.build(destination: destination)
                }
                .fullScreenCover(item: $coordinator.fullScreenCover) { destination in
                    coordinator.build(destination: destination)
                }
        }
        .environmentObject(coordinator)
    }
}
```

#### `Packages/Coordinator/Sources/CoordinatorBranch.swift`

```swift
import SwiftUI

/// Creates a child coordinator linked to a parent and its own navigation stack.
///
/// Use this inside **sheet** or **fullScreenCover** content when the presented
/// screen needs its own push/pop navigation. The child coordinator can dismiss
/// itself back to the parent via `coordinator.dismiss()`.
///
///     .init(id: "profile") {
///         CoordinatorBranch(rootCoordinator: $0) {
///             ProfileView(viewModel: .init(context: $0.context))
///         }
///     }
///
public struct CoordinatorBranch<V: View, Context: Sendable>: View {
    @StateObject private var coordinator: Coordinator<Context>

    private var build: (Coordinator<Context>) -> V

    public init(rootCoordinator: Coordinator<Context>, @ViewBuilder build: @escaping (Coordinator<Context>) -> V) {
        let coordinator = Coordinator(rootCoordinator: rootCoordinator)
        _coordinator = StateObject(wrappedValue: coordinator)
        self.build = build
    }

    public var body: some View {
        NavigationStack(path: $coordinator.path) {
            build(coordinator)
                .navigationDestination(for: Destination<Context>.self) { destination in
                    coordinator.build(destination: destination)
                        .toolbarRole(.editor)
                }
                .sheet(item: $coordinator.sheet) { destination in
                    coordinator.build(destination: destination)
                }
                .fullScreenCover(item: $coordinator.fullScreenCover) { destination in
                    coordinator.build(destination: destination)
                }
        }
        .environmentObject(coordinator)
    }
}
```

#### `Packages/Coordinator/Sources/Destination.swift`

```swift
import SwiftUI

@MainActor
public final class Destination<Context: Sendable>: Identifiable {

    public let id: String
    private var view: (Coordinator<Context>) -> AnyView

    func view(for coordinator: Coordinator<Context>) -> some View {
        view(coordinator)
    }

    public init<V: View>(id: String, @ViewBuilder buildView: @escaping (Coordinator<Context>) -> V) {
        self.id = id
        view = { AnyView(buildView($0)) }
    }
}

extension Destination: @preconcurrency Hashable {

    nonisolated public func hash(into hasher: inout Hasher) {
        hasher.combine(id)
    }

    nonisolated public static func == (lhs: Destination, rhs: Destination) -> Bool {
        lhs.id == rhs.id
    }
}
```

---

### 2.2 Core — Helpers

#### `$ARGUMENTS/Core/Helpers/Keychain.swift`

```swift
import Foundation
import Security

@propertyWrapper
struct Keychain<T: Codable> {
    private let key: String
    private let service: String
    private var cachedValue: T?
    private var hasCachedValue = false

    init(key: String, service: String = Bundle.main.bundleIdentifier ?? "DefaultService") {
        self.key = key
        self.service = service
    }

    var wrappedValue: T? {
        get {
            if hasCachedValue { return cachedValue }
            guard let data = getKeychainData() else { return nil }
            do {
                return try JSONDecoder().decode(T.self, from: data)
            } catch {
                assertionFailure("Failed to decode data from keychain: \(error)")
                return nil
            }
        }
        set {
            cachedValue = newValue
            hasCachedValue = true

            if let newValue {
                do {
                    let data = try JSONEncoder().encode(newValue)
                    saveToKeychain(data: data)
                } catch {
                    assertionFailure("Failed to encode data for keychain: \(error)")
                }
            } else {
                deleteFromKeychain()
            }
        }
    }

    // MARK: - Keychain interaction

    private func getKeychainData() -> Data? {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key,
            kSecReturnData as String: true,
            kSecMatchLimit as String: kSecMatchLimitOne
        ]

        var result: AnyObject?
        let status = SecItemCopyMatching(query as CFDictionary, &result)
        guard status == errSecSuccess else { return nil }
        return result as? Data
    }

    private func saveToKeychain(data: Data) {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key,
            kSecValueData as String: data
        ]

        SecItemDelete(query as CFDictionary)
        SecItemAdd(query as CFDictionary, nil)
    }

    private func deleteFromKeychain() {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key
        ]

        SecItemDelete(query as CFDictionary)
    }
}
```

#### `$ARGUMENTS/Core/Helpers/UserDefault.swift`

```swift
import Foundation

@propertyWrapper
struct UserDefault<K: RawRepresentable & CaseIterable & Sendable, T: Codable & Sendable>: Sendable where K.RawValue == String {
    private let key: K
    private let defaultValue: T
    private let userDefaults: UserDefaults

    init(key: K,
         defaultValue: T,
         userDefaults: UserDefaults = .local) {

        self.key = key
        self.defaultValue = defaultValue
        self.userDefaults = userDefaults
    }

    var wrappedValue: T {
        get {
            guard let data = userDefaults.data(forKey: key.rawValue) else {
                return defaultValue
            }
            let decoder = JSONDecoder()
            do {
                return try decoder.decode(T.self, from: data)
            } catch {
                assertionFailure("Failed to decode UserDefault for key \(key): \(error)")
                return defaultValue
            }
        }
        set {
            let encoder = JSONEncoder()
            do {
                let data = try encoder.encode(newValue)
                userDefaults.set(data, forKey: key.rawValue)
            } catch {
                assertionFailure("Failed to encode UserDefault for key \(key): \(error)")
            }
        }
    }
}

extension UserDefaults: @unchecked @retroactive Sendable {}

@propertyWrapper
struct RequiredUserDefault<K: RawRepresentable & CaseIterable & Sendable, T: Codable & Sendable>: Sendable where K.RawValue == String {
    private let key: K
    private let userDefaults: UserDefaults

    init(key: K, userDefaults: UserDefaults = .local) {
        self.key = key
        self.userDefaults = userDefaults
    }

    var wrappedValue: T {
        get {
            guard let data = userDefaults.data(forKey: key.rawValue) else {
                fatalError("No value found for key '\(key.rawValue)' — must be set before access")
            }
            do {
                return try JSONDecoder().decode(T.self, from: data)
            } catch {
                fatalError("Failed to decode UserDefault for key \(key): \(error)")
            }
        }
        set {
            let encoder = JSONEncoder()
            do {
                let data = try encoder.encode(newValue)
                userDefaults.set(data, forKey: key.rawValue)
            } catch {
                assertionFailure("Failed to encode UserDefault for key \(key): \(error)")
            }
        }
    }

    var initialValue: T {
        get { wrappedValue }
        set {
            guard userDefaults.data(forKey: key.rawValue) == nil else { return }
            let encoder = JSONEncoder()
            do {
                let data = try encoder.encode(newValue)
                userDefaults.set(data, forKey: key.rawValue)
            } catch {
                assertionFailure("Failed to encode UserDefault for key \(key): \(error)")
            }
        }
    }
}

extension UserDefaults {

    static func removeAll<K: RawRepresentable & CaseIterable>(for keyType: K.Type) where K.RawValue == String {
        keyType.allCases.forEach {
            standard.removeObject(forKey: $0.rawValue)
        }
    }
}
```

#### `$ARGUMENTS/Core/Helpers/UserDefaults+.swift`

```swift
import Foundation

extension UserDefaults {

    static var localKey: String { "com.$ARGUMENTS_LOWER.user-defaults" }

    static var local: UserDefaults {
        guard let defaults = UserDefaults(suiteName: localKey) else {
            assertionFailure("Failed to create UserDefaults suite: \(localKey)")
            return .standard
        }
        return defaults
    }

    static func clearLocal() {
        UserDefaults.local.removePersistentDomain(forName: localKey)
    }
}
```

#### `$ARGUMENTS/Core/Helpers/DataCache.swift`

```swift
import Foundation

final class DataCache<Key: Codable> {

    struct Cache<T: Codable>: Codable {
        let value: T
        let timestamp: Date
    }

    private let userDefaults: UserDefaults
    init(userDefaults: UserDefaults = .local) {
        self.userDefaults = userDefaults
    }

    func getKey(_ key: Key) throws -> String {
        let encoder = JSONEncoder()
        encoder.outputFormatting = [.sortedKeys]
        return try encoder.encode(key).base64EncodedString()
    }

    func store<T: Codable>(data: T, for key: Key) throws {
        let cache = Cache(value: data, timestamp: .now)
        let encoder = JSONEncoder()
        let data = try encoder.encode(cache)
        userDefaults.set(data, forKey: try getKey(key))
    }

    func get<T: Codable>(type: T.Type, for key: Key) throws -> Cache<T>? {
        guard let data = userDefaults.data(forKey: try getKey(key)) else { return nil }
        let decoder = JSONDecoder()
        return try decoder.decode(Cache<T>.self, from: data)
    }
}
```

#### `$ARGUMENTS/Core/Helpers/Codable+.swift`

```swift
import Foundation

extension JSONEncoder {

    static var `default`: JSONEncoder {
        let encoder = JSONEncoder()
        encoder.dateEncodingStrategy = .iso8601
        encoder.keyEncodingStrategy = .convertToSnakeCase
        return encoder
    }
}

extension JSONDecoder {

    static var `default`: JSONDecoder {
        let decoder = JSONDecoder()
        decoder.dateDecodingStrategy = .custom { decoder in
            let container = try decoder.singleValueContainer()
            let dateString = try container.decode(String.self)

            let isoFormatter = ISO8601DateFormatter()
            isoFormatter.formatOptions = [
                .withInternetDateTime,
                .withFractionalSeconds,
                .withDashSeparatorInDate
            ]

            if let date = isoFormatter.date(from: dateString) {
                return date
            }

            let dateOnlyFormatter = DateFormatter()
            dateOnlyFormatter.calendar = Calendar(identifier: .iso8601)
            dateOnlyFormatter.dateFormat = "yyyy-MM-dd"
            if let date = dateOnlyFormatter.date(from: dateString) {
                return date
            }

            throw DecodingError.dataCorruptedError(
                in: container,
                debugDescription: "Invalid date format: \(dateString)"
            )
        }
        decoder.keyDecodingStrategy = .convertFromSnakeCase
        return decoder
    }
}
```

#### `$ARGUMENTS/Core/Helpers/Error+.swift`

```swift
import Foundation

extension Error {

    static func `tryAwait`<T>(_ expression: @Sendable () async throws -> T, error errorMapper: @Sendable (Error) -> Self) async throws(Self) -> T {
        do {
            return try await expression()
        } catch {
            throw errorMapper(error)
        }
    }

    static func `try`<T>(_ expression: @Sendable () throws -> T, error errorMapper: @Sendable (Error) -> Self) throws(Self) -> T {
        do {
            return try expression()
        } catch {
            throw errorMapper(error)
        }
    }
}

extension String: @retroactive LocalizedError {
    public var errorDescription: String? { self }
}
```

#### `$ARGUMENTS/Core/Helpers/URL+.swift`

```swift
import Foundation

extension URL {

    static var base: URL {
        guard let url = URL(string: "https://YOUR_API_BASE_URL") else {
            fatalError("Invalid base URL")
        }
        return url
    }

    static var `default`: URL {
        base.appending(path: "api")
    }
}

extension URLComponents {

    static var `default`: URLComponents {
        guard let components = URLComponents(url: .default, resolvingAgainstBaseURL: false) else {
            fatalError("Invalid URL components")
        }
        return components
    }
}
```

---

### 2.3 Core — Networking

#### `$ARGUMENTS/Core/Networking/HttpMethod.swift`

```swift
import Foundation

enum HttpMethod: String {
    case get = "GET"
    case post = "POST"
    case patch = "PATCH"
    case delete = "DELETE"
}
```

#### `$ARGUMENTS/Core/Networking/AnyMutation.swift`

```swift
import Foundation

protocol AnyMutation: Encodable, Sendable {
    func url(baseURL: URL) -> URL
    var httpMethod: HttpMethod { get }
}

extension AnyMutation {
    var url: URL { url(baseURL: .default) }
}

extension AnyMutation {

    func request() throws(NetworkingServiceError) -> URLRequest {
        do {
            var request = URLRequest(url: url)
            request.httpMethod = httpMethod.rawValue
            request.httpBody = try JSONEncoder.default.encode(self)

            #if DEBUG
            print("🚀 Request URL: \(request.url?.absoluteString ?? \"nil\")")
            print(String(data: request.httpBody ?? Data(), encoding: .utf8) ?? "Can't print encoded JSON body")
            #endif

            request.addValue("application/json", forHTTPHeaderField: "Content-Type")
            return request
        } catch {
            throw .bodyEncode(error)
        }
    }
}

protocol AnyQuery: Sendable {
    func url(baseURL: URL) -> URL
}
```

#### `$ARGUMENTS/Core/Networking/NetworkingService.swift`

```swift
import Foundation

protocol AnyNetworkingService: Sendable {
    typealias DecodableResult = Decodable & Sendable
    func perform<Mutation: AnyMutation, R: DecodableResult>(with mutation: Mutation) async throws(NetworkingServiceError) -> R
    func perform<Query: AnyQuery, R: DecodableResult>(with query: Query, token: String) async throws(NetworkingServiceError) -> R
    func perform<Mutation: AnyMutation, R: DecodableResult>(with mutation: Mutation, token: String) async throws(NetworkingServiceError) -> R
}

final actor NetworkingService: AnyNetworkingService {

    private let configuration: URLSessionConfiguration = {
        let configuration = URLSessionConfiguration.default
        configuration.timeoutIntervalForRequest = 60
        configuration.timeoutIntervalForResource = 60
        configuration.httpCookieStorage = .shared
        configuration.httpShouldSetCookies = true
        configuration.requestCachePolicy = .useProtocolCachePolicy
        configuration.urlCache = .shared
        configuration.httpAdditionalHeaders = [
            "Accept": "application/json"
        ]
        return configuration
    }()

    private lazy var session: URLSession = .init(configuration: configuration)

    func perform<Mutation: AnyMutation, R: DecodableResult>(with mutation: Mutation) async throws(NetworkingServiceError) -> R {

        let request = try mutation.request()
        let data = try await perform(request: request)

        return try NetworkingServiceError.try {
            try JSONDecoder.default.decode(R.self, from: data)
        } error: {
            .decodingError($0, data: data)
        }
    }

    func perform<Mutation: AnyMutation, R: DecodableResult>(with mutation: Mutation, token: String) async throws(NetworkingServiceError) -> R {
        var request = try mutation.request()
        request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")

        let data = try await perform(request: request)

        return try NetworkingServiceError.try {
            try JSONDecoder.default.decode(R.self, from: data)
        } error: {
            .decodingError($0, data: data)
        }
    }

    private func perform(request: URLRequest) async throws(NetworkingServiceError) -> Data {

        let (data, response) = try await NetworkingServiceError.tryAwait {
            return try await session.data(for: request)
        } error: {
            .requestFailed($0)
        }

        guard let response = response as? HTTPURLResponse else {
            throw .invalidResponse
        }

        // If status code is 2xx, return data and response. Success case.
        guard !(200...299).contains(response.statusCode) else { return data }

        // Else error responses
        let error = try NetworkingServiceError.try {
            try JSONDecoder.default.decode(APIError.self, from: data)
        } error: {
            .decodingError($0, data: data)
        }

        throw .apiError(error, response: response)
    }

    func perform<Query: AnyQuery, R: DecodableResult>(with query: Query, token: String) async throws(NetworkingServiceError) -> R {
        let url = query.url(baseURL: .default)
        var request = URLRequest(url: url)
        request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")

        let data = try await perform(request: request)

        return try NetworkingServiceError.try {
            try JSONDecoder.default.decode(R.self, from: data)
        } error: {
            .decodingError($0, data: data)
        }
    }
}
```

#### `$ARGUMENTS/Core/Networking/NetworkingServiceError.swift`

```swift
import Foundation

enum NetworkingServiceError: LocalizedError {
    case unauthorised
    case invalidResponse
    case requestFailed(Error)
    case bodyEncode(Error)
    case apiError(APIError, response: HTTPURLResponse)
    case decodingError(Error, String?)

    static func decodingError(_ error: Error, data: Data) -> Self {
        let dataString = String(data: data, encoding: .utf8)
        return .decodingError(error, dataString)
    }

    var errorDescription: String? {
        switch self {
        case .unauthorised:
            return "Unauthorised access"
        case .invalidResponse:
            return "Invalid response type"
        case let .requestFailed(error):
            return error.localizedDescription
        case let .bodyEncode(error):
            return "Failed to encode request body: \(error.localizedDescription)"
        case let .apiError(apiError, _):
            return apiError.error
        case let .decodingError(error, dataString):
            var description = "Failed to decode response: \(error.localizedDescription)"
            if let dataString {
                description += "\nResponse Data: \(dataString)"
            }
            return description
        }
    }

    var isCancelled: Bool {
        if case .requestFailed(let error) = self {
            return (error as NSError).code == NSURLErrorCancelled || error is CancellationError
        }
        return false
    }
}
```

---

### 2.4 Core — Models

#### `$ARGUMENTS/Core/Models/User.swift`

```swift
import Foundation

struct User: Codable, Hashable, Sendable {
    let id: Int
    let email: String
    let name: String
}
```

#### `$ARGUMENTS/Core/Models/APISession.swift`

```swift
import Foundation

struct APISession: Codable, Sendable {
    let accessToken: String
    let refreshToken: String
    let expiresIn: Int
    let user: User?
}

struct RefreshedAPISession: Codable, Sendable {
    let accessToken: String
    let refreshToken: String
    let expiresIn: Int
}
```

#### `$ARGUMENTS/Core/Models/APIError.swift`

```swift
import Foundation

struct APIError: Decodable, Error {
    let error: String
}
```

#### `$ARGUMENTS/Core/Models/Mutations.swift`

```swift
import Foundation

struct LoginMutation: AnyMutation {
    let email: String
    let password: String

    var httpMethod: HttpMethod { .post }
    func url(baseURL: URL) -> URL {
        baseURL.appending(path: "auth/login")
    }
}

struct RefreshSessionMutation: AnyMutation {
    let refreshToken: String

    var httpMethod: HttpMethod { .post }
    func url(baseURL: URL) -> URL {
        baseURL.appending(path: "auth/refresh")
    }
}
```

---

### 2.5 Core — Session

#### `$ARGUMENTS/Core/Session/Session.swift`

```swift
import Foundation

protocol AnySession: Codable, Sendable {
    var accessToken: String { get }
    var refreshToken: String { get }
    var time: Date { get }
    var user: User { get }
}

extension AnySession {
    var isActive: Bool { time > .now }
}

struct Session: AnySession {
    let user: User
    let accessToken: String
    let refreshToken: String
    let time: Date

    init(user: User, accessToken: String, refreshToken: String, time: Date) {
        self.user = user
        self.accessToken = accessToken
        self.refreshToken = refreshToken
        self.time = time
    }

    init?(with apiSession: APISession) {
        guard let user = apiSession.user else { return nil }
        self.user = user
        accessToken = apiSession.accessToken
        refreshToken = apiSession.refreshToken
        time = Date.now.addingTimeInterval(TimeInterval(apiSession.expiresIn))
    }

    func refresh(with apiSession: RefreshedAPISession) -> Session {
        .init(
            user: user,
            accessToken: apiSession.accessToken,
            refreshToken: apiSession.refreshToken,
            time: Date.now.addingTimeInterval(TimeInterval(apiSession.expiresIn))
        )
    }
}
```

#### `$ARGUMENTS/Core/Session/SessionProvider.swift`

```swift
import Foundation

protocol AnySessionProvider: Observable, AnyObject, Sendable {
    @MainActor var isAuthorized: Bool { get }
    @MainActor var session: AnySession? { get }

    @MainActor func refreshSession() async throws(NetworkingServiceError) -> AnySession
    @MainActor func authorize(email: String, password: String) async throws(NetworkingServiceError)
    @MainActor func unauthorize() async throws(NetworkingServiceError)
}

@Observable
@MainActor
final class SessionProvider: AnySessionProvider {

    private let networkingService: AnyNetworkingService

    @ObservationIgnored @Keychain(key: "session")
    private var currentSession: Session?

    var isAuthorized: Bool = false
    @ObservationIgnored private(set) var session: AnySession?

    init(networkingService: AnyNetworkingService) {
        self.networkingService = networkingService
        session = currentSession
        isAuthorized = currentSession != nil
    }

    @discardableResult
    func refreshSession() async throws(NetworkingServiceError) -> AnySession {
        guard let session = currentSession else {
            try await unauthorize()
            throw .unauthorised
        }
        guard !session.isActive else { return session }

        let mutation = RefreshSessionMutation(refreshToken: session.refreshToken)
        let apiSession: RefreshedAPISession = try await networkingService.perform(with: mutation)
        let newSession = session.refresh(with: apiSession)

        applySession(newSession)
        return newSession
    }

    func authorize(email: String, password: String) async throws(NetworkingServiceError) {
        #if DEBUG
        if email == "test@test.com" && password == "test" {
            let mock = Session(
                user: User(id: 1, email: email, name: "Test User"),
                accessToken: "mock-token",
                refreshToken: "mock-refresh",
                time: .now.addingTimeInterval(3600)
            )
            applySession(mock)
            return
        }
        #endif

        let mutation = LoginMutation(email: email.lowercased(), password: password)
        let apiSession: APISession = try await networkingService.perform(with: mutation)

        if let session = Session(with: apiSession) {
            applySession(session)
        }
    }

    func unauthorize() async throws(NetworkingServiceError) {
        URLCache.shared.removeAllCachedResponses()
        currentSession = nil
        session = nil
        isAuthorized = false
    }

    private func applySession(_ newSession: Session) {
        currentSession = newSession
        session = newSession
        isAuthorized = true
    }
}
```

---

### 2.6 Core — Context & ServiceProvider

#### `$ARGUMENTS/Core/Context.swift`

```swift
import Foundation
import Observation

@MainActor
protocol AnyContext: AnyObject, Sendable {
    var session: AnySession { get }
    var sessionProvider: AnySessionProvider { get }
    var services: AnyServiceProvider { get }
}

@MainActor
final class AppContext: AnyContext {
    let session: AnySession
    let sessionProvider: AnySessionProvider
    let services: AnyServiceProvider

    init(session: AnySession, sessionProvider: AnySessionProvider, networkingService: AnyNetworkingService) {
        self.session = session
        self.sessionProvider = sessionProvider
        self.services = ServiceProvider(
            session: session,
            sessionProvider: sessionProvider,
            networkingService: networkingService
        )
    }
}
```

#### `$ARGUMENTS/Core/ServiceProvider.swift`

```swift
@MainActor
protocol AnyServiceProvider {
    var settings: AnyUserSettings { get }
    // TODO: Add app-specific services here
    // var someService: AnySomeService { get }
}

@MainActor
final class ServiceProvider: AnyServiceProvider {
    private let session: AnySession
    private let sessionProvider: AnySessionProvider
    private let networkingService: AnyNetworkingService

    lazy var settings: AnyUserSettings = UserSettings(session: session)

    // TODO: Add app-specific services here
    // lazy var someService: AnySomeService = SomeService(
    //     sessionProvider: sessionProvider,
    //     networkingService: networkingService,
    //     settings: settings
    // )

    init(session: AnySession, sessionProvider: AnySessionProvider, networkingService: AnyNetworkingService) {
        self.session = session
        self.sessionProvider = sessionProvider
        self.networkingService = networkingService
    }
}
```

#### `$ARGUMENTS/Core/Services/UserSettings.swift`

```swift
import Foundation

enum UserDefaultKeys: String, CaseIterable {
    case notificationsEnabled
}

enum CacheKeys: Codable {
    case placeholder
    // Add cache keys for your app data, e.g.:
    // case users
    // case posts
}

@MainActor
protocol AnyUserSettings: AnyObject, Sendable {
    var cache: DataCache<CacheKeys> { get }
    var notificationsEnabled: Bool { get set }
}

@MainActor
final class UserSettings: AnyUserSettings {

    let cache = DataCache<CacheKeys>()

    @UserDefault(key: UserDefaultKeys.notificationsEnabled, defaultValue: true)
    var notificationsEnabled: Bool

    init(session: AnySession) {}
}
```

---

### 2.7 App Entry Point

#### `$ARGUMENTS/${ARGUMENTS}App.swift`

```swift
import SwiftUI

@main
struct ${ARGUMENTS}App: App {
    let networkingService: NetworkingService
    @State var sessionProvider: SessionProvider

    init() {
        self.networkingService = NetworkingService()
        self.sessionProvider = SessionProvider(networkingService: networkingService)
    }

    var body: some Scene {
        WindowGroup {
            MainView(sessionProvider: sessionProvider) {
                AppContext(session: $0, sessionProvider: sessionProvider, networkingService: networkingService)
            }
        }
    }
}
```

---

### 2.8 UI — MainView

#### `$ARGUMENTS/UI/Interfaces/MainView.swift`

```swift
import SwiftUI
import Coordinator

struct MainView<Context: AnyContext, SessionProvider: AnySessionProvider>: View {
    @Bindable var sessionProvider: SessionProvider
    let contextProvider: (AnySession) -> Context

    var body: some View {
        Group {
            if sessionProvider.isAuthorized, let session = sessionProvider.session {
                TabsView(context: contextProvider(session))
            } else {
                CoordinatorRoot(context: sessionProvider, destination: .login)
            }
        }
    }
}
```

---

### 2.9 UI — TabsView

#### `$ARGUMENTS/UI/Interfaces/Tabs/TabsView.swift`

```swift
import SwiftUI
import Coordinator

enum Tabs: Hashable {
    case home
    case settings
}

struct TabsView<C: AnyContext>: View {

    let context: C
    @State private var selected: Tabs = .home

    var body: some View {
        TabView(selection: $selected) {
            Tab("Home", systemImage: "house", value: .home) {
                CoordinatorRoot(context: context, destination: .home)
            }

            Tab("Settings", systemImage: "gear", value: .settings) {
                CoordinatorRoot(context: context, destination: .settings)
            }
        }
    }
}
```

---

### 2.10 UI — Auth Screens

#### `$ARGUMENTS/UI/Interfaces/Auth/Login/LoginView.swift`

```swift
import SwiftUI
import Coordinator

struct LoginView<C: AnySessionProvider, VM: AnyLoginViewModel>: View {
    @EnvironmentObject var coordinator: Coordinator<C>
    @State var viewModel: VM

    var body: some View {
        VStack(spacing: 20) {
            Spacer()

            Text("$ARGUMENTS")
                .font(.largeTitle.bold())

            TextField("Email", text: $viewModel.email)
                .textContentType(.emailAddress)
                .autocorrectionDisabled()
                .textInputAutocapitalization(.never)
                .keyboardType(.emailAddress)
                .textFieldStyle(.roundedBorder)

            SecureField("Password", text: $viewModel.password)
                .textContentType(.password)
                .textFieldStyle(.roundedBorder)

            if let error = viewModel.error {
                Text(error)
                    .foregroundStyle(.red)
                    .font(.caption)
            }

            Button {
                Task { await viewModel.signIn() }
            } label: {
                if viewModel.isLoading {
                    ProgressView()
                        .frame(maxWidth: .infinity)
                } else {
                    Text("Sign In")
                        .frame(maxWidth: .infinity)
                }
            }
            .buttonStyle(.borderedProminent)
            .disabled(viewModel.isLoading)

            Button("Create account") {
                coordinator.push(.signUp)
            }

            Spacer()
        }
        .padding(.horizontal, 24)
    }
}
```

#### `$ARGUMENTS/UI/Interfaces/Auth/Login/LoginViewModel.swift`

```swift
import Foundation

@MainActor
protocol AnyLoginViewModel: Observable {
    var email: String { get set }
    var password: String { get set }
    var isLoading: Bool { get }
    var error: String? { get }
    func signIn() async
}

@Observable
final class LoginViewModel: AnyLoginViewModel {
    private let sessionProvider: AnySessionProvider

    var email: String = "test@test.com"
    var password: String = "test"
    var isLoading: Bool = false
    var error: String? = nil

    init(sessionProvider: AnySessionProvider) {
        self.sessionProvider = sessionProvider
    }

    func signIn() async {
        guard !email.isEmpty, !password.isEmpty else {
            error = "Please fill in all fields"
            return
        }
        isLoading = true
        error = nil
        defer { isLoading = false }

        do {
            try await sessionProvider.authorize(email: email, password: password)
        } catch {
            self.error = error.localizedDescription
        }
    }
}
```

#### `$ARGUMENTS/UI/Interfaces/Auth/Login/LoginViewBuilder.swift`

```swift
import Coordinator

@MainActor
extension Destination where Context: AnySessionProvider {

    static var login: Self {
        .init(id: "login") {
            LoginView<Context, LoginViewModel>(viewModel: .init(sessionProvider: $0.context))
        }
    }
}
```

#### `$ARGUMENTS/UI/Interfaces/Auth/SignUp/SignUpView.swift`

```swift
import SwiftUI
import Coordinator

struct SignUpView<C: AnySessionProvider, VM: AnySignUpViewModel>: View {
    @EnvironmentObject var coordinator: Coordinator<C>
    @State var viewModel: VM

    var body: some View {
        VStack(spacing: 20) {
            Spacer()

            Text("Create Account")
                .font(.largeTitle.bold())

            TextField("Email", text: $viewModel.email)
                .textContentType(.emailAddress)
                .autocorrectionDisabled()
                .textInputAutocapitalization(.never)
                .keyboardType(.emailAddress)
                .textFieldStyle(.roundedBorder)

            SecureField("Password", text: $viewModel.password)
                .textContentType(.newPassword)
                .textFieldStyle(.roundedBorder)

            SecureField("Confirm password", text: $viewModel.confirmPassword)
                .textContentType(.newPassword)
                .textFieldStyle(.roundedBorder)

            if let error = viewModel.error {
                Text(error)
                    .foregroundStyle(.red)
                    .font(.caption)
            }

            Button {
                Task { await viewModel.signUp() }
            } label: {
                if viewModel.isLoading {
                    ProgressView()
                        .frame(maxWidth: .infinity)
                } else {
                    Text("Sign Up")
                        .frame(maxWidth: .infinity)
                }
            }
            .buttonStyle(.borderedProminent)
            .disabled(viewModel.isLoading)

            Spacer()
        }
        .padding(.horizontal, 24)
    }
}
```

#### `$ARGUMENTS/UI/Interfaces/Auth/SignUp/SignUpViewModel.swift`

```swift
import Foundation

@MainActor
protocol AnySignUpViewModel: Observable {
    var email: String { get set }
    var password: String { get set }
    var confirmPassword: String { get set }
    var isLoading: Bool { get }
    var error: String? { get }
    func signUp() async
}

@Observable
final class SignUpViewModel: AnySignUpViewModel {
    private let sessionProvider: AnySessionProvider

    var email: String = ""
    var password: String = ""
    var confirmPassword: String = ""
    var isLoading: Bool = false
    var error: String? = nil

    init(sessionProvider: AnySessionProvider) {
        self.sessionProvider = sessionProvider
    }

    func signUp() async {
        guard !email.isEmpty, !password.isEmpty else {
            error = "Please fill in all fields"
            return
        }
        guard password == confirmPassword else {
            error = "Passwords don't match"
            return
        }
        isLoading = true
        error = nil
        defer { isLoading = false }

        // TODO: implement registration
        // try await sessionProvider.register(with: ...)
    }
}
```

#### `$ARGUMENTS/UI/Interfaces/Auth/SignUp/SignUpViewBuilder.swift`

```swift
import Coordinator

@MainActor
extension Destination where Context: AnySessionProvider {

    static var signUp: Self {
        .init(id: "signUp") {
            SignUpView<Context, SignUpViewModel>(viewModel: .init(sessionProvider: $0.context))
        }
    }
}
```

---

### 2.11 UI — Main App Screens

#### `$ARGUMENTS/UI/Interfaces/Home/HomeView.swift`

```swift
import SwiftUI
import Coordinator

struct HomeView<C: AnyContext, VM: AnyHomeViewModel>: View {
    @EnvironmentObject var coordinator: Coordinator<C>
    @State var viewModel: VM

    var body: some View {
        List {
            // MARK: - Push navigation example
            Section("Push") {
                ForEach(Array(viewModel.items.enumerated()), id: \.offset) { index, item in
                    Button(item) {
                        coordinator.push(.detail(id: index))
                    }
                }
            }

            // MARK: - Sheet presentation example
            Section("Sheet") {
                Button("Open Profile (sheet)") {
                    coordinator.present(sheet: .profile)
                }
            }

            // MARK: - FullScreenCover example
            Section("Full Screen Cover") {
                Button("Open Profile (fullscreen)") {
                    coordinator.present(fullScreenCover: .profileFullScreen)
                }
            }
        }
        .navigationTitle("Home")

        .task {
            await viewModel.loadData()
        }
        .overlay {
            if viewModel.isLoading {
                ProgressView()
            }
        }
    }
}
```

#### `$ARGUMENTS/UI/Interfaces/Home/HomeViewModel.swift`

```swift
import Foundation

@MainActor
protocol AnyHomeViewModel: Observable {
    var items: [String] { get }
    var isLoading: Bool { get }
    func loadData() async
}

@Observable
final class HomeViewModel: AnyHomeViewModel {
    private let context: AnyContext

    private(set) var items: [String] = []
    private(set) var isLoading: Bool = false

    init(context: AnyContext) {
        self.context = context
    }

    func loadData() async {
        isLoading = true
        defer { isLoading = false }

        // Simulate loading
        try? await Task.sleep(for: .milliseconds(500))
        items = ["Item 1", "Item 2", "Item 3"]
    }
}
```

#### `$ARGUMENTS/UI/Interfaces/Home/HomeViewBuilder.swift`

```swift
import Coordinator

@MainActor
extension Destination where Context: AnyContext {

    static var home: Self {
        .init(id: "home") {
            HomeView<Context, HomeViewModel>(viewModel: .init(context: $0.context))
        }
    }
}
```

#### `$ARGUMENTS/UI/Interfaces/Detail/DetailView.swift`

```swift
import SwiftUI
import Coordinator

struct DetailView<C: AnyContext, VM: AnyDetailViewModel>: View {
    @EnvironmentObject var coordinator: Coordinator<C>
    @State var viewModel: VM

    var body: some View {
        VStack {
            if viewModel.isLoading {
                ProgressView()
            } else {
                Text(viewModel.title)
                Button("Next") {
                    coordinator.push(.detail(id: viewModel.id))
                }
            }
        }
        .navigationTitle("Detail")
        .task {
            await viewModel.loadData()
        }
    }
}
```

#### `$ARGUMENTS/UI/Interfaces/Detail/DetailViewModel.swift`

```swift
import Foundation

@MainActor
protocol AnyDetailViewModel: Observable {
    var id: Int { get }
    var title: String { get }
    var isLoading: Bool { get }
    func loadData() async
}

@Observable
final class DetailViewModel: AnyDetailViewModel {
    private let context: AnyContext

    let id: Int
    private(set) var title: String = ""
    private(set) var isLoading: Bool = false

    init(context: AnyContext, id: Int) {
        self.context = context
        self.id = id
    }

    func loadData() async {
        isLoading = true
        defer { isLoading = false }
        // TODO: load from context.services
        title = "Detail #\(id)"
    }
}
```

#### `$ARGUMENTS/UI/Interfaces/Detail/DetailViewBuilder.swift`

```swift
import Coordinator

@MainActor
extension Destination where Context: AnyContext {

    static func detail(id: Int, hideTabBar: Bool = false) -> Self {
        .init(id: "detail-\(id)") {
            DetailView<Context, DetailViewModel>(
                viewModel: .init(context: $0.context, id: id)
            )
            .toolbarVisibility(hideTabBar ? .hidden : .automatic, for: .tabBar)
        }
    }
}
```

#### `$ARGUMENTS/UI/Interfaces/Profile/ProfileView.swift`

```swift
import SwiftUI
import Coordinator

struct ProfileView<C: AnyContext, VM: AnyProfileViewModel>: View {
    @EnvironmentObject var coordinator: Coordinator<C>
    @State var viewModel: VM

    var body: some View {
        VStack(spacing: 20) {
            Image(systemName: "person.circle.fill")
                .font(.system(size: 80))
                .foregroundStyle(.secondary)

            Text(viewModel.userName)
                .font(.title2.bold())

            // Push inside sheet/fullScreenCover (sub-navigation)
            Button("View detail") {
                coordinator.push(.detail(id: 1))
            }
            .buttonStyle(.bordered)

            Button("Dismiss") {
                coordinator.dismiss()
            }
            .buttonStyle(.borderedProminent)
        }
        .navigationTitle("Profile")
    }
}
```

#### `$ARGUMENTS/UI/Interfaces/Profile/ProfileViewModel.swift`

```swift
import Foundation

@MainActor
protocol AnyProfileViewModel: Observable {
    var userName: String { get }
}

@Observable
final class ProfileViewModel: AnyProfileViewModel {
    private let context: AnyContext

    private(set) var userName: String

    init(context: AnyContext) {
        self.context = context
        self.userName = context.session.user.name
    }
}
```

#### `$ARGUMENTS/UI/Interfaces/Profile/ProfileViewBuilder.swift`

```swift
import Coordinator

@MainActor
extension Destination where Context: AnyContext {

    static var profile: Self {
        .init(id: "profile") {
            CoordinatorBranch(rootCoordinator: $0) {
                ProfileView<Context, ProfileViewModel>(viewModel: .init(context: $0.context))
            }
        }
    }

    static var profileFullScreen: Self {
        .init(id: "profileFullScreen") {
            CoordinatorBranch(rootCoordinator: $0) {
                ProfileView<Context, ProfileViewModel>(viewModel: .init(context: $0.context))
            }
        }
    }
}
```

#### `$ARGUMENTS/UI/Interfaces/Settings/SettingsView.swift`

```swift
import SwiftUI
import Coordinator

struct SettingsView<C: AnyContext, VM: AnySettingsViewModel>: View {
    @EnvironmentObject var coordinator: Coordinator<C>
    @State var viewModel: VM

    var body: some View {
        List {
            Section("Navigation") {
                Button("Detail (push, hides tab bar)") {
                    coordinator.push(.detail(id: 42, hideTabBar: true))
                }
            }

            Section("Preferences") {
                Toggle("Notifications", isOn: $viewModel.notificationsEnabled)
            }

            if let error = viewModel.error {
                Section {
                    Text(error)
                        .foregroundStyle(.red)
                }
            }

            Section {
                Button("Logout", role: .destructive) {
                    Task { await viewModel.logout() }
                }
            }
        }
        .navigationTitle("Settings")
    }
}
```

#### `$ARGUMENTS/UI/Interfaces/Settings/SettingsViewModel.swift`

```swift
import Foundation

@MainActor
protocol AnySettingsViewModel: Observable {
    var isLoading: Bool { get }
    var error: String? { get }
    var notificationsEnabled: Bool { get set }
    func logout() async
}

@Observable
final class SettingsViewModel: AnySettingsViewModel {
    private let context: AnyContext

    private(set) var isLoading: Bool = false
    private(set) var error: String? = nil

    var notificationsEnabled: Bool {
        get { context.services.settings.notificationsEnabled }
        set { context.services.settings.notificationsEnabled = newValue }
    }

    init(context: AnyContext) {
        self.context = context
    }

    func logout() async {
        isLoading = true
        error = nil
        defer { isLoading = false }

        do {
            try await context.sessionProvider.unauthorize()
        } catch {
            self.error = error.localizedDescription
        }
    }
}
```

#### `$ARGUMENTS/UI/Interfaces/Settings/SettingsViewBuilder.swift`

```swift
import Coordinator

@MainActor
extension Destination where Context: AnyContext {

    static var settings: Self {
        .init(id: "settings") {
            SettingsView<Context, SettingsViewModel>(viewModel: .init(context: $0.context))
        }
    }
}
```

---

### 2.12 project.yml (XcodeGen)

#### `project.yml`

```yaml
name: $ARGUMENTS
options:
  bundleIdPrefix: com.$ARGUMENTS_LOWER
  deploymentTarget:
    iOS: "18.0"
  xcodeVersion: "16.0"
  groupSortPosition: top

packages:
  Coordinator:
    path: Packages/Coordinator

targets:
  $ARGUMENTS:
    type: application
    platform: iOS
    sources:
      - $ARGUMENTS
    dependencies:
      - package: Coordinator
    settings:
      base:
        SWIFT_VERSION: "6"
        GENERATE_INFOPLIST_FILE: true
        INFOPLIST_KEY_UILaunchScreen_Generation: true
        INFOPLIST_KEY_UISupportedInterfaceOrientations: UIInterfaceOrientationPortrait
```

---

## Step 3: Add SCREEN_GENERATION.md

Copy `SCREEN_GENERATION.md` from this repository into the root of the generated project:

```
$ARGUMENTS/SCREEN_GENERATION.md
```

This file describes how to add new screens to the project. It should be included so that any AI assistant working with the project later understands the architecture patterns and can generate new screens correctly.

Fetch it from GitHub:
```bash
curl -o $ARGUMENTS/SCREEN_GENERATION.md https://raw.githubusercontent.com/Lumyk/CoordinatorTemplate/main/SCREEN_GENERATION.md
```

---

## Step 4: Run XcodeGen

After generating all files, run:

```bash
cd $ARGUMENTS && xcodegen generate
```

If `xcodegen` is not installed, inform the user:
```
Install XcodeGen via Homebrew: brew install xcodegen
Then run: cd $ARGUMENTS && xcodegen generate
```

---

## Architecture Rules Summary

1. Every protocol starts with `Any` prefix: `AnyContext`, `AnySession`, `AnyHomeViewModel`
2. ViewModels use `@Observable` (not ObservableObject), marked `@MainActor`
3. Views are generic over `<C: ContextProtocol, VM: ViewModelProtocol>` — receive VM as `@State`, coordinator as `@EnvironmentObject`
4. Destination builders are `static var/func` extensions on `Destination where Context: ...`
5. Destination `id` is an explicit string (e.g., `"home"`, `"detail-\(id)"`)
6. Services call `sessionProvider.refreshSession()` before every API request
7. Networking uses typed throws: `async throws(NetworkingServiceError)`
8. Session stored in Keychain with in-memory cache, observable state set via `applySession()`
9. Each tab uses `CoordinatorRoot(context:, destination:)` — independent navigation stacks
10. Sheet/fullScreenCover destinations use `CoordinatorBranch(rootCoordinator:)` for sub-navigation
11. `CoordinatorRoot` conforms to `Equatable` with `===` — prevents unnecessary re-renders
12. `coordinator.dismiss()` clears path + presentations and dismisses parent's sheet/cover
13. `NetworkingServiceError` has static helpers `try {}` and `tryAwait {}` (from `Error+.swift`)
14. File headers: `// FileName.swift // $ARGUMENTS // Copyright © Yevhenii Kalashnikov. All rights reserved.`
15. `project.yml` (XcodeGen) — not `.xcodeproj` in version control
16. Mock auth in DEBUG: `test@test.com` / `test` skips API

---
> Source: [Lumyk/CoordinatorTemplate](https://github.com/Lumyk/CoordinatorTemplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
