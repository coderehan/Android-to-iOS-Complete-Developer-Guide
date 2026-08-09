# 24 - Animations: Compose Animations vs SwiftUI Animations

> Both frameworks make animation nearly free — wrap a state change in an animation call/modifier and the framework interpolates automatically. The APIs are strikingly similar in spirit.

---

## 🔑 Core Philosophy

| Compose | SwiftUI |
|---|---|
| `animate*AsState` for simple value animations | `.animation(_:value:)` modifier or `withAnimation { }` |
| `AnimatedVisibility` for enter/exit | `.transition()` + conditional view |
| `updateTransition` for coordinated multi-property animation | `withAnimation` wrapping multiple state changes |
| `rememberInfiniteTransition` | `.repeatForever()` |

---

## 🎯 Simple Value Animation

**Compose**
```kotlin
var expanded by remember { mutableStateOf(false) }
val size by animateDpAsState(if (expanded) 200.dp else 100.dp, label = "size")

Box(modifier = Modifier.size(size).clickable { expanded = !expanded })
```

**SwiftUI**
```swift
@State private var expanded = false

Rectangle()
    .frame(width: expanded ? 200 : 100, height: expanded ? 200 : 100)
    .animation(.default, value: expanded)
    .onTapGesture { expanded.toggle() }
```

`animate*AsState` (auto-animates any value change) ↔ `.animation(_:value:)` modifier (auto-animates whenever the given value changes). Both remove the need for manual interpolation code.

---

## 🎬 Explicit Animation Block

**Compose**
```kotlin
scope.launch {
    animate(initialValue = 0f, targetValue = 1f) { value, _ ->
        alpha = value
    }
}
```

**SwiftUI**
```swift
withAnimation(.easeInOut(duration: 0.3)) {
    isVisible.toggle()
}
```

`withAnimation { }` is SwiftUI's most common animation entry point — wrap any state mutation, and every dependent view that reads that state animates its changes automatically. This is arguably simpler than Compose's approach for coordinating multiple simultaneous property changes.

---

## 👋 Enter/Exit Animations

**Compose**
```kotlin
AnimatedVisibility(
    visible = isVisible,
    enter = fadeIn() + slideInVertically(),
    exit = fadeOut() + slideOutVertically()
) {
    Text("Hello")
}
```

**SwiftUI**
```swift
if isVisible {
    Text("Hello")
        .transition(.opacity.combined(with: .move(edge: .top)))
}
// wrap the toggle:
withAnimation { isVisible.toggle() }
```

Compose bundles enter/exit specs directly into `AnimatedVisibility`. SwiftUI attaches a `.transition()` to the view itself and relies on the surrounding `if` + `withAnimation` to trigger it — conceptually the same outcome, slightly different plumbing.

---

## 🔁 Infinite/Repeating Animation

**Compose**
```kotlin
val infiniteTransition = rememberInfiniteTransition(label = "pulse")
val scale by infiniteTransition.animateFloat(
    initialValue = 1f,
    targetValue = 1.2f,
    animationSpec = infiniteRepeatable(
        animation = tween(600),
        repeatMode = RepeatMode.Reverse
    ),
    label = "scale"
)
```

**SwiftUI**
```swift
@State private var scale: CGFloat = 1.0

Circle()
    .scaleEffect(scale)
    .onAppear {
        withAnimation(.easeInOut(duration: 0.6).repeatForever(autoreverses: true)) {
            scale = 1.2
        }
    }
```

---

## 🎛 Animation Curves / Specs

| Compose (`AnimationSpec`) | SwiftUI (`Animation`) |
|---|---|
| `tween(durationMillis, easing)` | `.easeInOut(duration:)`, `.linear(duration:)` |
| `spring(dampingRatio, stiffness)` | `.spring(response:dampingFraction:)` |
| `infiniteRepeatable` | `.repeatForever(autoreverses:)` |
| `keyframes { }` | `.keyframeAnimator` (iOS 17+) |

---

## 🧩 Matched/Shared Element Transitions

**Compose (SharedTransitionLayout, newer API)**
```kotlin
SharedTransitionLayout {
    // shared elements between screens
}
```

**SwiftUI (matchedGeometryEffect)**
```swift
@Namespace private var animation

Rectangle()
    .matchedGeometryEffect(id: "hero", in: animation)
```

Both provide a "hero animation" style mechanism for smoothly morphing a shared element (e.g. a thumbnail expanding into a detail view) between two states/screens.

---

## 📝 Quick Reference Table

| Concept | Jetpack Compose | SwiftUI |
|---|---|---|
| Animate a single value | `animate*AsState` | `.animation(_:value:)` |
| Animate arbitrary state change | `animate { }` in coroutine | `withAnimation { }` |
| Enter/exit | `AnimatedVisibility` | `.transition()` + conditional |
| Infinite/repeating | `rememberInfiniteTransition` | `.repeatForever(autoreverses:)` |
| Spring physics | `spring(dampingRatio, stiffness)` | `.spring(response:dampingFraction:)` |
| Shared element | `SharedTransitionLayout` | `matchedGeometryEffect` |
| Gesture-driven animation | `Modifier.pointerInput` + `Animatable` | `.gesture()` + `withAnimation` |

---
