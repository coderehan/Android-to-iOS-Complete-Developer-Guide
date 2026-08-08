# 09 - State Management: Compose State vs SwiftUI State

> This is the heart of both frameworks. Both are built on the same core idea: **UI = f(state)**. When state changes, the UI automatically re-renders. The property wrappers differ, but the mental model is the same.

---

## 🔑 Core Philosophy

| Compose | SwiftUI |
|---|---|
| `remember { mutableStateOf() }` | `@State` |
| `ViewModel` + `StateFlow`/`LiveData` | `ObservableObject` + `@Published` (or `@Observable`) |
| `collectAsState()` | Automatic via `@Published` / `@Observable` |
| Unidirectional data flow | Unidirectional data flow |
| Recomposition | Re-render |

---

## 📍 Local UI State

**Compose**
```kotlin
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }

    Button(onClick = { count++ }) {
        Text("Count: $count")
    }
}
```

**SwiftUI**
```swift
struct Counter: View {
    @State private var count = 0

    var body: some View {
        Button("Count: \(count)") {
            count += 1
        }
    }
}
```

`remember { mutableStateOf() }` ↔ `@State` — both are local, view-owned, memory-backed state that survives recomposition/re-render but not process death, and trigger a UI refresh when mutated.

---

## 🧠 ViewModel-Driven State

**Compose**
```kotlin
class CounterViewModel : ViewModel() {
    private val _count = MutableStateFlow(0)
    val count: StateFlow<Int> = _count.asStateFlow()

    fun increment() {
        _count.value++
    }
}

@Composable
fun Counter(viewModel: CounterViewModel = viewModel()) {
    val count by viewModel.count.collectAsState()

    Button(onClick = { viewModel.increment() }) {
        Text("Count: $count")
    }
}
```

**SwiftUI (using @Observable — modern approach)**
```swift
@Observable
class CounterViewModel {
    var count = 0

    func increment() {
        count += 1
    }
}

struct Counter: View {
    @State private var viewModel = CounterViewModel()

    var body: some View {
        Button("Count: \(viewModel.count)") {
            viewModel.increment()
        }
    }
}
```

**SwiftUI (older ObservableObject approach)**
```swift
class CounterViewModel: ObservableObject {
    @Published var count = 0

    func increment() {
        count += 1
    }
}

struct Counter: View {
    @StateObject private var viewModel = CounterViewModel()

    var body: some View {
        Button("Count: \(viewModel.count)") {
            viewModel.increment()
        }
    }
}
```

> `StateFlow` + `collectAsState()` ↔ `@Published` (auto-observed, no manual "collect" step needed). The newer `@Observable` macro (iOS 17+) removes the need for `@Published` on every property, similar to how Compose just needs `mutableStateOf` without extra boilerplate.

---

## 📤 Passing State Down / Events Up

**Compose**
```kotlin
@Composable
fun ParentScreen(viewModel: MyViewModel = viewModel()) {
    val state by viewModel.uiState.collectAsState()
    ChildComponent(text = state.text, onTextChanged = viewModel::onTextChanged)
}
```

**SwiftUI**
```swift
struct ParentScreen: View {
    @State private var viewModel = MyViewModel()

    var body: some View {
        ChildComponent(text: viewModel.text, onTextChanged: viewModel.onTextChanged)
    }
}
```

Both follow the same **unidirectional data flow**: state flows down, events flow up via callbacks.

---

## 🔗 Two-Way Binding

**Compose** has no built-in two-way binding primitive — you wire `value` + `onValueChange` manually.

**SwiftUI** has `@Binding` and `$` syntax for true two-way binding:
```swift
struct ChildView: View {
    @Binding var text: String

    var body: some View {
        TextField("Enter text", text: $text)
    }
}

// Parent
@State private var text = ""
ChildView(text: $text)
```

This is a genuine difference — SwiftUI's `Binding` type is a first-class citizen with no direct Compose equivalent; Compose always uses the explicit `value`/`onValueChange` pair pattern instead.

---

## 🆚 State Ownership: `@State` vs `@StateObject` vs `@ObservedObject`

| Wrapper | Meaning |
|---|---|
| `@State` | View owns simple value-type state |
| `@StateObject` | View owns and creates a reference-type observable object (created once) |
| `@ObservedObject` | View observes an object owned/passed by a parent |
| `@EnvironmentObject` | Object injected into the view hierarchy, available anywhere below |
| `@Observable` (iOS 17+) | Modern macro replacing most `@Published`/`ObservableObject` boilerplate |

Roughly: `@StateObject` ≈ "I created this ViewModel, this is my `viewModel()`/`hiltViewModel()` call." `@ObservedObject`/`@EnvironmentObject` ≈ "This was passed down to me, like a ViewModel shared across a nav graph or scoped to an Activity."

---

## 📝 Quick Reference Table

| Concept | Jetpack Compose | SwiftUI |
|---|---|---|
| Local state | `remember { mutableStateOf() }` | `@State` |
| Observable stream | `StateFlow` / `LiveData` | `@Published` / `@Observable` |
| Collecting state | `collectAsState()` | Automatic |
| Owned reference-type state | `viewModel()` | `@StateObject` |
| Passed reference-type state | Passed ViewModel param | `@ObservedObject` |
| App-wide shared state | `CompositionLocal` | `@EnvironmentObject` |
| Two-way binding | Manual (`value` + `onValueChange`) | `@Binding` / `$` |
| Derived/computed state | `derivedStateOf { }` | Computed property |

---
