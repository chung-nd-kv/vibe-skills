# AI-First Engineering — Hướng dẫn cài đặt và sử dụng

Plugin hỗ trợ quy trình phát triển AI-first với BDD (living documentation) cho dự án brownfield. 2 role chính: **Product** và **Engineer**.

---

## Tổng quan kiến trúc

```
Product (Cowork/Claude.ai)          Engineer (Claude Code / IDE)
┌──────────────────────┐            ┌──────────────────────────┐
│  lean-prd skill      │            │  prd-to-gherkin skill    │
│  ↓                   │            │  ↓                       │
│  Lean PRD (Confluence)│──link──→  │  Gherkin .feature files  │
│  ↓                   │            │  ↓                       │
│  prd-to-mockup skill │            │  gherkin-to-code skill   │
│  ↓                   │            │  ↓                       │
│  Wireframe (HTML)    │            │  Technical Solution      │
│                      │            │  ↓                       │
│                      │            │  Step Defs + Code        │
│                      │            │  ↓                       │
│                      │            │  Tech skills (CLAUDE.md) │
│                      │            │  (dotnet-tdd, code-review│
│                      │            │   hoặc skill khác)       │
└──────────────────────┘            └──────────────────────────┘
         │                                    │
         └────── Jira (tracking) ─────────────┘
```

### Source of Truth

| Artifact | Nơi lưu | Trả lời câu hỏi |
|----------|---------|-----------------|
| Lean PRD | Confluence | TẠI SAO làm feature này? Business rules là gì? |
| Feature files (.feature) | Living docs repo | CÁI GÌ hệ thống phải làm? Behaviors nào? |
| Technical Solution | Living docs repo | NHƯ THẾ NÀO ở mức architecture? Components nào thay đổi? |
| Automation tests | Living docs repo | CHẠY ĐƯỢC KHÔNG? Tests pass hay fail? |
| Code | Code repo(s) | NHƯ THẾ NÀO implement chi tiết? |
| Jira | Jira | TRẠNG THÁI task? Ai đang làm gì? |

### Traceability

```
Confluence PRD ←──link──→ Gherkin feature file
     ↕                          ↕
  Jira epic              # PRD: [confluence-url]
  Jira stories           # BR-M01, BR-S01...
```

---

## Cài đặt

### Cách 1: Cowork (cho Product)

1. Mở Cowork desktop app
2. Vào Settings → Plugins
3. Kéo thả file `ai-first-engineering.plugin` vào cửa sổ Cowork
4. Plugin sẽ xuất hiện trong danh sách skills

### Cách 2: Claude Code (cho Engineer)

1. Copy thư mục `ai-first-engineering/skills/` vào thư mục skills của dự án:
   ```bash
   # Từ root dự án living docs
   cp -r ai-first-engineering/skills/* .claude/skills/
   ```

2. Hoặc cài từ .plugin file:
   ```bash
   claude plugin install ai-first-engineering.plugin
   ```

3. Verify skills đã sẵn sàng:
   ```bash
   claude /skills
   ```

---

## Quy trình phát triển

### Role: Product

**Tool:** Cowork hoặc Claude.ai (không cần git, không cần IDE)

**Quy trình:**

```
1. Mô tả feature idea
   ↓
2. Claude hỏi 2-4 câu (lean-prd skill)
   ↓
3. Claude generate Lean PRD
   ↓
4. Product review + feedback
   ↓
5. Claude update PRD
   ↓
6. Lưu PRD lên Confluence (copy/paste hoặc MCP nếu connected)
   ↓
7. (Optional) Claude generate wireframe (prd-to-mockup skill)
   ↓
8. Tạo Jira epic/stories, link Confluence PRD
   ↓
9. Chuyển cho Engineer
```

**Prompt mẫu cho Product:**

```
Viết lean PRD cho feature: [mô tả ngắn feature]

Context:
- User: [ai là user chính]
- Vấn đề: [vấn đề hiện tại]
- Happy path: [luồng chính step by step]
```

Sau khi có PRD:
```
Vẽ wireframe từ PRD vừa viết
```

### Role: Engineer

**Tool:** Claude Code trong IDE hoặc terminal

**Quy trình:**

```
1. Nhận Jira ticket + link Confluence PRD
   ↓
2. Đọc PRD (Claude đọc qua Confluence MCP hoặc paste)
   ↓
3. Claude generate Gherkin feature files (prd-to-gherkin skill)
   ↓
4. Review feature files, adjust nếu cần
   ↓
5. Commit feature files vào living docs repo
   ↓
6. Claude phân tích technical solution (gherkin-to-code Phase 1)
   ↓
7. Engineer review + confirm technical solution
   ↓
8. Claude gen step definitions + implementation code (gherkin-to-code Phase 2-3)
   ↓
9. Claude dùng tech skills từ CLAUDE.md (dotnet-tdd, code-review...) để đảm bảo chất lượng
   ↓
10. Run tests → green
   ↓
11. Product UAT
```

**Prompt mẫu cho Engineer:**

Bước 1 — Gen feature files:
```
Tạo feature file từ PRD:
[paste PRD hoặc link Confluence]

Module: [tên module]
```

Bước 2 — Implement:
```
Implement feature từ features/billing/invoice-submission.feature
```

Hoặc nếu đã connect Confluence MCP:
```
Đọc PRD trên Confluence page [link], tạo feature files rồi implement
```

---

## Cấu trúc Living Docs Repo

```
living-docs/
├── features/                    # Gherkin feature files
│   ├── billing/
│   │   ├── invoice-submission.feature
│   │   └── invoice-cancellation.feature
│   ├── inventory/
│   │   └── stock-transfer.feature
│   └── support/                 # Shared step definitions
│       ├── steps/
│       └── hooks/
├── domain/                      # Domain knowledge cho dự án
│   ├── glossary.md             # Thuật ngữ domain cụ thể
│   ├── personas.md             # Personas cho dự án này
│   └── entities.md             # Entity definitions
├── CLAUDE.md                    # Context cho Claude Code
└── README.md
```

### CLAUDE.md mẫu (living docs repo)

```markdown
# [Tên dự án] — Living Documentation

## Domain
[Mô tả ngắn 2-3 câu về dự án]

## Conventions
- Gherkin steps viết tiếng Việt
- Gherkin syntax (Feature, Rule, Scenario, Given, When, Then) giữ tiếng Anh
- Mọi Scenario PHẢI nằm trong Rule: block
- Cấu trúc: features/[module]/[feature].feature
- Personas: xem domain/personas.md

## References
- PRDs: [link Confluence space]
- Jira: [link Jira project]
```

### CLAUDE.md mẫu (code repo — VD: .NET backend)

```markdown
# [Tên service] — Backend

## Domain
[Mô tả ngắn service này phụ trách domain gì]

## Tech stack
- .NET 8, Clean Architecture, EF Core, PostgreSQL
- Test: xUnit + FluentAssertions + Moq
- Living docs: git submodule tại ./living-docs

## Skills sử dụng
- Khi implement feature từ feature file → dùng **gherkin-to-code** (từ plugin)
- Khi viết unit tests → dùng **dotnet-tdd** (skill riêng, cài trong repo)
- Khi review code → dùng **code-review** (skill riêng, cài trong repo)

## Conventions
- Follow existing patterns trong src/
- Mỗi Rule trong feature file → 1 test class
- Naming: PascalCase cho classes, camelCase cho variables
- Commit message: feat(module): description [BR-Mxx]

## References
- Living docs: ./living-docs/features/
- PRDs: [link Confluence space]
```

> **Lưu ý:** Plugin `ai-first-engineering` chỉ chứa **process skills** (lean-prd, prd-to-gherkin, gherkin-to-code, prd-to-mockup). Các **tech skills** như `dotnet-tdd`, `code-review` là skill riêng, được khai báo trong CLAUDE.md của từng repo. Claude Code đọc CLAUDE.md khi mở repo và tự biết kết hợp cả hai.

### Git Submodule (liên kết code repos)

Mỗi code repo (backend, frontend, mobile) link tới living docs:

```bash
# Trong code repo
git submodule add [living-docs-repo-url] living-docs
git submodule update --init
```

Khi cần sync:
```bash
git submodule update --remote living-docs
```

---

## MCP Connections

Plugin hoạt động tốt hơn khi connect các MCP sau:

| MCP | Dùng cho | Cách connect |
|-----|---------|-------------|
| **Jira** | Tạo/đọc tickets, link PRD với tasks | Settings → Connectors → Jira |
| **Confluence** | Đọc/lưu PRD trực tiếp | Settings → Connectors → Confluence |
| **Figma** (optional) | Đọc design specs | Settings → Connectors → Figma |

Nếu chưa connect MCP, Product có thể copy/paste PRD. Engineer có thể paste PRD text vào prompt.

---

## FAQ

### Q: Dự án đã có feature cũ, bắt đầu từ đâu?
Với brownfield project, không cần viết PRD cho feature cũ. Chỉ viết lean PRD cho feature MỚI hoặc feature đang THAY ĐỔI. Living docs sẽ dần cover theo thời gian.

### Q: Product không biết git, làm sao?
Product không cần git. Product dùng Cowork/Claude.ai để viết PRD → lưu Confluence. Engineer lo phần git, feature files, automation.

### Q: 1 PRD sinh ra bao nhiêu feature files?
1 PRD = 1 Feature file nếu PRD nhỏ. PRD lớn (nhiều module) có thể sinh nhiều feature files, mỗi file cho 1 module.

### Q: Khi nào cần update PRD?
Khi business rules thay đổi. PRD là source of truth cho "tại sao" và "cái gì". Nếu rules thay đổi → update PRD → re-gen feature files.

### Q: Engineer có thể sửa feature file mà không cần Product?
Có — nếu là implementation detail (thêm edge case scenario). Không — nếu là business rule mới (cần Product confirm trước).

### Q: Skill này có xung đột với skill khác không?
Không. Plugin chỉ chứa process skills (cách viết PRD, cách gen Gherkin, cách implement). Tech skills như `code-review`, `dotnet-tdd` là skill riêng — hoạt động song song, bổ sung cho nhau. Khai báo tech skills trong CLAUDE.md của repo để Claude Code tự kết hợp.

### Q: Tech skills (dotnet-tdd, code-review) cài ở đâu?
Tech skills cài trong từng code repo (hoặc user-level nếu dùng cho nhiều repo). Khai báo trong CLAUDE.md để Claude Code biết dùng. Plugin ai-first-engineering KHÔNG chứa tech skills vì mỗi dự án dùng stack khác nhau.

### Q: gherkin-to-code có tự chọn stack không?
Không. Skill chỉ hướng dẫn process (đọc feature file → technical analysis → gen code). Stack và conventions lấy từ CLAUDE.md + existing code trong repo. Claude Code nhìn vào codebase hiện tại để biết viết theo pattern nào.
