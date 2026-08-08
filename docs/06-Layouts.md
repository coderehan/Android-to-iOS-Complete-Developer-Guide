# 06 - Layouts: Compose Layout System vs SwiftUI Layout System

> Both frameworks use a constraint-and-measurement pass system under the hood, but expose it through simple stacking primitives + alignment/weight modifiers.

---

## 🔑 Core Layout Primitives

| Compose | SwiftUI |
|---|---|
| `Column` | `VStack` |
| `Row` | `HStack` |
| `Box` | `ZStack` |
| `ConstraintLayout` | `GeometryReader` + alignment guides (or plain stacks for most cases) |
| `Modifier.weight()` | `.frame(maxWidth: .infinity)` / `Spacer()` |
| `Arrangement` | `spacing:` parameter + `Spacer()` |
| `Alignment` | `alignment:` parameter |

---

## 📏 Sizing

**Compose**
```kotlin
Box(
    modifier = Modifier
        .width(200.dp)
        .height(100.dp)
)

Box(modifier = Modifier.fillMaxSize())
Box(modifier = Modifier.wrapContentSize())
```

**SwiftUI**
```swift
Rectangle()
    .frame(width: 200, height: 100)

Rectangle()
    .frame(maxWidth: .infinity, maxHeight: .infinity)
// wrapContent is the SwiftUI default — views size to their content unless told otherwise
```

> Key mental shift: in Compose, "wrap content" is often explicit or default depending on the component. In SwiftUI, **hugging content size is the default behavior** for every view — you opt into filling space with `.frame(maxWidth: .infinity)`, not the other way around.

---

## ⚖️ Weight / Flexible Sizing

**Compose**
```kotlin
Row {
    Box(modifier = Modifier.weight(1f).background(Color.Red))
    Box(modifier = Modifier.weight(2f).background(Color.Blue))
}
```

**SwiftUI**
```swift
HStack(spacing: 0) {
    Color.red
        .frame(maxWidth: .infinity)
    Color.blue
        .frame(maxWidth: .infinity)
        .frame(width: /* proportionally larger, e.g. via GeometryReader */ nil)
}
```

SwiftUI has no built-in direct `weight()` equivalent for simple proportional splits — the common workaround is `GeometryReader` to read available space and compute proportions manually, or simply using equal `.frame(maxWidth: .infinity)` for equal splits.

```swift
GeometryReader { geo in
    HStack(spacing: 0) {
        Color.red.frame(width: geo.size.width / 3)
        Color.blue.frame(width: geo.size.width * 2 / 3)
    }
}
```

---

## ↔️ Alignment

**Compose**
```kotlin
Column(
    horizontalAlignment = Alignment.CenterHorizontally
) {
    Text("Centered")
}

Box(contentAlignment = Alignment.Center) {
    Text("Centered in box")
}
```

**SwiftUI**
```swift
VStack(alignment: .center) {
    Text("Centered")
}

ZStack(alignment: .center) {
    Text("Centered in box")
}
```

---

## 🔲 Padding & Spacing

**Compose**
```kotlin
Column(
    verticalArrangement = Arrangement.spacedBy(8.dp)
) {
    Text("Item 1")
    Text("Item 2")
}
```

**SwiftUI**
```swift
VStack(spacing: 8) {
    Text("Item 1")
    Text("Item 2")
}
```

Both support built-in spacing between children — no manual `Spacer` needed for uniform gaps (though `Spacer()` is still used for pushing content apart, e.g. flexible spacing).

---

## 🧭 GeometryReader vs BoxWithConstraints

**Compose**
```kotlin
BoxWithConstraints {
    if (maxWidth < 600.dp) {
        CompactLayout()
    } else {
        WideLayout()
    }
}
```

**SwiftUI**
```swift
GeometryReader { geometry in
    if geometry.size.width < 600 {
        CompactLayout()
    } else {
        WideLayout()
    }
}
```

Both give you access to the available constraint size for responsive/adaptive layouts (e.g. phone vs tablet, portrait vs landscape).

---

## 📝 Quick Reference Table

| Concept | Jetpack Compose | SwiftUI |
|---|---|---|
| Vertical layout | `Column` | `VStack` |
| Horizontal layout | `Row` | `HStack` |
| Overlapping layout | `Box` | `ZStack` |
| Fill available space | `Modifier.fillMaxSize()` | `.frame(maxWidth: .infinity, maxHeight: .infinity)` |
| Wrap content | Often default / `wrapContentSize()` | Default behavior |
| Proportional split | `Modifier.weight()` | `GeometryReader` (manual) |
| Spacing between children | `Arrangement.spacedBy()` | `spacing:` param on stack |
| Alignment | `Alignment.Center`, etc. | `.center`, `.leading`, `.trailing` |
| Responsive constraints | `BoxWithConstraints` | `GeometryReader` |
| Push content apart | `Spacer()` (with weight) | `Spacer()` |

---
