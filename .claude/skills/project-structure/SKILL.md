---
name: project-structure
description: >
  Create or reorganize the physical structure of a .NET Clean Architecture solution,
  including backend projects, project references, test projects, shared build files,
  and an Angular frontend. Use when scaffolding a new solution, adding or removing
  projects, or reorganizing solution-level structure.
---

# Project Structure

Use this skill when creating a new solution or changing its physical project structure.

Architectural rules are defined separately in `.claude/rules/architecture.md`.
Do not redefine or override those rules here.

## Goals

When scaffolding or reorganizing a solution:

1. Keep project boundaries explicit.
2. Enforce dependency direction through project references.
3. Keep backend, frontend, tests, and documentation clearly separated.
4. Avoid creating folders, projects, abstractions, or infrastructure that are not currently needed.
5. Prefer a small structure that can grow naturally over speculative scaffolding.
6. Finish by verifying that the solution restores and builds successfully.

---

# Default Solution Layout

Use the following layout as the default starting point:

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
└── .editorconfig
```

This is a target organization, not a requirement to create every directory immediately.

Do not create empty projects or folders purely for anticipated future needs.

---

# Backend Projects

## Domain

```text
src/
└── ProjectName.Domain/
    ├── Entities/
    └── Enums/
```

The Domain project contains business concepts that do not depend on application,
persistence, HTTP, or infrastructure concerns.

Create additional directories only when the domain actually requires them.

Examples that may be introduced later:

```text
ValueObjects/
DomainServices/
Events/
Exceptions/
```

Do not create these directories by default.

In particular, do not create `ValueObjects`, domain events, or other DDD-oriented
structure merely because the project uses Clean Architecture.

---

## Application

Default organization:

```text
src/
└── ProjectName.Application/
    ├── Abstractions/
    │   ├── Persistence/
    │   └── ExternalServices/
    │
    ├── Services/
    ├── DTOs/
    ├── Validation/
    └── DependencyInjection.cs
```

Create only directories that are needed.

### `Abstractions/Persistence`

Contains inward-facing persistence contracts such as:

```text
ITaskRepository.cs
IOrderRepository.cs
IUnitOfWork.cs
```

Do not put EF Core types or implementations here.

### `Abstractions/ExternalServices`

Contains abstractions required by Application but implemented outside it.

Examples:

```text
IEmailSender.cs
ICurrentUser.cs
IFileStorage.cs
IPaymentGateway.cs
```

Do not create this directory until such a dependency exists.

### `Services`

Contains Application Services and their public contracts when a separate contract
is useful.

Example:

```text
Services/
├── ITaskService.cs
└── TaskService.cs
```

Application Services are the default mechanism for use-case orchestration.

Do not create command/query handler structures, MediatR folders, or Vertical Slice
feature folders unless explicitly requested.

### `DTOs`

Contains application-level data-transfer or operation models when needed.

Do not use this directory as a dumping ground for every data structure.

### `Validation`

Contains application-level validators when validation logic belongs at the
application boundary.

Create it only when validation infrastructure is actually used.

### `DependencyInjection.cs`

Registers Application-owned services.

Typical responsibility:

```text
ITaskService -> TaskService
```

Do not register Infrastructure implementations here.

---

## Infrastructure

Default organization:

```text
src/
└── ProjectName.Infrastructure/
    ├── Persistence/
    │   ├── Configurations/
    │   ├── Repositories/
    │   └── Migrations/
    │
    ├── Services/
    └── DependencyInjection.cs
```

### `Persistence`

Contains EF Core and database-specific implementation details.

Typical contents:

```text
Persistence/
├── AppDbContext.cs
├── Configurations/
├── Repositories/
├── Migrations/
└── UnitOfWork.cs
```

### `Configurations`

Contains EF Core entity configurations when `IEntityTypeConfiguration<T>` is used.

Example:

```text
Configurations/
├── TaskConfiguration.cs
└── OrderConfiguration.cs
```

### `Repositories`

Contains concrete repository implementations.

Example:

```text
Repositories/
├── Repository.cs
├── TaskRepository.cs
└── OrderRepository.cs
```

An Infrastructure-only generic base repository may exist when it removes genuine
implementation duplication.

Do not expose a generic repository contract to Application.

### `Migrations`

Contains EF Core migrations.

Do not manually place unrelated persistence code in this directory.

### `Services`

Contains implementations of technical/external services declared by Application.

Examples:

```text
EmailSender.cs
FileStorage.cs
PaymentGateway.cs
```

Create subdirectories when a technical capability becomes large enough to justify
its own grouping.

### `DependencyInjection.cs`

Registers Infrastructure implementations and technical dependencies.

Typical responsibilities include:

```text
DbContext
repositories
IUnitOfWork
external-service implementations
database/provider configuration
```

---

## API

Default organization:

```text
src/
└── ProjectName.Api/
    ├── Controllers/
    ├── Contracts/
    │   ├── Requests/
    │   └── Responses/
    ├── Middleware/
    ├── Extensions/
    └── Program.cs
```

Create optional directories only when they are needed.

### `Controllers`

Contains ASP.NET Core controllers.

Use Controllers as the default HTTP endpoint mechanism.

Do not create Minimal API endpoint-group structures unless explicitly requested.

### `Contracts`

Contains HTTP-facing request and response contracts when keeping the transport
contract separate from Application models is useful.

Example:

```text
Contracts/
├── Requests/
│   └── CreateTaskRequest.cs
└── Responses/
    └── TaskResponse.cs
```

Do not create duplicate API and Application DTOs unless the separation provides
real value.

### `Middleware`

Contains custom ASP.NET Core middleware.

Do not create this directory until custom middleware exists.

### `Extensions`

May contain API-specific extension methods when needed.

Avoid using extension classes merely to move arbitrary code away from `Program.cs`.

### `Program.cs`

Acts as the application composition root.

It may call:

```csharp
builder.Services.AddApplication();
builder.Services.AddInfrastructure(builder.Configuration);
```

Keep implementation details inside the corresponding projects.

---

# Project References

Project references must enforce Clean Architecture dependency direction.

Use this reference graph:

```text
Domain
  ↑
Application
  ↑
Infrastructure

Api -> Application
Api -> Infrastructure
```

More explicitly:

```text
ProjectName.Domain
    -> no application project references

ProjectName.Application
    -> ProjectName.Domain

ProjectName.Infrastructure
    -> ProjectName.Application
    -> ProjectName.Domain

ProjectName.Api
    -> ProjectName.Application
    -> ProjectName.Infrastructure
```

The API reference to Infrastructure exists for application composition and dependency
registration.

Controllers must not use concrete Infrastructure types merely because the reference
exists.

Never add:

```text
Domain -> Application
Domain -> Infrastructure
Application -> Infrastructure
```

If a requested feature appears to require one of these references, stop and reconsider
the abstraction boundary instead of adding the reference.

---

# Tests

Test projects live outside `src`:

```text
tests/
├── ProjectName.Domain.Tests/
├── ProjectName.Application.Tests/
└── ProjectName.IntegrationTests/
```

Do not create every test project automatically.

Create only the test projects that the current solution requires.

Typical intent:

```text
Domain.Tests
    -> domain behavior and invariants

Application.Tests
    -> application-service/business orchestration tests

IntegrationTests
    -> API/application/infrastructure integration
```

Integration tests may reference the API project when required by
`WebApplicationFactory`.

Detailed testing conventions belong to the testing rule/skill, not here.

Do not duplicate testing policy in this skill.

---

# Angular Frontend

The frontend is Angular with TypeScript.

Keep it outside the .NET `src` directory:

```text
frontend/
└── ProjectName.Web/
    ├── src/
    │   ├── app/
    │   │   ├── core/
    │   │   ├── shared/
    │   │   └── features/
    │   ├── assets/
    │   └── environments/
    ├── angular.json
    ├── package.json
    └── tsconfig.json
```

Do not assume React or another frontend framework.

Do not over-scaffold Angular feature structure before features exist.

### `core`

Reserved for application-wide infrastructure and singleton concerns when needed.

Examples may include:

```text
authentication
HTTP interceptors
route guards
application-wide services
```

### `shared`

Contains genuinely reusable UI/components/utilities used by multiple features.

Do not move feature-specific code into `shared` merely to avoid duplication.

### `features`

Contains business-facing frontend features.

Example:

```text
features/
├── tasks/
└── users/
```

Detailed Angular conventions belong in frontend-specific rules, not in this skill.

---

# Documentation

Use:

```text
docs/
├── architecture/
├── decisions/
└── api/
```

only when corresponding documentation exists.

### `architecture`

Long-form architecture explanations and diagrams.

### `decisions`

Architecture Decision Records or equivalent documented decisions.

### `api`

API contracts or supporting API documentation when needed.

Do not create empty documentation directories merely to match the template.

---

# Shared Build Configuration

For a multi-project .NET solution, prefer solution-level shared configuration where
it removes duplication.

Typical files:

```text
Directory.Build.props
Directory.Packages.props
.editorconfig
```

## `Directory.Build.props`

Use for shared build/compiler configuration when appropriate.

Examples:

```text
TargetFramework
Nullable
ImplicitUsings
TreatWarningsAsErrors
analysis settings
```

Do not put project-specific settings here.

## `Directory.Packages.props`

Use Central Package Management when the solution contains enough .NET projects for
central package versioning to provide value.

Do not duplicate package versions across individual project files when central package
management is already enabled.

## `.editorconfig`

Use for repository-wide formatting and coding-style rules that tooling can enforce
deterministically.

Prefer deterministic tooling over duplicating enforceable formatting rules in Claude
instructions.

---

# Scaffolding Procedure

When asked to create a new solution, follow this sequence.

## 1. Determine scope

Before creating projects, determine which parts are actually required:

```text
backend only?
Angular frontend?
unit tests?
integration tests?
documentation?
```

Do not scaffold optional parts that the task does not require.

## 2. Create the solution

Typical structure:

```text
ProjectName.sln
src/
frontend/
tests/
docs/
```

## 3. Create backend projects

Normally:

```text
ProjectName.Domain
ProjectName.Application
ProjectName.Infrastructure
ProjectName.Api
```

Use the appropriate .NET project templates.

## 4. Add project references

Add only the references defined in the Project References section.

After adding them, verify the dependency graph.

## 5. Add test projects

Create only requested or currently useful test projects.

Follow the repository's testing conventions when selecting packages/frameworks.

Do not assume xUnit if the project's testing rules specify another framework.

## 6. Create Angular application

When frontend scope is included, create an Angular TypeScript application under:

```text
frontend/ProjectName.Web/
```

Follow the project's Angular/frontend rules for detailed configuration.

Do not substitute React, Vue, or another framework.

## 7. Add shared configuration

Create or update shared build/configuration files only when justified:

```text
Directory.Build.props
Directory.Packages.props
.editorconfig
```

## 8. Add minimal internal directories

Inside each project, create only directories immediately required by the initial
implementation.

Avoid generating an empty directory tree for hypothetical future capabilities.

## 9. Verify

Before finishing scaffolding:

```text
restore packages
build the .NET solution
build the Angular application if created
run relevant tests if test projects exist
inspect project references
inspect the final file tree
```

Do not report successful scaffolding if the generated solution does not build.

---

# Adding a New Project Later

When adding a project to an existing solution:

1. Determine which architectural layer or technical responsibility it belongs to.
2. Check whether a new project is actually warranted instead of another directory in
   an existing project.
3. Add the minimum required project references.
4. Never introduce an inward-to-outward dependency.
5. Register implementations at the composition boundary when necessary.
6. Build the full affected solution after the change.

Do not split projects merely to make the solution appear more architecturally complex.

---

# Reorganizing Existing Projects

When reorganizing an existing solution:

1. Inspect the current structure and project references first.
2. Identify concrete structural problems.
3. Preserve behavior.
4. Move code in small, reviewable steps.
5. Update namespaces and references consistently.
6. Avoid mixing structural reorganization with unrelated feature work.
7. Build and test after restructuring.

Do not perform large project moves merely to make an existing solution match this
template exactly.

The template is a default, not a justification for unnecessary churn.

---

# Do Not Introduce by Default

Do not scaffold these unless explicitly requested or justified by an existing
requirement:

```text
CQRS folders
Commands/
Queries/
Handlers/
Behaviors/
MediatR
Kommand
Vertical Slice feature handlers
SharedKernel
BuildingBlocks
DomainEvents
Specifications
EventBus
Messaging
Modules
Microservices
Docker
Kubernetes
cloud-specific projects
```

Likewise, do not create generic `Common`, `Helpers`, or `Utils` directories without
a clear and cohesive responsibility.

---

# Completion Checklist

Before completing a project-structure task, verify:

- The solution structure matches the requested scope.
- No unnecessary projects or empty speculative directories were created.
- Project references follow the allowed dependency direction.
- Domain has no outer-layer dependencies.
- Application does not reference Infrastructure.
- Infrastructure implementations remain outside Application.
- API acts as the composition root.
- Angular, when present, lives under `frontend/`.
- Test projects live under `tests/`.
- Shared configuration is repository-wide only when appropriate.
- The .NET solution builds.
- The Angular application builds when included.
- Relevant tests pass.
- No unrelated files were reorganized.