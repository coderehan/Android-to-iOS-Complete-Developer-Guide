# 34 - Deep Linking: Intent Filters vs Universal Links

> Both platforms support two flavors of deep linking: custom URL schemes (simple, less secure) and verified domain-based links (Android App Links / iOS Universal Links) that open the app directly from a real HTTPS URL without a chooser dialog.

---

## 🔑 Core Comparison

| Android | iOS |
|---|---|
| Custom scheme (`myapp://`) via Intent Filter | Custom scheme (`myapp://`) via `CFBundleURLTypes` |
| App Links (verified `https://` domain) | Universal Links (verified `https://` domain) |
| `assetlinks.json` hosted on your domain | `apple-app-site-association` (AASA) file hosted on your domain |
| `navController.handleDeepLink()` | `.onOpenURL` / `.onContinueUserActivity` |

---

## 🔗 Custom URL Scheme

**Android**
```xml
<!-- AndroidManifest.xml -->
<activity android:name=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="myapp" android:host="details" />
    </intent-filter>
</activity>
```

**iOS**
```xml
<!-- Info.plist -->
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLSchemes</key>
        <array><string>myapp</string></array>
    </dict>
</array>
```

```swift
.onOpenURL { url in
    // url = myapp://details/123
    handleDeepLink(url)
}
```

Both trigger when a link like `myapp://details/123` is tapped anywhere (browser, another app, notification) — but custom schemes on both platforms can be **claimed by multiple apps**, so there's no guarantee your app opens rather than a conflicting one.

---

## 🌐 Verified Domain Links (App Links / Universal Links)

These solve the custom-scheme conflict problem: a real `https://` URL opens your app directly (if installed) or falls back to the website (if not) — no chooser dialog, no scheme collisions.

**Android (App Links)**
```xml
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="https" android:host="example.com" android:pathPrefix="/details" />
</intent-filter>
```

Hosted at `https://example.com/.well-known/assetlinks.json`:
```json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.example.app",
    "sha256_cert_fingerprints": ["..."]
  }
}]
```

**iOS (Universal Links)**
```swift
.onContinueUserActivity(NSUserActivityTypeBrowsingWeb) { userActivity in
    guard let url = userActivity.webpageURL else { return }
    handleDeepLink(url)
}
```

Hosted at `https://example.com/.well-known/apple-app-site-association` (no file extension, served with `application/json` content type):
```json
{
  "applinks": {
    "details": [{
      "appID": "TEAMID.com.example.app",
      "paths": ["/details/*"]
    }]
  }
}
```

Plus enabling the **Associated Domains** capability in Xcode with `applinks:example.com`.

Both mechanisms follow the same trust model: your app declares the domain it wants to handle, and a file hosted on that domain (which only you control) confirms the app is authorized — proving ownership without requiring a manual App Store/Play Store review step for the linking itself.

---

## 🧭 Handling the Link + Navigating

**Android**
```kotlin
composable(
    "details/{id}",
    deepLinks = listOf(navDeepLink { uriPattern = "https://example.com/details/{id}" })
) { backStackEntry -> DetailsScreen(backStackEntry.arguments?.getString("id")) }
```

**iOS**
```swift
.onOpenURL { url in
    if let id = extractId(from: url) {
        path.append(DetailsRoute(id: id))
    }
}
```

Both ultimately parse the incoming URL and push the relevant screen onto the navigation stack (see [08 - Navigation](./08-Navigation.md)).

---

## 📝 Quick Reference Table

| Concept | Android | iOS |
|---|---|---|
| Custom scheme | Intent Filter (`myapp://`) | `CFBundleURLTypes` |
| Verified domain link | App Links | Universal Links |
| Verification file | `assetlinks.json` | `apple-app-site-association` |
| Hosting path | `/.well-known/assetlinks.json` | `/.well-known/apple-app-site-association` |
| Capability to enable | `autoVerify="true"` on Intent Filter | Associated Domains capability |
| Handling callback | `navDeepLink` in NavHost | `.onOpenURL` / `.onContinueUserActivity` |

---
