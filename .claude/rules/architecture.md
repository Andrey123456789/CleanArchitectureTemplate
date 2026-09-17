---
paths:
  - "src/**/*.cs"
  - "src/**/*.csproj"
  - "tests/**/*.cs"
  - "tests/**/*.csproj"
---

# Architecture Rules

## Dependency Direction

Dependencies must point inward.

- `Domain` depends on no other application project.
- `Application` depends on `Domain`.
- `Infrastructure` depends on `Application` and `Domain`.
- `Api` depends on `Application` and may reference `Infrastructure` only as required for application composition and dependency registration.
- Inner layers must never depend on outer layers.

Do not reference Infrastructure-specific types from Domain or Application.

## Domain

The Domain layer contains business concepts and rules that are independent of infrastructure and transport concerns.

- Keep Domain free of EF Core, ASP.NET Core, HTTP, persistence, and external-service dependencies.
- Put business invariants in Domain objects when they naturally belong there.
- Prefer encapsulation when it protects meaningful invariants.
- Do not introduce a Rich Domain Model or DDD tactical patterns unless justified by concrete domain complexity or explicitly requested.

## Application

The Application layer coordinates application use cases.

- Use Application Services as the default orchestration mechanism.
- Application Services may coordinate Domain objects, repositories, external-service abstractions, and the Unit of Work.
- Define persistence and external-service abstractions in Application when they are required by application use cases.
- Keep infrastructure implementation details out of Application.
- Do not use `DbContext`, `DbSet<T>`, EF Core APIs, or provider-specific types in Application.

Do not introduce CQRS handlers, MediatR, Kommand, Vertical Slice Architecture, or similar mediator-based organization unless explicitly approved.

A service becoming too large is a signal to reconsider its responsibilities, not an automatic reason to introduce another architectural pattern.

## API

ASP.NET Core Controllers are the default HTTP entry point.

Controllers must remain thin:

- bind and validate HTTP input;
- invoke Application services;
- translate application outcomes into HTTP responses.

Controllers must not:

- contain business logic;
- access `DbContext`;
- access concrete repositories;
- construct infrastructure services manually.

`Program.cs` is the application composition root and may register Infrastructure implementations.

## Repositories

Repository interfaces are inward-facing persistence contracts.

- Prefer specific repository interfaces that represent actual Application or Domain persistence needs.
- Repository interfaces normally correspond to meaningful aggregate roots or cohesive persistence boundaries rather than blindly to every database table.
- Define repository interfaces in Application.
- Implement repositories in Infrastructure.
- Keep LINQ-to-EF query construction inside Infrastructure.
- Do not expose `IQueryable<T>`, `DbSet<T>`, `DbContext`, EF Core expressions, or provider-specific types from repository interfaces.
- Return materialized entities, collections, projections, or explicit result models as appropriate.

Example:

```csharp
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(
        Guid id,
        CancellationToken cancellationToken);

    Task AddAsync(
        Order order,
        CancellationToken cancellationToken);
}
```

## Generic Repository

A generic repository may be used only as an Infrastructure implementation detail when it removes genuine duplication.

- Application code must not depend on `IGenericRepository<T>` or another generic CRUD contract.
- Concrete repositories may reuse an Infrastructure-only `Repository<TEntity>` base class.
- Do not expand the generic repository into a replacement for EF Core query capabilities.
- Use-case-specific queries belong in concrete repository implementations.

## Unit of Work

Use `IUnitOfWork` as the normal commit boundary for an Application use case.

```csharp
public interface IUnitOfWork
{
    Task<int> SaveChangesAsync(
        CancellationToken cancellationToken = default);
}