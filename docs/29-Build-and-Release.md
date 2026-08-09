# 29 - Build & Release: Play Store vs App Store

> Google Play's review is largely automated and fast. Apple's App Store review is manual (human reviewers), stricter, and slower — this affects release planning and timelines significantly.

---

## 🔑 Core Comparison

| Android (Google Play) | iOS (App Store) |
|---|---|
| Output format: `.aab` (Android App Bundle) | Output format: `.ipa` |
| Review: mostly automated, hours | Review: manual, typically 24–48 hrs (can be longer) |
| Google Play Console | App Store Connect |
| Staged rollout (%) | Phased release (7-day ramp) |
| Internal/Closed/Open testing tracks | TestFlight (internal/external) |

---

## 📦 Build Artifact

**Android**
```bash
./gradlew bundleRelease
# produces app/build/outputs/bundle/release/app-release.aab
```

**iOS**
```bash
xcodebuild archive -scheme MyApp -archivePath build/MyApp.xcarchive
xcodebuild -exportArchive -archivePath build/MyApp.xcarchive -exportPath build/ -exportOptionsPlist ExportOptions.plist
# produces build/MyApp.ipa
```

Android's `.aab` lets Google Play generate optimized APKs per device config automatically. iOS's `.ipa` is the final signed package uploaded directly — no server-side variant generation step.

---

## 📝 Store Listing Requirements

| Requirement | Google Play | App Store |
|---|---|---|
| Screenshots | Per device type (phone/tablet) | Per device size (6.9", 6.5", iPad, etc. — Apple requires specific sizes) |
| App description | Short + full description | Description + "What's New" per version |
| Privacy | Data safety form | App Privacy "Nutrition Labels" (very detailed data-use disclosure) |
| Age rating | Content rating questionnaire | Age rating questionnaire |
| Review notes | Optional | Often required (test account credentials, feature access notes) |

> ⚠️ Apple's **App Privacy details** (aka "nutrition labels") require disclosing exactly what data types are collected and whether they're linked to the user's identity, per third-party SDK — this is more granular than Play's Data Safety form and commonly trips up first-time submitters.

---

## 🕐 Review Process

**Android** — Google Play review is mostly automated (policy/malware scanning), typically completes within a few hours for most updates; new apps can take a bit longer.

**iOS** — App Store review is **human-reviewed**, typically 24-48 hours but can take longer, especially for the first submission. Common rejection reasons include:
- Missing/incomplete privacy disclosures
- Broken functionality found by the reviewer
- Guideline violations (e.g. missing Sign in with Apple when other social logins exist — see [18 - Authentication](./18-Authentication.md))
- Placeholder/incomplete content

Planning tip carried over from Android habits: **submit earlier than you think you need to** — the review can bounce back with feedback requiring a resubmission cycle, unlike Play's faster turnaround.

---

## 🎚 Rollout Strategy

**Android (staged rollout)**
```
Play Console → Release → set rollout percentage (e.g. 10% → 50% → 100%)
```

**iOS (phased release)**
```
App Store Connect → automatically ramps over 7 days:
Day 1: 1% → Day 7: 100% (Apple-controlled schedule, not fully custom percentages)
```

Android gives you full manual control over rollout percentage and pace. iOS's phased release follows Apple's fixed 7-day curve — you can pause it, but you can't set custom percentages like Android's staged rollout.

---

## 🧪 Beta Testing

| Android | iOS |
|---|---|
| Internal testing track (up to 100 testers, instant) | TestFlight internal (up to 100 testers, no review needed) |
| Closed testing (invite-only groups) | TestFlight external (up to 10,000 testers, **requires Beta App Review**, similar to full review but usually faster) |
| Open testing (public opt-in link) | Public TestFlight link (external group with public link enabled) |

---

## 📝 Quick Reference Table

| Concept | Google Play | App Store |
|---|---|---|
| Build artifact | `.aab` | `.ipa` |
| Console | Google Play Console | App Store Connect |
| Review type | Mostly automated | Manual (human reviewer) |
| Review time | Hours | 24–48+ hrs |
| Rollout control | Custom staged % | Apple's fixed 7-day phased curve |
| Beta testing | Internal/Closed/Open tracks | TestFlight (internal/external) |
| Privacy disclosure | Data Safety form | App Privacy "Nutrition Labels" |

---
