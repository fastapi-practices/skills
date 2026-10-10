# Coding Style Reference

Fully comply with `PEP 8` and follow the requirements below.

## Import Rules

- Each import statement should import only one module.
- Do not use `from xxx import *`.
- Use absolute imports and avoid relative imports.
- Do not add shebangs such as `#!/usr/bin/env python3`.
- Do not add encoding declarations such as `# -*- coding: utf-8 -*-`.
- Do not add file-level description comments.
- Do not define `__all__`.
- Do not add anything to `__init__.py` unless explicitly required.

## Typing

- All function parameters and return values must be typed, except the conventional `self` and `cls` receiver parameters.
- Use `Annotated` when framework metadata is needed.
- Use `|` for union types.

```python
from typing import Annotated

from fastapi import Path, Query


async def get_user(
    db: CurrentSession,
    pk: Annotated[int, Path(description='用户 ID')],
    username: Annotated[str | None, Query(description='用户名')] = None,
) -> ResponseSchemaModel[GetUserDetail]:
    ...
```

## Async Handling

- Use `async` / `await` for all I/O operations in API, service, and CRUD layers.

```python
class UserService:
    """用户服务类"""

    @staticmethod
    async def get(*, db: AsyncSession, pk: int) -> User:
        """
        获取用户详情

        :param db: 数据库会话
        :param pk: 用户 ID
        :return:
        """
        user = await user_dao.get(db, pk)
        if not user:
            raise errors.NotFoundError(msg='用户不存在')
        return user
```

## Keyword Arguments

- Service-layer methods must use keyword-only arguments.
- Use `*` to force keyword arguments where appropriate.

```python
class UserService:
    """用户服务类"""

    @staticmethod
    async def get(*, db: AsyncSession, pk: int) -> User:
        ...

    @staticmethod
    async def create(*, db: AsyncSession, obj: CreateUserParam) -> None:
        ...
```

## Function Definitions

- **Prefer module-level functions**: Logic that does not belong to any class should be defined as module-level functions rather than inside a class.
- **Choose method type based on actual binding semantics** (not habit):
  - Preserve binding and signatures required by an inherited SDK method, framework hook, or protocol.
  - If a method uses instance state or calls sibling instance methods, define it as an instance method and call those methods through `self`.
  - If a method needs class-level behavior or class state, define it as `@classmethod` and use `cls`; calling sibling class methods does not require conversion to an instance method.
  - Otherwise, if a method calls sibling service methods as part of the same workflow, use an instance method with `self` rather than hard-coding the class name.
  - If a method belongs to the class but does not call sibling methods or depend on instance/class state, keep it as `@staticmethod`.
- **Avoid misleading definitions**: Every method inside a class must explicitly use `self`, `cls`, or `@staticmethod`. Do not omit them.
- **Avoid abstraction as decoration**: Eliminate over-encapsulation, unnecessary helper layers, and utility dumping.

### Private Helpers and Test Boundaries

- Do not extract a function solely because of line count, phase count, lint complexity, or a desire to test or mock an internal step separately.
- Inline a helper with one production caller when it only renames a sequential block, forwards parameters, or hides obvious control flow without an independent responsibility. Test callers do not count as production reuse; framework registration and indirect calls do count.
- Keep functions with real responsibilities or required boundaries: SDK overrides, registered callbacks, async task entry points, resource/cancellation/transaction boundaries, and independently meaningful domain operations. A single direct caller does not make these functions redundant.
- Do not add production helpers, wrapper classes, public exports, or dependency parameters only for tests. Test observable behavior through existing service, API, or runtime entry points and replace external dependencies at their existing boundaries. An existing function with a real responsibility may still have focused unit tests.
- A leading underscore expresses an internal interface. Do not remove it merely to make tests or style checks easier, and do not add it to shared interfaces used by other production modules. When removing a redundant helper, move its logic back into its owner and update affected tests.
- Prefer module-level functions when no captured state is needed. Keep nested functions when closures, decorators, or callback registration require their local context; do not move them mechanically.

## Documentation and Comments

### Comments

- Use Chinese comments in project code.
- Do not end Chinese comments or docstring prose with `。` or `.`.
- Only add comments when they provide real value.
- Do not add comments that merely restate the code.

```python
if not user.status:
    raise errors.AuthorizationError(msg='用户已被锁定')
```

### Docstring Format

- Use `reStructuredText` style.
- Do not use `:raise:`, `:rtype:`, or similar tags.
- Use `:return:` with no trailing description.

```python
class UserService:
    """用户服务类"""

    @staticmethod
    async def get() -> User:
        """获取用户详情"""
        ...

    @staticmethod
    async def get(*, db: AsyncSession, pk: int) -> User:
        """
        获取用户详情

        :param db: 数据库会话
        :param pk: 用户 ID
        :return:
        """
        ...
```

### API Route Documentation

- `summary` is required.
- `description` is optional.

```python
@router.get(
    '/{pk}',
    summary='获取用户详情',
    description='通过 ID 获取用户详细信息，包括角色和部门',
    dependencies=[DependsJwtAuth],
)
async def get_user(
    db: CurrentSession,
    pk: Annotated[int, Path(description='用户 ID')],
) -> ResponseSchemaModel[GetUserDetail]:
    ...
```

## Function Body Spacing

### Core Rule

Blank lines inside a function body are only allowed to mark a logical phase transition.

Blank lines are not visual decoration. They are part of code structure.

### Rules

- Only use blank lines to separate logical phases.
- Do not insert blank lines inside the same phase.
- Short functions with a single linear flow may contain no blank lines.
- Use at most one blank line between phases.
- Do not use multiple consecutive blank lines.

### Logical Phases

Typical logical phases include:

- input parsing
- parameter validation
- permission validation
- state validation
- intermediate data preparation
- core business execution
- persistence or side effects
- return value assembly

Only add a blank line when moving from one phase to another.

### Additional Requirements

- Consecutive existence checks, permission checks, type checks, and state checks belong to the same phase and must not be split by blank lines.
- Multiple lines that prepare the same target, such as `payload`, `run_input`, `query`, or `persistence`, belong to the same phase and must not be split by blank lines.
- Do not add blank lines inside `if`, `else`, `try`, `except`, `with`, `for`, or `while` blocks unless there is a real phase transition inside the block.
- A blank line before `return` is only allowed when `return` starts a distinct final phase.
- Comments do not justify blank lines by themselves. A blank line is only valid if the comment marks a real new phase.

### Examples

Correct:

```python
class UserService:
    """用户服务类"""

    @staticmethod
    async def get(*, db: AsyncSession, pk: int) -> User:
        """
        获取用户详情

        :param db: 数据库会话
        :param pk: 用户 ID
        :return:
        """
        user = await user_dao.get(db, pk)
        if not user:
            raise errors.NotFoundError(msg='用户不存在')
        if not user.status:
            raise errors.AuthorizationError(msg='用户已被锁定')

        data = await user_dao.get_detail(db, pk)

        return data
```

Incorrect:

```python
class UserService:
    """用户服务类"""

    @staticmethod
    async def get(*, db: AsyncSession, pk: int) -> User:
        """
        获取用户详情

        :param db: 数据库会话
        :param pk: 用户 ID
        :return:
        """
        user = await user_dao.get(db, pk)

        if not user:
            raise errors.NotFoundError(msg='用户不存在')

        if not user.status:
            raise errors.AuthorizationError(msg='用户已被锁定')

        data = await user_dao.get_detail(db, pk)

        return data
```

### Recommended Density

Use the number of phases as a prompt to review responsibility, not as a quota. Do not add/remove blank lines or extract helpers to meet a numeric target. A cohesive workflow may remain in one function; split only when the function-definition rules above justify a separate responsibility or execution boundary.

## Code Formatting

### Ruff

- The project uses `Ruff` for formatting and linting.
- `Ruff` configuration is defined in `pyproject.toml`.
- `Ruff` can enforce structural blank-line rules, but it cannot enforce logical phase spacing inside function bodies.
- Function body spacing must be maintained through code review and disciplined implementation.

### Pre-commit

Built-in CLI:

```bash
fba format
```

For the requested scope, use scoped commands and inspect automatic fixes:

```bash
ruff check --fix <paths>
ruff format <paths>
ruff check --no-fix <paths>
ruff format --check <paths>
```

Do not enable unsafe fixes by default or weaken lint configuration to hide findings.

## Completion Review

Before reporting Python work complete, review the changed code; for a comprehensive style audit, cover every maintained module in the requested scope:

- Check function typing, method binding, service keyword-only arguments, absolute imports, naming, docstrings, Chinese punctuation, and logical phase spacing.
- Trace production callers of extracted/private helpers, including callbacks and framework registrations. Check that tests did not dictate artificial production interfaces.
- Check module ownership and import direction using `development.md` when the task changes structure or crosses layers. After moves, update callers, string-based patch targets, and relevant documentation; remove obsolete modules unless an actual compatibility requirement needs them.
- Run scoped Ruff checks and relevant existing behavioral tests. Add tests when behavior or risk warrants them, not merely to mirror function layout or assert formatting. Report the scope checked and any unresolved limitation; do not equate lint success with complete architectural compliance.
