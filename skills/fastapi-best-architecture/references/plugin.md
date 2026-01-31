# 插件开发规范

## 目录

- [插件类型](#插件类型)
- [plugin.toml 配置](#plugintoml-配置)
- [插件目录结构](#插件目录结构)
- [热插拔配置](#热插拔配置)
- [完整示例](#完整示例)
- [注意事项](#注意事项)

**相关文档**：[API 规范](api.md) | [数据库模型](model.md) | [配置参考](config.md)

---

## 插件类型

### 应用级插件（App Level）

像应用一样被注入到系统中，有完整的路由结构

```toml
[app]
router = ['v1']
```

### 扩展级插件（Extend Level）

被注入到 app 目录下已存在的应用中，必须 1:1 复制目标应用的 api 目录结构

```toml
[app]
extend = 'admin'
```

---

## plugin.toml 配置

### 应用级插件

```toml
[plugin]
icon = 'assets/icon.svg'          # 图标
summary = '代码生成'               # 摘要
version = '0.1.1'                 # 版本号 (x.y.z)
description = '生成通用业务代码'   # 描述
author = 'your-name'              # 作者
tags = ['other']                  # 标签: ai, mcp, agent, auth, storage, notification, task, payment, other
database = ['mysql', 'postgresql'] # 数据库支持

[app]
router = ['v1']

[settings]
MY_PLUGIN_CONFIG = 'value'        # 热插拔配置项
```

### 扩展级插件

```toml
[plugin]
icon = 'assets/icon.svg'
summary = '参数配置'
version = '0.0.2'
description = '动态配置系统参数'
author = 'your-name'
tags = ['other']
database = ['mysql', 'postgresql']

[app]
extend = 'admin'

[api.config]                      # xxx 对应接口文件名
prefix = '/configs'               # 路由前缀，必须以 '/' 开头
tags = '系统参数配置'              # Swagger 标签

[settings]
MY_PLUGIN_CONFIG = 'value'
```

---

## 插件目录结构

```
backend/plugin/my_plugin/         # 插件名（必须）
├── __init__.py                   # 作为 python 包保留（必须）
├── plugin.toml                   # 插件配置文件（必须）
├── README.md                     # 插件说明和联系方式（必须）
├── requirements.txt              # 依赖包文件
├── .env.example                  # 环境变量示例
├── api/                          # 接口（必须）
│   ├── __init__.py
│   ├── router.py                 # 应用级插件需要
│   └── v1/
│       └── my_feature.py
├── crud/                         # CRUD
│   ├── __init__.py
│   └── crud_my_model.py
├── model/                        # 模型
│   ├── __init__.py               # 在此导入所有模型类（目录存在则必须）
│   └── my_model.py
├── schema/                       # 数据传输
│   ├── __init__.py
│   └── my_schema.py
├── service/                      # 服务
│   ├── __init__.py
│   └── my_service.py
├── sql/                          # SQL 脚本（如需要）
│   ├── mysql/
│   │   ├── init.sql              # 自增 id 模式
│   │   └── init_snowflake.sql    # 雪花 id 模式
│   └── postgresql/
│       ├── init.sql
│       └── init_snowflake.sql
└── utils/                        # 工具包
```

---

## 热插拔配置

### 插件环境变量

在插件根目录添加 `.env.example`：

```env
# [ Plugin ] my_plugin
MY_PLUGIN_USERNAME=
MY_PLUGIN_PASSWORD=
```

### 插件基础配置

在 `plugin.toml` 的 `[settings]` 中添加：

```toml
[settings]
MY_PLUGIN_HOST = 'localhost'
MY_PLUGIN_PORT = 8080
MY_PLUGIN_ENABLED = true
```

### 全局配置（可选）

如需 IDE 键入提示，在 `backend/core/conf.py` 中添加：

```python
##################################################
# [ Plugin ] my_plugin
##################################################
# .env
MY_PLUGIN_USERNAME: str
MY_PLUGIN_PASSWORD: str

# 基础配置
MY_PLUGIN_HOST: str
MY_PLUGIN_PORT: int
MY_PLUGIN_ENABLED: bool
```

---

## 完整示例

### Model

```python
# backend/plugin/my_plugin/model/my_model.py
from sqlalchemy.orm import Mapped, mapped_column
from backend.common.model import Base, id_key


class MyModel(Base):
    """我的模型表"""

    __tablename__ = 'plugin_my_model'

    id: Mapped[id_key] = mapped_column(init=False)
    name: Mapped[str] = mapped_column(comment='名称')
    status: Mapped[int] = mapped_column(default=1, comment='状态(0禁用 1启用)')
```

### Schema

```python
# backend/plugin/my_plugin/schema/my_schema.py
from pydantic import Field
from backend.common.schema import SchemaBase


class MyModelSchemaBase(SchemaBase):
    name: str = Field(description='名称')
    status: int = Field(default=1, description='状态')


class CreateMyModelParam(MyModelSchemaBase):
    pass


class UpdateMyModelParam(SchemaBase):
    name: str | None = Field(None, description='名称')
    status: int | None = Field(None, description='状态')


class GetMyModelDetail(MyModelSchemaBase):
    id: int = Field(description='ID')
```

### CRUD

```python
# backend/plugin/my_plugin/crud/crud_my_model.py
from sqlalchemy import Select
from sqlalchemy.ext.asyncio import AsyncSession

from sqlalchemy_crud_plus import CRUDPlus

from backend.plugin.my_plugin.model import MyModel
from backend.plugin.my_plugin.schema.my_schema import CreateMyModelParam, UpdateMyModelParam


class CRUDMyModel(CRUDPlus[MyModel]):
    """Xxx 数据库操作类"""

    async def get(self, db: AsyncSession, pk: int) -> MyModel | None:
        """
        获取 Xxx

        :param db: 数据库会话
        :param pk: Xxx ID
        :return:
        """
        return await self.select_model(db, pk)

    async def get_select(self, name: str | None, status: int | None) -> Select:
        """
        获取 Xxx 列表查询表达式

        :param name: 名称
        :param status: 状态
        :return:
        """
        filters = {}
        if name is not None:
            filters['name__like'] = f'%{name}%'
        if status is not None:
            filters['status'] = status
        return await self.select_order('id', 'desc', **filters)

    async def create(self, db: AsyncSession, obj: CreateMyModelParam) -> None:
        """
        创建 Xxx

        :param db: 数据库会话
        :param obj: 创建 Xxx 参数
        :return:
        """
        await self.create_model(db, obj)

    async def update(self, db: AsyncSession, pk: int, obj: UpdateMyModelParam) -> int:
        """
        更新 Xxx

        :param db: 数据库会话
        :param pk: Xxx ID
        :param obj: 更新 Xxx 参数
        :return:
        """
        return await self.update_model(db, pk, obj)

    async def delete(self, db: AsyncSession, pk: int) -> int:
        """
        删除 Xxx

        :param db: 数据库会话
        :param pk: Xxx ID
        :return:
        """
        return await self.delete_model(db, pk)


my_model_dao: CRUDMyModel = CRUDMyModel(MyModel)
```

### Service

```python
# backend/plugin/my_plugin/service/my_service.py
from typing import Any

from sqlalchemy.ext.asyncio import AsyncSession

from backend.common.exception import errors
from backend.common.pagination import paging_data
from backend.plugin.my_plugin.crud.crud_my_model import my_model_dao
from backend.plugin.my_plugin.model import MyModel
from backend.plugin.my_plugin.schema.my_schema import CreateMyModelParam, UpdateMyModelParam


class MyModelService:
    """Xxx 服务类"""

    @staticmethod
    async def get(*, db: AsyncSession, pk: int) -> MyModel:
        """
        获取 Xxx

        :param db: 数据库会话
        :param pk: Xxx ID
        :return:
        """
        model = await my_model_dao.get(db, pk)
        if not model:
            raise errors.NotFoundError(msg='记录不存在')
        return model

    @staticmethod
    async def get_list(*, db: AsyncSession, name: str | None, status: int | None) -> dict[str, Any]:
        """
        获取 Xxx 列表

        :param db: 数据库会话
        :param name: 名称
        :param status: 状态
        :return:
        """
        select = await my_model_dao.get_select(name, status)
        return await paging_data(db, select)

    @staticmethod
    async def create(*, db: AsyncSession, obj: CreateMyModelParam) -> None:
        """
        创建 Xxx

        :param db: 数据库会话
        :param obj: 创建 Xxx 参数
        :return:
        """
        await my_model_dao.create(db, obj)

    @staticmethod
    async def update(*, db: AsyncSession, pk: int, obj: UpdateMyModelParam) -> int:
        """
        更新 Xxx

        :param db: 数据库会话
        :param pk: Xxx ID
        :param obj: 更新 Xxx 参数
        :return:
        """
        model = await my_model_dao.get(db, pk)
        if not model:
            raise errors.NotFoundError(msg='记录不存在')
        count = await my_model_dao.update(db, pk, obj)
        return count

    @staticmethod
    async def delete(*, db: AsyncSession, pk: int) -> int:
        """
        删除 Xxx

        :param db: 数据库会话
        :param pk: Xxx ID
        :return:
        """
        model = await my_model_dao.get(db, pk)
        if not model:
            raise errors.NotFoundError(msg='记录不存在')
        count = await my_model_dao.delete(db, pk)
        return count


my_model_service: MyModelService = MyModelService()
```

### API

```python
# backend/plugin/my_plugin/api/v1/my_feature.py
from typing import Annotated

from fastapi import APIRouter, Path, Query

from backend.common.pagination import DependsPagination, PageData
from backend.common.response.response_schema import ResponseModel, ResponseSchemaModel, response_base
from backend.common.security.jwt import DependsJwtAuth
from backend.database.db import CurrentSession, CurrentSessionTransaction
from backend.plugin.my_plugin.schema.my_schema import CreateMyModelParam, GetMyModelDetail, UpdateMyModelParam
from backend.plugin.my_plugin.service.my_service import my_model_service

router = APIRouter()


@router.get(
    '',
    summary='分页获取列表',
    dependencies=[
        DependsJwtAuth,
        DependsPagination,
    ],
)
async def get_my_models_paginated(
    db: CurrentSession,
    name: Annotated[str | None, Query(description='名称')] = None,
    status: Annotated[int | None, Query(description='状态')] = None,
) -> ResponseSchemaModel[PageData[GetMyModelDetail]]:
    page_data = await my_model_service.get_list(db=db, name=name, status=status)
    return response_base.success(data=page_data)


@router.get('/{pk}', summary='获取详情', dependencies=[DependsJwtAuth])
async def get_my_model(
    db: CurrentSession,
    pk: Annotated[int, Path(description='ID')],
) -> ResponseSchemaModel[GetMyModelDetail]:
    model = await my_model_service.get(db=db, pk=pk)
    return response_base.success(data=model)


@router.post('', summary='创建', dependencies=[DependsJwtAuth])
async def create_my_model(
    db: CurrentSessionTransaction,
    obj: CreateMyModelParam,
) -> ResponseModel:
    await my_model_service.create(db=db, obj=obj)
    return response_base.success()


@router.put('/{pk}', summary='更新', dependencies=[DependsJwtAuth])
async def update_my_model(
    db: CurrentSessionTransaction,
    pk: Annotated[int, Path(description='ID')],
    obj: UpdateMyModelParam,
) -> ResponseModel:
    count = await my_model_service.update(db=db, pk=pk, obj=obj)
    if count > 0:
        return response_base.success()
    return response_base.fail()


@router.delete('/{pk}', summary='删除', dependencies=[DependsJwtAuth])
async def delete_my_model(
    db: CurrentSessionTransaction,
    pk: Annotated[int, Path(description='ID')],
) -> ResponseModel:
    count = await my_model_service.delete(db=db, pk=pk)
    if count > 0:
        return response_base.success()
    return response_base.fail()
```

---

## 注意事项

> ⚠️ **警告**: 非必要情况下，插件代码中尽量不要引用架构中的现有方法。如果架构中的现有方法发生变更，则插件也必须同步变更，否则插件将被损坏。
