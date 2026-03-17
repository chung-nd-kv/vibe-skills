# Test Data Builder Pattern

Use builders to create complex test data with sensible defaults, overriding only what matters for each test.

## Basic Builder

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

    public InvoiceBuilder WithSellerTaxCode(string taxCode)
    {
        _sellerTaxCode = taxCode;
        return this;
    }

    public InvoiceBuilder WithAmount(decimal amount)
    {
        _amount = amount;
        return this;
    }

    public Invoice Build() => Invoice.Create(_buyerTaxCode, _sellerTaxCode, _amount);
}
```

## Usage in Tests

```csharp
// Only override what matters for the test
var invoice = new InvoiceBuilder().WithAmount(0m).Build();          // Testing zero amount
var invoice = new InvoiceBuilder().WithBuyerTaxCode("").Build();   // Testing empty tax code
var invoice = new InvoiceBuilder().Build();                         // Default valid invoice
```

## Builder with State Transitions

```csharp
public class InvoiceBuilder
{
    // ... fields and With methods ...

    public Invoice BuildDraft() => Build(); // Default is Draft

    public Invoice BuildSubmitted()
    {
        var invoice = Build();
        invoice.Submit();
        invoice.ClearDomainEvents();
        return invoice;
    }

    public Invoice BuildApproved()
    {
        var invoice = BuildSubmitted();
        invoice.Approve();
        invoice.ClearDomainEvents();
        return invoice;
    }
}
```

## Request/Command Builders

```csharp
public class CreateInvoiceRequestBuilder
{
    private string _buyerTaxCode = "0101234567";
    private string _sellerTaxCode = "0107654321";
    private decimal _amount = 1_000_000m;

    public CreateInvoiceRequestBuilder WithBuyerTaxCode(string v) { _buyerTaxCode = v; return this; }
    public CreateInvoiceRequestBuilder WithAmount(decimal v) { _amount = v; return this; }

    public CreateInvoiceRequest Build() => new()
    {
        BuyerTaxCode = _buyerTaxCode,
        SellerTaxCode = _sellerTaxCode,
        Amount = _amount
    };
}
```

## When to Use Builders

- Entity has 3+ constructor parameters
- Tests need entities in different states (Draft, Submitted, Approved)
- Multiple tests share similar setup with small variations
- Reducing noise in Arrange sections to focus on what matters
