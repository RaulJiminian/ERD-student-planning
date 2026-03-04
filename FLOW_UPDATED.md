The key idea is **templates/plans define structure**, while **workouts are the actual calendar instances** you perform/log.

---

# 1️⃣ What Templates Represent

A **WorkoutTemplate** is a reusable routine.

Example:

**Template: “Leg Day”**

```
1. Back Squat — 4x8
2. Walking Lunges — 3x10
3. Plank — 3x60s
```

Templates are:

* reusable
* shareable
* editable
* not tied to dates

So copying a template only creates **structure**, not scheduled workouts.

---

# 2️⃣ What Plans Represent

A **WorkoutPlan** is a **schedule + program structure**.

Example:

**Plan: “Beginner 2 Day Split”**

```
Monday → Upper Body Template
Thursday → Leg Day Template
```

The plan tells us:

* WHEN workouts should occur
* WHICH template to use

But a plan itself still does **not create workouts**.

---

# 3️⃣ What Workouts Represent

A **Workout** is a **real scheduled instance**.

Example:

```
Workout
Date: March 6
Template: Leg Day
Status: Planned
```

Inside that workout we copy the template items:

```
WorkoutItem
1. Squat
2. Lunges
3. Plank
```

Now the user can:

* log weight
* log reps
* mark completed
* skip

This is the **calendar data**.

---

# 4️⃣ The Typical Flow in the App

### Step 1 — User copies a public template

```
Public Template
    ↓
User Template
```

We create a new `WorkoutTemplate` owned by the user.

---

### Step 2 — User copies or creates a plan

```
WorkoutPlan
```

Example:

```
Monday → Upper Template
Thursday → Leg Template
```

This uses `WorkoutTemplatePlan`.

---

### Step 3 — Generate workouts from the plan

Now we run the generator.

```
WorkoutPlan
   ↓
Generate workouts
   ↓
Workout instances
```

Example result:

```
Mar 3 → Upper Body
Mar 6 → Leg Day
Mar 10 → Upper Body
Mar 13 → Leg Day
```

These are stored as **Workout rows**.

---

### Step 4 — Copy template items into workout items

For each generated workout:

```
WorkoutTemplateItem
     ↓ copy
WorkoutItem
```

So now each scheduled workout contains its exercises.

---

# 5️⃣ Why This Separation Is Important

This design prevents a major problem:

If templates changed, you wouldn't want to mutate historical workouts.

Example:

```
Template originally:
Squat 4x8

User changes template:
Squat 5x5
```

Old workouts should remain **4x8**, not magically change.

That’s why we **copy template items into workout items**.

---

# 6️⃣ Visual Architecture

```
Exercise
    ↑
WorkoutTemplateItem
    ↑
WorkoutTemplate
    ↑
WorkoutTemplatePlan
    ↑
WorkoutPlan
    ↓
Workout
    ↓
WorkoutItem
```

Templates define structure.

Workouts define real scheduled instances.

---

# 7️⃣ What happens when a user first enters the app?

You have two options:

### Option A — Let them browse templates/plans

```
Explore → Copy Template
Explore → Copy Plan
```

Then they generate workouts.

---

### Option B — Provide starter content

Seed:

* public templates
* public plans

Then the user presses:

```
"Start This Plan"
```

which:

1️⃣ copies the plan
2️⃣ copies templates
3️⃣ generates workouts

---

# 8️⃣ The Most Natural UX Flow

This is what apps like **Strong / Fitbod / Trainerize** do.

```
Browse Programs
      ↓
Select Program
      ↓
Start Program
      ↓
Generate Workouts
      ↓
Calendar populated
```

---

# 9️⃣ What your frontend will likely do

When the user presses:

```
Start Plan
```

Your frontend calls something like:

```
POST /plans/{id}/start/
```

Backend:

```
copy plan
copy templates
generate workouts
```

Then the calendar fills.

---

# 🔟 Key takeaway

You summarized it perfectly:

> Copying templates/plans creates structure.
> Generating workouts creates the actual calendar events.

Those two steps are intentionally separate.

---

Visiaul Diagram:

```
                EXERCISE LIBRARY
              (Seeded Reference Data)

           ┌────────────────────────┐
           │      MuscleGroup       │
           │  (Chest, Back, etc.)  │
           └──────────┬─────────────┘
                      │  M:N
           ┌──────────▼─────────────┐
           │        Exercise        │
           │  Squat, Bench, Run    │
           └──────────┬─────────────┘
                      │
                      │
                      │ used inside
                      │
           ┌──────────▼─────────────┐
           │   WorkoutTemplateItem  │
           │ sets, reps, order      │
           └──────────┬─────────────┘
                      │
           ┌──────────▼─────────────┐
           │     WorkoutTemplate    │
           │  "Leg Day", "Push"     │
           └──────────┬─────────────┘


                 PROGRAM LAYER
            (Structure + Scheduling)

           ┌──────────▼─────────────┐
           │  WorkoutTemplatePlan   │
           │  day_of_week, order    │
           └──────────┬─────────────┘
                      │
           ┌──────────▼─────────────┐
           │       WorkoutPlan      │
           │  Recurring schedule    │
           │  (Mon/Thu etc.)        │
           └──────────┬─────────────┘


                 CALENDAR LAYER
          (Actual workouts the user performs)

           ┌──────────▼─────────────┐
           │        Workout         │
           │  scheduled_start      │
           │  status               │
           └──────────┬─────────────┘
                      │
           ┌──────────▼─────────────┐
           │       WorkoutItem      │
           │  exercise, reps, wt   │
           └────────────────────────┘
```
