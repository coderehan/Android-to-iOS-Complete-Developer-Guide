# 28 - CI/CD: Android vs iOS

> CI/CD concepts transfer directly, but iOS pipelines have extra complexity around code signing, provisioning profiles, and Apple-specific tooling (Fastlane, Xcode Cloud) that Android doesn't need.

---

## 🔑 Core Comparison

| Android | iOS |
|---|---|
| GitHub Actions / Bitrise / GitLab CI | GitHub Actions (needs macOS runner) / Bitrise / Xcode Cloud |
| `./gradlew assembleRelease` | `xcodebuild archive` + `xcodebuild -exportArchive` |
| Signing via keystore (`.jks`) | Signing via certificates + provisioning profiles |
| Fastlane (optional, popular) | Fastlane (near-universal for iOS) |
| Google Play Console API for upload | App Store Connect API for upload |

---

## 🏗 Basic CI Pipeline (GitHub Actions)

**Android**
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with: { java-version: '17' }
      - run: ./gradlew assembleRelease
      - run: ./gradlew test
```

**iOS**
```yaml
jobs:
  build:
    runs-on: macos-14  # macOS runner required — noticeably slower/more expensive than Linux
    steps:
      - uses: actions/checkout@v4
      - run: xcodebuild -scheme MyApp -configuration Release -sdk iphoneos build
      - run: xcodebuild test -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 15'
```

> ⚠️ Key infra difference: Android builds run fine on cheap Linux runners. iOS builds **require a macOS runner** (GitHub-hosted macOS minutes are more expensive and slower to provision than Linux) — this is a real cost/speed consideration when estimating iOS CI pipelines.

---

## 🔏 Code Signing

**Android** — one keystore file (`.jks`), referenced via Gradle signing config:
```kotlin
signingConfigs {
    create("release") {
        storeFile = file("release.jks")
        storePassword = System.getenv("KEYSTORE_PASSWORD")
        keyAlias = "release"
        keyPassword = System.getenv("KEY_PASSWORD")
    }
}
```

**iOS** — requires a **Distribution Certificate** + a **Provisioning Profile** (tying the certificate, App ID, and allowed devices/capabilities together). Typically managed via **Fastlane Match**, which syncs certs/profiles across a team through an encrypted git repo:
```ruby
# Fastfile
lane :beta do
  match(type: "appstore")
  build_app(scheme: "MyApp")
  upload_to_testflight
end
```

iOS signing is meaningfully more complex than Android's single-keystore model — expect to spend real setup time here the first time, even though the ongoing workflow (via Fastlane Match) becomes simple once configured.

---

## 🚀 Fastlane (Both Platforms)

Fastlane is genuinely cross-platform and widely used on **both** Android and iOS for standardizing release automation:

```ruby
# Android lane
lane :deploy_android do
  gradle(task: "bundleRelease")
  upload_to_play_store(track: "internal")
end

# iOS lane
lane :deploy_ios do
  build_app(scheme: "MyApp")
  upload_to_testflight
end
```

If you're already comfortable with Fastlane on Android, the same tool and mental model carries over almost entirely — just different underlying `gradle`/`xcodebuild` actions.

---

## ☁️ Xcode Cloud (Apple's Native CI, iOS-only)

Apple's own CI product, tightly integrated with Xcode and App Store Connect — no YAML, configured mostly through the Xcode UI:

- Auto-detects your scheme and test plan
- Built-in TestFlight distribution
- No separate macOS runner management needed (Apple hosts it)

There's no direct Android equivalent from Google in the same "fully managed, IDE-integrated" sense — Firebase App Distribution is the closest comparable piece for beta distribution, but it isn't a full CI product like Xcode Cloud.

---

## 📦 Distribution

| Android | iOS |
|---|---|
| Internal testing track (Play Console) | TestFlight (internal/external testing) |
| APK/AAB direct install for QA | Ad-hoc distribution (device-limited) or TestFlight |
| Google Play Console API | App Store Connect API |
| Staged rollout percentages | Phased release (App Store Connect) |

---

## 📝 Quick Reference Table

| Concept | Android | iOS |
|---|---|---|
| CI runner OS | Linux (cheap, fast) | macOS (required, slower/costlier) |
| Build command | `./gradlew assembleRelease` | `xcodebuild archive` |
| Signing artifact | Keystore (`.jks`) | Certificate + Provisioning Profile |
| Cert/profile management | N/A (single file) | Fastlane Match (recommended) |
| Cross-platform automation | Fastlane | Fastlane |
| Native managed CI | — | Xcode Cloud |
| Beta distribution | Play Console internal track | TestFlight |
| Store upload API | Google Play Console API | App Store Connect API |

---
