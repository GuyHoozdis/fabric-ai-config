---
applyTo: "**/*.py"
---

# GitHub Copilot Python Instructions

Focused guidance for Python coding in this repository: keep it simple, consistent, and aligned with our domain-driven design approach.

## Context

Supported Python versions: 3.11 – 3.14 (code must run and quality checks must pass across this range). Treat 3.14 as forward‑compatibility; avoid relying on provisional features.

Primary paradigms (orthogonal):
1. Domain-Driven Design (DDD)
2. Event-Driven concepts where useful
3. Microservices (service boundaries handled outside this file)

Frameworks & libs: FastAPI, Pydantic, pydantic_settings, SQLAlchemy, Alembic, Requests / httpx, asyncio.
Package / build: uv.
Orchestration: nox.

## General Guidelines

* PEP 8 & Google-style docstrings (tests may omit docstrings).
* Prefer small, well-named functions and composition over inheritance.
* Mandatory type hints on public functions, classes, and module-level variables.
* Follow ruff config; fix lint after generation.  Use `autofix` first, then manual fixes as needed.
* No wildcard imports; explicit imports only.
* Keep logic domain-focused.
* Avoid unnecessary abstraction — prefer the simplest thing that satisfies the use case.
* Favor clarity over cleverness; document intent with brief docstrings or comments.
* Use `__all__` to define public API surface for modules.

## Unit Testing Strategy

* Unit tests MUST be written using the `unittest` package from the Python stdlib.
* Place new tests under `tests/aiml_onprem/...` mirroring module path.
* Test cases SHOULD be named `<SUT>TestCase`.
* A test module MUST map 1:1 with the module that contains the logic it tests.
* A test module CAN contain more than one test case.
* Test cases CAN map 1:1 with a class or function from the SUT, but MAY also represent a feature, behavior, or logical grouping of .
* Test methods within each case SHOULD use one of the following styles:
  - `test_it_<expected_behavior>[_when_condition]` (when the _TestCase_ name provides the needed context to clearly understand what "it" refers to)
  - `test_<system_under_test>_<expected-behavior>_<condition>`
  - `test_<system_under_test>_<condition>_<expected-result>`
* For Playwright-dependent code, isolate browser logic so tests can mock or skip when Playwright not installed (note the defensive try/except importing types in capture module).

Minimum per SUT (unit-style tests): one happy path test and (if meaningful) one edge case test representing the most likely failure/variant. Coverage targets will be defined separately—do not assume a percentage here.

Keep unit tests fast, deterministic, no network or filesystem side effects (use in-memory fakes or temp dirs). Silence stdout/stderr unless explicitly needed.

## Patterns to Follow

* Repository Pattern to abstract data access (see “Cosmic Python” reference).
* Dependency Injection via FastAPI `Depends` (function-scoped DI).
* Configuration via `pydantic_settings.BaseSettings` with an `@lru_cache` accessor for a single shared instance.
* Async FastAPI endpoints (define route handlers as `async def`).
* Pydantic models for request/response contracts; keep domain/internal models separate when needed.
* Custom exceptions organized per context.
* Structured logging through `aiml_core_services.logging`.

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
def entrypoint() -> int:  # noqa: D401 (simple)
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

## Review & Iteration

### Checklist:
Before Merging:
1. Run lint/style/mypy/tests (nox sessions) across supported Python versions.
2. Ensure new public interfaces have at least happy + edge tests (unit-style).
3. Confirm logging messages are structured and concise.
4. Keep changes minimal; refactor if complexity grows.

Maintain simplicity — prefer removing code over adding abstraction.
If in doubt, choose the clearer implementation and document intent with a brief docstring.
