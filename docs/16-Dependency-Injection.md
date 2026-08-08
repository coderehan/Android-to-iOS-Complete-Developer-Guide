# 16 - Dependency Injection: Hilt/Dagger vs Swift DI

> This is one of the bigger philosophical gaps. Android has a mature, compiler-verified DI framework (Hilt/Dagger) baked into the ecosystem. iOS has **no official DI framework** — the community uses lighter patterns: manual constructor injection, the Environment/EnvironmentObject system, or third-party libraries like Factory/Resolver.

---

## 🔑 Core Philosophy

| Hilt / Dagger | Swift DI |
|---|---|
| Annotation-driven, compile-time dependency graph | No official framework — manual injection is the Apple-recommended default |
| `@Inject`, `@Module`, `@Provides` | Initializer injection, `@Environment`, or 3rd-party (Factory, Resolver, Swinject) |
| Scoped components (`@Singleton`, `@ActivityScoped`) | Manual lifetime management, or `@EnvironmentObject` for app-wide sharing |
| `hiltViewModel()` for ViewModel injection | Manual `init` in the View, or a DI container passed via Environment |

---

## 🏗 Manual (Constructor) Injection — The "Default" iOS Pattern

**Kotlin (what Hilt automates)**
```kotlin
class UserRepository @Inject constructor(
    private val api: ApiService,
    private val db: UserDao
)
```

**Swift (manual — most common in small/medium projects)**
```swift
class UserRepository {
    private let api: APIService
    private let db: UserDatabase

    init(api: APIService, db: UserDatabase) {
        self.api = api
        self.db = db
    }
}

// Wiring happens manually, usually in a "Composition Root" / AppFactory
let repository = UserRepository(api: APIService(), db: UserDatabase())
let viewModel = UserViewModel(repository: repository)
```

> Apple's own guidance and most SwiftUI sample code favor **plain constructor injection with manual wiring** — no magic, no annotation processing, no build-time codegen. For small-to-medium apps this is entirely sufficient and is what most teams start with.

---

## 🌍 App-Wide Shared Dependencies: @EnvironmentObject

**Hilt (Singleton scope)**
```kotlin
@Singleton
class SessionManager @Inject constructor() { /* ... */ }
```

**SwiftUI (@EnvironmentObject / @Environment)**
```swift
@Observable
class SessionManager {
    var isLoggedIn = false
}

// Injected at the root:
WindowGroup {
    ContentView()
        .environment(sessionManager)
}

// Accessed anywhere below in the hierarchy:
struct SomeChildView: View {
    @Environment(SessionManager.self) private var session
}
```

`@Environment`/`@EnvironmentObject` is the closest built-in SwiftUI equivalent to a Hilt singleton scoped to the app — injected once at the root, available implicitly to any descendant view without manual threading through every initializer.

---

## 🏭 DI Containers / Frameworks (Closer to Dagger/Hilt)

For larger teams that want compile-time safety and less manual wiring, popular options include:

**Factory** (popular, lightweight, Swift-native)
```swift
extension Container {
    var apiService: Factory<APIServiceProtocol> {
        Factory(self) { APIService() }
    }
    var userRepository: Factory<UserRepository> {
        Factory(self) { UserRepository(api: self.apiService()) }
    }
}

// Usage
class UserViewModel {
    @Injected(\.userRepository) var repository
}
```

**Resolver / Swinject** — similar service-locator-style patterns, registering factories/types and resolving them at injection points.

None of these are as deeply integrated or compiler-enforced as Dagger's annotation processor, but they solve the same core problem: centralizing dependency wiring and avoiding manual object graphs scattered everywhere.

---

## 🧪 Injecting into a ViewModel (SwiftUI + @Observable)

**Compose (Hilt)**
```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel()

@Composable
fun UserScreen(viewModel: UserViewModel = hiltViewModel()) { /* ... */ }
```

**SwiftUI (manual)**
```swift
@Observable
class UserViewModel {
    private let repository: UserRepository

    init(repository: UserRepository) {
        self.repository = repository
    }
}

struct UserScreen: View {
    @State private var viewModel: UserViewModel

    init(repository: UserRepository) {
        _viewModel = State(initialValue: UserViewModel(repository: repository))
    }
}
```

There's no `hiltViewModel()`-style auto-resolution — the caller (often a coordinator, factory, or parent view) is responsible for constructing and passing in the ViewModel with its dependencies.

---

## 📝 Quick Reference Table

| Concept | Hilt / Dagger | Swift |
|---|---|---|
| Official framework | ✅ Yes (Google-maintained) | ❌ None (Apple-endorsed pattern is manual DI) |
| Constructor injection | `@Inject constructor` | Plain `init(...)` |
| App-wide singleton | `@Singleton` | `@Environment` / `@EnvironmentObject` |
| ViewModel injection | `hiltViewModel()` | Manual `init` / factory |
| Compile-time graph verification | ✅ Yes | ❌ Only with 3rd-party (Factory, etc.) |
| Popular 3rd-party alternative | — | Factory, Resolver, Swinject |
| Scoping | `@ActivityScoped`, `@ViewModelScoped`, etc. | Manual lifetime ownership |

---
