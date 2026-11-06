# 05 Testing

The course requires: **test every service operation**, covering **normal cases, edge cases, and invalid operations**. Unit tests on services are the priority. Widget tests are a bonus.

## How to structure tests

- One test file per service in `test/services/`, mirroring `lib/services/`.
- One `group` per operation, with at least one normal, one edge and one invalid test inside.
- Each test sets up fresh in-memory repositories, so tests never depend on each other.
- Control time: pass "now" into anything that depends on dates, so overdue tests don't break tomorrow.
- Test names describe the rule in plain words, for example "completing a task twice awards coins once".

## Test plan

Use this as a checklist. Tick each row when its test passes.

### TaskService

| Operation | Normal | Edge | Invalid |
|---|---|---|---|
| createTask | [ ] Valid task saved, not completed | [ ] Title with spaces around is trimmed | [ ] Empty title rejected [ ] Whitespace title rejected [ ] Unknown course rejected |
| updateTask | [ ] Title and priority change | [ ] Completed state and reward flag kept | [ ] Unknown id [ ] Empty title |
| deleteTask | [ ] Task removed | [ ] Balance unchanged after deleting a completed task | [ ] Unknown id |
| completeTask | [ ] Completed and 5 coins awarded | [ ] Complete, uncomplete, complete awards once [ ] 10th task of the day rewarded, 11th earns 0 but completes [ ] Cap resets the next day | [ ] Unknown id |
| uncompleteTask | [ ] Back to not completed | [ ] Coins not removed | [ ] Unknown id |
| filters | [ ] Today returns only today's tasks | [ ] Task with no due date | |

### AssignmentService

| Operation | Normal | Edge | Invalid |
|---|---|---|---|
| createAssignment | [ ] Saved under its course | [ ] Due date today counts as upcoming | [ ] Empty title [ ] Unknown course [ ] Workload 0 |
| updateAssignment | [ ] Fields change | [ ] Completed subtasks still completed | [ ] Unknown id |
| toggleSubtask | [ ] Subtask completes, coins awarded | [ ] All subtasks done, assignment still not completed [ ] Toggle off and on awards once | [ ] Unknown subtask |
| allSubtasksDone | [ ] True when all done | [ ] False with zero subtasks | |
| completeAssignment | [ ] Completed and 10 coins awarded | [ ] Uncomplete and complete again awards once | [ ] Unknown id |
| statusOf | [ ] Upcoming [ ] Completed | [ ] Due exactly now [ ] Completed after due date shows completed | |
| deleteAssignment | [ ] Removed with subtasks | | [ ] Unknown id |

### FocusService

| Operation | Normal | Edge | Invalid |
|---|---|---|---|
| startSession | [ ] Running session created | [ ] 1 minute session | [ ] 0 minutes [ ] Negative minutes [ ] Unowned sound [ ] Second session while one runs |
| pause / resume | [ ] Running to paused to running | [ ] Paused time not counted | [ ] Pause when paused [ ] Resume when running |
| completeSession | [ ] 25 min session reached gives 25 coins | [ ] 20 of 25 min gives 10 coins [ ] 4 min focused gives 0 [ ] Exactly 5 min gives 2 [ ] Odd minutes round down (9 min gives 4) | [ ] Complete twice [ ] Complete a cancelled session |
| cancelSession | [ ] Status cancelled, no full reward | [ ] Cancel from paused | [ ] Cancel a completed session |

### WalletService

| Operation | Normal | Edge | Invalid |
|---|---|---|---|
| award | [ ] Balance increases, transaction recorded | [ ] Two different sources both rewarded | [ ] Same source twice [ ] Amount 0 or negative |
| spend | [ ] Balance decreases | [ ] Spending exactly the whole balance leaves 0 | [ ] More than balance: rejected, balance unchanged [ ] Amount 0 or negative |

### ShopService

| Operation | Normal | Edge | Invalid |
|---|---|---|---|
| purchase | [ ] Coins deducted, item owned | [ ] Price equal to balance | [ ] Not enough coins: no deduction, not owned [ ] Already owned [ ] Unknown item |
| equip | [ ] Theme replaces previous theme | [ ] Equip already equipped item | [ ] Not owned |
| starter items | [ ] Owned from the start, price 0 | | [ ] Buying a starter item rejected as owned |

## Widget tests (optional)

Good candidates: a task card shows the priority label (not only color), the Save button is disabled with an empty title, the purchase button is disabled when coins are short.

## Manual UI checks

Before each milestone demo, check every screen on:

- A small phone (360 dp wide) and a standard phone (390 dp wide).
- The largest system font size: no clipped or overlapping text.
- Empty data (no tasks, no assignments): empty states show, nothing crashes.

`UI-QA-Audit-Prompt.md` has the full visual checklist.
