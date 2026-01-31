# API 和路由规范

## 目录

- [路由组织结构](#路由组织结构)
- [事务处理](#事务处理)
- [响应规范](#响应规范)
- [JWT 认证](#jwt-认证)
- [RBAC 权限](#rbac-权限)
- [限流](#限流)
- [国际化](#国际化)

**相关文档**：[Schema 定义](schema.md) | [编码风格](coding-style.md) | [插件开发](plugin.md)

---

## 路由组织结构

fba 中的路由遵循 RESTful API 规范

### 包含子包的应用

```
backend/
└── app/
    └── xxx/                        # 自定义应用
        ├── __init__.py
        └── api/
            ├── __init__.py
            ├── router.py           # 在此文件内注册所有子包 __init__.py 文件中的路由
            └── v1/
                ├── __init__.py
                ├── auth/           # 子包
                │   ├── __init__.py # 在此文件内注册子包内 xxx.py 文件中的路由
                │   ├── auth.py
                │   └── captcha.py
                └── sys/            # 子包
                    ├── __init__.py # 在此文件内注册子包内 xxx.py 文件中的路由
                    ├── user.py
                    └── role.py
```

### 不包含子包的应用

```
backend/
└── app/
    └── xxx/                        # 自定义应用
        ├── __init__.py
        └── api/
            ├── __init__.py
            ├── router.py           # 在此文件内注册所有 xxx.py 文件中的路由
            └── v1/
                ├── __init__.py     # 不做任何操作
                ├── task.py
                └── job.py
```

### 路由导入规范

统一命名所有接口路由参数为 `router`，导入时务必使用 `as` 别名避免冲突：

```python
# 正确 ✓
from backend.app.admin.api.v1.sys.api import router as api_router
from backend.app.admin.api.v1.sys.user import router as user_router

# 错误 ✗ - 会导致命名冲突
from backend.app.admin.api.v1.sys.api import router
from backend.app.admin.api.v1.sys.user import router
```

### RESTful 路由约定

```
GET    /api/v1/resources/all     # 所有（不分页）
GET    /api/v1/resources         # 列表（分页）
GET    /api/v1/resources/{pk}    # 详情
POST   /api/v1/resources         # 创建
PUT    /api/v1/resources/{pk}    # 更新
DELETE /api/v1/resources/{pk}    # 删除
DELETE /api/v1/resources         # 批量删除
```

---

## 事务处理

### CurrentSession（只读会话）

用于查询操作，**不自动开启事务**：

```python
from backend.database.db import CurrentSession


@router.get('/users')
async def get_users(db: CurrentSession) -> ResponseModel:
    users = await user_service.get_list(db=db)
    return response_base.success(data=users)
```

### CurrentSessionTransaction（事务会话）

用于增删改操作，**自动开启事务并提交**：

```python
from backend.database.db import CurrentSessionTransaction


@router.post('/users')
async def create_user(db: CurrentSessionTransaction, obj: CreateUserParam) -> ResponseModel:
    await user_service.create(db=db, obj=obj)
    return response_base.success()
```

### 手动事务（begin）

用于需要在任意位置开启事务的场景：

```python
from backend.database.db import async_db_session


async def create(*, obj: CreateIns) -> None:
    async with async_db_session.begin() as db:
        await xxx_dao.create(db, obj)
```

---

## 响应规范

### 响应模型

```python
from backend.common.response.response_schema import ResponseModel, ResponseSchemaModel, response_base


# 无数据响应
@router.delete('/{pk}')
async def delete_item(db: CurrentSessionTransaction, pk: int) -> ResponseModel:
    await item_service.delete(db=db, pk=pk)
    return response_base.success()


# 带数据响应（Swagger 文档可见数据结构）
@router.get('/{pk}')
async def get_item(db: CurrentSession, pk: int) -> ResponseSchemaModel[GetItemDetail]:
    item = await item_service.get(db=db, pk=pk)
    return response_base.success(data=item)
```

### 返回方法

| 方法 | 用途 | 默认响应 |
|------|------|----------|
| `response_base.success()` | 成功响应 | `{"code": 200, "msg": "请求成功", "data": null}` |
| `response_base.fail()` | 失败响应 | `{"code": 400, "msg": "请求错误", "data": null}` |
| `response_base.fast_success()` | 高性能响应（大型 JSON） | 同 success，但跳过 Pydantic 验证 |

### 驼峰返回

如需响应数据自动转为小驼峰命名，修改 `backend/common/schema.py`：

```python
from pydantic.alias_generators import to_camel


class SchemaBase(BaseModel):
    model_config = ConfigDict(
        populate_by_name=True,
        alias_generator=to_camel,
    )
```

---

## JWT 认证

### 接口鉴权

```python
from backend.common.security.jwt import DependsJwtAuth


@router.get('/users', summary='获取用户列表', dependencies=[DependsJwtAuth])
async def get_users(db: CurrentSession) -> ResponseModel:
    pass
```

### Token 授权方式

fba 内置 token 授权方式遵循 [RFC 6750](https://datatracker.ietf.org/doc/html/rfc6750)：

- **Swagger 登录**: 快捷授权方式，仅用于调试
- **验证码登录**: 配合前端实现登录授权

---

## RBAC 权限

### 角色菜单模式（默认）

```python
from fastapi import Depends
from backend.common.security.permission import RequestPermission
from backend.common.security.rbac import DependsRBAC


@router.post(
    '/users',
    summary='创建用户',
    dependencies=[
        Depends(RequestPermission('sys:user:add')),  # 权限标识
        DependsRBAC,  # RBAC 鉴权
    ],
)
async def create_user(db: CurrentSessionTransaction, obj: CreateUserParam) -> ResponseModel:
    pass
```

### 权限标识格式

`模块:资源:操作`，例如：

- `sys:user:add` - 添加用户
- `sys:user:edit` - 编辑用户
- `sys:user:del` - 删除用户
- `sys:role:list` - 角色列表

---

## 限流

```python
from fastapi import Depends
from fastapi_limiter.depends import RateLimiter


# 1分钟最多5次请求
@router.post('/login', dependencies=[Depends(RateLimiter(times=5, minutes=1))])
async def login():
    pass


# 组合限流：1分钟最多5次 + 30秒最多3次
@router.post('/login', dependencies=[
    Depends(RateLimiter(times=5, minutes=1)),
    Depends(RateLimiter(times=3, seconds=30)),
])
async def login():
    pass
```

---

## 国际化

### 使用语法

```python
from backend.common.i18n import t

# 获取语言包中的字段值
msg = t('response.success')

# 链式获取
msg = t('error.captcha.expired')
```

### 语言包位置

`backend/locale` 目录，支持 json 和 yaml/yml 格式

### 动态切换

自动获取请求头中的 `Accept-Language` 参数
