# Code Generation Workflow

## Generation Order

Generate code strictly in dependency order:

### Layer 1: Domain
1. **Entity** — Branded ID types, readonly interfaces, factory functions, pure functions
2. **Repository Port** — Interface + InjectionToken
3. Update `domain/index.ts` barrel export

### Layer 2: Application
4. **DTOs** — API response/request interfaces
5. **Mapper** — Static class: toDomain, toCreateDto, toUpdateDto
6. **Queries** — QueryBase for read operations (one per read)
7. **UseCases** — UseCaseBase for write operations (create, update, delete)
8. Update `application/index.ts` barrel exports at all levels

### Layer 3: Infrastructure
9. **API Client** — HttpClient, Observable returns, error handling
10. **Repository Implementation** — Implements domain port, firstValueFrom, Result wrapping
11. **Providers** — Provider[] array with token-based DI
12. Update `infrastructure/index.ts` barrel export

### Layer 4: Presentation
13. **State Service** — Signal-based state, auto-reload stream
14. **Page Component** — State service integration, dialog actions
15. **Grid Component** — Data table wrapper, column definitions
16. **Form Component** — Dialog form, reactive forms, validation
17. **Routes** — Lazy-loaded route configuration with providers

## After Each Layer

- Update barrel exports (index.ts) at every level
- Run lint to check for errors
- Fix any import issues

## Rules

- ALWAYS read an existing reference module before generating
- ALWAYS update barrel exports after creating each file
- NEVER skip index.ts files — they are required for imports to work
- Use deep nesting for application layer
- Match import paths used in the project
