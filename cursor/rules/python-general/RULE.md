---
description: "This rule provides general guidance for Python coding practices."
alwaysApply: false
---

- Prefer small, well-named functions and composition over inheritance.
- Mandatory type hints on public functions, classes, and module-level variables.
- Never use the `Any` type hint unless absolutely necessary. 
- Favor clarity over cleverness; document intent with brief docstrings or comments.
- Avoid unnecessary abstraction — prefer the simplest thing that satisfies the use case.
- Maintain simplicity — prefer removing code over adding abstraction.

## Patterns to Avoid

The following items are **NEVER** acceptable.  If you find that you have emitted one of these, refactor before committing.

* NEVER use global mutable state (never use the `global` keyword). ==> Use cached providers (e.g. `@lru_cache(get_config)`) instead.
* NEVER use hardcoded absolute paths, secrets, API keys. ==> Use configuration-driven patterns.
* NEVER leak internal stack traces or error internals to clients.
* NEVER use error suppression comments (e.g., `# noqa: E722`, `# pylint: disable=...`, `# mypy: ignore`, `# flake8: noqa`, `# ruff: noqa`, `# type: ignore`, `# pragma: no cover`, ...).

## Configuration & Secrets

Use `pydantic_settings` for environment-driven configuration. A typical pattern:
```python
from functools import lru_cache
from pydantic import BaseModel, UUID4
from pydantic_settings import BaseSettings, SettingsConfigDict


class APISettings(BaseModel):
    key: str
    tenant: UUID4


class LoggingSettings(BaseModel):
    enabled: bool = False
    level: str = "info"  # e.g., "debug", "info", "warning", "error"


class ApplicationSettings(BaseSettings):
    model_config = SettingsConfigDict(
        env_file_encoding="utf-8",
        env_prefix="AIML_ONPREM_",
        env_nested_delimiter="_",
        env_nested_max_split=1,
    )

    api: APISettings = APISettings()
    log: LoggingSettings = LoggingSettings()


@lru_cache
def get_config() -> ApplicationSettings:
    return ApplicationSettings()
```
Future integration with a secret manager (e.g., cloud vault) will layer on top of `BaseSettings`—do not inline secret fetch logic throughout the codebase.

## Logging & Observability

Use `aiml_core_services.logging` to obtain loggers. Log structured key/value pairs (avoid embedding JSON as strings). Keep payload fields concise—truncate large text fields via provided utilities. Tracing/metrics hooks via `aiml_core_services.opentelemetry` when needed; keep spans short for request scope.

Example:
```python
from aiml_core_services.logging import get_logger
logger = get_logger(__name__)
logger.info("Creating session", persona_id=persona.persona_id)
```

If the application needs to extend or customize logging behavior (e.g., add custom processors or handlers), then do so in a `logging` module.
* Application-wide: `aiml_onprem.logging`
* Context-specific: e.g., `aiml_onprem.capture.logging`

## Exceptions

Define custom exception classes in an `exceptions` module:
* Application-wide: `aiml_onprem.exceptions`
* Context-specific: e.g., `aiml_onprem.capture.exceptions`

Route layer maps domain/application exceptions to HTTP responses (centralized handler module). Keep exception classes lightweight—store only actionable data.

## Type Hints

Provide type hints for all public functions, classes, and module-level variables. Use modern syntax (e.g., `list[int]`, `dict[str, Any]`, `int | None`). Avoid `Any` unless absolutely necessary; prefer `object` if the type is truly unknown.
Define custom types in a `typing` module.
* Application-wide: `aiml_onprem.typing`
* Context-specific: e.g., `aiml_onprem.capture.typing`

## CLI Entry Points

Each CLI utility exposes `entrypoint() -> int` in a `__main__` (or referenced) module. Return 0 for success; return 1 for general failure. For richer semantics (rare) you may adopt `sysexits` codes. Example minimal pattern:
```python
def entrypoint() -> int:
    # ... work ...
    return 0
```
Registered under `[project.scripts]` in `pyproject.toml`.

## Async & DI

All FastAPI route handlers are `async def`. Delegate work to service layer objects injected via `Depends`. Prefer function-scoped resources; for shared singletons (e.g., settings, DB engine) expose via cached factory functions.

## Data Access

Use one SQLAlchemy session per request (FastAPI dependency) drawn from a pooled engine. Repositories encapsulate queries; route/service code calls repository methods rather than embedding ORM logic.

## Dependency Management

* Change dependencies via `pyproject.toml` only; then run `uv sync`.
* Prefer semver-compatible version specifiers (e.g., `^1.2.3`) unless a strict pin is required for reproducibility.
* Remove unused dependencies promptly.
* Avoid adding transitive-only packages directly unless they become first-class.

## Security (Python-Level)

* Never commit secrets or tokens—use environment variables / secret manager.
* Validate all external input with Pydantic models; reject on validation failure (FastAPI will 422 automatically).
* Avoid dynamic `exec`/`eval`; avoid shell injection (use subprocess args lists if needed).
* Treat user-provided text as untrusted—escape where rendered (outside Python scope if purely API JSON).

## Models & Terminology

Two broad categories:
* Contracts: Pydantic models defining API request/response shapes (e.g., placed in a `contracts` module next to the `endpoints` module that uses them). These drive automatic validation and serialization.
* Data / domain models: Internal or persistence-facing representations that express invariants and domain behavior.

Keep transformations explicit — do not silently reuse contract models for persistence.

## Versioning

Use Semantic Versioning. Bump versions with `uv version --bump <part>` (`major|minor|patch`). Major: breaking public contract change. Minor: backward-compatible feature. Patch: bug fix / internal improvements.

## References

* Google Python Style Guide
* FastAPI, Pydantic & pydantic_settings docs
* SQLAlchemy & Alembic docs
* Ruff, Nox, UV documentation
* “Building an Architecture to Support Domain Modeling” (Cosmic Python)
* Domain-Driven Design references
* Structlog & Python logging best practices
* OpenTelemetry Python instrumentation


