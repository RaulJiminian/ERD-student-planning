# Workout App (Django + DRF + React) — Master API + Frontend Flow (README)

This document is the single source of truth for:

- What each **frontend screen** calls on the API
- What **data payloads** you need for every **POST/PATCH**
- The end-to-end **user journey** from login → templates/plans → **generated workouts** → calendar
- A draft DRF **generate workouts** endpoint: `POST /workout-plans/:id/generate/`

---

## Table of Contents

1. [Core Concepts](#core-concepts)
2. [Frontend Screens → API Calls](#frontend-screens--api-calls)
3. [POST/PATCH Payload Cheat Sheet](#postpatch-payload-cheat-sheet)
4. [User Journey](#user-journey)
5. [Generate Workouts Endpoint](#generate-workouts-endpoint)
6. [Suggested Postman Test Flow](#suggested-postman-test-flow)

---

## Core Concepts

### Seeded Reference Library (read-only)
- `MuscleGroup` (seeded)
- `Exercise` (seeded)
- `Exercise ↔ MuscleGroup` is Many-to-Many

### Templates (Reusable routines)
- `WorkoutTemplate`: shareable/copyable routine container
- `WorkoutTemplateItem`: one exercise per item + prescription info (sets/reps/weight/etc)

### Plans (Schedule + composition)
- `WorkoutPlan`: recurrence rule + program container
- `WorkoutTemplatePlan`: through model linking plan ↔ templates (e.g., template on Monday)

### Workouts (Calendar instances)
- `Workout`: a dated/scheduled calendar event that the user completes/skips
- `WorkoutItem`: items inside a workout instance (copied from template items at generation time)

**Key rule:**  
Templates/Plans define structure and schedule.  
**Workouts are what populate the calendar.**

---

## Frontend Screens → API Calls

### Auth
**Login Screen**
- `POST /users/login/`

**Signup Screen**
- `POST /users/register/`

**App Boot / Refresh Session**
- `GET /users/token/refresh/`

---

### Exercise Library
**Exercise Library**
- `GET /exercises/`
- (optional filters) `GET /muscle-groups/`

**Exercise Detail**
- `GET /exercises/:id/`

---

### Templates
**Templates List**
- `GET /workout-templates/`

**Template Builder (Create / Edit)**
- Create: `POST /workout-templates/` (nested `items`)
- Update: `PATCH /workout-templates/:id/` (nested `items` replaces existing)
- Delete: `DELETE /workout-templates/:id/` (optional)

**Template Detail**
- `GET /workout-templates/:id/`
- Copy (frontend-driven): `POST /workout-templates/` with copied items

---

### Plans
**Plans List**
- `GET /workout-plans/`

**Plan Builder (Create / Edit)**
- Create: `POST /workout-plans/` (nested `template_links`)
- Update: `PATCH /workout-plans/:id/` (nested `template_links` replaces existing)
- Delete: `DELETE /workout-plans/:id/` (optional)

**Plan Detail**
- `GET /workout-plans/:id/`
- Generate: `POST /workout-plans/:id/generate/` ✅ (custom action, below)

---

### Calendar + Workouts
**Calendar Screen**
- `GET /workouts/?start=ISO_DATETIME&end=ISO_DATETIME`

**Workout Detail**
- `GET /workouts/:id/`
- `PATCH /workouts/:id/` (status/notes/time)
- (optional) edit items:
  - easiest: `PATCH /workouts/:id/` with nested `items` (replaces)
  - or item-by-item:
    - `POST /workout-items/`
    - `PATCH /workout-items/:id/`
    - `DELETE /workout-items/:id/`

**One-off Workout Create**
- `POST /workouts/` (nested `items`)

---

## POST/PATCH Payload Cheat Sheet

### Create Template
`POST /workout-templates/`

```json
{
  "title": "Leg Day",
  "description": "Lower body basics",
  "is_public": false,
  "source_template": null,
  "items": [
    { "exercise": 1, "order": 1, "sets": 4, "reps": 8, "weight": "185.00", "weight_unit": "lb" },
    { "exercise": 8, "order": 2, "sets": 3, "reps": 10 },
    { "exercise": 9, "order": 3, "sets": 3, "duration_seconds": 60, "notes": "3 x 60s" }
  ]
}
````

### Update Template

`PATCH /workout-templates/:id/`
If you include `items`, current items are replaced.

```json
{
  "title": "Leg Day (Updated)",
  "items": [{ "exercise": 1, "order": 1, "sets": 5, "reps": 5 }]
}
```

---

### Create Plan (with attached templates)

`POST /workout-plans/`

```json
{
  "title": "Beginner 2-Day Split",
  "start_datetime": "2026-03-05T18:00:00-05:00",
  "timezone": "America/New_York",
  "frequency": "weekly",
  "interval": 1,
  "by_weekdays": ["MO", "TH"],
  "until": "2026-06-01T00:00:00-05:00",
  "is_active": true,
  "template_links": [
    { "template": 2, "day_of_week": "MO", "order": 1 },
    { "template": 1, "day_of_week": "TH", "order": 2 }
  ]
}
```

### Update Plan

`PATCH /workout-plans/:id/`
If you include `template_links`, links are replaced.

```json
{
  "is_active": true,
  "template_links": [
    { "template": 2, "day_of_week": "MO", "order": 1 },
    { "template": 1, "day_of_week": "TH", "order": 2 }
  ]
}
```

---

### Create One-off Workout (calendar instance)

`POST /workouts/`

```json
{
  "plan": null,
  "template": null,
  "title": "Custom Workout",
  "scheduled_start": "2026-03-10T18:00:00-05:00",
  "status": "planned",
  "items": [
    { "exercise": 1, "order": 1, "sets": 3, "reps": 10 }
  ]
}
```

### Update Workout Status

`PATCH /workouts/:id/`

```json
{
  "status": "completed",
  "notes": "Felt strong today."
}
```

---

## User Journey

### 1) Login

* User logs in → token stored
* App can now access protected resources

### 2) Browse Library (seeded)

* User views `Exercises` and `MuscleGroups` (read-only)

### 3) Create or Copy Templates

* User creates a `WorkoutTemplate` with `WorkoutTemplateItems`
* Or copies public templates into their account

### 4) Create or Copy Plans

* User creates a `WorkoutPlan` with schedule fields
* User attaches templates via `WorkoutTemplatePlan` links (e.g., Monday = Upper)

### 5) Generate Workouts (critical step)

* Plan does NOT populate calendar automatically unless you generate instances
* The user clicks **Generate next 8 weeks**
* Backend creates:

  * `Workout` rows for each date
  * `WorkoutItem` rows copied from template items

### 6) Calendar Populated

* React calendar queries:

  * `GET /workouts/?start=...&end=...`

### 7) Logging / Completion

* User opens a workout → updates weights/reps → marks complete

---

## Generate Workouts Endpoint

### Desired route

`POST /workout-plans/:id/generate/`

### Behavior (student-friendly defaults)

* Generates workout instances for the next `horizon_weeks` (default 8)
* Uses `WorkoutTemplatePlan.day_of_week` to decide which template runs on which weekday
* Avoids duplicates (won’t create a workout for a datetime that already exists for that plan + template)
* Copies `WorkoutTemplateItem` → `WorkoutItem`

### Request body

Optional (safe defaults exist):

```json
{
  "horizon_weeks": 8,
  "replace_future_planned": false
}
```

* `replace_future_planned=false` means: keep existing planned workouts; only create missing ones
* If you set it to true, you can delete planned future workouts in that horizon and regenerate

---

## Draft Implementation (DRF)

### 1) Add a generator service

Create: `main_app/services/workout_generator.py`

```python
from __future__ import annotations

from datetime import timedelta

from django.db import transaction
from django.utils import timezone

from main_app.models import Workout, WorkoutItem, WorkoutPlan, WorkoutTemplatePlan


WEEKDAY_ORDER = ["MO", "TU", "WE", "TH", "FR", "SA", "SU"]


def _weekday_code(dt) -> str:
    # Monday=0 ... Sunday=6
    return WEEKDAY_ORDER[dt.weekday()]


@transaction.atomic
def generate_workouts_for_plan(
    plan: WorkoutPlan,
    *,
    horizon_weeks: int = 8,
    replace_future_planned: bool = False,
) -> dict:
    """
    Generates Workout + WorkoutItem instances for a plan for the next horizon_weeks.

    Assumptions (MVP):
    - frequency="weekly" is the primary supported mode
    - WorkoutTemplatePlan.day_of_week determines which template runs on which weekday
    - scheduled_start uses the plan.start_datetime's time-of-day

    Returns a summary dict suitable for API response.
    """
    if not plan.is_active:
        return {"created_workouts": 0, "created_items": 0, "skipped_existing": 0}

    start_dt = plan.start_datetime

    now = timezone.now()
    window_start = max(start_dt, now)
    window_end = window_start + timedelta(weeks=horizon_weeks)

    # Optional stop condition (until)
    if plan.until:
        window_end = min(window_end, plan.until)

    if window_end <= window_start:
        return {"created_workouts": 0, "created_items": 0, "skipped_existing": 0}

    # Build a map: weekday -> list of (template, link_order)
    links = WorkoutTemplatePlan.objects.filter(plan=plan).select_related("template").order_by("order", "id")
    by_day = {}
    for link in links:
        if not link.day_of_week:
            # If day_of_week is missing, skip in MVP (or treat as "any day" if you want).
            continue
        by_day.setdefault(link.day_of_week, []).append(link)

    # If replace mode: delete future planned workouts in window for this plan (only planned)
    if replace_future_planned:
        Workout.objects.filter(
            user=plan.user,
            plan=plan,
            status=Workout.Status.PLANNED,
            scheduled_start__gte=window_start,
            scheduled_start__lt=window_end,
        ).delete()

    created_workouts = 0
    created_items = 0
    skipped_existing = 0

    # Iterate day-by-day in the horizon
    cursor = window_start
    while cursor < window_end:
        day_code = _weekday_code(cursor)
        todays_links = by_day.get(day_code, [])
        if not todays_links:
            cursor += timedelta(days=1)
            continue

        # schedule at plan start time-of-day
        scheduled_start = cursor.replace(
            hour=start_dt.hour,
            minute=start_dt.minute,
            second=start_dt.second,
            microsecond=0,
        )

        for link in todays_links:
            template = link.template

            # Avoid duplicates: same plan+template+scheduled_start
            exists = Workout.objects.filter(
                user=plan.user,
                plan=plan,
                template=template,
                scheduled_start=scheduled_start,
            ).exists()

            if exists:
                skipped_existing += 1
                continue

            workout = Workout.objects.create(
                user=plan.user,
                plan=plan,
                template=template,
                title=template.title,
                scheduled_start=scheduled_start,
                status=Workout.Status.PLANNED,
            )
            created_workouts += 1

            # Copy template items -> workout items
            template_items = template.items.all().order_by("order", "id")
            items_to_create = []
            for ti in template_items:
                items_to_create.append(
                    WorkoutItem(
                        workout=workout,
                        exercise=ti.exercise,
                        order=ti.order,
                        sets=ti.sets,
                        reps=ti.reps,
                        weight=ti.weight,
                        weight_unit=ti.weight_unit,
                        duration_seconds=ti.duration_seconds,
                        distance_meters=ti.distance_meters,
                        rpe=ti.rpe,
                        notes=ti.notes,
                    )
                )

            if items_to_create:
                WorkoutItem.objects.bulk_create(items_to_create)
                created_items += len(items_to_create)

        cursor += timedelta(days=1)

    return {
        "created_workouts": created_workouts,
        "created_items": created_items,
        "skipped_existing": skipped_existing,
    }
```

---

### 2) Add a ViewSet action

In `main_app/views.py`, inside your `WorkoutPlanViewSet`:

```python
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework import status
from rest_framework.exceptions import ValidationError

from main_app.models import WorkoutPlan
from main_app.services.workout_generator import generate_workouts_for_plan


class WorkoutPlanViewSet(viewsets.ModelViewSet):
    # ... existing code ...

    @action(detail=True, methods=["post"], url_path="generate")
    def generate(self, request, pk=None):
        """
        POST /workout-plans/:id/generate/
        Body (optional): { "horizon_weeks": 8, "replace_future_planned": false }
        """
        plan: WorkoutPlan = self.get_object()

        horizon_weeks = request.data.get("horizon_weeks", 8)
        replace_future_planned = request.data.get("replace_future_planned", False)

        try:
            horizon_weeks = int(horizon_weeks)
        except (TypeError, ValueError):
            raise ValidationError({"horizon_weeks": "Must be an integer."})

        if horizon_weeks < 1 or horizon_weeks > 52:
            raise ValidationError({"horizon_weeks": "Must be between 1 and 52."})

        if not isinstance(replace_future_planned, bool):
            # If Postman sends "true"/"false" as string, you can normalize here if needed.
            replace_future_planned = str(replace_future_planned).lower() == "true"

        summary = generate_workouts_for_plan(
            plan,
            horizon_weeks=horizon_weeks,
            replace_future_planned=replace_future_planned,
        )

        return Response(
            {
                "plan_id": plan.id,
                "horizon_weeks": horizon_weeks,
                **summary,
            },
            status=status.HTTP_200_OK,
        )
```

**Good news:** you do NOT need to add a new route in `urls.py` when using DRF routers + `@action`.
The router will automatically expose `/workout-plans/:id/generate/`.

---

## Suggested Postman Test Flow

1. Login

* `POST /users/login/`

2. Load exercise library

* `GET /exercises/`

3. Create template

* `POST /workout-templates/`

4. Create plan (attach template to weekday)

* `POST /workout-plans/`

5. Generate workouts

* `POST /workout-plans/:id/generate/`

  * Body: `{ "horizon_weeks": 8 }`

6. Verify calendar data exists

* `GET /workouts/?start=2026-03-01T00:00:00-05:00&end=2026-04-01T00:00:00-05:00`

7. Mark a workout complete

* `PATCH /workouts/:id/` with `{ "status": "completed" }`

---

## Notes / MVP Constraints (explicit)

* The generator above is intentionally MVP-focused:

  * It primarily supports weekly plans driven by `WorkoutTemplatePlan.day_of_week`
  * It schedules workouts at the time-of-day from `plan.start_datetime`
  * It avoids duplicates and can optionally “replace future planned workouts” within the horizon
  * This is perfect for a student project and covers the calendar use-case cleanly.


Next step:
Consider also adding a “Copy Plan / Start Plan” flow (single button) (e.g. `POST /workout-plans/:id/start/` that copies + generates in one shot).
