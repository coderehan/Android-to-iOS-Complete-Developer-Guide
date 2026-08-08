# 07 - Components: Reusable UI Components

> Composable functions and SwiftUI Views are both just functions/structs — meaning "component" in both worlds is just "a smaller piece of UI you extracted and reused."

---

## 🧩 Custom Reusable Component

**Compose**
```kotlin
@Composable
fun PrimaryButton(
    text: String,
    onClick: () -> Unit,
    modifier: Modifier = Modifier
) {
    Button(
        onClick = onClick,
        modifier = modifier,
        colors = ButtonDefaults.buttonColors(containerColor = Color.Blue)
    ) {
        Text(text, color = Color.White)
    }
}

// Usage
PrimaryButton(text = "Continue", onClick = { /* ... */ })
```

**SwiftUI**
```swift
struct PrimaryButton: View {
    let text: String
    let action: () -> Void

    var body: some View {
        Button(action: action) {
            Text(text)
                .foregroundColor(.white)
        }
        .background(Color.blue)
    }
}

// Usage
PrimaryButton(text: "Continue", action: { /* ... */ })
```

Both patterns are identical in spirit: define parameters, define a callback closure, compose smaller primitives inside.

---

## 🎛 Common Built-in Components

| Compose | SwiftUI |
|---|---|
| `Text` | `Text` |
| `Button` | `Button` |
| `TextField` | `TextField` |
| `Image` | `Image` |
| `Checkbox` | `Toggle` (styled) |
| `Switch` | `Toggle` |
| `RadioButton` | Custom, or `Picker(.segmented)` |
| `Slider` | `Slider` |
| `CircularProgressIndicator` | `ProgressView` |
| `LinearProgressIndicator` | `ProgressView(value:)` |
| `AlertDialog` | `.alert()` modifier |
| `Snackbar` | `.toast` (3rd-party) or custom overlay |
| `BottomSheet` | `.sheet()` / `.presentationDetents()` |
| `Divider` | `Divider` |
| `Card` | Custom view + `.background()` + `.cornerRadius()` |
| `Chip` | Custom view |

---

## 🔤 TextField Example

**Compose**
```kotlin
var text by remember { mutableStateOf("") }

OutlinedTextField(
    value = text,
    onValueChange = { text = it },
    label = { Text("Email") }
)
```

**SwiftUI**
```swift
@State private var text = ""

TextField("Email", text: $text)
    .textFieldStyle(.roundedBorder)
```

Note the `$text` — SwiftUI's `$` prefix creates a **Binding**, which is conceptually similar to passing `value` + `onValueChange` together as one unit in Compose.

---

## ⚠️ Dialogs / Alerts

**Compose**
```kotlin
if (showDialog) {
    AlertDialog(
        onDismissRequest = { showDialog = false },
        title = { Text("Delete item?") },
        confirmButton = {
            TextButton(onClick = { /* delete */ }) { Text("Delete") }
        },
        dismissButton = {
            TextButton(onClick = { showDialog = false }) { Text("Cancel") }
        }
    )
}
```

**SwiftUI**
```swift
.alert("Delete item?", isPresented: $showDialog) {
    Button("Delete", role: .destructive) { /* delete */ }
    Button("Cancel", role: .cancel) { }
}
```

SwiftUI attaches alerts as a **modifier** on an existing view (driven by a boolean binding), rather than conditionally composing a dialog into the tree like Compose does.

---

## 📋 Bottom Sheet

**Compose**
```kotlin
ModalBottomSheet(onDismissRequest = { showSheet = false }) {
    SheetContent()
}
```

**SwiftUI**
```swift
.sheet(isPresented: $showSheet) {
    SheetContent()
        .presentationDetents([.medium, .large])
}
```

---

## 🧱 Component Composition Pattern

Both ecosystems encourage **small, single-purpose components composed together** rather than large monolithic screens — this is a shared best practice, not just a syntax mapping:

```
ScreenComposable / ScreenView
 ├── HeaderComponent
 ├── ContentList
 │    └── ListItemComponent (repeated)
 └── FooterComponent
```

---

## 📝 Quick Reference Table

| Concept | Jetpack Compose | SwiftUI |
|---|---|---|
| Text input | `TextField` / `OutlinedTextField` | `TextField` |
| Two-way binding | `value` + `onValueChange` | `$binding` |
| Toggle | `Switch` | `Toggle` |
| Alert/dialog | `AlertDialog` (conditional) | `.alert()` modifier |
| Bottom sheet | `ModalBottomSheet` | `.sheet()` + `.presentationDetents()` |
| Progress indicator | `CircularProgressIndicator` | `ProgressView` |
| Divider line | `Divider` | `Divider` |
| Custom reusable UI | `@Composable fun` | `struct: View` |

---
