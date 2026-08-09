# 38 - Project Architecture Example: A Real Feature End-to-End

> This walks through one complete feature — a user profile screen — through every architectural layer, so you can see how everything from earlier docs fits together in practice.

---

## 🏗 Folder Structure for This Feature

```
Profile/
├── Presentation/
│   ├── ProfileScreen.swift
│   └── ProfileViewModel.swift
├── Domain/
│   ├── GetUserProfileUseCase.swift
│   └── UserProfile.swift
└── Data/
    ├── UserProfileRepository.swift
    ├── UserProfileRepositoryImpl.swift
    ├── Network/
    │   ├── ProfileAPIService.swift
    │   └── UserProfileDTO.swift
    └── Persistence/
        └── UserProfileEntity.swift
```

(Compare directly to the Android structure in [17 - Architecture](./17-Architecture.md) — same shape, same reasoning.)

---

## 1️⃣ Domain Model (framework-agnostic)

```swift
// Domain/UserProfile.swift
struct UserProfile: Identifiable {
    let id: String
    let name: String
    let email: String
    let avatarURL: URL?
}
```

---

## 2️⃣ Network DTO (matches API shape, not domain shape)

```swift
// Data/Network/UserProfileDTO.swift
struct UserProfileDTO: Codable {
    let userId: String
    let fullName: String
    let emailAddress: String
    let avatarUrl: String?

    enum CodingKeys: String, CodingKey {
        case userId = "user_id"
        case fullName = "full_name"
        case emailAddress = "email"
        case avatarUrl = "avatar_url"
    }

    func toDomain() -> UserProfile {
        UserProfile(
            id: userId,
            name: fullName,
            email: emailAddress,
            avatarURL: avatarUrl.flatMap(URL.init)
        )
    }
}
```

---

## 3️⃣ Persistence Model (SwiftData)

```swift
// Data/Persistence/UserProfileEntity.swift
@Model
class UserProfileEntity {
    @Attribute(.unique) var id: String
    var name: String
    var email: String
    var avatarURLString: String?

    init(from domain: UserProfile) {
        self.id = domain.id
        self.name = domain.name
        self.email = domain.email
        self.avatarURLString = domain.avatarURL?.absoluteString
    }

    func toDomain() -> UserProfile {
        UserProfile(id: id, name: name, email: email, avatarURL: avatarURLString.flatMap(URL.init))
    }
}
```

---

## 4️⃣ Repository (abstracts data sources behind a protocol)

```swift
// Data/UserProfileRepository.swift
protocol UserProfileRepository {
    func getProfile(id: String) async throws -> UserProfile
}

// Data/UserProfileRepositoryImpl.swift
class UserProfileRepositoryImpl: UserProfileRepository {
    private let api: ProfileAPIService
    private let modelContext: ModelContext

    init(api: ProfileAPIService, modelContext: ModelContext) {
        self.api = api
        self.modelContext = modelContext
    }

    func getProfile(id: String) async throws -> UserProfile {
        do {
            let dto = try await api.fetchProfile(id: id)
            let domain = dto.toDomain()
            modelContext.insert(UserProfileEntity(from: domain))
            try? modelContext.save()
            return domain
        } catch {
            // offline fallback (see 35-Offline-First.md)
            let descriptor = FetchDescriptor<UserProfileEntity>(predicate: #Predicate { $0.id == id })
            guard let cached = try modelContext.fetch(descriptor).first else { throw error }
            return cached.toDomain()
        }
    }
}
```

---

## 5️⃣ Use Case

```swift
// Domain/GetUserProfileUseCase.swift
struct GetUserProfileUseCase {
    let repository: UserProfileRepository

    func callAsFunction(id: String) async throws -> UserProfile {
        try await repository.getProfile(id: id)
    }
}
```

---

## 6️⃣ ViewModel

```swift
// Presentation/ProfileViewModel.swift
@Observable
class ProfileViewModel {
    private let getUserProfile: GetUserProfileUseCase

    var profile: UserProfile?
    var isLoading = false
    var errorMessage: String?

    init(getUserProfile: GetUserProfileUseCase) {
        self.getUserProfile = getUserProfile
    }

    func load(id: String) async {
        isLoading = true
        defer { isLoading = false }
        do {
            profile = try await getUserProfile(id: id)
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}
```

---

## 7️⃣ View (dumb, renders state)

```swift
// Presentation/ProfileScreen.swift
struct ProfileScreen: View {
    @State private var viewModel: ProfileViewModel
    let userId: String

    init(userId: String, viewModel: ProfileViewModel) {
        self.userId = userId
        _viewModel = State(initialValue: viewModel)
    }

    var body: some View {
        VStack {
            if viewModel.isLoading {
                ProgressView()
            } else if let profile = viewModel.profile {
                KFImage(profile.avatarURL)
                    .resizable()
                    .frame(width: 80, height: 80)
                    .clipShape(Circle())
                Text(profile.name).font(.title2)
                Text(profile.email).foregroundStyle(.secondary)
            } else if let error = viewModel.errorMessage {
                Text(error).foregroundStyle(.red)
            }
        }
        .task { await viewModel.load(id: userId) }
    }
}
```

---

## 8️⃣ Composition Root (manual DI wiring — see [16 - Dependency Injection](./16-Dependency-Injection.md))

```swift
// AppFactory.swift
func makeProfileScreen(userId: String, modelContext: ModelContext) -> ProfileScreen {
    let api = ProfileAPIService()
    let repository = UserProfileRepositoryImpl(api: api, modelContext: modelContext)
    let useCase = GetUserProfileUseCase(repository: repository)
    let viewModel = ProfileViewModel(getUserProfile: useCase)
    return ProfileScreen(userId: userId, viewModel: viewModel)
}
```

---

## 🔁 Data Flow Summary

```
View.task { }
   → ViewModel.load()
      → UseCase.callAsFunction()
         → Repository.getProfile()
            → API (network) — or → SwiftData (offline fallback)
         ← UserProfile (domain model)
      ← published to ViewModel.profile
   ← View re-renders automatically (@Observable)
```

This is the exact same data flow shape as the Android MVVM + Clean Architecture pattern from [17 - Architecture](./17-Architecture.md) — just Swift syntax throughout.

---
