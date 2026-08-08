# 02 - IDE: Android Studio vs Xcode

> Both are official, vendor-built IDEs tightly integrated with their platform's build system. The workflows map closely once you learn where things moved.

---

## 🖥 High-Level Comparison

| Android Studio | Xcode |
|---|---|
| Built on IntelliJ IDEA | Apple's own IDE (built on LLVM/Clang) |
| Gradle build system | Xcode Build System (xcodebuild) |
| Layout Editor / Compose Preview | Interface Builder / SwiftUI Previews (Canvas) |
| AVD Manager (emulators) | Simulator |
| Logcat | Console / Debug Area |
| Profiler | Instruments |
| `build.gradle` | `project.pbxproj` / Package.swift |
| Gradle Sync | Indexing / Resolve Package Graph |

---

## 📁 Project Files

| Android Studio | Xcode |
|---|---|
| `build.gradle.kts` (module/app level) | `.xcodeproj` / `.xcworkspace` |
| `settings.gradle.kts` | Workspace settings |
| `AndroidManifest.xml` | `Info.plist` |
| `gradle.properties` | Build Settings / `.xcconfig` files |
| `local.properties` | Scheme-based environment configs |

Where Gradle uses `.kts` or Groovy scripts to define build logic, Xcode largely uses GUI-driven "Build Settings" and "Build Phases," plus optional `.xcconfig` files for text-based config (closer to what Android devs are used to).

---

## 🎨 UI Preview

**Android Studio (Jetpack Compose)**
```kotlin
@Preview(showBackground = true)
@Composable
fun MyScreenPreview() {
    MyScreen()
}
```
Rendered live in the **Compose Preview** pane.

**Xcode (SwiftUI)**
```swift
#Preview {
    MyScreen()
}
```
Rendered live in the **Canvas**, right next to your code — conceptually identical to Compose Preview, including live interactivity if you click "Live Preview."

---

## 📱 Emulator vs Simulator

| Android Studio | Xcode |
|---|---|
| AVD Manager → create virtual devices | Xcode → Window → Devices and Simulators |
| Runs a full virtualized Android OS (slower) | Runs a simulated iOS environment (not full ARM emulation — faster, but doesn't test all hardware-level behavior) |
| Can test different API levels | Can test different iOS versions/device sizes |

> ⚠️ Key difference: the iOS Simulator does **not** emulate real ARM hardware the way an AVD emulates a full Android device — it's a native macOS process simulating iOS behavior. For hardware-specific testing (camera, performance, Bluetooth), you'll want a physical device either way, but this matters more on iOS.

---

## 🪵 Logging & Debugging

| Android Studio | Xcode |
|---|---|
| Logcat | Debug Console / Console app |
| `Log.d("TAG", "message")` | `print("message")` or `os_log` |
| Breakpoints, Debugger | Breakpoints, LLDB Debugger |
| Layout Inspector | View Debugger (Debug View Hierarchy) |
| Profiler (CPU/Memory/Network) | Instruments (Time Profiler, Allocations, Network) |

---

## 📦 Dependency Management

| Android Studio | Xcode |
|---|---|
| Gradle (`implementation(...)` in `build.gradle.kts`) | Swift Package Manager (SPM) — add via File → Add Package Dependencies |
| Maven Central / JitPack repos | Swift Package Index / direct GitHub URLs |
| (Older) Maven repos | (Older/legacy) CocoaPods, Carthage |

> Modern iOS projects default to **Swift Package Manager**, the direct equivalent of Gradle dependency declarations — just GUI-driven rather than text-file-driven by default (though `Package.swift` is the text-file equivalent for Swift packages themselves).

---

## ⌨️ Common Shortcuts (macOS)

| Action | Android Studio | Xcode |
|---|---|---|
| Build | `Cmd + F9` | `Cmd + B` |
| Run | `Ctrl + R` | `Cmd + R` |
| Find in file | `Cmd + F` | `Cmd + F` |
| Find in project | `Cmd + Shift + F` | `Cmd + Shift + F` |
| Quick Fix | `Option + Enter` | `Cmd + .` (Fix-it suggestions) |
| Go to declaration | `Cmd + B` | `Cmd + Click` |
| Reformat code | `Cmd + Option + L` | `Ctrl + I` |

---

## 📝 Quick Reference Table

| Concept | Android Studio | Xcode |
|---|---|---|
| Build system | Gradle | Xcode Build System |
| UI preview | Compose Preview | SwiftUI Canvas |
| Virtual device | AVD (Emulator) | Simulator |
| Logs | Logcat | Console |
| Profiling | Android Profiler | Instruments |
| Dependency manager | Gradle | Swift Package Manager |
| Manifest/config file | `AndroidManifest.xml` | `Info.plist` |
| View hierarchy debug | Layout Inspector | View Debugger |

---
