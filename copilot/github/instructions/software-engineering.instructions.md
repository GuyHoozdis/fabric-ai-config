---
description: Standards and best practices broadly applicable regardless of language/framework.
---

# Software Engineering Guidelines

These guidelines apply broadly across languages and frameworks. See language-specific files for additional conventions.

## Principles

Build simple, compact, clear, modular, and extensible code that can be easily maintained and repurposed by developers other than its creators.

Build functions, classes, modules, components, applications, and services such that they are composable into larger systems.  They do one thing and do it well.


- Use SOLID principles.
- One entrance, one exit per function/method.
  - Use guard clauses to detect errors early and abort (the one exception to one exit rule)
  - Guard clauses improve readability.
  - Guard clauses reduce nesting.
- Avoid deep nesting.  Invert conditions and use guard clauses to reduce nesting.
- Prefer composition over inheritance.
- Avoid premature optimization.

- Write code for humans first, machines second.
- Prioritize readability and maintainability.
- Document intent, not implementation details.
- Test behavior, not implementation details.

- Fail fast and loudly; avoid silent failures.
- Write tests that are fast, reliable, and isolated.

- Use meaningful names for variables, functions, classes, and modules.
- Prefer immutability where possible.
- Handle errors gracefully; use exceptions for exceptional cases.

- DRY (Don't Repeat Yourself).  By the 3rd time you write something, consider refactoring it into a reusable component.
- KISS (Keep It Simple, Stupid).  Don't over engineer.  Do the simplest thing that could possibly work.
- YAGNI (You Aren't Gonna Need It).  Don't spend time/effort on a feature that isn't explicitly required.


## The Zen of Python

While written about Python, these aphorisms are broadly applicable.  From small scripts or utilities to large enterprise grade systems; ponder what these ideas mean and how they apply to software engineering in general.
```
$ python -m this
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```
