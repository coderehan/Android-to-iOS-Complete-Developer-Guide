# 27 - Debugging: Android Studio Debugger vs Xcode/LLDB

> Both IDEs offer full breakpoint debugging, conditional breakpoints, and view-hierarchy inspection. The underlying debuggers differ (JDWP-based for Android vs LLDB for iOS), but the day-to-day workflow feels very similar.

---

## 🔑 Core Comparison

| Android Studio | Xcode |
|---|---|
| JDWP debugger (JVM-based) | LLDB debugger |
| Logcat for logs | Console / `os_log` |
| Layout Inspector | View Debugger (Debug View Hierarchy) |
| Breakpoints, watches, conditional breakpoints | Breakpoints, watches, conditional breakpoints |
| Database Inspector (Room) | No direct equivalent — use third-party (e.g. DB Browser for SQLite on the simulator's file path) |

---

## 🪲 Breakpoints

Both IDEs support standard breakpoints (click the gutter), conditional breakpoints (right-click → add condition), and exception breakpoints.

**Android Studio** — right-click a breakpoint → "Condition" field, e.g. `user.id == "123"`

**Xcode** — right-click a breakpoint → "Edit Breakpoint" → Condition field, same expression-based syntax, e.g. `user.id == "123"`

---

## 🖨 Logging

**Android**
```kotlin
Log.d("MyTag", "User loaded: $user")
Log.e("MyTag", "Error occurred", exception)
```

**iOS**
```swift
print("User loaded: \(user)")

// Structured logging (preferred for production/Console.app filtering)
import os
let logger = Logger(subsystem: "com.example.app", category: "networking")
logger.debug("User loaded: \(user.name)")
logger.error("Error occurred: \(error.localizedDescription)")
```

`os.Logger` is the modern iOS equivalent of Android's tagged `Log.d`/`Log.e` — it supports categories/subsystems (like Log tags) and integrates with the Console app for filtering, similar to filtering Logcat by tag.

---

## 🖥 Interactive Debugger Console

**Android Studio** — Evaluate Expression (Alt+F8) lets you run arbitrary Kotlin/Java expressions while paused at a breakpoint.

**Xcode (LLDB console)**
```
(lldb) po user.name
(lldb) po viewModel.uiState
(lldb) expr user.name = "Changed"
```
`po` ("print object") is LLDB's equivalent of Android Studio's "Evaluate Expression" watch — you can inspect and even mutate live objects while paused.

---

## 🌳 View Hierarchy Inspection

**Android Studio (Layout Inspector)** — Live 3D view of the Compose/View tree, shows recomposition counts, attribute values per node.

**Xcode (Debug View Hierarchy)** — Click the "Debug View Hierarchy" button while paused; gives a live 3D-exploded view of the UIKit/SwiftUI view tree, including frame sizes and constraint conflicts.

Both let you click into any rendered element and see its properties without adding temporary debug code.

---

## 🌐 Network Debugging

**Android** — Android Profiler's Network tab, or Flipper/Chucker for request/response inspection.

**iOS** — Instruments' Network template, or point the simulator's proxy at Charles Proxy / Proxyman for full request/response inspection (very common workflow, since there's no built-in Xcode network inspector as detailed as Android's).

---

## 💥 Crash Reporting & Symbolication

| Android | iOS |
|---|---|
| Stack traces are readable directly (JVM) | Crash logs are often symbol-stripped — requires the `.dSYM` file to symbolicate |
| Firebase Crashlytics (Android SDK) | Firebase Crashlytics (iOS SDK) — same product, same dashboard |
| ProGuard/R8 mapping file needed for obfuscated crashes | `.dSYM` file needed for symbolicated crashes |

Both require uploading a mapping/symbol file (ProGuard mapping vs `.dSYM`) so crash reports show readable function names instead of obfuscated/stripped addresses — this step is easy to forget on both platforms and is a common real-world gotcha.

---

## 📝 Quick Reference Table

| Concept | Android Studio | Xcode |
|---|---|---|
| Debugger engine | JDWP | LLDB |
| Logging | `Log.d/e/w` | `print()` / `os.Logger` |
| Evaluate expression | Alt+F8 | `po` in LLDB console |
| View hierarchy debug | Layout Inspector | Debug View Hierarchy |
| Network inspection | Profiler / Flipper | Instruments / Charles Proxy |
| Crash symbolication | ProGuard/R8 mapping | `.dSYM` file |
| Crash reporting | Firebase Crashlytics | Firebase Crashlytics |

---
