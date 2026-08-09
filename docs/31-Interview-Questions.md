# 31 - Interview Questions: iOS for Android Developers

> Common iOS interview questions, framed with the Android concept you already know alongside them — useful for quick prep before an iOS-track interview.

---

## 🧠 Language Fundamentals

**1. What's the difference between `struct` and `class` in Swift, and when do you choose one over the other?**
> Structs are value types (copied on assignment); classes are reference types (shared). Prefer structs unless you need identity, inheritance, or shared mutable state. (Similar reasoning to when Kotlin devs choose `data class` for immutable models vs a regular class needing shared reference behavior.)

**2. Explain Optionals. How is `String?` different from `String`?**
> `String?` may hold `nil`; the compiler forces you to unwrap it before use — directly analogous to Kotlin's `String?` nullable type and `!!`/`?.` handling.

**3. What is a `guard` statement, and how does it differ from `if let`?**
> `guard let` requires an early exit (`return`/`throw`/`break`) if the condition fails, keeping the "happy path" unindented — similar in spirit to Kotlin's early-return pattern with `?: return`.

**4. What's the difference between `async/await` and `Combine`?**
> `async/await` is for sequential asynchronous work (like Kotlin's `suspend fun`); `Combine` is a reactive streams framework for continuous event pipelines (closer to Kotlin `Flow`).

---

## 🎨 SwiftUI / UI

**5. How does SwiftUI decide when to re-render a view?**
> When `@State`, `@Published`/`@Observable` properties, or `@Binding` values the view reads change — same principle as Compose recomposition triggered by reading a `State` object.

**6. What's the difference between `@State`, `@StateObject`, and `@ObservedObject`?**
> `@State` for local value types; `@StateObject` for reference-type objects the view *creates and owns*; `@ObservedObject` for reference-type objects *passed in* from a parent. (See [09 - State Management](./09-State-Management.md).)

**7. How would you implement a shared element / hero transition in SwiftUI?**
> `matchedGeometryEffect` with a shared `@Namespace` — comparable to Compose's `SharedTransitionLayout`.

---

## 🏗 Architecture

**8. How would you structure a SwiftUI app using MVVM?**
> View (dumb, renders state) → ViewModel (`@Observable`, holds state + calls use cases) → Repository (protocol-based abstraction over data sources) → Data sources (network/persistence). Identical layering to Android MVVM + Clean Architecture (see [17 - Architecture](./17-Architecture.md)).

**9. Why does iOS not have an official DI framework like Hilt?**
> Apple favors explicit constructor injection; DI containers (Factory, Resolver) exist as community solutions rather than an official framework. Discuss trade-offs: less magic/build-time codegen, more manual wiring.

---

## 🗄 Data & Persistence

**10. When would you choose SwiftData over Core Data?**
> SwiftData (iOS 17+) for new projects — modern, Swift-native, less boilerplate. Core Data for apps needing to support older iOS versions or with an existing large Core Data schema.

**11. Where would you store an auth token securely on iOS?**
> Keychain — never `UserDefaults`, which is unencrypted (same reasoning as `EncryptedSharedPreferences` vs plain `SharedPreferences` on Android).

---

## ⚙️ System / Platform

**12. Why can't you rely on a background task running exactly when scheduled on iOS?**
> `BGTaskScheduler`'s `earliestBeginDate` is a hint, not a guarantee — the OS decides based on battery, usage patterns, and Low Power Mode (see [23 - Background Tasks](./23-Background-Tasks.md)). This is a common "gotcha" question testing whether you understand iOS's stricter background execution model vs Android's WorkManager guarantees.

**13. What's the difference between the iOS Simulator and a physical device for testing?**
> The Simulator doesn't fully replicate ARM hardware behavior, camera, push notifications (real APNs delivery), or precise performance characteristics — similar caveat to Android's AVD vs physical device gap, but more pronounced on iOS.

**14. Walk through what happens when a user denies a permission on iOS vs Android.**
> Tests understanding that iOS gives no in-app re-prompt after denial — you must deep-link to Settings, unlike Android's rationale + re-request flow (see [19 - Permissions](./19-Permissions.md)).

---

## 🚀 App Store & Release

**15. Why might App Store review take longer than Google Play review?**
> Apple uses human reviewers; Google Play review is largely automated. Also discuss App Privacy "nutrition label" requirements as a common rejection cause.

---

## 💡 Tips for the Interview

- Lean on your architecture/Clean Architecture experience — it's the strongest transferable skill and interviewers value it highly.
- Be upfront about being new to Swift/SwiftUI specifics, but frame answers by explicitly bridging from Android concepts you know deeply — this signals fast ramp-up ability rather than a knowledge gap.
- Expect at least one question probing iOS-specific constraints (background execution, App Store review, memory management differences) — these are the areas Android experience doesn't automatically cover.

---
