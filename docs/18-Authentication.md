# 18 - Authentication: Android vs iOS

> Authentication concepts (token storage, biometrics, OAuth, social login) are the same on both platforms — the SDKs and native biometric APIs differ.

---

## 🔑 Core Comparison

| Android | iOS |
|---|---|
| Firebase Auth SDK (Android) | Firebase Auth SDK (iOS) — same backend, different client |
| BiometricPrompt (fingerprint/face) | LocalAuthentication (Face ID/Touch ID) |
| Google Sign-In SDK | Sign in with Apple / Google Sign-In SDK |
| EncryptedSharedPreferences for tokens | Keychain for tokens |
| Custom Tabs for OAuth web flows | ASWebAuthenticationSession for OAuth web flows |

---

## 🔥 Firebase Authentication

**Android**
```kotlin
val auth = FirebaseAuth.getInstance()

auth.signInWithEmailAndPassword(email, password)
    .addOnSuccessListener { result -> /* success */ }
    .addOnFailureListener { e -> /* error */ }

// or with coroutines
suspend fun login(email: String, password: String): AuthResult {
    return auth.signInWithEmailAndPassword(email, password).await()
}
```

**iOS**
```swift
import FirebaseAuth

func login(email: String, password: String) async throws -> AuthDataResult {
    try await Auth.auth().signIn(withEmail: email, password: password)
}
```

Firebase Auth's iOS SDK mirrors the Android SDK's API surface closely — same methods, same concepts, just Swift-native async/await instead of Kotlin coroutines/listeners.

---

## 👆 Biometric Authentication

**Android (BiometricPrompt)**
```kotlin
val biometricPrompt = BiometricPrompt(
    this,
    ContextCompat.getMainExecutor(this),
    object : BiometricPrompt.AuthenticationCallback() {
        override fun onAuthenticationSucceeded(result: BiometricPrompt.AuthenticationResult) {
            // unlocked
        }
    }
)

val promptInfo = BiometricPrompt.PromptInfo.Builder()
    .setTitle("Unlock App")
    .setNegativeButtonText("Cancel")
    .build()

biometricPrompt.authenticate(promptInfo)
```

**iOS (LocalAuthentication)**
```swift
import LocalAuthentication

func authenticateWithBiometrics() async throws -> Bool {
    let context = LAContext()
    return try await context.evaluatePolicy(
        .deviceOwnerAuthenticationWithBiometrics,
        localizedReason: "Unlock App"
    )
}
```

Both platforms abstract away *which* biometric is used (fingerprint vs face on Android; Face ID vs Touch ID on iOS) — you just request "biometric auth" and the OS handles the specific modality.

> ⚠️ iOS requires an `NSFaceIDUsageDescription` key in `Info.plist` (like Android's runtime permission rationale), or the app crashes when calling Face ID APIs.

---

## 🌐 OAuth / Social Login Web Flow

**Android (Custom Tabs / AppAuth)**
```kotlin
val authRequest = AuthorizationRequest.Builder(
    serviceConfig, clientId, ResponseTypeValues.CODE, redirectUri
).build()

val authIntent = authService.getAuthorizationRequestIntent(authRequest)
startActivityForResult(authIntent, RC_AUTH)
```

**iOS (ASWebAuthenticationSession)**
```swift
import AuthenticationServices

func startOAuthFlow() async throws -> URL {
    let session = ASWebAuthenticationSession(
        url: authURL,
        callbackURLScheme: "myapp"
    ) { callbackURL, error in
        // handle callback
    }
    session.presentationContextProvider = self
    session.start()
}
```

Both present a secure, sandboxed browser view for the OAuth flow (rather than a plain WebView, which is discouraged/blocked by many providers for security) and redirect back via a custom URL scheme or universal link.

---

##  Sign in with Apple (iOS-Specific Requirement)

If your iOS app offers **any** third-party login (Google, Facebook, etc.), Apple requires you to also offer **Sign in with Apple** as an option — this has no Android equivalent requirement.

```swift
import AuthenticationServices

let request = ASAuthorizationAppleIDProvider().createRequest()
request.requestedScopes = [.fullName, .email]

let controller = ASAuthorizationController(authorizationRequests: [request])
controller.delegate = self
controller.performRequests()
```

---

## 🔐 Storing Auth Tokens Securely

| Android | iOS |
|---|---|
| `EncryptedSharedPreferences` | `Keychain` |
| Token refresh via OkHttp `Authenticator` | Token refresh via Alamofire `RequestInterceptor` or custom `URLSession` retry logic |

(See [14 - Local Storage](./14-Local-Storage.md) for detailed secure-storage code examples.)

---

## 📝 Quick Reference Table

| Concept | Android | iOS |
|---|---|---|
| Firebase Auth | `FirebaseAuth.getInstance()` | `Auth.auth()` |
| Biometric prompt | `BiometricPrompt` | `LAContext` / LocalAuthentication |
| OAuth browser flow | Custom Tabs + AppAuth | `ASWebAuthenticationSession` |
| Apple-mandated login | — | Sign in with Apple (required if other social logins exist) |
| Secure token storage | `EncryptedSharedPreferences` | `Keychain` |
| Permission description | Runtime permission dialog | `NSFaceIDUsageDescription` in `Info.plist` |

---

