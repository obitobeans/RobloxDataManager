# RobloxDataManager — Safe DataStore Wrapper

![Luau](https://img.shields.io/badge/Luau-!strict-blue)
![Roblox](https://img.shields.io/badge/Context-Server-orange)
![Status](https://img.shields.io/badge/Status-Portfolio_Reference-yellow)

A portfolio-focused ModuleScript that wraps Roblox `DataStoreService` to reduce throttling, avoid overwrites, and prevent common duplication/data-loss cases.  
Instead of every script talking to DataStores directly(throttling, overwrites, race conditions, scripts talk to **`DataManager`**, which handles caching, batching, locking, retries, and shutdown safety.

---

## Important Notice (Portfolio / Reference Use)

**This repository is primarily provided as a portfolio project and reference implementation.**  
You are welcome to use it to learn how to build a DataStore wrapper, but you should **not** treat it as a “drop-in guaranteed production solution” without:

- Reviewing the code against your game’s requirements (economy, exploit model, schema size, GDPR/privacy needs)
- Load testing with realistic player counts
- Adjusting autosave intervals and lock timings for your game
- Adding analytics/telemetry and alerting for DataStore failures

**Use at your own risk.** DataStores can throttle, fail temporarily, and behave differently under live load. Always design with backups, monitoring, and graceful failure.

---

## Features

### 1) Session Locking (Anti-Duplication)
Prevents the classic “Server A overwrites Server B” duplication bug:

- On load, the record is locked with:
  - `JobId` (server ID)
  - `Timestamp` (unix time)
- If another server owns the lock:
  - If the lock is **recent (< 2 minutes)** → player is kicked: `"Save in progress, please rejoin"`
  - If the lock is **stale (> 5 minutes)** → lock is stolen (assume crash)

### 2) UpdateAsync-Only Commits
All writes use **`UpdateAsync`** (no `SetAsync`) to reduce the chance of overwriting new data with stale data.

### 3) Data Reconciling (Schema / Template Handling)
Loaded data is reconciled against a `DefaultData` template (recursive), filling missing keys so old save files still work after new stats are added.

### 4) Cache Pattern + Autosave
- Gameplay updates write **only to RAM** (fast, instant)
- DataStore writes happen on an autosave timer (batched)
- Prevents save spam and respects DataStore throttling limits

### 5) BindToClose Shutdown Handler
Ensures data is saved during server shutdowns/restarts (when `PlayerRemoving` may not fire in time).

---
