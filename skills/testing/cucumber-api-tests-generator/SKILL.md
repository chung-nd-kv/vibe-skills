---
name: cucumber-api-tests-generator
description: Generate API test step definitions and client classes from Cucumber/BDD feature files. Use when user provides a .feature file with API scenarios and wants to implement HTTP request steps, create API client classes, or generate test code for REST API testing. Triggers include requests to implement API test steps, generate API tests from Gherkin scenarios, write step definitions for API calls, create HTTP client wrappers, or convert BDD feature files to API test code. Works with TypeScript/axios stack following Cucumber patterns. Always use this skill when user mentions feature files + API tests, BDD API automation, or Gherkin scenarios involving HTTP calls, endpoints, request/response validation.
---

# Cucumber API Tests Generator

Generate TypeScript step definitions and API Client classes from Cucumber feature files for REST API testing.

## Workflow Overview

1. **Parse feature file** → Extract API scenarios and steps
2. **Analyze API operations** → Identify endpoints, methods, request/response schemas
3. **Check existing implementations** → Avoid duplicates
4. **Generate API Client class** → Typed HTTP client with methods per endpoint
5. **Generate step definitions** → Given/When/Then with axios/supertest calls
6. **Generate type models** → Request/Response TypeScript interfaces

## Step 1: Parse Feature File

Read the `.feature` file and extract API-specific information:

- Feature name and description
- Background steps (auth setup, base URL, etc.)
- All scenarios with steps
- HTTP methods (GET, POST, PUT, DELETE, PATCH)
- Endpoint paths (e.g., `/api/v1/products`)
- Request parameters (path, query, body, headers)
- Expected response status codes and body fields
- Data tables (request payload, expected fields)

```typescript
// Example parsed API scenario
interface ParsedApiScenario {
  name: string;
  tags: string[];          // @smoke, @regression, @auth
  steps: Array<{
    keyword: 'Given' | 'When' | 'Then' | 'And';
    text: string;
    dataTable?: Record<string, string>[];
    docString?: string;    // JSON body as doc string
  }>;
}
```

## Step 2: Analyze API Operations

Identify from the steps:

| Pattern in step | Inference |
|---|---|
| "I send a GET request to {string}" | GET method, dynamic URL |
| "I call the {string} API with body:" | POST/PUT, has request body |
| "the response status should be {int}" | Status assertion |
| "the response should contain {string}" | Body field assertion |
| "I am authenticated as {string}" | Auth setup needed |
| "I set header {string} to {string}" | Custom request headers |

## Step 3: Check Existing Implementations

Scan project for existing API step files:

```bash
# Check existing step definitions
grep -r "Given\|When\|Then" src/step_definitions/api/*.ts 2>/dev/null || echo "No API steps found"

# Check existing API clients
ls src/support/api_clients/ 2>/dev/null || echo "No API clients found"
```

Report to user:
- ✅ Steps already implemented (skip)
- ⚠️ Steps needing new implementation

## Step 4: Generate API Client Class

Create a typed HTTP client class per API domain.

See → [references/api-client-template.md](references/api-client-template.md)

**File location:** `src/support/api_clients/{domain}_api_client.ts`

**Naming conventions:**
- Class: `PascalCase` + `ApiClient` suffix (e.g., `ProductApiClient`, `OrderApiClient`)
- File: `snake_case` + `_api_client.ts`
- Methods: verb + resource (e.g., `getProduct`, `createOrder`, `updateInventory`)
- Response types: `PascalCase` + `Response` suffix
- Request types: `PascalCase` + `Request` suffix

**Key requirements:**
- Extend `BaseApiClient` for shared auth/headers/baseURL logic
- Return typed `ApiResponse<T>` with `status`, `data`, `headers`
- Handle errors without throwing — return error status in `ApiResponse`
- Support Bearer token injection from World context

## Step 5: Generate Step Definitions

Create API step definition file.

See → [references/api-step-template.md](references/api-step-template.md)

**File location:** `src/step_definitions/api/{feature_name}_api_steps.ts`

**Patterns to follow:**
- Use `Given`, `When`, `Then` from `@cucumber/cucumber`
- Store last response in `this.apiResponse` (World context)
- Use `FluentAssertion`-style checks with `expect` from chai or jest
- Support data tables for request body construction
- Support doc strings for raw JSON payloads

## Step 6: Generate TypeScript Models

Create request/response interfaces.

**File location:** `src/models/api/{domain}.ts`

```typescript
// Auto-generate from feature file data tables and expected fields
export interface CreateProductRequest {
  name: string;
  price: number;
  categoryId: string;
  // ... from data table headers in feature
}

export interface ProductResponse {
  id: string;
  name: string;
  price: number;
  // ... from "response should contain" steps
}
```

## Project Structure Reference

```
src/
├── api_clients/                # Typed HTTP clients
│   ├── base_api_client.ts      # Base class with auth, headers, axios setup
│   └── {domain}_api_client.ts  # Per-domain clients
├── step_definitions/
│   ├── api/                    # API step definitions
│   │   └── {feature}_api_steps.ts
│   └── web/                    # (existing) E2E steps
├── models/
│   └── api/                    # TypeScript request/response types
│       └── {domain}.ts
└── support/
    ├── world.ts                # World context (add apiResponse, authToken)
    ├── helpers/
    │   └── api_helpers.ts      # Auth token helpers, request builders
    └── fixture/
        └── api_data.ts         # API test data fixtures
```

## Code Style Requirements

- TypeScript with strict typing — no `any`
- async/await for all HTTP calls
- Destructuring for imports and response data
- Optional chaining (`?.`) for nullable response fields
- Template literals for dynamic endpoints
- Descriptive error messages in assertions
- Group imports: cucumber → api client → models → support

## Common Step Patterns

### Authentication Setup
```typescript
Given('I am authenticated as {string}', async function(this: CustomWorld, role: string) {
  const credentials = getApiCredentials(role);
  this.authToken = await authApiClient.login(credentials);
});
```

### Request Execution
```typescript
When('I send a {string} request to {string}', async function(
  this: CustomWorld, method: string, endpoint: string
) {
  this.apiResponse = await this.apiClient.request(method, endpoint, {
    headers: { Authorization: `Bearer ${this.authToken}` }
  });
});
```

### Response Assertions
```typescript
Then('the response status should be {int}', async function(
  this: CustomWorld, expectedStatus: number
) {
  expect(this.apiResponse.status).toBe(expectedStatus);
});

Then('the response body should contain:', async function(
  this: CustomWorld, dataTable: DataTable
) {
  const expected = dataTable.rowsHash();
  for (const [key, value] of Object.entries(expected)) {
    expect(this.apiResponse.data).toHaveProperty(key, value);
  }
});
```

## References

- [API Client Template](references/api-client-template.md) — BaseApiClient, domain client examples
- [API Step Template](references/api-step-template.md) — Complete step definition patterns for API testing
- [Assertion Patterns](references/assertion-patterns.md) — Response validation, schema checks, data assertions
