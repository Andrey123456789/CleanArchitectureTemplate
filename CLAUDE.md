# Project Instructions

This repository is a reusable template for ASP.NET Core + Angular applications
based on Clean Architecture.

## Core Defaults

- Use Clean Architecture as the default architectural style.
- Do not change the architectural style or introduce major architectural
  patterns without explicit approval.
- Prefer simple, maintainable solutions over unnecessary abstractions.
- Do not introduce frameworks, libraries, infrastructure, or architectural
  patterns without a concrete need.
- Keep changes focused on the requested scope.
- Preserve existing behavior unless the task explicitly requires changing it.

## Working Agreement

- Inspect the relevant existing code before making non-trivial changes.
- Follow applicable rules from `.claude/rules/`.
- Use relevant skills from `.claude/skills/` for specialized procedures and
  technical guidance.
- Before declaring implementation work complete, apply the `verify` workflow.
- Do not perform commit, push, merge, rebase, reset, force-push, branch deletion,
  or other repository-changing Git operations unless explicitly requested.
- If a consequential requirement or architectural decision is genuinely
  ambiguous, ask before implementing.
- Before adding or updating any NuGet or npm dependency, follow
  `.claude/rules/dependencies.md`.