# 33 - Configuration: Multi-Environment App Setup

> Beyond raw environment variables, this covers the broader picture: app icons/names per environment, feature flags, and remote config — the operational side of running Dev/Staging/Prod builds side by side.

---

## 🔑 Core Comparison

| Android | iOS |
|---|---|
| Product Flavors → separate `applicationIdSuffix`, icon, name | Multiple Targets or Schemes → separate Bundle ID, icon, name |
| Firebase Remote Config | Firebase Remote Config (same product) |
| Feature flag libraries (LaunchDarkly, etc.) | Same libraries — cross-platform SDKs |
| Multiple `google-services.json` per flavor | Multiple `GoogleService-Info.plist` per target |

---

## 📱 Running Multiple Environments Side-by-Side (Separate App Icon/Name)

**Android** — flavors automatically get separate `applicationIdSuffix`, so Dev and Prod install as distinct apps on the same device:
```kotlin
productFlavors {
    create("dev") {
        applicationIdSuffix = ".dev"
        resValue("string", "app_name", "MyApp Dev")
    }
}
```

**iOS** — requires either **multiple Targets** (each with its own Bundle ID, icon set, and `Info.plist`) or a single target with per-Scheme `.xcconfig` overrides:
```
# Config-Dev.xcconfig
PRODUCT_BUNDLE_IDENTIFIER = com.example.app.dev
PRODUCT_NAME = MyApp Dev
ASSETCATALOG_COMPILER_APPICON_NAME = AppIcon-Dev
```

> Setting up multiple Targets in Xcode is more manual than Gradle's flavor system — you typically duplicate the target, then diverge the `.xcconfig`/Info.plist/asset catalog per environment. It's more setup effort the first time, but functionally achieves the same "Dev and Prod installed side-by-side" result Android gets more automatically.

---

## 🎌 Feature Flags

**Android**
```kotlin
if (remoteConfig.getBoolean("new_checkout_enabled")) {
    NewCheckoutScreen()
} else {
    LegacyCheckoutScreen()
}
```

**iOS**
```swift
if remoteConfig["new_checkout_enabled"].boolValue {
    NewCheckoutScreen()
} else {
    LegacyCheckoutScreen()
}
```

Firebase Remote Config's API is nearly identical across both SDKs — the same dashboard, same rollout/targeting rules, just platform-specific client code.

---

## 🔧 GoogleService Config Files

**Android** — `google-services.json`, one per flavor, placed in `app/src/<flavor>/`.

**iOS** — `GoogleService-Info.plist`, one per Target, added to that Target's build phase (a common CI step swaps in the right file per environment before building):
```bash
# Fastlane / CI script example
cp "GoogleService-Info-${ENVIRONMENT}.plist" "MyApp/GoogleService-Info.plist"
```

---

## 🏷 App Icon & Launch Screen per Environment

**Android** — separate `mipmap` sets per flavor's `res/` source set.

**iOS** — separate image sets in `Assets.xcassets` (e.g. `AppIcon-Dev`, `AppIcon-Prod`), referenced via the `ASSETCATALOG_COMPILER_APPICON_NAME` build setting shown above.

A visually distinct Dev icon (often with a colored badge/ribbon) is a common practice on both platforms so QA/testers can't confuse which build they're using.

---

## 📝 Quick Reference Table

| Concept | Android | iOS |
|---|---|---|
| Multi-environment mechanism | Product Flavors | Multiple Targets or Schemes + `.xcconfig` |
| Distinct bundle/package ID | `applicationIdSuffix` | `PRODUCT_BUNDLE_IDENTIFIER` per config |
| Distinct app name | `resValue("string", "app_name", ...)` | `PRODUCT_NAME` per config |
| Distinct app icon | Per-flavor `mipmap` | Per-config `ASSETCATALOG_COMPILER_APPICON_NAME` |
| Remote config / feature flags | Firebase Remote Config | Firebase Remote Config (same product) |
| Firebase config file | `google-services.json` per flavor | `GoogleService-Info.plist` per target |

---
