# Development Architecture Reference

Use this guide before implementing or reviewing cross-layer fba features.

## Layer Responsibilities

- API: declare routes, dependencies, request parameters, response models, and return `response_base` results. Do not put business rules here.
- Schema: define Pydantic request and response DTOs with `SchemaBase` and clear `Field(description=...)` metadata.
- Service: hold business logic, validation, orchestration, cache invalidation, task dispatch, and exception handling.
- CRUD: encapsulate database reads/writes and return ORM objects, scalar values, or SQLAlchemy `Select` objects for pagination.
- Model: define SQLAlchemy ORM tables with `Base`, explicit `__tablename__`, and explicit primary keys.

Keep API, Service, and CRUD I/O async. Use keyword-only service arguments for clarity.

## Module Boundaries and SDK Integration

- Place shared request/response DTOs in `schema/`. Schema modules must not import Service, CRUD, Agent runtime, or execution-lifecycle implementations to obtain their data contracts.
- Keep business validation and orchestration in Service or the existing domain runtime that owns the workflow. Keep database access in CRUD. Put pure conversion helpers in `utils/` only when they have a clear shared purpose; do not move business workflows there to shorten another file.
- Group modules by cohesive responsibility. A protocol or SDK package is reasonable when it owns an actual integration boundary; do not collect DTOs, business rules, persistence, and runtime lifecycle code under one protocol name merely because they use the same dependency. Avoid one-helper files and extra package levels without a distinct responsibility.
- Prefer the SDK's native contracts, converters, and lifecycle hooks. Retain the adapters needed for application semantics or metadata; avoid duplicating SDK behavior. Directory flattening and adapter line counts are not goals in themselves.
- When restructuring, inspect import direction and callers first, preserve public behavior and required SDK interfaces, then update production imports, test imports, string-based mocks, and documentation together. Remove the old package when no compatibility contract requires it; do not leave forwarding modules solely to avoid updating local callers.

## Implementation Flow

1. Define or update the database model and migration.
2. Define request/response schemas.
3. Define routes with `summary`, dependency declarations, response annotations, and session dependencies.
4. Implement service logic and validation.
5. Implement CRUD queries and persistence helpers.
6. Add or update tests, seed SQL, permissions, cache invalidation, and i18n messages as needed.

## Sessions and Transactions

- Use `CurrentSession` for read-only API handlers.
- Use `CurrentSessionTransaction` for create, update, delete, and batch mutation handlers.
- Use `async with async_db_session.begin() as db:` only when a service or task must open its own transaction boundary.
- Keep transaction boundaries outside low-level CRUD helpers unless the helper is explicitly designed to own the transaction.

## Responses and Pagination

- Mutations that do not return data should use `ResponseModel` and `response_base.success()`.
- Detail or typed query responses should use `ResponseSchemaModel[GetXxxDetail]`.
- Normal pagination uses `DependsPagination`, `PageData[XxxDetail]`, and `paging_data(db, select)` in the Service layer.
- Cursor pagination uses `DependsCursorPagination` and `CursorPageData[XxxDetail]` for infinite-scroll or conversation-like lists.
- Use `response_base.fast_success()` only for large JSON payloads where skipping Pydantic validation is intentional.

## Cache

- Prefer declarative cache decorators for standard read/update flows: `@cached` for reads and `@cache_invalidate` for mutations.
- Use programmatic Redis operations for complex keys, batch operations, warm-up, or custom serialization.
- When mutating cached data, keep database write success and cache invalidation behavior explicit in the Service layer.

## Celery and Background Work

- Built-in Celery code lives under `backend/app/task`.
- Start local task components with `fba celery worker`, `fba celery beat`, and `fba celery flower` when task result APIs are needed.
- Docker build targets use `SERVER_TYPE`, for example `SERVER_TYPE=fba_celery_worker`.
- Put task dispatch in Service logic rather than API handlers when it belongs to a business workflow.

## I18n

- Language packs live in `backend/locale` and support JSON, YAML, and YML.
- Use `t('response.success')` style chained keys.
- fba reads `Accept-Language` from the request and falls back to the configured default language.

## Code Generation

- The code generator is backend-only.
- Business generation configuration lives in `backend/plugin/code_generator/model/business.py`.
- Generated code is a scaffold: review naming, schemas, permissions, pagination, transaction usage, and formatting before accepting it.

## Multi-tenancy

- The main repository does not provide full tenant business capabilities by default.
- Tenant features usually come from plugins; follow the plugin README and review model fields, unique constraints, query filters, SQL initialization, permissions, and plugin compatibility.
- Do not assume `TENANT_*` core settings exist in the base project.

## Docker and Local Runtime Notes

- The backend container maps service port `8001` by default in Docker workflows.
- For local Docker runs that access host services, change `.env` loopback hosts such as `127.0.0.1` to `host.docker.internal`.
