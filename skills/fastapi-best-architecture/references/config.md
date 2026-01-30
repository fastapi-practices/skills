# 配置参考

## 目录

- [环境配置](#环境配置)
- [FastAPI 配置](#fastapi-配置)
- [数据库配置](#数据库配置)
- [Redis 配置](#redis-配置)
- [Token 配置](#token-配置)
- [用户安全配置](#用户安全配置)
- [RBAC 配置](#rbac-配置)
- [日志配置](#日志配置)
- [插件配置](#插件配置)

**相关文档**：[数据库模型](model.md) | [插件开发](plugin.md) | [API 规范](api.md)

配置文件位置：`backend/core/conf.py`

带 `env` 标记的配置默认为环境变量配置。

---

## 环境配置

| 配置项           | 类型                       | 说明                       |
|---------------|--------------------------|--------------------------|
| `ENVIRONMENT` | `Literal['dev', 'prod']` | 环境模式，prod 时禁用 openapi 文档 |

---

## FastAPI 配置

| 配置项                    | 类型     | 说明              |
|------------------------|--------|-----------------|
| `FASTAPI_API_V1_PATH`  | `str`  | API 版本号配置       |
| `FASTAPI_TITLE`        | `str`  | openapi 文档标头    |
| `FASTAPI_DESCRIPTION`  | `str`  | openapi 文档描述    |
| `FASTAPI_DOCS_URL`     | `str`  | docs 文档地址       |
| `FASTAPI_REDOC_URL`    | `str`  | redoc 文档地址      |
| `FASTAPI_OPENAPI_URL`  | `str`  | openapi JSON 地址 |
| `FASTAPI_STATIC_FILES` | `bool` | 是否开启静态文件服务      |

---

## 数据库配置

| 配置项                 | 类型                               | 说明               |
|---------------------|----------------------------------|------------------|
| `DATABASE_TYPE`     | `Literal['postgresql', 'mysql']` | 数据库类型 (env)      |
| `DATABASE_HOST`     | `str`                            | 主机地址 (env)       |
| `DATABASE_PORT`     | `int`                            | 端口号 (env)        |
| `DATABASE_USER`     | `str`                            | 用户名 (env)        |
| `DATABASE_PASSWORD` | `str`                            | 密码 (env)         |
| `DATABASE_SCHEMA`   | `str`                            | 数据库名             |
| `DATABASE_CHARSET`  | `str`                            | 字符集（仅 MySQL）     |
| `DATABASE_PK_MODE`  | `str`                            | 主键模式 ⚠️ 不要随意更改   |
| `DATABASE_ECHO`     | `bool \| Literal['debug']`       | 输出 sqlalchemy 日志 |

---

## Redis 配置

| 配置项              | 类型    | 说明             |
|------------------|-------|----------------|
| `REDIS_HOST`     | `str` | 主机地址           |
| `REDIS_PORT`     | `int` | 端口号            |
| `REDIS_PASSWORD` | `str` | 密码             |
| `REDIS_DATABASE` | `int` | 逻辑数据库索引 (0-15) |
| `REDIS_TIMEOUT`  | `int` | 超时时间 (env)     |

---

## Token 配置

| 配置项                            | 类型          | 说明                     |
|--------------------------------|-------------|------------------------|
| `TOKEN_SECRET_KEY`             | `str`       | token 密钥 ⚠️ 妥善保管 (env) |
| `TOKEN_ALGORITHM`              | `str`       | 加密算法                   |
| `TOKEN_EXPIRE_SECONDS`         | `int`       | 过期时长（秒）                |
| `TOKEN_REFRESH_EXPIRE_SECONDS` | `int`       | 刷新 token 过期时长          |
| `TOKEN_REDIS_PREFIX`           | `str`       | Redis 前缀               |
| `TOKEN_REQUEST_PATH_EXCLUDE`   | `list[str]` | JWT/RBAC 路由白名单         |

---

## 用户安全配置

| 配置项                                  | 类型     | 说明             |
|--------------------------------------|--------|----------------|
| `USER_LOCK_THRESHOLD`                | `int`  | 密码错误锁定阈值（0 禁用） |
| `USER_LOCK_SECONDS`                  | `int`  | 锁定时长（秒）        |
| `USER_PASSWORD_EXPIRY_DAYS`          | `int`  | 密码有效期（0 永不过期）  |
| `USER_PASSWORD_MIN_LENGTH`           | `int`  | 密码最小长度         |
| `USER_PASSWORD_MAX_LENGTH`           | `int`  | 密码最大长度         |
| `USER_PASSWORD_REQUIRE_SPECIAL_CHAR` | `bool` | 是否需要特殊字符       |

---

## RBAC 配置

| 配置项                      | 类型          | 说明            |
|--------------------------|-------------|---------------|
| `RBAC_ROLE_MENU_MODE`    | `bool`      | 是否开启角色菜单模式    |
| `RBAC_ROLE_MENU_EXCLUDE` | `list[str]` | 跳过 RBAC 鉴权的标识 |

---

## 日志配置

| 配置项                           | 类型    | 说明          |
|-------------------------------|-------|-------------|
| `LOG_FORMAT`                  | `str` | 日志格式        |
| `LOG_STD_LEVEL`               | `str` | 控制台日志级别     |
| `LOG_FILE_ACCESS_LEVEL`       | `str` | 访问日志级别      |
| `LOG_FILE_ERROR_LEVEL`        | `str` | 错误日志级别      |
| `TRACE_ID_REQUEST_HEADER_KEY` | `str` | 跟踪 ID 请求头键名 |

---

## 插件配置

| 配置项                    | 类型     | 说明            |
|------------------------|--------|---------------|
| `PLUGIN_PIP_CHINA`     | `bool` | pip 使用国内源     |
| `PLUGIN_PIP_INDEX_URL` | `str`  | pip 索引地址      |
| `PLUGIN_PIP_MAX_RETRY` | `int`  | pip 下载最大重试次数  |
| `PLUGIN_REDIS_PREFIX`  | `str`  | 插件信息 Redis 前缀 |

---

## 其他常用配置

| 配置项                     | 类型          | 说明                    |
|-------------------------|-------------|-----------------------|
| `DATETIME_TIMEZONE`     | `str`       | 全局时区                  |
| `DATETIME_FORMAT`       | `str`       | 时间字符串格式               |
| `MIDDLEWARE_CORS`       | `bool`      | 是否启用跨域中间件             |
| `CORS_ALLOWED_ORIGINS`  | `list[str]` | 允许的跨域来源               |
| `DEMO_MODE`             | `bool`      | 演示模式（仅允许 GET/OPTIONS） |
| `I18N_DEFAULT_LANGUAGE` | `str`       | 国际化默认语言               |
