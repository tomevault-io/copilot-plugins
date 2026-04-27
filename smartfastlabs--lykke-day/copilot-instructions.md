## lykke-day

> Commands handle write operations (mutations) that change system state. They use Unit of Work for transaction management.


# Application Commands Rules

Commands handle write operations (mutations) that change system state. They use Unit of Work for transaction management.

---

## Mandatory: Typed Read-Only Repository Declarations (mypy)

If a command handler uses a read-only repository via `self.*_ro_repo`, it must declare that dependency as a class attribute with the exact protocol type.

This is **required** for:
- `BaseHandler` runtime dependency wiring
- mypy type checking

```python
class RescheduleDayHandler(BaseCommandHandler[RescheduleDayCommand, DayContext]):
    day_ro_repo: DayRepositoryReadOnlyProtocol
    task_ro_repo: TaskRepositoryReadOnlyProtocol
```

No typed declaration means the dependency is not guaranteed to be wired and is not mypy-safe.

---

## Command Structure

Commands follow a specific pattern with a `Command` dataclass and `CommandHandler` class:

```python
from dataclasses import dataclass

from lykke.application.commands.base import BaseCommandHandler, Command
from lykke.application.handler_factory_protocols import ReadOnlyRepositoryFactoryProtocol
from lykke.application.unit_of_work import UnitOfWorkFactory
from lykke.domain.entities import UserEntity
from lykke.domain import value_objects
from lykke.domain.entities import DayEntity


@dataclass(frozen=True)
class ScheduleDayCommand(Command):
    """Command to schedule a day."""
    
    date: date
    template_id: UUID | None = None


class ScheduleDayHandler(BaseCommandHandler[ScheduleDayCommand, value_objects.DayContext]):
    """Schedules a day with tasks from routines."""

    day_template_ro_repo: DayTemplateRepositoryReadOnlyProtocol

    async def handle(self, command: ScheduleDayCommand) -> value_objects.DayContext:
        """Schedule a day with tasks from routines."""
        async with self.new_uow() as uow:
            # Read entities via uow ro_repo properties
            template = await uow.day_template_ro_repo.get(command.template_id)
            
            # Create new entity
            day = DayEntity.create_for_date(
                command.date,
                user_id=self.user.id,
                template=template,
            )
            
            # Create entity (marks as created and adds to UoW)
            await uow.create(day)
            
            # Create tasks
            for task in tasks:
                await uow.create(task)
            
            # commit() is called automatically when exiting context
            return value_objects.DayContext(day=day, tasks=tasks)
```

---

## Key Rules for Commands

1. **Extend BaseCommandHandler** - Use `BaseCommandHandler[CommandT, ResultT]` base class
2. **Define Command dataclass** - Use frozen dataclass extending `Command`
3. **Implement handle()** - Implement async `handle(command: CommandT) -> ResultT`
4. **Use Unit of Work** - Use `self.new_uow()` to create transaction-scoped UoW
5. **Declare typed ro repos** - For any `self.*_ro_repo` access, declare protocol-typed class attributes first
6. **Read via `uow.*_ro_repo` properties** - Use read-only repositories from UoW for transactional reads
7. **Use `uow.create()` for new entities** - Marks entity as created and adds to UoW
8. **Use `uow.add()` for updates** - Track modified entities for persistence
9. **Use `uow.delete()` for deletions** - Marks entity for deletion
10. **Return a single typed result** - Return a domain entity, a value object, or a **result dataclass**. Do not return raw tuples or untyped dicts; for multiple values use a frozen dataclass. Never return infrastructure types.

**Result type:** Command handlers must return a single typed value (entity, value object, or a frozen result dataclass). Do not return raw `tuple`, `list`, or `dict` as the handler return type.

---

## Unit of Work Pattern

Commands use Unit of Work for transaction management:

```python
async with self.new_uow() as uow:
    # All operations share the same transaction
    day = await uow.day_ro_repo.get(day_id)
    task = await uow.task_ro_repo.get(task_id)
    
    # Modify entities
    day.update_high_level_plan(new_plan)
    
    # Track entities for persistence
    uow.add(day)
    
    # commit() is called automatically when exiting context
    # Or call await uow.commit() explicitly for early commit
```

**Important:** The `commit()` method:
1. Processes all added entities (create, update, delete based on domain events)
2. Collects domain events from all aggregates
3. Dispatches domain events to handlers (BEFORE commit)
4. Commits the database transaction
5. Publishes events to Redis (AFTER commit)

---

## Entity Lifecycle Methods

The UoW provides convenience methods for entity lifecycle:

```python
# Create new entity - marks as created, adds EntityCreatedEvent
day = DayEntity.create_for_date(date, user_id=user_id, template=template)
await uow.create(day)  # Equivalent to: day.create(); uow.add(day)

# Update entity - just add it to UoW (entity methods add appropriate events)
day.update_high_level_plan(new_plan)  # Entity method adds DayUpdatedEvent
uow.add(day)

# Delete entity - marks as deleted, adds EntityDeletedEvent
await uow.delete(day)  # Equivalent to: day.delete(); uow.add(day)

# Apply update object pattern
updated_day = day.apply_update(update_object, DayUpdatedEvent)
uow.add(updated_day)
```

---

## Gotchas

### 1. Commands Must Use Unit of Work

**Issue:** Commands must use `self.new_uow()` to create UoW. Don't create repositories directly.

```python
# Wrong
repo = TaskRepository(user=self.user)
task = await repo.get(task_id)

# Right
async with self.new_uow() as uow:
    task = await uow.task_ro_repo.get(task_id)
```

### 2. Entities Must Be Added to UoW

**Issue:** Only entities added via `uow.add()` or `uow.create()` are persisted.

```python
# Wrong
day = DayEntity.create_for_date(date, user_id=user_id, template=template)
# Entity won't be saved!

# Right
day = DayEntity.create_for_date(date, user_id=user_id, template=template)
await uow.create(day)  # Now it will be saved
```

### 3. Commits Dispatch Events

**Issue:** Domain events are dispatched during `commit()`, not when `_add_event()` is called.

```python
async with self.new_uow() as uow:
    task.complete()  # Adds TaskCompletedEvent to task._domain_events
    uow.add(task)
    # Event handlers run BEFORE database commit
    # Event is dispatched when context exits (auto-commit)
```

### 4. Use BaseCommandHandler Dependencies

**Issue:** `BaseCommandHandler` has `self.user`, `self.new_uow()`, and annotation-wired dependencies.

```python
class MyHandler(BaseCommandHandler[MyCommand, MyResult]):
    task_ro_repo: TaskRepositoryReadOnlyProtocol

    async def handle(self, command: MyCommand) -> MyResult:
        # self.user is already the authenticated user entity
        user = self.user

        # Annotation-backed read-only repo access
        task = await self.task_ro_repo.get(command.task_id)

        # Or within a UoW context for transactional reads
        async with self.new_uow() as uow:
            task = await uow.task_ro_repo.get(command.task_id)
```

### 5. Missing Type Annotation = Missing Dependency

**Issue:** Accessing `self.*_ro_repo` without protocol annotation is invalid.

```python
# Wrong - dependency not declared
class BadHandler(BaseCommandHandler[Cmd, Result]):
    async def handle(self, command: Cmd) -> Result:
        return await self.day_ro_repo.get(command.day_id)

# Right - explicit protocol annotation
class GoodHandler(BaseCommandHandler[Cmd, Result]):
    day_ro_repo: DayRepositoryReadOnlyProtocol

    async def handle(self, command: Cmd) -> Result:
        return await self.day_ro_repo.get(command.day_id)
```

### 6. Do Not Add Pass-Through `__init__`

**Issue:** If `__init__` only forwards to `super().__init__`, remove it.

```python
# Wrong - pass-through constructor
class NoisyCommandHandler(BaseCommandHandler[Cmd, Result]):
    def __init__(
        self,
        *,
        user: UserEntity,
        uow_factory: UnitOfWorkFactory,
        repository_factory: ReadOnlyRepositoryFactoryProtocol,
    ) -> None:
        super().__init__(
            user=user,
            uow_factory=uow_factory,
            repository_factory=repository_factory,
        )

# Right - no constructor unless extra dependencies/setup are needed
class CleanCommandHandler(BaseCommandHandler[Cmd, Result]):
    task_ro_repo: TaskRepositoryReadOnlyProtocol
```

Keep `__init__` only when handler-specific dependencies are added (beyond base DI wiring).

---

## Testing Guidance

### Unit Testing Commands

Test command handlers with mocked Unit of Work:

```python
from dobles import allow, expect

@pytest.mark.asyncio
async def test_schedule_day(mock_ro_repo_factory, mock_uow_factory, mock_uow, test_user):
    """Test schedule_day command."""
    handler = ScheduleDayHandler(
        user=test_user,
        uow_factory=mock_uow_factory,
        repository_factory=mock_ro_repo_factory,
    )

    allow(mock_uow_factory).create(test_user).and_return(mock_uow)
    # Mock UoW context manager
    allow(mock_uow).__aenter__().and_return(mock_uow)
    allow(mock_uow).__aexit__(...).and_return(None)

    # Mock repository calls
    allow(mock_uow.day_template_ro_repo).get(...).and_return(template)

    command = ScheduleDayCommand(date=date(2025, 1, 1))
    result = await handler.handle(command)

    # Verify results
    assert result.day.date == date(2025, 1, 1)
```

### Test Location

Command handler tests go in:
- `tests/unit/application/commands/` - Command handler tests
- Use `dobles` for mocking protocols
- Mock all dependencies (UoW, repositories)

### Test Patterns

1. **Mock protocols** - Use `dobles` to mock protocol methods
2. **Test business logic** - Verify handlers orchestrate correctly
3. **Test error cases** - Verify error handling
4. **Test transaction boundaries** - Verify UoW usage

---

## Common Patterns

### Command Handler Pattern

```python
@dataclass(frozen=True)
class MyCommand(Command):
    entity_id: UUID
    new_value: str


class MyCommandHandler(BaseCommandHandler[MyCommand, EntityType]):
    entity_ro_repo: MyEntityRepositoryReadOnlyProtocol

    async def handle(self, command: MyCommand) -> EntityType:
        async with self.new_uow() as uow:
            # Read entity
            entity = await uow.entity_ro_repo.get(command.entity_id)
            
            # Modify entity
            entity.update_value(command.new_value)
            
            # Track entity
            uow.add(entity)
            
            return entity
```

### Injecting Other Handlers

```python
class ScheduleDayHandler(BaseCommandHandler[ScheduleDayCommand, DayContext]):
    def __init__(
        self,
        *,
        user: UserEntity,
        uow_factory: UnitOfWorkFactory,
        repository_factory: ReadOnlyRepositoryFactoryProtocol,
        preview_day_handler: PreviewDayHandler,  # Injected handler
    ) -> None:
        super().__init__(
            user=user,
            uow_factory=uow_factory,
            repository_factory=repository_factory,
        )
        self.preview_day_handler = preview_day_handler
    
    async def handle(self, command: ScheduleDayCommand) -> DayContext:
        # Use injected handler
        preview = await self.preview_day_handler.preview_day(command.date)
        # ...
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/smartfastlabs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
