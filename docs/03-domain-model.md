# 03 Domain Model

The domain model is the set of entities the app works with. The course asks for **at least 4 meaningful entities with meaningful relationships**. TaskStella has more, so build the core ones first and add the rest only if time allows.

## Priority of entities

| Priority | Entities | Why |
|---|---|---|
| Core (Milestone 2) | Task, Course, Assignment, Subtask, FocusSession, Wallet, CoinTransaction, ShopItem, Inventory | Cover all four user stories (US-01 to US-04) |
| Later | Note, ScheduleBlock, UserProfile | Nice to have, not needed for the minimums |

## Relationships

```
Course 1 ──── * Assignment 1 ──── * Subtask
Course 1 ──── * Note                         (later)
Task   * ──── 0..1 Course
FocusSession * ──── 0..1 Task  or  0..1 Assignment
Wallet 1 ──── * CoinTransaction
CoinTransaction * ──── 1 source (Task, Subtask, FocusSession, or ShopItem purchase)
Inventory * ──── * ShopItem                   (owned items)
Inventory ──── 1 equipped theme, 1 equipped chibi, 1 default sound
```

Draw this as a UML class diagram for Milestone 2.

## Entities

Field types are suggestions. Use immutable models (final fields) and create a changed copy instead of mutating, unless the team agrees otherwise.

### Task (US-01)

| Field | Type | Rule |
|---|---|---|
| id | String | Unique |
| title | String | Not empty, not whitespace only |
| priority | Priority (low, medium, high) | Required, default medium |
| dueDate | DateTime, optional | |
| courseId | String, optional | Must reference an existing course if set |
| isCompleted | bool | |
| rewardClaimed | bool | Set to true the first time coins are awarded. Never reset. Prevents double rewards. |
| createdAt | DateTime | |

### Course

| Field | Type | Rule |
|---|---|---|
| id | String | Unique |
| name | String | Not empty |
| colorKey | String | One of the theme tint keys, not a hex value |

Deleting a course: decide whether its assignments are deleted too or blocked. Record the decision and test it.

### Assignment (US-02)

| Field | Type | Rule |
|---|---|---|
| id | String | |
| courseId | String | Required, must exist |
| title | String | Not empty, not whitespace only |
| description | String | Can be empty |
| dueDate | DateTime | Required |
| priority | Priority | |
| estimatedHours | double | Greater than 0 |
| isCompleted | bool | Only changes when the user explicitly marks it |
| rewardClaimed | bool | Same double-reward protection as Task |
| subtasks | List of Subtask | Kept when the assignment is edited |

**Derived status** (computed, not stored):

| Status | Condition |
|---|---|
| completed | `isCompleted` is true |
| overdue | not completed and `dueDate` is before now |
| upcoming | not completed and `dueDate` is now or later |

Pass "now" into the status calculation as a parameter so tests can control time.

### Subtask

| Field | Type | Rule |
|---|---|---|
| id | String | |
| title | String | Not empty |
| isCompleted | bool | |
| rewardClaimed | bool | Same double-reward protection as Task |

### FocusSession (US-03)

| Field | Type | Rule |
|---|---|---|
| id | String | |
| plannedMinutes | int | Greater than 0 |
| actualSeconds | int | Time actually focused, pauses excluded |
| status | running, paused, completed, cancelled | See state diagram in [04-services-and-rules.md](04-services-and-rules.md#focusservice) |
| taskId or assignmentId | String, optional | What the user focused on |
| soundId | String, optional | Must be an owned sound |
| startedAt, endedAt | DateTime | |
| coinsAwarded | int | 0 until the session ends |

### Wallet and CoinTransaction (US-04)

| Wallet field | Type | Rule |
|---|---|---|
| balance | int | Never below 0 |

| CoinTransaction field | Type | Rule |
|---|---|---|
| id | String | |
| amount | int | Positive for earning, negative for spending |
| type | taskReward, subtaskReward, focusReward, purchase | |
| sourceId | String | The task, subtask, session or item that caused it |
| createdAt | DateTime | |

The transaction list is the history. It also lets the service check "has this source already been rewarded" as a second safety check.

### ShopItem

Loaded from the bundled catalog (see [08-assets.md](08-assets.md)). Read only at runtime.

| Field | Type | Rule |
|---|---|---|
| id | String | |
| name | String | |
| type | sound, theme, chibi | |
| price | int | 0 or more |
| assetPath | String | Points to a bundled file |
| isStarter | bool | Starter items are free and always owned |

### Inventory

| Field | Type | Rule |
|---|---|---|
| ownedItemIds | Set of String | Always includes the three starter items |
| equippedThemeId | String | Must be owned and of type theme |
| equippedChibiId | String | Must be owned and of type chibi |
| defaultSoundId | String | Must be owned and of type sound |

### Later entities

- **Note:** id, title, body, optional courseId, updatedAt.
- **ScheduleBlock:** id, title, start, end (end after start), repeat (none, daily, weekly), optional link to a course or assignment.
- **UserProfile:** display name, streak count, onboarding seen flag.
