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

## 🧭 Where to Go Deeper

Every row above links back to a full explanation with code samples:

[01-Language](./01-Programming-Language.md) · [02-IDE](./02-IDE.md) · [03-Structure](./03-Project-Structure.md) · [04-Build](./04-Build-System.md) · [05-UI](./05-UI.md) · [06-Layouts](./06-Layouts.md) · [07-Components](./07-Components.md) · [08-Navigation](./08-Navigation.md) · [09-State](./09-State-Management.md) · [10-Lifecycle](./10-Lifecycle.md) · [11-Networking](./11-Networking.md) · [12-JSON](./12-JSON-Parsing.md) · [13-Images](./13-Image-Loading.md) · [14-LocalStorage](./14-Local-Storage.md) · [15-Database](./15-Database.md) · [16-DI](./16-Dependency-Injection.md) · [17-Architecture](./17-Architecture.md) · [18-Auth](./18-Authentication.md) · [19-Permissions](./19-Permissions.md) · [20-Camera](./20-Camera.md) · [21-Maps](./21-Maps.md) · [22-Push](./22-Push-Notifications.md) · [23-Background](./23-Background-Tasks.md) · [24-Animations](./24-Animations.md) · [25-Performance](./25-Performance.md) · [26-Testing](./26-Testing.md) · [27-Debugging](./27-Debugging.md) · [28-CI/CD](./28-CI-CD.md) · [29-Release](./29-Build-and-Release.md) · [30-Libraries](./30-Best-Libraries.md) · [31-Interview](./31-Interview-Questions.md) · [32-EnvVars](./32-Environment-Variables.md) · [33-Config](./33-Configuration.md) · [34-DeepLinking](./34-Deep-Linking.md) · [35-Offline](./35-Offline-First.md) · [36-Files](./36-File-Handling.md) · [37-DepMgmt](./37-Dependency-Management.md) · [38-Example](./38-Project-Architecture-Example.md) · [39-Commands](./39-Useful-Commands.md) · [40-Resources](./40-Resources.md)

---

🎉 **That's the full guide.** If you know Android deeply, you now have every mapping needed to ramp up on iOS fast. Happy building!

