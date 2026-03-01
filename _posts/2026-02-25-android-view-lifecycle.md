---
title: Android Activity & Fragment Lifecycle
date: 2026-02-25 00:00:00
categories: [Android]
tags: [lifecycle, activity, fragment]
---

> **Sources:** [Activity Lifecycle](https://developer.android.com/guide/components/activities/activity-lifecycle) · [Fragment Lifecycle](https://developer.android.com/guide/fragments/lifecycle)

---

## 🔵 Activity Lifecycle

An `Activity` has **6 core lifecycle callbacks** invoked by the system as it transitions between states.

### Lifecycle Flow

```
onCreate() → onStart() → onResume()
                             ↓
                         [App Running]
                             ↓
onPause() → onStop() → onDestroy()
     ↑            ↑
     └── onResume()  └── onRestart() → onStart()
```

### Callback Reference

| Callback | State | Description |
|---|---|---|
| `onCreate()` | **Created** | First callback. Perform one-time setup: inflate layout (`setContentView`), bind data, init ViewModels. Receives `savedInstanceState` (null on first creation). |
| `onStart()` | **Started** | Activity becomes **visible** to the user. Initialize UI maintenance code. Completes quickly. |
| `onResume()` | **Resumed** | Activity is in the **foreground** and interactive. Start camera, animations, or other resources needed while visible. |
| `onPause()` | **Paused** | User is **leaving** the activity (not always destroyed). Activity may still be visible (e.g., multi-window). Release sensors (GPS, camera), pause animations. **Do NOT** do heavy work here — it's brief. |
| `onStop()` | **Stopped** | Activity is **no longer visible**. Perform heavy-load shutdowns (e.g., save to DB). Use this instead of `onPause()` for UI-related releasing in multi-window mode. |
| `onDestroy()` | **Destroyed** | Final callback. Called when: (1) user dismisses the activity / `finish()` is called, or (2) system destroys due to a config change (rotation, multi-window). Use `ViewModel` to survive config changes. |

### Key Tips

- **`onRestart()`** is called when the activity returns from the Stopped state (not Destroyed) back to visible.
- **Never** save user data or make network calls in `onPause()` — use `onStop()` instead.
- **Configuration changes** (rotation) destroy and recreate the Activity; use `ViewModel` to preserve UI data across this.
- **Memory management**: The system kills processes (not activities directly) to free RAM. A `Stopped` activity is more likely to be killed than a `Paused` one.

### State & Memory Kill Likelihood

| Likelihood of being killed | Process state | Activity state |
|---|---|---|
| Least | Foreground | Resumed |
| More | Background (invisible) | Stopped |
| Most | — | Created / Started (before save) |

### Saving & Restoring State

Use a **combination** of strategies:

- **`ViewModel`** — survives config changes; preferred for complex UI state.
- **`onSaveInstanceState(outState: Bundle)`** — for lightweight transient state across process death.
- **`onRestoreInstanceState(savedInstanceState: Bundle)`** — called after `onStart()` when state is available.

```kotlin
override fun onSaveInstanceState(outState: Bundle) {
    outState.putString("GAME_STATE_KEY", gameState)
    super.onSaveInstanceState(outState)
}

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    gameState = savedInstanceState?.getString("GAME_STATE_KEY")
}
```

---

## 🟢 Fragment Lifecycle

A `Fragment` lifecycle is closely tied to its **host Activity** but has its own additional states and callbacks, especially around its **View**.

### Lifecycle States

Fragments move through these 5 states (managed by `FragmentManager`):

```
INITIALIZED → CREATED → STARTED → RESUMED
                ↓
            DESTROYED
```

A fragment progresses **upward** (e.g., added to back stack) and **downward** (e.g., popped from back stack).

> **Rule**: A child fragment's state can **never exceed** its parent fragment's or host activity's state.

### Callback Reference

#### ⬆️ Upward Transitions (Creating)

| Callback | When called | Use for |
|---|---|---|
| `onAttach(context)` | Fragment attached to host Activity via `FragmentManager` | Access host context; always called **before** any lifecycle state change |
| `onCreate(savedInstanceState)` | Fragment reaches **CREATED** state | Restore non-view state; fragment's **view is NOT yet created** here |
| `onCreateView(inflater, container, savedInstanceState)` | View is being created | Inflate or programmatically create the fragment's View |
| `onViewCreated(view, savedInstanceState)` | View fully created (View INITIALIZED) | Set up initial view state, observe `LiveData`, set adapters (`RecyclerView`, `ViewPager2`) |
| `onViewStateRestored(savedInstanceState)` | View state restored (View CREATED) | Restore additional view-related state |
| `onStart()` | Fragment & View reach **STARTED** | Safe to perform `FragmentTransaction` on child `FragmentManager` |
| `onResume()` | Fragment & View reach **RESUMED** | Fragment is fully visible, all animations done — user can interact |

#### ⬇️ Downward Transitions (Destroying)

| Callback | When called | Use for |
|---|---|---|
| `onPause()` | Fragment & View move back to **STARTED** | User is leaving fragment but it may still be visible |
| `onStop()` | Fragment & View move to **CREATED** | Fragment is no longer visible; last safe point for `FragmentTransaction` |
| `onDestroyView()` | View reaches **DESTROYED** | Remove all references to the fragment's View (allow GC) |
| `onDestroy()` | Fragment reaches **DESTROYED** | Clean up fragment-level resources |
| `onDetach()` | Fragment removed from `FragmentManager` | Always called **after** lifecycle state changes |

### Fragment vs View Lifecycle

A Fragment has **two** `Lifecycle` objects:
- **Fragment's own Lifecycle** — tied to the fragment instance.
- **Fragment's View Lifecycle** (`viewLifecycleOwner`) — tied to the view, only exists when a view is present.

> ✅ **Best practice**: Observe `LiveData` using `viewLifecycleOwner` (not `this`) to avoid memory leaks when the view is destroyed but the fragment instance remains.

```kotlin
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    // Use viewLifecycleOwner, NOT 'this'
    viewModel.data.observe(viewLifecycleOwner) { data ->
        // update UI
    }
}
```

### `onSaveInstanceState()` vs `onStop()` ordering

| API Level | Order |
|---|---|
| < API 28 | `onSaveInstanceState()` → `onStop()` |
| ≥ API 28 | `onStop()` → `onSaveInstanceState()` |

---

## 📊 Activity vs Fragment Lifecycle Comparison

| Activity | Fragment |
|---|---|
| `onCreate()` | `onCreate()` |
| — | `onCreateView()` |
| — | `onViewCreated()` |
| `onStart()` | `onStart()` |
| `onResume()` | `onResume()` |
| `onPause()` | `onPause()` |
| `onStop()` | `onStop()` |
| — | `onDestroyView()` |
| — | `onDestroy()` |
| `onDestroy()` | `onDetach()` |

Fragment has **more granular** callbacks, especially around its View lifecycle (`onCreateView`, `onViewCreated`, `onDestroyView`), because the View can be destroyed and recreated independently of the Fragment instance (e.g., when the fragment is on the back stack).

