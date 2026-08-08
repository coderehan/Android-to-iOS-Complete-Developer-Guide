# 17 - Architecture: MVVM & Clean Architecture on Android vs iOS

> The architectural thinking transfers almost 1:1. MVVM, Clean Architecture, and unidirectional data flow aren't Android-specific — they're UI-framework-agnostic patterns that SwiftUI adopts just as naturally as Compose.

---

## 🔑 Core Philosophy

| Android | iOS |
|---|---|
| MVVM is the de facto standard with Compose | MVVM is the de facto standard with SwiftUI |
| Clean Architecture layers: UI → Domain → Data | Same layering: View → Domain → Data |
| `ViewModel` holds UI state, survives config change | `@Observable` class holds UI state, survives view re-render |
| Repository pattern abstracts data sources | Repository pattern — identical concept |

---

## 🧱 Typical Layered Structure

**Android**
```
presentation/
  ├── ui/          (Composables)
  └── viewmodel/   (ViewModels)
domain/
  ├── usecase/     (Use Cases / Interactors)
  └── model/       (Domain models)
data/
  ├── repository/  (Repository implementations)
  ├── remote/      (API service, DTOs)
  └── local/       (Room DAOs, entities)
```

**iOS**
```
Presentation/
  ├── Views/           (SwiftUI Views)
  └── ViewModels/       (@Observable classes)
Domain/
  ├── UseCases/         (Use Cases / Interactors)
  └── Models/            (Domain models)
Data/
  ├── Repositories/      (Repository implementations)
  ├── Network/            (API service, DTOs)
  └── Persistence/        (SwiftData models, Core Data)
```

Same three-layer separation — the folder names and even the reasoning behind the boundaries carry over directly.

---

## 🔄 MVVM Flow

**Android**
```kotlin
// View (Composable) — dumb, observes state
@Composable
fun ProfileScreen(viewModel: ProfileViewModel = hiltViewModel()) {
    val state by viewModel.uiState.collectAsState()
    ProfileContent(state = state, onRefresh = viewModel::refresh)
}

// ViewModel — holds state, calls use cases
class ProfileViewModel @Inject constructor(
    private val getUserUseCase: GetUserUseCase
) : ViewModel() {
    private val _uiState = MutableStateFlow(ProfileUiState())
    val uiState: StateFlow<ProfileUiState> = _uiState.asStateFlow()

    fun refresh() {
        viewModelScope.launch {
            _uiState.update { it.copy(user = getUserUseCase()) }
        }
    }
}
```

**iOS**
```swift
// View — dumb, observes state
struct ProfileScreen: View {
    @State private var viewModel: ProfileViewModel

    var body: some View {
        ProfileContent(state: viewModel.uiState, onRefresh: viewModel.refresh)
    }
}

// ViewModel — holds state, calls use cases
@Observable
class ProfileViewModel {
    private let getUserUseCase: GetUserUseCase
    var uiState = ProfileUiState()

    init(getUserUseCase: GetUserUseCase) {
        self.getUserUseCase = getUserUseCase
    }

    func refresh() {
        Task {
            uiState.user = try await getUserUseCase()
        }
    }
}
```

The pattern is identical: View is a dumb renderer of state, ViewModel owns state + business logic orchestration, Use Case encapsulates a single business operation.

---

## 🧩 Use Case / Interactor Pattern

**Android**
```kotlin
class GetUserUseCase @Inject constructor(
    private val repository: UserRepository
) {
    suspend operator fun invoke(id: String): User = repository.getUser(id)
}
```

**iOS**
```swift
struct GetUserUseCase {
    let repository: UserRepository

    func callAsFunction(id: String) async throws -> User {
        try await repository.getUser(id: id)
    }
}
```

Kotlin's `operator fun invoke()` (making an object callable like a function) maps directly to Swift's `func callAsFunction()` — both let you call the use case instance as if it were a function: `getUserUseCase(id)`.

---

## 🗃 Repository Pattern

**Android**
```kotlin
interface UserRepository {
    suspend fun getUser(id: String): User
}

class UserRepositoryImpl @Inject constructor(
    private val api: ApiService,
    private val dao: UserDao
) : UserRepository {
    override suspend fun getUser(id: String): User {
        return try {
            val remote = api.getUser(id)
            dao.insertUser(remote.toEntity())
            remote
        } catch (e: IOException) {
            dao.getUser(id).toDomain()
        }
    }
}
```

**iOS**
```swift
protocol UserRepository {
    func getUser(id: String) async throws -> User
}

class UserRepositoryImpl: UserRepository {
    let api: APIService
    let db: UserDatabase

    func getUser(id: String) async throws -> User {
        do {
            let remote = try await api.getUser(id: id)
            try db.insert(remote.toEntity())
            return remote
        } catch {
            return try db.getUser(id: id).toDomain()
        }
    }
}
```

Kotlin's `interface` ↔ Swift's `protocol` — both express the same "abstract the data source behind an interface" idea, enabling easy mocking for tests and swapping implementations.

---

## 📝 Quick Reference Table

| Concept | Android | iOS |
|---|---|---|
| Architecture pattern | MVVM + Clean Architecture | MVVM + Clean Architecture |
| UI layer | Composables | SwiftUI Views |
| State holder | `ViewModel` | `@Observable` class |
| Abstraction contract | `interface` | `protocol` |
| Callable object | `operator fun invoke()` | `func callAsFunction()` |
| Business logic unit | Use Case / Interactor | Use Case / Interactor (same term used) |
| Data abstraction | Repository pattern | Repository pattern |
| Domain model vs DTO separation | ✅ Common practice | ✅ Common practice |

---
