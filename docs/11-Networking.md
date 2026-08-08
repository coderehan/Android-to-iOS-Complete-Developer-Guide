# 11 - Networking: Retrofit vs URLSession / Alamofire

> Retrofit is a declarative, annotation-based HTTP client built on OkHttp. iOS's native `URLSession` is lower-level and imperative by default — most teams add **Alamofire** on top for a Retrofit-like developer experience.

---

## 🔑 Core Philosophy

| Retrofit | URLSession / Alamofire |
|---|---|
| Annotation-based interface definitions | Manual request building (URLSession) or fluent chaining (Alamofire) |
| Built on OkHttp | Built on native `URLSession` (Apple's networking stack) |
| `suspend fun` for coroutines | `async/await` native support |
| Automatic JSON parsing via converters (Moshi/Gson) | `Codable` protocol for automatic JSON parsing |

---

## 📡 Defining an API Call

**Retrofit**
```kotlin
interface ApiService {
    @GET("users/{id}")
    suspend fun getUser(@Path("id") id: String): User

    @POST("users")
    suspend fun createUser(@Body user: User): User
}

val api = Retrofit.Builder()
    .baseUrl("https://api.example.com/")
    .addConverterFactory(MoshiConverterFactory.create())
    .build()
    .create(ApiService::class.java)
```

**URLSession (native, no library)**
```swift
func getUser(id: String) async throws -> User {
    let url = URL(string: "https://api.example.com/users/\(id)")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(User.self, from: data)
}
```

**Alamofire (Retrofit-like ergonomics)**
```swift
func getUser(id: String) async throws -> User {
    try await AF.request("https://api.example.com/users/\(id)")
        .serializingDecodable(User.self)
        .value
}
```

---

## 🧬 Model / DTO Definition

**Kotlin (with Moshi/Gson)**
```kotlin
data class User(
    val id: String,
    val name: String,
    val email: String
)
```

**Swift (Codable)**
```swift
struct User: Codable {
    let id: String
    let name: String
    let email: String
}
```

`Codable` is Swift's **built-in** serialization protocol — no external library needed for basic JSON encode/decode, unlike Kotlin which needs Moshi/Gson/kotlinx.serialization as a separate dependency.

---

## 🔀 Custom Key Mapping (snake_case → camelCase)

**Kotlin (Moshi)**
```kotlin
data class User(
    @Json(name = "user_id") val userId: String,
    @Json(name = "full_name") val fullName: String
)
```

**Swift (Codable)**
```swift
struct User: Codable {
    let userId: String
    let fullName: String

    enum CodingKeys: String, CodingKey {
        case userId = "user_id"
        case fullName = "full_name"
    }
}
```

Or globally, for consistent snake_case APIs:
```swift
let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase
```

---

## 🌐 POST Request with Body

**Retrofit**
```kotlin
@POST("login")
suspend fun login(@Body request: LoginRequest): LoginResponse
```

**Alamofire**
```swift
func login(request: LoginRequest) async throws -> LoginResponse {
    try await AF.request(
        "https://api.example.com/login",
        method: .post,
        parameters: request,
        encoder: JSONParameterEncoder.default
    )
    .serializingDecodable(LoginResponse.self)
    .value
}
```

---

## 🔑 Headers & Interceptors (Auth Tokens)

**Retrofit (OkHttp Interceptor)**
```kotlin
class AuthInterceptor(private val tokenProvider: () -> String) : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request().newBuilder()
            .addHeader("Authorization", "Bearer ${tokenProvider()}")
            .build()
        return chain.proceed(request)
    }
}
```

**Alamofire (RequestInterceptor)**
```swift
final class AuthInterceptor: RequestInterceptor {
    func adapt(_ urlRequest: URLRequest, for session: Session, completion: @escaping (Result<URLRequest, Error>) -> Void) {
        var request = urlRequest
        request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        completion(.success(request))
    }
}
```

Both frameworks support global interceptors for auth headers, retry logic, and refresh-token flows.

---

## ⚠️ Error Handling

**Kotlin**
```kotlin
try {
    val user = api.getUser(id)
} catch (e: HttpException) {
    // non-2xx response
} catch (e: IOException) {
    // network failure
}
```

**Swift**
```swift
do {
    let user = try await getUser(id: id)
} catch let error as URLError {
    // network failure
} catch let error as DecodingError {
    // JSON parsing failure
} catch {
    // other
}
```

---

## 📝 Quick Reference Table

| Concept | Retrofit (Kotlin) | URLSession/Alamofire (Swift) |
|---|---|---|
| API definition style | Annotated interface | Function per call (manual or Alamofire chain) |
| JSON parsing | Moshi/Gson (external) | `Codable` (built-in) |
| Async support | `suspend fun` | `async/await` (native) |
| Custom key mapping | `@Json(name = "...")` | `CodingKeys` enum |
| Auth interceptor | OkHttp `Interceptor` | Alamofire `RequestInterceptor` |
| Base client | OkHttp | `URLSession` |
| Popular 3rd-party wrapper | (Retrofit itself) | Alamofire |
| Multipart/file upload | `@Multipart` | `MultipartFormData` (Alamofire) |

---
