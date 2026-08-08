# 12 - JSON Parsing: Moshi/Gson vs Codable

> Kotlin needs an external library for JSON serialization. Swift's `Codable` is part of the standard library — this is one area where iOS is actually simpler out of the box.

---

## 🔑 Core Philosophy

| Kotlin (Moshi/Gson/kotlinx.serialization) | Swift (Codable) |
|---|---|
| External dependency required | Built into the standard library |
| Reflection-based (Gson) or codegen (Moshi) | Compiler-synthesized `Encodable`/`Decodable` conformance |
| Annotations for custom mapping | `CodingKeys` enum for custom mapping |

---

## 📦 Basic Model Parsing

**Kotlin (Moshi)**
```kotlin
@JsonClass(generateAdapter = true)
data class Product(
    val id: Int,
    val name: String,
    val price: Double
)

val moshi = Moshi.Builder().build()
val adapter = moshi.adapter(Product::class.java)
val product = adapter.fromJson(jsonString)
```

**Swift (Codable)**
```swift
struct Product: Codable {
    let id: Int
    let name: String
    let price: Double
}

let decoder = JSONDecoder()
let product = try decoder.decode(Product.self, from: jsonData)
```

No annotation, no adapter generation step needed — just conform to `Codable` (which is `Encodable & Decodable` combined).

---

## 🔀 Nested Objects & Arrays

**Kotlin**
```kotlin
data class Order(
    val id: Int,
    val items: List<Product>,
    val customer: Customer
)
```

**Swift**
```swift
struct Order: Codable {
    let id: Int
    let items: [Product]
    let customer: Customer
}
```

Nested `Codable` structs/arrays decode automatically — same as nested data classes with Moshi, no extra config needed as long as every nested type also conforms.

---

## 🏷 Custom Key Names

**Kotlin (Moshi)**
```kotlin
data class User(
    @Json(name = "first_name") val firstName: String
)
```

**Swift (Codable)**
```swift
struct User: Codable {
    let firstName: String

    enum CodingKeys: String, CodingKey {
        case firstName = "first_name"
    }
}
```

---

## ❓ Optional / Nullable Fields

**Kotlin**
```kotlin
data class User(
    val id: Int,
    val nickname: String?  // nullable, may be missing
)
```

**Swift**
```swift
struct User: Codable {
    let id: Int
    let nickname: String?  // optional, may be missing or null
}
```

Both handle missing/null JSON keys gracefully when the field is nullable/optional. Swift's `Codable` will fail to decode if a **non-optional** field is missing — same strict behavior as Moshi with non-nullable fields.

---

## 🎭 Enums from JSON

**Kotlin**
```kotlin
enum class Status {
    @Json(name = "active") ACTIVE,
    @Json(name = "inactive") INACTIVE
}
```

**Swift**
```swift
enum Status: String, Codable {
    case active
    case inactive
}
```

If the JSON string matches the case name exactly, Swift needs zero extra config (`RawRepresentable` conformance via `String` handles it). Custom mismatched names use the same `CodingKeys`-style approach as key mapping.

---

## 🧩 Custom Decoding Logic

**Kotlin (Moshi custom adapter)**
```kotlin
class DateAdapter {
    @FromJson
    fun fromJson(dateString: String): Date = /* parse */
    @ToJson
    fun toJson(date: Date): String = /* format */
}
```

**Swift (custom init(from:))**
```swift
struct Event: Codable {
    let date: Date

    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        let dateString = try container.decode(String.self, forKey: .date)
        date = /* parse dateString */
        // ...
    }
}
```

Or, more commonly, set a custom date strategy globally:
```swift
let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .iso8601
```

---

## 🔄 Encoding (Object → JSON)

**Kotlin**
```kotlin
val json = adapter.toJson(product)
```

**Swift**
```swift
let encoder = JSONEncoder()
let jsonData = try encoder.encode(product)
```

---

## 📝 Quick Reference Table

| Concept | Kotlin (Moshi/Gson) | Swift (Codable) |
|---|---|---|
| Library required | Yes (external) | No (built-in) |
| Base protocol/annotation | `@JsonClass` | `Codable` (= `Encodable & Decodable`) |
| Custom key mapping | `@Json(name = "...")` | `CodingKeys` enum |
| Nullable field | `String?` | `String?` |
| Enum from string | `@Json(name = "...")` on cases | `String` raw value enum |
| Custom parsing logic | Custom adapter class | Custom `init(from:)` / strategy |
| Decode call | `adapter.fromJson(json)` | `decoder.decode(Type.self, from: data)` |
| Encode call | `adapter.toJson(obj)` | `encoder.encode(obj)` |

---
