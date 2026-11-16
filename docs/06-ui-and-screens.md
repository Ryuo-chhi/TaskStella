# 06 UI and Screens

Screen layouts, colors and the full navigation table live in `TaskStella-Design-Spec.md`. This page covers how to build them in Flutter under the course rules: reusable `StatelessWidget` and `StatefulWidget`, `setState`, callbacks and `Navigator`.

## Screen priority

The course asks for at least 4 screens. Build in this order so a working app exists early.

| Order | Screen (spec number)                                                                 | User story | Stateful?            |
| ----- | ------------------------------------------------------------------------------------ | ---------- | -------------------- |
| 1     | Home (04)                                                                            | All        | Yes: tasks, balance  |
| 2     | Planner: Tasks (05), Add/Edit Task (06)                                              | US-01      | Yes                  |
| 3     | Assignments (07), Assignment Detail (08), Add/Edit Assignment (09), Courses (10, 11) | US-02      | Yes                  |
| 4     | Focus Setup (14), Focus Timer (15), Session Complete (16)                            | US-03      | Yes, the timer ticks |
| 5     | Shop (17), Purchase sheet (18), My Collection (19)                                   | US-04      | Yes                  |
| Later | Splash, Onboarding, Notes, Schedule, Stats, Profile, Settings                        |            |                      |

## Stateless or stateful

- **StatelessWidget:** anything that only displays what it is given. Most shared widgets: TaskCard, CoinChip, PriorityPill, PrimaryButton, EmptyState.
- **StatefulWidget:** screens that load and change data, and widgets with their own short-lived UI state (a text field with validation, the timer ring, the segmented tabs).

Rule of thumb: keep state as high as needed and as low as possible. A checkbox does not own the task's completed state, the screen does.

## Data flow pattern

```
Screen (StatefulWidget)
  ├─ reads lists from a service in initState
  ├─ passes data + callbacks down
  │     TaskCard(task, onToggle, onTap)
  └─ on callback:
        call service  ->  catch errors  ->  setState  ->  show feedback
```

Example flow for "tap checkbox on Home":

1. TaskCard calls `onToggle(task)`.
2. Home calls `taskService.completeTask(task.id)`.
3. Home reloads the task list and balance, then calls `setState`.
4. Home shows the coin animation if coins were awarded, or nothing if the reward was already claimed.

## Reusable widgets

Build these in `lib/ui/widgets/` before the screens that use them. One widget per file, with a short comment on its purpose and states. Widgets read colors and sizes from the theme (see [07-design-tokens.md](07-design-tokens.md)), never hard-coded values.

| Group      | Widgets                                                                  | Used on                               |
| ---------- | ------------------------------------------------------------------------ | ------------------------------------- |
| Layout     | AppScaffold, ScreenHeader, SectionHeader (title + See all)               | Every screen                          |
| Navigation | BottomNavBar (5 tabs), SegmentedTabs, WeekStrip                          | Home, Planner, Shop                   |
| Buttons    | PrimaryButton, SecondaryButton, DestructiveTextButton, FloatingAddButton | Forms, lists                          |
| Inputs     | AppTextField (with error text), DatePickerRow, ChipSelector              | Add Task, Add Assignment, Focus Setup |
| Cards      | TaskCard, AssignmentCard, CourseCard, HeroCard, ShopItemCard             | Lists                                 |
| Indicators | CoinChip, PriorityPill, StatusChip, ProgressBar, ProgressRing            | Many                                  |
| Feedback   | EmptyState, ConfirmDialog, snackbar helper                               | Lists, delete, cancel session         |
| Media      | CompanionView, ThemeScene, SoundChip                                     | Home, Focus, Shop                     |

Start with the ones the first screens need. Add a widget to this table only when a second screen needs it, otherwise keep it private to the screen.

## UI rules that come from business rules

| Rule                 | UI behavior                                                                                                                       |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| Empty title rejected | Inline error under the field. Save stays enabled but shows the error, or is disabled until valid. Pick one and use it everywhere. |
| Never color alone    | Priority and status always show a label or icon.                                                                                  |
| All subtasks done    | Show "All subtasks done. Mark assignment complete?", never auto-complete.                                                         |
| Duration 0           | Start focus button disabled.                                                                                                      |
| Cancel session       | Confirm dialog saying the full reward is lost.                                                                                    |
| Not enough coins     | Buy button disabled, shows how many coins are missing.                                                                            |
| Delete               | Always a confirm dialog. Swipe to delete also has a visible menu option.                                                          |

## Focus Timer notes

- Use a `Timer.periodic` inside the timer screen's `State`. Cancel it in `dispose`.
- Stop and release the audio player in `dispose` as well, so the sound never keeps playing after leaving the screen.
- Send the elapsed seconds to FocusService when the session ends. The screen counts time, the service decides the reward.

## States every data screen needs

- **Empty:** illustration plus one short line and an action ("No tasks yet. Add your first one.").
- **Error:** a message that says what went wrong and how to fix it.
- **Loading:** only needed if storage becomes asynchronous.
