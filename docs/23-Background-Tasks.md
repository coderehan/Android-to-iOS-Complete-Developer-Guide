# 23 - Background Tasks: WorkManager vs BGTaskScheduler

> WorkManager guarantees eventual execution with a flexible constraint system. iOS's background execution is far more restricted by the OS — the system decides *if and when* your background task actually runs, prioritizing battery life over app reliability.

---

## 🔑 Core Philosophy

| WorkManager | BGTaskScheduler |
|---|---|
| Guarantees eventual execution (survives reboot, app kill) | System-scheduled, **not guaranteed** — OS decides based on usage patterns, battery, etc. |
| One-time & periodic work | `BGAppRefreshTask` (short refresh) & `BGProcessingTask` (longer, e.g. charging + WiFi) |
| Constraints: network, charging, storage | Constraints: network, charging (via `BGProcessingTaskRequest`) |
| Runs even if app is killed | Runs only if OS decides to wake the app briefly |

---

## 🛠 Defining Background Work

**Android (WorkManager)**
```kotlin
class SyncWorker(context: Context, params: WorkerParameters) : CoroutineWorker(context, params) {
    override suspend fun doWork(): Result {
        syncData()
        return Result.success()
    }
}

val request = PeriodicWorkRequestBuilder<SyncWorker>(1, TimeUnit.HOURS)
    .setConstraints(
        Constraints.Builder().setRequiredNetworkType(NetworkType.CONNECTED).build()
    )
    .build()

WorkManager.getInstance(context).enqueue(request)
```

**iOS (BGTaskScheduler)**
```swift
// Register (in AppDelegate/App init, before app finishes launching)
BGTaskScheduler.shared.register(forTaskWithIdentifier: "com.example.app.sync", using: nil) { task in
    handleSync(task: task as! BGAppRefreshTask)
}

// Schedule
func scheduleSync() {
    let request = BGAppRefreshTaskRequest(identifier: "com.example.app.sync")
    request.earliestBeginDate = Date(timeIntervalSinceNow: 60 * 60) // hint only
    try? BGTaskScheduler.shared.submit(request)
}

func handleSync(task: BGAppRefreshTask) {
    task.expirationHandler = { task.setTaskCompleted(success: false) }
    Task {
        await syncData()
        task.setTaskCompleted(success: true)
        scheduleSync() // must re-schedule after every run
    }
}
```

> ⚠️ Critical difference: `earliestBeginDate` is only a **hint**, not a guarantee. iOS may run the task much later, or skip it entirely if the app isn't used often enough, battery is low, or Low Power Mode is active. WorkManager's periodic work is far more reliable — this is a genuine platform limitation, not a missing API you can work around.

---

## 📋 Task Types

| WorkManager | BGTaskScheduler |
|---|---|
| `OneTimeWorkRequest` | Single `BGAppRefreshTaskRequest` submission |
| `PeriodicWorkRequest` | Re-submit `BGAppRefreshTaskRequest` after each run |
| Expedited work (`setExpedited`) | `BGProcessingTaskRequest` (longer-running, e.g. large syncs, ML processing) |

**BGProcessingTaskRequest (heavier tasks — like a WorkManager job with charging constraint)**
```swift
let request = BGProcessingTaskRequest(identifier: "com.example.app.heavysync")
request.requiresNetworkConnectivity = true
request.requiresExternalPower = true
try? BGTaskScheduler.shared.submit(request)
```

---

## 🧾 Required Setup (Info.plist + Capabilities)

Unlike WorkManager (just a Gradle dependency), BGTaskScheduler requires explicit declaration:

```xml
<!-- Info.plist -->
<key>BGTaskSchedulerPermittedIdentifiers</key>
<array>
    <string>com.example.app.sync</string>
    <string>com.example.app.heavysync</string>
</array>
```

Plus enabling **"Background Modes" → "Background fetch" / "Background processing"** capability in Xcode's project settings.

---

## 🧪 Testing Background Tasks

**Android** — WorkManager work can be triggered instantly via `WorkManager.getInstance(context).enqueue()` in debug builds, or inspected via `adb shell dumpsys jobscheduler`.

**iOS** — You can force-trigger a registered background task in the Xcode debugger console while the app is paused at a breakpoint:
```
e -l objc -- (void)[[BGTaskScheduler sharedScheduler] _simulateLaunchForTaskWithIdentifier:@"com.example.app.sync"]
```

---

## 📝 Quick Reference Table

| Concept | WorkManager (Android) | BGTaskScheduler (iOS) |
|---|---|---|
| Guarantee of execution | ✅ Reliable | ❌ Best-effort, OS-controlled |
| One-time task | `OneTimeWorkRequest` | `BGAppRefreshTaskRequest` (single submit) |
| Recurring task | `PeriodicWorkRequest` | Re-submit after every completion |
| Heavy/long task | Expedited work | `BGProcessingTaskRequest` |
| Constraints | `Constraints.Builder()` | `requiresNetworkConnectivity`, `requiresExternalPower` |
| Setup required | Gradle dependency only | `Info.plist` array + Xcode capability |
| Survives app kill | ✅ Yes | ⚠️ Only if OS chooses to wake app |

---
