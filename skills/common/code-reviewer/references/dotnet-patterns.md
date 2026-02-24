# .NET / Clean Architecture Review Patterns

Review checklist specific to .NET Core / C# projects using Clean Architecture.

## Clean Architecture Layers

When reviewing, verify changes are in the correct layer:

| Layer | Contains | Should NOT contain |
|-------|----------|-------------------|
| Domain | Entities, Value Objects, Domain Events, Interfaces | Infrastructure concerns, EF references |
| Application | Use Cases, DTOs, Validators, Interfaces | UI logic, direct DB access |
| Infrastructure | EF DbContext, Repositories, External Services | Business logic |
| API/Presentation | Controllers, Middleware, Filters | Business logic, direct DB queries |

### Common violations

- Domain entity with `[Table]` or `[Column]` EF attributes → move to Infrastructure mapping
- Application service calling `HttpClient` directly → inject via interface
- Controller containing business logic → extract to Application use case
- Repository with business rules → move to Domain service

## Entity Framework Core Patterns

### Query optimization
- Use `.AsNoTracking()` for read-only queries
- Use `.Select()` projection instead of loading full entities
- Avoid `Include()` chains deeper than 2 levels
- Use `AsSplitQuery()` for complex includes to avoid cartesian explosion

### Anti-patterns to flag
```csharp
// ❌ N+1 query — loading related data in a loop
foreach (var order in orders)
{
    var items = await _context.OrderItems
        .Where(i => i.OrderId == order.Id).ToListAsync();
}

// ✅ Eager load or batch query
var orders = await _context.Orders
    .Include(o => o.Items)
    .ToListAsync();
```

```csharp
// ❌ Loading all then filtering in memory
var allUsers = await _context.Users.ToListAsync();
var activeUsers = allUsers.Where(u => u.IsActive);

// ✅ Filter at database level
var activeUsers = await _context.Users
    .Where(u => u.IsActive).ToListAsync();
```

## Multi-Tenant Patterns

### Data isolation checks
- Every query MUST filter by TenantId / MerchantId
- Global query filters configured in DbContext
- Verify no cross-tenant data leakage in joins
- Check that tenant context is set before any DB operation

```csharp
// ❌ Missing tenant filter
var invoices = await _context.Invoices
    .Where(i => i.Status == "pending").ToListAsync();

// ✅ Tenant-scoped
var invoices = await _context.Invoices
    .Where(i => i.TenantId == _tenantContext.TenantId)
    .Where(i => i.Status == "pending").ToListAsync();
```

## Async/Await Patterns

- Prefer `async/await` over `.Result` or `.Wait()` (deadlock risk)
- Use `ConfigureAwait(false)` in library code
- Don't use `async void` except for event handlers
- Return `Task` from async methods, not `void`

## Dependency Injection

- Services should depend on interfaces, not implementations
- Verify correct lifetime: Scoped for per-request, Singleton for stateless, Transient for lightweight
- Don't inject Scoped services into Singleton services
- Use `IOptions<T>` pattern for configuration