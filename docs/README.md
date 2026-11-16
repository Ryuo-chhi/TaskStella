# TaskStella Developer Guide

TaskStella is a gamified study planner for high school and university students. Students plan tasks and assignments, focus with ambient sound, earn coins, and spend coins on sounds, theme backgrounds, and Companions.

This guide is for the two developers building the app. It explains **what** to build and **why**, not the code itself. Every line of Dart in this project is written by the team.

## Course rules that shape every decision

These come from the Mobile Dev Term 1 project description. They override any other document in this repository.

| Rule                          | What it means for us                                                                                                            |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Your code must be your own    | No AI-generated code in the submission. Docs describe behavior, the team writes the code.                                       |
| 100% offline                  | No backend, no APIs, no network calls. Sounds, Companions and themes are bundled assets.                                        |
| Every member contributes      | Both members commit their own work and can explain it. See [10-team-workflow.md](10-team-workflow.md).                          |
| No state management framework | Only `StatefulWidget`, `setState`, callbacks and `Navigator`.                                                                   |
| Layered architecture          | UI, models, services and data are separate. Business logic stays out of screens.                                                |
| Minimums                      | 4+ entities, 2+ services with real rules, 1+ operation across several entities, a test for every service operation, 4+ screens. |

## Where to start

1. [01-setup.md](01-setup.md): install Flutter and run the app.
2. [02-architecture.md](02-architecture.md): understand the layers before writing any file.
3. [03-domain-model.md](03-domain-model.md) and [04-services-and-rules.md](04-services-and-rules.md): build these first. They need no UI.
4. [05-testing.md](05-testing.md): write tests together with each service.
5. [06-ui-and-screens.md](06-ui-and-screens.md), [07-design-tokens.md](07-design-tokens.md), [08-assets.md](08-assets.md): build the screens.
6. [09-milestones.md](09-milestones.md): check what each milestone interview expects.

## Other documents in the repository

| File                                      | Use it for                                                                                                                                                                                |
| ----------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `TaskStella-Design-Spec.md`               | Screen layouts, colors, navigation flow. The design source of truth.                                                                                                                      |
| `UI-QA-Audit-Prompt.md`                   | Visual checklist before a milestone demo.                                                                                                                                                 |
| `AGENTS.md`, `TaskStella-Build-Phases.md` | Written for AI agents. Use only as design reference, not as a way to generate code. Where they disagree with this guide (feature folders, packages), this guide follows the course rules. |

## Decisions still open

Track these in [02-architecture.md](02-architecture.md#open-decisions) and update the docs when decided.

- Local storage: in-memory only, `shared_preferences`, or a local database.
- Audio package for ambient sounds.
