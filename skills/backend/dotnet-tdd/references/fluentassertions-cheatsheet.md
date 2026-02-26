# FluentAssertions Cheat Sheet

## Basic Assertions

```csharp
result.Should().NotBeNull();
result.Should().Be(expected);
result.Should().BeEquivalentTo(expected);
result.Should().NotBe(unexpected);
result.Should().BeTrue();
result.Should().BeFalse();
```

## String Assertions

```csharp
name.Should().Be("expected");
name.Should().Contain("partial");
name.Should().StartWith("prefix");
name.Should().BeNullOrWhiteSpace();
name.Should().MatchRegex(@"\d{10}");
```

## Collection Assertions

```csharp
list.Should().HaveCount(3);
list.Should().ContainSingle().Which.Should().BeOfType<MyType>();
list.Should().BeEmpty();
list.Should().NotBeEmpty();
list.Should().Contain(x => x.Id == expectedId);
list.Should().OnlyContain(x => x.IsActive);
list.Should().BeInAscendingOrder(x => x.CreatedAt);
```

## Exception Assertions

```csharp
// Sync
act.Should().Throw<DomainException>().WithMessage("*required*");
act.Should().NotThrow();

// Async
await act.Should().ThrowAsync<ValidationException>();
await act.Should().NotThrowAsync();

// Inner exception
act.Should().Throw<AggregateException>()
    .WithInnerException<InvalidOperationException>();
```

## Type Assertions

```csharp
result.Should().BeOfType<InvoiceDto>();
result.Should().BeAssignableTo<IEntity>();
result.Should().NotBeOfType<string>();
```

## Numeric Assertions

```csharp
amount.Should().BeGreaterThan(0);
amount.Should().BeInRange(1, 100);
amount.Should().BeApproximately(3.14, 0.01);
```

## DateTime Assertions

```csharp
date.Should().BeAfter(DateTime.UtcNow.AddDays(-1));
date.Should().BeBefore(DateTime.UtcNow);
date.Should().BeCloseTo(expected, TimeSpan.FromSeconds(1));
```
