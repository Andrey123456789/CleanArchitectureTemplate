# Solution Layout

This reference defines the default physical repository layout.

Use it together with `architecture.md`.

# Top-Level Layout

```text
ProjectName/
├── src/
│   ├── ProjectName.Domain/
│   ├── ProjectName.Application/
│   ├── ProjectName.Infrastructure/
│   └── ProjectName.Api/
│
├── frontend/
│   └── ProjectName.Web/
│
├── tests/
│   ├── ProjectName.Domain.Tests/
│   ├── ProjectName.Application.Tests/
│   └── ProjectName.IntegrationTests/
│
├── docs/
│
├── .claude/
│
├── ProjectName.sln
├── Directory.Build.props
├── Directory.Packages.props
├── global.json
├── .editorconfig
├── .gitignore
└── README.md
```

This is a target structure.

Do not create optional directories or projects merely to make the repository
match the diagram.

# Source Projects

All backend production projects live under:

```text
src/
```

The normal Clean Architecture projects are:

```text
ProjectName.Domain
ProjectName.Application
ProjectName.Infrastructure
ProjectName.Api
```

Detailed backend layout belongs to:

```text
backend-layout.md
```

# Frontend

The Angular application lives outside the .NET `src` tree:

```text
frontend/
└── ProjectName.Web/
```

The Angular workspace owns its own:

```text
package.json
package-lock.json
angular.json
tsconfig files
src/
public/
```

Do not manually impose an obsolete Angular directory structure when the current
Angular CLI generates a different supported structure.

Detailed frontend architecture belongs to the `angular` skill.

# Tests

.NET test projects live under:

```text
tests/
```

Possible projects include:

```text
ProjectName.Domain.Tests/
ProjectName.Application.Tests/
ProjectName.IntegrationTests/
```

Create only the projects currently justified by the application.

Typical intent:

```text
Domain.Tests
    domain behavior and invariants

Application.Tests
    application-service/use-case behavior

IntegrationTests
    API + Application + Infrastructure integration
```

Integration tests may reference the API project for
`WebApplicationFactory<Program>`.

Detailed test policy belongs to the `testing` rule and skill.

# Documentation

Use:

```text
docs/
```

for maintained project documentation.

Possible directories include:

```text
architecture/
decisions/
api/
```

Create them only when corresponding documentation exists.

Do not create empty documentation trees.

# Shared .NET Build Configuration

For this multi-project template, use repository-level build/package configuration
by default.

## `Directory.Build.props`

Use it for shared .NET project settings such as:

```text
TargetFramework
Nullable
ImplicitUsings
analysis/compiler settings
```

Do not place genuinely project-specific settings there.

## `Directory.Packages.props`

Use NuGet Central Package Management.

Keep package versions in one place rather than repeating versions across
individual project files.

Package selection and version policy is defined in:

```text
technology-stack.md
```

## `global.json`

Pin the repository to the intended supported .NET SDK line.

When scaffolding, select the current stable SDK for the target .NET release.

Do not select an RC/preview SDK unless explicitly requested.

Allow normal patch/feature-band servicing according to the repository's chosen
`rollForward` policy.

# `.editorconfig`

Keep deterministic repository-wide formatting and code-style configuration here.

Prefer enforceable tooling configuration over duplicating mechanical formatting
rules in Claude instructions.

# Git Ignore

Ignore normal generated/local artifacts, including where applicable:

```text
bin/
obj/
.vs/
node_modules/
dist/
TestResults/
coverage/
local secret/config overrides
```

Do not ignore files that are intentionally part of the Development configuration
contract.

# Optional Infrastructure

Do not create these merely because a future application might need them:

```text
Docker files
deployment manifests
Kubernetes
cloud infrastructure
messaging infrastructure
additional services
```

Introduce them when a concrete requirement exists.