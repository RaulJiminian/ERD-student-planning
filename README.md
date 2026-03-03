# ERD-student-planning

We will need to introduce a **WorkoutTemplate** layer.

---

## Updated Model split

### A) Calendar / logging (instances)

* **WorkoutPlan** (recurrence series)
* **Workout** (one dated instance on the calendar)
* **WorkoutItem** (exercise + sets/reps/etc for that instance)

✅ `WorkoutPlan (1) -> (many) Workout`

---

### B) Sharing / copying / composing programs (templates)

Add:

* **WorkoutTemplate** (a reusable routine like "Leg Raises + Core")
* **WorkoutTemplateItem** (one exercise per item, like your WorkoutItem but no date)

Then your "plans can include the same workout in multiple plans" becomes:

✅ `WorkoutPlan (many) <-> (many) WorkoutTemplate`

---

## Final schema that supports your goals

### 1) Seeded

* `Exercise`
* `MuscleGroup`
  (Exercise M2M MuscleGroup)

### 2) Templates for copying/sharing

#### `WorkoutTemplate`

* `user` (creator/owner)
* `title`
* `description` (optional)
* `is_public` (bool) (so others can browse/copy)
* `source_template` (FK to self, optional) (to track "copied from")
* timestamps

#### `WorkoutTemplateItem` (1 exercise each)

* `template` (FK)
* `exercise` (FK)
* `order`
* `sets`, `reps`, `weight`, etc (nullable like before)
* `notes` (optional)

### 3) Plans for recurrence + composition

#### `WorkoutPlan`

* `user`
* recurrence fields (`frequency`, `interval`, `by_weekdays`, `until`, etc.)
* `title`
* `is_public` (optional if you want to share plans too)
* timestamps

#### Link table: `PlanWorkoutTemplate` (through model)

This is where you attach templates to a plan, and optionally specify *when* they occur.

* `plan` (FK)
* `template` (FK)
* `day_of_week` (optional) (if weekly plan)
* `week_index` (optional) (for rotating schedules like 4-week programs)
* `order` (optional)

This "through" table is what makes plans flexible.

### 4) Actual scheduled instances (calendar)

#### `Workout`

* `user`
* `plan` (FK nullable)  ← keeps 1->many series relationship
* `template` (FK nullable) ← what it was generated from
* `scheduled_start`, `scheduled_end`
* `status`, `notes`, timestamps

#### `WorkoutItem`

* `workout` (FK)
* `exercise` (FK)
* `order`
* sets/reps/weight/duration/etc

**When generating workouts**, you copy items from `WorkoutTemplateItem` into `WorkoutItem`.

---

## Why this is the cleanest approach

* A **WorkoutPlan** is a scheduling rule + program structure.
* A **WorkoutTemplate** is reusable content (what users share/copy).
* A **Workout instance** is history + calendar.

This supports:

* browsing other users' public templates
* copying a template and modifying it
* including the same template in multiple plans (legs plan + full body plan)
* generating future calendar workouts from a plan/template combo
* preserving history if the template later changes (past workouts shouldn't mutate)
