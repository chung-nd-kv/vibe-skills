---
name: dotnet-tdd
description: .NET Core TDD workflow using xUnit, Moq, and FluentAssertions. Guides the Plan-Test-Implement-Review cycle for C# code. Use when user says "write tests first", "TDD", "Red-Green-Refactor", "unit test this handler", or asks to implement a feature test-driven. Do NOT use for integration tests, E2E tests, BDD/Gherkin scenarios (use bdd-practices skill), or simple code snippets that don't need test coverage.
metadata:
  author: CuongTL
  version: 1.0.0
  tags: [dotnet, tdd, xunit, moq, unit-testing, clean-architecture]
---

# .NET Core TDD Workflow

Test-Driven Development workflow for C# / .NET Core using xUnit + Moq + FluentAssertions.

Every piece of code follows the **Plan → Test (RED) → Implement (GREEN + REFACTOR) → Review** cycle.

---

## Phase 1: PLAN

Analyze the requirement before writing any code or test.

1. **Understand the requirement** — what is the expected behavior?
2. **Identify the component type** — which test strategy applies? (see matrix below)
3. **List test cases** — happy path, edge cases, error cases
4. **Define the public API** — method signatures, input/output types

### Component → Test Strategy Matrix

| Component | What to test | Mocking strategy |
|-----------|-------------|------------------|
| **Aggregate/Entity** | Factory methods, behavior, invariants, domain events | No mocks — pure domain logic |
| **Value Object** | Creation validation, equality, behavior | No mocks — pure logic |
| **UseCaseHandler** | Business flow, repository calls, UoW commit, errors | Mock repositories, UoW, domain services |
| **QueryHandler** | SQL correctness, mapping, null handling | Mock IDbConnection (or integration test) |
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
└── _usings.cs
```

### Naming Convention

```
{MethodUnderTest}_{Scenario}_{ExpectedResult}
```

Examples: `Create_WithValidData_ShouldReturnInvoice`, `HandleAsync_WithValidUseCase_ShouldCallRepositoryAdd`

### Test Structure: Arrange-Act-Assert

Every test follows AAA pattern. For full code examples of each component type, consult `references/test-examples.md`.

```csharp
[Fact]
public async Task HandleAsync_WithValidUseCase_ShouldAddInvoiceAndCommit()
{
    // Arrange
    var useCase = new CreateInvoiceUseCase(new CreateInvoiceRequest
    {
        BuyerTaxCode = "0101234567",
        SellerTaxCode = "0107654321",
        Amount = 1_000_000m
    });

    // Act
    await _handler.HandleAsync(useCase);

    // Assert
    _invoiceRepositoryMock.Verify(
        r => r.AddAsync(It.Is<Invoice>(i =>
            i.BuyerTaxCode == "0101234567" && i.Amount == 1_000_000m)),
        Times.Once);
    _unitOfWorkMock.Verify(u => u.CommitAsync(default), Times.Once);
}
```

### Key RED Phase Rules

1. **Write the test first** — it MUST fail (compile error counts as failing)
2. **One test at a time** — don't write all tests before implementing
3. **Test behavior, not implementation** — focus on WHAT, not HOW
4. **Use FluentAssertions** (`.Should()`) for readable assertions
5. **Use `Theory` + `InlineData`** for parameterized edge cases

For Moq patterns, consult `references/moq-cheatsheet.md`.
For FluentAssertions patterns, consult `references/fluentassertions-cheatsheet.md`.

---

## Phase 3: IMPLEMENT (GREEN + REFACTOR)

### GREEN: Minimal Code to Pass

Write the **absolute minimum** code to make the failing test pass. No premature optimization, no extra features, no "while I'm here" changes.

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
Write ONE test (RED) → Write minimal code (GREEN) → Refactor → Run tests → Next test
```

---

## Phase 4: REVIEW

After implementing the feature, perform a systematic review:

- Are all test cases from the PLAN phase covered?
- Is the test naming consistent (`{Method}_{Scenario}_{Expected}`)?
- Are there missing edge cases?
- Is Arrange-Act-Assert followed in every test?
- Are mocks properly verified (no over-mocking)?
- Does the code follow DDD patterns for the component type?

---

## Bug Fix Workflow

Bug fixes also follow TDD:

1. **PLAN**: Understand the bug and its root cause
2. **TEST (RED)**: Write a test that reproduces the bug — it MUST fail
3. **IMPLEMENT (GREEN)**: Fix the bug — the test passes
4. **REFACTOR**: Clean up if needed
5. **REVIEW**: Verify fix doesn't break other tests

---

## Common Issues

### Test project setup
If the test project doesn't exist yet, create it:
```bash
dotnet new xunit -n {Module}.{Layer}.Tests
dotnet add package Moq
dotnet add package FluentAssertions
```

### Global usings for test projects
Create `_usings.cs` in the test project root:
```csharp
global using Xunit;
global using Moq;
global using FluentAssertions;
```

### Tests won't compile
This is expected in RED phase. The test references classes/methods that don't exist yet. Proceed to GREEN phase to create the minimal implementation.

---

## References

For detailed examples and cheat sheets, consult these files as needed:

- `references/test-examples.md` — Full test examples for each component type (Entity, UseCase, Value Object, Domain Event)
- `references/moq-cheatsheet.md` — Common Moq patterns (setup, verify, capture, throws)
- `references/fluentassertions-cheatsheet.md` — FluentAssertions patterns (basic, collections, exceptions, types)
- `references/test-data-builders.md` — Test Data Builder pattern for complex test data
