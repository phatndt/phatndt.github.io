---
title: Android Presentation Layer Architecture — MVP, MVVM, and MVI
date: 2026-02-24 00:00:00
categories: [Android]
tags: [architecture, mvp, mvvm, mvi, android]
---

The **Presentation Layer** is where your app meets the user — it handles UI rendering, user interactions, and state management. Choosing the right architectural pattern for this layer has a significant impact on code quality, testability, and maintainability.

In this post, we'll explore the three most widely used presentation patterns in Android development:

- **MVP** — Model-View-Presenter
- **MVVM** — Model-View-ViewModel
- **MVI** — Model-View-Intent

---

## 1. MVP — Model-View-Presenter

### Overview

MVP was one of the first patterns adopted widely in Android to address the problem of **fat Activities** — Activities that held both UI logic and business logic. It separates concerns into three components:

```
┌────────┐   user events   ┌───────────┐   data ops   ┌─────────┐
│  View  │ ──────────────► │ Presenter │ ────────────► │  Model  │
│        │ ◄────────────── │           │ ◄──────────── │         │
└────────┘  update UI      └───────────┘  data result  └─────────┘
```

- **Model** — Manages data (business logic, repository calls, etc.).
- **View** — A passive interface (Activity/Fragment) that renders UI and delegates all interaction to the Presenter.
- **Presenter** — The middleman. It retrieves data from the Model and tells the View what to display. It holds a direct reference to the View interface.

### How It Works

1. User taps a button in the **View**.
2. The **View** calls a method on the **Presenter** (e.g., `presenter.onLoginClicked(email, password)`).
3. The **Presenter** calls the **Model** (e.g., a Repository) to fetch or process data.
4. Once data is ready, the **Presenter** calls the **View** interface (e.g., `view.showUser(user)` or `view.showError(msg)`).

### Example (Kotlin)

```kotlin
// View Interface
interface LoginView {
    fun showLoading()
    fun hideLoading()
    fun showUser(name: String)
    fun showError(message: String)
}

// Presenter
class LoginPresenter(private val repo: UserRepository) {
    private var view: LoginView? = null

    fun attach(view: LoginView) { this.view = view }
    fun detach() { view = null }

    fun onLoginClicked(email: String, password: String) {
        view?.showLoading()
        repo.login(email, password) { result ->
            view?.hideLoading()
            result.fold(
                onSuccess = { view?.showUser(it.name) },
                onFailure = { view?.showError(it.message ?: "Unknown error") }
            )
        }
    }
}

// Activity
class LoginActivity : AppCompatActivity(), LoginView {
    private val presenter = LoginPresenter(UserRepository())

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        presenter.attach(this)
        binding.loginBtn.setOnClickListener {
            presenter.onLoginClicked(binding.email.text.toString(), binding.password.text.toString())
        }
    }

    override fun onDestroy() { presenter.detach(); super.onDestroy() }
    override fun showLoading() { binding.progress.visibility = View.VISIBLE }
    override fun hideLoading() { binding.progress.visibility = View.GONE }
    override fun showUser(name: String) { binding.userName.text = name }
    override fun showError(message: String) { Toast.makeText(this, message, Toast.LENGTH_SHORT).show() }
}
```

### Pros

- **Testable Presenter** — The Presenter has no Android dependency and can be tested with plain JUnit tests.
- **Clear separation** — The View is dumb; logic lives entirely in the Presenter.
- **Easy to understand** — Straightforward bi-directional communication.

### Cons

- **Tight coupling** — The Presenter holds a direct reference to the View interface, requiring careful attach/detach lifecycle management.
- **Boilerplate** — Every screen needs a View interface with many one-off methods (`showLoading`, `hideLoading`, `showX`, `showError`, …).
- **No built-in state survival** — Configuration changes (screen rotation) destroy the View, requiring manual state restoration.
- **Scales poorly** — Large Presenters become a new kind of "god class".

---

## 2. MVVM — Model-View-ViewModel

### Overview

MVVM is the **official recommended pattern** by Google for Android, built around `ViewModel` (from Android Architecture Components) and observable data streams (`LiveData` or `StateFlow`).

```
┌────────┐  observes state  ┌───────────┐  calls   ┌─────────┐
│  View  │ ◄──────────────  │ ViewModel │ ────────► │  Model  │
│        │  user events ──► │           │           │         │
└────────┘                  └───────────┘           └─────────┘
```

- **Model** — Data layer / Domain layer (UseCases, Repositories).
- **ViewModel** — Holds and manages UI state. Survives configuration changes. Does **not** hold a reference to the View.
- **View** — Observes state exposed by the ViewModel and re-renders accordingly.

### How It Works

1. The **ViewModel** exposes a `StateFlow<UiState>` (or `LiveData`).
2. The **View** (Activity/Fragment/Composable) collects/observes the state and renders UI accordingly.
3. User interactions are passed to the ViewModel via direct method calls.
4. The ViewModel calls the Model, updates the state, and the View reacts automatically.

### Example (Kotlin + StateFlow)

```kotlin
// UI State
data class LoginUiState(
    val isLoading: Boolean = false,
    val userName: String? = null,
    val errorMessage: String? = null
)

// ViewModel
class LoginViewModel(private val loginUseCase: LoginUseCase) : ViewModel() {

    private val _uiState = MutableStateFlow(LoginUiState())
    val uiState: StateFlow<LoginUiState> = _uiState.asStateFlow()

    fun login(email: String, password: String) {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true) }
            loginUseCase(email, password)
                .onSuccess { user ->
                    _uiState.update { it.copy(isLoading = false, userName = user.name) }
                }
                .onFailure { e ->
                    _uiState.update { it.copy(isLoading = false, errorMessage = e.message) }
                }
        }
    }
}

// Fragment
class LoginFragment : Fragment() {
    private val viewModel: LoginViewModel by viewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        viewLifecycleOwner.lifecycleScope.launch {
            viewModel.uiState.collect { state ->
                binding.progress.isVisible = state.isLoading
                state.userName?.let { binding.userName.text = it }
                state.errorMessage?.let { showToast(it) }
            }
        }
        binding.loginBtn.setOnClickListener {
            viewModel.login(binding.email.text.toString(), binding.password.text.toString())
        }
    }
}
```

### Pros

- **Lifecycle-aware** — ViewModel survives rotation; no manual attach/detach needed.
- **Testable** — ViewModel is easily unit-tested; just verify state emissions.
- **Reactive** — State changes automatically drive UI updates.
- **Less boilerplate** — No View interface required; state is a single data class.
- **Google-recommended** — First-class support with Jetpack, Compose, and Hilt.

### Cons

- **State can be fragmented** — Without discipline, multiple `LiveData`/`StateFlow` fields can make the state hard to reason about.
- **Two-way data binding pitfalls** — If using XML data binding (as in classic MVVM), logic can leak into layout files.
- **One-time events are tricky** — Navigation and snackbar events don't map cleanly to continuous state; requires workarounds (e.g., `Channel`, `SharedFlow`).
- **View still drives events** — Unlike MVI, there is no enforced single entry point for events.

---

## 3. MVI — Model-View-Intent

### Overview

MVI takes a **unidirectional data flow (UDF)** approach inspired by functional programming and Redux. It enforces a strict cycle:

```
         Intent (user action)
              │
              ▼
┌─────────────────────────────┐
│         ViewModel           │
│  process intent → reduce    │
│  current state + intent     │
│       = new state           │
└─────────────────────────────┘
              │
         State (new UI state)
              │
              ▼
          View renders
```

- **Model** — The current `UiState` — a single, immutable snapshot of what the UI should display.
- **View** — Renders state and emits `UiIntent`s (user actions).
- **Intent** — A sealed class representing every possible user action or event (e.g., `LoginIntent.Submit`, `LoginIntent.ClearError`).

### How It Works

1. The **View** emits an **Intent** (e.g., user taps login → `LoginIntent.Submit(email, password)`).
2. The **ViewModel** receives the Intent and feeds it into a **reducer** — a pure function: `(currentState, intent) → newState`.
3. The new **State** is emitted, and the **View** re-renders.
4. Side effects (navigation, one-time events) are emitted via a separate `UiEffect` channel.

### Example (Kotlin + MVI)

```kotlin
// Intent — all user actions
sealed class LoginIntent {
    data class Submit(val email: String, val password: String) : LoginIntent()
    object ClearError : LoginIntent()
}

// State — single source of truth
data class LoginUiState(
    val isLoading: Boolean = false,
    val userName: String? = null,
    val errorMessage: String? = null
)

// Effect — one-time side effects
sealed class LoginUiEffect {
    object NavigateToHome : LoginUiEffect()
}

// ViewModel
class LoginViewModel(private val loginUseCase: LoginUseCase) : ViewModel() {

    private val _uiState = MutableStateFlow(LoginUiState())
    val uiState: StateFlow<LoginUiState> = _uiState.asStateFlow()

    private val _effect = Channel<LoginUiEffect>(Channel.BUFFERED)
    val effect = _effect.receiveAsFlow()

    fun handleIntent(intent: LoginIntent) {
        when (intent) {
            is LoginIntent.Submit -> login(intent.email, intent.password)
            is LoginIntent.ClearError -> _uiState.update { it.copy(errorMessage = null) }
        }
    }

    private fun login(email: String, password: String) {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true, errorMessage = null) }
            loginUseCase(email, password)
                .onSuccess {
                    _uiState.update { it.copy(isLoading = false, userName = it.userName) }
                    _effect.send(LoginUiEffect.NavigateToHome)
                }
                .onFailure { e ->
                    _uiState.update { it.copy(isLoading = false, errorMessage = e.message) }
                }
        }
    }
}

// Fragment
class LoginFragment : Fragment() {
    private val viewModel: LoginViewModel by viewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        // Collect state
        viewLifecycleOwner.lifecycleScope.launch {
            viewModel.uiState.collect { state ->
                binding.progress.isVisible = state.isLoading
                state.userName?.let { binding.userName.text = it }
                state.errorMessage?.let { showToast(it) }
            }
        }

        // Collect effects
        viewLifecycleOwner.lifecycleScope.launch {
            viewModel.effect.collect { effect ->
                when (effect) {
                    is LoginUiEffect.NavigateToHome -> findNavController().navigate(R.id.action_to_home)
                }
            }
        }

        // Emit intents
        binding.loginBtn.setOnClickListener {
            viewModel.handleIntent(LoginIntent.Submit(binding.email.text.toString(), binding.password.text.toString()))
        }
    }
}
```

### Pros

- **Predictable state** — A single immutable state object is the only source of truth; no state can change without going through an Intent.
- **Easy debugging** — Every state transition is explicit and traceable (similar to Redux DevTools).
- **Clean one-time events** — Side effects are handled via a dedicated `Effect` channel.
- **Great for Compose** — Unidirectional data flow aligns perfectly with Jetpack Compose's rendering model.
- **Highly testable** — Input an Intent, assert on the resulting State/Effect — no mocking of Views needed.

### Cons

- **Verbosity** — Requires defining `Intent`, `State`, and `Effect` sealed classes, which is extra boilerplate even for simple screens.
- **Learning curve** — The pattern is less intuitive for developers new to functional/reactive programming.
- **Over-engineering risk** — For trivially simple screens, MVI can feel like overkill.
- **Immutable state copies** — Deeply nested state objects can be awkward to update with `copy()`.

---

## 4. Side-by-Side Comparison

| Feature | MVP | MVVM | MVI |
|---|---|---|---|
| **State management** | Manual, scattered methods | Reactive (`StateFlow`/`LiveData`) | Single immutable state object |
| **Data flow** | Bi-directional (View ↔ Presenter) | Bi-directional, but View-driven | Strictly unidirectional |
| **One-time events** | Callback methods on View | Requires workarounds (Channel/SharedFlow) | First-class `Effect` channel |
| **Testability** | Presenter is testable (mock View) | ViewModel is testable (no View needed) | Easiest: pure Intent → State assertions |
| **Lifecycle handling** | Manual attach/detach | Automatic (ViewModel survives rotation) | Automatic (ViewModel survives rotation) |
| **Boilerplate** | Medium (View interface + methods) | Low-medium | High (Intent + State + Effect) |
| **Debugging** | Hard (scattered state) | Medium | Easy (explicit state transitions) |
| **Compose compatibility** | Poor | Good | Excellent |
| **Learning curve** | Low | Medium | High |

---

## 5. When to Use Which Pattern

### Use **MVP** when:
- You're maintaining a **legacy codebase** that already uses it.
- Your team is new to Android architecture and needs a simple, understandable starting point.
- You're working in an environment where Kotlin Coroutines and Jetpack components are not available.

> ⚠️ For new projects, MVP is generally **not recommended**. Consider MVVM or MVI instead.

### Use **MVVM** when:
- You're building a **new Android project** with standard complexity.
- Your team is already familiar with `ViewModel`, `LiveData`, or `StateFlow`.
- You're using **Jetpack Compose** or a standard XML-based UI.
- The app has moderate complexity where full MVI feels like overkill.
- This is the **Google-recommended default** — a solid choice for most apps.

### Use **MVI** when:
- Your screens have **complex state** with many independent UI elements that need to be coordinated.
- You want **strict unidirectional data flow** and complete auditability of every state change.
- You're building with **Jetpack Compose**, where MVI's model aligns naturally.
- You have a team experienced with reactive/functional programming.
- You need **robust handling of one-time events** (navigation, dialogs, snackbars) without workarounds.

---

## 6. Summary

All three patterns solve the same fundamental problem — separating UI from business logic — but they do so with different trade-offs:

- **MVP** was the first step away from fat Activities but introduces tight coupling and lifecycle pain.
- **MVVM** is the modern sweet spot: reactive, lifecycle-safe, and Google-recommended.
- **MVI** takes MVVM further with a strict unidirectional cycle — ideal for complex, state-heavy screens, especially with Compose.

There is no universal "best" pattern. Choose based on your team's experience, app complexity, and the UI toolkit you're using. In practice, many Android codebases today use **MVVM as a base** and adopt **MVI conventions** (single state object, sealed Intent class, Effect channel) to get the best of both worlds.

---

## 7. References

- [Guide to app architecture — Android Developers](https://developer.android.com/topic/architecture)
- [ViewModel overview — Android Developers](https://developer.android.com/topic/libraries/architecture/viewmodel)
- [MVI Architecture — Orbit MVI](https://orbit-mvi.org/)
- [Unidirectional data flow on Android — Google I/O](https://www.youtube.com/watch?v=0IKHxjkgop4)
- [MVP, MVVM, MVI — ProAndroidDev](https://proandroiddev.com/mvi-architecture-with-kotlin-flows-and-channels-d36820b2028d)
