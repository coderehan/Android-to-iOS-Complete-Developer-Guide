# 03 - Project Structure: Android vs iOS

> The folder philosophy is similar — separate concerns, group by feature or layer — but the file types and entry points differ.

---

## 📁 Typical Android Project Structure

```
app/
├── src/
│   ├── main/
│   │   ├── java/com/example/app/
│   │   │   ├── MainActivity.kt
│   │   │   ├── ui/
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── di/
│   │   ├── res/
│   │   │   ├── layout/
│   │   │   ├── drawable/
│   │   │   ├── values/
│   │   │   └── mipmap/
│   │   └── AndroidManifest.xml
│   ├── test/
│   └── androidTest/
├── build.gradle.kts
└── settings.gradle.kts
```

---

## 📁 Typical iOS Project Structure

```
MyApp/
├── MyApp/
│   ├── MyAppApp.swift          (entry point — like MainActivity + Application)
│   ├── ContentView.swift       (root view)
│   ├── Views/
│   ├── ViewModels/
│   ├── Models/
│   ├── Services/
│   ├── Persistence/
│   ├── Resources/
│   │   ├── Assets.xcassets     (like drawable/mipmap)
│   │   └── Localizable.strings (like values/strings.xml)
│   └── Info.plist              (like AndroidManifest.xml)
├── MyAppTests/                 (like test/)
├── MyAppUITests/                (like androidTest/)
├── MyApp.xcodeproj
└── Package.swift (if using SPM as source of truth)
```

---

## 🚪 Entry Point

**Android**
```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyApp()
        }
    }
}
```

**iOS (SwiftUI)**
```swift
@main
struct MyAppApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

The `@main struct: App` is the iOS equivalent of Android's `Application` class + launcher `Activity` combined — it's where the app boots and the root view/scene is declared.

---

## 🗂 Resource Files

| Android | iOS |
|---|---|
| `res/drawable/` | `Assets.xcassets` (image sets) |
| `res/mipmap/` | `Assets.xcassets` (AppIcon set) |
| `res/values/strings.xml` | `Localizable.strings` / String Catalogs |
| `res/values/colors.xml` | `Assets.xcassets` (color sets) |
| `res/values/dimens.xml` | No direct equivalent — usually defined as Swift constants |
| `res/layout/*.xml` | No direct equivalent — SwiftUI is code-first (like Compose) |
| `res/font/` | Fonts added to `Info.plist` + bundled files |

> If you're coming from **Jetpack Compose** (not XML layouts), this will feel very natural — SwiftUI, like Compose, has no separate layout XML files. Everything is declarative Swift code.

---

## 📝 Manifest / Config File

**AndroidManifest.xml**
```xml
<manifest>
    <application android:label="MyApp">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
            </intent-filter>
        </activity>
    </application>
    <uses-permission android:name="android.permission.CAMERA" />
</manifest>
```

**Info.plist** (iOS)
```xml
<key>CFBundleDisplayName</key>
<string>MyApp</string>
<key>NSCameraUsageDescription</key>
<string>We need camera access to scan documents</string>
```

Both declare app metadata, permissions (with usage descriptions on iOS), and configuration. Android permissions are declared and requested at runtime separately; iOS **requires** a human-readable usage description string in `Info.plist` for every sensitive permission — without it, the app crashes when requesting that permission.

---

## 🏗 Module Structure (Multi-Module Apps)

| Android | iOS |
|---|---|
| Gradle modules (`:feature:login`, `:core:network`) | Swift Packages (local SPM packages per feature/layer) |
| `settings.gradle.kts` declares modules | `Package.swift` per local package, referenced in Xcode workspace |

Large iOS codebases increasingly use **local Swift Packages** the same way Android uses Gradle modules — to enforce boundaries and speed up build times via incremental compilation.

---

## 📝 Quick Reference Table

| Concept | Android | iOS |
|---|---|---|
| Entry point | `MainActivity` + `Application` | `@main struct: App` |
| Root UI | `setContent { }` | `WindowGroup { ContentView() }` |
| Images | `res/drawable` | `Assets.xcassets` |
| Strings/localization | `res/values/strings.xml` | `Localizable.strings` |
| Permissions/metadata | `AndroidManifest.xml` | `Info.plist` |
| Unit tests folder | `src/test` | `MyAppTests` |
| UI tests folder | `src/androidTest` | `MyAppUITests` |
| Modularization | Gradle modules | Local Swift Packages |

---
