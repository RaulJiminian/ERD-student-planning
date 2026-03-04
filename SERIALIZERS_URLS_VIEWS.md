```python
# serializers.py
from django.db import transaction
from rest_framework import serializers

from .models import (
    MuscleGroup,
    Exercise,
    WorkoutTemplate,
    WorkoutTemplateItem,
    WorkoutPlan,
    WorkoutTemplatePlan,
    Workout,
    WorkoutItem,
)


# -------------------------
# Seeded / read-only models
# -------------------------

class MuscleGroupSerializer(serializers.ModelSerializer):
    class Meta:
        model = MuscleGroup
        fields = ["id", "name", "description"]


class ExerciseSerializer(serializers.ModelSerializer):
    muscle_groups = MuscleGroupSerializer(many=True, read_only=True)
    muscle_group_ids = serializers.PrimaryKeyRelatedField(
        many=True,
        write_only=True,
        source="muscle_groups",
        queryset=MuscleGroup.objects.all(),
        required=False,
    )

    class Meta:
        model = Exercise
        fields = [
            "id",
            "name",
            "exercise_type",
            "equipment",
            "instructions",
            "video_url",
            "muscle_groups",
            "muscle_group_ids",
        ]


# -------------------------
# Templates
# -------------------------

class WorkoutTemplateItemSerializer(serializers.ModelSerializer):
    exercise_detail = ExerciseSerializer(source="exercise", read_only=True)

    class Meta:
        model = WorkoutTemplateItem
        fields = [
            "id",
            "template",
            "exercise",
            "exercise_detail",
            "order",
            "sets",
            "reps",
            "weight",
            "weight_unit",
            "duration_seconds",
            "distance_meters",
            "rpe",
            "notes",
        ]
        read_only_fields = ["template"]


class WorkoutTemplateSerializer(serializers.ModelSerializer):
    items = WorkoutTemplateItemSerializer(many=True, required=False)
    # If you want an easy "copy" action later, keeping source_template is helpful.
    source_template = serializers.PrimaryKeyRelatedField(
        queryset=WorkoutTemplate.objects.all(),
        required=False,
        allow_null=True,
    )

    class Meta:
        model = WorkoutTemplate
        fields = [
            "id",
            "title",
            "description",
            "is_public",
            "source_template",
            "items",
            "created_at",
            "updated_at",
        ]
        read_only_fields = ["created_at", "updated_at"]

    def create(self, validated_data):
        request = self.context["request"]
        items_data = validated_data.pop("items", [])
        validated_data["user"] = request.user

        with transaction.atomic():
            template = WorkoutTemplate.objects.create(**validated_data)
            if items_data:
                WorkoutTemplateItem.objects.bulk_create(
                    [
                        WorkoutTemplateItem(template=template, **item)
                        for item in items_data
                    ]
                )
        return template

    def update(self, instance, validated_data):
        # Simple, student-friendly approach:
        # - Update template fields
        # - If `items` is provided, replace all items (delete + recreate)
        items_data = validated_data.pop("items", None)

        with transaction.atomic():
            for attr, val in validated_data.items():
                setattr(instance, attr, val)
            instance.save()

            if items_data is not None:
                instance.items.all().delete()
                if items_data:
                    WorkoutTemplateItem.objects.bulk_create(
                        [
                            WorkoutTemplateItem(template=instance, **item)
                            for item in items_data
                        ]
                    )

        return instance


# -------------------------
# Plans (recurrence + composition)
# -------------------------

class WorkoutTemplatePlanSerializer(serializers.ModelSerializer):
    template_detail = WorkoutTemplateSerializer(source="template", read_only=True)

    class Meta:
        model = WorkoutTemplatePlan
        fields = [
            "id",
            "plan",
            "template",
            "template_detail",
            "day_of_week",
            "week_index",
            "order",
        ]
        read_only_fields = ["plan"]


class WorkoutPlanSerializer(serializers.ModelSerializer):
    # Write: send list of through-table objects
    template_links = WorkoutTemplatePlanSerializer(
        many=True, required=False, source="workouttemplateplan_set"
    )

    class Meta:
        model = WorkoutPlan
        fields = [
            "id",
            "title",
            "start_datetime",
            "timezone",
            "frequency",
            "interval",
            "by_weekdays",
            "until",
            "is_active",
            "template_links",
            "created_at",
            "updated_at",
        ]
        read_only_fields = ["created_at", "updated_at"]

    def create(self, validated_data):
        request = self.context["request"]
        links_data = validated_data.pop("workouttemplateplan_set", [])
        validated_data["user"] = request.user

        with transaction.atomic():
            plan = WorkoutPlan.objects.create(**validated_data)
            if links_data:
                WorkoutTemplatePlan.objects.bulk_create(
                    [
                        WorkoutTemplatePlan(plan=plan, **link)
                        for link in links_data
                    ]
                )
        return plan

    def update(self, instance, validated_data):
        links_data = validated_data.pop("workouttemplateplan_set", None)

        with transaction.atomic():
            for attr, val in validated_data.items():
                setattr(instance, attr, val)
            instance.save()

            if links_data is not None:
                # Replace links if provided
                WorkoutTemplatePlan.objects.filter(plan=instance).delete()
                if links_data:
                    WorkoutTemplatePlan.objects.bulk_create(
                        [
                            WorkoutTemplatePlan(plan=instance, **link)
                            for link in links_data
                        ]
                    )
        return instance


# -------------------------
# Workouts (calendar instances) + items
# -------------------------

class WorkoutItemSerializer(serializers.ModelSerializer):
    exercise_detail = ExerciseSerializer(source="exercise", read_only=True)

    class Meta:
        model = WorkoutItem
        fields = [
            "id",
            "workout",
            "exercise",
            "exercise_detail",
            "order",
            "sets",
            "reps",
            "weight",
            "weight_unit",
            "duration_seconds",
            "distance_meters",
            "rpe",
            "notes",
        ]
        read_only_fields = ["workout"]


class WorkoutSerializer(serializers.ModelSerializer):
    items = WorkoutItemSerializer(many=True, required=False)

    class Meta:
        model = Workout
        fields = [
            "id",
            "plan",
            "template",
            "title",
            "scheduled_start",
            "scheduled_end",
            "status",
            "notes",
            "items",
            "created_at",
            "updated_at",
        ]
        read_only_fields = ["created_at", "updated_at"]

    def create(self, validated_data):
        request = self.context["request"]
        items_data = validated_data.pop("items", [])
        validated_data["user"] = request.user

        with transaction.atomic():
            workout = Workout.objects.create(**validated_data)
            if items_data:
                WorkoutItem.objects.bulk_create(
                    [WorkoutItem(workout=workout, **item) for item in items_data]
                )
        return workout

    def update(self, instance, validated_data):
        items_data = validated_data.pop("items", None)

        with transaction.atomic():
            for attr, val in validated_data.items():
                setattr(instance, attr, val)
            instance.save()

            if items_data is not None:
                instance.items.all().delete()
                if items_data:
                    WorkoutItem.objects.bulk_create(
                        [WorkoutItem(workout=instance, **item) for item in items_data]
                    )
        return instance
```

```python
# views.py
from django.utils.dateparse import parse_datetime
from rest_framework import permissions, viewsets
from rest_framework.exceptions import ValidationError

from .models import (
    MuscleGroup,
    Exercise,
    WorkoutTemplate,
    WorkoutTemplateItem,
    WorkoutPlan,
    WorkoutTemplatePlan,
    Workout,
    WorkoutItem,
)
from .serializers import (
    MuscleGroupSerializer,
    ExerciseSerializer,
    WorkoutTemplateSerializer,
    WorkoutTemplateItemSerializer,
    WorkoutPlanSerializer,
    WorkoutTemplatePlanSerializer,
    WorkoutSerializer,
    WorkoutItemSerializer,
)


class IsOwnerOrReadOnlyPublic(permissions.BasePermission):
    """
    - Owners can do anything.
    - Non-owners can read if object has `is_public=True` (templates/plans).
    """
    def has_object_permission(self, request, view, obj):
        if request.method in permissions.SAFE_METHODS:
            if hasattr(obj, "is_public") and obj.is_public:
                return True
            return getattr(obj, "user_id", None) == getattr(request.user, "id", None)
        return getattr(obj, "user_id", None) == getattr(request.user, "id", None)


# -------------------------
# Seeded (read-only in Postman)
# -------------------------

class MuscleGroupViewSet(viewsets.ReadOnlyModelViewSet):
    queryset = MuscleGroup.objects.all().order_by("name")
    serializer_class = MuscleGroupSerializer
    permission_classes = [permissions.IsAuthenticated]


class ExerciseViewSet(viewsets.ReadOnlyModelViewSet):
    queryset = Exercise.objects.all().order_by("name")
    serializer_class = ExerciseSerializer
    permission_classes = [permissions.IsAuthenticated]


# -------------------------
# Templates
# -------------------------

class WorkoutTemplateViewSet(viewsets.ModelViewSet):
    serializer_class = WorkoutTemplateSerializer
    permission_classes = [permissions.IsAuthenticated, IsOwnerOrReadOnlyPublic]

    def get_queryset(self):
        # Owners can see their templates; everyone can see public templates
        return WorkoutTemplate.objects.filter(
            (models.Q(user=self.request.user) | models.Q(is_public=True))
        ).distinct().order_by("-updated_at")


class WorkoutTemplateItemViewSet(viewsets.ModelViewSet):
    serializer_class = WorkoutTemplateItemSerializer
    permission_classes = [permissions.IsAuthenticated]

    def get_queryset(self):
        # Only items belonging to templates you own (or public templates if you want view-only)
        return WorkoutTemplateItem.objects.filter(template__user=self.request.user).order_by("order", "id")

    def perform_create(self, serializer):
        # Force template ownership check
        template = serializer.validated_data["template"]
        if template.user != self.request.user:
            raise ValidationError("You do not own this template.")
        serializer.save()


# -------------------------
# Plans + through links
# -------------------------

class WorkoutPlanViewSet(viewsets.ModelViewSet):
    serializer_class = WorkoutPlanSerializer
    permission_classes = [permissions.IsAuthenticated, IsOwnerOrReadOnlyPublic]

    def get_queryset(self):
        from django.db.models import Q
        return WorkoutPlan.objects.filter(Q(user=self.request.user) | Q(is_public=True)).distinct().order_by("-updated_at")


class WorkoutTemplatePlanViewSet(viewsets.ModelViewSet):
    serializer_class = WorkoutTemplatePlanSerializer
    permission_classes = [permissions.IsAuthenticated]

    def get_queryset(self):
        return WorkoutTemplatePlan.objects.filter(plan__user=self.request.user).order_by("order", "id")

    def perform_create(self, serializer):
        plan = serializer.validated_data["plan"]
        template = serializer.validated_data["template"]
        if plan.user != self.request.user:
            raise ValidationError("You do not own this plan.")
        if template.user != self.request.user and not template.is_public:
            raise ValidationError("You can only attach your own templates (or public templates, if allowed).")
        serializer.save()


# -------------------------
# Workouts (calendar instances) + items
# -------------------------

class WorkoutViewSet(viewsets.ModelViewSet):
    serializer_class = WorkoutSerializer
    permission_classes = [permissions.IsAuthenticated]

    def get_queryset(self):
        qs = Workout.objects.filter(user=self.request.user).order_by("scheduled_start", "id")

        # Optional date filtering for calendar views:
        # GET /api/workouts/?start=2026-03-01T00:00:00Z&end=2026-04-01T00:00:00Z
        start = self.request.query_params.get("start")
        end = self.request.query_params.get("end")
        if start and end:
            start_dt = parse_datetime(start)
            end_dt = parse_datetime(end)
            if not start_dt or not end_dt:
                raise ValidationError("Invalid start/end. Use ISO datetime strings.")
            qs = qs.filter(scheduled_start__gte=start_dt, scheduled_start__lt=end_dt)

        return qs


class WorkoutItemViewSet(viewsets.ModelViewSet):
    serializer_class = WorkoutItemSerializer
    permission_classes = [permissions.IsAuthenticated]

    def get_queryset(self):
        return WorkoutItem.objects.filter(workout__user=self.request.user).order_by("order", "id")

    def perform_create(self, serializer):
        workout = serializer.validated_data["workout"]
        if workout.user != self.request.user:
            raise ValidationError("You do not own this workout.")
        serializer.save()
```

```python
# urls.py  (inside your app, e.g. main_app/urls.py)
from django.urls import include, path
from rest_framework.routers import DefaultRouter

from .views import (
    MuscleGroupViewSet,
    ExerciseViewSet,
    WorkoutTemplateViewSet,
    WorkoutTemplateItemViewSet,
    WorkoutPlanViewSet,
    WorkoutTemplatePlanViewSet,
    WorkoutViewSet,
    WorkoutItemViewSet,
)

router = DefaultRouter()
router.register(r"muscle-groups", MuscleGroupViewSet, basename="musclegroup")
router.register(r"exercises", ExerciseViewSet, basename="exercise")

router.register(r"workout-templates", WorkoutTemplateViewSet, basename="workouttemplate")
router.register(r"workout-template-items", WorkoutTemplateItemViewSet, basename="workouttemplateitem")

router.register(r"workout-plans", WorkoutPlanViewSet, basename="workoutplan")
router.register(r"workout-template-plans", WorkoutTemplatePlanViewSet, basename="workouttemplateplan")

router.register(r"workouts", WorkoutViewSet, basename="workout")
router.register(r"workout-items", WorkoutItemViewSet, basename="workoutitem")

urlpatterns = [
    path("api/", include(router.urls)),
]
```

```python
# IMPORTANT: add this import fix in views.py (I used models.Q once)
# At the top of views.py, add:
from django.db import models
```

### Postman testing quick hits (what to send)

**Create template with items (nested):** `POST /api/workout-templates/`

```json
{
  "title": "Leg Day A",
  "description": "Quads + glutes focus",
  "is_public": false,
  "items": [
    { "exercise": 1, "order": 1, "sets": 4, "reps": 8, "weight": "185.00", "weight_unit": "lb" },
    { "exercise": 2, "order": 2, "sets": 3, "reps": 12 }
  ]
}
```

**Create plan with template links (nested through table):** `POST /api/workout-plans/`

```json
{
  "title": "PPL Program",
  "start_datetime": "2026-03-03T09:00:00-05:00",
  "frequency": "weekly",
  "interval": 1,
  "by_weekdays": ["MO","WE","FR"],
  "until": "2026-06-01T00:00:00-05:00",
  "is_active": true,
  "template_links": [
    { "template": 10, "day_of_week": "MO", "order": 1 },
    { "template": 11, "day_of_week": "WE", "order": 2 },
    { "template": 12, "day_of_week": "FR", "order": 3 }
  ]
}
```

**Create a workout instance with items (nested):** `POST /api/workouts/`

```json
{
  "plan": 3,
  "template": 10,
  "title": "Leg Day A",
  "scheduled_start": "2026-03-04T18:00:00-05:00",
  "status": "planned",
  "items": [
    { "exercise": 1, "order": 1, "sets": 4, "reps": 8, "weight": "185.00", "weight_unit": "lb" }
  ]
}
```

**Calendar query:** `GET /api/workouts/?start=2026-03-01T00:00:00-05:00&end=2026-04-01T00:00:00-05:00`
