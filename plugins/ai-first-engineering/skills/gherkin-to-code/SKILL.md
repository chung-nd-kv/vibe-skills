---
name: gherkin-to-code
description: Implement feature từ Gherkin feature file — technical analysis, step definitions, và business logic code. Use when engineer nói "implement feature từ feature file", "gen code từ gherkin", "tạo step definitions", "implement scenarios", "code từ living docs", "triển khai feature", hoặc có .feature file và muốn viết code. Skill này cover 3 phase: technical solution design, step definitions, và implementation code. MANDATORY TRIGGERS: implement feature, gen code, step definitions, gherkin to code, triển khai, code từ feature file.
---

# Gherkin to Code — Feature Implementation

Đọc Gherkin feature file từ living docs → phân tích technical solution → gen step definitions + implementation code. Skill này là bridge từ **WHAT** (feature file) sang **HOW** (code).

## Triết lý

> **Feature file là contract. Code phải thỏa mãn contract đó — không hơn, không kém.**

Skill này KHÔNG quyết định stack, framework, hay conventions — đó là việc của CLAUDE.md và existing code trong repo. Skill này chỉ hướng dẫn **process**: đọc gì → phân tích gì → gen gì → verify gì.

## Ngôn ngữ

- Code comments, commit messages: theo convention repo (CLAUDE.md)
- Technical solution summary: tiếng Việt (consistent với PRD và feature file)
- Code: theo ngôn ngữ lập trình của repo

---

## Phase 1 — Technical Analysis (bắt buộc, engineer review trước khi code)

Phase này là bước architecture cho feature-level. KHÔNG skip — gen code mà không có technical analysis sẽ dẫn tới refactor.

### 1.1. Đọc và hiểu feature file

```
Input: .feature file (hoặc nhiều files nếu feature span nhiều module)

Extract:
├── Feature name + narrative → hiểu context
├── Rules → list business rules cần implement
├── Scenarios per Rule → behaviors cần thỏa mãn
├── Tags → test layer (@api/@web/@integration), priority (@p2/@future)
├── # PRD link → đọc PRD nếu cần thêm context
└── # TODO comments → decisions chưa resolve
```

### 1.2. Scan codebase hiện tại

Đọc repo qua git submodule hoặc trực tiếp:

```
Scan:
├── CLAUDE.md → stack, conventions, patterns
├── Existing step definitions → pattern đang dùng
├── Domain models / entities → data model hiện tại
├── API routes / controllers → endpoints hiện có
├── Database schema → tables, relationships
└── Related features code → follow existing patterns
```

### 1.3. Đề xuất Technical Solution

Output dạng summary để engineer review:

```markdown
## Technical Solution: [Feature Name]

### Tổng quan
[1-2 câu: approach chính để implement feature này]

### Components cần thay đổi

| Component | Thay đổi | Lý do (Rule ID) | Impact |
|-----------|----------|-----------------|--------|
| [service/module] | [new/modify/extend] | [BR-Mxx] | [low/medium/high] |

### Data model changes
| Entity | Thay đổi | Fields | Migration? |
|--------|----------|--------|-----------|
| [entity] | [new table / add columns / modify] | [field list] | [yes/no] |

### API changes
| Endpoint | Method | Thay đổi | Guards (Rules) |
|----------|--------|----------|---------------|
| [path] | [GET/POST/...] | [new/modify] | [BR-Mxx] |

### Quyết định kỹ thuật
- [Decision 1]: [option chọn] — *lý do: [ngắn gọn]*
- [Decision 2]: [option chọn] — *lý do: [ngắn gọn]*

### Breaking changes
- [ ] [Mô tả breaking change nếu có] → *ảnh hưởng: [ai/service nào]*

### Implementation order
1. [Component/step đầu tiên] — *vì: [dependency reason]*
2. [Component/step tiếp theo]
3. ...
```

**Sau khi output Technical Solution → DỪNG LẠI, hỏi engineer:**
> "Technical solution trên có ok không? Cần adjust gì trước khi gen code?"

Chỉ tiếp Phase 2 sau khi engineer confirm.

### 1.4. Lưu Technical Solution (optional)

Nếu engineer muốn lưu, ghi vào living docs:

```
features/[module]/[feature-name].solution.md
```

Hoặc ghi tóm tắt trong feature file:

```gherkin
# Solution: [1 dòng tóm tắt approach]
# Components: [list services/modules affected]
# Decision: [key technical decision]
```

---

## Phase 2 — Step Definitions (test glue code)

### 2.1. Nguyên tắc gen step definitions

**Follow existing patterns TRƯỚC:**
- Scan thư mục step definitions hiện có
- Reuse step đã có nếu match (ví dụ: `Given Minh có gói trả phí đang active` có thể đã có)
- Follow naming convention, file organization của repo
- Chỉ tạo step mới cho behavior chưa có

**Mapping Gherkin → Step:**

```
Gherkin step                    → Step definition làm gì
─────────────────────────────────────────────────────────
Given [trạng thái]              → Setup test data / fixtures
  Given Minh có gói trả phí    → Create merchant + active subscription
  Given Minh có đơn hàng       → Create order with status completed

When [hành động]                → Call API / service method / UI action
  When Minh yêu cầu xuất       → POST /invoices hoặc service.createInvoice()

Then [kết quả]                  → Assert response / state / side-effect
  Then hóa đơn được tạo        → Assert invoice exists, status = Draft
  Then hệ thống từ chối        → Assert error response, no invoice created
```

### 2.2. Step definition theo test layer

Dựa trên tag trong feature file:

**@api scenarios:**
```
Given → DB setup / API setup calls
When  → HTTP request (REST client)
Then  → Assert HTTP response + DB state
```

**@web scenarios:**
```
Given → DB setup + navigate to page
When  → Browser interaction (Playwright/Selenium)
Then  → Assert page content / UI state
```

**@integration scenarios:**
```
Given → Setup across multiple services
When  → Trigger in service A
Then  → Assert observable in service B
```

### 2.3. Output

- Step definition files theo convention repo
- Reuse existing steps khi có thể
- Comments reference Rule ID: `// BR-M01: Chỉ merchant active mới được xuất`

---

## Phase 3 — Implementation Code (business logic)

### 3.1. Implementation theo Rules

Mỗi Business Rule trong feature file → code cụ thể:

```
Rule type               → Code pattern
─────────────────────────────────────────
Permission/Guard        → Middleware / guard clause / policy
Validation              → Validator class / validation function
State transition        → State machine / status update logic
Calculation             → Pure function / service method
Side-effect/Notification → Event handler / background job
Uniqueness/Conflict     → DB constraint + application check
Default/Auto-fill       → Factory / builder / mapper
Time-bound              → Scheduler / expiry check
Cascade/Dependency      → Transaction / saga / event chain
```

### 3.2. Implementation order

```
1. Data model changes (migration nếu cần)
   → Entities, DB schema

2. Core business logic (Rules - Must Have trước)
   → Validation, calculation, state machine

3. API layer (endpoints, controllers)
   → Route, request/response mapping

4. Side-effects (notifications, integrations)
   → Event handlers, background jobs

5. Should Have rules (@p2)
   → Sau khi Must Have green

6. Skip @future/@p3
   → Chỉ có feature file làm documentation
```

### 3.3. Nguyên tắc gen code

- **Follow existing patterns** — đọc code hiện có, follow conventions
- **Mỗi Rule = testable unit** — code phải tổ chức sao cho mỗi rule test được độc lập
- **Exception handling từ feature file** — cột Exception trong Business Rules table → error responses
- **Không over-engineer** — chỉ implement đúng những gì feature file yêu cầu
- **Commit message reference** — `feat(billing): implement invoice submission [BR-M01, BR-M02]`

---

## Phase 4 — Verify

### 4.1. Run tests

```
Chạy step definitions với feature file:
├── Must Have scenarios → PHẢI green
├── Should Have (@p2) → NÊN green
├── @future/@p3 → Skip (pending)
└── @wip → Expected fail ok
```

### 4.2. Checklist trước khi done

- [ ] Mọi Must Have Rule có code implement tương ứng
- [ ] Mọi Must Have Scenario chạy green
- [ ] Exception paths (error scenarios) xử lý đúng
- [ ] Code follow existing patterns trong repo (check CLAUDE.md)
- [ ] Không có code cho business rule KHÔNG có trong feature file
- [ ] Data model changes có migration
- [ ] Breaking changes (nếu có) đã communicate
- [ ] Step definitions reuse existing steps khi có thể
- [ ] Commit messages reference Rule IDs

### 4.3. Nếu test fail

```
Test fail → Đọc scenario lại:
├── Given sai? → Fix test data setup
├── When sai? → Fix API call / action
├── Then sai? → Bug trong implementation HOẶC scenario không chính xác
│   ├── Implementation bug → fix code
│   └── Scenario không chính xác → flag cho Product review
└── Không rõ → đọc PRD (link trong feature file header)
```

---

## Ví dụ thực tế: Engineer prompt → Output

**Engineer prompt:**
```
Implement feature từ features/billing/invoice-submission.feature
```

**Claude Code sẽ:**

```
Phase 1: Technical Analysis
├── Đọc invoice-submission.feature
│   ├── 4 Rules: BR-M01 (permission), BR-M02 (calculation), BR-S01 (form), BR-S02 (notification)
│   ├── 7 Scenarios: 4 @api, 2 @web, 1 @future
│   └── PRD link: https://confluence.../pages/123
├── Scan repo
│   ├── CLAUDE.md: .NET 8, Clean Architecture, EF Core
│   ├── Existing: OrderController, InvoiceEntity (chưa có submit logic)
│   └── Step defs: 12 existing steps trong billing/
├── Output Technical Solution
│   ├── New: InvoiceSubmissionService
│   ├── Modify: InvoiceController (add POST /invoices)
│   ├── New: SubscriptionGuard middleware
│   ├── DB: add column Invoice.SubmittedAt, Invoice.ReferenceNumber
│   └── No breaking changes
└── → HỎI ENGINEER CONFIRM

Phase 2: Step Definitions
├── Reuse: 3 existing steps (merchant setup, order setup, login)
├── New: 4 steps (submit invoice, check status, check error, check total)
└── Files: steps/billing/invoice-submission.steps.cs

Phase 3: Implementation
├── 1. Migration: AddInvoiceSubmissionColumns
├── 2. InvoiceSubmissionService (BR-M01 guard, BR-M02 calc)
├── 3. InvoiceController.Submit() endpoint
├── 4. Skip BR-S02 (@future)
└── Run tests → 6/6 green (1 @future skipped)
```

---

## Reference Files

- `references/step-patterns.md` — Common step definition patterns theo test layer (@api, @web, @integration)
