# 41 - iOS CheatSheet: The Master Quick Reference

> One page to rule them all — every Android → iOS mapping from this entire guide, condensed. Bookmark this one.

---

## 🗣 Language

| Android (Kotlin) | iOS (Swift) |
|---|---|
| `val` | `let` |
| `var` | `var` |
| `String?` | `String?` |
| `!!` | `!` |
| `?.` | `?.` |
| `?:` | `??` |
| `data class` | `struct` |
| `sealed class` | `enum` (associated values) |
| `suspend fun` | `async func` |
| `interface` | `protocol` |
| `object` (singleton) | `static let shared` |
| `operator fun invoke()` | `func callAsFunction()` |

---

## 🎨 UI

| Android (Compose) | iOS (SwiftUI) |
|---|---|
| `@Composable fun` | `struct: View` |
| `Column` | `VStack` |
| `Row` | `HStack` |
| `Box` | `ZStack` |
| `LazyColumn` | `List` |
| `Modifier` chain | View modifier chain |
| `remember { mutableStateOf() }` | `@State` |
| `StateFlow` + `collectAsState()` | `@Published` / `@Observable` |
| `@Preview` | `#Preview` |
| `NavHost` / `NavController` | `NavigationStack` |
| `AnimatedVisibility` | `.transition()` |
| `animate*AsState` | `.animation(_:value:)` |

---

## 🏗 Architecture & DI

| Android | iOS |
|---|---|
| `ViewModel` | `@Observable` class |
| Hilt / Dagger | Manual DI / Factory / Resolver |
| `@Singleton` | `@Environment` object |
| Repository pattern | Repository pattern (same) |
| Use Case / Interactor | Use Case (same term) |

---

## 🌐 Data

| Android | iOS |
|---|---|
| Retrofit | Alamofire / `URLSession` |
| Moshi/Gson | `Codable` (built-in) |
| Room | SwiftData (or Core Data) |
| SharedPreferences | `UserDefaults` |
| DataStore | `@AppStorage` |
| EncryptedSharedPreferences | `Keychain` |
| Coil | Kingfisher |

---

## 📱 Platform APIs

| Android | iOS |
|---|---|
| `AndroidManifest.xml` | `Info.plist` |
| Runtime Permissions | Usage Description keys (`NS...UsageDescription`) |
| CameraX | AVFoundation |
| Google Maps SDK | MapKit |
| BiometricPrompt | LocalAuthentication (`LAContext`) |
| WorkManager | BGTaskScheduler (less guaranteed) |
| FCM (`FirebaseMessagingService`) | APNs via Firebase (`UNUserNotificationCenterDelegate`) |
| Intent Filters (App Links) | Universal Links |
| `ConnectivityManager` | `NWPathMonitor` |
| `MediaStore` | `PHPhotoLibrary` |

---

## 🏢 Lifecycle

| Android | iOS |
|---|---|
| `onCreate`/`onStart`/`onResume` | `.active` scene phase |
| `onPause`/`onStop` | `.inactive` / `.background` |
| Fragment `onViewCreated`/`onDestroyView` | `.onAppear` / `.onDisappear` |
| `LaunchedEffect(Unit)` | `.task { }` |
| `onSaveInstanceState` | `@SceneStorage` |

---

## 🛠 Tooling

| Android | iOS |
|---|---|
| Android Studio | Xcode |
| Gradle | Xcode Build System / SPM |
| `./gradlew` | `xcodebuild` |
| AVD / Emulator | Simulator |
| `adb` | `xcrun simctl` |
| Logcat | Console / `os.Logger` |
| Layout Inspector | Debug View Hierarchy |
| Android Profiler | Instruments |
| LeakCanary | Instruments → Leaks |
| JUnit + MockK | XCTest + manual mocks |
| Espresso | XCUITest |
| ktlint/detekt | SwiftLint/SwiftFormat |

---

## 🚀 Release

| Android | iOS |
|---|---|
| `.aab` | `.ipa` |
| Google Play Console | App Store Connect |
| Automated review (hours) | Manual review (24–48+ hrs) |
| Staged rollout (%) | Phased release (7-day curve) |
| Internal/Closed/Open testing | TestFlight |
| Data Safety form | App Privacy "Nutrition Labels" |
| ProGuard/R8 mapping | `.dSYM` |

---

🎉 **That's the full guide.** If you know Android deeply, you now have every mapping needed to ramp up on iOS fast. Happy building!

