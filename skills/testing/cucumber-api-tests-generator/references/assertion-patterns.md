# Assertion Patterns for API Testing

## Status Code Assertions

```typescript
// Exact status
expect(this.apiResponse?.status).toBe(200);
expect(this.apiResponse?.status).toBe(201);  // Created
expect(this.apiResponse?.status).toBe(204);  // No Content
expect(this.apiResponse?.status).toBe(400);  // Bad Request
expect(this.apiResponse?.status).toBe(401);  // Unauthorized
expect(this.apiResponse?.status).toBe(403);  // Forbidden
expect(this.apiResponse?.status).toBe(404);  // Not Found
expect(this.apiResponse?.status).toBe(422);  // Unprocessable Entity (validation)
expect(this.apiResponse?.status).toBe(500);  // Server Error

// Range assertions
expect(this.apiResponse?.status).toBeGreaterThanOrEqual(200);
expect(this.apiResponse?.status).toBeLessThan(300);
expect(this.apiResponse?.ok).toBe(true);     // 2xx shorthand
```

---

## Response Body Assertions

### Field Existence
```typescript
// Field exists
expect(this.apiResponse?.data).toHaveProperty('id');
expect(this.apiResponse?.data).toHaveProperty('data.items');  // nested

// Field does not exist
expect(this.apiResponse?.data).not.toHaveProperty('password');
expect(this.apiResponse?.data).not.toHaveProperty('internalNote');

// Field is not null/undefined
const data = this.apiResponse?.data as Record<string, unknown>;
expect(data?.id).toBeDefined();
expect(data?.id).not.toBeNull();
```

### Field Values
```typescript
const product = this.apiResponse?.data as ProductResponse;

// String equality
expect(product?.name).toBe('Laptop Dell XPS');
expect(product?.status).toBe('active');

// Number equality
expect(product?.price).toBe(1500000);
expect(product?.stock).toBe(100);

// Boolean
expect(product?.isActive).toBe(true);

// Partial string match
expect(product?.description).toContain('high performance');

// Pattern match
expect(product?.code).toMatch(/^SP-\d{6}$/);

// Date format
expect(product?.createdAt).toMatch(/^\d{4}-\d{2}-\d{2}T/);
```

### Numeric Comparisons
```typescript
expect(product?.price).toBeGreaterThan(0);
expect(product?.discount).toBeLessThanOrEqual(100);
expect(product?.stock).toBeGreaterThanOrEqual(0);
```

---

## List/Array Assertions

```typescript
const listData = this.apiResponse?.data as { items: ProductResponse[]; total: number };

// Count
expect(listData.items).toHaveLength(10);
expect(listData.items.length).toBeGreaterThan(0);
expect(listData.total).toBeGreaterThanOrEqual(listData.items.length);

// Contains item matching condition
expect(listData.items).toEqual(
  expect.arrayContaining([
    expect.objectContaining({ name: 'Laptop Dell XPS' })
  ])
);

// All items match condition
listData.items.forEach(item => {
  expect(item.status).toBe('active');
  expect(item.price).toBeGreaterThan(0);
});

// No item matches condition (negative)
const hasDeleted = listData.items.some(item => item.status === 'deleted');
expect(hasDeleted).toBe(false);
```

---

## Error Response Assertions

### Standard Error Format
```typescript
interface ErrorResponse {
  message: string;
  code?: string;
  errors?: Record<string, string[]>;
}

const error = this.apiResponse?.data as ErrorResponse;

// Message assertions
expect(error.message).toBe('Product not found');
expect(error.message).toContain('required');

// Error code assertions
expect(error.code).toBe('PRODUCT_NOT_FOUND');

// Validation errors (422)
expect(error.errors).toBeDefined();
expect(error.errors?.['name']).toContain('Name is required');
expect(error.errors?.['price']).toContain('Price must be greater than 0');
```

### Multiple Validation Errors
```typescript
Then('the response should have validation errors:', function(
  this: CustomWorld, dataTable: DataTable
) {
  const expectedErrors = dataTable.hashes();
  const error = this.apiResponse?.data as ErrorResponse;

  expect(this.apiResponse?.status).toBe(422);
  expect(error.errors).toBeDefined();

  for (const { field, message } of expectedErrors) {
    expect(error.errors?.[field]).toBeDefined();
    expect(error.errors?.[field]).toContain(message);
  }
});
```

---

## Pagination Assertions

```typescript
interface PaginatedResponse<T> {
  items: T[];
  total: number;
  page: number;
  pageSize: number;
  totalPages: number;
}

Then('the response should be paginated with {int} items per page', function(
  this: CustomWorld, pageSize: number
) {
  const data = this.apiResponse?.data as PaginatedResponse<unknown>;
  expect(data.pageSize).toBe(pageSize);
  expect(data.items.length).toBeLessThanOrEqual(pageSize);
  expect(data.total).toBeDefined();
  expect(data.totalPages).toBe(Math.ceil(data.total / data.pageSize));
});
```

---

## Schema Validation (Optional — requires ajv or zod)

```typescript
import { z } from 'zod';

const productSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1),
  price: z.number().positive(),
  stock: z.number().int().min(0),
  createdAt: z.string().datetime(),
});

Then('the response should match the product schema', function(this: CustomWorld) {
  const result = productSchema.safeParse(this.apiResponse?.data);
  expect(result.success).toBe(true);
  if (!result.success) {
    // Log specific schema errors
    console.error('Schema errors:', result.error.errors);
  }
});
```

---

## Performance Assertions

```typescript
// Track response time — requires modification in BaseApiClient
interface TimedApiResponse<T> extends ApiResponse<T> {
  durationMs: number;
}

Then('the response time should be under {int} milliseconds', function(
  this: CustomWorld, maxMs: number
) {
  const timedResponse = this.apiResponse as TimedApiResponse<unknown>;
  expect(timedResponse.durationMs).toBeLessThan(maxMs);
});
```

---

## Chained Scenario Patterns

For scenarios where step B uses data from step A:

```typescript
// Feature file example:
// When I create a product with name "Test Product"
// Then I save the product id as "createdProductId"
// When I get the product with saved id "createdProductId"
// Then the response status should be 200

// Step implementation
Then('I save the response {string} as {string}', function(
  this: CustomWorld,
  fieldPath: string,
  variableName: string
) {
  if (!this.savedValues) this.savedValues = {};
  const value = getNestedValue(this.apiResponse?.data, fieldPath);
  this.savedValues[variableName] = value;
  expect(value).toBeDefined();  // Fail fast if field missing
});

When('I get the product with saved id {string}', async function(
  this: CustomWorld,
  variableName: string
) {
  const productId = this.savedValues?.[variableName] as string;
  expect(productId).toBeDefined();

  const client = this.getApiClient(ProductApiClient);
  this.apiResponse = await client.getProductById(productId);
});
```

---

## Common Gherkin → Assertion Mapping

| Gherkin step | Code assertion |
|---|---|
| `status should be 200` | `expect(apiResponse.status).toBe(200)` |
| `response should contain field "id"` | `expect(data).toHaveProperty('id')` |
| `field "name" should be "Laptop"` | `expect(data.name).toBe('Laptop')` |
| `list should have 5 items` | `expect(data.items).toHaveLength(5)` |
| `error message should contain "required"` | `expect(error.message).toContain('required')` |
| `response should be empty` | `expect(data).toBeNull()` |
| `field "price" should be greater than 0` | `expect(data.price).toBeGreaterThan(0)` |
| `response time should be under 500ms` | `expect(durationMs).toBeLessThan(500)` |
