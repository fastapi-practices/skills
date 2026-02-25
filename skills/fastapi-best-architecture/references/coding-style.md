# Coding Style Reference

## Import Rules

- Each import statement should only import one module
- Avoid using `from xxx import *`
- Use absolute imports, avoid relative imports
- No shebang (`#!/usr/bin/env python3`) or encoding declaration (`# -*- coding: utf-8 -*-`) needed
- No file description needed
- No `__all__` needed
- Do not add anything in `__init__.py` unless explicitly required

## Typing

### Typing Are Required

All function parameters and return values should have typing:

```python
async def get_user(db: AsyncSession, pk: int) -> User:
    ...


async def create_user(db: AsyncSession, obj: CreateUserParam) -> None:
    ...
```

### Use Annotated for Enhanced Types

```python
from typing import Annotated
from fastapi import Depends, Path, Query


async def get_user(
    db: CurrentSession,
    pk: Annotated[int, Path(description='用户 ID')],
    username: Annotated[str | None, Query(description='用户名')] = None,
) -> ResponseSchemaModel[GetUserDetail]:
    ...
```

### Use `|` for Union Types

```python
result: str | int | None = None
```

## Async Handling

### Use async/await for All I/O Operations

**API**

```python
@router.get('/{pk}')
async def get_user(db: CurrentSession, pk: int) -> ResponseSchemaModel[GetUserDetail]:
    data = await user_service.get(db=db, pk=pk)
    return response_base.success(data=data)
```

**Service**

```python
class UserService:
    @staticmethod
    async def get(*, db: AsyncSession, pk: int) -> User:
        user = await user_dao.get(db, pk)
        if not user:
            raise errors.NotFoundError(msg='用户不存在')
        return user
```

**CRUD**

```python
class CRUDUser(CRUDPlus[User]):
    async def get(self, db: AsyncSession, pk: int) -> User | None:
        return await self.select_model(db, pk)
```

## Keyword Arguments

### Service Layer Must Use Keyword Arguments

Use `*` to force callers to use keyword arguments:

```python
class UserService:
    @staticmethod
    async def get(*, db: AsyncSession, pk: int) -> User:
        ...

    @staticmethod
    async def create(*, db: AsyncSession, obj: CreateUserParam) -> None:
        ...
```

## Documentation and Comments

### Comments

The project uses Chinese comments

```python
if not user.status:
    raise errors.AuthorizationError(msg='用户已被锁定')
```

### Docstring Format

- Use reStructuredText style
- Do not use `:raise:` `:rtype:` or other tags

The following examples include class, function without parameters, and function with parameters

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

- `summary` is required
- `description` is optional

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

## Code Formatting

### Ruff Configuration

The project uses Ruff for code formatting and linting, Ruff configuration is located in pyproject.toml

### Pre-commit

```bash
fba format
```

