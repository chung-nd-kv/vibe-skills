---
name: dotnet-tdd
description: ".NET Core TDD workflow skill using xUnit and Moq. Use this skill whenever writing new code, implementing features, fixing bugs, or refactoring in .NET Core C# projects. Trigger on any mention of TDD, test-driven development, unit testing, writing tests first, Red-Green-Refactor, or when the user asks to implement a feature, use case, query handler, domain entity, or any code that should be test-driven. Also trigger when the user says implement, build, create a new handler or service or entity, or add a feature, as these all imply the TDD workflow should be followed. Even for bug fixes, use this workflow to write a failing test reproducing the bug first, then fix it."
---

# .NET Core TDD Workflow

Test-Driven Development workflow for C# / .NET Core using xUnit + Moq.

Every piece of code follows the **Plan → Test → Implement → Review** cycle.

---

## The Workflow

```
┌─────────┐     ┌──────────┐     ┌─────────────┐     ┌──────────┐
│  PLAN   │────▶│   TEST   │────▶│ IMPLEMENT   │────▶│  REVIEW  │
│         │     │  (RED)   │     │  (GREEN +   │     │          │
│ Analyze │     │  Write   │     │  REFACTOR)  │     │ Code     │
│ Design  │     │  failing │     │  Minimal    │     │ Review   │
│ Cases   │     │  tests   │     │  then clean │     │ skill    │
└─────────┘     └──────────┘     └─────────────┘     └──────────┘
      ▲                                                    │
      └────────────── next feature/fix ◄───────────────────┘
```

---

## Phase 1: PLAN

Analyze the requirement before writing any code or test.

### Steps

1. **Understand the requirement** — what is the expected behavior?
2. **Identify the component type** — which test strategy applies?
3. **List test cases** — happy path, edge cases, error cases
4. **Define the public API** — method signatures, input/output types

### Component → Test Strategy Matrix

| Component | What to test | Mocking strategy |
|-----------|-------------|------------------|
| **Aggregate/Entity** | Factory methods, behavior methods, invariants, domain events | No mocks — pure domain logic |
| **Value Object** | Creation validation, equality, behavior | No mocks — pure logic |
| **UseCaseHandler** | Business flow, repository calls, UoW commit, error handling | Mock repositories, UoW, domain services |
| **QueryHandler** | SQL correctness, mapping, null handling | Mock IDbConnection (or use integration test) |
| **Domain Service** | Cross-aggregate logic | Mock repositories |
| **Application Service** | Orchestration, validation | Mock dependencies |

### Example Plan Output

```
Feature: Create Invoice UseCase

Test Cases:
1. ✅ Should create invoice with valid data and commit
2. ✅ Should raise InvoiceCreatedEvent domain event
3. ✅ Should call repository.AddAsync with correct entity
4. ❌ Should throw when buyer tax code is empty
5. ❌ Should throw when amount is negative
6. 🔄 Should handle concurrent creation (idempotency)
```

---

## Phase 2: TEST (RED)

Write failing tests BEFORE any implementation. Tests must fail for the RIGHT reason.

### Test Project Structure

```
tests/{Module}.{Layer}.Tests/
├── UseCaseHandlers/
│   └── CreateInvoiceUseCaseHandlerTests.cs
├── QueryHandlers/
│   └── GetInvoiceByIdQueryHandlerTests.cs
├── Entities/
│   └── InvoiceTests.cs
├── ValueObjects/
│   └── MoneyTests.cs
└── _usings.cs          # Global usings for test project
```

### Naming Convention

```
{MethodUnderTest}_{Scenario}_{ExpectedResult}
```

Examples:
- `Create_WithValidData_ShouldReturnInvoice`
- `Create_WithEmptyTaxCode_ShouldThrowDomainException`
- `HandleAsync_WithValidUseCase_ShouldCallRepositoryAdd`
- `Submit_WhenAlreadySubmitted_ShouldThrowDomainException`

### Test Structure: Arrange-Act-Assert

```csharp
public class CreateInvoiceUseCaseHandlerTests
{
    private readonly Mock<IInvoiceRepository> _invoiceRepositoryMock;
    private readonly Mock<IUnitOfWork> _unitOfWorkMock;
    private readonly CreateInvoiceUseCaseHandler _handler;

    public CreateInvoiceUseCaseHandlerTests()
    {
        _invoiceRepositoryMock = new Mock<IInvoiceRepository>();
        _unitOfWorkMock = new Mock<IUnitOfWork>();
        _handler = new CreateInvoiceUseCaseHandler(
            _invoiceRepositoryMock.Object,
            _unitOfWorkMock.Object);
    }

    [Fact]
    public async Task HandleAsync_WithValidUseCase_ShouldAddInvoiceAndCommit()
    {
        // Arrange
        var request = new CreateInvoiceRequest
        {
            BuyerTaxCode = "0101234567",
            SellerTaxCode = "0107654321",
            Amount = 1_000_000m
        };
        var useCase = new CreateInvoiceUseCase(request);

        // Act
        await _handler.HandleAsync(useCase);

        // Assert
        _invoiceRepositoryMock.Verify(
            r => r.AddAsync(It.Is<Invoice>(i =>
                i.BuyerTaxCode == "0101234567" &&
                i.Amount == 1_000_000m)),
            Times.Once);

        _unitOfWorkMock.Verify(u => u.CommitAsync(default), Times.Once);
    }

    [Fact]
    public async Task HandleAsync_WithValidUseCase_ShouldRaiseDomainEvent()
    {
        // Arrange
        var request = new CreateInvoiceRequest
        {
            BuyerTaxCode = "0101234567",
            SellerTaxCode = "0107654321",
            Amount = 500_000m
        };
        var useCase = new CreateInvoiceUseCase(request);
        Invoice? capturedInvoice = null;

        _invoiceRepositoryMock
            .Setup(r => r.AddAsync(It.IsAny<Invoice>()))
            .Callback<Invoice>(i => capturedInvoice = i);

        // Act
        await _handler.HandleAsync(useCase);

        // Assert
        capturedInvoice.Should().NotBeNull();
        capturedInvoice!.DomainEvents.Should().ContainSingle()
            .Which.Should().BeOfType<InvoiceCreatedEvent>();
    }
}
```

### Testing Domain Entities (No Mocks)

```csharp
public class InvoiceTests
{
    [Fact]
    public void Create_WithValidData_ShouldReturnInvoiceWithDraftStatus()
    {
        // Act
        var invoice = Invoice.Create(
            buyerTaxCode: "0101234567",
            sellerTaxCode: "0107654321",
            amount: 1_000_000m);

        // Assert
        invoice.Status.Should().Be(InvoiceStatus.Draft);
        invoice.BuyerTaxCode.Should().Be("0101234567");
        invoice.DomainEvents.Should().ContainSingle()
            .Which.Should().BeOfType<InvoiceCreatedEvent>();
    }

    [Theory]
    [InlineData("")]
    [InlineData(null)]
    [InlineData("   ")]
    public void Create_WithInvalidBuyerTaxCode_ShouldThrowDomainException(string? taxCode)
    {
        // Act
        var act = () => Invoice.Create(
            buyerTaxCode: taxCode!,
            sellerTaxCode: "0107654321",
            amount: 1_000_000m);

        // Assert
        act.Should().Throw<DomainException>()
            .WithMessage("*Buyer tax code*");
    }

    [Fact]
    public void Submit_WhenDraft_ShouldTransitionToSubmitted()
    {
        // Arrange
        var invoice = Invoice.Create("0101234567", "0107654321", 1_000_000m);
        invoice.ClearDomainEvents(); // Clear creation event

        // Act
        invoice.Submit();

        // Assert
        invoice.Status.Should().Be(InvoiceStatus.Submitted);
        invoice.DomainEvents.Should().ContainSingle()
            .Which.Should().BeOfType<InvoiceSubmittedEvent>();
    }
}
```

### Testing Value Objects

```csharp
public class MoneyTests
{
    [Fact]
    public void Add_SameCurrency_ShouldReturnSum()
    {
        var a = Money.VND(100_000m);
        var b = Money.VND(200_000m);

        var result = a.Add(b);

        result.Amount.Should().Be(300_000m);
        result.Currency.Should().Be("VND");
    }

    [Fact]
    public void Add_DifferentCurrency_ShouldThrow()
    {
        var vnd = Money.VND(100_000m);
        var usd = new Money(100m, "USD");

        var act = () => vnd.Add(usd);

        act.Should().Throw<DomainException>();
    }
}
```

### Key RED Phase Rules

1. **Write the test first** — it MUST fail (compile error counts as failing)
2. **One test at a time** — don't write all tests before implementing
3. **Test behavior, not implementation** — focus on WHAT, not HOW
4. **Use FluentAssertions** (`.Should()`) for readable assertions
5. **Use `Theory` + `InlineData`** for parameterized edge cases

---

## Phase 3: IMPLEMENT (GREEN + REFACTOR)

### GREEN: Minimal Code to Pass

Write the **absolute minimum** code to make the failing test pass. No premature optimization,
no extra features, no "while I'm here" changes.

```csharp
// Test says: Create should return invoice with Draft status
// GREEN — just enough to pass:
public static Invoice Create(string buyerTaxCode, string sellerTaxCode, decimal amount)
{
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
```

### REFACTOR: Clean Without Changing Behavior

After GREEN, refactor with confidence — tests are your safety net.

**Refactoring checklist:**
- Extract repeated logic into private methods
- Rename for clarity (match Ubiquitous Language)
- Remove duplication (DRY, but don't over-abstract)
- Apply domain patterns (Value Objects, guard clauses)
- Ensure single responsibility

**CRITICAL**: Run tests after EVERY refactoring step. If a test fails, undo the refactor.

### The Inner Loop

```
Write ONE test (RED)
    → Write minimal code (GREEN)
    → Refactor
    → Run tests (all green?)
    → Write NEXT test (RED)
    → Repeat
```

---

## Phase 4: REVIEW

After implementing the feature, delegate to the **code-reviewer** skill for a systematic review.

The review should cover:
- Does the code follow DDD patterns? (use `dotnet-ddd` skill as reference)
- Are all test cases from the PLAN phase covered?
- Is the test naming consistent?
- Are there missing edge cases?
- Is the Arrange-Act-Assert pattern followed?
- Are mocks properly verified?

---

## Testing Cheat Sheet

### Common Moq Patterns

```csharp
// Setup return value
mock.Setup(r => r.GetByIdAsync(It.IsAny<Guid>()))
    .ReturnsAsync(someEntity);

// Setup to return null
mock.Setup(r => r.GetByIdAsync(It.IsAny<Guid>()))
    .ReturnsAsync((Invoice?)null);

// Capture argument
Invoice? captured = null;
mock.Setup(r => r.AddAsync(It.IsAny<Invoice>()))
    .Callback<Invoice>(i => captured = i);

// Verify called once
mock.Verify(r => r.AddAsync(It.IsAny<Invoice>()), Times.Once);

// Verify never called
mock.Verify(r => r.UpdateAsync(It.IsAny<Invoice>()), Times.Never);

// Verify with argument matching
mock.Verify(r => r.AddAsync(It.Is<Invoice>(i =>
    i.BuyerTaxCode == "expected" && i.Status == InvoiceStatus.Draft)),
    Times.Once);

// Setup throws
mock.Setup(r => r.GetByIdAsync(It.IsAny<Guid>()))
    .ThrowsAsync(new Exception("DB error"));
```

### Common FluentAssertions Patterns

```csharp
// Basic
result.Should().NotBeNull();
result.Should().Be(expected);
result.Should().BeEquivalentTo(expected);

// Collections
list.Should().HaveCount(3);
list.Should().ContainSingle().Which.Should().BeOfType<MyType>();
list.Should().BeEmpty();
list.Should().Contain(x => x.Id == expectedId);

// Exceptions
act.Should().Throw<DomainException>().WithMessage("*required*");
act.Should().ThrowAsync<ValidationException>();
act.Should().NotThrow();

// Type checking
result.Should().BeOfType<InvoiceDto>();
result.Should().BeAssignableTo<IEntity>();
```

### Test Data Builders (for complex test data)

```csharp
public class InvoiceBuilder
{
    private string _buyerTaxCode = "0101234567";
    private string _sellerTaxCode = "0107654321";
    private decimal _amount = 1_000_000m;

    public InvoiceBuilder WithBuyerTaxCode(string taxCode)
    {
        _buyerTaxCode = taxCode;
        return this;
    }

    public InvoiceBuilder WithAmount(decimal amount)
    {
        _amount = amount;
        return this;
    }

    public Invoice Build() => Invoice.Create(_buyerTaxCode, _sellerTaxCode, _amount);
}

// Usage in tests:
var invoice = new InvoiceBuilder().WithAmount(500_000m).Build();
```

---

## Bug Fix Workflow

Bug fixes also follow TDD:

1. **PLAN**: Understand the bug and its root cause
2. **TEST (RED)**: Write a test that reproduces the bug — it must FAIL
3. **IMPLEMENT (GREEN)**: Fix the bug — the test passes
4. **REFACTOR**: Clean up if needed
5. **REVIEW**: Verify fix doesn't break other tests

```csharp
[Fact]
public void Submit_WhenAmountIsZero_ShouldThrowDomainException()
{
    // This test reproduces the bug: zero-amount invoices were being submitted
    var invoice = Invoice.Create("0101234567", "0107654321", amount: 0m);

    var act = () => invoice.Submit();

    act.Should().Throw<DomainException>()
        .WithMessage("*amount must be greater than zero*");
}
```