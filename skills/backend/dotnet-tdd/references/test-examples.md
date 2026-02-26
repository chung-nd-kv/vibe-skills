# Test Examples by Component Type

## UseCaseHandler Tests

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

## Domain Entity Tests (No Mocks)

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
        invoice.ClearDomainEvents();

        // Act
        invoice.Submit();

        // Assert
        invoice.Status.Should().Be(InvoiceStatus.Submitted);
        invoice.DomainEvents.Should().ContainSingle()
            .Which.Should().BeOfType<InvoiceSubmittedEvent>();
    }
}
```

## Value Object Tests

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

## Bug Fix Test Example

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
