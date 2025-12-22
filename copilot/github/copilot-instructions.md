# Purpose & Architecture
This repo packages an "OnPrem Digital Human" runtime: a set of backend services (conversation service, LLM service, Redis photo store, Chroma vector DB, optional static frontend) plus local utilities (session capture, data loading). It is Python-first (>=3.11) using FastAPI-based services that are pulled as container images; this repo itself provides orchestration, utilities, and glue code.

Key directories:
* `src/aiml_onprem/`    : Python package (entrypoints, config, utilities like capture tool).
  - `server/`           : OnPrem API
  - `capture/`          : Capture utility
* `scripts/`            : Operational helper scripts (data load, backup, installer assets).
* `compose.yml`         : Defines runtime topology & service dependencies.
* `data/`               : Persisted assets:
  - `indexes/`          : Mounted drive for vector-db service in `compose.yml`.
  - `photos/`           : Mounted drive for photo-service-db in `compose.yml`.
  - `sources/`          : Original doc sources, chunk JSONs, raw data, etc.
* `tests/`              : Unit tests (unittest framework) mirroring `aiml_onprem` structure.
* `images/`             : Locally saved container tarballs for air‑gapped deployment.

Service interaction flow (runtime):
User / frontend → conversation-service → (llm-service + vector-db + photo-service-db) → responses back to user.

## Tooling & Workflows
Dependency/build system uses `uv` (see `[tool.uv]` in `pyproject.toml`). Nox orchestrates lint, style, type check, tests.

Common tasks:
* Lint: `nox -s lint`
* Style check: `nox -s style`
* Auto-fix: `nox -s autofix`
* Types: `nox -s mypy`
* Tests: `nox -s tests` (coverage run logged; separate coverage report task)
* Coverage reports: `nox -s coverage`
* Coverage reports: `nox -s coverage` (reports written under `coverage/`)

CLI entrypoints (installed via scripts section):
* `aiml-onprem` → `aiml_onprem.__main__:entrypoint`
* `aiml-capture` → `aiml_onprem.utils.capture.main:entrypoint`

When adding new utilities, mirror this pattern: create a module with an `entrypoint()` returning int and register in `[project.scripts]`.

## Coding Conventions
Refer to the `.github/instructions/software-engineering.instructions.md` and `.github/instructions/python.instructions.md` files for general software engineering and Python-specific guidelines.

* Conventions enforced by Ruff.  Use the `nox -s autofix` task to automatically fix issues where possible.
* TDD is strongly encouraged.  Write tests that are fast, reliable, and isolated.
* Test behavior, not implementation details.

## Docker & Offline Deployment
* Core services (`conversation-service`, `llm-service`, `vector-db`, `photo-service-db`) run via `docker compose` with tag/version provided by env (`CS_TAG`, `LLMS_TAG`, etc.). Update compose changes cautiously—respect existing env var indirection.
* Air‑gapped flow: pull/build images → `docker save` to `images/` → copy to target → `docker load` there. Do not hardcode image digests; rely on `${VAR}` placeholders.
* Proprietary packages (e.g., `aiml-core-services`) must be downloaded locally and made available during the build/install process so that `--find-links` can reference them without requireing credentials to the private index at install/build time.

## Configuration & Secrets
* Runtime configuration via `.env` files (`cs.env`, `llms.env`, etc.) referenced in `compose.yml` with `env_file`. Do not commit secrets; sample patterns in `env/`.
* Scripts should accept overriding paths via CLI flags instead of embedding absolute paths.

## Adding New Containerized Services, API Endpoints, or Utilities
* For a new Python utility:
  - Create module under `src/aiml_onprem/<area>/`
  - Add structured logging, expose `entrypoint()`
  - Register in `pyproject.toml` scripts
* For a new API endpoint:
  - Follow existing FastAPI routing patterns
  - Add to `src/aiml_onprem/server/routes/<version>/<area>/endpoints.py`
  - Define document any new request/response schemas in a `contracts` module alongside the endpoint
  - Use dependency injection to give endpoints access to services and repositories
  - Register new endpoints via `app.include_router` in the application factory method (see `src/aiml_onprem/server/api.py`)
* For an additional containerized service:
  - Add to `compose.yml` with network placement (use `api` network if externally exposed; `backend` otherwise)
  - Mount a named volume or local path under `./data/` if persistence required
  - Gate ports with env vars following existing `${NAME_HOST_PORT}:${NAME_CONTAINER_PORT}` pattern
  - Follow existing service patterns (config via env, structured logging, health check endpoint)
  - Manage any new configuration options in `src/aiml_onprem/server/config.py`
  - Document any new env vars in `env/` samples

## Documentation
* Maintain the `README.md` for the project.  Update instructions and examples as the codebase evolves.
* Leverage the contents defined in `src/aiml_onprem/server/config.py` and `src/aiml_onprem/capture/config.py` to generate `env/` sample files.
