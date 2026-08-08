# 15 - Database: Room vs SwiftData / Core Data

> Room is Android's official SQLite abstraction layer. iOS has two options: **Core Data** (mature, Objective-C-era, object-graph based) and **SwiftData** (modern, Swift-native, introduced iOS 17 — Apple's answer to Room's ergonomics).

---

## 🔑 Core Philosophy

| Room | SwiftData | Core Data |
|---|---|---|
| Annotation-based entities on top of SQLite | Macro-based models (`@Model`) on top of Core Data | Object graph + `.xcdatamodeld` visual schema |
| DAO interfaces | Direct model queries via `@Query`/`ModelContext` | `NSFetchRequest` / `NSManagedObject` |
| Compile-time SQL verification | Compile-time type safety | Runtime string-based predicates |

> For new projects (iOS 17+), **SwiftData** is the modern default and the closer conceptual match to Room. Core Data is still common in older/larger codebases.

---

## 🗂 Defining an Entity/Model

**Room**
```kotlin
@Entity(tableName = "users")
data class UserEntity(
    @PrimaryKey val id: Int,
    val name: String,
    val email: String
)
```

**SwiftData**
```swift
import SwiftData

@Model
class UserEntity {
    @Attribute(.unique) var id: Int
    var name: String
    var email: String

    init(id: Int, name: String, email: String) {
        self.id = id
        self.name = name
        self.email = email
    }
}
```

`@Model` macro ≈ Room's `@Entity` — it auto-generates the persistence machinery. Note SwiftData models are **classes**, not structs (they need reference/identity semantics for the persistence framework to track changes).

---

## 📖 DAO vs Direct Queries

**Room (DAO)**
```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM users WHERE id = :id")
    suspend fun getUser(id: Int): UserEntity?

    @Insert
    suspend fun insertUser(user: UserEntity)

    @Delete
    suspend fun deleteUser(user: UserEntity)
}
```

**SwiftData (no separate DAO — query via ModelContext / @Query)**
```swift
// Fetching
@Query(filter: #Predicate<UserEntity> { $0.id == targetId })
var users: [UserEntity]

// Inserting
modelContext.insert(UserEntity(id: 1, name: "Rehan", email: "r@example.com"))

// Deleting
modelContext.delete(user)

// Saving
try modelContext.save()
```

SwiftData has no DAO interface concept — you interact with a `ModelContext` directly, or use the `@Query` property wrapper inside a SwiftUI View for automatically-updating fetch results (similar to Room's `Flow`-returning DAO methods combined with `collectAsState()`).

---

## 🗄 Database Setup

**Room**
```kotlin
@Database(entities = [UserEntity::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}

val db = Room.databaseBuilder(context, AppDatabase::class.java, "app_db").build()
```

**SwiftData**
```swift
let container = try ModelContainer(for: UserEntity.self)

// Injected into the SwiftUI environment:
WindowGroup {
    ContentView()
}
.modelContainer(container)
```

`ModelContainer` ≈ Room's `AppDatabase` — the top-level object that owns the schema and storage. It's typically attached once at the app root via `.modelContainer()`, making `ModelContext` available anywhere in the view hierarchy via `@Environment(\.modelContext)`.

---

## 🔄 Reactive Queries in UI

**Compose**
```kotlin
val users by userDao.getAllUsersFlow().collectAsState(initial = emptyList())

LazyColumn {
    items(users) { user -> Text(user.name) }
}
```

**SwiftUI**
```swift
struct UserListView: View {
    @Query private var users: [UserEntity]

    var body: some View {
        List(users) { user in
            Text(user.name)
        }
    }
}
```

`@Query` in SwiftData automatically re-fetches and re-renders the view when underlying data changes — directly analogous to Room's `Flow<List<T>>` + `collectAsState()`.

---

## 🔀 Relationships

**Room**
```kotlin
data class UserWithOrders(
    @Embedded val user: UserEntity,
    @Relation(parentColumn = "id", entityColumn = "userId")
    val orders: List<OrderEntity>
)
```

**SwiftData**
```swift
@Model
class UserEntity {
    var id: Int
    var name: String
    @Relationship(deleteRule: .cascade) var orders: [OrderEntity] = []
}
```

SwiftData relationships are declared directly on the model with `@Relationship`, including cascade delete rules — conceptually similar to Room's `@Relation` + foreign key setup, but more tightly integrated into the model definition itself.

---

## 📝 Quick Reference Table

| Concept | Room | SwiftData |
|---|---|---|
| Entity definition | `@Entity` data class | `@Model` class |
| Primary key | `@PrimaryKey` | `@Attribute(.unique)` |
| Database container | `RoomDatabase` | `ModelContainer` |
| Query interface | `@Dao` interface | `ModelContext` / `@Query` |
| Reactive fetch in UI | `Flow` + `collectAsState()` | `@Query` (auto-reactive) |
| Relationships | `@Relation` | `@Relationship` |
| Migrations | `Migration` objects + version bump | `VersionedSchema` + `SchemaMigrationPlan` |
| Older/legacy alternative | — | Core Data (`NSManagedObject`) |

---

