# 10 Team Workflow

TaskStella is built by a team of two. The course checks that each member contributes through Git commits and can explain their own code in interviews. Split the work so each person owns complete vertical slices: model, service, tests and screens for the same feature.

## Ownership split (proposal)

| Area     | Member A (Tiek Chhunhour)                              | Member B (Yeat David)                                                        |
| -------- | ------------------------------------------------------ | ---------------------------------------------------------------------------- |
| Models   | Task, Assignment, Subtask (Course deferred to M3)      | FocusSession, Wallet, CoinTransaction, ShopItem, Inventory                   |
| Services | TaskService, AssignmentService                         | FocusService, WalletService, ShopService                                     |
| Tests    | Tests for A's services (`test/service/`)              | Tests for B's services (`test/service/`)                                     |
| Screens  | Home, Planner, Add/Edit Task, Assignments, Assignment Detail | Focus Setup, Focus Timer, Session Complete, Shop, Purchase sheet, Collection |
| Shared   | Theme tokens, layout widgets, BottomNavBar             | Cards for shop, CoinChip, CompanionView, ThemeScene, audio                   |

Names: Member A = `Tiek Chhunhour`, Member B = `Yeat David`.

Shared contract: FocusService calls WalletService to award coins when focus sessions are completed and claimed. Agree on WalletService's operations early (see [04-services-and-rules.md](04-services-and-rules.md#4-walletservice-author-yeat-david)) so both sides can work in parallel.

Both members should still understand the whole app, since the jury can ask anyone about anything.

## Git workflow

- `main` always runs and passes tests.
- Work on a branch per feature: `feature/task-service`, `feature/shop-screen`.
- Small commits with clear messages that describe what changed, for example "Add rewardClaimed flag to prevent double session rewards".
- Before merging into `main`: `flutter analyze` is clean, `dart format .` is applied, `flutter test` passes.
- The other member reviews the merge request and reads the code. Reviewing is a good way to prepare for interview questions about the partner's part.
- Never commit for the partner or squash their commits into yours. Commits are the evidence of contribution.

## Definition of done for a feature

- [ ] Model and service written by the owner
- [ ] Tests cover normal, edge and invalid cases for every operation
- [ ] Screen uses theme tokens and shared widgets
- [ ] Empty and error states handled
- [ ] Checked on a 360 dp and a 390 dp screen, and with large system font
- [ ] Docs updated if a rule, field or decision changed
- [ ] Owner can explain every line

## Keeping docs current

When a decision is made (storage, audio package, coin amounts), update the table in [02-architecture.md](02-architecture.md#open-decisions) in the same commit as the code that uses it.
