# 22 - Push Notifications: FCM (Android) vs APNs (iOS)

> Firebase Cloud Messaging works on both platforms, but under the hood it routes through different native push services — FCM directly on Android, and through Apple's APNs on iOS. Understanding APNs is essential even if you use Firebase as your abstraction layer.

---

## 🔑 Core Philosophy

| Android | iOS |
|---|---|
| FCM talks directly to the device | FCM (if used) is a wrapper over **APNs** — Apple's own push service |
| No special provisioning needed | Requires an APNs certificate/key + Push Notifications capability in Xcode |
| `FirebaseMessagingService` | `UNUserNotificationCenterDelegate` + `AppDelegate` hooks |
| Notification channels (Android 8+) | No channels — categories serve a related but different purpose |

---

## 📲 Receiving a Push Notification

**Android (FirebaseMessagingService)**
```kotlin
class MyFirebaseMessagingService : FirebaseMessagingService() {
    override fun onMessageReceived(message: RemoteMessage) {
        val title = message.notification?.title
        val body = message.notification?.body
        showNotification(title, body)
    }

    override fun onNewToken(token: String) {
        // send token to backend
    }
}
```

**iOS (AppDelegate + UNUserNotificationCenterDelegate)**
```swift
class AppDelegate: NSObject, UIApplicationDelegate, UNUserNotificationCenterDelegate {
    func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        // send token to backend (or let Firebase handle it via Messaging.messaging().apnsToken)
    }

    func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        completionHandler([.banner, .sound, .badge])
    }
}
```

SwiftUI apps still need a small `AppDelegate` bridge (via `@UIApplicationDelegateAdaptor`) for push notification setup — there's no pure-SwiftUI way to handle APNs registration.

```swift
@main
struct MyApp: App {
    @UIApplicationDelegateAdaptor(AppDelegate.self) var appDelegate

    var body: some Scene {
        WindowGroup { ContentView() }
    }
}
```

---

## 🔔 Requesting Permission & Registering

**Android (13+, runtime permission)**
```kotlin
// See 19-Permissions.md — POST_NOTIFICATIONS
FirebaseMessaging.getInstance().token.addOnSuccessListener { token ->
    // send to backend
}
```

**iOS**
```swift
UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .sound, .badge]) { granted, _ in
    if granted {
        DispatchQueue.main.async {
            UIApplication.shared.registerForRemoteNotifications()
        }
    }
}
```

iOS requires **two steps**: requesting notification permission from the user AND explicitly calling `registerForRemoteNotifications()` to obtain a device token from APNs — Android's flow is more unified through the FCM SDK.

---

## 🏷 Notification Channels vs Categories

**Android** — Channels group notifications by type and let users control importance/sound per channel:
```kotlin
val channel = NotificationChannel(
    "messages", "Messages", NotificationManager.IMPORTANCE_HIGH
)
notificationManager.createNotificationChannel(channel)
```

**iOS** — No direct equivalent for user-configurable grouping. The closest concept is **Notification Categories**, which define available *actions* (buttons) on a notification, not user-facing importance settings:
```swift
let category = UNNotificationCategory(
    identifier: "MESSAGE_CATEGORY",
    actions: [UNNotificationAction(identifier: "REPLY", title: "Reply", options: [])],
    intentIdentifiers: []
)
UNUserNotificationCenter.current().setNotificationCategories([category])
```

> ⚠️ This is a real gap, not just a naming difference: Android users can mute/adjust specific channels (e.g. "Promotions" vs "Order Updates") right from system settings. iOS gives users only one blanket on/off toggle per app (with sub-toggles for alert style, sound, badge) — there's no per-topic granularity unless you build it yourself in-app.

---

## 🎯 Foreground vs Background Handling

| Android | iOS |
|---|---|
| `onMessageReceived` fires in foreground and background (data messages) | `willPresent` delegate method fires only in foreground |
| Background/killed state handled by system tray automatically for notification-type messages | Background delivery requires `content-available: 1` + background modes capability |

---

## 📝 Quick Reference Table

| Concept | Android (FCM) | iOS (APNs / Firebase) |
|---|---|---|
| Receiving service | `FirebaseMessagingService` | `UNUserNotificationCenterDelegate` |
| Token registration | Automatic via FCM SDK | Two-step: permission + `registerForRemoteNotifications()` |
| Grouping/importance | Notification Channels | Not directly supported (single toggle) |
| Action buttons | `NotificationCompat.Action` | `UNNotificationCategory` + `UNNotificationAction` |
| Required Xcode capability | — | Push Notifications capability + APNs key/cert |
| SwiftUI-only integration | N/A | Requires `@UIApplicationDelegateAdaptor` bridge |

---
