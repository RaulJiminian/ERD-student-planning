## Folder + load order

Put these in:

```
main_app/fixtures/
  templates_and_items.json
  plans.json
```

Load in this order:

```bash
python manage.py loaddata muscle_groups.json
python manage.py loaddata exercises.json
python manage.py loaddata templates_and_items.json
python manage.py loaddata plans.json
```

## 1) `main_app/fixtures/templates_and_items.json`

```json
[
  {
    "model": "main_app.workouttemplate",
    "pk": 101,
    "fields": {
      "user": 1,
      "title": "Upper Body Push (Beginner)",
      "description": "Chest/shoulders/triceps basics.",
      "is_public": true,
      "parent": null,
      "duration_minutes": 45
    }
  },
  {
    "model": "main_app.workouttemplate",
    "pk": 102,
    "fields": {
      "user": 1,
      "title": "Upper Body Pull (Beginner)",
      "description": "Back/biceps basics.",
      "is_public": true,
      "parent": null,
      "duration_minutes": 45
    }
  },
  {
    "model": "main_app.workouttemplate",
    "pk": 103,
    "fields": {
      "user": 1,
      "title": "Lower Body (Beginner)",
      "description": "Quads/hamstrings/glutes basics.",
      "is_public": true,
      "parent": null,
      "duration_minutes": 55
    }
  },
  {
    "model": "main_app.workouttemplate",
    "pk": 104,
    "fields": {
      "user": 1,
      "title": "Core + Conditioning (Beginner)",
      "description": "Simple core + short cardio finisher.",
      "is_public": true,
      "parent": null,
      "duration_minutes": 30
    }
  },

  /* -------------------------------
     TEMPLATE 101 ITEMS (PUSH)
     Uses: Dips (Chest-Focused)=16, Overhead Press=30, Lateral Raise=33, Tricep Pushdown=46
   ------------------------------- */
  {
    "model": "main_app.workouttemplateitem",
    "pk": 1001,
    "fields": {
      "template": 101,
      "exercise": 16,
      "order": 1,
      "sets": 3,
      "reps": 8,
      "weight": null,
      "weight_unit": null,
      "duration_seconds": null,
      "distance_meters": null,
      "rpe": 7,
      "notes": "Scale with assistance if needed."
    }
  },
  {
    "model": "main_app.workouttemplateitem",
    "pk": 1002,
    "fields": {
      "template": 101,
      "exercise": 30,
      "order": 2,
      "sets": 3,
      "reps": 8,
      "weight": "65.00",
      "weight_unit": "lb",
      "duration_seconds": null,
      "distance_meters": null,
      "rpe": 7,
      "notes": "Overhead Press"
    }
  },
  {
    "model": "main_app.workouttemplateitem",
    "pk": 1003,
    "fields": {
      "template": 101,
      "exercise": 33,
      "order": 3,
      "sets": 3,
      "reps": 12,
      "weight": "15.00",
      "weight_unit": "lb",
      "duration_seconds": null,
      "distance_meters": null,
      "rpe": 6,
      "notes": "Lateral Raise"
    }
  },
  {
    "model": "main_app.workouttemplateitem",
    "pk": 1004,
    "fields": {
      "template": 101,
      "exercise": 46,
      "order": 4,
      "sets": 3,
      "reps": 12,
      "weight": null,
      "weight_unit": null,
      "duration_seconds": null,
      "distance_meters": null,
      "rpe": 7,
      "notes": "Tricep Pushdown"
    }
  },

  /* -------------------------------
     TEMPLATE 102 ITEMS (PULL)
     Uses: Pull-ups=17, Barbell Row=20, Lat Pulldown=19, Barbell Curl=38
   ------------------------------- */
  {
    "model": "main_app.workouttemplateitem",
    "pk": 1101,
    "fields": {
      "template": 102,
      "exercise": 17,
      "order": 1,
      "sets": 3,
      "reps": 6,
      "weight": null,
      "weight_unit": null,
      "duration_seconds": null,
      "distance_meters": null,
      "rpe": 8,
      "notes": "Scale if needed."
    }
  },
  {
    "model": "main_app.workouttemplateitem",
    "pk": 1102,
    "fields": {
      "template": 102,
      "exercise": 20,
      "order": 2,
      "sets": 3,
      "reps": 8,
      "weight": "95.00",
      "weight_unit": "lb",
      "duration_seconds": null,
      "distance_meters": null,
      "rpe": 7,
      "notes": "Barbell Row"
    }
  },
  {
    "model": "main_app.workouttemplateitem",
    "pk": 1103,
    "fields": {
      "template": 102,
      "exercise": 19,
      "order": 3,
      "sets": 3,
      "reps": 10,
      "weight": null,
      "weight_unit": null,
      "duration_seconds": null,
      "distance_meters": null,
      "rpe": 7,
      "notes": "Lat Pulldown"
    }
  },
  {
    "model": "main_app.workouttemplateitem",
    "pk": 1104,
    "fields": {
      "template": 102,
      "exercise": 38,
      "order": 4,
      "sets": 3,
      "reps": 12,
      "weight": "45.00",
      "weight_unit": "lb",
      "duration_seconds": null,
      "distance_meters": null,
      "rpe": 6,
      "notes": "Barbell Curl"
    }
  },

  /* -------------------------------
     TEMPLATE 103 ITEMS (LOWER)
     Uses: Back Squat=51, Leg Press=54, Romanian Deadlift=26, Hip Thrust=61, Standing Calf Raise=63
   ------------------------------- */
  {
    "model": "main_app.workouttemplateitem",
    "pk": 1201,
    "fields": {
      "template": 103,
      "exercise": 51,
      "order": 1,
      "sets": 3,
      "reps": 8,
      "weight": "135.00",
      "weight_unit": "lb",
      "duration_seconds": null,
      "distance_meters": null,
      "rpe": 7,
      "notes": "Back Squat"
    }
  },
  {
    "model": "main_app.workouttemplateitem",
    "pk": 1202,
    "fields": {
      "template": 103,
      "exercise": 54,
      "order": 2,
      "sets": 3,
      "reps": 10,
      "weight": null,
      "weight_unit": null,
      "duration_seconds": null,
      "distance_meters": null,
      "rpe": 7,
      "notes": "Leg Press"
    }
  },
  {
    "model": "main_app.workouttemplateitem",
    "pk": 1203,
    "fields": {
      "template": 103,
      "exercise": 26,
      "order": 3,
      "sets": 3,
      "reps": 10,
      "weight": "95.00",
      "weight_unit": "lb",
      "duration_seconds": null,
      "distance_meters": null,
      "rpe": 7,
      "notes": "Romanian Deadlift"
    }
  },
  {
    "model": "main_app.workouttemplateitem",
    "pk": 1204,
    "fields": {
      "template": 103,
      "exercise": 61,
      "order": 4,
      "sets": 3,
      "reps": 10,
      "weight": "135.00",
      "weight_unit": "lb",
      "duration_seconds": null,
      "distance_meters": null,
      "rpe": 7,
      "notes": "Hip Thrust"
    }
  },
  {
    "model": "main_app.workouttemplateitem",
    "pk": 1205,
    "fields": {
      "template": 103,
      "exercise": 63,
      "order": 5,
      "sets": 3,
      "reps": 12,
      "weight": null,
      "weight_unit": null,
      "duration_seconds": null,
      "distance_meters": null,
      "rpe": 6,
      "notes": "Standing Calf Raise"
    }
  },

  /* -------------------------------
     TEMPLATE 104 ITEMS (CORE + COND)
     Uses: Plank=68, Russian Twists=69, Treadmill Running=70
   ------------------------------- */
  {
    "model": "main_app.workouttemplateitem",
    "pk": 1301,
    "fields": {
      "template": 104,
      "exercise": 68,
      "order": 1,
      "sets": 3,
      "reps": null,
      "weight": null,
      "weight_unit": null,
      "duration_seconds": 45,
      "distance_meters": null,
      "rpe": 7,
      "notes": "3 x 45s plank"
    }
  },
  {
    "model": "main_app.workouttemplateitem",
    "pk": 1302,
    "fields": {
      "template": 104,
      "exercise": 69,
      "order": 2,
      "sets": 3,
      "reps": 20,
      "weight": null,
      "weight_unit": null,
      "duration_seconds": null,
      "distance_meters": null,
      "rpe": 6,
      "notes": "20 total twists (10 per side)"
    }
  },
  {
    "model": "main_app.workouttemplateitem",
    "pk": 1303,
    "fields": {
      "template": 104,
      "exercise": 70,
      "order": 3,
      "sets": 1,
      "reps": null,
      "weight": null,
      "weight_unit": null,
      "duration_seconds": 900,
      "distance_meters": null,
      "rpe": 6,
      "notes": "15 min steady run"
    }
  }
]
```

---

## 2) `main_app/fixtures/plans.json`

Because your `WorkoutPlan` no longer has weekdays, the simplest (and consistent) interpretation for seeding is:

* A plan contains an **ordered list** of templates via `WorkoutTemplatePlan.order`
* Each link also defines a **time-of-day** via `WorkoutTemplatePlan.time`
* The plan’s `start_dt` is when the plan begins
* Your generator can repeat the ordered sequence over time (example: order 1 = day 0, order 2 = day 1, order 3 = day 2, then repeat weekly / by interval)

Here are 2 starter plans:

```json
[
  {
    "model": "main_app.workoutplan",
    "pk": 201,
    "fields": {
      "user": 1,
      "title": "Beginner 3-Session Sequence",
      "start_dt": "2026-03-09T00:00:00Z",
      "interval": 1,
      "is_public": true
    }
  },
  {
    "model": "main_app.workoutplan",
    "pk": 202,
    "fields": {
      "user": 1,
      "title": "Beginner 2-Session Sequence",
      "start_dt": "2026-03-09T00:00:00Z",
      "interval": 1,
      "is_public": true
    }
  },

  /* ----------------------------
     THROUGH: WorkoutTemplatePlan
     Fields: plan, template, order, time
   ---------------------------- */

  {
    "model": "main_app.workouttemplateplan",
    "pk": 3001,
    "fields": { "plan": 201, "template": 101, "order": 1, "time": "18:00:00" }
  },
  {
    "model": "main_app.workouttemplateplan",
    "pk": 3002,
    "fields": { "plan": 201, "template": 102, "order": 2, "time": "18:00:00" }
  },
  {
    "model": "main_app.workouttemplateplan",
    "pk": 3003,
    "fields": { "plan": 201, "template": 103, "order": 3, "time": "18:00:00" }
  },

  {
    "model": "main_app.workouttemplateplan",
    "pk": 3004,
    "fields": { "plan": 202, "template": 101, "order": 1, "time": "18:30:00" }
  },
  {
    "model": "main_app.workouttemplateplan",
    "pk": 3005,
    "fields": { "plan": 202, "template": 103, "order": 2, "time": "18:30:00" }
  }
]
```
