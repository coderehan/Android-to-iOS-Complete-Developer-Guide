# 39 - Useful Commands: iOS CLI Cheat Sheet

> The `xcodebuild`/`xcrun`/`simctl` family is iOS's equivalent of `adb`/`./gradlew` — here's a quick-reference cheat sheet for common terminal tasks.

---

## 🏗 Build & Test

| Task | Android (`./gradlew`) | iOS (`xcodebuild`) |
|---|---|---|
| Build debug | `./gradlew assembleDebug` | `xcodebuild -scheme MyApp -configuration Debug build` |
| Build release | `./gradlew assembleRelease` | `xcodebuild -scheme MyApp -configuration Release build` |
| Run unit tests | `./gradlew test` | `xcodebuild test -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 15'` |
| Clean build | `./gradlew clean` | `xcodebuild clean -scheme MyApp` |
| List schemes | — | `xcodebuild -list` |
| Archive for release | `./gradlew bundleRelease` | `xcodebuild archive -scheme MyApp -archivePath build/MyApp.xcarchive` |

---

## 📱 Simulator Management (`simctl`) — iOS's `adb` equivalent

| Task | Android (`adb`) | iOS (`xcrun simctl`) |
|---|---|---|
| List devices | `adb devices` | `xcrun simctl list devices` |
| Boot a simulator | (emulator starts via AVD Manager) | `xcrun simctl boot "iPhone 15"` |
| Install an app | `adb install app.apk` | `xcrun simctl install booted MyApp.app` |
| Uninstall an app | `adb uninstall com.example.app` | `xcrun simctl uninstall booted com.example.app` |
| Launch an app | `adb shell am start -n com.example/.MainActivity` | `xcrun simctl launch booted com.example.app` |
| Take a screenshot | `adb shell screencap` | `xcrun simctl io booted screenshot out.png` |
| Record video | `adb shell screenrecord` | `xcrun simctl io booted recordVideo out.mov` |
| Reset simulator | `adb shell pm clear com.example.app` | `xcrun simctl erase "iPhone 15"` |
| Open a URL (deep link test) | `adb shell am start -a android.intent.action.VIEW -d "myapp://..."` | `xcrun simctl openurl booted "myapp://..."` |

---

## 📦 Swift Package Manager

| Task | Command |
|---|---|
| Resolve dependencies | `swift package resolve` |
| Update dependencies | `swift package update` |
| Show dependency graph | `swift package show-dependencies` |
| Build a package | `swift build` |
| Test a package | `swift test` |
| Generate Xcode project (older SPM workflows) | `swift package generate-xcodeproj` |

---

## 🔍 Device & Log Inspection

| Task | Android | iOS |
|---|---|---|
| View live logs | `adb logcat` | `xcrun simctl spawn booted log stream --predicate 'subsystem == "com.example.app"'` |
| List installed apps | `adb shell pm list packages` | `xcrun simctl listapps booted` |
| Pull app container/data | `adb pull` | `xcrun simctl get_app_container booted com.example.app data` |
| List connected physical devices | `adb devices` | `xcrun xctrace list devices` |

---

## 🧹 Cache & DerivedData Cleanup (Common First-Debug-Step)

Xcode's build cache (`DerivedData`) is a very common source of "why won't this build" issues — the iOS equivalent of clearing Gradle caches:

```bash
rm -rf ~/Library/Developer/Xcode/DerivedData
```

Comparable to:
```bash
./gradlew clean
rm -rf ~/.gradle/caches
```

---

## 🚀 Fastlane Shortcuts

| Task | Command |
|---|---|
| Run a specific lane | `fastlane ios beta` |
| List available lanes | `fastlane lanes` |
| Sync certs/profiles (Match) | `fastlane match appstore` |
| Upload to TestFlight | `fastlane pilot upload` |

---

## 📝 Quick Reference Table

| Category | Android Tool | iOS Tool |
|---|---|---|
| Build system CLI | `./gradlew` | `xcodebuild` |
| Device/simulator control | `adb` | `xcrun simctl` |
| Package manager CLI | `gradle` (implicit) | `swift package` |
| Log streaming | `adb logcat` | `xcrun simctl spawn booted log stream` |
| Build cache location | `~/.gradle/caches` | `~/Library/Developer/Xcode/DerivedData` |
| Release automation | Fastlane (optional) | Fastlane (near-standard) |

---
