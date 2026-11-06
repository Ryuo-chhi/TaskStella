# 09 Milestones

What each milestone needs, which doc supports it, and the kind of questions to prepare for. Teachers grade how the solution was engineered and whether you can explain it, not the number of features.

## Timeline

| Week | Milestone | Deliver | Docs |
|---|---|---|---|
| 5 | M1 Proposal | Goal and users, use case diagram, user stories with acceptance criteria | `TaskStella-Proposal.pdf` (done) |
| 6 | M2 Model, Services and Tests | Domain model (class diagram), services, unit tests | 03, 04, 05 |
| 9 | M3 UI Design and Widgets | Manual Figma mockups, widget architecture | 06, 07, 08, design spec |
| 12 | M4 State, Data and Interaction | User interactions, app state with `setState`, data handling | 02, 06 |
| 14 | Final Jury | Demo of the final product | All |

## Checklist per milestone

### M2: Model, Services and Tests
- [ ] Class diagram with at least 4 entities and their relationships
- [ ] At least 2 services, plain Dart, no Flutter imports
- [ ] At least 1 multi-entity operation (completeTask, toggleSubtask, completeSession or purchase)
- [ ] Every service operation has normal, edge and invalid tests
- [ ] `flutter test` passes, `flutter analyze` clean
- [ ] Both members have commits in models, services and tests

### M3: UI Design and Widgets
- [ ] Figma mockups made by hand for at least 4 screens
- [ ] Widget tree sketch for each main screen showing stateless and stateful widgets
- [ ] Reusable widgets list (see [06-ui-and-screens.md](06-ui-and-screens.md))
- [ ] Theme tokens in place, no hard-coded colors in widgets

### M4: State, Data and Interaction
- [ ] Screens call services and refresh with `setState`
- [ ] Callbacks pass events from widgets up to screens
- [ ] Navigation follows the spec flow table
- [ ] Invalid input shows clear errors in the UI
- [ ] Storage decision implemented (or in-memory, explained)

### Final Jury
- [ ] A rehearsed demo path covering US-01 to US-04
- [ ] Each member demonstrates and explains their own code

## Questions to prepare for

From the course description, expect questions like these. Each member should be able to answer them for their own part.

- How did you engineer the solution to satisfy this requirement?
- Why did you choose this design?
- What happens if the data is missing, invalid or changed?
- How did you make your business logic testable?

TaskStella-specific practice questions:

- How do you stop a task from awarding coins twice? Show the test.
- What happens if a purchase fails halfway? Why can't the balance go negative?
- Why is the timer in the screen but the reward logic in FocusService?
- How does the coin balance on Home update after a purchase in the Shop?
- What changes if you switch from in-memory storage to saved data?
- Why are colors named by role instead of by color?
