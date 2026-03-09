---
title: Android Activity Launch Mode
date: 2026-03-08 00:00:00
categories: [Android]
tags: [activity, launch mode, task, back stack]
---

> **Source:** [Android Docs — Tasks and the back stack](https://developer.android.com/guide/components/activities/tasks-and-back-stack)

---

## 📌 Tasks & the Back Stack

Before diving into launch modes, it's important to understand **tasks** and the **back stack**.

A **task** is a collection of activities that a user interacts with when performing a job. Activities in a task are arranged in a **back stack** — a last-in, first-out (LIFO) structure.

```
Task (Back Stack)
┌──────────────┐
│  Activity D  │ ← Top (current foreground)
├──────────────┤
│  Activity C  │
├──────────────┤
│  Activity B  │
├──────────────┤
│  Activity A  │ ← Bottom (root)
└──────────────┘
```

- When a new activity is started, it is **pushed** onto the top of the stack.
- When the user presses **Back**, the current activity is **popped** off the stack and the one underneath becomes active.
- The task moves to the **background** when the user navigates to the home screen, but the back stack is preserved.

---

## 🚀 What is Launch Mode?

**Launch mode** defines how a new instance of an activity is created when it is started, and how it interacts with the current task and back stack.

You can set launch mode in two ways:

**1. `AndroidManifest.xml`** — applies to all launches of the activity:
```xml
<activity
    android:name=".MyActivity"
    android:launchMode="singleTop" />
```

**2. Intent flags** — applies dynamically per-launch, overrides manifest:
```kotlin
val intent = Intent(this, MyActivity::class.java)
intent.flags = Intent.FLAG_ACTIVITY_SINGLE_TOP
startActivity(intent)
```

---

## 🔵 The Four Launch Modes

### 1. `standard` (Default)

| Property | Value |
|---|---|
| Multiple instances? | ✅ Yes |
| Must be on top to reuse? | N/A — always creates new |
| `onNewIntent()` called? | ❌ No |
| Own task? | ❌ No |

**Behavior:** Every `startActivity()` call creates a **brand new instance** of the activity, regardless of whether one already exists in the stack. Multiple instances can exist in the same task or in different tasks.

```
Start A → Start B → Start A → Start A

Back Stack: [ A | B | A | A ]
                          ↑ top
```

**Use case:** Most activities — detail screens, editor screens, forms. Each instance holds independent state (e.g., `EmailDetailActivity` showing a different email each time).

---

### 2. `singleTop`

| Property | Value |
|---|---|
| Multiple instances? | ✅ Yes (but not consecutive duplicates on top) |
| Must be on top to reuse? | ✅ Yes |
| `onNewIntent()` called? | ✅ Yes (when reused) |
| Own task? | ❌ No |

**Behavior:** If an instance of the activity is **already at the top** of the current task's back stack, the system does **not** create a new instance. Instead, it delivers the new intent to the existing instance via `onNewIntent()`. If the activity is anywhere else (not at top), a new instance is created normally.

```
Back Stack: [ A | B | C ]  →  Start C again  →  [ A | B | C ]
                                                           ↑ onNewIntent() called

Back Stack: [ A | B | C ]  →  Start B         →  [ A | B | C | B ]
                                                              ↑ new instance (B not on top)
```

```kotlin
override fun onNewIntent(intent: Intent) {
    super.onNewIntent(intent)
    // Handle the new intent (e.g., update search query)
    val newQuery = intent.getStringExtra("QUERY")
    performSearch(newQuery)
}
```

**Use case:** Search results screen, notification-triggered screens. Prevents stacking duplicate activities when a user taps a notification while that screen is already open.

---

### 3. `singleTask`

| Property | Value |
|---|---|
| Multiple instances? | ❌ No (only one globally) |
| Must be on top to reuse? | ❌ No — found anywhere in its task |
| `onNewIntent()` called? | ✅ Yes |
| Own task? | ✅ Yes (acts as root of its task) |

**Behavior:** Only **one instance** can exist in the entire system. If the activity already exists, the system brings its task to the foreground, **clears all activities above it** in that task (they are destroyed), and delivers the new intent via `onNewIntent()`. If no instance exists, a new task is created with the activity as its root.

```
Task: [ A | B | C | D ]   →   Start C (singleTask)

Result: [ A | B | C ]
                  ↑ D was destroyed, onNewIntent() called on C
```

```xml
<activity
    android:name=".MainActivity"
    android:launchMode="singleTask"
    android:taskAffinity="" />
```

**Use case:** Main/home screen of an app, dashboard activity. Ensures the user always returns to a single, consistent entry point regardless of how deep they navigated.

> ⚠️ **Warning:** Activities above the `singleTask` activity in the stack are **permanently destroyed** when it is re-launched. Make sure they don't hold unsaved state.

---

### 4. `singleInstance`

| Property | Value |
|---|---|
| Multiple instances? | ❌ No (only one globally) |
| Must be on top to reuse? | ❌ No |
| `onNewIntent()` called? | ✅ Yes |
| Own task? | ✅ Yes — **exclusively**, no other activity can join |

**Behavior:** Like `singleTask`, but even more isolated. The activity lives in its **own dedicated task** and **no other activity** can be launched into that task. Any activity started from a `singleInstance` activity is placed in a **different task**.

```
App Task: [ A | B ]      Singleton Task: [ D ]
                ↓ start D (singleInstance)
App Task: [ A | B ]      Singleton Task: [ D ]  ← D gets its own task

Start E from D:
App Task: [ A | B | E ]  Singleton Task: [ D ]  ← E is in the app task, not D's task
```

**Use case:** System-level screens that must be completely isolated — a dialer, in-app browser, camera, or any screen shared across apps via an implicit intent.

> ⚠️ **Note:** Pressing **Back** from a `singleInstance` activity may feel unintuitive to users since the task boundary is hidden from them.

---

### 5. `singleInstancePerTask` (API 31+)

| Property | Value |
|---|---|
| Multiple instances? | ✅ Yes (one per task) |
| Must be on top to reuse? | ❌ No |
| `onNewIntent()` called? | ✅ Yes (when reused in same task) |
| Own task? | ✅ Yes (as root), but other activities CAN join |

**Behavior:** Introduced in **Android 12 (API 31)**. Only **one instance per task** is allowed. Unlike `singleInstance`, other activities *can* be launched into the same task. If the activity already exists as the root of a task, launching it again clears activities above it and calls `onNewIntent()`. Multiple instances can exist, but each must reside in a **different task**.

```xml
<activity
    android:name=".FlowRootActivity"
    android:launchMode="singleInstancePerTask" />
```

**Use case:** Root activity of a flow that can be launched into multiple tasks simultaneously (e.g., multi-window scenarios, split-screen flows).

---

## 📊 Launch Mode Comparison

| Launch Mode | New Instance? | `onNewIntent()`? | Own Task? | Multiple Instances? |
|---|---|---|---|---|
| `standard` | Always | ❌ | ❌ | ✅ |
| `singleTop` | Only if not on top | ✅ (when reused) | ❌ | ✅ |
| `singleTask` | Only if none exists | ✅ (when reused) | ✅ | ❌ |
| `singleInstance` | Only if none exists | ✅ (when reused) | ✅ (exclusive) | ❌ |
| `singleInstancePerTask` | One per task | ✅ (when reused) | ✅ (non-exclusive) | ✅ (one per task) |

---

## 🏷️ Common Intent Flags

Intent flags let you dynamically control launch behavior at the call site, complementing (or overriding) manifest-declared launch modes.

| Flag | Behavior |
|---|---|
| `FLAG_ACTIVITY_NEW_TASK` | Equivalent to `singleTask`. Launches in a new task. Required when starting activities from non-Activity contexts (e.g., `Service`, `BroadcastReceiver`). |
| `FLAG_ACTIVITY_SINGLE_TOP` | Equivalent to `singleTop`. Reuses the activity if it's on top. |
| `FLAG_ACTIVITY_CLEAR_TOP` | If the target activity exists in the task, all activities above it are destroyed. Often paired with `FLAG_ACTIVITY_NEW_TASK`. |
| `FLAG_ACTIVITY_CLEAR_TASK` | Clears the entire existing task before launching. Must be combined with `FLAG_ACTIVITY_NEW_TASK`. Useful for post-logout navigation. |
| `FLAG_ACTIVITY_NO_HISTORY` | The activity is not kept in the back stack. It is finished as soon as the user leaves it. |
| `FLAG_ACTIVITY_REORDER_TO_FRONT` | If the activity already exists in the task, it is moved to the top of the stack without clearing activities above it. |

### Common Combination: Logout & Clear Task

```kotlin
// Navigate to LoginActivity after logout, clearing all previous screens
val intent = Intent(this, LoginActivity::class.java).apply {
    flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
}
startActivity(intent)
```

### Combination: Deep Link to Existing Screen

```kotlin
// If HomeActivity exists in the stack, bring it to top and clear activities above it
val intent = Intent(this, HomeActivity::class.java).apply {
    flags = Intent.FLAG_ACTIVITY_CLEAR_TOP or Intent.FLAG_ACTIVITY_SINGLE_TOP
}
startActivity(intent)
```

> ✅ **Rule of thumb:** Intent flags take **precedence** over manifest-declared `launchMode` when both are specified and they conflict.

---

## 💡 Quick-Reference: Which Mode to Use?

| Scenario | Recommended Mode |
|---|---|
| General-purpose screen | `standard` |
| Prevent back-stack spam from notifications | `singleTop` |
| App home / dashboard that resets navigation | `singleTask` |
| Fully isolated screen shared across apps | `singleInstance` |
| Flow root that supports multi-window | `singleInstancePerTask` |
| Clear everything and restart (logout) | `FLAG_ACTIVITY_NEW_TASK \| FLAG_ACTIVITY_CLEAR_TASK` |
