# Angular Codebase Exploration Checklist

Use this checklist when starting work on an unfamiliar Angular project or module.

---

## 1. Project Configuration

### package.json
- Angular version (`@angular/core`)
- Build tool (Angular CLI, Nx, custom webpack)
- Test runner (`jest`, `karma`, `vitest`)
- Key dependencies (Angular Material, PrimeNG, Tailwind, NgRx, etc.)
- Scripts: `test`, `build`, `lint`, `serve`

### angular.json / project.json (Nx)
- Builder configuration (esbuild vs webpack)
- Test configuration (jest vs karma)
- Architect targets (build, serve, test, lint)
- Assets, styles, scripts configuration

### tsconfig.json
- `strict` mode enabled?
- Path aliases (`@app/*`, `@shared/*`, `@features/*`)
- `compilerOptions.target` and `module`

---

## 2. Project Type Detection

### Nx Monorepo
- Check for `nx.json` in root
- Check `libs/` and `apps/` directories
- Review `nx.json` for `targetDefaults`, `namedInputs`
- Check module boundary rules in `.eslintrc.json` (`@nx/enforce-module-boundaries`)

### Standalone App
- Single `angular.json` with one project
- `src/app/` as main source directory

### NgModule vs Standalone
- Check `main.ts`: `bootstrapApplication()` = standalone, `platformBrowserDynamic().bootstrapModule()` = NgModule
- Check components: `standalone: true` in `@Component` decorator?
- Check routing: `provideRouter()` vs `RouterModule.forRoot()`

---

## 3. Component Patterns

### Component Style
- **Standalone components** with `imports` array vs **NgModule** declarations
- **Signal inputs**: `input()`, `input.required()` vs `@Input()` decorator
- **Signal outputs**: `output()` vs `@Output() EventEmitter`
- **Change detection**: `OnPush` strategy? Signal-based?
- **Template syntax**: `@if/@for/@switch` (new control flow) vs `*ngIf/*ngFor`

### Component Organization
- Feature-based: `features/<feature>/components/`
- Atomic design: `atoms/`, `molecules/`, `organisms/`
- Smart/Dumb separation: `pages/` (smart) vs `components/` (dumb)
- Barrel exports (`index.ts`) in each directory?

---

## 4. Routing Patterns

- **Lazy loading**: `loadComponent()` / `loadChildren()` vs eager loading
- **Route guards**: Functional guards vs class-based `CanActivate`
- **Resolvers**: Data pre-fetching pattern
- **Route structure**: Nested routes? Named outlets?
- **Route file location**: `<feature>.routes.ts` pattern?

---

## 5. State Management

### Signal-based
- `signal()`, `computed()`, `effect()` in services
- `linkedSignal()` for derived state
- State service pattern with `WritableSignal`

### NgRx
- Store, Actions, Reducers, Effects, Selectors
- Feature state vs root state
- `@ngrx/component-store` for component-level state

### RxJS Services
- `BehaviorSubject` / `ReplaySubject` as state containers
- `Observable` streams for async data

### State Service Pattern
- Injectable service with signals or subjects
- Methods for state mutations
- Computed/derived state

---

## 6. Testing Setup

### Test Runner
- **Jest**: `jest.config.ts`, `setupJest.ts`, `@angular-builders/jest`
- **Karma**: `karma.conf.js`, `test.ts`
- **Vitest**: `vitest.config.ts`, `@analogjs/vitest-angular`

### Testing Patterns
- `TestBed.configureTestingModule()` usage
- Mock strategy: `jest.Mocked<T>` vs `jasmine.SpyObj<T>`
- InjectionToken mocks vs `useValue` providers
- Existing test helpers or factories
- Coverage thresholds in config

### What to Look For in Existing Tests
- AAA pattern (Arrange-Act-Assert) usage
- Signal testing patterns (`component.signal()` assertions)
- Component testing (shallow vs deep rendering)
- Service testing (mock dependencies pattern)

---

## 7. Styling Approach

- **SCSS/CSS**: Component-scoped styles? Global styles?
- **Tailwind CSS**: `tailwind.config.js`, utility classes in templates
- **Angular Material**: `@angular/material`, theme configuration
- **CSS Variables**: Design tokens defined?
- **Responsive approach**: Media queries? Container queries? Tailwind breakpoints?

---

## 8. API Layer

- **HttpClient**: Direct usage or wrapper service?
- **Interceptors**: Auth, error handling, logging
- **API client pattern**: One client per domain entity?
- **Error handling**: Global error handler? Result pattern?
- **Base URL**: Environment-based configuration?

---

## 9. Output Template

Fill this in after exploration:

```
Angular Version: ___
Component Style: [standalone / NgModule]
Reactivity: [Signals / RxJS / hybrid]
Template Syntax: [@if/@for / *ngIf/*ngFor]
Project Type: [Nx monorepo / standalone app]
Structure: [feature-based / hexagonal / atomic]
Testing: [Jest / Karma] + [TestBed patterns]
Mock Strategy: [jest.Mocked<T> / jasmine.SpyObj<T>]
Styling: [SCSS / Tailwind / Material / CSS]
State Management: [Signals / NgRx / RxJS services]
API Layer: [HttpClient + interceptors / custom client]
Routing: [lazy / eager] + [functional guards / class guards]
Key Conventions:
- ___
- ___
- ___
```
