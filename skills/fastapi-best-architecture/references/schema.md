# Schema 定义规范

## 目录

- [类命名规范](#类命名规范)
- [基类使用](#基类使用)
- [Field 定义](#field-定义)
- [完整示例](#完整示例)
- [驼峰返回](#驼峰返回)

**相关文档**：[API 规范](api.md) | [数据库模型](model.md) | [命名规范](naming.md)

---

## 类命名规范

| 类型             | 命名模式                         | 示例                          |
|----------------|------------------------------|-----------------------------|
| 基础 Schema      | `XxxSchemaBase(SchemaBase)`  | `UserSchemaBase`            |
| 接口入参           | `XxxParam()`                 | `UserParam`                 |
| 新增入参           | `CreateXxxParam()`           | `CreateUserParam`           |
| 更新入参           | `UpdateXxxParam()`           | `UpdateUserParam`           |
| 批量删除入参         | `DeleteXxxParam()`           | `DeleteUserParam`           |
| 查询详情           | `GetXxxDetail()`             | `GetUserDetail`             |
| 查询详情(join)     | `GetXxxWithJoinDetail()`     | `GetUserWithJoinDetail`     |
| 查询详情(relation) | `GetXxxWithRelationDetail()` | `GetUserWithRelationDetail` |
| 查询树            | `GetXxxTree()`               | `GetMenuTree`               |

---

## 基类使用

所有 Schema 应继承自 `SchemaBase`：

```python
from backend.common.schema import SchemaBase


class UserSchemaBase(SchemaBase):
    """用户基础模型"""

    username: str
    email: str | None = None
```

---

## Field 定义

### 必填字段

**不建议**将必填字段默认值设置为 `...`：

```python
# 推荐 ✓
username: str = Field(description='用户名')

# 不推荐 ✗
username: str = Field(..., description='用户名')
```

### description 参数

**建议**为所有字段添加 `description` 参数，这对于 API 文档非常有用：

```python
class CreateUserParam(SchemaBase):
    username: str = Field(description='用户名')
    email: str | None = Field(None, description='邮箱')
    status: int = Field(default=1, description='状态(0禁用 1启用)')
```

### 可选字段

更新入参通常所有字段都是可选的：

```python
class UpdateUserParam(SchemaBase):
    username: str | None = Field(None, description='用户名')
    email: str | None = Field(None, description='邮箱')
    status: int | None = Field(None, description='状态')
```

---

## 完整示例

```python
from datetime import datetime

from pydantic import Field

from backend.common.schema import SchemaBase


class ArticleSchemaBase(SchemaBase):
    """文章基础模型"""

    title: str = Field(description='标题')
    content: str = Field(description='内容')
    status: int = Field(default=1, description='状态(0草稿 1发布)')


class CreateArticleParam(ArticleSchemaBase):
    """创建文章参数"""

    pass


class UpdateArticleParam(SchemaBase):
    """更新文章参数"""

    title: str | None = Field(None, description='标题')
    content: str | None = Field(None, description='内容')
    status: int | None = Field(None, description='状态')


class DeleteArticleParam(SchemaBase):
    """批量删除文章参数"""

    ids: list[int] = Field(description='文章ID列表')


class GetArticleDetail(ArticleSchemaBase):
    """文章详情响应"""

    id: int = Field(description='文章ID')
    created_time: datetime = Field(description='创建时间')
    updated_time: datetime | None = Field(None, description='更新时间')


class GetArticleWithAuthorDetail(GetArticleDetail):
    """文章详情响应（含作者信息）"""

    author_name: str = Field(description='作者名称')
```

---

## 驼峰返回

如需响应数据自动转为小驼峰命名（如 `created_time` → `createdTime`），修改 `backend/common/schema.py`：

```python
from pydantic import ConfigDict
from pydantic.alias_generators import to_camel


class SchemaBase(BaseModel):
    model_config = ConfigDict(
        populate_by_name=True,
        alias_generator=to_camel,
    )
```

配置后，响应数据将自动转换：

```json
{
  "id": 1,
  "title": "文章标题",
  "createdTime": "2024-01-01T00:00:00",
  "updatedTime": null
}
```
