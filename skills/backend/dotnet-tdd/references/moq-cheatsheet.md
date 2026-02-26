# Moq Cheat Sheet

## Setup Return Values

```csharp
// Return a value
mock.Setup(r => r.GetByIdAsync(It.IsAny<Guid>()))
    .ReturnsAsync(someEntity);

// Return null
mock.Setup(r => r.GetByIdAsync(It.IsAny<Guid>()))
    .ReturnsAsync((Invoice?)null);

// Return based on input
mock.Setup(r => r.GetByIdAsync(It.Is<Guid>(id => id == specificId)))
    .ReturnsAsync(specificEntity);
```

## Capture Arguments

```csharp
Invoice? captured = null;
mock.Setup(r => r.AddAsync(It.IsAny<Invoice>()))
    .Callback<Invoice>(i => captured = i);
```

## Verify Calls

```csharp
// Called once
mock.Verify(r => r.AddAsync(It.IsAny<Invoice>()), Times.Once);

// Never called
mock.Verify(r => r.UpdateAsync(It.IsAny<Invoice>()), Times.Never);

// With argument matching
mock.Verify(r => r.AddAsync(It.Is<Invoice>(i =>
    i.BuyerTaxCode == "expected" && i.Status == InvoiceStatus.Draft)),
    Times.Once);

// Called N times
mock.Verify(r => r.SaveAsync(), Times.Exactly(2));
```

## Setup Exceptions

```csharp
// Throw on call
mock.Setup(r => r.GetByIdAsync(It.IsAny<Guid>()))
    .ThrowsAsync(new Exception("DB error"));

// Throw on specific input
mock.Setup(r => r.GetByIdAsync(Guid.Empty))
    .ThrowsAsync(new ArgumentException("Invalid ID"));
```

## Setup Sequences

```csharp
mock.SetupSequence(r => r.GetNextAsync())
    .ReturnsAsync(firstItem)
    .ReturnsAsync(secondItem)
    .ThrowsAsync(new InvalidOperationException("No more items"));
```
