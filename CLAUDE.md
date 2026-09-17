# Project Instructions

This repository is a reusable template for ASP.NET Core applications based on Clean Architecture.

## Core Defaults

- Use Clean Architecture as the default architectural style.
- Do not change the architectural style or introduce major architectural patterns without explicit approval.
- Prefer simple, maintainable solutions over unnecessary abstractions.
- Do not introduce a framework, library, or infrastructure dependency unless it provides clear value for the current requirement.

## Development Workflow

- For non-trivial changes, inspect the existing code and understand the affected area before editing.
- Keep changes within the requested scope.
- Do not refactor unrelated code as part of a feature or bug fix.
- Preserve existing behavior unless the task explicitly requires changing it.
- If requirements or architectural intent are ambiguous and the decision has significant consequences, ask before implementing.

## Verification

Before declaring a code change complete:

- Build all affected projects.
- Run relevant automated tests.
- Review the final diff for unintended changes.
- Do not report completion while known build or test failures remain.
- Remove disposable debugging or verification artifacts unless they have an explicit reusable purpose.

## Git

- Do not commit, push, merge, rebase, reset, force-push, or delete branches unless explicitly requested.
- Keep unrelated changes out of the current branch.
- Parallel implementation work must use separate branches and worktrees.

## Instructions

Detailed standards are defined in `.claude/rules/`.

Use relevant skills from `.claude/skills/` when their specialized guidance or workflow is useful.