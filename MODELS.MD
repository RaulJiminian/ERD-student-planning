```python
# models.py
from django.conf import settings
from django.db import models


class MuscleGroup(models.Model):
    name = models.CharField(max_length=80, unique=True)
    description = models.TextField(blank=True)

    def __str__(self) -> str:
        return self.name


class Exercise(models.Model):
    class ExerciseType(models.TextChoices):
        STRENGTH = "strength", "Strength"
        CARDIO = "cardio", "Cardio"
        MOBILITY = "mobility", "Mobility"
        STRETCH = "stretch", "Stretch"
        SPORT = "sport", "Sport"

    name = models.CharField(max_length=120, unique=True)
    exercise_type = models.CharField(
        max_length=20, choices=ExerciseType.choices, default=ExerciseType.STRENGTH
    )

    # seeded catalog relationship
    muscle_groups = models.ManyToManyField(
        MuscleGroup, related_name="exercises", blank=True
    )

    # optional extras for a nice UX
    equipment = models.CharField(max_length=60, blank=True)  # keep free-form for MVP
    instructions = models.TextField(blank=True)
    video_url = models.URLField(blank=True)

    def __str__(self) -> str:
        return self.name


class WorkoutTemplate(models.Model):
    """
    Reusable routine users can copy/share and attach to WorkoutPlans.
    """
    user = models.ForeignKey(
        settings.AUTH_USER_MODEL, on_delete=models.CASCADE, related_name="workout_templates"
    )
    title = models.CharField(max_length=140)
    description = models.TextField(blank=True)

    # sharing/copying
    is_public = models.BooleanField(default=False)
    source_template = models.ForeignKey(
        "self",
        null=True,
        blank=True,
        on_delete=models.SET_NULL,
        related_name="forks",
        help_text="If this template was copied from another template, reference it here.",
    )

    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        unique_together = ("user", "title")

    def __str__(self) -> str:
        return self.title


class WorkoutTemplateItem(models.Model):
    """
    One exercise per item (confirmed). These are NOT dated/logged; they're template content.
    """
    template = models.ForeignKey(
        WorkoutTemplate, on_delete=models.CASCADE, related_name="items"
    )
    exercise = models.ForeignKey(
        Exercise, on_delete=models.PROTECT, related_name="template_items"
    )
    order = models.PositiveIntegerField(default=0)

    # prescription fields (nullable to support different exercise types)
    sets = models.PositiveIntegerField(null=True, blank=True)
    reps = models.PositiveIntegerField(null=True, blank=True)
    weight = models.DecimalField(max_digits=7, decimal_places=2, null=True, blank=True)
    weight_unit = models.CharField(
        max_length=5, choices=(("lb", "lb"), ("kg", "kg")), null=True, blank=True
    )
    duration_seconds = models.PositiveIntegerField(null=True, blank=True)
    distance_meters = models.PositiveIntegerField(null=True, blank=True)
    rpe = models.PositiveIntegerField(null=True, blank=True)  # 1–10

    notes = models.TextField(blank=True)

    class Meta:
        ordering = ["order", "id"]

    def __str__(self) -> str:
        return f"{self.template.title} - {self.exercise.name}"


class WorkoutPlan(models.Model):
    """
    Recurrence + program container. Generates Workout instances (calendar events).
    Attaches to templates via the through table WorkoutTemplatePlan.
    """
    class Frequency(models.TextChoices):
        DAILY = "daily", "Daily"
        WEEKLY = "weekly", "Weekly"
        MONTHLY = "monthly", "Monthly"

    user = models.ForeignKey(
        settings.AUTH_USER_MODEL, on_delete=models.CASCADE, related_name="workout_plans"
    )
    title = models.CharField(max_length=140)

    start_datetime = models.DateTimeField()
    timezone = models.CharField(
        max_length=64, blank=True, help_text="IANA timezone, e.g. America/New_York"
    )

    frequency = models.CharField(max_length=10, choices=Frequency.choices)
    interval = models.PositiveIntegerField(default=1)

    # store weekdays like ["MO","WE","FR"] (weekly plans)
    by_weekdays = models.JSONField(null=True, blank=True)

    # choose one recurrence stop condition; until is simplest
    until = models.DateTimeField(null=True, blank=True)

    is_active = models.BooleanField(default=True)

    # M2M to templates (composition)
    templates = models.ManyToManyField(
        WorkoutTemplate,
        through="WorkoutTemplatePlan",
        related_name="plans",
        blank=True,
    )

    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        unique_together = ("user", "title")

    def __str__(self) -> str:
        return self.title


class WorkoutTemplatePlan(models.Model):
    """
    Through table connecting WorkoutPlans <-> WorkoutTemplates.

    This is where you can express scheduling/composition rules like:
    - which weekdays a template runs on
    - ordering
    - rotating multi-week programs (week_index)
    """
    WEEKDAY_CHOICES = (
        ("MO", "Monday"),
        ("TU", "Tuesday"),
        ("WE", "Wednesday"),
        ("TH", "Thursday"),
        ("FR", "Friday"),
        ("SA", "Saturday"),
        ("SU", "Sunday"),
    )

    plan = models.ForeignKey(WorkoutPlan, on_delete=models.CASCADE)
    template = models.ForeignKey(WorkoutTemplate, on_delete=models.CASCADE)

    # optional scheduling hints (use what you need for MVP)
    day_of_week = models.CharField(max_length=2, choices=WEEKDAY_CHOICES, null=True, blank=True)
    week_index = models.PositiveIntegerField(
        null=True, blank=True,
        help_text="Use for rotating programs (e.g., week 1/2/3/4)."
    )
    order = models.PositiveIntegerField(default=0)

    class Meta:
        unique_together = ("plan", "template", "day_of_week", "week_index")
        ordering = ["order", "id"]

    def __str__(self) -> str:
        return f"{self.plan.title} ↔ {self.template.title}"


class Workout(models.Model):
    """
    Dated calendar instance (what the React calendar queries).
    Can be generated from a plan and/or template, but both are nullable for one-offs.
    """
    class Status(models.TextChoices):
        PLANNED = "planned", "Planned"
        COMPLETED = "completed", "Completed"
        SKIPPED = "skipped", "Skipped"
        CANCELED = "canceled", "Canceled"

    user = models.ForeignKey(
        settings.AUTH_USER_MODEL, on_delete=models.CASCADE, related_name="workouts"
    )

    plan = models.ForeignKey(
        WorkoutPlan,
        null=True,
        blank=True,
        on_delete=models.SET_NULL,
        related_name="workouts",
    )
    template = models.ForeignKey(
        WorkoutTemplate,
        null=True,
        blank=True,
        on_delete=models.SET_NULL,
        related_name="generated_workouts",
        help_text="Template used to generate this workout instance (if applicable).",
    )

    title = models.CharField(max_length=140)
    scheduled_start = models.DateTimeField(db_index=True)
    scheduled_end = models.DateTimeField(null=True, blank=True)

    status = models.CharField(
        max_length=10, choices=Status.choices, default=Status.PLANNED
    )
    notes = models.TextField(blank=True)

    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ["scheduled_start", "id"]

    def __str__(self) -> str:
        return f"{self.title} @ {self.scheduled_start}"


class WorkoutItem(models.Model):
    """
    One exercise per item (confirmed) within a specific Workout instance.
    This is the logged/scheduled data, not the reusable template data.
    """
    workout = models.ForeignKey(
        Workout, on_delete=models.CASCADE, related_name="items"
    )
    exercise = models.ForeignKey(
        Exercise, on_delete=models.PROTECT, related_name="workout_items"
    )
    order = models.PositiveIntegerField(default=0)

    # performance / prescription fields
    sets = models.PositiveIntegerField(null=True, blank=True)
    reps = models.PositiveIntegerField(null=True, blank=True)
    weight = models.DecimalField(max_digits=7, decimal_places=2, null=True, blank=True)
    weight_unit = models.CharField(
        max_length=5, choices=(("lb", "lb"), ("kg", "kg")), null=True, blank=True
    )
    duration_seconds = models.PositiveIntegerField(null=True, blank=True)
    distance_meters = models.PositiveIntegerField(null=True, blank=True)
    rpe = models.PositiveIntegerField(null=True, blank=True)  # 1–10

    notes = models.TextField(blank=True)

    class Meta:
        ordering = ["order", "id"]

    def __str__(self) -> str:
        return f"{self.workout.title} - {self.exercise.name}"
```

A couple quick implementation notes (so this doesn’t bite you later):

* `Exercise` uses `PROTECT` in WorkoutItem/TemplateItem so seeded exercises can’t be deleted if referenced.
* `Workout.plan` + `Workout.template` are **nullable** to support one-off workouts and custom workouts not generated from templates.
* `WorkoutTemplatePlan` is your renamed through table and has optional fields (`day_of_week`, `week_index`) to support simple weekly schedules *or* rotating programs.

Next steps:

* serializers for nested `Workout` + `WorkoutItem`
* a service function to “generate next 8 weeks” of `Workout` instances from a `WorkoutPlan` and clone template items into workout items
