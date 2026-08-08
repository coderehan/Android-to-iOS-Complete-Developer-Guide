# 10 - Lifecycle: Android Lifecycle vs iOS Lifecycle

> Android's lifecycle is explicit and callback-heavy (Activity/Fragment states). iOS's SwiftUI lifecycle is more implicit, tied to view appearance/disappearance and Scene phases rather than a rigid Activity state machine.

---

## 🔑 High-Level Comparison

| Android | iOS |
|---|---|
| `Activity` lifecycle (`onCreate`, `onStart`, `onResume`, `onPause`, `onStop`, `onDestroy`) | `Scene` phases (`active`, `inactive`, `background`) |
| `Fragment` lifecycle | `View` lifecycle (`onAppear`, `onDisappear`) |
| `ProcessLifecycleOwner` (app-wide) | `ScenePhase` via `@Environment(\.scenePhase)` |
| Composable enters/leaves composition | `onAppear` / `onDisappear` |

---

## 🏁 App-Level Lifecycle

**Android**
```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) { /* setup */ }
    override fun onStart() { /* becoming visible */ }
    override fun onResume() { /* interactive */ }
    override fun onPause() { /* losing focus */ }
    override fun onStop() { /* not visible */ }
    override fun onDestroy() { /* cleanup */ }
}
```

**iOS (SwiftUI)**
```swift
@main
struct MyAppApp: App {
    @Environment(\.scenePhase) private var scenePhase

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .onChange(of: scenePhase) { oldPhase, newPhase in
            switch newPhase {
            case .active:
                break // like onResume
            case .inactive:
                break // like onPause
            case .background:
                break // like onStop
            @unknown default:
                break
            }
        }
    }
}
```

iOS collapses Android's 6-state Activity lifecycle into **3 Scene phases**: `active`, `inactive`, `background`. There's no separate `onCreate`/`onDestroy` equivalent exposed at this level — app launch/termination setup usually happens in the `App` struct's `init()` or via `UIApplicationDelegate` adaptor for lower-level hooks.

---

## 👁 View-Level Lifecycle

**Compose**
```kotlin
@Composable
fun MyScreen() {
    DisposableEffect(Unit) {
        // like onStart / Fragment onViewCreated
        onDispose {
            // like onStop / onDestroyView — cleanup
        }
    }
}
```

**SwiftUI**
```swift
struct MyScreen: View {
    var body: some View {
        Text("Hello")
            .onAppear {
                // view became visible
            }
            .onDisappear {
                // view became invisible
            }
    }
}
```

`onAppear`/`onDisappear` are the most commonly used lifecycle hooks in SwiftUI — closer in spirit to Compose's `DisposableEffect` or `LaunchedEffect(Unit) { }` than to a full Activity lifecycle.

---

## ⚡ Side Effects on Composition/Appear

**Compose**
```kotlin
LaunchedEffect(Unit) {
    viewModel.loadData()
}
```

**SwiftUI**
```swift
.task {
    await viewModel.loadData()
}
```

`.task { }` is SwiftUI's async-aware equivalent of `LaunchedEffect` — it automatically cancels when the view disappears, just like `LaunchedEffect`'s coroutine is cancelled when leaving composition.

---

## 🔄 Configuration Changes / State Restoration

| Android | iOS |
|---|---|
| Rotation triggers Activity recreate (unless handled) | No Activity-recreate equivalent — SwiftUI views naturally re-lay-out on size class change |
| `rememberSaveable` survives config change | `@SceneStorage` / `@AppStorage` for persisted UI state |
| `ViewModel` survives config change (scoped to Activity) | `@StateObject`/`@Observable` object survives view re-render, but not app termination unless persisted |
| `onSaveInstanceState` | `NSUserActivity` / State Restoration APIs |

> ⚠️ Key difference: iOS has no "configuration change" concept like Android's rotation-triggered Activity recreation. SwiftUI views simply react to new size/trait information (`horizontalSizeClass`, `verticalSizeClass`) without destroying and recreating the whole screen — much lighter weight than Android's historical rotation handling.

---

## 🧬 Process Death & Background Handling

**Android**
```kotlin
override fun onSaveInstanceState(outState: Bundle) {
    outState.putString("key", value)
}
```

**iOS**
```swift
@SceneStorage("key") private var savedValue: String = ""
```

Both provide lightweight state-restoration hooks for surviving backgrounding/termination — Android's is more manual (Bundle key-value), iOS's `@SceneStorage`/`@AppStorage` behaves more like automatic property persistence.

---

## 📝 Quick Reference Table

| Concept | Android | iOS |
|---|---|---|
| App active | `onResume` | `.active` scene phase |
| App losing focus | `onPause` | `.inactive` scene phase |
| App backgrounded | `onStop` | `.background` scene phase |
| View appears | Fragment `onViewCreated`/`onStart` | `.onAppear` |
| View disappears | Fragment `onDestroyView`/`onStop` | `.onDisappear` |
| Run async task tied to view | `LaunchedEffect(Unit)` | `.task { }` |
| Save simple state | `onSaveInstanceState` | `@SceneStorage` |
| Survive config change | `ViewModel` + `rememberSaveable` | Not applicable — no recreate cycle |

---
