# 🌟 TaskStella

**TaskStella** is a gamified mobile study planner designed for high school and university students. It helps students centralize their tasks and assignment deadlines, break overwhelming projects into manageable steps, and stay focused with timed study sessions—all while earning Star Coins to personalize their app experience.

---

## 👥 Team Information

- **Team:** 12
- **Members:** Tiek Chhunhour, Yeat David

---

## 🎯 Problem & Goal

### Who Has the Problem?

High school and university students who juggle multiple courses at once—each with its own assignments and deadlines—on top of part-time work, family duties, and other personal commitments.

### What the Problem Costs Them Today

- **Scattered Deadlines:** Assignment dates live across class chats, lecture slides, paper notes, and memory. Students often realize an assignment is due when it is too late to do it well, costing marks and causing last-minute stress.
- **Overwhelming Workload:** Big assignments feel too daunting to start. Without a structured way to break work into smaller steps, students procrastinate and rush the work in a single night.
- **Interrupted Study Time:** Students sit down to study but easily drift toward their phones, resulting in hours of "studying" with little real progress and no record of actual focused time.
- **Low Retention in Planner Apps:** Ordinary to-do apps offer no incentive to return after the first week, causing students to abandon them and fall back into unproductive habits.

### The TaskStella Solution

TaskStella keeps all tasks and assignment deadlines in one place, breaks big assignments into small subtasks, and tracks focused study time. To build lasting habits and keep students coming back, completed tasks and focus sessions award **Star Coins** that can be spent in a reward shop to customize and personalize the app.

---

## 🚀 Core Features (Must-Have Scope)

### 1. Daily Task Management (`US-01`)

Organize daily study activities and personal responsibilities.

- Create tasks with a title, priority level, and optional due date.
- Edit, complete, and delete tasks.
- Validation prevents blank or whitespace-only task titles.
- Safeguards ensure completing a task updates its status and never processes duplicate completion rewards.

### 2. Assignment & Deadline Tracking (`US-02`)

Prioritize upcoming coursework and tackle large projects step by step.

- Create assignments with a title, due date, priority, and optional description.
- Automatic status tracking: categorizes assignments as **Upcoming**, **Completed**, or **Overdue** based on status and due date.
- Break assignments down into actionable **subtasks** and check them off individually.
- Completing all subtasks leaves the main assignment open for final review rather than auto-completing it.
- Editing an assignment preserves completed subtasks unless explicitly removed.

### 3. Focus Sessions & Timer (`US-03`)

Concentrate on selected tasks and track actual study duration.

- Start, pause, resume, and complete timed focus sessions.
- Automatically records the duration of completed sessions (rejecting zero or negative durations).
- Cancelled or incomplete sessions do not grant full completion rewards.

### 4. Gamification & Reward Shop (`US-04`)

Stay motivated by earning and spending Star Coins.

- Earn **Star Coins** from completed tasks and focus sessions based on fixed rules (with anti-farming checks so the same activity cannot be rewarded twice).
- Browse the **Rewards Shop** and purchase items to personalize the app.
- Built-in wallet validation: rejects purchases if the coin balance is insufficient, prevents negative balances, and prevents buying duplicate items already owned.

---

## 🗺️ Functional Overview (Use-Case Summary)

The primary actor is the **User (Student)**, interacting across three main modules:

1. **Daily Planning:** View Daily Tasks, Create Task, Update/Delete Task, Complete Task.
2. **Focus & Productivity:** Start Focus Session → Complete Focus Session → Claim Session Reward.
3. **Gamification & Rewards:** Earn Star Coins (triggered by claiming completed focus mode sessions), Browse Rewards Shop, Purchase Reward Item.

---

## 🔮 Future Enhancements (Optional Scope)

_These features are outside the initial committed scope and will be developed once all core user stories are completed and tested:_

- 📚 **Courses:** Group assignments by specific courses.
- 🎧 **Soundscapes:** Play ambient background sounds during focus sessions, with additional sound packs available in the reward shop.
- 📊 **Focus History:** View historical focus sessions and total accumulated study time.
- 📝 **Notes:** Write short notes linked directly to a course or assignment.
- 🔔 **Reminders:** Receive push notifications before a task or assignment is due.

---
