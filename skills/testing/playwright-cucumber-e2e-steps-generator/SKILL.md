---
name: playwright-cucumber-e2e-steps-generator
description: Generate Playwright E2E test step definitions from Cucumber feature files. Use when user provides a .feature file and wants to implement step definitions, create Page Object classes, or generate test code. Triggers include requests to implement steps, generate E2E tests from Gherkin scenarios, create POM classes for pages, or convert feature files to Playwright tests. Supports interactive selector discovery via Playwright MCP tools or manual user input.
---

# Playwright Cucumber Steps Generator

Generate TypeScript step definitions and Page Object classes from Cucumber feature files for Playwright E2E testing.

## Workflow Overview

1. **Parse feature file** → Extract scenarios and steps
2. **Check existing implementations** → Identify which steps need implementation
3. **Determine selector strategy** → Ask user: auto-detect via MCP or manual input
4. **Generate Page Object** → Create/update POM class with selectors and methods
5. **Generate step definitions** → Create step file with Given/When/Then implementations

## Step 1: Parse Feature File

Read the provided `.feature` file and extract:
- Feature name and description
- Background steps (if any)
- All scenarios with their steps
- Step parameters (quoted strings, numbers, data tables)

```typescript
// Example parsed structure
interface ParsedFeature {
  featureName: string;
  scenarios: Array<{
    name: string;
    steps: Array<{
      keyword: 'Given' | 'When' | 'Then' | 'And' | 'But';
      text: string;
      params: string[];
    }>;
  }>;
}
```

## Step 2: Check Existing Implementations

Before generating, scan `src/step_definitions/web/` for existing step patterns:

```bash
grep -r "Given\|When\|Then" src/step_definitions/web/*.ts
```

Report to user:
- ✅ Steps already implemented (skip these)
- ⚠️ Steps needing implementation (generate these)

## Step 3: Selector Strategy Decision

**Always ask user before proceeding:**

> Để implement các steps này, tôi cần selectors cho các elements trên trang.
> Bạn muốn tôi:
> 1. **Tự động lấy** - Sử dụng Playwright MCP để đọc page và tìm selectors
> 2. **Bạn cung cấp** - Bạn sẽ cho tôi selectors cho từng element
>
> Chọn option nào?

### Option 1: Auto-detect via MCP

Use `read_page` or `find` tools from Playwright MCP:

```typescript
// Use find tool to locate elements
find({ query: "login button", tabId: <tabId> })
find({ query: "username input field", tabId: <tabId> })
```

### Option 2: Manual Input

Ask user for each element needed:

> Cho step "I click the login button", tôi cần selector cho:
> - Login button: (nhập selector, vd: `button[data-testid="login"]`)

## Step 4: Generate Page Object

Create POM class extending `BasePage`. See [references/page-object-template.md](references/page-object-template.md).

**File location:** `src/page_objects/{page_name}_page.ts`

**Naming conventions:**
- Class: `PascalCase` + `Page` suffix (e.g., `LoginPage`, `DashboardPage`)
- File: `snake_case` + `_page.ts` suffix
- Methods: `camelCase`, action-based (e.g., `clickLoginButton`, `enterUsername`)
- Selectors: private constants, descriptive names

## Step 5: Generate Step Definitions

Create step definition file. See [references/step-definition-template.md](references/step-definition-template.md).

**File location:** `src/step_definitions/web/{feature_name}_steps.ts`

**Patterns:**
- Use `Given`, `When`, `Then` from `@cucumber/cucumber`
- Access page via `this.page` from World context
- Use async/await for all Playwright operations
- Include error handling with descriptive messages

## Code Style Requirements

- TypeScript with strict typing
- async/await for all async operations
- Destructuring for imports and parameters
- Optional chaining (`?.`) and nullish coalescing (`??`)
- Template literals for dynamic strings
- Comments for complex logic
- Follow existing project patterns in `src/support/`

## Project Structure Reference

```
src/
├── page_objects/           # POM classes extend BasePage
│   └── {page}_page.ts
├── step_definitions/
│   └── web/               # Web browser step definitions
│       └── {feature}_steps.ts
└── support/
    ├── world.ts           # Cucumber World context
    ├── helpers/           # Utility functions
    └── fixture/           # Test data (account.ts pattern)
```

## Common Patterns

### Authentication
```typescript
import { getAccountCredentials } from '@support/fixture/account';
const accounts = getAccountCredentials();
const { username, password } = accounts['AccountTest'].users.admin;
```

### Page Navigation
```typescript
await this.page.goto(config.web.baseUrl);
await this.page.waitForLoadState('networkidle');
```

### Element Interactions
```typescript
await this.page.fill(selector, value);
await this.page.click(selector);
await this.page.waitForSelector(selector);
```

## References

- [Page Object Template](references/page-object-template.md) - POM class structure and examples
- [Step Definition Template](references/step-definition-template.md) - Step implementation patterns
- [Common Steps](references/common-steps.md) - Reusable step patterns for auth, navigation, assertions
