# 命名规范

## 目录

- [文件和目录命名](#文件和目录命名)
- [类命名](#类命名)
- [CRUD 方法命名](#crud-方法命名)
- [Service 方法命名](#service-方法命名)
- [API 函数命名](#api-函数命名)
- [Schema 命名](#schema-命名)

**相关文档**：[编码风格](coding-style.md) | [Schema 定义](schema.md) | [数据库模型](model.md)

---

## 文件和目录命名

全部小写，下划线分隔：

```
crud_user.py              # CRUD 文件
user_service.py           # Service 文件
```

## 类命名

大驼峰 PascalCase：

```python
class UserService:  # 服务类
    ...


class CRUDUser:  # CRUD 类
    ...


class User:  # 模型类
    ...
```

## CRUD 方法命名

fba 使用 [sqlalchemy-crud-plus](https://github.com/fastapi-practices/sqlalchemy-crud-plus)，遵循以下命名：

| 方法名                   | 用途                 |
|-----------------------|--------------------|
| `get()`               | 获取/查询详情            |
| `get_by_xxx()`        | 通过 xxx 获取/查询详情     |
| `get_select()`        | 获取/查询列表表达式         |
| `get_list()`          | 获取/查询列表            |
| `get_all()`           | 获取/查询所有            |
| `get_with_join()`     | 连接查询（join）         |
| `get_with_relation()` | 关系查询（relationship） |
| `get_children()`      | 子查询                |
| `create()`            | 创建                 |
| `update()`            | 更新                 |
| `delete()`            | 删除                 |

## Service 方法命名

与 CRUD 层保持一致，使用 `*` 强制关键字参数：

| 方法名          | 用途       |
|--------------|----------|
| `get()`      | 获取详情     |
| `get_list()` | 获取列表（分页） |
| `get_all()`  | 获取所有     |
| `create()`   | 创建       |
| `update()`   | 更新       |
| `delete()`   | 删除       |

## API 函数命名

小写下划线，分页列表使用 `_paginated` 后缀：

| 操作   | 命名模式                 | 示例                    |
|------|----------------------|-----------------------|
| 分页列表 | `get_xxxs_paginated` | `get_users_paginated` |
| 获取详情 | `get_xxx`            | `get_user`            |
| 创建   | `create_xxx`         | `create_user`         |
| 更新   | `update_xxx`         | `update_user`         |
| 删除   | `delete_xxx`         | `delete_user`         |
| 批量删除 | `delete_xxxs`        | `delete_users`        |

## Schema 命名

详见 [schema.md](schema.md#类命名规范)

| 类型        | 命名模式             | 示例                |
|-----------|------------------|-------------------|
| 基础 Schema | `XxxSchemaBase`  | `UserSchemaBase`  |
| 新增入参      | `CreateXxxParam` | `CreateUserParam` |
| 更新入参      | `UpdateXxxParam` | `UpdateUserParam` |
| 查询详情      | `GetXxxDetail`   | `GetUserDetail`   |
