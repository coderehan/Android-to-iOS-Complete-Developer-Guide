# 21 - Maps: Google Maps SDK vs MapKit

> Google Maps dominates Android; Apple's own MapKit is the native default on iOS (Google Maps SDK for iOS also exists, but MapKit is far more common for native SwiftUI apps).

---

## 🔑 Core Philosophy

| Google Maps (Android) | MapKit (iOS) |
|---|---|
| `GoogleMap` composable (via Maps Compose library) | `Map` view (native SwiftUI, iOS 17+) |
| Google-hosted map tiles/data | Apple-hosted map tiles/data |
| `Marker`, `Polyline`, `Polygon` | `Marker`, `MapPolyline`, `MapPolygon` (SwiftUI Map, iOS 17+) |
| `CameraPositionState` | `MapCameraPosition` |

---

## 🗺 Basic Map with a Marker

**Android (Maps Compose)**
```kotlin
val cameraPositionState = rememberCameraPositionState {
    position = CameraPosition.fromLatLngZoom(LatLng(12.9716, 77.5946), 12f)
}

GoogleMap(cameraPositionState = cameraPositionState) {
    Marker(
        state = MarkerState(position = LatLng(12.9716, 77.5946)),
        title = "Bengaluru"
    )
}
```

**iOS (SwiftUI Map, iOS 17+)**
```swift
import MapKit

@State private var position: MapCameraPosition = .region(
    MKCoordinateRegion(
        center: CLLocationCoordinate2D(latitude: 12.9716, longitude: 77.5946),
        span: MKCoordinateSpan(latitudeDelta: 0.1, longitudeDelta: 0.1)
    )
)

Map(position: $position) {
    Marker("Bengaluru", coordinate: CLLocationCoordinate2D(latitude: 12.9716, longitude: 77.5946))
}
```

The modern SwiftUI `Map` API (iOS 17+) is a very close conceptual match to Maps Compose — declarative, composable annotations, state-driven camera position.

---

## 📍 Multiple Markers from Data

**Android**
```kotlin
GoogleMap(cameraPositionState = cameraPositionState) {
    stores.forEach { store ->
        Marker(
            state = MarkerState(position = LatLng(store.lat, store.lng)),
            title = store.name
        )
    }
}
```

**iOS**
```swift
Map(position: $position) {
    ForEach(stores) { store in
        Marker(store.name, coordinate: store.coordinate)
    }
}
```

Both use a declarative loop to render markers from a data list — same pattern as rendering any other collection in Compose/SwiftUI.

---

## 🎯 Camera / Region Control

**Android**
```kotlin
LaunchedEffect(selectedStore) {
    cameraPositionState.animate(
        CameraUpdateFactory.newLatLngZoom(selectedStore.latLng, 15f)
    )
}
```

**iOS**
```swift
.onChange(of: selectedStore) { _, newStore in
    withAnimation {
        position = .region(
            MKCoordinateRegion(center: newStore.coordinate, span: closeSpan)
        )
    }
}
```

---

## 🛣 Polylines (Routes)

**Android**
```kotlin
Polyline(
    points = routePoints,
    color = Color.Blue,
    width = 8f
)
```

**iOS**
```swift
MapPolyline(coordinates: routeCoordinates)
    .stroke(.blue, lineWidth: 8)
```

---

## 🧭 User Location

**Android**
```kotlin
GoogleMap(
    properties = MapProperties(isMyLocationEnabled = true)
)
```

**iOS**
```swift
Map(position: $position) {
    UserAnnotation()
}
.mapControls {
    MapUserLocationButton()
}
```

Both require the location permission (see [19 - Permissions](./19-Permissions.md)) to be granted before the "my location" dot/button functions.

---

## ⚠️ Older UIKit-Based MapKit (Pre-iOS 17 / Legacy Projects)

For apps supporting iOS versions before 17, or in UIKit codebases, MapKit is wrapped imperatively via `MKMapView` + `UIViewRepresentable`:

```swift
struct MapViewRepresentable: UIViewRepresentable {
    func makeUIView(context: Context) -> MKMapView {
        MKMapView()
    }
    func updateUIView(_ uiView: MKMapView, context: Context) {
        // add annotations, set region, etc.
    }
}
```

This is analogous to using the older Android `MapView` (View-based, pre-Compose) wrapped in an `AndroidView` interop block.

---

## 📝 Quick Reference Table

| Concept | Google Maps (Android) | MapKit (iOS) |
|---|---|---|
| Map composable/view | `GoogleMap` | `Map` (SwiftUI, iOS 17+) |
| Camera state | `CameraPositionState` | `MapCameraPosition` |
| Marker/pin | `Marker` | `Marker` |
| Route line | `Polyline` | `MapPolyline` |
| Shape overlay | `Polygon` | `MapPolygon` |
| User location dot | `MapProperties(isMyLocationEnabled = true)` | `UserAnnotation()` |
| Legacy/imperative fallback | Older `MapView` + `AndroidView` | `MKMapView` + `UIViewRepresentable` |

---
