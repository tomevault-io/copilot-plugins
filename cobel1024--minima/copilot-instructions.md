## minima

> Open-source LMS. Backend: Django (async) + Django Ninja. Frontend: SolidJS + TanStack Router.

# Minima LMS

Open-source LMS. Backend: Django (async) + Django Ninja. Frontend: SolidJS + TanStack Router.
Multi-tenant by subdomain: `student`, `studio`, `tutor`, `desk`, `preview`.

## Dev

```bash
sh dev.sh up          # start all services (Docker Compose)
uv run dev.py lint    # ruff check + format + pyrefly type check
cd web && npm run lint # biome
```

Django :8000 · MinIO :9001 · Mailpit :8025 · nginx subdomain routing

## Architecture

**API auto-discovery:** `minima/api.py` scans `apps/{app}/api/v1.py` for `router` and mounts at `/api/v1/{app}`.

**Auth:** `request.auth` = user ID string · `request.roles` = list[str] · realm mismatch = 401

**Roles:** `editor` (studio) · `grader` · `partner_staff`

**Content `context`** (on `Watch` and attempt models) — isolates activity per session. Format `model::model_id::session_id` (e.g. `course::<course_id>::<engagement_id>`, built by `issue_active_context()`); empty `""` = standalone access (watched/attempted directly, outside any course session). `normalize_context()` drops `session_id` (`course::id::sess` → `course=id`) to dedup across sessions of the same parent. `(user, media, context)` is unique, so **filtering `context=""` already yields one row per (user, media) — no `DISTINCT ON` / normalization / dedup needed.** Only reach for those when a query legitimately spans non-empty contexts.

**Global exception handlers** (`minima/api.py`) — never handle these manually in views or models:

| Exception | Status |
|-----------|--------|
| `ValueError(MessageCode.X)` | 400 |
| `ValidationError` | 400 |
| `ObjectDoesNotExist` / `Http404` | 404 |
| `AuthenticationError` | 401 |
| `Throttled` | 429 |

---

## Before Writing Any Code

1. Read the relevant model files in full
2. Map every DB relationship that the feature touches
3. Write the queryset shape (what `select_related` / `prefetch_related` you'll need) before writing any logic
4. Only then write the implementation

This order eliminates N+1s and structural rewrites after the fact.

---

## Naming

**Names must read like natural English** — a noun, not a noun decorated with metadata.

**DateTime fields** — no `_at` suffix. The name itself is the event:
```python
published   # not published_at
completed   # not completed_at
confirmed   # not confirmed_at
deleted     # not deleted_at
started     # not started_at
lock        # a deadline point in time, concise
```

**Boolean fields** — no `is_` prefix unless the word would be ambiguous without it:
```python
active      # not is_active
featured    # not is_featured
passed      # not is_passed
verified    # not is_verified
public      # not is_public
```
Exception: Django's own `User.is_active`, `User.is_staff`, `User.is_superuser` — leave these alone.

**Variables** — full words, never abbreviations:
```python
exam          # not e, ex, exam_obj, exam_instance
template_channel  # not tc
channel_impl      # not ch, ci
content_type      # not ct
enrollment        # not enr, enrl
```

**FK identifier args/fields** — `_id` suffix is Django convention, keep it:
```python
learner_id, owner_id, exam_id, context_id  # fine
```

**Local variables** — short but complete. One word is ideal, two is fine, three is a smell:
```python
attempt = await Attempt.objects.aget(...)   # good
existing_attempt = ...                       # acceptable when disambiguating
existing_active_attempt_for_user = ...       # too long — restructure instead
```

---

## Imports

Top-level only. Never import inside a function body unless the import is circular and unavoidable.

**Circular imports** — comment must show the full cycle:
```python
from apps.partner.models import Member  # circular: account → partner → account
```

**Type-only imports** — put in `TYPE_CHECKING` block:
```python
from typing import TYPE_CHECKING
if TYPE_CHECKING:
    from apps.learning.models import Enrollment
```

Never use a function-level import to work around a circular dependency that could be resolved with `TYPE_CHECKING` or `apps.get_model()`.

---

## Types

**Annotate for the reader, not the type checker.** If pyrefly already knows the type, don't repeat it.

```python
# wrong — checker knows this is list[str]
result: list[str] = []

# wrong — pyrefly infers return type of cls.objects.aget() chain
async def get_for_edit(cls, *, id: str, owner_id: str) -> "Exam": ...

# right — TypedDict fields always annotated (that's their purpose)
class SessionDict(TypedDict):
    step: LearningSessionStep
    attempt: NotRequired["Attempt"]

# right — annotate where the return type isn't obvious from the body
async def get_session(cls, ...) -> SessionDict:
    ...

# right — annotate when narrowing from a broad return type
exam: Exam = await cls.objects.aget(id=exam_id)
```

Rules:
- `TypedDict` fields: always annotate
- Function signatures: annotate params and return when not obvious
- `cls.objects.aget()` / `cls.objects.select_related(...).aget()` return types: **never annotate** — pyrefly fully infers
- Local variables: only when the type is not immediately obvious from the RHS
- No `Any` without a comment explaining why
- `TypedDict` + `NotRequired[T]` for optional keys in complex return shapes
- `TYPE_CHECKING` guard for forward refs and FK `_id` field annotations on models

---

## App Dependency Rules

**Domain models must not know about realm apps** (`studio`, `desk`, `tutor`, `preview`).

Model classmethod names must be abstract domain concepts — never reference the caller:

```python
# wrong — model learns about its caller
async def get_for_studio_edit(cls, ...): ...
async def get_for_desk_view(cls, ...): ...

# right — abstract domain concept; any caller can use it
async def get_for_edit(cls, ...): ...      # fetches full editable shape
async def get_grading_paper(cls, ...): ... # fetches grading workflow shape
```

Realm apps (`studio`, `desk`, etc.) may freely import from domain apps (`exam`, `course`, etc.), but never the reverse.

---

## Backend Rules

### Views — thin, always

1–3 lines. All logic in model classmethods. No try/except. No dicts returned on error.

```python
@router.post("/{id}/attempt", response=AttemptSchema)
@active_context()
@access_date("exam", "exam")
async def start_attempt(request: HttpRequest, id: str):
    return await Attempt.start(
        exam_id=id, learner_id=request.auth,
        lock=request.access_date["end"], context=request.active_context,
    )
```

### Models — business logic home

Classmethods with keyword-only args after `cls`. `IntegrityError` is the only exception caught — only when converting a known DB constraint to a user error.

```python
@classmethod
async def start(cls, *, exam_id: str, learner_id: str, lock: datetime, context: str):
    exam = await Exam.objects.prefetch_related("question_pool__questions").aget(id=exam_id)
    try:
        attempt = await cls.objects.acreate(exam=exam, learner_id=learner_id, lock=lock, context=context)
    except IntegrityError:
        raise ValueError(MessageCode.ATTEMPT_ALREADY_STARTED)
    return attempt
```

### DB queries — no N+1

**Plan the queryset before writing the method body.** Chain `select_related` + `prefetch_related` before the fetch.

```python
# correct
attempt = (
    await cls.objects
    .select_related("exam", "submission", "grade")
    .prefetch_related(
        Prefetch("questions", queryset=Question.objects.select_related("solution").order_by("id"))
    )
    .aget(exam_id=exam_id, learner_id=learner_id, active=True)
)

# wrong — N+1
attempt = await cls.objects.aget(exam_id=exam_id)
exam = await attempt.exam
questions = [q async for q in attempt.questions.all()]
```

`asave(update_fields=["field"])` for partial saves. `bulk_update` for batch. `annotate()` for DB-level computed fields.

`asyncio.gather()` for independent parallel queries. Post-fetch prefetch with `aprefetch_related_objects()`.

**Reverse relations in async context** — always use `select_related` before accessing any relation. Accessing an unfetched relation in async raises `SynchronousOnlyOperation`. After `select_related`, a missing reverse OneToOne returns `None` — never `DoesNotExist` — so no try/except is needed. This also prevents N+1s. Rule: `select_related` or raise; never lazy-load.

### Errors — MessageCode only

```python
raise ValueError(MessageCode.ATTEMPT_ALREADY_STARTED)  # correct
raise ValueError("Attempt already started")             # wrong
```

New codes go in `apps/common/policy.py` `MessageCode` enum.

### Models — conventions

```python
@pghistory.track()
class Exam(LearningObjectMixin, GradeWorkflowMixin):
    owner = ForeignKey(User, CASCADE)
    question_pool = Foreignaey(QuestionPool, CASCADE, related_name="+")  # "+" = no backreference

    class Meta:
        constraints = [
            UniqueConstraint(fields=["owner", "title"], name="exam_exam_ow_ti_uniq")
            #                                                  {app}_{model}_{fields}_uniq
        ]
        indexes = [Index(fields=["learner_id", "active"])]

    if TYPE_CHECKING:      # annotate FK _id fields and reverse relations here
        pk: str
        question_pool_id: int
        submission: "Submission"
```

Constraint naming: `{app}_{model_abbrev}_{field_abbrevs}_uniq`

### Notifications — track_fields

`GradeFieldMixin` uses `@track_fields("completed", "confirmed")` which fires `on_{field}_changed(self, old_value)` after save.

```python
async def on_completed_changed(self, old_value: datetime | None):
    if self.completed and not self.confirmed:
        await user_message_created.asend(
            source=self.attempt.exam,
            message=MessageType(
                user_id=self.attempt.learner_id,
                title=MessageCode.EXAM_GRADING_COMPLETED,
                body=self.attempt.exam.title,
            ),
        )
```

### Schemas — manual only

`ModelSchema` breaks with inheritance. Always write manually.

```python
class ExamSchema(LearningObjectMixinSchema):
    id: str
    duration_seconds: int

    @staticmethod
    def resolve_duration_seconds(obj: Exam) -> int:
        return int(obj.duration.total_seconds())
```

All schemas inherit `Schema` from `apps.common.schema` (has `to_camel` alias).
Optional session fields: `Annotated[T, Field(None)]`

### Custom Signals

Never use raw `Signal()` — no type safety. Subclass and override `asend()` with typed kwargs:

```python
class ExamGradeConfirmedSignal(Signal):
    async def asend(self, *, sender, attempt: "Attempt", **kwargs):
        return await super().asend(sender=sender, attempt=attempt, **kwargs)

exam_grade_confirmed = ExamGradeConfirmedSignal()
```

---

## Testing

Flow-based E2E. One test = one complete user journey.

```python
@pytest.mark.e2e
@pytest.mark.django_db
def test_exam_flow(client: Client, mimesis: Generic, admin_user: AdminUser):
    admin_user.login()
    exam = ExamFactory(verification_required=True)
    EnrollmentFactory(content_type=..., content_id=exam.id, user_id=admin_user.id)

    res = client.get(f"/api/v1/exam/{exam.id}/session")
    assert res.status_code == 200, "get exam session"
```

- `assert res.status_code == 200, "description"` — always include the message
- Every model has a factory in `apps/{app}/tests/factories.py`
- Default `client` fixture: `Client(SERVER_NAME="student.testserver")` — other realms need explicit `SERVER_NAME`
- No DB mocking. Tests run against a real database.
- Two suites: `pytest -m 'not e2e'` (unit/model) and `pytest -m e2e` (API flows)

---

## Refactoring

A bare "refactor X" request means **behavior- and query-shape-preserving readability work** — never a redesign, strategy pattern, or new abstraction unless explicitly asked.

- `# [query-reviewed]` / `# [N+1-reviewed]` comments mark intentional, non-obvious code (raw SQL, `apps.get_model()` dispatch, hand-tuned query shapes). **Never "simplify" or ORM-ify these without saying why and getting confirmation first.**
- The safety net for behavior-preserving work is the test suite. If the target behavior isn't covered, write a characterization test **before** changing anything.
- Preserve side effects: `@track_fields` hooks (`on_*_changed`), signals, and `@pghistory.track()`.
- A long method is not automatically a problem. Prefer named locals, guard clauses, and comments over splitting into single-use private helpers. Don't extract for the sake of extracting.
- When done, report what you changed, what you deliberately left alone (especially `[query-reviewed]` spots), and the test/lint result.

For the step-by-step procedure, use the `refactor` skill.

---

## Never

- try/except in views
- `raise ValueError("string")` — use MessageCode
- N+1 queries (plan the queryset first)
- `ModelSchema`
- `unique_together` — use `UniqueConstraint`
- sync ORM in async context
- Abstractions for one-time use
- Docstrings on unchanged code
- Backwards-compatibility shims
- Inline imports inside function bodies (unless circular, with comment showing the full cycle)
- Abbreviated variable names (`tc`, `ch`, `ct`, `enr`, `e`, `g`)
- `is_` prefix on custom boolean fields or `_at` suffix on datetime fields
- Over-annotating types pyrefly already infers
- DB hits inside loops — always prefetch or bulk
- Patching over a structural problem — fix the structure

---
> Source: [cobel1024/minima](https://github.com/cobel1024/minima) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
