Routes:

```
/muscle-groups/
/exercises/
/workout-templates/
/workout-plans/
/workout-template-plans/
/workouts/
/workout-items/
```

---

# services/workoutService.js

```javascript
import api from "./apiConfig.js";

/* =================================
   MUSCLE GROUPS
================================= */

export const getMuscleGroups = async () => {
  const resp = await api.get("/muscle-groups/");
  return resp.data;
};


/* =================================
   EXERCISES
================================= */

export const getExercises = async () => {
  const resp = await api.get("/exercises/");
  return resp.data;
};

export const getExerciseById = async (exerciseId) => {
  const resp = await api.get(`/exercises/${exerciseId}/`);
  return resp.data;
};


/* =================================
   WORKOUT TEMPLATES
================================= */

export const getTemplates = async () => {
  const resp = await api.get("/workout-templates/");
  return resp.data;
};

export const getTemplate = async (templateId) => {
  const resp = await api.get(`/workout-templates/${templateId}/`);
  return resp.data;
};

export const createTemplate = async (templateData) => {
  const resp = await api.post("/workout-templates/", templateData);
  return resp.data;
};

export const updateTemplate = async (templateId, templateData) => {
  const resp = await api.patch(`/workout-templates/${templateId}/`, templateData);
  return resp.data;
};

export const deleteTemplate = async (templateId) => {
  const resp = await api.delete(`/workout-templates/${templateId}/`);
  return resp.data;
};


/* =================================
   TEMPLATE ITEMS
================================= */

export const addTemplateItem = async (itemData) => {
  const resp = await api.post("/workout-template-items/", itemData);
  return resp.data;
};

export const updateTemplateItem = async (itemId, itemData) => {
  const resp = await api.patch(`/workout-template-items/${itemId}/`, itemData);
  return resp.data;
};

export const deleteTemplateItem = async (itemId) => {
  const resp = await api.delete(`/workout-template-items/${itemId}/`);
  return resp.data;
};


/* =================================
   WORKOUT PLANS
================================= */

export const getPlans = async () => {
  const resp = await api.get("/workout-plans/");
  return resp.data;
};

export const getPlan = async (planId) => {
  const resp = await api.get(`/workout-plans/${planId}/`);
  return resp.data;
};

export const createPlan = async (planData) => {
  const resp = await api.post("/workout-plans/", planData);
  return resp.data;
};

export const updatePlan = async (planId, planData) => {
  const resp = await api.patch(`/workout-plans/${planId}/`, planData);
  return resp.data;
};

export const deletePlan = async (planId) => {
  const resp = await api.delete(`/workout-plans/${planId}/`);
  return resp.data;
};


/* =================================
   PLAN TEMPLATE LINKS
================================= */

export const addTemplateToPlan = async (linkData) => {
  const resp = await api.post("/workout-template-plans/", linkData);
  return resp.data;
};

export const updatePlanTemplateLink = async (linkId, linkData) => {
  const resp = await api.patch(`/workout-template-plans/${linkId}/`, linkData);
  return resp.data;
};

export const removeTemplateFromPlan = async (linkId) => {
  const resp = await api.delete(`/workout-template-plans/${linkId}/`);
  return resp.data;
};


/* =================================
   WORKOUTS (CALENDAR)
================================= */

export const getWorkouts = async (start, end) => {
  const resp = await api.get("/workouts/", {
    params: {
      start,
      end,
    },
  });
  return resp.data;
};

export const getWorkout = async (workoutId) => {
  const resp = await api.get(`/workouts/${workoutId}/`);
  return resp.data;
};

export const createWorkout = async (workoutData) => {
  const resp = await api.post("/workouts/", workoutData);
  return resp.data;
};

export const updateWorkout = async (workoutId, workoutData) => {
  const resp = await api.patch(`/workouts/${workoutId}/`, workoutData);
  return resp.data;
};

export const deleteWorkout = async (workoutId) => {
  const resp = await api.delete(`/workouts/${workoutId}/`);
  return resp.data;
};


/* =================================
   WORKOUT ITEMS
================================= */

export const addWorkoutItem = async (itemData) => {
  const resp = await api.post("/workout-items/", itemData);
  return resp.data;
};

export const updateWorkoutItem = async (itemId, itemData) => {
  const resp = await api.patch(`/workout-items/${itemId}/`, itemData);
  return resp.data;
};

export const deleteWorkoutItem = async (itemId) => {
  const resp = await api.delete(`/workout-items/${itemId}/`);
  return resp.data;
};
```

---

# Example frontend usage

### Load exercises for a template builder

```javascript
const exercises = await getExercises();
```

---

### Create a template with items

```javascript
await createTemplate({
  title: "Leg Day",
  description: "Basic leg workout",
  is_public: false,
  items: [
    { exercise: 1, order: 1, sets: 4, reps: 8 },
    { exercise: 8, order: 2, sets: 3, reps: 10 }
  ]
});
```

---

### Load calendar workouts

```javascript
const workouts = await getWorkouts(
  "2026-03-01T00:00:00Z",
  "2026-04-01T00:00:00Z"
);
```

---

# Why this structure works well

You now have **service functions matching each backend resource**:

| Resource        | Endpoint                   |
| --------------- | -------------------------- |
| Exercises       | `/exercises/`              |
| Muscle Groups   | `/muscle-groups/`          |
| Templates       | `/workout-templates/`      |
| Plans           | `/workout-plans/`          |
| Plan ↔ Template | `/workout-template-plans/` |
| Workouts        | `/workouts/`               |
| Workout Items   | `/workout-items/`          |

This makes your frontend code **very predictable and easy to maintain**.

---

**Folder structure for services**:

```
services/
  exerciseService.js
  templateService.js
  planService.js
  workoutService.js
```
