---
name: ef-core
description: >
  Entity Framework Core implementation guidance for the Infrastructure layer.
  Covers DbContext configuration, repositories, Unit of Work implementation,
  tracking, projections, pagination, migrations, transactions, bulk operations,
  compiled queries, interceptors, and query performance.
  Use when implementing or reviewing EF Core persistence, LINQ queries,
  DbContext configuration, migrations, repositories, database transactions,
  or database performance.
---

# Entity Framework Core

## Role in the Architecture

EF Core is an Infrastructure implementation detail.

Application code must not depend on:

- `DbContext`;
- `DbSet<T>`;
- EF Core APIs;
- `IQueryable<T>` backed by EF Core;
- provider-specific database types.

Application defines persistence contracts such as specific repository interfaces
and `IUnitOfWork`. Infrastructure implements those contracts using EF Core.

EF Core's `DbContext` already provides change tracking and unit-of-work behavior
internally. `IUnitOfWork` is the Application-facing abstraction over the commit
boundary; its Infrastructure implementation normally delegates to the same
`DbContext`.

## DbContext Configuration

Keep entity configuration separate with `IEntityTypeConfiguration<T>`.

```csharp
internal sealed class AppDbContext(
    DbContextOptions<AppDbContext> options)
    : DbContext(options)
{
    public DbSet<Order> Orders => Set<Order>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.ApplyConfigurationsFromAssembly(
            typeof(AppDbContext).Assembly);
    }
}
```

```csharp
internal sealed class OrderConfiguration
    : IEntityTypeConfiguration<Order>
{
    public void Configure(EntityTypeBuilder<Order> builder)
    {
        builder.HasKey(x => x.Id);

        builder.Property(x => x.Total)
            .HasPrecision(18, 2);

        builder.HasIndex(x => x.CreatedAt);
    }
}
```

Keep persistence-specific configuration out of Domain entities where practical.

## Registration

Register EF Core and persistence implementations in Infrastructure.

```csharp
public static class DependencyInjection
{
    public static IServiceCollection AddInfrastructure(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        services.AddDbContext<AppDbContext>(options =>
            options.UseSqlServer(
                configuration.GetConnectionString("DefaultConnection")));

        services.AddScoped<IOrderRepository, OrderRepository>();
        services.AddScoped<IUnitOfWork, UnitOfWork>();

        return services;
    }
}
```

Use the database provider selected by the actual project. Do not add a provider
merely because it appears in an example.

## Specific Repositories

Implement Application repository interfaces in Infrastructure.

```csharp
internal sealed class OrderRepository(AppDbContext db)
    : IOrderRepository
{
    public Task<Order?> GetByIdAsync(
        Guid id,
        CancellationToken cancellationToken)
    {
        return db.Orders
            .FirstOrDefaultAsync(
                x => x.Id == id,
                cancellationToken);
    }

    public async Task AddAsync(
        Order order,
        CancellationToken cancellationToken)
    {
        await db.Orders.AddAsync(order, cancellationToken);
    }
}
```

Repositories normally stage changes. They do not call `SaveChangesAsync`
after every operation.

Do not expose EF query objects from repository interfaces.

## Unit of Work Implementation

Unit of Work must contain SaveChangesAsync method and links to all repositories.

```csharp
internal sealed class UnitOfWork(AppDbContext db)
    : IUnitOfWork
{
    public Task<int> SaveChangesAsync(
        CancellationToken cancellationToken = default)
    {
        return db.SaveChangesAsync(cancellationToken);
    }
}
```

Repositories participating in the same application operation must use the same
scoped `AppDbContext`.

## Generic Repository

A generic repository may be used internally in Infrastructure to remove genuine
implementation duplication.

```csharp
internal abstract class Repository<TEntity>(AppDbContext db)
    where TEntity : class
{
    protected DbSet<TEntity> Set => db.Set<TEntity>();
}
```

Do not expose `IGenericRepository<T>` to Application.

Do not force complex EF queries through a generic CRUD abstraction.
Concrete repositories may use the full EF Core API internally.

## Read Queries and Projections

For read-only queries that need only selected data, prefer database-side
projection.

```csharp
public Task<OrderSummary?> GetSummaryAsync(
    Guid id,
    CancellationToken cancellationToken)
{
    return db.Orders
        .Where(x => x.Id == id)
        .Select(x => new OrderSummary(
            x.Id,
            x.Total,
            x.CreatedAt))
        .FirstOrDefaultAsync(cancellationToken);
}
```

Projection is not mandatory when the use case genuinely needs the entity.

Avoid loading a complete aggregate only to return two scalar fields.

## Tracking

Use tracking intentionally.

Use normal tracking queries when loaded entities will be modified and committed
through the current Unit of Work.

Use `AsNoTracking()` for read-only entity queries where change tracking provides
no value.

Do not add `AsNoTracking()` mechanically to projections that already produce
non-entity DTOs.

## Primary-Key Lookups

`FindAsync` is appropriate when:

- querying by primary key;
- an entity instance is required;
- returning an already tracked entity is desirable.

```csharp
var order = await db.Orders.FindAsync(
    [orderId],
    cancellationToken);
```

Use LINQ when additional predicates, projection, related data, or query shaping
are required.

## Related Data

Prefer explicit query shape.

Use projection when only selected related data is needed.

Use `Include` / `ThenInclude` when an entity graph itself is required.

Avoid lazy loading by default because it hides database access and can cause
N+1 queries.

For large collection includes, consider split queries when appropriate and
verify the generated SQL and performance characteristics.

## Pagination

Apply filtering, ordering, and pagination in the database.

```csharp
var query = db.Orders
    .AsNoTracking()
    .Where(x => x.CustomerId == customerId)
    .OrderByDescending(x => x.CreatedAt);

var totalCount = await query.CountAsync(cancellationToken);

var items = await query
    .Skip((page - 1) * pageSize)
    .Take(pageSize)
    .Select(x => new OrderSummary(
        x.Id,
        x.Total,
        x.CreatedAt))
    .ToListAsync(cancellationToken);
```

Always use deterministic ordering for paged queries.

For very large or frequently traversed datasets, consider keyset pagination
instead of large `Skip` offsets when justified.

## Bulk Update and Delete

`ExecuteUpdateAsync` and `ExecuteDeleteAsync` can be useful for set-based
operations.

```csharp
await db.Orders
    .Where(x => x.Status == OrderStatus.Cancelled)
    .ExecuteDeleteAsync(cancellationToken);
```

These operations execute directly against the database and bypass normal change
tracking.

Do not mix them casually with tracked entities representing the same rows.
Understand their transaction and consistency implications before using them.

## Transactions

A single `SaveChangesAsync` is normally sufficient as the transaction boundary
for one application operation.

Do not create explicit transactions by default.

Use an explicit transaction when a concrete use case requires multiple database
operations or multiple `SaveChangesAsync` calls to succeed atomically.

Do not attempt to solve distributed transactions with EF transaction APIs.

## Compiled Queries

Normal EF Core LINQ is the default.

EF Core already caches query compilation based on query shape.

Use `EF.CompileQuery` or `EF.CompileAsyncQuery` only for a measured hot path
where profiling demonstrates that query compilation/cache lookup overhead is
material.

Do not introduce compiled queries merely because a query executes frequently.

## Interceptors

Use EF Core interceptors for infrastructure-level cross-cutting behavior when
they provide a clear benefit, for example auditing.

```csharp
internal sealed class AuditInterceptor(TimeProvider clock)
    : SaveChangesInterceptor
{
    public override ValueTask<InterceptionResult<int>> SavingChangesAsync(
        DbContextEventData eventData,
        InterceptionResult<int> result,
        CancellationToken cancellationToken = default)
    {
        var context = eventData.Context;

        if (context is null)
            return ValueTask.FromResult(result);

        var now = clock.GetUtcNow();

        foreach (var entry in context.ChangeTracker.Entries<IAuditable>())
        {
            if (entry.State == EntityState.Added)
                entry.Entity.CreatedAt = now;

            if (entry.State is EntityState.Added or EntityState.Modified)
                entry.Entity.UpdatedAt = now;
        }

        return ValueTask.FromResult(result);
    }
}
```

Do not hide business behavior in persistence interceptors.

## Migrations

Treat migrations as source code.

Create migrations from the Infrastructure project using the API project as the
startup project when required.

```bash
dotnet ef migrations add AddOrderIndex --project src/MyApp.Infrastructure --startup-project src/MyApp.Api
```

Review generated migrations before committing them.

Pay particular attention to:

- destructive schema changes;
- unexpected column recreation;
- data migrations;
- indexes;
- foreign keys and delete behavior.

Do not automatically apply migrations to production on application startup
unless the project has explicitly adopted and secured that deployment strategy.

For controlled production deployment, generated migration scripts are often
preferable.

## Query Performance

Start with clear normal LINQ.

Optimize after identifying a real problem.

Before introducing specialized optimization, inspect:

- number of database round trips;
- selected columns;
- N+1 behavior;
- indexes;
- generated SQL;
- query plan where necessary;
- amount of data materialized;
- tracking overhead.

Do not use `ValueTask`, compiled queries, raw SQL, caching, or manual pooling
merely as speculative optimization.

## Raw SQL

Use raw SQL only when EF Core cannot express the query effectively or measured
performance justifies it.

Always parameterize external values.

Prefer EF APIs that preserve parameterization rather than constructing SQL with
string concatenation.

## Anti-Patterns

Do not:

- expose `DbContext`, `DbSet<T>`, or `IQueryable<T>` from Infrastructure;
- make Application depend directly on EF Core;
- expose a generic CRUD repository contract to Application;
- call `SaveChangesAsync` independently from every repository operation;
- load all rows and filter in memory when the database can filter;
- use lazy loading by default;
- fire-and-forget EF asynchronous operations;
- introduce compiled queries without evidence;
- introduce raw SQL merely to appear more performant.

## Decision Guide

| Scenario | Default |
|---|---|
| Application persistence dependency | Specific repository interface |
| Commit application changes | `IUnitOfWork.SaveChangesAsync` |
| Repository implementation | EF Core inside Infrastructure |
| Read-only entity query | `AsNoTracking()` when useful |
| Read model | Database-side projection |
| Primary-key entity lookup | `FindAsync` when its semantics fit |
| Complex entity graph | Explicit `Include` / projection |
| Set-based mass update/delete | `ExecuteUpdateAsync` / `ExecuteDeleteAsync` when appropriate |
| Normal query | Ordinary LINQ |
| Measured extremely hot query | Consider compiled query |
| Multiple commits requiring atomicity | Explicit transaction |
| Production schema change | Reviewed migration / controlled deployment |