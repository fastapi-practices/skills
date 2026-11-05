# Plugin Lifecycle Workflow

Use this runbook for end-to-end plugin work: converting an existing feature, creating a new plugin, preparing a release, submitting it to the marketplace, or publishing an update.

This file coordinates the workflow. Load `plugin.md` for development rules and `plugin-sharing.md` for repository and marketplace rules. Do not duplicate those specifications here.

## Operating Principles

- Prefer behavior from the `fastapi-best-architecture` source when documentation and implementation differ.
- Keep the requested scope. A plugin conversion does not authorize unrelated repository-wide cleanup.
- Do not silently modify main-project core files. If a plugin cannot work through supported extension points, report the blocker.
- Inspect before editing. Produce a concrete mapping from source files to plugin files.
- Use stage gates. Development and publication are separate decisions.
- Do not run `git add`, `git commit`, `git push`, `gh repo create`, fork operations, submodule updates, or PR creation unless the user explicitly authorizes repository writes.
- Never execute untrusted plugin code during marketplace validation. Parse manifests and inspect files only.

## Lifecycle Stages

| Stage | Goal | Write access |
|-------|------|--------------|
| Intake | Classify the plugin and collect missing context | No |
| Audit | Map the source feature, coupling, and blockers | No |
| Build | Create or convert plugin code and documentation | Yes, project files only |
| Validate | Run manifest, structure, style, tests, and runtime checks | No repository writes |
| Package | Confirm repository root layout and release metadata | No repository writes |
| Publish | Create a repository and open a marketplace PR | Explicit authorization required |
| Update | Publish a new plugin version and refresh the submodule pointer | Explicit authorization required |

## Intake

Collect or infer the following before changing files:

| Field | Required |
|-------|----------|
| fba project root | Yes |
| Source feature path or new plugin scope | Yes |
| Plugin name | Yes |
| Plugin type: app-level, extend-level, or capability-level | Yes |
| Target application for extend-level plugins | Yes when applicable |
| Target databases and primary-key strategy | Yes when the plugin has models or SQL |
| Frontend companion plugin | No |
| Required hooks, settings, environment variables, and permissions | No |
| Target GitHub owner and repository | Yes for publishing |
| Release mode: audit only, build only, prepare only, or publish | Yes |

Use `plugin.md` to classify the plugin and determine the minimum required directory structure and manifest sections.

Immediately stop the intake when:

- The name conflicts with an existing marketplace plugin.
- The requested name cannot be a valid Python package name.
- The feature requires private application internals that are not part of a supported extension point.
- The user expects a plugin to modify main-project core files automatically.
- Database compatibility cannot be expressed with supported SQLAlchemy types or complete SQL scripts.

## Audit

Perform a read-only inventory of the source feature and classify every relevant item:

| Decision | Meaning |
|----------|---------|
| `MOVE` | Move into the plugin without behavioral change |
| `ADAPT` | Reuse the logic with plugin-specific imports, configuration, or paths |
| `KEEP` | Leave in the main project because it is shared infrastructure or an explicit extension point |
| `REMOVE` | Do not carry the item into the plugin because it is unrelated or dead code |

The audit must identify:

- API routes, schemas, services, CRUD operations, models, enums, listeners, and hooks.
- Database tables, migrations, seed data, and initialization/destroy scripts.
- Configuration keys, environment variables, permissions, menus, and i18n resources.
- Dependencies between the plugin and other plugins.
- Imports that depend on the main application's private modules.
- Frontend routes, API clients, views, locales, and assets when a companion plugin exists.
- Tests and verification commands that should move with the plugin.

Return a mapping table, blockers, assumptions, and an implementation plan before editing. If the task entered `AUDIT` mode, stop after this report.

## Build

Before editing, read:

- `plugin.md` for plugin structure and manifest rules.
- `naming.md` for package, route, schema, entity, and table names.
- `development.md` for cross-layer feature placement.
- `coding-style.md` and the relevant API, schema, model, and config references for Python changes.

Build rules:

- Place the plugin under `backend/plugin/<plugin_name>`.
- Copy plugin directory contents to the repository root when packaging. Do not nest the plugin directory again.
- Keep all plugin imports relative to stable fba extension points.
- Add a complete `plugin.toml` and a canonical `README.md` from the beginning.
- Model and SQL changes require complete initialization and destroy scripts for every declared database.
- Move plugin configuration into `[settings]` and `.env.example` where hot-pluggable configuration supports it.
- Declare `depends_on` only for real plugin folder names and keep the dependency graph acyclic.
- Preserve route prefixes, permissions, and response contracts unless the conversion explicitly changes them.
- Add or update tests and run them before packaging.

During conversion, do not leave duplicate runtime implementations enabled in both the main application and the plugin. Mark the old source for removal only when the user authorizes that change.

## Validate

Validation is mandatory even when the user only asks for development, unless they explicitly request a draft.

### Manifest and structure

- `plugin.toml`, `README.md`, and `__init__.py` exist at the backend plugin root.
- Frontend plugins have `plugin.toml` at the repository root.
- The manifest passes `backend/plugin/validator.py` behavior and marketplace validation.
- Route, application, API, settings, database, and dependency sections match the actual plugin.
- The plugin name matches the repository and marketplace submodule path rules.
- SQL directories align with `plugin.database` when models exist.
- README documents configuration, usage, uninstall behavior, and feedback.

### Code and runtime

- Run the narrowest relevant tests first, then the plugin or application test suite.
- Run Ruff formatting and lint checks on maintained plugin files.
- Start the application or run the relevant integration check to verify route injection, hooks, settings, and startup order.
- Exercise install, enable, disable, and uninstall behavior when the task changes lifecycle integration.
- Confirm no main-project core files are modified without explicit approval.

### Marketplace parity

When a local marketplace checkout is available, run:

```bash
pnpm install --frozen-lockfile
pnpm validate <plugin_name>
```

Do not publish when manifest validation, structure checks, tests, or replacement checks fail.

## Package

Before creating or updating a public repository:

- Confirm the repository is public.
- Confirm the backend repository name equals the plugin folder name.
- Confirm frontend repositories use the `_ui` or `-ui` suffix.
- Confirm the repository root contains plugin contents, not an outer plugin folder.
- Confirm every submodule URL uses HTTPS, never SSH.
- Confirm the version is `x.y.z` and has been increased for a release.
- Confirm the README and manifest describe the same configuration and behavior.

## Publish

Publishing requires a separate explicit authorization from the user. State the target repository, branch, plugin name, and marketplace path before writing.

The preferred flow is:

1. Create or update the public plugin repository from the appropriate template.
2. Push the plugin contents to the repository root.
3. Run the `plugin-sharing.md` self-check against the marketplace validation commands.
4. Report the result and wait for explicit authorization.
5. From the contributor's authenticated GitHub account, fork the marketplace, push a branch, and open the submodule PR.
6. Review the PR validation report and fix any failures.

When `gh` is unavailable or unauthenticated, use the browser fallback in `plugin-sharing.md` instead of installing tools.

Publication safety:

- Never create the marketplace PR with a maintainer or bot identity. The contributor's account must own the PR.
- Never force-push a contributor branch without explicit authorization.
- Never publish a private repository.
- Never reuse an existing plugin name.
- Never bypass failed validation by disabling workflows or branch protection.
- Never execute install scripts, build scripts, or test code from the submitted repository during marketplace validation.

## Update

For an update:

1. Validate the new plugin version in its own repository.
2. Increase the manifest version according to the impact of the change.
3. Publish the plugin repository changes.
4. Refresh the marketplace submodule pointer.
5. Open a PR describing behavior changes, breaking changes, migration steps, and validation evidence.
6. Do not update the submodule pointer to an uncommitted or unpushed plugin revision.

## Output Contract

At the end of a lifecycle task, report:

- Current lifecycle stage and completion status.
- Files created, modified, or intentionally left unchanged.
- Plugin type, name, dependencies, database requirements, settings, and hooks.
- Verification commands and their results.
- Repository and marketplace state when publishing.
- Remaining blockers, manual approvals, or follow-up work.

If repository writes were not authorized, provide the exact commands that remain instead of running them.
