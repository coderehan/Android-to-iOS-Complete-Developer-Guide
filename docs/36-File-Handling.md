# 36 - File Handling: Android Storage vs iOS FileManager

> Android's storage model has evolved significantly (Scoped Storage, SAF). iOS has always had a simpler, more sandboxed model — every app gets its own private container, with no equivalent to Android's shared/external storage complexity.

---

## 🔑 Core Philosophy

| Android | iOS |
|---|---|
| Internal storage (private, app-specific) | App's sandboxed container (always private) |
| External/Shared storage (Scoped Storage since Android 10) | No shared storage equivalent — sandbox only |
| Storage Access Framework (SAF) for user-picked files | `UIDocumentPickerViewController` for user-picked files |
| `MediaStore` for shared media | `PHPhotoLibrary` (Photos framework) for photo library access |

> ✅ iOS's sandboxing model is actually **simpler** to reason about than Android's — there's no "internal vs external storage" distinction to learn. Every app just gets one private container, full stop.

---

## 📁 App's Private Directory

**Android**
```kotlin
val internalDir = context.filesDir
val file = File(internalDir, "notes.txt")
file.writeText("Hello")

val cacheDir = context.cacheDir // cleared by system when low on space
```

**iOS**
```swift
let documentsDir = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
let fileURL = documentsDir.appendingPathComponent("notes.txt")
try "Hello".write(to: fileURL, atomically: true, encoding: .utf8)

let cachesDir = FileManager.default.urls(for: .cachesDirectory, in: .userDomainMask).first! // system may clear
```

| Android Directory | iOS Directory |
|---|---|
| `context.filesDir` | `Documents/` (user-visible if file sharing enabled) |
| `context.cacheDir` | `Library/Caches/` (system may purge) |
| `context.noBackupFilesDir` | `Library/Application Support/` (excluded from backup if flagged) |
| `context.getExternalFilesDir()` | No equivalent — sandbox has no "external" concept |

---

## 📤 Sharing Files with Other Apps

**Android**
```kotlin
val uri = FileProvider.getUriForFile(context, "${context.packageName}.provider", file)
val intent = Intent(Intent.ACTION_SEND).apply {
    type = "text/plain"
    putExtra(Intent.EXTRA_STREAM, uri)
}
startActivity(Intent.createChooser(intent, "Share"))
```

**iOS**
```swift
let activityVC = UIActivityViewController(activityItems: [fileURL], applicationActivities: nil)
// present via UIViewControllerRepresentable in SwiftUI
```

`FileProvider` + `ACTION_SEND` ↔ `UIActivityViewController` — both present the OS-native share sheet, letting the user pick a target app (Mail, Messages, another app, etc.) without your app needing direct integration with each destination.

---

## 📂 Letting the User Pick a File

**Android (Storage Access Framework)**
```kotlin
val launcher = registerForActivityResult(ActivityResultContracts.OpenDocument()) { uri ->
    uri?.let { /* read from it */ }
}
launcher.launch(arrayOf("application/pdf"))
```

**iOS (UIDocumentPickerViewController via SwiftUI)**
```swift
.fileImporter(isPresented: $showPicker, allowedContentTypes: [.pdf]) { result in
    switch result {
    case .success(let url): /* read from it */
    case .failure(let error): /* handle */
    }
}
```

SwiftUI's `.fileImporter` modifier is a direct, native equivalent of Android's `OpenDocument` contract — both hand you a security-scoped URI/URL rather than direct file access, requiring you to request access before reading.

---

## 🖼 Saving to Photo Library

**Android**
```kotlin
val values = ContentValues().apply {
    put(MediaStore.Images.Media.DISPLAY_NAME, "photo.jpg")
    put(MediaStore.Images.Media.MIME_TYPE, "image/jpeg")
}
val uri = context.contentResolver.insert(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, values)
```

**iOS**
```swift
import Photos

PHPhotoLibrary.shared().performChanges({
    PHAssetChangeRequest.creationRequestForAsset(from: image)
}) { success, error in
    // handle result
}
```

Both require a permission (`WRITE_EXTERNAL_STORAGE`/scoped media permission on Android, `NSPhotoLibraryAddUsageDescription` on iOS) before writing to the shared photo library.

---

## 📝 Quick Reference Table

| Concept | Android | iOS |
|---|---|---|
| Private files dir | `context.filesDir` | `Documents/` directory |
| Cache dir | `context.cacheDir` | `Library/Caches/` |
| Share a file | `FileProvider` + `ACTION_SEND` | `UIActivityViewController` |
| Let user pick a file | Storage Access Framework | `.fileImporter` |
| Save to photo library | `MediaStore` | `PHPhotoLibrary` |
| Permission for media write | Scoped storage permission | `NSPhotoLibraryAddUsageDescription` |

---
