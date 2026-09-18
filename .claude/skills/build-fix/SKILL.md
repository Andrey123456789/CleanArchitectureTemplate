---
name: build-fix
description: >
  Diagnose and fix broken .NET builds or failing test suites through a bounded,
  evidence-driven iteration loop. Use when compilation fails, tests are broken
  after a change, package/API upgrades introduce errors, or the user explicitly
  asks to make the solution green.
---

# Build Fix

## Goal

Restore the repository to a green build/test state without hiding failures or
introducing unrelated changes.

## Principles

- Diagnose before editing.
- Fix root causes before downstream symptoms.
- Keep each iteration small.
- Re-run the failing verification after each meaningful fix.
- Do not weaken tests merely to make them pass.
- Do not suppress compiler/analyzer errors without understanding them.
- Preserve architectural rules while fixing compilation.
- Stop when progress has clearly stalled rather than making speculative edits.

## Build-Fix Flow

Start with the narrowest command that reproduces the problem.

Typical commands:

```bash
dotnet build
```

or:

```bash
dotnet build src/MyApp.Api/MyApp.Api.csproj
```

Capture the complete error output.

Group related errors by probable root cause, for example:

```text
missing reference / namespace
API signature changed
nullability
type mismatch
interface contract changed
package version change
generated code problem
configuration/build file problem
```

Fix the highest-leverage root cause first.

One missing project reference or changed interface may produce many secondary
errors.

After each fix:

```bash
dotnet build
```

Compare the new failure set to the previous one.

## Test-Fix Flow

When compilation succeeds but tests fail:

1. Read the failing test.
2. Read the relevant production behavior.
3. Determine whether the defect is in production code, test setup, test
   expectation, or an intentionally changed contract.
4. Make the smallest correct change.
5. Re-run the affected test.
6. Run broader relevant tests after the focused failure is fixed.

Do not change:

```text
strong assertion
→ weak assertion
```

merely to obtain a green test.

If expected behavior intentionally changed, update the test to express the new
contract and ensure the change was actually requested.

## Iteration Limits

Use bounded iteration.

A reasonable default is several focused attempts, not an unlimited loop.

Stop and reassess when:

- the same error survives repeated materially different fixes;
- fixes cause increasing unrelated failures;
- the installed SDK/toolchain is incompatible;
- required external infrastructure is unavailable;
- the correct behavior cannot be determined from the project or request.

Do not repeatedly apply variants of the same failed guess.

## Regressions

If a fix introduces new failures:

- determine whether they expose a legitimate dependency of the change;
- otherwise revert or correct that fix before continuing.

Do not leave the codebase knowingly worse than the state from which the
iteration started.

## Package/API Upgrades

When failures follow a package upgrade:

- inspect the actual installed package/API version;
- read compiler diagnostics;
- update call sites deliberately;
- check release/migration documentation when necessary.

Do not fabricate new API signatures from memory when they can be verified.

## Architecture

A build fix must not bypass project architecture merely to compile.

Do not solve an Application/Infrastructure dependency problem by introducing an
invalid project reference if the architecture requires inversion through an
abstraction.

Follow `architecture.md` and relevant skills.

## Tests Created During Diagnosis

If a new behavioral test or probe is used to establish correct behavior, follow
the `testing` skill.

The behavior must remain represented by maintained automated coverage unless
equivalent coverage already exists.

## Completion

A build-fix task is complete when relevant evidence is green.

Typically:

```text
build passes
relevant tests pass
no known new regression remains
final diff contains no accidental workaround
```

Use the `verify` skill for the final verification pass when appropriate.

## Reporting

Report:

```text
Initial failure
Root cause
Changes made
Build result
Test result
Anything still unresolved
```

Do not claim the problem is fixed when a known relevant failure remains.