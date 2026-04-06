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

- All function parameters and return values must be typed.
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

## Documentation and Comments

### Comments

- Use Chinese comments in project code.
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

- Single linear short function: `0` blank lines
- Typical function: `1` to `3` blank lines
- Complex function: avoid more than `4` blank lines
- If a function needs `5+` internal phase splits, reconsider its responsibility and split the function instead

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

Generic commands:

```bash
ruff format
```

```bash
ruff check --fix --unsafe-fixes
```
