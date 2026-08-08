# 01 - Programming Language: Kotlin vs Swift

> If you know Kotlin, Swift will feel very familiar. Both are modern, type-safe, concise languages designed to replace an older, more verbose language (Java → Kotlin, Objective-C → Swift).

---

## 🔑 Core Philosophy

| Kotlin | Swift |
|---|---|
| Null safety via `?` and `!!` | Optionals via `?` and `!` |
| Runs on JVM | Compiles to native machine code |
| Interop with Java | Interop with Objective-C |
| `val` / `var` | `let` / `var` |
| Data classes | Structs (with synthesized `Equatable`) |
| Sealed classes | Enums with associated values |
| Coroutines | async/await + Structured Concurrency |
| Extension functions | Extensions |

---

## 📦 Variables & Constants

**Kotlin**
```kotlin
val name: String = "Rehan"   // immutable
var age: Int = 28             // mutable
```

**Swift**
```swift
let name: String = "Rehan"   // immutable
var age: Int = 28             // mutable
```

Same concept, same keyword-pattern intent — `val`→`let`, `var`→`var`.

---

## ❓ Null Safety / Optionals

**Kotlin**
```kotlin
var city: String? = null
city?.let { print(it) }
val length = city!!.length  // force unwrap, crashes if null
```

**Swift**
```swift
var city: String? = nil
if let city = city {
    print(city)
}
let length = city!.count  // force unwrap, crashes if nil
```

Both languages force you to explicitly handle nullability. Swift's `if let` / `guard let` map closely to Kotlin's `?.let { }`.

---

## 🏗 Classes & Structs

Kotlin only has classes (reference types). Swift has **both** classes (reference types) and **structs** (value types) — and SwiftUI favors structs heavily.

**Kotlin (data class)**
```kotlin
data class User(val id: Int, val name: String)
```

**Swift (struct)**
```swift
struct User {
    let id: Int
    let name: String
}
```

> ⚠️ This is one of the biggest mental shifts for Android developers: in iOS, prefer `struct` over `class` unless you specifically need reference semantics (shared mutable state, identity).

---

## 🔀 Sealed Classes vs Enums with Associated Values

**Kotlin**
```kotlin
sealed class Result
data class Success(val data: String) : Result()
data class Error(val message: String) : Result()
```

**Swift**
```swift
enum Result {
    case success(data: String)
    case failure(message: String)
}
```

Swift enums with associated values are the direct equivalent of Kotlin sealed classes — used constantly for state modeling (loading/success/error).

---

## ⚡ Coroutines vs async/await

**Kotlin**
```kotlin
suspend fun fetchUser(): User {
    return withContext(Dispatchers.IO) {
        api.getUser()
    }
}
```

**Swift**
```swift
func fetchUser() async throws -> User {
    try await api.getUser()
}
```

Kotlin's `suspend fun` maps to Swift's `async func`. Kotlin's `try/catch` maps to Swift's `try`/`catch` combined with `throws`. Structured concurrency exists in both (`coroutineScope` ↔ `TaskGroup`).

---

## 🧩 Extension Functions

**Kotlin**
```kotlin
fun String.isValidEmail(): Boolean {
    return this.contains("@")
}
```

**Swift**
```swift
extension String {
    func isValidEmail() -> Bool {
        return self.contains("@")
    }
}
```

Nearly identical concept and usage.

---

## 📝 Quick Reference Table

| Concept | Kotlin | Swift |
|---|---|---|
| Immutable variable | `val` | `let` |
| Mutable variable | `var` | `var` |
| Nullable type | `String?` | `String?` |
| Force unwrap | `!!` | `!` |
| Safe call | `?.` | `?.` |
| Null coalescing | `?:` | `??` |
| Data model | `data class` | `struct` |
| Union/state type | `sealed class` | `enum` (with associated values) |
| Async function | `suspend fun` | `async func` |
| Lambda | `{ x -> x * 2 }` | `{ x in x * 2 }` |
| Extension | `fun String.foo()` | `extension String { func foo() }` |
| Interface | `interface` | `protocol` |
| Singleton | `object` | `static let shared` pattern |

---

