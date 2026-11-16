# 05 Testing

The course requires: **test every service operation**, covering **normal cases, edge cases, and invalid operations**. Unit tests on services are the priority. Widget tests are a bonus.

## How to structure tests

- One test file per service in `test/service/`, mirroring `lib/service/`.
- One `group` per operation, with at least one normal, one edge and one invalid test inside.
- Each test sets up fresh in-memory repositories/services, so tests never depend on each other.
- Control time: pass "now" into anything that depends on dates, so overdue tests don't break tomorrow.
- Test names describe the rule in plain words, for example "completing a task twice awards coins once".

## Test plan

Use this as a checklist. Tick each row when its test passes.

### TaskService (Author: Tiek Chhunhour)

| Operation | Normal | Edge | Invalid |
|---|---|---|---|
| `createTask` | [ ] Valid task saved, not completed | [ ] Title with spaces around is trimmed | [ ] Empty title rejected [ ] Whitespace title rejected |
| `updateTask` | [ ] Title and priority change | [ ] Status preserved | [ ] Unknown id [ ] Empty title |
| `deleteTask` | [ ] Task removed | [ ] Task with no due date removed cleanly | [ ] Unknown id |
| `completeTask` | [ ] Completed, status becomes done (0 coins awarded) | [ ] Complete, uncomplete, complete | [ ] Unknown id |
| `uncompleteTask` | [ ] Back to not completed (todo) | [ ] Uncompleting already todo task | [ ] Unknown id |
| `filters` | [ ] Today returns only today's tasks | [ ] Task with no due date | |

### AssignmentService (Author: Tiek Chhunhour)

| Operation | Normal | Edge | Invalid |
|---|---|---|---|
| `createAssignment` | [ ] Valid assignment saved | [ ] Due date today counts as upcoming | [ ] Empty title [ ] Whitespace title |
| `updateAssignment` | [ ] Fields change | [ ] Completed subtasks still completed | [ ] Unknown id [ ] Empty title |
| `deleteAssignment` | [ ] Assignment removed and all child subtasks cascade-deleted (Composition) | [ ] Assignment with zero subtasks removed cleanly | [ ] Unknown id |
| `addSubtask` | [ ] Subtask added to assignment | [ ] Multiple subtasks added sequentially | [ ] Empty subtask title [ ] Unknown assignment id |
| `toggleSubtask` | [ ] Subtask completes (0 coins awarded) | [ ] All subtasks done, parent assignment still not completed | [ ] Unknown subtask [ ] Unknown assignment id |
| `allSubtasksDone` | [ ] True when all done | [ ] False with zero subtasks | [ ] Unknown assignment id |
| `completeAssignment` | [ ] Completed, status becomes done (0 coins awarded) | [ ] Complete when subtasks are partially done | [ ] Unknown id |
| `statusOf` | [ ] Upcoming [ ] InProgress [ ] Completed | [ ] Due exactly now [ ] Completed after due date shows completed | |

### FocusService (Author: Yeat David)

| Operation | Normal | Edge | Invalid |
|---|---|---|---|
| `startSession` | [ ] Running session created (25-min focus or 5-min break) | [ ] 1 minute session | [ ] 0 minutes [ ] Negative minutes [ ] Unowned sound [ ] Second session while one runs |
| `pause` / `resume` | [ ] Running to paused to running | [ ] Paused time not counted | [ ] Pause when paused [ ] Resume when running |
| `completeSession` | [ ] 25 min focus session completed (`rewardClaimed: false`) | [ ] 5 min break session completed | [ ] Complete twice [ ] Complete a cancelled session |
| `claimReward` | [ ] Completed 25 min focus session awards 25 coins, locks `rewardClaimed = true` | [ ] 20 of 25 min gives 10 coins [ ] Exactly 5 min gives 2 coins [ ] Odd minutes round down (9 min gives 4 coins) [ ] < 5 min focused gives 0 coins | [ ] Claim twice on same session rejected (`rewardClaimed == true`) [ ] Claim on 5-min break session rejected (`isBreak == true`) [ ] Claim on incomplete/cancelled session rejected |
| `cancelSession` | [ ] Status cancelled, awards 0 coins | [ ] Cancel from paused | [ ] Cancel a completed session |

### WalletService (Author: Yeat David)

| Operation | Normal | Edge | Invalid |
|---|---|---|---|
| `award` | [ ] Balance increases, transaction recorded | [ ] Two different sources both rewarded | [ ] Same source twice [ ] Amount 0 or negative |
| `spend` | [ ] Balance decreases | [ ] Spending exactly the whole balance leaves 0 | [ ] More than balance: rejected, balance unchanged [ ] Amount 0 or negative |

### ShopService (Author: Yeat David)

| Operation | Normal | Edge | Invalid |
|---|---|---|---|
| `purchase` | [ ] Coins deducted, item owned | [ ] Price equal to balance | [ ] Not enough coins: no deduction, not owned [ ] Already owned rejected [ ] Unknown item |
| `equip` | [ ] Theme replaces previous theme | [ ] Equip already equipped item | [ ] Not owned |
| `starter items` | [ ] Owned from the start, price 0 | | [ ] Buying a starter item rejected as owned |

## Widget tests (optional)

Good candidates: a task card shows the priority label (not only color), the Save button is disabled with an empty title, the purchase button is disabled when coins are short.

## Manual UI checks

Before each milestone demo, check every screen on:

- A small phone (360 dp wide) and a standard phone (390 dp wide).
- The largest system font size: no clipped or overlapping text.
- Empty data (no tasks, no assignments): empty states show, nothing crashes.

`UI-QA-Audit-Prompt.md` has the full visual checklist.
