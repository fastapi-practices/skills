# 编码风格规范

## 目录

- [Import 组织](#import-组织)
- [类型注解](#类型注解)
- [异步处理](#异步处理)
- [关键字参数](#关键字参数)
- [文档和注释](#文档和注释)
- [代码格式化](#代码格式化)

**相关文档**：[API 规范](api.md) | [Schema 定义](schema.md) | [命名规范](naming.md)

---

## Import 组织

按以下顺序组织，每组之间空一行：

```python
# 1. 标准库
from datetime import datetime
from typing import Annotated, Any

# 2. 第三方库
from fastapi import APIRouter, Depends, Path
from sqlalchemy.ext.asyncio import AsyncSession

# 3. 本地模块
from backend.app.admin.schema.user import CreateUserParam
from backend.common.exception import errors
from backend.common.response.response_schema import ResponseModel
```

### Import 规范

- 每个 import 语句只导入一个模块
- 避免使用 `from xxx import *`
- 使用绝对导入，避免相对导入
- **不需要** shebang (`#!/usr/bin/env python3`) 和编码声明 (`# -*- coding: utf-8 -*-`)

---

## 类型注解

### 必须添加类型注解

所有函数参数和返回值都应有类型注解：

```python
async def get_user(db: AsyncSession, pk: int) -> User:
    pass


async def create_user(db: AsyncSession, obj: CreateUserParam) -> None:
    pass
```

### 使用 Annotated 增强类型

```python
from typing import Annotated
from fastapi import Depends, Path, Query


async def get_user(
    db: Annotated[AsyncSession, Depends(get_db)],
    pk: Annotated[int, Path(description='用户 ID')],
    username: Annotated[str | None, Query(description='用户名')] = None,
) -> ResponseSchemaModel[GetUserDetail]:
    pass
```

### Union 类型使用 | 操作符

```python
# 推荐 ✓ (Python 3.10+)
result: str | int | None = None

# 不推荐 ✗
from typing import Union

result: Union[str, int, None] = None
```

---

## 异步处理

### 所有 I/O 操作使用 async/await

```python
# API 层
@router.get('/users/{pk}')
async def get_user(db: CurrentSession, pk: int) -> ResponseSchemaModel[GetUserDetail]:
    user = await user_service.get(db=db, pk=pk)
    return response_base.success(data=user)


# Service 层
class UserService:
    @staticmethod
    async def get(*, db: AsyncSession, pk: int) -> User:
        user = await user_dao.get(db, pk)
        if not user:
            raise errors.NotFoundError(msg='用户不存在')
        return user


# CRUD 层
class CRUDUser(CRUDPlus[User]):
    async def get(self, db: AsyncSession, pk: int) -> User | None:
        return await self.select_model(db, pk)
```

---

## 关键字参数

### Service 层强制使用关键字参数

使用 `*` 强制调用者使用关键字参数：

```python
class UserService:
    @staticmethod
    async def get(*, db: AsyncSession, pk: int) -> User:
        """* 之后的参数必须使用关键字传递"""
        pass

    @staticmethod
    async def create(*, db: AsyncSession, obj: CreateUserParam) -> None:
        pass


# 调用时
await user_service.get(db=db, pk=pk)  # ✓ 正确
await user_service.get(db, pk)  # ✗ 错误，会报 TypeError
```

---

## 文档和注释

### 中文注释

项目使用中文注释：

```python
# 获取用户信息
user = await user_dao.get(db, pk)

# 用户状态为0表示已停用，需要阻止登录
if not user.status:
    raise errors.AuthorizationError(msg='用户已被锁定')
```

### Docstring 格式

- 使用 reStructuredText 风格
- 类：单行 """Xxx 服务类"""
- 方法：无参数单行；有参数多行格式，描述后空一行，:param 参数说明，:return: 后不添加内容
- 不使用 :raise:、:rtype: 等其他标签

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
        pass
```

### API 路由文档

```python
@router.get(
    '/{pk}',
    summary='获取用户详情',  # 必须添加
    description='通过 ID 获取用户详细信息，包括角色和部门',  # 可选添加
    dependencies=[DependsJwtAuth],
)
async def get_user(
    db: CurrentSession,
    pk: Annotated[int, Path(description='用户 ID')],
) -> ResponseSchemaModel[GetUserDetail]:
    pass
```

---

## 代码格式化

### Ruff 配置

项目使用 Ruff 进行代码格式化和检查，配置文件为 `.ruff.toml`

### Pre-commit 钩子

```bash
prek run --all-files
```

### 代码风格要点

- 行长度限制：120 字符
- 使用 4 空格缩进
- 类和函数之间空一行
- 方法之间空两行
- 字符串使用单引号
- 文件末尾保留一个空行
