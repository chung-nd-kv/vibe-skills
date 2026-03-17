---
name: angular-tdd
description: Angular TDD workflow with codebase exploration and Figma design integration. Guides the Explore-Plan-Test-Implement-Refactor-Review cycle for Angular applications. Use when implementing Angular features test-first, "TDD", "Red-Green-Refactor", developing UI components from Figma designs, or when user says "write tests first". Do NOT use for Angular architecture setup (use angular-hexagonal), BDD/Gherkin scenarios (use bdd-practices), or E2E tests (use playwright-* skills).
metadata:
  version: 1.0.0
  tags: [angular, tdd, testing, jest, figma, frontend, red-green-refactor]
---

# Angular TDD Workflow

Test-Driven Development workflow for Angular applications using Jest + TestBed.

Every piece of code follows the **Explore → Plan → Test (RED) → Implement (GREEN) → Refactor → Review** cycle.

---

## Phase 0: EXPLORE

Before writing any code, understand the project you are working in.

1. **Identify Angular version** — standalone components or NgModule? Signal-based or RxJS?
2. **Understand project structure** — Nx monorepo? Feature libs? Hexagonal layers?
3. **Find testing setup** — Jest or Karma? TestBed patterns? Existing mock strategies?
4. **Review existing patterns** — component style, state management, data fetching approach
5. **Check styling** — SCSS, Tailwind, Angular Material, component-scoped vs global

### Exploration Output Template

```
Angular Version: [version] ([standalone/NgModule])
Reactivity: [Signals / RxJS / hybrid]
Project Type: [Nx monorepo / standalone app]
Structure: [feature-based / layer-based / hexagonal]
Testing: [Jest / Karma] + [TestBed patterns]
Styling: [SCSS / Tailwind / Angular Material / CSS]
State: [Signals / NgRx / RxJS services / state service pattern]
API Layer: [HttpClient + interceptors / custom client]
Key Conventions: [list notable patterns]
```

For the full exploration checklist, consult `references/exploration-checklist.md`.

---

## Phase 1: PLAN

Analyze the requirement before writing any code or test.

### Step 1: Figma Design Check

**If the task involves UI work**, ask the user:

> "Does this task have a Figma design? If yes, please share the Figma link so I can review the design specifications (layout, spacing, colors, typography, responsive breakpoints)."

When a Figma link is provided:
- Identify all visual components needed
- Note spacing, colors, typography tokens
- Identify responsive breakpoints and variants
- Map design components to Angular components

For detailed Figma analysis process, consult `references/figma-workflow.md`.

### Step 2: Requirement Analysis

1. **Understand the requirement** — what is the expected behavior?
2. **Identify components** — which Angular components, services, use cases are needed?
3. **List test cases** — happy path, edge cases, error states, loading states, empty states
4. **Define the public API** — component inputs/outputs, service methods, return types

### Component → Test Strategy Matrix

| Component | What to test | Mocking strategy |
|---|---|---|
| **Smart/Page Component** | Data flow, child interactions, routing | Mock services via InjectionToken |
| **Dumb/UI Component** | Rendering, inputs/outputs, user interactions | No mocks — test inputs/outputs directly |
| **UseCase/Query** | Business logic, repository calls, Result mapping | Mock repository via InjectionToken |
| **State Service** | Signal mutations, computed values, async actions | Mock use cases/queries |
| **Form Component** | Validation, submission, error display, field interactions | Mock form submission handler |
| **API Client** | Request formation, response mapping, error handling | Mock HttpClient |
| **Pipe/Directive** | Transform logic, DOM behavior | Minimal mocks |

### Plan Output Template

```
Feature: [Feature Name]

Components Needed:
- [ComponentName] — [responsibility]

Test Cases:
1. ✅ Should [happy path behavior]
2. ✅ Should [another happy path]
3. ❌ Should [error case]
4. ❌ Should [edge case]
5. ⏳ Should [loading state]
6. 📭 Should [empty state]
```

---

## Phase 2: TEST (RED)

Write failing tests BEFORE any implementation. Tests must fail for the RIGHT reason.

### Test File Location

Test files live next to the source file: `entity.component.ts` → `entity.component.spec.ts`

### Naming Convention

```typescript
describe('[ComponentName]', () => {
  it('should [expected behavior] when [scenario]', () => { ... });
});
```

### Test Structure: Arrange-Act-Assert

Every test follows the AAA pattern:

```typescript
it('should create entity successfully', async () => {
  // Arrange
  const request = { name: 'Test Entity' };
  const expected = { id: 1, name: 'Test Entity' };
  mockRepository.create.mockResolvedValue(Result.success(expected));

  // Act
  const result = await useCase.execute(request);

  // Assert
  expect(result.isSuccess).toBe(true);
  expect(result.value).toEqual(expected);
  expect(mockRepository.create).toHaveBeenCalled();
});
```

### Key RED Phase Rules

1. **Write the test first** — it MUST fail (compile/type error counts as failing)
2. **One test at a time** — don't write all tests before implementing
3. **Test behavior, not implementation** — focus on what the user sees/experiences
4. **Use the project's existing test utilities** — don't introduce new libraries without reason
5. **Use `jest.Mocked<T>`** for typed mocks with InjectionToken
6. **Test signals** with direct `signal()` assertions

For Angular-specific TDD patterns (component, service, form, HTTP testing), consult `references/tdd-patterns.md`.

---

## Phase 3: IMPLEMENT (GREEN)

### Minimal Code to Pass

Write the **absolute minimum** code to make the failing test pass. No premature optimization, no extra features, no "while I'm here" changes.

- Follow existing project patterns (standalone components, `inject()`, signals)
- Use the project's established styling approach
- Match naming conventions discovered in EXPLORE phase

---

## Phase 4: REFACTOR

After GREEN, refactor with confidence — tests are your safety net.

**Refactoring checklist:**
- Extract reusable components (if pattern repeats 3+ times)
- Extract shared logic into services or use cases
- Consolidate styling (use design tokens, theme variables)
- Ensure proper TypeScript types (no `any`)
- Remove dead code and unused imports
- Check accessibility (semantic HTML, ARIA attributes, keyboard navigation)
- Verify responsive behavior matches design (if Figma provided)

**CRITICAL**: Run tests after EVERY refactoring step. If a test fails, undo the refactor.

### The Inner Loop

```
Write ONE test (RED) → Write minimal code (GREEN) → Refactor → Run tests → Next test
```

---

## Phase 5: REVIEW

After implementing the feature, perform a systematic review:

- Are all test cases from the PLAN phase covered?
- Is the AAA pattern followed consistently in every test?
- Is test naming consistent with project conventions?
- Are there missing edge cases (loading, error, empty states)?
- Does the UI match the Figma design (if provided)?
- Is the component accessible (keyboard, screen reader)?
- Does it follow the project's existing patterns discovered in EXPLORE?
- Are there unnecessary re-renders or performance issues?

---

## Bug Fix Workflow

Bug fixes also follow TDD:

1. **EXPLORE**: Understand the current behavior and reproduce the bug
2. **PLAN**: Identify root cause and affected components
3. **TEST (RED)**: Write a test that reproduces the bug — it MUST fail
4. **IMPLEMENT (GREEN)**: Fix the bug — the test passes
5. **REFACTOR**: Clean up if needed
6. **REVIEW**: Verify fix doesn't break other tests

---

## References

For detailed guidance, consult these files as needed:

- `references/exploration-checklist.md` — What to look for when exploring an Angular codebase
- `references/tdd-patterns.md` — Angular-specific TDD patterns (component, service, form, HTTP testing)
- `references/figma-workflow.md` — How to analyze and work with Figma designs for Angular components
