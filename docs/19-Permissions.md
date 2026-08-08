# 19 - Permissions: Android Runtime Permissions vs iOS Usage Descriptions

> Android asks for permission at runtime with an explicit request/response flow. iOS ties permission prompts to the **first actual use** of the API, driven by a required description string in `Info.plist`.

---

## 🔑 Core Philosophy

| Android | iOS |
|---|---|
| Declare in `AndroidManifest.xml` + request at runtime | Declare usage description in `Info.plist`; system prompts automatically on first API use |
| Explicit `requestPermissions()` call | No explicit "request" call — prompt fires when you touch the protected API |
| Can re-request if denied (with rationale) | Denied once → must send user to Settings app to change (no in-app re-prompt) |
| Permissions can be "granted once" or "always" (Android 11+) | Similar "Allow Once" / "Allow While Using App" / "Always" for location |

---

## 📷 Requesting a Permission (Camera Example)

**Android**
```kotlin
// Manifest
<uses-permission android:name="android.permission.CAMERA" />

// Runtime request
val launcher = registerForActivityResult(
    ActivityResultContracts.RequestPermission()
) { isGranted ->
    if (isGranted) { /* proceed */ }
}

launcher.launch(Manifest.permission.CAMERA)
```

**iOS**
```xml
<!-- Info.plist -->
<key>NSCameraUsageDescription</key>
<string>We need camera access to scan documents</string>
```

```swift
import AVFoundation

// The prompt fires automatically the first time you call this:
AVCaptureDevice.requestAccess(for: .video) { granted in
    if granted { /* proceed */ }
}
```

There's no separate "ask permission" step distinct from "use the API" on iOS — the *first call* to a protected API (camera session, location manager, etc.) triggers the system prompt automatically, as long as the `Info.plist` description key is present. **Missing the key crashes the app** the moment that API is touched.

---

## 🗺 Location Permission

**Android**
```kotlin
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

ActivityCompat.requestPermissions(
    this, arrayOf(Manifest.permission.ACCESS_FINE_LOCATION), REQUEST_CODE
)
```

**iOS**
```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>We use your location to show nearby stores</string>
```

```swift
import CoreLocation

let manager = CLLocationManager()
manager.requestWhenInUseAuthorization()
```

Both platforms distinguish "while using the app" vs "always" (background) location — Android via separate `ACCESS_BACKGROUND_LOCATION`, iOS via `NSLocationAlwaysAndWhenInUseUsageDescription` + `requestAlwaysAuthorization()`.

---

## 🔔 Notification Permission

**Android (13+)**
```kotlin
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />

launcher.launch(Manifest.permission.POST_NOTIFICATIONS)
```

**iOS**
```swift
import UserNotifications

UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .sound, .badge]) { granted, error in
    // handle result
}
```

No `Info.plist` key is required for notifications specifically — the `requestAuthorization` call itself triggers the native system dialog directly.

---

## ✅ Checking Current Permission Status

**Android**
```kotlin
val granted = ContextCompat.checkSelfPermission(
    context, Manifest.permission.CAMERA
) == PackageManager.PERMISSION_GRANTED
```

**iOS**
```swift
let status = AVCaptureDevice.authorizationStatus(for: .video)
let granted = status == .authorized
```

---

## 🚫 Handling Denial

**Android** — can show a rationale and re-request, or direct to Settings if permanently denied (`shouldShowRequestPermissionRationale`).

**iOS** — once denied, **there is no in-app re-prompt**. You must detect the `.denied` status and manually direct the user to Settings:
```swift
if let settingsURL = URL(string: UIApplication.openSettingsURLString) {
    UIApplication.shared.open(settingsURL)
}
```

> ⚠️ This is a meaningful UX difference: on Android you get more flexibility for a second in-app attempt with rationale; on iOS, after the first denial, your only path back is deep-linking the user into the Settings app.

---

## 📝 Quick Reference Table

| Concept | Android | iOS |
|---|---|---|
| Declaration | `AndroidManifest.xml` `<uses-permission>` | `Info.plist` usage description key |
| Trigger | Explicit `requestPermissions()` call | Automatic on first protected API call |
| Re-prompt after denial | ✅ Possible with rationale | ❌ Must redirect to Settings |
| Check current status | `checkSelfPermission()` | `authorizationStatus(for:)` |
| Camera key | `CAMERA` | `NSCameraUsageDescription` |
| Location key | `ACCESS_FINE_LOCATION` | `NSLocationWhenInUseUsageDescription` |
| Notifications | `POST_NOTIFICATIONS` (13+) | `UNUserNotificationCenter.requestAuthorization` |
| Deep link to app settings | `Settings.ACTION_APPLICATION_DETAILS_SETTINGS` intent | `UIApplication.openSettingsURLString` |

---
