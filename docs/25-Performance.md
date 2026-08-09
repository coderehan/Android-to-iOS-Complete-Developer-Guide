# 25 - Performance: Profiling & Optimization

> Both platforms provide deep native profiling tools, and both frameworks share the same core performance principle: **minimize unnecessary re-renders/recompositions**.

---

## 🔑 Core Philosophy

| Android | iOS |
|---|---|
| Android Profiler (CPU, Memory, Network, Energy) | Instruments (Time Profiler, Allocations, Leaks, Energy Log) |
| Recomposition = re-running a `@Composable` | Re-render = re-evaluating a `View`'s `body` |
| Layout Inspector (recomposition counts) | SwiftUI `_printChanges()` / Instruments' SwiftUI template |
| Baseline Profiles for startup | App startup optimization via Instruments' "App Launch" template |

---

## 🐢 Avoiding Unnecessary Recomposition / Re-render

**Compose**
```kotlin
// Stable, immutable data classes avoid unnecessary recomposition
data class UiState(val items: List<Item>) // marked @Stable/@Immutable implicitly if all fields are stable

// Avoid reading state too high up the tree — read it as close to usage as possible
@Composable
fun ScreenContent(viewModel: MyViewModel) {
    val count by viewModel.count.collectAsState() // scoped read
    Counter(count) // only this recomposes when count changes
}
```

**SwiftUI**
```swift
// Break large views into small subviews — each subview only re-renders
// when the specific state it reads changes
struct ScreenContent: View {
    var body: some View {
        VStack {
            HeaderView()      // doesn't re-render when count changes
            CounterView()     // only this re-renders when count changes
        }
    }
}

// Debugging: print why a view re-rendered
struct CounterView: View {
    let _ = Self._printChanges()
    var body: some View { /* ... */ }
}
```

Same underlying rule in both: **granular state reads + small, focused view/composable functions minimize wasted rendering work.**

---

## 🧵 Profiling Tools

| Task | Android | iOS |
|---|---|---|
| CPU profiling | Android Profiler → CPU tab | Instruments → Time Profiler |
| Memory leaks | Android Profiler → Memory tab, LeakCanary | Instruments → Leaks / Allocations |
| Network inspection | Android Profiler → Network tab, Flipper | Instruments → Network / Charles Proxy |
| Frame rendering / jank | Layout Inspector, `Choreographer` | Instruments → Core Animation (FPS, frame drops) |
| Startup time | Baseline Profiles + Macrobenchmark | Instruments → App Launch template |
| Energy/battery | Android Profiler → Energy tab | Instruments → Energy Log |

---

## 📈 Lazy List Performance

**Compose**
```kotlin
LazyColumn {
    items(items = list, key = { it.id }) { item -> // stable keys avoid re-binding
        ItemRow(item)
    }
}
```

**SwiftUI**
```swift
List(items) { item in // Identifiable conformance provides stable identity
    ItemRow(item: item)
}
```

Both `LazyColumn`'s `key` parameter and `List`'s reliance on `Identifiable`/`id:` serve the same purpose: giving the diffing algorithm a stable identity per row so the framework can efficiently reuse/animate rows instead of re-creating everything on every data change.

---

## 🧊 Image & Memory Optimization

| Android | iOS |
|---|---|
| Downsample bitmaps before display (`BitmapFactory.Options`) | Downsample images via `UIGraphicsImageRenderer` or Kingfisher's `.downsampling()` |
| `Modifier.drawWithCache` for expensive draw ops | `.drawingGroup()` to flatten complex view hierarchies into a single layer |
| Avoid allocations inside recomposition | Avoid heavy work inside `body` — compute once, store in `@State`/`@Observable` |

---

## 🚀 App Startup Optimization

**Android**
```kotlin
// Baseline Profiles pre-compile hot paths ahead-of-time
// Avoid heavy work in Application.onCreate()
```

**iOS**
```swift
// Avoid heavy synchronous work in App.init() or AppDelegate's didFinishLaunching
// Defer non-critical setup using Task { } after the first frame renders
```

Both platforms penalize apps that do expensive synchronous setup before the first frame — deferred/lazy initialization is the shared best practice.

---

## 📝 Quick Reference Table

| Concept | Android | iOS |
|---|---|---|
| Main profiling tool | Android Profiler | Instruments |
| Recomposition/re-render debug | Layout Inspector | `Self._printChanges()` |
| Memory leak detection | LeakCanary | Instruments → Leaks |
| Stable list identity | `key = { it.id }` | `Identifiable` conformance |
| Startup optimization | Baseline Profiles | Defer work post-launch |
| Flatten expensive view tree | `Modifier.drawWithCache` | `.drawingGroup()` |
| Frame rate / jank debug | `Choreographer`, Layout Inspector | Instruments → Core Animation |

---
