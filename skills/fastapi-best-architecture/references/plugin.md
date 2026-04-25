# Plugin Development Standards

## Plugin Types

### App-level Plugin

An app-level plugin is injected into the system like a normal application. In fba, first-level folders under `app` are treated as applications, and the same rule applies to app-level plugins.

App-level plugins must follow the normal route structure completely.

```toml
[app]
router = ['v1']
```

### Extend-level Plugin

An extend-level plugin is injected into an existing application under the `app` directory.

Extend-level plugins must copy the target application's `api` directory structure 1:1.

```toml
[app]
extend = 'admin'
```

## Plugin Route Injection

If a plugin satisfies the plugin development requirements, all routes in the plugin are automatically injected into the FastAPI application.

Startup time can increase as the number of plugins grows because fba parses all plugins in real time before each startup.

### App-level Routes

Develop routes according to the standard fba route structure.

### Extend-level Routes

Replicate the existing application's `api` directory structure 1:1. For example, the built-in `notice` plugin extends an existing application by mirroring its API layout.

## Database Compatibility

Official fba implementations support both MySQL and PostgreSQL.

Third-party plugins are not required to support both databases, but plugin authors should declare supported databases in `plugin.toml`.

For cross-database SQLAlchemy compatibility, use SQLAlchemy 2.0 mechanisms such as `TypeDecorator` and `with_variant`.

## Backend Plugin Directory Structure

Plugins are placed under `backend/plugin`.

```text
xxx                             # Plugin name
├── api                         # API routes
├── crud                        # CRUD
├── model                       # Models
│   ├── __init__.py             # Import all model classes here
│   └── ...
├── schema                      # Data transfer schemas
├── service                     # Services
├── sql                         # Recommended when the plugin executes SQL
│   ├── mysql
│   │   ├── destroy.sql         # Auto-increment ID cleanup, executed on uninstall
│   │   ├── destroy_snowflake.sql # Snowflake ID cleanup
│   │   ├── init.sql            # Auto-increment ID initialization, executed on install
│   │   └── init_snowflake.sql  # Snowflake ID initialization
│   └── postgresql
│       └── ...                 # Same file names as mysql
├── utils                       # Utilities
├── .env.example                # Environment variables
├── __init__.py                 # Kept as a Python package
├── ...                         # More content, e.g. enums.py
├── hooks.py                    # Optional plugin hook functions
├── plugin.toml                 # Plugin configuration file
├── README.md                   # Usage instructions and contact information
└── requirements.txt            # Dependency packages
```

## plugin.toml Configuration

Every plugin must contain `plugin.toml`.

### Common Plugin Metadata

```toml
[plugin]
# Icon path inside the plugin repository or an icon URL
icon = 'assets/icon.svg'
# Short summary
summary = ''
# Version
version = ''
# Description
description = ''
# Author
author = ''
# Supported tags: ai, mcp, agent, auth, storage, notification, task, payment, other
tags = ['']
# Supported databases: mysql, postgresql
database = ['']
```

### App-level Plugin Configuration

```toml
# Plugin metadata
[plugin]
icon = 'assets/icon.svg'
summary = ''
version = ''
description = ''
author = ''
tags = ['']
database = ['']

# Application configuration
[app]
# Final router instance names.
# See backend/app/admin/api/router.py; usually named v1.
router = ['v1']

# Code-level configuration keys in uppercase.
# Optional. See Hot-pluggable Configuration.
[settings]
XXX = 'value'
```

### Extend-level Plugin Configuration

```toml
# Plugin metadata
[plugin]
icon = 'assets/icon.svg'
summary = ''
version = ''
description = ''
author = ''
tags = ['']
database = ['']

# Application configuration
[app]
# Target application folder name
extend = 'application_folder_name'

# API configuration
[api.xxx]
# xxx is the file name under the plugin api directory without extension.
# Example: for notice.py, use [api.notice].
# Multiple API files require multiple [api.xxx] sections.
# Route prefix, must start with '/'.
prefix = ''
# Tags for Swagger documentation
tags = ''

# Code-level configuration keys in uppercase.
# Optional. See Hot-pluggable Configuration.
[settings]
XXX = 'value'
```

## Global Configuration

fba uses one global configuration file, similar to Django.

During development, add plugin global configuration to `backend/core/conf.py` for typing hints and explicit configuration management.

```python
##################################################
# [ Plugin ] email
##################################################
# .env
EMAIL_USERNAME: str
EMAIL_PASSWORD: str

# Basic configuration
EMAIL_HOST: str
EMAIL_PORT: int
EMAIL_SSL: bool
EMAIL_CAPTCHA_REDIS_PREFIX: str
EMAIL_CAPTCHA_EXPIRE_SECONDS: int
```

The structure should contain:

1. Plugin configuration comment block.
2. Plugin environment variable declarations and comments.
3. Plugin basic configuration declarations and comments.

Published plugins cannot modify the user's `backend/core/conf.py` directly. Document required global configuration in the plugin `README.md`.

## Hot-pluggable Configuration

Since fba v1.13.0, plugins can adapt to hot-pluggable installation when configured correctly.

### Plugin Environment Variables

If the plugin requires environment variables, add `.env.example` in the plugin root directory.

```dotenv
# [ Plugin ] email
EMAIL_USERNAME: str
EMAIL_PASSWORD: str
```

### Plugin Basic Configuration

If the plugin requires basic configuration, add uppercase configuration keys under `[settings]` in `plugin.toml`.

Do not confuse `plugin.toml` settings with `backend/core/conf.py` declarations. Their formats are different.

```toml
[settings]
EMAIL_HOST = 'smtp.qq.com'
EMAIL_PORT = 465
EMAIL_SSL = true
EMAIL_CAPTCHA_REDIS_PREFIX = 'fba:email:captcha'
EMAIL_CAPTCHA_EXPIRE_SECONDS = 180
```

After `.env.example` and `[settings]` are configured, plugins installed through CLI or Git can adapt to hot-pluggable behavior without extra manual changes, provided the plugin has no additional integration requirements.

### Global Configuration Priority

Configuration priority flows in this order:

```text
System environment variables -> .env -> conf.py -> plugin [settings]
```

Development recommendation:

- Add global configuration declarations in `backend/core/conf.py` during development.
- Document those declarations in the published plugin `README.md`.
- Use this approach when IDE typing hints are important for plugin developers or users.

## Hook Functions

Since fba v1.13.3, plugins support hook functions for more flexible configuration and reduced manual adaptation.

Hook functions must be defined in `hooks.py` at the plugin root.

fba also provides helper functions in `backend/plugin/patching.py` for plugin configuration.

### lifespan

Defines a FastAPI lifespan function. It is automatically registered before application startup.

### setup

Defines startup logic. Both synchronous and asynchronous setup functions are supported. The function is automatically executed before application startup.

## Frontend Plugin Directory Structure

Frontend plugins are placed under `apps/web-antd/src/plugins`.

```text
xxx                             # Plugin name
├── api                         # API client code
│   └── index.ts
├── langs                       # I18n resources
│   ├── en-US
│   │   └── plugin_name.json
│   └── zh-CN
│       └── plugin_name.json
├── public
│   └── images                  # Page preview images
├── routes                      # Routes
│   └── index.ts
├── views                       # Views
│   ├── index.vue
│   └── ...
├── ...                         # More content
└── plugin.toml                 # Plugin configuration file
```

## Frontend plugin.toml Configuration

Every frontend plugin must contain `plugin.toml`.

```toml
[plugin]
# Icon path inside the plugin repository or an icon URL
icon = 'assets/icon.svg'
# Short summary
summary = ''
# Version
version = ''
# Description
description = ''
# Author
author = ''
# Supported tags: ai, mcp, agent, auth, storage, notification, task, payment, other
tags = ['']
```

## Plugin README Convention

When creating, reviewing, or updating a plugin `README.md`, follow these rules strictly.

### Required Structure

A plugin `README.md` must contain only the following content, in this order:

1. Title
2. Description
3. Plugin Type
4. Configuration
5. Usage
6. Uninstall
7. Contact

### Section Rules

#### Title

Use the plugin display name as the H1 title.

Example:

```md
# OAuth2
```

#### Description

Place a short description immediately below the title.

Keep it concise and use this part to explain the plugin capabilities.

Capability summaries may be written as short paragraphs or short bullet lists directly under the title.

Do not create a separate feature section for plugin capabilities.

#### Plugin Type

Only describe the plugin type, such as app-level or extend-level.

For extend-level plugins, include the target app name such as `admin` when useful.

Do not include route prefixes, API mount paths, or endpoint information.

#### Configuration

Directly explain how to configure the plugin.

Describe relevant `.env`, `plugin.toml`, and `backend/core/conf.py` content clearly.

Always present configuration in this order:

1. What to add in `.env`
2. What is contained in `[settings]` of `plugin.toml`
3. What to add in `backend/core/conf.py`

Only include configuration sources that actually have meaningful content.

Do not add placeholder lines such as `No extra content required`, `No additional configuration required`, or equivalent no-op statements for omitted sources.

When the plugin has corresponding fields in `backend/core/conf.py`, include the exact field definitions or explain that they are already present in the current project.

When showing `backend/core/conf.py` content, keep the actual comment lines and grouping style consistent with the real file, including lines such as `##################################################`, `# .env`, and `# 基础配置（in plugin.toml）` when they exist.

When the plugin does not need extra `backend/core/conf.py` fields, state that explicitly.

Use direct instruction wording.

Avoid conditional phrasing such as `if needed`, `when enabled`, or equivalent optional wording.

Do not add per-item configuration explanations in this section unless the user explicitly asks for them.

For `plugin.toml`, prefer wording such as `plugin.toml` 的 `[settings]` 中包含以下内容`rather than`添加以下内容`.

#### Usage

Describe only the core usage flow in plain language.

Keep this section short and focused.

Do not list API endpoints, route prefixes, request paths, or interface details.

#### Uninstall

Describe which related configuration should be removed and what integrations should be cleaned up.

Use high-level cleanup wording by default.

Do not enumerate specific configuration keys in the uninstall section unless the user explicitly asks for them.

#### Contact

Provide author/contact entry in a concise way.

### Forbidden Content

Do not include the following in plugin `README.md` files:

- Route prefixes
- API endpoint lists
- Interface descriptions
- Feature sections
- Warning sections
- Note sections
- FAQ sections
- Extra headings outside the required structure

### Punctuation Rule

Do not end prose lines or list items with the Chinese full stop `。`.

This rule does not apply to code blocks.

### Style Rule

Keep wording concise, direct, and operational.

Prefer short paragraphs and short numbered lists.

## Important Notes

Unless necessary, avoid referencing existing architecture methods from plugin code.

If existing architecture methods change, plugins that depend on those methods must be updated, otherwise they can break.
