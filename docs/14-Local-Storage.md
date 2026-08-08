# 14 - Local Storage: SharedPreferences vs UserDefaults / Keychain

> Both platforms offer a simple key-value store for lightweight app preferences, plus a separate secure store for sensitive data (tokens, passwords).

---

## 🔑 Core Philosophy

| Android | iOS |
|---|---|
| `SharedPreferences` (or Jetpack `DataStore`) | `UserDefaults` |
| Not encrypted by default | Not encrypted by default |
| `EncryptedSharedPreferences` for sensitive data | `Keychain` for sensitive data |

---

## 💾 Basic Key-Value Storage

**Android (SharedPreferences)**
```kotlin
val prefs = context.getSharedPreferences("app_prefs", Context.MODE_PRIVATE)

prefs.edit {
    putString("username", "rehan")
    putBoolean("is_logged_in", true)
}

val username = prefs.getString("username", null)
```

**iOS (UserDefaults)**
```swift
UserDefaults.standard.set("rehan", forKey: "username")
UserDefaults.standard.set(true, forKey: "is_logged_in")

let username = UserDefaults.standard.string(forKey: "username")
```

Nearly a 1:1 mapping — both are synchronous, both auto-persist, both are meant for small, non-sensitive data (flags, preferences, cached settings).

---

## 🆕 Modern Approach: DataStore vs @AppStorage

**Android (Jetpack DataStore — Preferences)**
```kotlin
val dataStore: DataStore<Preferences> = context.dataStore

suspend fun saveUsername(name: String) {
    dataStore.edit { it[stringPreferencesKey("username")] = name }
}

val usernameFlow: Flow<String?> = dataStore.data
    .map { it[stringPreferencesKey("username")] }
```

**iOS (@AppStorage — SwiftUI property wrapper)**
```swift
struct SettingsView: View {
    @AppStorage("username") private var username: String = ""

    var body: some View {
        Text("Hello, \(username)")
    }
}
```

`@AppStorage` is a thin, reactive wrapper directly over `UserDefaults` — declaring it in a View auto-updates the UI when the value changes, which is more like DataStore's `Flow` reactivity than raw `SharedPreferences`.

> ⚠️ Unlike DataStore, `@AppStorage`/`UserDefaults` is still synchronous and not designed for large or complex data — it remains a simple settings store, not a database replacement.

---

## 🔐 Secure Storage (Tokens, Passwords)

**Android (EncryptedSharedPreferences)**
```kotlin
val masterKey = MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
    .build()

val encryptedPrefs = EncryptedSharedPreferences.create(
    context,
    "secure_prefs",
    masterKey,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
)

encryptedPrefs.edit { putString("auth_token", token) }
```

**iOS (Keychain)**
```swift
import Security

func saveToken(_ token: String) {
    let data = token.data(using: .utf8)!
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: "auth_token",
        kSecValueData as String: data
    ]
    SecItemAdd(query as CFDictionary, nil)
}
```

Both are the correct place for tokens, credentials, and any sensitive data. Keychain's raw API is notoriously verbose — most iOS projects wrap it in a helper class or use a small library (e.g. **KeychainAccess**) for ergonomics closer to `EncryptedSharedPreferences`.

**KeychainAccess (popular wrapper library)**
```swift
let keychain = Keychain(service: "com.example.app")
keychain["auth_token"] = token
let token = keychain["auth_token"]
```

---

## 🗑 Removing Data

**Android**
```kotlin
prefs.edit { remove("username") }
prefs.edit { clear() }
```

**iOS**
```swift
UserDefaults.standard.removeObject(forKey: "username")
// Clearing all requires removing the persistent domain:
UserDefaults.standard.removePersistentDomain(forName: Bundle.main.bundleIdentifier!)
```

---

## 📝 Quick Reference Table

| Concept | Android | iOS |
|---|---|---|
| Simple key-value store | `SharedPreferences` | `UserDefaults` |
| Modern/reactive store | Jetpack `DataStore` | `@AppStorage` |
| Secure store | `EncryptedSharedPreferences` | `Keychain` (raw or via KeychainAccess) |
| Async/Flow-based access | `DataStore.data: Flow<T>` | Not native — Combine wrapper needed |
| Remove single key | `prefs.edit { remove(key) }` | `removeObject(forKey:)` |
| Clear all | `prefs.edit { clear() }` | `removePersistentDomain(forName:)` |

---
