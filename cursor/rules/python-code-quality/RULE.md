---
description: "This rule provides guidlines for using linter directives."
alwaysApply: true
---

The project uses `nox` to manage linting, formatting, type-checking, and other code quality tools.
- The `nox` tasks will return non-zero exit code if issues are found.
- Code modifications should be made to resolve any issues reported by these tools.
- If iterating changes in a red/green/refactor cycle, it is acceptable for a test, or some tests, to
  fail as long as they are expected to fail given the current stage in the cycle and the failures
  will be resolved in a subsequent step.


| **Command**   | **Purpose** |
|---------------|-------------|
| `nox -l` | Lists all available nox sessions. |
| `nox -s style lint` | Runs the style enforcement and linting tools together. |
| `nox -s autofix` | Attempts to fix issues identified by style and linting tools. |
| `nox -s mypy` | Runs the static type-checker over all supported Python versions. |
| `nox -s tests` | Runs the test suite over all supported Python versions. |

## Type Hints
-  Use `list`, `dict`, `tuple`, `set`, etc. built-in generic types for type hinting instead of
   importing from `typing` (e.g., use `list[int]` instead of `typing.List[int]`).
- Use `collections.abc` generics for abstract base classes (e.g., use `collections.abc.Iterable`
   instead of `typing.Iterable`).
- NEVER use `Any` as a type hint unless absolutely necessary. Instead, use more specific
  types or `object` if the type is truly unknown.
- Use `TypeVar` with appropriate constraints instead of `Any` when defining generic functions or
  classes.


