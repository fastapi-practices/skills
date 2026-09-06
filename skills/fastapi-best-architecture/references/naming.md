# Naming Conventions

## File and Directory Naming

All lowercase, separated by underscores.

- `crud_user.py`
- `user_service.py`

## Class Naming

All PascalCase.

The class stem `Xxx` comes from the **filename entity**.

When a namespace prefix is required, put it in front of that stem. Do not splice extra path segments such as `backend`, `plugin`, `app`, `model`, or nested folders inside the plugin.

```python
class UserService:
    ...


class CRUDUser:
    ...


class User:
    ...
```

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

The namespace prefix identifies **which plugin or app owns the type**. It is not optional for app-level plugins.

| Kind | Table prefix | Class prefix | Notes |
|------|--------------|--------------|--------|
| Core `admin` app | `sys_` | none | `User` / `sys_user` |
| Extend-level into `admin` | `sys_` | usually none | `DictData` / `sys_dict_data`, `Notice` / `sys_notice` |
| App-level plugin | `{plugin_dir}_` | PascalCase of `{plugin_dir}` | file `item.py` in plugin `order_center` → `OrderCenterItem` / `order_center_item` |
| Related plugins in one domain | share the primary plugin prefix | share the primary plugin prefix | do not invent a second prefix from the related plugin's own folder |

Rules:

- Extend-level plugins that inject into `admin` should use `sys_` on tables. The plugin folder does not need to start with `sys_`.
- App-level plugins use the plugin directory itself as the prefix. Convert it to PascalCase for classes and keep the snake_case directory plus `_` for tables. Do not strip it down to a generic token that no longer identifies the plugin.
- If the directory name is long, a unique abbreviation that still names the plugin is allowed. A leftover generic token is not.
- Related plugins in the same domain share the **primary** plugin prefix, including when one plugin extends another.
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
