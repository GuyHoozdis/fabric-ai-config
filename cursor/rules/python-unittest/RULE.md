---
description: "This rule describes how unit tests should be written and maintained in this project."
alwaysApply: false
---

- Unit tests MUST use the `unittest` framework provided by Python's standard library - NEVER `pytest`.
- Ensure 1:1 test module at `tests/<package>/<subpackage>/test_<module>.py` mirroring `src/<package>/<subpackage>/<module>.py`.
- Each SUT (function or class) gets its own TestCase named `<SUT>TestCase` (PascalCase function name or class name verbatim + `TestCase`). Examples: `normalize_email` ==> `NormalizeEmailTestCase`; `EmailNormalizer` ==> `EmailNormalizerTestCase`.
- Always add at end of file:
    ```python
    if __name__ == "__main__":
         unittest.main()
    ```
- Prefer `@patch` decorators over context managers; elevate to class-level if all tests share the patch.
- Prefer code that uses dependency injection over `@patch`ing tests.
- Always use `spec=` (or `spec_set=`) when patching to constrain mocks to real interfaces.
- Test bodies should be organized into three code blocks:
   - Arrange: setup inputs, mocks, SUT instance.
   - Act: invoke SUT.
   - Assert: verify outcomes with assertions.
   Separate blocks with a single blank line for clarity.
   DO NOT add a code comment that labels the section.


## Allowed / Disallowed
Allowed: `unittest.mock` (with `spec=`), `polyfactory`, `tempfile` only if user permits.
Disallowed: suppression comments (`# noqa`, `# pragma: no cover`, `# pyright: ignore`, `# type: ignore`, etc.), real network, sleeps, uncontrolled filesystem mutation, assertions on private/internal attributes, test docstrings, inline comments (except allowed TODO/context), `__all__` declarations.


## Naming Rules
Test module: `tests/<same package path>/test_<module>.py`.
TestCase: `<PascalCase(SUT)>TestCase`.
Test method naming patterns (choose one style per TestCase):
- `test_it_<expected_behavior>[_when_condition]`
- `test_<sut>_<expected_behavior>_<condition>`
- `test_<sut>_<condition>_<expected_result>`
Lowercase with underscores; intent-revealing.


