# 数据库模型规范

## 目录

- [模型基类](#模型基类)
- [字段类型](#字段类型)
- [主键模式](#主键模式)
- [数据库迁移](#数据库迁移)
- [完整示例](#完整示例)

**相关文档**：[Schema 定义](schema.md) | [命名规范](naming.md) | [配置参考](config.md)

---

## 模型基类

```python
from sqlalchemy.orm import Mapped, mapped_column
from backend.common.model import Base, id_key


class MyModel(Base):
    """模型表 - docstring 会作为表注释"""

    __tablename__ = 'my_model'  # 显式指定表名

    # 主键必须显式定义
    id: Mapped[id_key] = mapped_column(init=False)

    # 字段定义
    name: Mapped[str] = mapped_column(comment='名称')
    status: Mapped[int] = mapped_column(default=1, comment='状态')

    # Base 自动添加 created_time, updated_time
```

## 字段类型

```python
import sqlalchemy as sa
from backend.common.model import TimeZone, UniversalText

# 字符串（常用长度：32、64、128、256、512）
name: Mapped[str] = mapped_column(sa.String(64), comment='名称')

# 可空字符串
email: Mapped[str | None] = mapped_column(sa.String(256), default=None, comment='邮箱')

# 整数
status: Mapped[int] = mapped_column(default=1, comment='状态')

# 布尔
is_active: Mapped[bool] = mapped_column(default=True, comment='是否激活')

# 日期时间（兼容时区）
event_time: Mapped[datetime] = mapped_column(TimeZone, comment='事件时间')

# 长文本（兼容 MySQL/PostgreSQL）
content: Mapped[str] = mapped_column(UniversalText, comment='内容')

# 唯一索引
username: Mapped[str] = mapped_column(sa.String(64), unique=True, index=True, comment='用户名')
```

## 主键模式

通过 `DATABASE_PK_MODE` 配置：

- **autoincrement**: 自增 ID（默认）
- **snowflake**: 雪花算法 ID

> ⚠️ **警告**: 不要随意切换主键模式，否则将导致致命问题！

## 数据库迁移

```bash
# 生成迁移脚本
fba alembic revision --autogenerate -m "描述信息"

# 执行迁移
fba alembic upgrade head

# 回滚
fba alembic downgrade -1
```

## 完整示例

```python
import sqlalchemy as sa
from sqlalchemy.orm import Mapped, mapped_column
from backend.common.model import Base, id_key, TimeZone, UniversalText


class Article(Base):
    """文章表"""

    __tablename__ = 'sys_article'

    id: Mapped[id_key] = mapped_column(init=False)
    title: Mapped[str] = mapped_column(sa.String(256), comment='标题')
    content: Mapped[str] = mapped_column(UniversalText, comment='内容')
    author_id: Mapped[int] = mapped_column(sa.BigInteger, index=True, comment='作者ID')
    status: Mapped[int] = mapped_column(default=1, comment='状态(0草稿 1发布)')
    published_at: Mapped[datetime | None] = mapped_column(TimeZone, default=None, comment='发布时间')
    view_count: Mapped[int] = mapped_column(default=0, comment='浏览次数')
```
