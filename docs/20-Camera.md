# 20 - Camera: CameraX vs AVFoundation

> Both are the platform-native, low-level camera frameworks. CameraX is Jetpack's modern abstraction over Camera2; AVFoundation is Apple's long-standing (and still current) media capture framework.

---

## 🔑 Core Philosophy

| CameraX | AVFoundation |
|---|---|
| Jetpack library, abstracts Camera2 complexity | Apple's native, lower-level media framework |
| Lifecycle-aware (binds to `LifecycleOwner`) | Manual session lifecycle management |
| `Preview`, `ImageCapture`, `ImageAnalysis` use cases | `AVCaptureSession` + inputs/outputs |
| Simple third-party wrapper: CameraX itself | Simple third-party wrapper: `VisionCamera` (popular in RN, also usable natively) |

---

## 📸 Basic Camera Preview + Capture Setup

**Android (CameraX)**
```kotlin
val preview = Preview.Builder().build().also {
    it.setSurfaceProvider(previewView.surfaceProvider)
}

val imageCapture = ImageCapture.Builder().build()

val cameraProvider = ProcessCameraProvider.getInstance(context).get()
cameraProvider.bindToLifecycle(
    lifecycleOwner,
    CameraSelector.DEFAULT_BACK_CAMERA,
    preview,
    imageCapture
)
```

**iOS (AVFoundation)**
```swift
let session = AVCaptureSession()
session.sessionPreset = .photo

guard let device = AVCaptureDevice.default(for: .video),
      let input = try? AVCaptureDeviceInput(device: device) else { return }
session.addInput(input)

let photoOutput = AVCapturePhotoOutput()
session.addOutput(photoOutput)

let previewLayer = AVCaptureVideoPreviewLayer(session: session)
previewLayer.frame = view.bounds
view.layer.addSublayer(previewLayer)

session.startRunning()
```

AVFoundation is noticeably more manual/verbose than CameraX — you assemble a session, wire inputs/outputs, and manage the running state yourself. CameraX's `bindToLifecycle` (auto start/stop tied to the Activity/Fragment lifecycle) has no direct 1:1 equivalent; you manage `startRunning()`/`stopRunning()` yourself, typically in `onAppear`/`onDisappear`.

---

## 📷 Taking a Photo

**Android**
```kotlin
imageCapture.takePicture(
    outputFileOptions,
    ContextCompat.getMainExecutor(context),
    object : ImageCapture.OnImageSavedCallback {
        override fun onImageSaved(output: ImageCapture.OutputFileResults) {
            // saved
        }
        override fun onError(exception: ImageCaptureException) {
            // error
        }
    }
)
```

**iOS**
```swift
class PhotoCaptureDelegate: NSObject, AVCapturePhotoCaptureDelegate {
    func photoOutput(_ output: AVCapturePhotoOutput, didFinishProcessingPhoto photo: AVCapturePhoto, error: Error?) {
        guard let data = photo.fileDataRepresentation() else { return }
        // save `data` to disk
    }
}

photoOutput.capturePhoto(with: AVCapturePhotoSettings(), delegate: photoCaptureDelegate)
```

Both use a callback/delegate pattern — Android's `OnImageSavedCallback` ↔ iOS's `AVCapturePhotoCaptureDelegate`.

---

## 🎨 SwiftUI Wrapper Pattern

SwiftUI (like Compose) has no built-in camera view — you wrap `AVCaptureSession` in a `UIViewRepresentable`, the SwiftUI equivalent of Compose's `AndroidView` interop:

```swift
struct CameraPreview: UIViewRepresentable {
    let session: AVCaptureSession

    func makeUIView(context: Context) -> UIView {
        let view = UIView()
        let previewLayer = AVCaptureVideoPreviewLayer(session: session)
        previewLayer.frame = view.bounds
        view.layer.addSublayer(previewLayer)
        return view
    }

    func updateUIView(_ uiView: UIView, context: Context) {}
}
```

Compare to Compose's `AndroidView` wrapping a `PreviewView` — same "drop down to the imperative UIKit/View layer for camera preview" pattern in both.

---

## 🔍 Third-Party: Vision Camera (Popular Cross-Ecosystem Wrapper)

Many teams (especially those with React Native experience, like from your RN guide) reach for **Vision Camera** even in native iOS/Android contexts for a higher-level, more CameraX-like API with frame processors for ML/QR scanning use cases — worth mentioning since it smooths over a lot of AVFoundation's verbosity.

---

## 📝 Quick Reference Table

| Concept | CameraX (Android) | AVFoundation (iOS) |
|---|---|---|
| Session/provider | `ProcessCameraProvider` | `AVCaptureSession` |
| Preview | `Preview` use case + `PreviewView` | `AVCaptureVideoPreviewLayer` |
| Photo capture | `ImageCapture` use case | `AVCapturePhotoOutput` |
| Frame analysis (ML/QR) | `ImageAnalysis` use case | `AVCaptureVideoDataOutput` |
| Lifecycle binding | Automatic (`bindToLifecycle`) | Manual (`startRunning`/`stopRunning`) |
| UI embedding | `AndroidView` (Compose interop) | `UIViewRepresentable` (SwiftUI interop) |
| Permission key | `CAMERA` | `NSCameraUsageDescription` |

---
