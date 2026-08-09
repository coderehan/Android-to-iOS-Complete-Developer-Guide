# 37 - Dependency Management: Gradle vs Swift Package Manager

> Both are modern, declarative dependency managers with transitive resolution and lock files. SPM is simpler and more tightly scoped than Gradle (which is a full general-purpose build system) — SPM's job is narrower: just resolving and building package dependencies.

---

## 🔑 Core Comparison

| Gradle | Swift Package Manager (SPM) |
|---|---|
| Full build system (tasks, plugins, DSL) | Focused dependency resolver + package builder |
| `build.gradle.kts` | `Package.swift` (for packages) or Xcode's package graph UI (for apps) |
| Maven Central, Google Maven, JitPack | Swift Package Index, direct GitHub URLs |
| `gradle.lockfile` | `Package.resolved` |
| Version catalogs (`libs.versions.toml`) | No direct equivalent — versions declared inline per dependency |

---

## 📦 Adding a Dependency

**Gradle**
```kotlin
dependencies {
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
}
```

**SPM (via Xcode UI)**
```
File → Add Package Dependencies → paste GitHub URL → choose version rule → Add
```

**SPM (via Package.swift, for library authors)**
```swift
dependencies: [
    .package(url: "https://github.com/Alamofire/Alamofire.git", from: "5.9.0")
],
targets: [
    .target(name: "MyLib", dependencies: ["Alamofire"])
]
```

Most iOS **app** developers add dependencies through the Xcode GUI rather than hand-editing a `Package.swift` — a notable workflow difference from Gradle, where editing the `.kts` file directly is the norm even for app-level projects.

---

## 🔢 Version Resolution Rules

**Gradle**
```kotlin
implementation("com.squareup.retrofit2:retrofit:2.9.0")       // exact
implementation("com.squareup.retrofit2:retrofit:2.9.+")        // dynamic (discouraged)
```

**SPM**
```swift
.package(url: "...", from: "5.9.0")                    // >= 5.9.0, < 6.0.0 (semver-compatible)
.package(url: "...", exact: "5.9.0")                    // exact
.package(url: "...", branch: "main")                    // track a branch (discouraged for prod)
.package(url: "...", "5.0.0"..<"6.0.0")                  // explicit range
```

SPM leans on semantic versioning ranges as the default, idiomatic pattern (`from:`) — closer in spirit to npm's `^` caret ranges than to Gradle's typical exact-pin convention.

---

## 🗃 Local/Modular Packages

**Gradle (local module)**
```kotlin
// settings.gradle.kts
include(":feature:login")

// app/build.gradle.kts
dependencies {
    implementation(project(":feature:login"))
}
```

**SPM (local package)**
```swift
// In Xcode: File → Add Package Dependencies → Add Local...
// Or reference a local path in Package.swift:
.package(path: "../LoginFeature")
```

Both support splitting a large app into local modules/packages for build-time isolation and enforced boundaries — the underlying motivation (faster incremental builds, clearer ownership) is identical.

---

## 🔒 Lock Files

**Gradle** — `gradle.lockfile` (opt-in, must be explicitly enabled via `dependencyLocking { }`).

**SPM** — `Package.resolved` is generated **automatically** the moment you resolve dependencies, and should be committed to git for reproducible builds — no opt-in step needed, unlike Gradle's lock file.

---

## 🆚 CocoaPods & Carthage (Legacy, Still Encountered)

Older iOS projects — especially those you might inherit or interview about — may still use:

| Tool | Analogy |
|---|---|
| **CocoaPods** | Older, Ruby-based dependency manager (`Podfile`, `pod install`) — similar era/role to old Ivy/Maven-only Gradle setups before modern Gradle DSL |
| **Carthage** | Decentralized, build-artifact-based dependency manager — less common today |

Modern projects (post-2020) default to SPM; expect CocoaPods in older/legacy codebases, especially those with heavy Objective-C interop or older third-party SDKs that haven't migrated to SPM yet.

---

## 📝 Quick Reference Table

| Concept | Gradle | Swift Package Manager |
|---|---|---|
| Manifest file | `build.gradle.kts` | `Package.swift` |
| Typical app workflow | Edit `.kts` directly | Xcode GUI (Add Package Dependencies) |
| Lock file | `gradle.lockfile` (opt-in) | `Package.resolved` (automatic) |
| Default version strategy | Exact pin | Semver range (`from:`) |
| Local module reference | `project(":module")` | `.package(path:)` |
| Legacy alternative | Old Maven/Ivy setups | CocoaPods, Carthage |

---
