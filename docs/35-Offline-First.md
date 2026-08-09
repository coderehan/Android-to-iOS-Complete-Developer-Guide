# 35 - Offline First: Sync Patterns on Android vs iOS

> "Offline-first" isn't a single API — it's an architecture pattern: local database as the single source of truth, with a background sync layer reconciling with the server. The pattern maps almost directly between platforms; only the underlying storage/sync primitives differ.

---

## 🔑 Core Pattern (Same on Both Platforms)

```
UI reads from local DB (always available, instant)
        ↑
Local DB (Room / SwiftData) ← single source of truth
        ↑
Sync layer (WorkManager / BGTaskScheduler) reconciles with server
        ↑
Remote API (Retrofit / Alamofire)
```

Neither platform's UI ever talks to the network directly in a well-built offline-first app — it always reads from the local database, which the sync layer keeps updated opportunistically.

---

## 🗄 Local Source of Truth

**Android (Room + Repository)**
```kotlin
class TaskRepository(private val dao: TaskDao, private val api: ApiService) {
    fun getTasks(): Flow<List<Task>> = dao.getAllTasksFlow()

    suspend fun refresh() {
        try {
            val remote = api.getTasks()
            dao.insertAll(remote)
        } catch (e: IOException) {
            // offline — UI still has last-synced local data via the Flow above
        }
    }
}
```

**iOS (SwiftData + Repository)**
```swift
class TaskRepository {
    let modelContext: ModelContext
    let api: APIService

    func getTasks() -> [Task] {
        // @Query in the View handles reactive local reads directly
        try! modelContext.fetch(FetchDescriptor<Task>())
    }

    func refresh() async {
        do {
            let remote = try await api.getTasks()
            remote.forEach { modelContext.insert($0) }
            try? modelContext.save()
        } catch {
            // offline — @Query-backed views still show last-synced local data
        }
    }
}
```

Both patterns: attempt a refresh, silently fall back to local data on failure, and let the UI layer's reactive read (Flow/`collectAsState` or `@Query`) naturally reflect whatever the local DB currently holds.

---

## ✍️ Queuing Writes Made While Offline

**Android (WorkManager for guaranteed eventual sync)**
```kotlin
// Mark locally, queue a sync job
dao.markPendingSync(taskId)

val request = OneTimeWorkRequestBuilder<SyncTaskWorker>()
    .setConstraints(Constraints.Builder().setRequiredNetworkType(NetworkType.CONNECTED).build())
    .build()

WorkManager.getInstance(context).enqueue(request)
```

**iOS (local flag + BGTaskScheduler, less guaranteed — see note below)**
```swift
task.pendingSync = true
try? modelContext.save()

let request = BGProcessingTaskRequest(identifier: "com.example.app.sync")
request.requiresNetworkConnectivity = true
try? BGTaskScheduler.shared.submit(request)
```

> ⚠️ Reminder from [23 - Background Tasks](./23-Background-Tasks.md): iOS's `BGTaskScheduler` is best-effort, not guaranteed. For critical writes that must sync reliably, many iOS apps **also attempt an immediate foreground sync** the moment connectivity returns (via `NWPathMonitor`), rather than relying solely on the background scheduler — a pattern less necessary on Android given WorkManager's stronger guarantees.

---

## 📶 Network Reachability Monitoring

**Android**
```kotlin
val connectivityManager = context.getSystemService(ConnectivityManager::class.java)
connectivityManager.registerDefaultNetworkCallback(object : ConnectivityManager.NetworkCallback() {
    override fun onAvailable(network: Network) {
        // trigger sync
    }
})
```

**iOS**
```swift
import Network

let monitor = NWPathMonitor()
monitor.pathUpdateHandler = { path in
    if path.status == .satisfied {
        // trigger sync
    }
}
monitor.start(queue: DispatchQueue.global())
```

`NWPathMonitor` is iOS's direct equivalent of Android's `ConnectivityManager` network callback — both let you react the moment the device regains connectivity to kick off a sync.

---

## ⚔️ Conflict Resolution

Both platforms need the same strategy decisions, independent of framework:
- **Last-write-wins** (simplest, uses a timestamp)
- **Server-wins** (client always defers to server state on conflict)
- **Merge/CRDT-based** (for collaborative data — more complex, framework-agnostic)

Neither Room/WorkManager nor SwiftData/BGTaskScheduler provide conflict resolution out of the box — this logic lives in your repository/sync layer on both platforms equally.

---

## 📝 Quick Reference Table

| Concept | Android | iOS |
|---|---|---|
| Local source of truth | Room | SwiftData |
| Reactive local read | `Flow` + `collectAsState()` | `@Query` |
| Background sync trigger | WorkManager (reliable) | BGTaskScheduler (best-effort) |
| Reachability monitoring | `ConnectivityManager` | `NWPathMonitor` |
| Guaranteed write sync | ✅ Via WorkManager | ⚠️ Needs foreground fallback too |
| Conflict resolution | App-level, not framework-provided | App-level, not framework-provided |

---
