# 1️⃣ Folder Structure

Inside your app:

```
main_app/
    models.py
    views.py
    serializers.py
    fixtures/
        muscle_groups.json
        exercises.json
```

---

# 2️⃣ `main_app/fixtures/muscle_groups.json`

```json
[
  {
    "model": "main_app.musclegroup",
    "pk": 1,
    "fields": {
      "name": "Chest",
      "description": "Pectoral muscles."
    }
  },
  {
    "model": "main_app.musclegroup",
    "pk": 2,
    "fields": {
      "name": "Back",
      "description": "Lats, traps, and spinal erectors."
    }
  },
  {
    "model": "main_app.musclegroup",
    "pk": 3,
    "fields": {
      "name": "Shoulders",
      "description": "Deltoids."
    }
  },
  {
    "model": "main_app.musclegroup",
    "pk": 4,
    "fields": {
      "name": "Biceps",
      "description": "Elbow flexors."
    }
  },
  {
    "model": "main_app.musclegroup",
    "pk": 5,
    "fields": {
      "name": "Triceps",
      "description": "Elbow extensors."
    }
  },
  {
    "model": "main_app.musclegroup",
    "pk": 6,
    "fields": {
      "name": "Quads",
      "description": "Quadriceps femoris."
    }
  },
  {
    "model": "main_app.musclegroup",
    "pk": 7,
    "fields": {
      "name": "Hamstrings",
      "description": "Posterior thigh muscles."
    }
  },
  {
    "model": "main_app.musclegroup",
    "pk": 8,
    "fields": {
      "name": "Glutes",
      "description": "Gluteal muscles."
    }
  },
  {
    "model": "main_app.musclegroup",
    "pk": 9,
    "fields": {
      "name": "Calves",
      "description": "Lower leg muscles."
    }
  },
  {
    "model": "main_app.musclegroup",
    "pk": 10,
    "fields": {
      "name": "Core",
      "description": "Abdominals and stabilizers."
    }
  }
]
```

---

# 3️⃣ `main_app/fixtures/exercises.json`

```json
[
  {
    "model": "main_app.exercise",
    "pk": 1,
    "fields": {
      "name": "Back Squat",
      "exercise_type": "strength",
      "equipment": "barbell",
      "instructions": "Brace core, squat to depth, drive upward.",
      "video_url": "",
      "muscle_groups": [6, 8, 10]
    }
  },
  {
    "model": "main_app.exercise",
    "pk": 2,
    "fields": {
      "name": "Bench Press",
      "exercise_type": "strength",
      "equipment": "barbell",
      "instructions": "Lower bar to chest, press to lockout.",
      "video_url": "",
      "muscle_groups": [1, 3, 5]
    }
  },
  {
    "model": "main_app.exercise",
    "pk": 3,
    "fields": {
      "name": "Deadlift",
      "exercise_type": "strength",
      "equipment": "barbell",
      "instructions": "Hinge at hips, keep bar close, stand tall.",
      "video_url": "",
      "muscle_groups": [2, 7, 8, 10]
    }
  },
  {
    "model": "main_app.exercise",
    "pk": 4,
    "fields": {
      "name": "Pull-Up",
      "exercise_type": "strength",
      "equipment": "bodyweight",
      "instructions": "Pull chin over bar with controlled descent.",
      "video_url": "",
      "muscle_groups": [2, 4]
    }
  },
  {
    "model": "main_app.exercise",
    "pk": 5,
    "fields": {
      "name": "Overhead Press",
      "exercise_type": "strength",
      "equipment": "barbell",
      "instructions": "Press bar overhead while keeping core tight.",
      "video_url": "",
      "muscle_groups": [3, 1, 10]
    }
  },
  {
    "model": "main_app.exercise",
    "pk": 6,
    "fields": {
      "name": "Bicep Curl",
      "exercise_type": "strength",
      "equipment": "dumbbell",
      "instructions": "Curl dumbbells upward with controlled motion.",
      "video_url": "",
      "muscle_groups": [4]
    }
  },
  {
    "model": "main_app.exercise",
    "pk": 7,
    "fields": {
      "name": "Tricep Dip",
      "exercise_type": "strength",
      "equipment": "bodyweight",
      "instructions": "Lower body using arms, press back up.",
      "video_url": "",
      "muscle_groups": [5, 1, 3]
    }
  },
  {
    "model": "main_app.exercise",
    "pk": 8,
    "fields": {
      "name": "Walking Lunge",
      "exercise_type": "strength",
      "equipment": "bodyweight",
      "instructions": "Step forward and lower knee toward floor.",
      "video_url": "",
      "muscle_groups": [6, 7, 8, 10]
    }
  },
  {
    "model": "main_app.exercise",
    "pk": 9,
    "fields": {
      "name": "Plank",
      "exercise_type": "strength",
      "equipment": "bodyweight",
      "instructions": "Hold a rigid plank position with neutral spine.",
      "video_url": "",
      "muscle_groups": [10]
    }
  },
  {
    "model": "main_app.exercise",
    "pk": 10,
    "fields": {
      "name": "Running",
      "exercise_type": "cardio",
      "equipment": "none",
      "instructions": "Steady-state run.",
      "video_url": "",
      "muscle_groups": [6, 7, 9]
    }
  }
]
```

---

# 4️⃣ Load the data

Run:

```bash
python manage.py loaddata muscle_groups
python manage.py loaddata exercises
```

or:

```bash
python manage.py loaddata muscle_groups exercises
```

---

# 5️⃣ Verify in Django shell

```python
python manage.py shell
```

```python
from main_app.models import Exercise, MuscleGroup

Exercise.objects.count()
MuscleGroup.objects.count()
```

---

Think about generating a **much larger exercise library (~120 exercises)** with:

* strength
* cardio
* bodyweight
* machines
* Olympic lifts
* isolation movements
