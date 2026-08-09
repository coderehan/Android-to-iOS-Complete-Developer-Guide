# 26 - Testing: JUnit/Espresso vs XCTest/XCUITest

> Apple's testing story is unusually clean: **XCTest covers both unit and UI testing** in one framework, whereas Android splits unit tests (JUnit/Mockito) from instrumented UI tests (Espresso) more distinctly.

---

## 🔑 Core Comparison

| Android | iOS |
|---|---|
| JUnit (unit tests) | XCTest (unit tests) |
| Mockito / MockK (mocking) | Swift's protocol-based mocking, or Mockingbird/Cuckoo |
| Espresso (UI tests) | XCUITest (UI tests) — same framework family as XCTest |
| Turbine (Flow testing) | Combine/async testing via `XCTestExpectation` |
| Compose UI Testing (`createComposeRule`) | SwiftUI testing via `ViewInspector` (3rd-party) or XCUITest |

---

## ✅ Basic Unit Test

**Android (JUnit)**
```kotlin
class CalculatorTest {
    @Test
    fun `addition returns correct sum`() {
        val result = Calculator().add(2, 3)
        assertEquals(5, result)
    }
}
```

**iOS (XCTest)**
```swift
final class CalculatorTests: XCTestCase {
    func testAdditionReturnsCorrectSum() {
        let result = Calculator().add(2, 3)
        XCTAssertEqual(result, 5)
    }
}
```

Nearly identical structure — a test class, annotated/named test methods, and assertion functions (`assertEquals` ↔ `XCTAssertEqual`).

---

## 🎭 Mocking Dependencies

**Android (MockK)**
```kotlin
val repository = mockk<UserRepository>()
coEvery { repository.getUser(any()) } returns User(id = "1", name = "Rehan")

val viewModel = UserViewModel(repository)
viewModel.loadUser("1")

coVerify { repository.getUser("1") }
```

**iOS (protocol-based manual mock — most common native approach)**
```swift
protocol UserRepository {
    func getUser(id: String) async throws -> User
}

final class MockUserRepository: UserRepository {
    var getUserCalled = false
    var stubbedUser = User(id: "1", name: "Rehan")

    func getUser(id: String) async throws -> User {
        getUserCalled = true
        return stubbedUser
    }
}

let mockRepo = MockUserRepository()
let viewModel = UserViewModel(repository: mockRepo)
await viewModel.loadUser(id: "1")

XCTAssertTrue(mockRepo.getUserCalled)
```

> Swift's testing culture leans heavily on **manual protocol conformance mocks** rather than reflection-based mocking libraries like MockK/Mockito (Swift's strict typing and lack of runtime reflection make library-based mocking harder). This is the biggest workflow shift — expect to hand-write more mock classes on iOS, or adopt a library like Mockingbird for codegen-based mocking closer to MockK's ergonomics.

---

## ⏳ Testing Async Code

**Android (coroutines test)**
```kotlin
@Test
fun `loadUser updates state`() = runTest {
    viewModel.loadUser("1")
    assertEquals(expectedUser, viewModel.uiState.value.user)
}
```

**iOS (async test — modern XCTest supports async natively)**
```swift
func testLoadUserUpdatesState() async throws {
    await viewModel.loadUser(id: "1")
    XCTAssertEqual(viewModel.uiState.user, expectedUser)
}
```

Both platforms now support `async`/`suspend` test functions natively — no special wrapper needed beyond marking the test function itself `async`.

---

## 🖼 UI Testing

**Android (Compose UI Testing)**
```kotlin
@get:Rule
val composeTestRule = createComposeRule()

@Test
fun clickingButton_showsGreeting() {
    composeTestRule.setContent { MyScreen() }
    composeTestRule.onNodeWithText("Click me").performClick()
    composeTestRule.onNodeWithText("Hello!").assertIsDisplayed()
}
```

**iOS (XCUITest)**
```swift
final class MyAppUITests: XCTestCase {
    func testClickingButtonShowsGreeting() {
        let app = XCUIApplication()
        app.launch()
        app.buttons["Click me"].tap()
        XCTAssertTrue(app.staticTexts["Hello!"].exists)
    }
}
```

Both frameworks locate elements by text/identifier and simulate interactions. Espresso and Compose UI Testing run **in-process** against the app; XCUITest runs as a **separate process** driving the app via accessibility APIs — meaning XCUITest is closer to Android's older `UIAutomator` in execution model, even though its syntax feels like Espresso.

---

## 🧪 Snapshot Testing

Both ecosystems support snapshot/screenshot testing via popular third-party libraries:

| Android | iOS |
|---|---|
| Paparazzi, Shot | swift-snapshot-testing (Point-Free) |

---

## 📝 Quick Reference Table

| Concept | Android | iOS |
|---|---|---|
| Unit test framework | JUnit | XCTest |
| Assertion style | `assertEquals(expected, actual)` | `XCTAssertEqual(actual, expected)` |
| Mocking approach | MockK/Mockito (reflection-based) | Manual protocol mocks (or Mockingbird) |
| Async test support | `runTest { }` | `async` test functions (native) |
| UI test framework | Espresso / Compose UI Testing | XCUITest |
| UI test execution | In-process | Separate process (accessibility-driven) |
| Snapshot testing | Paparazzi, Shot | swift-snapshot-testing |

---
