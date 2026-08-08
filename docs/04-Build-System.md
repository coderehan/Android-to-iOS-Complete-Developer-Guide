# 04 - Build System: Gradle vs Xcode Build System

> Android's build system is a general-purpose, script-driven build tool (Gradle). iOS's build system is more GUI-driven and tightly coupled to Xcode, with text-based config available via `.xcconfig` files.

---

## 🔑 Core Philosophy

| Gradle | Xcode Build System |
|---|---|
| Script-based (Kotlin DSL / Groovy) | GUI-based (Build Settings tab), with optional `.xcconfig` text files |
| Task graph execution | Build phases (Compile, Link, Copy Resources, Run Script) |
| Flavors + Build Types | Configurations (Debug/Release) + Schemes |
| Version catalogs / `libs.versions.toml` | Swift Package Manager manifest |

---

## ⚙️ Build Variants

**Android** — Build Types × Product Flavors
```kotlin
android {
    buildTypes {
        debug { }
        release {
            isMinifyEnabled = true
        }
    }
    flavorDimensions += "environment"
    productFlavors {
        create("dev") { applicationIdSuffix = ".dev" }
        create("prod") { }
    }
}
```
Produces variants like `devDebug`, `prodRelease`.

**iOS** — Configurations × Schemes
```
Configurations: Debug, Release (can add custom ones like "Staging")
Schemes: Dev, Prod (each pointing to a Configuration + set of environment variables)
```

There's no direct "flavor × build type" matrix multiplication like Gradle — instead, you typically create **multiple Schemes**, each tied to a **Configuration**, and use `.xcconfig` files or `#if DEBUG` / custom compilation flags to branch environment-specific values (API base URLs, bundle IDs, etc.).

---

## 🏷 Bundle ID / Application ID

| Android | iOS |
|---|---|
| `applicationId "com.example.app"` in Gradle | Bundle Identifier in target's General settings |
| Suffix per flavor (`.dev`, `.staging`) | Separate Bundle ID per Scheme/Configuration (often via `.xcconfig`) |

---

## 📦 Dependency Declaration

**Android (Gradle)**
```kotlin
dependencies {
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("io.coil-kt:coil-compose:2.4.0")
    testImplementation("junit:junit:4.13.2")
}
```

**iOS (Swift Package Manager)**
```swift
// Package.swift, or via Xcode: File → Add Package Dependencies
dependencies: [
    .package(url: "https://github.com/Alamofire/Alamofire.git", from: "5.8.0"),
    .package(url: "https://github.com/onevcat/Kingfisher.git", from: "7.10.0")
]
```

Both resolve transitive dependencies automatically and lock versions (Gradle via `gradle.lockfile`, SPM via `Package.resolved`).

---

## 🧱 Build Phases

Android's Gradle task graph (`assembleDebug`, `compileKotlin`, `mergeResources`, etc.) maps to Xcode's **Build Phases**:

| Gradle Task (conceptual) | Xcode Build Phase |
|---|---|
| `compileKotlin` | Compile Sources |
| `mergeResources` | Copy Bundle Resources |
| Custom Gradle task (`doLast { }`) | Run Script Phase |
| `processDebugManifest` | Process Info.plist |
| Link/Dex step | Link Binary With Libraries |

You can add **Run Script phases** in Xcode the same way you'd add a custom Gradle task — e.g., for SwiftLint, versioning scripts, or Firebase config swaps per environment.

---

## 🔡 Environment Variables / Config

**Android**
```kotlin
buildConfigField("String", "BASE_URL", "\"https://api.dev.example.com\"")
```
Accessed via generated `BuildConfig.BASE_URL`.

**iOS**
```
// Config-Dev.xcconfig
API_BASE_URL = https://api.dev.example.com
```
Injected into `Info.plist` as a placeholder (`$(API_BASE_URL)`), then read at runtime via `Bundle.main.infoDictionary`.

> There's no direct `BuildConfig`-style auto-generated class in iOS — the `.xcconfig` → `Info.plist` → `Bundle.main` chain is the closest equivalent, and it's more manual.

---

## 🚀 Command-Line Builds

**Android**
```bash
./gradlew assembleDebug
./gradlew assembleRelease
./gradlew test
```

**iOS**
```bash
xcodebuild -scheme MyApp -configuration Debug build
xcodebuild -scheme MyApp -configuration Release archive
xcodebuild test -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 15'
```

Both are fully scriptable for CI/CD — `xcodebuild` is the CLI equivalent of `gradlew`.

---

## 📝 Quick Reference Table

| Concept | Android (Gradle) | iOS (Xcode) |
|---|---|---|
| Build config language | Kotlin DSL / Groovy | GUI + `.xcconfig` |
| Variant system | Build Types × Flavors | Configurations + Schemes |
| Dependency file | `build.gradle.kts` | `Package.swift` |
| Lock file | `gradle.lockfile` | `Package.resolved` |
| Custom build step | Gradle task | Run Script Phase |
| Env-specific constants | `BuildConfig` fields | `.xcconfig` → `Info.plist` |
| CLI build tool | `./gradlew` | `xcodebuild` |
| Bundle identifier | `applicationId` | Bundle Identifier |

---
