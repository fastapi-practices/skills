# Naming Conventions

## File and Directory Naming

All lowercase, separated by underscores.

- `crud_user.py`
- `user_service.py`

## Class Naming

All PascalCase.

The class stem `Xxx` comes from the **filename entity**, not from the plugin or app directory path.

```python
class UserService:
    ...


class CRUDUser:
    ...


class User:
    ...
```

Do not concatenate the file hierarchy into a class name. If the model file is `template.py`, the class is based on `Template`, not on the parent directories.

## Entity Stem

Pick one entity stem `Xxx` from the model filename and reuse it across layers:

| Layer   | File                  | Class             |
|---------|-----------------------|-------------------|
| Model   | `user.py`             | `User`            |
| Schema  | `user.py`             | `UserSchemaBase`  |
| CRUD    | `crud_user.py`        | `CRUDUser`        |
| Service | `user_service.py`     | `UserService`     |
| Table   | `__tablename__`       | `sys_user`        |

Keep the same stem in Schema, CRUD, Service, API functions, and the table name. When a namespace prefix is required, apply it consistently to the class and the table, still based on this stem.

## Namespace Prefix

Use a **short domain prefix** for ownership and collision avoidance. Do not dump the plugin folder name into every class and table.

Plugin identity belongs in the plugin directory, frontend route, and menu name. It does not belong in every class or table.

| Kind | Table prefix | Class prefix | Notes |
|------|--------------|--------------|--------|
| Core `admin` app | `sys_` | none | `User` / `sys_user` |
| Extend-level into `admin` | `sys_` | usually none | `DictData` / `sys_dict_data`, `Notice` / `sys_notice` |
| App-level plugin | short domain | matching PascalCase | choose a short token, not the full directory name |
| Related plugins in one domain | share the domain prefix | share the domain prefix | do not give each related plugin its own folder-derived prefix |

Rules:

- Extend-level plugins that inject into `admin` should use `sys_` on tables. The plugin folder does not need to start with `sys_`.
- App-level plugins choose a short domain prefix, not the full directory name. Prefer a short token that will not collide with other modules.
- Related plugins in the same domain share that short prefix.
- Do not invent a third prefix set in the same plugin. New code must follow the existing stem and prefix of that plugin.
- Do not rename existing tables only because the plugin folder was renamed.

## Schema Naming And Definition Order

Following these naming conventions:

| Type                   | Naming Pattern               | Example                     |
|------------------------|------------------------------|-----------------------------|
| Base Schema            | `XxxSchemaBase(SchemaBase)`  | `UserSchemaBase`            |
| API param              | `XxxParam()`                 | `UserParam`                 |
| Create param           | `CreateXxxParam()`           | `CreateUserParam`           |
| Update param           | `UpdateXxxParam()`           | `UpdateUserParam`           |
| Batch delete param     | `DeleteXxxParam()`           | `DeleteUserParam`           |
| Get details            | `GetXxxDetail()`             | `GetUserDetail`             |
| Get details (join)     | `GetXxxWithJoinDetail()`     | `GetUserWithJoinDetail`     |
| Get details (relation) | `GetXxxWithRelationDetail()` | `GetUserWithRelationDetail` |
| Get tree               | `GetXxxTree()`               | `GetMenuTree`               |

## API Function Naming And Definition Order

Lowercase with underscores, paginated lists use `_paginated` suffix:

| Operation      | Naming Pattern       | Example               |
|----------------|----------------------|-----------------------|
| Get all        | `get_all_xxxs`       | `get_all_users`       |
| Paginated list | `get_xxxs_paginated` | `get_users_paginated` |
| Get details    | `get_xxx`            | `get_user`            |
| Create         | `create_xxx`         | `create_user`         |
| Update         | `update_xxx`         | `update_user`         |
| Delete         | `delete_xxx`         | `delete_user`         |
| Batch delete   | `delete_xxxs`        | `delete_users`        |

## Service Method Naming And Definition Order

Following these naming conventions:

| Method       | Purpose              |
|--------------|----------------------|
| `get_all()`  | Get all              |
| `get()`      | Get details          |
| `get_list()` | Get list (paginated) |
| `create()`   | Create               |
| `update()`   | Update               |
| `delete()`   | Delete               |

## CRUD Method Naming And Definition Order

Following these naming conventions:

| Method                | Purpose                       |
|-----------------------|-------------------------------|
| `get()`               | Get/query details             |
| `get_by_xxx()`        | Get/query details by xxx      |
| `get_select()`        | Get/query list expression     |
| `get_list()`          | Get/query list                |
| `get_all()`           | Get/query all                 |
| `get_with_join()`     | Join query (join)             |
| `get_with_relation()` | Relation query (relationship) |
| `get_children()`      | Sub-query                     |
| `create()`            | Create                        |
| `update()`            | Update                        |
| `delete()`            | Delete                        |
