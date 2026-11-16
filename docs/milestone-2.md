# CADT IDT — Mobile Development Term 1 (Gen 11)
# Project Milestone 2: Model, Service & Test Layers

**Project Name:** TaskStella  
**Team:** 12  
**Members & Work Allocation:**
- **Tiek Chhunhour (Member A):** `Task`, `Assignment`, `Subtask` models; `TaskService`, `AssignmentService`; `task_service_test.dart`, `assignment_service_test.dart`.
- **Yeat David (Member B):** `FocusSession`, `Wallet`, `CoinTransaction`, `ShopItem`, `Inventory` models; `FocusService`, `WalletService`, `ShopService`; `focus_service_test.dart`, `wallet_service_test.dart`, `shop_service_test.dart`.

---

## 1. Architecture

TaskStella follows a strict 3-tier domain-driven layered architecture to ensure separation of concerns, maintainability, and 100% testability without any Flutter widget or UI dependencies in the core business logic.

### 1.1 Directory Structure
The test suite mirrors the `lib/` package structure directly under `test/service/`, and all domain enumerations (`TaskStatus`, `AssignmentStatus`, `SessionStatus`, `ShopItemType`, `Priority`, `TransactionType`) have their own standalone files:

```text
task_stella/
├── lib/
│   ├── model/
│   │   ├── priority.dart
│   │   ├── task_status.dart
│   │   ├── assignment_status.dart
│   │   ├── session_status.dart
│   │   ├── transaction_type.dart
│   │   ├── shop_item_type.dart
│   │   ├── task.dart
│   │   ├── assignment.dart
│   │   ├── subtask.dart
│   │   ├── focus_session.dart
│   │   ├── wallet.dart
│   │   ├── coin_transaction.dart
│   │   ├── shop_item.dart
│   │   └── inventory.dart
│   └── service/
│       ├── task_service.dart
│       ├── assignment_service.dart
│       ├── focus_service.dart
│       ├── wallet_service.dart
│       └── shop_service.dart
└── test/
    └── service/
        ├── task_service_test.dart
        ├── assignment_service_test.dart
        ├── focus_service_test.dart
        ├── wallet_service_test.dart
        └── shop_service_test.dart
```

### 1.2 Responsibilities of Each Layer
- **`lib/model/` (Domain Model Layer):** Pure business entities and value objects. Classes encapsulate strong typing, immutability (`copyWith`), precondition assertions, and domain logic helpers. Zero external or UI dependencies.
- **`lib/service/` (Business Logic & Service Layer):** Implements all business operations, multi-entity workflows, anti-farming validations, coin balance rules, and state machines. Services interact via dependency injection.
- **`test/service/` (Verification & Test Layer):** Automated unit tests executing in a pure Dart harness, mirroring `lib/service/`. Covers normal, invalid, and boundary conditions for every service operation.

---

## 2. Model Layer

### 2.1 Domain Entities Overview
TaskStella models 8 core business domain entities directly mapped to the 4 user stories (US-01 through US-04). In strict accordance with the Milestone 1 project overview (`README.md`), `Course` is deferred to Milestone 3 as a future enhancement so students can track work immediately without setup friction:

1. **`Task`** (US-01): Standalone daily task with title, priority, status lifecycle (`todo`, `inProgress`, `done`), and optional due date.
2. **`Assignment`** (US-02): Major academic coursework with title, description, due date, priority, status lifecycle, and decomposed subtasks.
3. **`Subtask`** (US-02): Actionable mini-step composing an assignment with independent completion tracking.
4. **`FocusSession`** (US-03): Focus Mode session tracking 25-minute study intervals, 5-minute break intervals, session states (`running`, `paused`, `completed`, `cancelled`), and session coin reward claims (`rewardClaimed`).
5. **`Wallet`** (US-04): Centralized coin balance container guaranteeing non-negative currency balance (≥ 0).
6. **`CoinTransaction`** (US-04): Immutable audit ledger entry recording coin credits and debits with timestamps and source references.
7. **`ShopItem`** (US-04): Catalog item available in the reward shop (themes, soundscapes, companion avatars) with pricing and asset references.
8. **`Inventory`** (US-04): User's owned items container tracking unlocked items and active equipped customizations.

### 2.2 Scope Alignment with Milestone 1
- **Direct Alignment with README.md:** In the Milestone 1 overview, Courses are designated under *Future Enhancements (Optional Scope)*. Removing Course from Milestone 2 keeps the scope focused, clean, and frictionless for students.
- **Rubric Compliance via Core Composition:** The grading rubric requirement for Composition and Aggregation is completely satisfied by the core entities:
  - **Composition 1:** `Assignment ──* Subtask` (an Assignment directly owns its Subtasks; deleting an assignment cascade-deletes all child subtasks).
  - **Composition 2:** `Wallet ──* CoinTransaction` (the Wallet directly owns its immutable transaction audit trail).
  - **Aggregation 1:** `Inventory ──* ShopItem` (the Inventory references owned catalog item IDs, while items exist independently in the catalog).
  - **Aggregation 2:** `FocusSession ── Task / Assignment` (focus sessions optionally reference a target task or assignment).

### 2.3 Composition and Aggregation Relationships
1. **Composition: `Assignment` ──\* `Subtask`**  
   An Assignment directly owns a collection of Subtasks (`List<Subtask>`). Subtasks have no independent lifecycle outside their parent assignment; deleting an assignment cascade-deletes all its subtasks.
2. **Composition: `Wallet` ──\* `CoinTransaction`**  
   The Wallet encapsulates its immutable transaction history (`List<CoinTransaction>`). Balance mutations create permanent audit records owned by the wallet.
3. **Aggregation: `Inventory` ──\* `ShopItem`**  
   The Inventory references owned catalog item IDs (`Set<String> ownedItemIds`). Shop items exist independently in the static catalog; deleting or unequipping an item does not delete it from the shop catalog.
4. **Aggregation / Association: `FocusSession` ── `Task` / `Assignment`**  
   A focus session optionally associates with a target task or assignment for tracking. The target entity continues to exist independently regardless of session outcome.

### 2.4 Entity Relationship Hierarchy Diagram
```text
TaskStella Domain Model
├── Assignment (Composition Root)
│   └── List<Subtask> (Composition - cascade delete)
├── Task
├── FocusSession
│   └── (Optional Task / Assignment reference by targetId)
├── Wallet
│   └── List<CoinTransaction> (Composition - audit ledger)
└── Inventory
    └── Set<String> ownedItemIds (Aggregation - catalog references)
```

### 2.5 Entity Specifications & Key Methods

#### A. `Task` (Author: Tiek Chhunhour)
- **Attributes:** `id: String`, `title: String`, `priority: Priority` (default: `medium`), `dueDate: DateTime?`, `status: TaskStatus` (default: `todo`), `createdAt: DateTime`.
- **Key Methods:**
  - `Task({...})` with assertion: `assert(title.trim().isNotEmpty, 'Task title cannot be empty')`
  - `Task copyWith({...})` for immutable state updates
  - `bool get isCompleted => status == TaskStatus.done`
  - `bool get isOverdue => dueDate != null && !isCompleted && DateTime.now().isAfter(dueDate!)`

#### B. `Assignment` (Author: Tiek Chhunhour)
- **Attributes:** `id: String`, `title: String`, `description: String?`, `dueDate: DateTime`, `priority: Priority` (default: `medium`), `status: TaskStatus` (default: `todo`), `subtasks: List<Subtask>` (default: `[]`), `createdAt: DateTime`.
- **Key Methods:**
  - `Assignment({...})` with assertion: `assert(title.trim().isNotEmpty, 'Assignment title cannot be empty')`
  - `Assignment copyWith({...})`
  - `bool get isCompleted => status == TaskStatus.done`
  - `double get progressRatio`: Returns ratio of completed subtasks (`0.0` if subtasks empty)
  - `bool get areAllSubtasksComplete`: `subtasks.isNotEmpty && subtasks.every((s) => s.isCompleted)`
  - `AssignmentStatus getStatus(DateTime now)`: Computes `completed` if `isCompleted`, `overdue` if `!isCompleted && now.isAfter(dueDate)`, `inProgress` if `status == TaskStatus.inProgress || progressRatio > 0`, else `upcoming`.

#### C. `Subtask` (Author: Tiek Chhunhour)
- **Attributes:** `id: String`, `title: String`, `status: TaskStatus` (default: `todo`).
- **Key Methods:**
  - `Subtask({...})` with assertion: `assert(title.trim().isNotEmpty, 'Subtask title cannot be empty')`
  - `Subtask copyWith({...})`
  - `bool get isCompleted => status == TaskStatus.done`

#### D. `FocusSession` (Author: Yeat David)
- **Attributes:** `id: String`, `sessionIndex: int` (default: `1`, e.g., session 1 to 4 in a 4-session focus cycle), `isBreak: bool` (default: `false`, differentiates 25-minute focus work from 5-minute rest breaks), `plannedMinutes: int` (default: `25` for focus, `5` for break), `actualSeconds: int` (default: `0`), `status: SessionStatus` (default: `running`), `rewardClaimed: bool` (default: `false`), `targetTaskId: String?`, `targetAssignmentId: String?`, `soundId: String?`, `startedAt: DateTime`, `endedAt: DateTime?`, `coinsAwarded: int` (default: `0`).
- **Key Methods:**
  - `FocusSession({...})` with assertion: `assert(plannedMinutes > 0, 'Planned minutes must be greater than zero')`
  - `FocusSession copyWith({...})`
  - `bool get isCompleted => status == SessionStatus.completed`
  - `bool get canClaimReward => isCompleted && !isBreak && !rewardClaimed`

#### E. `Wallet` & `CoinTransaction` (Author: Yeat David)
- **Attributes (`Wallet`):** `balance: int` (≥ 0), `history: List<CoinTransaction>`.
- **Attributes (`CoinTransaction`):** `id: String`, `amount: int`, `type: TransactionType` (`focusReward`, `purchase`), `sourceId: String`, `reason: String`, `createdAt: DateTime`.
- **Key Methods (`Wallet`):**
  - `Wallet({...})` with assertion: `assert(balance >= 0, 'Balance cannot be negative')`
  - `Wallet copyWith({...})`

#### F. `ShopItem` & `Inventory` (Author: Yeat David)
- **Attributes (`ShopItem`):** `id: String`, `name: String`, `type: ShopItemType` (`theme`, `sound`, `companionAvatar`), `price: int` (≥ 0), `assetPath: String`, `isStarter: bool` (default: `false`).
- **Attributes (`Inventory`):** `ownedItemIds: Set<String>`, `equippedThemeId: String`, `equippedAvatarId: String`, `defaultSoundId: String`.
- **Key Methods:**
  - `ShopItem({...})` with assertions: `assert(name.trim().isNotEmpty)`, `assert(price >= 0, 'Price must be non-negative')`
  - `bool Inventory.isOwned(String itemId)`: Returns true if item is starter item or contained in `ownedItemIds`.
  - `Inventory copyWith({...})`

---

## 3. Service Layer

The Service Layer encapsulates all business rules, input validations, state transitions, and multi-entity orchestrations across 5 dedicated services.

### 3.1 Authorship & Service Allocation

| Service | Owner / Author | Responsibility & Managed Scope |
|---|---|---|
| **`TaskService`** | **Tiek Chhunhour** | Task lifecycle, filtering, completion status updates (`todo` ↔ `done`). Tasks do not award coins. |
| **`AssignmentService`** | **Tiek Chhunhour** | Assignment lifecycle, subtask decomposition & cascade deletion, subtask toggling, explicit assignment submission. Assignments do not award coins. |
| **`FocusService`** | **Yeat David** | Focus cycle management (4 sessions: 25-min focus, 5-min break), timer state machine (`running` → `paused` → `completed` / `cancelled`), session reward claiming (`rewardClaimed` lock), sound validation. |
| **`WalletService`** | **Yeat David** | Sole authority for balance mutations, non-negative assertions (≥ 0), and immutable audit history. |
| **`ShopService`** | **Yeat David** | Catalog querying, purchase validation (balance check, duplicate ownership prevention), item equipping. |

---

### 3.2 Complete Listing of All Service Operations

#### 1. `TaskService` (Author: Tiek Chhunhour)

| Operation | Parameters & Return | Rules & Preconditions | Invalid Cases / Errors | Multi-Entity? |
|---|---|---|---|---|
| `createTask` | `(title, priority, dueDate?) -> Task` | Trims whitespace. Creates new task with `status: TaskStatus.todo`. | Throws `ArgumentError` if title is empty or whitespace only. | No |
| `updateTask` | `(id, changes) -> Task` | Updates fields via `copyWith`. Preserves status and created timestamp. | Throws `ArgumentError` if task ID not found or updated title is empty/whitespace. | No |
| `deleteTask` | `(id) -> void` | Removes task from storage. | Throws `ArgumentError` if task ID not found. | No |
| `completeTask` | `(id) -> Task` | Transitions `status` to `TaskStatus.done`. Does not award coins (tasks do not claim rewards). | Throws `ArgumentError` if task ID not found. | No |
| `uncompleteTask` | `(id) -> Task` | Reverts `status` to `TaskStatus.todo`. | Throws `ArgumentError` if task ID not found. | No |
| `tasksForDate` | `(DateTime date) -> List<Task>` | Filters tasks whose `dueDate` matches the specified calendar date. | None (returns empty list if none match). | No |
| `tasksByFilter` | `({Priority? priority, TaskStatus? status}) -> List<Task>` | Returns filtered view of tasks matching provided criteria. | None. | No |

#### 2. `AssignmentService` (Author: Tiek Chhunhour)

| Operation | Parameters & Return | Rules & Preconditions | Invalid Cases / Errors | Multi-Entity? |
|---|---|---|---|---|
| `createAssignment` | `(title, dueDate, priority, {description}) -> Assignment` | Creates assignment with `status: TaskStatus.todo`. | Throws `ArgumentError` if title is empty or whitespace only. | No |
| `updateAssignment` | `(id, changes) -> Assignment` | Updates assignment fields while strictly preserving existing subtasks and their completed states. | Throws `ArgumentError` if assignment ID not found or title empty. | No |
| `deleteAssignment` | `(id) -> void` | **Composition Rule:** Deleting an assignment cascade-deletes the assignment and all of its nested child subtasks simultaneously. | Throws `ArgumentError` if assignment ID not found. | **Yes** (`Assignment`, `Subtask`) |
| `addSubtask` | `(assignmentId, title) -> Subtask` | Appends a new subtask (`status: TaskStatus.todo`) to the assignment's subtask list. | Throws `ArgumentError` if assignment ID not found or subtask title is empty. | No |
| `toggleSubtask` | `(assignmentId, subtaskId) -> Subtask` | Toggles subtask status between `todo` and `done`. Does not award coins (subtasks do not claim rewards). Completing all subtasks does **not** auto-complete the assignment. | Throws `ArgumentError` if assignment or subtask ID not found. | No |
| `allSubtasksDone`| `(assignmentId) -> bool` | Returns true if assignment contains at least one subtask and all are `done`. Returns false if subtasks list is empty. | Throws `ArgumentError` if assignment ID not found. | No |
| `completeAssignment`| `(assignmentId) -> Assignment` | Explicit student submission. Transitions assignment `status` to `TaskStatus.done`. Does not award coins (assignments do not claim rewards). | Throws `ArgumentError` if assignment ID not found. | No |
| `statusOf` | `(assignment, DateTime now) -> AssignmentStatus` | Pure calculation returning `completed`, `overdue`, `inProgress`, or `upcoming`. | None. | No |
| `assignmentsByStatus` | `(status, DateTime now) -> List<Assignment>` | Filters assignments by their computed dynamic status. | None. | No |

#### 3. `FocusService` (Author: Yeat David)

Focus mode is structured into sessions (e.g., 4 focus sessions per cycle, each lasting 25 minutes, followed by a 5-minute break session). Students can claim coin rewards upon completing each 25-minute focus session (`rewardClaimed: bool`), but not on break sessions, and not from tasks or assignments.

| Operation | Parameters & Return | Rules & Preconditions | Invalid Cases / Errors | Multi-Entity? |
|---|---|---|---|---|
| `startSession` | `(plannedMinutes, {sessionIndex = 1, isBreak = false, targetId?, soundId?}) -> FocusSession` | Creates new session in `SessionStatus.running`. Rejects if another session is already running. Validates sound is owned if `soundId` provided. Standard focus session is 25 minutes; break session is 5 minutes. | Throws `ArgumentError` if plannedMinutes ≤ 0. Throws `StateError` if a session is currently running. Throws `ArgumentError` if sound not owned. | No |
| `pause` | `(sessionId) -> FocusSession` | Transitions session from `running` to `paused`. Paused time is not counted toward focus duration. | Throws `ArgumentError` if session not found. Throws `StateError` if session is not in `running` state. | No |
| `resume` | `(sessionId) -> FocusSession` | Transitions session from `paused` back to `running`. | Throws `ArgumentError` if session not found. Throws `StateError` if session is not `paused`. | No |
| `completeSession` | `(sessionId, actualSeconds) -> FocusSession` | Transitions to `SessionStatus.completed`. Records `actualSeconds` and `endedAt`. Prepares session for coin claim (`rewardClaimed: false`). Break sessions transition without reward eligibility. | Throws `ArgumentError` if session not found or actualSeconds ≤ 0. Throws `StateError` if session already completed or cancelled. | No |
| `claimReward` | `(sessionId) -> FocusSession` | **Session Reward Claiming (US-03 & US-04):** Only completed focus sessions (`!isBreak`) can claim rewards. Verifies `rewardClaimed == false`. Awards 25 coins for a completed 25-min session (1 coin per focus minute). If early finish with ≥ 300s (5 min), awards 1 coin per 2 full minutes (`floor(actualSeconds / 120)`). If < 300s, awards 0 coins. Calls `WalletService.award(...)` and locks `rewardClaimed = true` on the session. | Throws `ArgumentError` if session not found. Throws `StateError` if session is not completed, if session is a break session, or if reward was already claimed (`rewardClaimed == true`). | **Yes** (`FocusSession`, `Wallet`, `CoinTransaction`) |
| `cancelSession` | `(sessionId) -> FocusSession` | Transitions session to `SessionStatus.cancelled`. Awards 0 coins; reward cannot be claimed. | Throws `ArgumentError` if session not found. Throws `StateError` if session already completed or cancelled. | No |

**Early Finish Reward Calculation Examples (`actualSeconds ~/ 120`, minimum 5 minutes / 300s):**

| Focus Time | Calculation | Coins Awarded | Notes |
|---|---|---|---|
| **4m 30s** | Under 5 minutes | **0 coins** | **Anti-farming threshold:** sessions under 5 minutes (< 300s) award nothing. |
| **5 minutes** | `5 ~/ 2 = 2.5` → | **2 coins** | 4 minutes counted (2 × 2 min), 1 minute leftover dropped. |
| **9 minutes** | `9 ~/ 2 = 4.5` → | **4 coins** | 8 minutes counted (4 × 2 min), 1 minute leftover dropped. |
| **10 minutes** | `10 ~/ 2 = 5.0` → | **5 coins** | Exactly five 2-minute blocks. |
| **20 minutes** | `20 ~/ 2 = 10.0` → | **10 coins** | Exactly ten 2-minute blocks. |
| **25 minutes (Full)** | Full completion | **25 coins** | Completed the planned goal; full rate (1 coin per planned minute) applies. |

#### 4. `WalletService` (Author: Yeat David)

| Operation | Parameters & Return | Rules & Preconditions | Invalid Cases / Errors | Multi-Entity? |
|---|---|---|---|---|
| `balance` | `-> int` (getter) | Returns current verified Star Coin balance. Mathematically guaranteed ≥ 0. | None. | No |
| `award` | `(amount, type, sourceId, reason) -> void` | Increases balance by `amount`. Appends immutable `CoinTransaction` record. Rejects duplicate awards for identical `type` and `sourceId`. | Throws `ArgumentError` if `amount <= 0`. Throws `StateError` if duplicate reward attempted for same source. | **Yes** (`Wallet`, `CoinTransaction`) |
| `spend` | `(amount, sourceId, reason) -> void` | Decreases balance by `amount`. Appends immutable negative `CoinTransaction` record. | Throws `ArgumentError` if `amount <= 0`. Throws `StateError('Insufficient coin balance')` if `balance < amount`; balance remains unchanged. | **Yes** (`Wallet`, `CoinTransaction`) |
| `history` | `-> List<CoinTransaction>` | Returns unmodifiable list of coin transactions in reverse chronological order (newest first). | None. | No |

#### 5. `ShopService` (Author: Yeat David)

| Operation | Parameters & Return | Rules & Preconditions | Invalid Cases / Errors | Multi-Entity? |
|---|---|---|---|---|
| `purchase` | `(itemId) -> void` | **Milestone 1 Shop Rule:** Checks item exists, checks item is not already owned (`!inventory.isOwned(itemId)`), verifies `wallet.balance >= item.price`. Debits coins via `WalletService.spend(...)` and adds `itemId` to `inventory.ownedItemIds`. Atomicity guaranteed. | Throws `ArgumentError` if item ID not in catalog. Throws `StateError('Item already owned')` if user already owns it. Throws `StateError('Insufficient coin balance')` if balance too low. | **Yes** (`ShopItem`, `Inventory`, `Wallet`, `CoinTransaction`) |
| `equip` | `(itemId) -> void` | Equips an owned item. Updates `equippedThemeId` (if theme), `equippedAvatarId` (if avatar), or `defaultSoundId` (if sound). | Throws `ArgumentError` if item is not owned. | No |
| `itemsByType` | `(ShopItemType type) -> List<ShopItem>` | Retrieves catalog items filtered by type with current user ownership metadata. | None. | No |
| `isOwned` | `(itemId) -> bool` | Checks if item is a free starter item (`isStarter == true`) or present in user's `Inventory.ownedItemIds`. | None. | No |

---

## 4. Test Layer

The test suite validates normal flows, rejected invalid states, and boundary conditions across all 5 services. Tests are organized under `test/service/` mirroring `lib/service/`.

### 4.1 Test Cases Matrix

| Test ID | Service & Operation | Category | Scenario Description | Expected Outcome |
|---|---|---|---|---|
| **TC-01** | `TaskService.createTask` | **Normal** | Create task with valid title, priority, due date. | Task created with `status: TaskStatus.todo`. |
| **TC-02** | `TaskService.createTask` | **Invalid** | Create task with empty string `""` or whitespace `"   "`. | Throws `ArgumentError`, task creation rejected. |
| **TC-03** | `TaskService.completeTask` | **Normal** | Complete a `todo` task. | Status becomes `TaskStatus.done`. 0 coins awarded (tasks do not claim rewards). |
| **TC-04** | `TaskService.uncompleteTask` | **Normal** | Uncomplete a done task back to `todo`. | Status transitions from `TaskStatus.done` back to `TaskStatus.todo`. |
| **TC-05** | `AssignmentService.createAssignment` | **Normal** | Create assignment with title, due date, priority. | Assignment created successfully with `status: TaskStatus.todo`. |
| **TC-06** | `AssignmentService.toggleSubtask` | **Normal** | Complete subtasks of an assignment. | Subtask marked `done`; 0 coins awarded; parent assignment remains `status: TaskStatus.todo`. |
| **TC-07** | `AssignmentService.deleteAssignment` | **Composition Rule** | Delete an assignment that contains multiple subtasks. | Assignment deleted; all child subtasks are cascade-deleted simultaneously. |
| **TC-08** | `AssignmentService.completeAssignment` | **Normal** | Explicitly complete assignment with all subtasks done. | Assignment status becomes `TaskStatus.done`, 0 coins awarded (assignments do not claim rewards). |
| **TC-09** | `FocusService.startSession` | **Invalid** | Start session with planned minutes ≤ 0. | Throws `ArgumentError('Planned minutes must be greater than zero')`. |
| **TC-10** | `FocusService.completeSession` | **Normal** | Complete full 25-minute focus session (1500 seconds). | Status is `completed`, `rewardClaimed` remains `false` until claimed. |
| **TC-11** | `FocusService.claimReward` | **Normal & Multi** | Claim reward for a completed 25-minute focus session (session 1 of 4). | Status is `completed`, awards 25 coins to Wallet, sets `rewardClaimed: true`, 1 transaction logged. |
| **TC-12** | `FocusService.claimReward` | **Boundary / Anti-Farm** | Attempt to claim reward again on already claimed session (`rewardClaimed: true`). | Throws `StateError('Reward already claimed for this session')`, Wallet balance unchanged. |
| **TC-13** | `FocusService.claimReward` | **Boundary / Break Session** | Attempt to claim reward on a 5-minute break session (`isBreak: true`). | Throws `StateError('Break sessions do not earn rewards')`, 0 coins awarded. |
| **TC-14** | `FocusService.claimReward` | **Boundary / Ratio (Early Finish)** | Finish focus session early at 9m (540s) out of 25m planned. | Payout is 1 coin per 2 full minutes = floor(9 / 2) = 4 coins, `rewardClaimed` set to `true`. |
| **TC-15** | `FocusService.claimReward` | **Boundary / Threshold** | Finish early at 4m 59s (< 300s minimum threshold). | Payout is 0 coins (anti-farming protection). |
| **TC-16** | `ShopService.purchase` | **Normal & Multi** | Buy a 50-coin item with balance of 100 coins. | Balance decreases to 50 coins, item added to `ownedItemIds`, transaction recorded. |
| **TC-17** | `ShopService.purchase` | **Invalid / Duplicate** | Attempt to buy an item already owned in inventory. | Throws `StateError('Item already owned')`, no coins deducted. |
| **TC-18** | `ShopService.purchase` | **Invalid / Balance** | Attempt to buy a 100-coin item with only 40 coins in wallet. | Throws `StateError('Insufficient coin balance')`, balance stays 40, item not acquired. |
| **TC-19** | `ShopService.purchase` | **Boundary / Exact** | Buy an item costing exactly the current balance (balance = 50, cost = 50). | Purchase succeeds, new balance is exactly 0 (boundary non-negative check). |

---

## 5. Defense Preparation: Questions & Answers (CADT IDT Jury)

### Q1: How did you divide the workload between the two team members?
**Answer:**  
We established a strict vertical slice ownership model documented in `10-team-workflow.md`:
- **Tiek Chhunhour (Member A):** Owned the planning core—models (`Task`, `Assignment`, `Subtask`), services (`TaskService`, `AssignmentService`), and their unit test suites.
- **Yeat David (Member B):** Owned the gamification and focus systems—models (`FocusSession`, `Wallet`, `CoinTransaction`, `ShopItem`, `Inventory`), services (`FocusService`, `WalletService`, `ShopService`), and their unit test suites.  
Both members collaborated on the shared contract (`WalletService`), allowing parallel feature development.

### Q2: Why is Course not part of Milestone 2?
**Answer:**  
In our Milestone 1 problem statement and project overview (`README.md`), Courses were explicitly categorized as *Future Enhancements (Optional Scope)* for Milestone 3, prioritizing immediate, low-friction task and deadline tracking. For Milestone 2, our core domain strictly models the 4 committed user stories (US-01 through US-04). The rubric's requirement for Composition and Aggregation is completely demonstrated through `Assignment ──* Subtask` (where subtasks have no independent lifecycle and are cascade-deleted) and `Wallet ──* CoinTransaction`.

### Q3: How do you enforce the Milestone 1 shop rule that an owned item cannot be bought again?
**Answer:**  
In `ShopService.purchase(itemId)`, the service inspects `Inventory.isOwned(itemId)` prior to executing any transaction. If the item is already present in `ownedItemIds` (or is a free starter item), the operation immediately throws a `StateError('Item already owned')`. Coins are neither deducted nor is a transaction logged.

### Q4: Why did you separate the timer UI from `FocusService`?
**Answer:**  
The UI layer is responsible for ticking the clock on screen (via a Flutter `Ticker` or `Timer.periodic`), while `FocusService` is responsible for business integrity and reward evaluation. Keeping the service pure means session state transitions (`running` → `paused` → `completed`), anti-farming thresholds (< 5 minutes = 0 coins), and coin payouts can be unit-tested instantaneously without waiting 25 real-world minutes.

### Q5: How do you prevent a student from cheating or farming coins?
**Answer:**  
We engineered three independent defense layers:
1. **Task & Assignment Decoupling (No Checkbox Farming):** Tasks, subtasks, and assignments do not award coins (`rewardClaimed` is not allowed on tasks or assignments), preventing students from creating and checking off trivial tasks to farm currency.
2. **Session-Based Reward Claiming (`rewardClaimed: bool`):** Coins can only be earned and claimed through completed Focus Mode sessions (e.g., in a 4-session focus cycle of 25-minute focus intervals and 5-minute break intervals). When a 25-minute focus session completes, the student can claim their coin reward, which permanently locks `rewardClaimed = true` (default: `false`). Duplicate claims for the same session are strictly rejected, and break sessions (5 min) award 0 coins.
3. **Minimum Focus Threshold:** Finishing a study session early under 5 minutes awards 0 coins, and cancelled sessions award 0 coins, preventing spamming start/finish cycles.
