# Kotlin Version Differences for Android Developers (1.6.0 - 2.1.20)

This document summarizes key Android-relevant language and coroutine changes in Kotlin versions from 1.6.0 to 2.1.20.

## Kotlin 1.6.0

### Language
- **Stable Exhaustive `when` statements for sealed types:** The Kotlin compiler now provides better warnings for non-exhaustive `when` statements on sealed types, helping to prevent runtime errors.
  ```kotlin
  sealed class Result
  data class Success(val data: String) : Result()
  data class Error(val exception: Exception) : Result()

  fun handleResult(result: Result) {
      when (result) { // Compiler warning if a subtype is missed and no else branch
          is Success -> println("Success: ${result.data}")
          is Error -> println("Error: ${result.exception.message}")
      }
  }
  ```
- **Improvements to type inference:** More scenarios benefit from improved type inference, reducing boilerplate.
- **Stable `suspend` conversions:** Functions that take suspend functions as parameters are now stable.
  ```kotlin
  fun execute(action: suspend () -> Unit) { /* ... */ }
  ```

### Coroutines (kotlinx.coroutines 1.6.0)
- **New `tryCatch` block for `Flow`:** Simplified error handling in Flows. (Note: This was introduced in earlier versions but refined).
  ```kotlin
  flow {
      emit(1)
      throw RuntimeException("Error in flow")
  }.catch { e ->
      println("Caught error: ${e.message}")
  }.collect { value ->
      println("Received $value")
  }
  ```
- **`Channel` API improvements:** Enhancements to the Channel API for better concurrent programming.

## Kotlin 1.7.0

### Language
- **Builder inference improvements:** Builder inference, which helps the compiler infer type arguments for generic builder functions, became stable and more powerful.
  ```kotlin
  fun <T> buildList(builderAction: MutableList<T>.() -> Unit): List<T> {
      return mutableListOf<T>().apply(builderAction)
  }

  val myList = buildList { // Type of 'this' inside lambda is MutableList<String>
      add("Hello")
      add("World")
      // No need to specify type arguments for buildList explicitly
  }
  ```
- **Underscore operator for type arguments:** You can use `_` to automatically infer one or more type arguments when others are specified.
  ```kotlin
  class MyGenericClass<T, U>
  val instance: MyGenericClass<String, _> = MyGenericClass<String, Int>()
  ```
- **Opt-in requirements are stable:** The `@RequiresOptIn` and `@OptIn` annotations are now stable, providing a standardized way to handle APIs that require explicit consent to use.

### Coroutines (kotlinx.coroutines 1.6.x continued)
- **`SharedFlow` and `StateFlow` enhancements:** Ongoing improvements for these hot flow types, making them more robust for state management in Android.
- **Performance improvements:** Various performance optimizations in the coroutines library.

## Kotlin 1.8.0

### Language
- **Recursive generic types:** Improved support for defining recursive generic types.
- **`kotlin-reflect` performance improvements:** Reflection library optimizations.
- **New experimental functions for `java.nio.file.Path`:** Utility functions for working with file paths (more relevant for server-side/tooling but can be used in Android where appropriate).

### Coroutines (kotlinx.coroutines 1.7.x)
- **`updateAndGet` and `getAndUpdate` for `MutableStateFlow`:** Atomic operations for updating `MutableStateFlow` more conveniently.
  ```kotlin
  val stateFlow = MutableStateFlow(0)
  // Increment and get the new value
  val newValue = stateFlow.updateAndGet { currentValue -> currentValue + 1 }
  // Get the old value and then update
  val oldValue = stateFlow.getAndUpdate { currentValue -> currentValue + 1 }
  ```
- **Test scheduler improvements:** Enhanced capabilities for testing coroutines, including better control over time.

## Kotlin 1.9.0

### Language
- **Stable Kotlin K2 compiler (Alpha/Beta):** While not fully stable for production in 1.9.0 for all projects, the K2 compiler effort brought significant performance and architecture improvements. The focus was on making it available for testing and feedback.
- **Enum class `entries` property (Preview):** A modern, performant replacement for the `values()` function to get enum constants.
  ```kotlin
  enum class Color { RED, GREEN, BLUE }

  // Old way
  // Color.values().forEach { println(it) }

  // New way (Preview in 1.9.0, Stable in 1.9.20+)
  // Color.entries.forEach { println(it) }
  ```
- **Data Objects:** For objects that are primarily data containers without identity semantics, similar to data classes but for `object` declarations.
  ```kotlin
  data object MySingletonServiceConfig {
      const val API_URL = "https://example.com"
  }
  ```

### Coroutines (kotlinx.coroutines 1.7.x continued)
- **Further stability and performance improvements.**
- **Integration with K2 compiler:** Ensuring coroutines work seamlessly with the new compiler frontend.

## Kotlin 2.0.0 (Focus on K2 Compiler)

### Language
- **Stable Kotlin K2 Compiler:** The K2 compiler becomes the default and stable compiler. This is the biggest change, bringing:
    - **Faster compilation:** Significant speed improvements for many projects.
    - **Improved type inference algorithms.**
    - **More robust and maintainable compiler architecture.**
    - **Better support for multiplatform projects.**
- **Stable Enum class `entries` property:** The `entries` property for enums is now stable.
  ```kotlin
  enum class Status { LOADING, SUCCESS, FAILED }

  for (statusEntry in Status.entries) {
      println(statusEntry.name)
  }
  ```
- **Value classes with multiple properties (Experimental):** Extending the capabilities of value classes.
- **Context Receivers (Beta):** A powerful new feature for abstracting over context, allowing functions and classes to declare what contextual information they need without explicitly passing it as parameters. This can be particularly useful for dependency injection and managing contextual data in a cleaner way.
  ```kotlin
  // Hypothetical example, syntax might evolve
  context(CoroutineScope, DispatcherProvider)
  fun loadData() {
    // launch coroutines using CoroutineScope
    // get dispatchers from DispatcherProvider
  }
  ```

### Coroutines (kotlinx.coroutines 1.8.0)
- **Full compatibility and optimization for K2 compiler.**
- **`Channel` and `Flow` performance improvements.**
- **Refinements to Structured Concurrency APIs and debugging.**
- **Updates to `kotlinx-coroutines-test` for better testing of `StateFlow` and `SharedFlow`.**

## Kotlin 2.0.20 (Aug 2024)

This is an incremental update to Kotlin 2.0.0.

*   **Language:**
    *   **Data class `copy()` visibility to match constructor (Warning phase):** In future releases, the generated `copy()` function will have the same visibility as its primary constructor. Kotlin 2.0.20 issues a warning where visibility will change. `@ConsistentCopyVisibility` to opt-in now, `@ExposedCopyVisibility` to opt-out at declaration.
        ```kotlin
        // @file:OptIn(ExperimentalStdlibApi::class) // If using @ConsistentCopyVisibility
        data class User private constructor(val id: Int, val name: String) {
            // @ConsistentCopyVisibility // Opt-in to new behavior
            companion object {
                fun create(id: Int, name: String) = User(id, name)
            }
        }
        // val user1 = User.create(1, "Alice")
        // val user2 = user1.copy(name = "Bob") // Warning in 2.0.20
        ```
    *   **Context receivers being replaced by context parameters (Warning phase):** Using experimental context receivers (`-Xcontext-receivers`) now issues a warning. Migration to explicit parameters or extension members is advised.

*   **Kotlin/Native (Experimental):**
    *   **Concurrent marking in GC (`kotlin.native.binary.gc=cms`):** Aims to reduce GC pause times.

*   **Compose Compiler (Bundled with Kotlin 2.0.20):**
    *   **Strong skipping mode enabled by default.**
    *   Fix for recomposition issues from 2.0.0.
    *   Default parameters in abstract composable functions supported.

*   **Standard Library (Experimental):**
    *   **UUID support (`kotlin.uuid.Uuid`, `@OptIn(ExperimentalUuidApi)`).**
        ```kotlin
        // import kotlin.uuid.Uuid // Requires opt-in
        // @OptIn(kotlin.uuid.ExperimentalUuidApi::class)
        // fun generateId(): String = Uuid.random().toString()
        ```
    *   **Base64 decoder now requires padding by default; `withPadding()` for configuration.**

## Kotlin 2.1.0 (Nov 2024 estimate)

*   **Language (Preview, Opt-in):**
    *   **Guard conditions in `when` with subject (`-Xwhen-guards`):** `is Type if condition ->`.
        ```kotlin
        sealed interface Response {
            data class Success(val body: String, val code: Int) : Response()
            data object Failure : Response()
        }
        fun handle(response: Response) {
            when (response) {
                is Response.Success if response.code == 200 -> println("OK: ${response.body}")
                is Response.Success if response.code == 204 -> println("No Content")
                is Response.Success -> println("Success code ${response.code}")
                is Response.Failure -> println("Failed")
            }
        }
        ```
    *   **Non-local `break` and `continue` (`-Xnon-local-break-continue`):** For inline function lambda loops.
    *   **Multi-dollar string interpolation (`-Xmulti-dollar-interpolation`).**

*   **K2 Compiler:**
    *   K2 `kapt` implementation improved (Alpha).

*   **Kotlin/JVM:**
    *   JSpecify nullability mismatch diagnostics `strict` by default.

*   **Kotlin Multiplatform:**
    *   **New Gradle DSL for KMP compiler options (Stable).**
    *   **Basic Swift Export (Early Preview, `kotlin.experimental.swift-export.enabled=true`).**

*   **Kotlin/Native:**
    *   `iosArm64` promoted to Tier 1.
    *   LLVM updated to 16.0.0.

*   **Kotlin/Wasm (Alpha):**
    *   Incremental compilation support (Opt-in `kotlin.incremental.wasm=true`).
    *   Browser APIs moved to `kotlinx-browser` library.

*   **Standard Library:**
    *   Several API deprecations raised to ERROR (locale-sensitive case conversion, Native freezing API, `appendln`).
    *   Stable file tree traversal extensions for `java.nio.file.Path`.

## Kotlin 2.1.20 (Mar 2025 estimate) & 2.1.21 (May 2025 - Bug Fixes)

Kotlin 2.1.20 introduces further refinements and new experimental features. 2.1.21 is a bug-fix release for 2.1.20.

*   **K2 Compiler:**
    *   **K2 `kapt` plugin default.**
    *   Lombok plugin: `@SuperBuilder` support.

*   **Kotlin Multiplatform (Experimental):**
    *   New `executable{}` DSL for KMP JVM targets (replaces Gradle Application plugin usage).
        ```kotlin
        // kotlin {
        //  jvm {
        //      @OptIn(ExperimentalKotlinGradlePluginApi::class)
        //      binaries {
        //          executable { mainClass.set("com.example.MainKt") }
        //      }
        //  }
        // }
        ```

*   **Kotlin/Native (Experimental):**
    *   Support for Xcode 16.3.
    *   New inlining optimization (`-Xbinary=preCodegenInlineThreshold=N`).

*   **Standard Library (Experimental):**
    *   **Common atomic types (`kotlin.concurrent.atomics`, `@OptIn(ExperimentalAtomicApi)`).**
        ```kotlin
        // import kotlin.concurrent.atomics.AtomicInt // Requires opt-in
        // @OptIn(kotlin.concurrent.atomics.ExperimentalAtomicApi::class)
        // val atomicCounter = AtomicInt(0)
        // fun increment() { atomicCounter.incrementAndGet() }
        ```
    *   **`kotlin.time.Clock` and `kotlin.time.Instant` (from `kotlinx-datetime`, `@OptIn(ExperimentalTime)`).**
        ```kotlin
        // import kotlin.time.Clock // Requires opt-in
        // @OptIn(kotlin.time.ExperimentalTime::class)
        // val now = Clock.System.now()
        ```

*   **Compose Compiler (Bundled):**
    *   Default arguments in `open @Composable` functions supported.
    *   `final` overridden `@Composable` functions can be restartable.

*Note: For versions 2.0.20 and later, some features are marked Experimental or Preview. Their API and behavior might change in future stable releases. Always refer to the official Kotlin documentation for the latest status.*
