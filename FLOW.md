Here’s a clean “what gets created in what order” flow that matches your schema (seeded catalog → templates → plans → generated workouts → logging).

## 0) One-time setup (admin / seed scripts)

1. **MuscleGroup** rows (seed)
2. **Exercise** rows (seed)
3. Link **Exercise ↔ MuscleGroup** (M2M)

That’s your global library.

---

## 1) User creates reusable content (templates)

### A) Create a WorkoutTemplate

1. **WorkoutTemplate** (title, description, is_public)

### B) Add items to the template

2. For each exercise the user adds:

   * **WorkoutTemplateItem**

     * FK → template
     * FK → exercise
     * sets/reps/weight/etc
     * order

At this point, the user has a shareable routine like “Legs + Core”.

---

## 2) User creates a recurring program (plan)

### A) Create WorkoutPlan

3. **WorkoutPlan**

   * title
   * start_datetime
   * frequency/interval/by_weekdays/until
   * is_active

### B) Attach templates to the plan

4. Create **WorkoutTemplatePlan** rows (your through model)

   * FK → plan
   * FK → template
   * (optional) day_of_week / week_index / order

Now the plan knows *what routines belong to it* and optionally *on which days*.

---

## 3) Generate scheduled workouts (instances for calendar)

5. When plan is created (or user clicks “Generate next 8 weeks”):

   * Compute upcoming dates from the recurrence rule
   * For each date, decide which template(s) should run that day
   * Create a **Workout** instance:

     * user
     * plan
     * template (the one used)
     * scheduled_start (+ optional end)
     * title
     * status = planned

6. Clone the template’s items into the workout:

   * For each **WorkoutTemplateItem** in that template:

     * create a **WorkoutItem**

       * FK → workout
       * FK → exercise
       * copy sets/reps/weight/etc
       * order

✅ Now the React calendar can just query `Workout` by date range.

---

## 4) User uses the app day-to-day

### Calendar view

7. React asks backend:

   * `GET /workouts?start=...&end=...`
   * show each **Workout** on the calendar

### Completing/logging

8. User opens a workout instance and logs what happened:

   * update **Workout.status** to `completed` (or skipped/canceled)
   * optionally edit **WorkoutItem** fields (actual reps/weight/etc)
   * save

---

## 5) Copying someone else’s template/plan (social feature)

### Copy a WorkoutTemplate

1. Create new **WorkoutTemplate** for the user

   * set `source_template = original`
2. Copy all **WorkoutTemplateItem** rows

### Copy a WorkoutPlan

1. Create new **WorkoutPlan**
2. Copy the **WorkoutTemplatePlan** rows (plan ↔ templates)
3. Generate workouts (same as step 3)

---

## 6) What happens when users edit stuff later (important rules)

### Editing a WorkoutTemplate

* Should **not** retroactively change existing workouts (history).
* It only affects workouts generated in the future.

### Editing a WorkoutPlan recurrence

* Most student-friendly rule:

  * Regenerate **future planned** workouts only
  * Leave completed workouts alone

---

### Cheat-sheet summary

**Seed → Template → TemplateItems → Plan → Plan↔Templates → Generate Workouts → Clone Items → Calendar + Log**
