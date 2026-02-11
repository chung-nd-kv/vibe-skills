---
name: dotnet-ddd
description: ".NET Core DDD and Hexagonal/Clean Architecture skill for C# development. Use this skill whenever building domain models, aggregates, entities, value objects, domain events, repositories, use cases, queries, or designing bounded contexts in .NET Core projects. Trigger on any mention of DDD patterns, Clean Architecture, Hexagonal Architecture, Ports and Adapters, CQRS separation, domain modeling, use case handlers, query handlers, outbox pattern, or bounded context design. Also trigger when creating new modules, designing domain layers, or structuring .NET solutions following modular architecture. Even if the user does not say DDD explicitly, use this skill when they are working on domain entities, business logic separation, or application service layers."
---

# .NET Core DDD + Hexagonal/Clean Architecture

This skill guides building .NET Core applications following Domain-Driven Design with
Hexagonal (Ports & Adapters) and Clean Architecture principles.

## Architecture Overview

### Dependency Flow (Inward)

```
Presentation (APIs / Workers)
       │
       ▼
   Application (UseCases, Queries, DTOs)
       │
       ▼
     Domain (Entities, Value Objects, Domain Events, Interfaces)
       │
       ▲
 Infrastructure (Repos, External Services, Adapters)
       │
       ▼
     Core (Shared kernel, Base abstractions)
```

**The dependency rule**: Dependencies always point inward. Domain has ZERO dependencies
on Infrastructure or Application. Infrastructure implements Domain interfaces (Ports & Adapters).

### Modular Solution Structure

Each bounded context is a module with three projects:

```
src/Modules/{ModuleName}/
├── {Namespace}.Application/       # Use Cases, Queries, DTOs, Handlers
│   ├── UseCases/                  # Command-side (write operations)
│   │   └── {Name}UseCase.cs
│   ├── UseCaseHandlers/           # UseCase implementations
│   │   └── {Name}UseCaseHandler.cs
│   ├── Queries/                   # Query-side (read operations)
│   │   └── Get{Name}Query.cs
│   ├── QueryHandlers/             # Query implementations (Dapper)
│   │   └── Get{Name}QueryHandler.cs
│   ├── DTOs/                      # Data Transfer Objects
│   ├── Interfaces/                # Application-level service interfaces
│   └── Helpers/                   # Application utilities
│
├── {Namespace}.Domain/            # Pure domain model
│   ├── Entities/                  # Aggregates and Entities
│   ├── ValueObjects/              # Value Objects
│   ├── Enums/                     # Domain enumerations
│   ├── Events/                    # Domain Events
│   ├── Interfaces/                # Repository interfaces (Ports)
│   └── Exceptions/                # Domain-specific exceptions
│
└── {Namespace}.Infrastructure/    # Adapters
    ├── Repositories/              # Repository implementations (EF Core)
    ├── Services/                  # External service adapters
    └── Persistence/               # DB configurations, migrations
```

Shared layers sit outside modules:

```
src/Core/{Namespace}.Core/              # Base interfaces, shared kernel
src/Infrastructure/{Namespace}.Infrastructure/  # Cross-cutting (DB context, caching, messaging)
src/Presentation/                       # API and Worker entry points
```

---

## CQRS Separation

Strict separation between reads and writes. Read the reference file for complete interface
definitions: `references/core-interfaces.md`

### Write Side: UseCase + UseCaseHandler

UseCases represent business commands. They always use **Repository Pattern** with **EF Core**.

**UseCase** — a data carrier describing the intent:

```csharp
public class CreateInvoiceUseCase : IUseCase
{
    public CreateInvoiceRequest Request { get; }
    public CreateInvoiceUseCase(CreateInvoiceRequest request)
    {
        Request = request ?? throw new ArgumentNullException(nameof(request));
    }
}
```

**UseCaseHandler** — contains the business logic:

```csharp
public class CreateInvoiceUseCaseHandler : IUseCaseHandlerWithInput<CreateInvoiceUseCase>
{
    private readonly IInvoiceRepository _invoiceRepository;
    private readonly IUnitOfWork _unitOfWork;

    public CreateInvoiceUseCaseHandler(
        IInvoiceRepository invoiceRepository,
        IUnitOfWork unitOfWork)
    {
        _invoiceRepository = invoiceRepository;
        _unitOfWork = unitOfWork;
    }

    public async Task HandleAsync(CreateInvoiceUseCase useCase)
    {
        var request = useCase.Request;

        // Domain logic — use factory method on Aggregate
        var invoice = Invoice.Create(
            buyerTaxCode: request.BuyerTaxCode,
            sellerTaxCode: request.SellerTaxCode,
            amount: request.Amount
        );

        await _invoiceRepository.AddAsync(invoice);
        await _unitOfWork.CommitAsync();
    }
}
```

**UseCase variants** (choose the right interface):

| Variant | Interface | When |
|---------|-----------|------|
| Input + Output | `IUseCaseHandler<TUseCase, TOutput>` | Command returns result |
| Input only | `IUseCaseHandlerWithInput<TUseCase>` | Fire-and-forget command |
| Output only | `IUseCaseHandlerWithOutput<TOutput>` | Parameterless command |

**Dispatching** via `IUseCaseService`:

```csharp
// In Controller or Worker
await _useCaseService.ExecuteUseCaseAsync(new CreateInvoiceUseCase(request));
```

### Read Side: Query + QueryHandler

Queries are read-only and always use **Dapper** for performance. Never use EF Core for queries.

```csharp
// Query definition
public class GetInvoiceByIdQuery : IQuery<InvoiceDto?>
{
    public Guid InvoiceId { get; }
    public GetInvoiceByIdQuery(Guid invoiceId) => InvoiceId = invoiceId;
}

// Query Handler — uses Dapper, NOT repositories
public class GetInvoiceByIdQueryHandler : IQueryHandler<GetInvoiceByIdQuery, InvoiceDto?>
{
    private readonly IDbConnection _connection;

    public GetInvoiceByIdQueryHandler(IDbConnection connection)
    {
        _connection = connection;
    }

    public async Task<InvoiceDto?> HandleAsync(GetInvoiceByIdQuery query)
    {
        const string sql = """
            SELECT Id, BuyerTaxCode, SellerTaxCode, Amount, Status
            FROM Invoices
            WHERE Id = @InvoiceId
            """;

        return await _connection.QuerySingleOrDefaultAsync<InvoiceDto>(
            sql, new { query.InvoiceId });
    }
}
```

**Dispatching** via `IQueryService`:

```csharp
var invoice = await _queryService.ExecuteAsync(new GetInvoiceByIdQuery(id));
```

---

## Domain Modeling

### Aggregates & Entities

Aggregates are the consistency boundary. Use factory methods for creation to enforce invariants
and raise domain events.

```csharp
public class Invoice : BaseEntity, IHasDomainEvents
{
    private readonly List<IDomainEvent> _domainEvents = new();

    // Properties — private setters enforce invariants
    public Guid Id { get; private set; }
    public string BuyerTaxCode { get; private set; } = string.Empty;
    public string SellerTaxCode { get; private set; } = string.Empty;
    public decimal Amount { get; private set; }
    public InvoiceStatus Status { get; private set; }

    // IHasDomainEvents
    public IReadOnlyCollection<IDomainEvent> DomainEvents => _domainEvents.AsReadOnly();
    public void AddDomainEvent(IDomainEvent domainEvent) => _domainEvents.Add(domainEvent);
    public void RemoveDomainEvent(IDomainEvent domainEvent) => _domainEvents.Remove(domainEvent);
    public void ClearDomainEvents() => _domainEvents.Clear();

    // Factory method — the ONLY way to create
    public static Invoice Create(string buyerTaxCode, string sellerTaxCode, decimal amount)
    {
        // Guard clauses / invariant validation
        if (string.IsNullOrWhiteSpace(buyerTaxCode))
            throw new DomainException("Buyer tax code is required");

        var invoice = new Invoice
        {
            Id = Guid.NewGuid(),
            BuyerTaxCode = buyerTaxCode,
            SellerTaxCode = sellerTaxCode,
            Amount = amount,
            Status = InvoiceStatus.Draft
        };

        invoice.AddDomainEvent(new InvoiceCreatedEvent(invoice.Id));
        return invoice;
    }

    // Behavior methods — express business rules
    public void Submit()
    {
        if (Status != InvoiceStatus.Draft)
            throw new DomainException("Only draft invoices can be submitted");

        Status = InvoiceStatus.Submitted;
        AddDomainEvent(new InvoiceSubmittedEvent(Id));
    }
}
```

**Key rules for Aggregates:**
1. Use **factory methods** (`Create(...)`) — never expose public constructors for creation
2. Use **private setters** — state changes only through behavior methods
3. **Raise domain events** inside factory methods and behavior methods
4. **Validate invariants** at creation and on every state transition
5. Keep aggregates **small** — reference other aggregates by ID, not by navigation property

### Value Objects

Immutable objects with no identity, compared by value.

```csharp
public record Money(decimal Amount, string Currency)
{
    public static Money VND(decimal amount) => new(amount, "VND");

    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new DomainException("Cannot add different currencies");
        return new Money(Amount + other.Amount, Currency);
    }
}

public record TaxCode
{
    public string Value { get; }

    public TaxCode(string value)
    {
        if (string.IsNullOrWhiteSpace(value) || value.Length > 14)
            throw new DomainException("Invalid tax code");
        Value = value;
    }
}
```

### Domain Events

Events raised by aggregates, saved to Outbox table in the same transaction, then published
via CDC (Debezium) → Kafka → Consumer workers.

```csharp
public interface IDomainEvent
{
    DateTime OccurredOn { get; }
}

public record InvoiceCreatedEvent(Guid InvoiceId) : IDomainEvent
{
    public DateTime OccurredOn { get; } = DateTime.UtcNow;
}
```

**Outbox flow:**
1. Aggregate raises domain event via `AddDomainEvent()`
2. `SaveChangesAsync` interceptor serializes events to `OutboxEvents` table in the SAME transaction
3. CDC (Debezium) captures changes from `OutboxEvents` table
4. Debezium publishes to Kafka topic
5. Consumer workers process the events

### Repository Interfaces (Ports)

Defined in the **Domain** layer. Implemented in **Infrastructure** (Adapters).

```csharp
// Domain/Interfaces/IInvoiceRepository.cs
public interface IInvoiceRepository
{
    Task<Invoice?> GetByIdAsync(Guid id);
    Task AddAsync(Invoice invoice);
    Task UpdateAsync(Invoice invoice);
}
```

**Rules:**
- Repository interfaces live in `Domain/Interfaces/`
- Repository implementations live in `Infrastructure/Repositories/`
- One repository per Aggregate Root
- Repositories work with domain entities, NEVER with DTOs

---

## Error Handling

Use Result pattern as default, with exceptions for truly exceptional cases.

```csharp
public class Result<T>
{
    public bool IsSuccess { get; }
    public T? Value { get; }
    public string? Error { get; }

    public static Result<T> Success(T value) => new() { IsSuccess = true, Value = value };
    public static Result<T> Failure(string error) => new() { IsSuccess = false, Error = error };
}
```

**When to use what:**
- **Result pattern**: Validation failures, business rule violations, expected error paths
- **Exceptions**: Infrastructure failures, unexpected errors, programming bugs
- **DomainException**: Domain invariant violations inside aggregates

---

## Strategic DDD

For guidance on Bounded Context identification, Context Mapping patterns, and
Ubiquitous Language practices, read: `references/strategic-ddd.md`

---

## Checklist: Creating a New Module

1. Identify the Bounded Context and its Ubiquitous Language
2. Create the three projects: `.Domain`, `.Application`, `.Infrastructure`
3. Define Aggregates with factory methods and domain events
4. Define Repository interfaces in Domain
5. Create UseCases and UseCaseHandlers for write operations
6. Create Queries and QueryHandlers (Dapper) for read operations
7. Implement Repositories with EF Core in Infrastructure
8. Register DI in the Presentation layer
9. Wire up Outbox for domain event publishing