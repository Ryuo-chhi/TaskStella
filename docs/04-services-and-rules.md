# 04 Services and Business Rules

Services hold the business logic. They are plain Dart classes with no Flutter imports, so every operation can be unit tested. The course asks for **at least 2 services**, **rules beyond CRUD**, and **at least 1 operation involving several entities**. TaskStella has five services. The operations marked **multi-entity** are the ones to show in interviews.

Every operation below lists its rules and its invalid cases. Each row becomes at least one test (see [05-testing.md](05-testing.md)).

## Overview

| Service | Owns | Depends on |
|---|---|---|
| TaskService | Tasks | TaskRepository, WalletService |
| AssignmentService | Courses, assignments, subtasks | AssignmentRepository, CourseRepository, WalletService |
| FocusService | Focus sessions | SessionRepository, WalletService, Inventory (for sound ownership) |
| WalletService | Balance and coin history | WalletRepository |
| ShopService | Purchases and equipping | Catalog, InventoryRepository, WalletService |

WalletService is the only class allowed to change the balance. Other services ask it to award or spend coins.

## Coin rules

Decided 2026-10-30. Keep these values in one constants place so tests and code use the same numbers.

| Activity | Coins | Notes |
|---|---|---|
| Complete a task | 5 | Once per task, ever |
| Complete a subtask | 2 | Once per subtask, ever |
| Complete an assignment | 10 | Once per assignment, only when the user marks it complete |
| Complete a focus session | 1 per planned minute | 25 min session = 25 coins. Full reward only when planned time is reached. |
| Finish a session early | 1 per 2 full minutes focused, rounded down | 20 of 25 min = 10 coins. Under 5 min focused = 0. |
| Cancel a session | 0 | |
| Daily task reward cap | 10 rewarded tasks per day | The 11th task still completes, it just earns 0. Stops farming coins with empty tasks. |

### Why these numbers

- **Focus is the main earner.** It is the behavior the app most wants to build, so one normal 25 minute session (25 coins) is worth five tasks.
- **Tasks and subtasks pay a little.** They reward planning without making it faster to create fake tasks than to study. The daily cap backs this up.
- **Finishing early still pays something, at half the rate.** Stopping early is better than cancelling, but never better than finishing.
- **The 5 minute minimum** stops start-and-stop farming of early-finish rewards.

### Shop prices (to match the earning rate)

A student doing one session and a few tasks a day earns about 40 to 50 coins.

| Item type | Price range | Earned in about |
|---|---|---|
| Sound | 60 to 100 | 2 days |
| Chibi | 150 to 250 | 4 to 5 days |
| Theme background | 200 to 350 | 5 to 7 days |
| Starter items | 0 | Always owned |

Set each item's exact price in the catalog (see [08-assets.md](08-assets.md)).

## TaskService

| Operation | Rules | Invalid cases |
|---|---|---|
| createTask(title, priority, dueDate?, courseId?) | Trims the title. Saves a new task, not completed. | Empty or whitespace title. Unknown courseId. |
| updateTask(id, changes) | Same title rules. Keeps `isCompleted` and `rewardClaimed`. | Unknown id. Empty title. |
| deleteTask(id) | Removes the task. Coins already earned are kept. | Unknown id. |
| completeTask(id, now) **multi-entity** | Sets completed. If `rewardClaimed` is false and fewer than 10 tasks were rewarded today, awards 5 coins through WalletService and sets `rewardClaimed` true. Over the cap, the task completes with 0 coins and `rewardClaimed` is still set to true. | Unknown id. Already completed: no change, no coins. |
| uncompleteTask(id) | Sets not completed. Does not remove coins and does not reset `rewardClaimed`. | Unknown id. |
| tasksForDate(date), tasksByFilter(filter) | Today, high priority, completed. | |

Key test: complete, uncomplete, complete again gives coins **once**.

## AssignmentService

| Operation | Rules | Invalid cases |
|---|---|---|
| createCourse(name, colorKey) | | Empty name. Duplicate name (decide if allowed). |
| createAssignment(...) | Must belong to an existing course. | Empty title. Unknown course. Workload of 0 or less. |
| updateAssignment(id, changes) | Keeps every existing subtask and its completed state. | Unknown id. Empty title. Unknown course. |
| deleteAssignment(id) | Removes it and its subtasks. | Unknown id. |
| addSubtask(assignmentId, title) | | Empty title. Unknown assignment. |
| toggleSubtask(assignmentId, subtaskId) **multi-entity** | Flips completed. Awards coins the first time only. Never completes the assignment. | Unknown ids. |
| allSubtasksDone(assignmentId) | True when there is at least one subtask and all are done. The UI uses this to ask "Mark assignment complete?". | |
| completeAssignment(id) **multi-entity** | Only by explicit user action. Awards 10 coins the first time only. | Unknown id. Already completed: no change, no coins. |
| statusOf(assignment, now) | upcoming, overdue, completed (see domain model). | |
| assignmentsByStatus(status, now) | | |

## FocusService

Session states:

```
            start
  (none) ─────────> running ──pause──> paused
                      │  ^              │
                      │  └───resume─────┘
          time ends / │
          finish early│            cancel (from running or paused)
                      v                 v
                  completed         cancelled
```

| Operation | Rules | Invalid cases |
|---|---|---|
| startSession(plannedMinutes, target?, soundId?) | Creates a running session. | Minutes 0 or negative. Sound not owned. Another session already running. |
| pause(id) | Running to paused. Paused time does not count. | Not running. |
| resume(id) | Paused to running. | Not paused. |
| completeSession(id, actualSeconds) **multi-entity** | Running or paused to completed. Records duration. Awards the full reward if planned time was reached, a reduced reward if finished early. Rewards once. | Already completed or cancelled. |
| cancelSession(id) | To cancelled. No full reward (0 coins under the default rule). | Already ended. |

Keep the timer itself (ticking every second) in the Focus Timer screen. The service only receives the elapsed time, which keeps it testable without waiting in real time.

## WalletService

| Operation | Rules | Invalid cases |
|---|---|---|
| balance | Current coins. | |
| award(amount, type, sourceId) | Adds coins and records a transaction. Rejects a second reward for the same type and sourceId. | Amount 0 or negative. Duplicate source. |
| spend(amount, sourceId) | Subtracts and records. | Amount greater than balance: rejected, balance unchanged. Amount 0 or negative. |
| history() | Transactions, newest first. | |

## ShopService

| Operation | Rules | Invalid cases |
|---|---|---|
| itemsByType(type) | From the catalog, with state for each item: buyable, cannot afford, owned, equipped. | |
| purchase(itemId) **multi-entity** | Checks ownership and balance, spends coins through WalletService, adds the item to the inventory. Either everything happens or nothing does. | Unknown item. Already owned. Not enough coins: nothing deducted, item not added. |
| equip(itemId) | Theme: replaces equipped theme. Chibi: replaces equipped chibi. Sound: sets the default sound. | Item not owned. |
| isOwned(itemId) | Starter items are always owned. | |

## Rules summary (from the proposal)

| Rule | Where enforced |
|---|---|
| Blank or whitespace titles rejected | TaskService, AssignmentService |
| A completion never rewards twice | `rewardClaimed` flag plus WalletService duplicate check |
| All subtasks done does not complete the assignment | AssignmentService.toggleSubtask |
| Editing an assignment keeps completed subtasks | AssignmentService.updateAssignment |
| Focus duration must be greater than 0 | FocusService.startSession |
| Cancelled sessions do not get the full reward | FocusService.cancelSession and completeSession |
| Not enough coins: rejected, nothing deducted | WalletService.spend, ShopService.purchase |
| Balance never negative | WalletService |
| Owned items cannot be bought twice | ShopService.purchase |
| One theme and one chibi equipped | ShopService.equip |
| Starter items free and always owned | Catalog and Inventory |
