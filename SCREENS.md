Here’s a clean set of **frontend screens** (and route patterns) that covers everything you’ve modeled + what you’ll hit in Postman, but in React.

## Core navigation buckets

* **Library (seeded data):** Exercises + muscle groups (read-only)
* **Build (user content):** Templates + Plans
* **Do (calendar + logging):** Workouts + workout items

---

## Auth + shell

1. **Login / Signup**

* `/login`
* `/signup`
* (optional) `/logout` action

2. **App Layout / Dashboard (optional)**

* `/` (or `/dashboard`)
* Quick links: “Next workout”, “This week”, “Create template”, etc.

---

## Exercise library screens (seeded)

3. **Exercise Library**

* `/exercises`
* Search + filters (type, muscle group, equipment)

4. **Exercise Detail**

* `/exercises/:exerciseId`
* Show muscle groups, instructions/video
* Buttons: “Add to template item” (opens modal/chooser)

5. **Muscle Groups**

* `/muscle-groups`
* (optional) **Muscle Group Detail**

  * `/muscle-groups/:id` (lists exercises that hit it)

---

## Template screens (share/copy/modify)

6. **Templates List**

* `/templates`
* Tabs or filters: “Mine” vs “Public”
* Actions: Create / Duplicate / Edit / Delete

7. **Template Builder**

* `/templates/new`
* `/templates/:templateId/edit`
* Add items (each item = exercise + sets/reps/etc), reorder list

8. **Template Detail**

* `/templates/:templateId`
* View-only + “Copy” button if public/other user

> These map nicely to:
>
> * `GET/POST /workout-templates/`
> * nested items via serializer
> * `GET /workout-template-items/` (if you expose it separately)

---

## Plan screens (recurrence + template attachments)

9. **Plans List**

* `/plans`
* Tabs: “Mine” vs “Public” (if you make plans shareable)
* Actions: Create / Edit / Pause / Duplicate

10. **Plan Builder**

* `/plans/new`
* `/plans/:planId/edit`
* Step-based UI works well:

  1. recurrence rule (weekly/daily, weekdays, start/until)
  2. attach templates (with `day_of_week` + ordering)
  3. generate (or auto-generate) workouts

11. **Plan Detail**

* `/plans/:planId`
* Show recurrence, attached templates, upcoming generated workouts
* Action: “Generate next 8 weeks” (if you add that endpoint)

> Maps to:
>
> * `GET/POST /workout-plans/`
> * through objects shown/edited as `template_links`

---

## Calendar + Workout execution screens (instances)

12. **Calendar**

* `/calendar`
* Month/week view
* Fetches workouts by date range:

  * `GET /workouts?start=...&end=...`
* Clicking an event navigates to workout detail

13. **Workout Detail (Log / Perform Workout)**

* `/workouts/:workoutId`
* Shows items list (one per exercise), allows edits
* Actions:

  * Mark completed / skipped / canceled
  * Edit sets/reps/weight/duration
  * Add/remove/reorder items (optional)

14. **Create/Edit One-off Workout**

* `/workouts/new`
* `/workouts/:workoutId/edit`
* Useful for workouts not tied to a plan

> Maps to:
>
> * `GET/POST/PATCH /workouts/`
> * nested `items` updates

---

## Social / Browse (optional but fits your “copy others” goal)

15. **Explore (Public Content Hub)**

* `/explore`
* Shows public templates and/or plans
* Routes:

  * `/explore/templates`
  * `/explore/plans`
* CTA: “Copy to my account”

---

## Profile / settings

16. **Profile**

* `/profile`
* Preferences (units lb/kg), timezone, etc.

---

# Minimal MVP screen set (if they need to scope down)

If time is tight, MVP can be:

* `/login`
* `/calendar`
* `/workouts/:id`
* `/templates` + `/templates/new|:id/edit`
* `/plans` + `/plans/new|:id/edit`
* `/exercises` (picker/search)

---

Potential calendar libraries: FullCalendar, React Big Calendar, MUI X Calendar
