# 02 Architecture

TaskStella follows the layered architecture taught in the course: **model**, **service**, **test**, plus a **UI** layer and a small **data** layer for storage. Each layer has one job and only talks to the layer below it.

## Layers

| Layer             | Job                                                                                    | Knows about                    | Must not know about                      |
| ----------------- | -------------------------------------------------------------------------------------- | ------------------------------ | ---------------------------------------- |
| Model             | Describes the data: Task, Assignment, Wallet... Holds simple validation for itself.    | Nothing else                   | Flutter, services, storage               |
| Service           | Business rules: completing a task, awarding coins, buying an item. Plain Dart classes. | Models, repositories           | Flutter widgets, `BuildContext`, screens |
| Data (repository) | Stores and loads models. Starts as in-memory lists.                                    | Models                         | Services, UI                             |
| UI                | Screens and widgets. Shows data, collects input, calls services, calls `setState`.     | Services, models, theme        | Storage details, business rules          |
| Test              | Proves every service operation works.                                                  | Models, services, repositories | UI (unit tests only)                     |

Dependency direction: **UI → Service → Repository → Model**. Nothing points upward.

### Why this split

- Services are plain Dart, so they can be unit tested without running the app. The course grades this directly.
- Screens stay small. A screen asks a service to do something and redraws, it never decides whether coins are awarded.
- Storage can change later (in-memory to `shared_preferences`) without touching services or screens, because only repositories change.

## Folder structure

```
assets/
  fonts/  sounds/  themes/  companions/  icons/
data/
  catalog/                  bundled shop catalog (sounds, themes, companions)
lib/
  main.dart                 app entry, creates services, starts the app
  model/                    one file per entity and enum
  service/                  one file per service
  ui/
    theme/                  colors, text styles, spacing, radii, durations
    widgets/                reusable widgets shared by several screens
    screens/                one folder per area: home, planner, focus, shop, notes, profile
test/
  service/
```

This is layer-first, not feature-first. `AGENTS.md` suggests feature folders, but the course architecture wins.

## How state works without a framework

The course forbids state management packages. The approach:

1. `main.dart` creates the repositories and services **once**.
2. Services are passed down to screens through constructor parameters.
3. A screen that shows changing data is a `StatefulWidget`. It reads data from a service in `initState`, keeps what it needs in its `State`, and calls `setState` after any change.
4. Child widgets receive data and **callbacks** (for example `onToggleComplete`). They never call services directly. The screen handles the callback, calls the service, then calls `setState`.
5. When returning from a screen that changed data (Add Task, Purchase), the caller refreshes after `Navigator.push` completes. `Navigator.pop` can return a result such as "saved" or "deleted".

### Shared values across tabs

The coin balance, equipped theme and equipped Companion appear on many screens. Keep the services as the single owner of these values. The root app widget is a `StatefulWidget` that rebuilds the `MaterialApp` theme when the equipped theme changes, and passes a callback such as `onThemeChanged` down to the screens that can change it (Shop, Collection).

## Navigation

Use Flutter's built-in `Navigator` with `MaterialPageRoute`.

- The bottom navigation bar switches between five tabs: Home, Planner, Focus, Shop, Profile. The tab host is a `StatefulWidget` that holds the selected index.
- Detail and form screens (Add Task, Assignment Detail, Focus Timer) are pushed on top of the tab host.
- The full tap-to-destination table is in `TaskStella-Design-Spec.md` section 2.2.

## Error handling

- Services report invalid operations with clear errors (for example an exception or a result type, choose one approach and use it everywhere). Record the choice in the decisions table.
- The UI catches these and shows an inline message or a snackbar. No error should crash the app.

## Open decisions

Fill in this table as the team decides. Ask the teacher if a package is allowed before adding it.

| Decision                | Options                                                             | Chosen                                                                                                | Reason                                | Date       |
| ----------------------- | ------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- | ------------------------------------- | ---------- |
| Storage                 | In-memory only, `shared_preferences` (JSON), local database         |                                                                                                       |                                       |            |
| Audio package           | `just_audio`, `audioplayers`                                        |                                                                                                       |                                       |            |
| Error style in services | Exceptions, result objects                                          |                                                                                                       |                                       |            |
| Coin amounts            | See [04-services-and-rules.md](04-services-and-rules.md#coin-rules) | Focus 1 per minute (25 coins per 25-min session, claimed per session with rewardClaimed lock; 5-min break earns 0). Tasks/assignments earn 0 to prevent checkbox farming. | Focus earns most, task farming prevented | 2026-10-30 |
| Flutter version         |                                                                     |                                                                                                       |                                       |            |
