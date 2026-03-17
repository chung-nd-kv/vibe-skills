# API Step Definition Template

## File Structure

```typescript
/**
 * Step definitions for {FeatureName} API
 * Feature: features/api/{feature_name}.feature
 */
import { Given, When, Then, DataTable } from '@cucumber/cucumber';
import { expect } from '@jest/globals';   // or 'chai'
import { CustomWorld } from '@support/world';
import { {Domain}ApiClient } from '@api_clients/{domain}_api_client';
import { authApiClient } from '@api_clients/auth_api_client';
import { getApiCredentials } from '@support/fixture/api_data';
import type { {Domain}Request, {Domain}Response } from '@models/api/{domain}';
```

---

## Authentication Steps (reusable — check common-api-steps.ts first)

```typescript
Given('I am authenticated as {string}', async function(this: CustomWorld, role: string) {
  const credentials = getApiCredentials(role);
  const response = await authApiClient.login(credentials);

  expect(response.status).toBe(200);
  expect(response.data.accessToken).toBeDefined();

  this.authToken = response.data.accessToken;
  this.refreshAllClientTokens();
});

Given('I am not authenticated', function(this: CustomWorld) {
  this.authToken = undefined;
  this.refreshAllClientTokens();
});
```

---

## Generic HTTP Request Steps

Useful for exploratory/negative tests. Domain-specific steps preferred for happy paths.

```typescript
When('I send a {string} request to {string}', async function(
  this: CustomWorld,
  method: string,
  endpoint: string
) {
  this.lastEndpoint = endpoint;
  const client = this.getApiClient({Domain}ApiClient);
  this.apiResponse = await client.request(method, endpoint);
});

When('I send a {string} request to {string} with body:', async function(
  this: CustomWorld,
  method: string,
  endpoint: string,
  docString: string        // JSON string from feature file
) {
  this.lastEndpoint = endpoint;
  this.lastRequestBody = JSON.parse(docString);
  const client = this.getApiClient({Domain}ApiClient);
  this.apiResponse = await client.request(method, endpoint, {
    body: this.lastRequestBody
  });
});

When('I send a {string} request to {string} with data:', async function(
  this: CustomWorld,
  method: string,
  endpoint: string,
  dataTable: DataTable     // key-value data table
) {
  this.lastEndpoint = endpoint;
  this.lastRequestBody = dataTable.rowsHash();
  const client = this.getApiClient({Domain}ApiClient);
  this.apiResponse = await client.request(method, endpoint, {
    body: this.lastRequestBody
  });
});
```

---

## Domain-Specific Request Steps (Generated per feature)

```typescript
// Example for Product feature
When('I create a product with:', async function(this: CustomWorld, dataTable: DataTable) {
  const requestData = dataTable.rowsHash() as CreateProductRequest;
  const client = this.getApiClient(ProductApiClient);
  this.apiResponse = await client.createProduct(requestData);
});

When('I get the product with id {string}', async function(this: CustomWorld, productId: string) {
  const client = this.getApiClient(ProductApiClient);
  this.apiResponse = await client.getProductById(productId);
});

When('I update product {string} with:', async function(
  this: CustomWorld,
  productId: string,
  dataTable: DataTable
) {
  const updateData = dataTable.rowsHash() as UpdateProductRequest;
  const client = this.getApiClient(ProductApiClient);
  this.apiResponse = await client.updateProduct(productId, updateData);
});

When('I delete the product with id {string}', async function(this: CustomWorld, productId: string) {
  const client = this.getApiClient(ProductApiClient);
  this.apiResponse = await client.deleteProduct(productId);
});

When('I search products with keyword {string}', async function(this: CustomWorld, keyword: string) {
  const client = this.getApiClient(ProductApiClient);
  this.apiResponse = await client.getProducts({ keyword });
});
```

---

## Response Assertion Steps

```typescript
// ── Status code ──────────────────────────────────────────────
Then('the response status should be {int}', function(this: CustomWorld, expectedStatus: number) {
  expect(this.apiResponse?.status).toBe(expectedStatus);
});

Then('the request should succeed', function(this: CustomWorld) {
  expect(this.apiResponse?.ok).toBe(true);
});

Then('the request should fail', function(this: CustomWorld) {
  expect(this.apiResponse?.ok).toBe(false);
});

// ── Body field assertions ─────────────────────────────────────
Then('the response field {string} should be {string}', function(
  this: CustomWorld,
  fieldPath: string,   // supports dot notation: "data.product.name"
  expectedValue: string
) {
  const actual = getNestedValue(this.apiResponse?.data, fieldPath);
  expect(String(actual)).toBe(expectedValue);
});

Then('the response field {string} should be {int}', function(
  this: CustomWorld,
  fieldPath: string,
  expectedValue: number
) {
  const actual = getNestedValue(this.apiResponse?.data, fieldPath);
  expect(Number(actual)).toBe(expectedValue);
});

Then('the response should contain fields:', function(this: CustomWorld, dataTable: DataTable) {
  const expected = dataTable.rowsHash();
  for (const [key, value] of Object.entries(expected)) {
    const actual = getNestedValue(this.apiResponse?.data, key);
    expect(String(actual)).toBe(value);
  }
});

Then('the response body should not be empty', function(this: CustomWorld) {
  expect(this.apiResponse?.data).toBeDefined();
  expect(this.apiResponse?.data).not.toBeNull();
});

// ── Collection assertions ─────────────────────────────────────
Then('the response list should have {int} items', function(this: CustomWorld, count: number) {
  const data = this.apiResponse?.data as { items?: unknown[]; data?: unknown[] };
  const list = data?.items ?? data?.data ?? data;
  expect(Array.isArray(list)).toBe(true);
  expect((list as unknown[]).length).toBe(count);
});

Then('the response list should not be empty', function(this: CustomWorld) {
  const data = this.apiResponse?.data as { items?: unknown[]; data?: unknown[] };
  const list = data?.items ?? data?.data ?? data;
  expect(Array.isArray(list)).toBe(true);
  expect((list as unknown[]).length).toBeGreaterThan(0);
});

// ── Error assertions ──────────────────────────────────────────
Then('the error message should contain {string}', function(
  this: CustomWorld,
  expectedText: string
) {
  const data = this.apiResponse?.data as { message?: string; error?: string };
  const errorMessage = data?.message ?? data?.error ?? '';
  expect(errorMessage).toContain(expectedText);
});

Then('the response should contain validation errors for {string}', function(
  this: CustomWorld,
  fieldName: string
) {
  const data = this.apiResponse?.data as { errors?: Record<string, string[]> };
  expect(data?.errors).toBeDefined();
  expect(data?.errors?.[fieldName]).toBeDefined();
  expect(data?.errors?.[fieldName]?.length).toBeGreaterThan(0);
});
```

---

## Stored Value Steps (for chained scenarios)

```typescript
// Store a value from response to use in subsequent steps
Then('I save the response field {string} as {string}', function(
  this: CustomWorld,
  fieldPath: string,
  variableName: string
) {
  if (!this.savedValues) this.savedValues = {};
  this.savedValues[variableName] = getNestedValue(this.apiResponse?.data, fieldPath);
});

// Use stored value in a step
When('I get the resource with the saved {string}', async function(
  this: CustomWorld,
  variableName: string
) {
  const savedId = this.savedValues?.[variableName];
  expect(savedId).toBeDefined();
  // Use savedId in API call...
});
```

---

## Header Assertion Steps

```typescript
Then('the response header {string} should be {string}', function(
  this: CustomWorld,
  headerName: string,
  expectedValue: string
) {
  const actual = this.apiResponse?.headers?.[headerName.toLowerCase()];
  expect(actual).toBe(expectedValue);
});

Then('the response should have header {string}', function(
  this: CustomWorld,
  headerName: string
) {
  expect(this.apiResponse?.headers?.[headerName.toLowerCase()]).toBeDefined();
});
```

---

## Helper Utilities

```typescript
// Add to src/support/helpers/api_helpers.ts

/**
 * Get nested object value by dot-notation path
 * Example: getNestedValue({data: {product: {name: 'foo'}}}, 'data.product.name') → 'foo'
 */
export function getNestedValue(obj: unknown, path: string): unknown {
  return path.split('.').reduce((current, key) => {
    if (current === null || current === undefined) return undefined;
    return (current as Record<string, unknown>)[key];
  }, obj);
}

/**
 * Build request body from DataTable (rowsHash format)
 * Handles type coercion for numbers and booleans
 */
export function buildRequestBody(dataTable: DataTable): Record<string, unknown> {
  const rawHash = dataTable.rowsHash();
  const result: Record<string, unknown> = {};

  for (const [key, value] of Object.entries(rawHash)) {
    if (value === 'true') result[key] = true;
    else if (value === 'false') result[key] = false;
    else if (!isNaN(Number(value)) && value !== '') result[key] = Number(value);
    else result[key] = value;
  }

  return result;
}
```

---

## Complete Example: Product API Steps

```typescript
/**
 * Step definitions for Product API
 * Feature: features/api/product_management.feature
 */
import { Given, When, Then, DataTable } from '@cucumber/cucumber';
import { expect } from '@jest/globals';
import { CustomWorld } from '@support/world';
import { ProductApiClient } from '@api_clients/product_api_client';
import { authApiClient } from '@api_clients/auth_api_client';
import { getApiCredentials } from '@support/fixture/api_data';
import { buildRequestBody } from '@support/helpers/api_helpers';

// Background
Given('the product API is available', async function(this: CustomWorld) {
  const client = this.getApiClient(ProductApiClient);
  const response = await client.getProducts({ pageSize: 1 });
  expect(response.status).not.toBe(0); // 0 = network error
});

// Create
When('I create a product with:', async function(this: CustomWorld, dataTable: DataTable) {
  const requestData = buildRequestBody(dataTable);
  const client = this.getApiClient(ProductApiClient);
  this.apiResponse = await client.createProduct(requestData as CreateProductRequest);
});

// Read
When('I get products with keyword {string}', async function(this: CustomWorld, keyword: string) {
  const client = this.getApiClient(ProductApiClient);
  this.apiResponse = await client.getProducts({ keyword });
});

// Assert created product
Then('the product should be created successfully', function(this: CustomWorld) {
  expect(this.apiResponse?.status).toBe(201);
  expect(this.apiResponse?.data).toHaveProperty('id');
});

Then('the product name should be {string}', function(this: CustomWorld, name: string) {
  const product = this.apiResponse?.data as ProductResponse;
  expect(product?.name).toBe(name);
});
```
