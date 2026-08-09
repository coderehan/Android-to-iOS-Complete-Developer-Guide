# 30 - Best Libraries: Recommended iOS Stack

> A curated list of the most commonly used iOS libraries, mapped to their Android counterparts, for quick reference when setting up a new project.

---

## 📦 Recommended Stack by Category

| Category | Android | iOS Recommendation |
|---|---|---|
| Language | Kotlin | Swift |
| UI Framework | Jetpack Compose | SwiftUI |
| Networking | Retrofit + OkHttp | Alamofire (or native `URLSession`) |
| JSON Parsing | Moshi / Gson | `Codable` (built-in — no library needed) |
| Image Loading | Coil | Kingfisher |
| Database | Room | SwiftData (iOS 17+) or Core Data (legacy support) |
| Local Key-Value Storage | DataStore / SharedPreferences | `@AppStorage` / `UserDefaults` |
| Secure Storage | EncryptedSharedPreferences | Keychain (raw) or KeychainAccess (wrapper) |
| Dependency Injection | Hilt / Dagger | Manual DI, or Factory / Resolver |
| Async Programming | Coroutines + Flow | async/await + Combine |
| Navigation | Navigation Component | `NavigationStack` (native) |
| Testing (Unit) | JUnit + MockK | XCTest + manual protocol mocks |
| Testing (UI) | Espresso / Compose UI Testing | XCUITest |
| Testing (Snapshot) | Paparazzi / Shot | swift-snapshot-testing |
| CI/CD | GitHub Actions + Fastlane | GitHub Actions (macOS runner) + Fastlane |
| Crash Reporting | Firebase Crashlytics | Firebase Crashlytics |
| Analytics | Firebase Analytics | Firebase Analytics |
| Push Notifications | Firebase Cloud Messaging | Firebase Cloud Messaging (via APNs) |
| Maps | Google Maps SDK | MapKit (native) |
| Camera | CameraX | AVFoundation |
| Lottie Animations | Lottie-Android | lottie-ios |
| Charts | MPAndroidChart / Vico | Swift Charts (native, iOS 16+) |
| Linting/Formatting | ktlint / detekt | SwiftLint / SwiftFormat |

---

## 🌟 iOS-Specific Notables (No Direct Android Equivalent)

| Library | Purpose |
|---|---|
| **Swift Charts** | Native declarative charting framework (Apple-built, SwiftUI-native) |
| **TCA (The Composable Architecture)** | Popular opinionated state-management architecture, more prescriptive than plain MVVM |
| **Nuke** | Alternative to Kingfisher for image loading, popular in performance-sensitive apps |
| **Combine** | Apple's native reactive streams framework — used alongside/before async-await adoption |
| **swift-dependencies** (Point-Free) | Lightweight DI library popular in TCA-based codebases |

---

## 🧩 What Ships Natively (No Extra Dependency Needed)

One notable difference from Android: several things that require an external library on Android ship **built into Apple's platform SDKs**:

- **JSON parsing** → `Codable` (no Moshi/Gson equivalent needed)
- **Charts** → `Swift Charts` (iOS 16+)
- **Networking basics** → `URLSession` (usable without Alamofire for simple apps)
- **Database (modern)** → `SwiftData` (iOS 17+, no Room-equivalent third-party needed)

> When starting a new iOS project, evaluate whether you actually need the "Android-equivalent" library, or whether Apple's native option already covers your needs — this is a common area where Android devs over-add dependencies out of habit.

---

## 📝 Quick Setup Reference (Swift Package Manager)

```swift
// Package.swift dependencies, or via Xcode → File → Add Package Dependencies
dependencies: [
    .package(url: "https://github.com/Alamofire/Alamofire.git", from: "5.9.0"),
    .package(url: "https://github.com/onevcat/Kingfisher.git", from: "7.11.0"),
    .package(url: "https://github.com/airbnb/lottie-ios.git", from: "4.4.0"),
    .package(url: "https://github.com/hmlongco/Factory.git", from: "2.3.0"),
    .package(url: "https://github.com/pointfreeco/swift-snapshot-testing.git", from: "1.15.0")
]
```

---
