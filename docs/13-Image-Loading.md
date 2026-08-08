# 13 - Image Loading: Coil vs Kingfisher / AsyncImage

> Both platforms need a caching, async image-loading solution for remote images. iOS actually ships a basic native option (`AsyncImage`), but most production apps still reach for a library for caching control and features.

---

## 🔑 Core Philosophy

| Coil | Kingfisher / AsyncImage |
|---|---|
| Compose-first async image library | `AsyncImage` (native, basic) or Kingfisher (full-featured, like Coil) |
| Memory + disk caching built-in | Memory + disk caching (Kingfisher) or basic caching (AsyncImage) |
| Built on Coroutines | Built on Combine/async-await (Kingfisher) or native (AsyncImage) |

---

## 🖼 Basic Remote Image

**Compose (Coil)**
```kotlin
AsyncImage(
    model = "https://example.com/photo.jpg",
    contentDescription = "Profile photo",
    modifier = Modifier.size(100.dp),
    contentScale = ContentScale.Crop
)
```

**SwiftUI (native AsyncImage)**
```swift
AsyncImage(url: URL(string: "https://example.com/photo.jpg")) { image in
    image
        .resizable()
        .aspectRatio(contentMode: .fill)
} placeholder: {
    ProgressView()
}
.frame(width: 100, height: 100)
.clipped()
```

**SwiftUI (Kingfisher — closer to Coil's ergonomics)**
```swift
KFImage(URL(string: "https://example.com/photo.jpg"))
    .resizable()
    .aspectRatio(contentMode: .fill)
    .frame(width: 100, height: 100)
    .clipped()
```

> ⚠️ Native `AsyncImage` has **no disk caching** by default (only an in-memory `URLCache`-backed cache tied to the session) and reloads more aggressively than Coil. For production apps with lots of images (feeds, grids), **Kingfisher or SDWebImage** is the standard choice — much closer to what Coil gives you out of the box on Android.

---

## 🔄 Placeholder & Error States

**Coil**
```kotlin
AsyncImage(
    model = ImageRequest.Builder(context)
        .data(url)
        .placeholder(R.drawable.placeholder)
        .error(R.drawable.error_image)
        .build(),
    contentDescription = null
)
```

**Kingfisher**
```swift
KFImage(url)
    .placeholder { ProgressView() }
    .onFailure { error in
        // handle failure
    }
    .fallback(Image("error_image"))
```

---

## 🎯 Transformations (Circular, Rounded, Resized)

**Coil**
```kotlin
AsyncImage(
    model = url,
    contentDescription = null,
    modifier = Modifier
        .size(80.dp)
        .clip(CircleShape)
)
```

**SwiftUI (any image source)**
```swift
KFImage(url)
    .resizable()
    .frame(width: 80, height: 80)
    .clipShape(Circle())
```

Both frameworks apply shape clipping as a modifier on the final rendered view rather than baking it into the loader — the loader's job is just fetching/caching/decoding.

---

## 💾 Caching Configuration

**Coil**
```kotlin
val imageLoader = ImageLoader.Builder(context)
    .memoryCache {
        MemoryCache.Builder(context).maxSizePercent(0.25).build()
    }
    .diskCache {
        DiskCache.Builder().directory(context.cacheDir.resolve("image_cache")).build()
    }
    .build()
```

**Kingfisher**
```swift
ImageCache.default.memoryStorage.config.totalCostLimit = 50 * 1024 * 1024
ImageCache.default.diskStorage.config.sizeLimit = 200 * 1024 * 1024
```

Both provide global, app-wide cache configuration for memory and disk limits.

---

## 📥 Prefetching / Preloading

**Coil**
```kotlin
imageLoader.enqueue(ImageRequest.Builder(context).data(url).build())
```

**Kingfisher**
```swift
ImagePrefetcher(urls: [url1, url2, url3]).start()
```

---

## 📝 Quick Reference Table

| Concept | Coil (Compose) | Kingfisher / AsyncImage (SwiftUI) |
|---|---|---|
| Basic remote image | `AsyncImage(model = url)` | `AsyncImage(url:)` / `KFImage(url)` |
| Disk caching out of the box | ✅ Yes | ❌ Not with native `AsyncImage`; ✅ with Kingfisher |
| Placeholder | `.placeholder()` | `.placeholder { }` |
| Error fallback | `.error()` | `.fallback()` |
| Shape clipping | `Modifier.clip(CircleShape)` | `.clipShape(Circle())` |
| Prefetching | `imageLoader.enqueue()` | `ImagePrefetcher` |
| Cache config | `ImageLoader.Builder` | `ImageCache.default` config |

---
