# 08 - Navigation: Navigation Component vs SwiftUI Navigation

> Both platforms moved from manual/imperative navigation (FragmentTransactions, push/pop UIViewControllers) to declarative, state-driven navigation graphs.

---

## 🔑 Core Philosophy

| Android Navigation Component | SwiftUI Navigation |
|---|---|
| `NavHost` + `NavController` | `NavigationStack` + path binding |
| Destinations defined as composables | Destinations defined as views |
| `navController.navigate("route")` | `path.append(value)` or `NavigationLink` |
| Back stack managed by `NavController` | Back stack managed by `NavigationStack`'s path |
| Deep links via `navDeepLink` | Deep links via `.onOpenURL` / `NavigationPath` |

---

## 🗺 Basic Setup

**Compose**
```kotlin
val navController = rememberNavController()

NavHost(navController = navController, startDestination = "home") {
    composable("home") { HomeScreen(navController) }
    composable("details/{id}") { backStackEntry ->
        val id = backStackEntry.arguments?.getString("id")
        DetailsScreen(id)
    }
}
```

**SwiftUI**
```swift
NavigationStack {
    HomeScreen()
        .navigationDestination(for: String.self) { id in
            DetailsScreen(id: id)
        }
}
```

---

## ➡️ Navigating Forward

**Compose**
```kotlin
Button(onClick = { navController.navigate("details/123") }) {
    Text("Go to Details")
}
```

**SwiftUI**
```swift
NavigationLink("Go to Details", value: "123")

// or programmatically:
Button("Go to Details") {
    path.append("123")
}
```

---

## ⬅️ Navigating Back

**Compose**
```kotlin
navController.popBackStack()
navController.navigateUp()
```

**SwiftUI**
```swift
dismiss() // via @Environment(\.dismiss) private var dismiss

// or, if using a path binding:
path.removeLast()
```

---

## 📦 Passing Arguments

**Compose**
```kotlin
composable(
    route = "profile/{userId}",
    arguments = listOf(navArgument("userId") { type = NavType.StringType })
) { backStackEntry ->
    val userId = backStackEntry.arguments?.getString("userId")
    ProfileScreen(userId)
}
```

**SwiftUI**
```swift
struct UserRoute: Hashable {
    let userId: String
}

.navigationDestination(for: UserRoute.self) { route in
    ProfileScreen(userId: route.userId)
}
```

SwiftUI encourages passing typed, `Hashable` route values (structs/enums) rather than string-templated routes — a cleaner, more type-safe version of what Android's Navigation Component's Safe Args plugin tries to achieve on top of string routes.

---

## 📚 Bottom Navigation / Tabs

**Compose**
```kotlin
NavigationBar {
    NavigationBarItem(
        selected = selectedTab == 0,
        onClick = { selectedTab = 0 },
        icon = { Icon(Icons.Default.Home, null) },
        label = { Text("Home") }
    )
}
```

**SwiftUI**
```swift
TabView(selection: $selectedTab) {
    HomeScreen()
        .tabItem { Label("Home", systemImage: "house") }
        .tag(0)
}
```

`TabView` is SwiftUI's direct equivalent of `NavigationBar`/`BottomNavigationView` — each tab holds its own content, optionally its own `NavigationStack` for independent back stacks per tab (like Android's nested nav graphs per bottom-nav destination).

---

## 🔗 Deep Linking

**Compose**
```kotlin
composable(
    "details/{id}",
    deepLinks = listOf(navDeepLink { uriPattern = "myapp://details/{id}" })
) { /* ... */ }
```

**SwiftUI**
```swift
.onOpenURL { url in
    // Parse url, push onto NavigationPath programmatically
    if let id = extractId(from: url) {
        path.append(DetailsRoute(id: id))
    }
}
```

---

## 📝 Quick Reference Table

| Concept | Android Navigation Component | SwiftUI |
|---|---|---|
| Navigation container | `NavHost` | `NavigationStack` |
| Controller | `NavController` | Path binding (`NavigationPath` / `[Route]`) |
| Navigate forward | `navController.navigate(route)` | `NavigationLink` / `path.append()` |
| Navigate back | `popBackStack()` | `dismiss()` / `path.removeLast()` |
| Pass arguments | Route string + `navArgument` | Typed `Hashable` route struct |
| Bottom tabs | `NavigationBar` | `TabView` |
| Deep linking | `navDeepLink` | `.onOpenURL` |
| Modal/full-screen | `dialog()` destination | `.fullScreenCover()` |

---
