---
description: "This rule provides guidlines for using directives in code comments."
alwaysApply: true
---

- Avoid using directives unless absolutely necessary.  It is almost NEVER necessary.
- When using directives, document the reason for their using the following format:
  ```python
  # <error_code>: <definition/meaning>
  # <Explanation of why the directive is necessary>
  # <directive>: <error_code>
  def some_function(...):  # <directive>: <error_code>
      ...
  ```
- Be explicit about which error codes are being ignored rather than using broad directives that
  suppress multiple checks.
  ```python
  ## Yes
  # --------------------------------------------------
  # ANN401: Any not allowed in kwargs
  # This signature is written to match `azure.keyvault.secrets.SecretClient.get_secret`, so that SecretClient
  # is seen, to the type checker, as an implementation of the `SecretReader` protocol.
  def get_secret(self, name: str, version: str | None = None, **kwargs: Any) -> SecretContainer:  # noqa: ANN401
      """Read a secret from a vault."""
      ...

  ## No
  # --------------------------------------------------
  def get_secret(self, name: str, version: str | None = None, **kwargs: Any) -> SecretContainer:  # noqa
      """Read a secret from a vault."""
      ...
  ```
- Regularly review and remove directives that are no longer needed as the codebase evolves.
  - Ask permission before removing directives added by others.
  - Be more lintent when `fmt: off` is being used to preserve a multi-line block for readability.


Here is a list of common directives and their purposes:
| **Directive**                     | **Tool**   | **Purpose**                                      
|-----------------------------------|------------|--------------------------------------------------
| `# pylint: disable=<rule>`        | Pylint     | Disables a specific linting rule for the scope.
| `# pylint: enable=<rule>`         | Pylint     | Re-enables a previously disabled rule.
| `# noqa`                          | Flake8     | Ignores all linting errors on that line. NEVER use this form.
| `# noqa: <error_code>`            | Flake8     | Ignores only the specified error(s) on that line
| `# type: ignore`                  | mypy       | Ignores type-checking errors for that line. NEVER use this form.
| `# type: ignore[<error_code>]`    | mypy       | Ignores only specific type-checking error(s).
| `# fmt: off` / `# fmt: on`        | Black      | Disables/enables auto-formatting for a code block.
| `# isort: skip`                   | isort      | Skips sorting imports for that line.
| `# isort: off` / `# isort: on`    | isort      | Disables/enables import sorting for a block.
| `# pyright: ignore`               | Pyright    | Ignores all type-checking errors for that line. NEVER use this form.   
| `# pyright: report<RuleName>=false` | Pyright | Disables a specific rule for the file or block.
