---
name: verify
description: >
  Verify completed .NET changes before declaring them finished. Covers build,
  relevant tests, behavioral verification, formatting/analyzers when configured,
  architecture checks, dependency/security checks when relevant, and final diff
  review. Use when finishing a feature, bug fix, refactor, or preparing a PR.
---

# Verify Changes

## Goal

Produce evidence that the requested change is complete and has not introduced
obvious regressions.

Verification should match the scope and risk of the change.

Do not require optional tooling that the repository has not configured.

## 1. Inspect the Change

Review the final changed-file set and diff.

Confirm:

- changes match the requested scope;
- unrelated refactoring was not introduced accidentally;
- temporary diagnostics and probes are removed;
- no generated/build artifacts were accidentally added;
- no secrets or environment-specific credentials were introduced.

## 2. Build

Build the affected solution or projects.

Typical command:

```bash
dotnet build
```

Use a more targeted project/solution command when appropriate.

Do not use `--no-restore` unless restore is already known to be complete.

A build failure means the implementation is not complete.

## 3. Run Relevant Tests

Run tests covering the changed area.

Typical command:

```bash
dotnet test --no-build
```

For small solutions, running the complete suite is appropriate.

For large solutions, affected test projects may be run first, followed by any
broader suite required by the repository or change risk.

Any newly failing relevant test must be investigated.

Do not ignore a failure merely because it appears unrelated without checking it.

## 4. Preserve Behavioral Verification

If a new behavioral probe was used to demonstrate that the implementation works,
follow the `testing` skill.

The verified behavior must be represented by maintained automated test coverage,
unless equivalent coverage already exists.

Temporary curl, PowerShell, HTTP, database, or diagnostic scripts are not a
substitute for maintained behavior tests.

Build, compiler, formatter, and static-analysis commands do not themselves
require new tests.

## 5. Architecture Check

When the change touches architectural boundaries, verify the relevant
`architecture.md` rules.

Examples:

- project references;
- API → Application flow;
- repository abstractions;
- EF Core boundary;
- Unit of Work behavior;
- dependency direction.

Do not perform a full architecture audit for a documentation-only or unrelated
change.

## 6. Formatting and Static Analysis

If the repository configures formatting or analyzers, run the appropriate checks.

For example:

```bash
dotnet format --verify-no-changes
```

Do not add a formatter/analyzer dependency solely in order to complete ordinary
verification unless requested.

Treat analyzer output as evidence that still requires context.

## 7. Dependency and Security Verification

For dependency changes or security-sensitive work, inspect package
vulnerabilities and changed security behavior.

For example, when supported by the installed SDK:

```bash
dotnet list package --vulnerable --include-transitive
```

Also inspect relevant changes for:

- secrets;
- authorization;
- SQL/raw query safety;
- CORS/TLS changes;
- sensitive logging;
- disabled certificate validation.

A full security scan is not necessary for every small business-logic change.

## 8. Final Diff Review

Review the final diff after all fixes.

Check for:

- accidental unrelated edits;
- commented-out code;
- temporary logging;
- TODO/FIXME markers introduced unintentionally;
- stale temporary tests or scripts;
- architecture drift;
- behavior that was verified manually but never preserved as a test.

## Fix-and-Retry

When verification fails:

1. Identify the concrete failure.
2. Make the smallest appropriate fix.
3. Re-run the affected verification.
4. Re-run build/tests when the fix changed code.
5. Repeat until the change is green or user input is required.

Do not hide a known failure and declare completion.

## Reporting

Report what was actually run.

Example:

```text
Build: PASS
Tests: PASS — 42 tests
Formatting: PASS
Security/dependency scan: NOT RUN — no dependency/security change
Diff review: PASS
```

Use `NOT RUN` or `NOT APPLICABLE` rather than pretending an unchecked area passed.