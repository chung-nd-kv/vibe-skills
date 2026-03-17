---
name: prd-to-gherkin
description: Convert Lean PRD thành Gherkin .feature files. Use when engineer có PRD (từ lean-prd skill hoặc Confluence) và cần generate Gherkin scenarios. Trigger khi user nói "tạo feature file từ PRD", "generate Gherkin", "convert spec to BDD", "viết Gherkin từ spec", "gen scenarios", hoặc paste PRD document. Scenarios LUÔN nằm trong Rule: block — không bao giờ có scenario độc lập. Steps viết tiếng Việt, giữ Gherkin syntax tiếng Anh. MANDATORY TRIGGERS: gherkin, feature file, BDD, scenarios, gen gherkin, prd to gherkin.
---

# PRD to Gherkin Feature File Writer

Convert Lean PRD thành Gherkin `.feature` files. Mỗi scenario **bắt buộc** nằm trong `Rule:` block — không có scenario nào được phép đứng độc lập.

## Ngôn ngữ

- **Gherkin syntax** (Feature, Rule, Scenario, Given, When, Then, And, But, Background, Scenario Outline, Examples) → **tiếng Anh**
- **Nội dung steps, Rule description, Feature description** → **tiếng Việt**
- Tag names → tiếng Anh (lowercase, kebab-case)

Ví dụ:
```gherkin
Feature: Xuất hóa đơn điện tử
  As a chủ cửa hàng
  I want xuất hóa đơn trực tiếp từ POS
  So that tránh nhập lại dữ liệu thủ công

  Rule: Chỉ merchant có gói trả phí active mới được xuất hóa đơn

    @api @smoke
    Scenario: Merchant gói trả phí xuất hóa đơn thành công
      Given Minh có gói trả phí đang active
      And Minh có đơn hàng đã hoàn thành
      When Minh yêu cầu xuất hóa đơn cho đơn hàng
      Then hóa đơn được tạo với trạng thái "Nháp"
```

---

## Core Mental Model: PRD → Example Mapping → Feature File

```
PRD Section              →  Gherkin Construct
─────────────────────────────────────────────
Feature Name             →  Feature: <tên>
Bối cảnh                 →  Feature description (narrative)
User Stories (As a...)   →  Persona context trong Background hoặc step names
Business Rule Must Have  →  Rule: <business rule> → Scenarios
Business Rule Should Have →  Rule: <business rule> → Scenarios (tag @p2)
Business Rule Won't Have →  Không generate scenario
Open Questions           →  # TODO: comment trong feature file
```

**Rule: keyword là bắt buộc** — nó nhóm scenarios theo business rule, tạo traceability ngược về PRD.

### Quy tắc tuyệt đối: Scenario PHẢI nằm trong Rule

```gherkin
# ✅ ĐÚNG — scenario nằm trong Rule
Rule: Mã số thuế phải đúng format 10 hoặc 13 chữ số

  Scenario: Merchant nhập mã số thuế 10 chữ số hợp lệ
    ...

  Scenario: Hệ thống từ chối mã số thuế sai format
    ...

# ❌ SAI — scenario không nằm trong Rule, KHÔNG BAO GIỜ LÀM ĐIỀU NÀY
Scenario: Merchant nhập mã số thuế
  ...
```

---

## Step 1 — Parse và hiểu PRD

Khi nhận PRD (từ lean-prd format, Confluence link, hoặc text), extract:

1. **Feature name** — đang build gì (1 Feature per PRD / epic)
2. **Primary persona(s)** — user là ai (từ User Stories hoặc Bối cảnh)
3. **Business Rules** — mỗi Must Have / Should Have → 1 Rule
4. **Examples** — behaviors cụ thể hoặc edge cases (từ examples, constraints, logic ngầm)
5. **Open questions** — decisions chưa resolve → `# TODO:` comments

Nếu PRD thiếu thông tin quan trọng, **hỏi 1 câu targeted** trước khi tiếp. Không hỏi nhiều câu cùng lúc.

---

## Step 2 — Cấu trúc Feature File

### Cấu trúc thư mục (bắt buộc)

```
features/
├── [module-name]/
│   ├── [feature-name].feature
│   ├── [feature-name-2].feature
│   └── ...
├── [module-name-2]/
│   └── ...
└── support/
    └── ...
```

Thứ tự luôn là: **features → module → feature files**

### File naming
Dùng kebab-case, prefix = module name:
```
features/billing/invoice-submission.feature
features/billing/invoice-cancellation.feature
features/inventory/stock-transfer.feature
features/auth/subscription-management.feature
```

### Feature file structure (LUÔN theo thứ tự này)

```gherkin
# PRD: [link Confluence]
# Generated from Lean PRD v[x.x]

@[domain-tag]
Feature: [Tên Feature từ PRD]
  As a [persona chính]
  I want [mục tiêu từ bối cảnh]
  So that [giá trị kinh doanh / kết quả]

  Background: (optional — chỉ khi TẤT CẢ scenarios có cùng setup)
    Given [precondition chung]

  Rule: [Business Rule — từ Must Have requirement]

    @[test-layer] @smoke
    Scenario: [Happy path — thành công trông như thế nào]
      Given [trạng thái ban đầu — ai và context gì]
      When [hành động persona thực hiện]
      Then [kết quả observable]

    @[test-layer] @regression
    Scenario: [Alternative / Edge case]
      Given [trạng thái ban đầu khác]
      When [cùng hoặc liên quan action]
      Then [kết quả observable khác]

  Rule: [Business Rule tiếp theo]

    Scenario: [...]
      ...
```

---

## Step 3 — Áp dụng BDD Quality Rules

### Cardinal Rules
- **1 scenario = 1 behavior**. Nếu viết `When... Then... When... Then...` → tách thành 2 scenarios.
- **Declarative, không imperative**. Không mô tả UI actions (click, nhập, navigate). Mô tả business intent.
- **Ngôi thứ ba, thì hiện tại**. `Minh yêu cầu xuất hóa đơn` không phải `Tôi yêu cầu` hay `Minh sẽ yêu cầu`.
- **3–5 steps mỗi scenario** (max 9). Nếu dài hơn, abstract setup vào `Background:` hoặc persona steps.

### Rule quality check
Mỗi `Rule:` phải:
- Là **business rule statement**, không phải technical statement
  - ✅ `Rule: Chỉ merchant active mới được gửi hóa đơn`
  - ❌ `Rule: Check merchant status trong database trước API call`
- Trace được về 1 requirement cụ thể trong PRD (ghi BR-ID nếu có)
- Testable — viết được ít nhất 2 scenarios (happy + error)

### Scenario quality check
Mỗi `Scenario:` title phải:
- Hoàn thành câu: "Hệ thống hoạt động đúng khi..."
  - ✅ `Scenario: Merchant active gửi hóa đơn với dữ liệu hợp lệ`
  - ❌ `Scenario: Test nút gửi`
- Không lặp lại tên Rule

---

## Step 4 — Áp dụng Tag Taxonomy

Tags phục vụ 3 mục đích: documentation, CI/CD pipeline slicing, và test layer routing.

Mỗi scenario cần tag từ **mỗi dimension**:

```
Scenario = [test layer] + [execution scope] + [priority nếu không phải Must Have]

Ví dụ: @api @smoke
       @web @regression @p2
       @integration @smoke
```

### Dimension 1 — Test Layer (REQUIRED trên mỗi Scenario)

| Tag | Layer | Khi nào dùng |
|---|---|---|
| `@web` | E2E / Browser | Flow user-facing cần browser — login, form, UI state |
| `@api` | API | Business logic verify qua HTTP — không cần browser |
| `@integration` | Integration | Cross-service — service A trigger observable ở service B |

**Quyết định nhanh — layer nào?**

```
Business rule enforce ở backend (validation, calculation, state machine)
  VÀ scenario không cần nhìn UI element
  → @api

User phải tương tác qua browser (UI state, navigation, form)
  HOẶC acceptance criterion là về user NHÌN THẤY gì
  → @web

Scenario span 2 service/system riêng biệt
  → @integration
```

### Dimension 2 — Execution Scope (REQUIRED)

| Tag | Khi nào dùng |
|---|---|
| `@smoke` | Happy path only — 1–2 per Rule max. Chạy mỗi push. |
| `@regression` | Tất cả scenarios còn lại. Chạy trên PR / nightly. |
| `@wip` | Đang develop, expected to fail. Excluded khỏi CI. |
| `@future` | Chưa automate (Could Have / Won't Have). Living documentation. |

### Dimension 3 — Priority (map từ PRD MoSCoW)

| PRD Priority | Tag |
|---|---|
| Must Have | *(không tag — default)* |
| Should Have | `@p2` |
| Could Have / Won't Have | `@p3 @future` |

### Dimension 4 — Domain (trên Feature block)

Customize theo product area. Ví dụ: `@billing`, `@inventory`, `@auth`, `@reporting`

---

## Step 5 — Handle Scenario Outline

Dùng `Scenario Outline` CHỈ KHI:
- Có nhiều **equivalence classes** (không chỉ data values khác nhau)
- Behavior thay đổi có nghĩa theo mỗi row
- Table ≤ 5 rows

```gherkin
# ✅ Dùng tốt — tier khác nhau = behavior khác nhau
Rule: Quyền truy cập phụ thuộc vào gói dịch vụ

  Scenario Outline: Thành viên truy cập nội dung theo gói
    Given <user> có gói <tier>
    When <user> yêu cầu truy cập <loại_nội_dung>
    Then hệ thống <cho_phép_hoặc_từ_chối> truy cập

    Examples:
      | user | tier      | loại_nội_dung     | cho_phép_hoặc_từ_chối |
      | Minh | trả phí   | báo cáo premium   | cho phép               |
      | Lan  | miễn phí  | báo cáo premium   | từ chối                |
      | Hung | premium   | báo cáo premium   | cho phép               |
```

---

## Output Format

Luôn output:

1. **Feature files** — theo cấu trúc `features/[module]/[feature].feature`
2. **Traceability Matrix** sau feature file:

```
## Traceability Matrix

| Rule | PRD Requirement (BR-ID) | Scenarios | Tags |
|------|------------------------|-----------|------|
| Chỉ merchant active mới gửi được | BR-M01 Must Have | 2 | @api @smoke @regression |
| Tổng hóa đơn phải khớp dòng chi tiết | BR-M02 Must Have | 2 | @api @regression |
| Merchant có thể lưu nháp | BR-S01 Should Have | 3 | @web @regression @p2 |
```

3. **Open questions block** nếu PRD có unresolved items:

```
## Open Questions (từ PRD)

Cần clarify trước khi viết scenario:

- [ ] Khi merchant gửi số hóa đơn trùng thì xử lý thế nào?
      → Cần cho: Rule "Số hóa đơn phải unique"
- [ ] Hóa đơn nháp có timeout tự hủy không?
      → Cần cho: Rule "Hóa đơn nháp có thể lưu"
```

---

## Full Example

```gherkin
# PRD: https://your-site.atlassian.net/wiki/spaces/PROJ/pages/123456
# Generated from Lean PRD v1.0

@billing @einvoice
Feature: Xuất hóa đơn điện tử
  As a chủ cửa hàng sử dụng POS
  I want xuất hóa đơn điện tử trực tiếp từ POS
  So that tránh nhập lại dữ liệu thủ công và giảm lỗi

  Background:
    Given Minh là merchant đã đăng ký với cơ quan thuế

  Rule: Chỉ merchant có gói trả phí active mới được xuất hóa đơn
    # BR-M01

    @api @smoke
    Scenario: Merchant gói trả phí xuất hóa đơn thành công
      Given Minh có gói trả phí đang active
      And Minh có đơn hàng đã hoàn thành #ORD-001
      When Minh yêu cầu xuất hóa đơn cho đơn hàng
      Then hóa đơn được tạo với trạng thái "Nháp"
      And Minh nhận được mã hóa đơn

    @api @regression
    Scenario: Merchant gói miễn phí bị từ chối xuất hóa đơn
      Given Lan có gói miễn phí
      When Lan yêu cầu xuất hóa đơn
      Then hệ thống từ chối với thông báo "Cần nâng cấp gói dịch vụ"
      And hóa đơn không được tạo

  Rule: Tổng tiền hóa đơn phải bằng tổng các dòng chi tiết
    # BR-M02

    @api @regression
    Scenario: Hệ thống tính đúng tổng tiền hóa đơn
      Given Minh có hóa đơn với 3 dòng chi tiết
      And dòng 1 là 100.000đ, dòng 2 là 200.000đ, dòng 3 là 150.000đ
      When hệ thống tính tổng hóa đơn
      Then tổng tiền hóa đơn là 450.000đ

    @api @regression
    Scenario: Hệ thống phát hiện tổng tiền không khớp
      Given Minh có hóa đơn với tổng tiền đã sửa tay
      When hệ thống kiểm tra tổng tiền
      Then hệ thống báo lỗi "Tổng tiền không khớp với chi tiết"

  Rule: Form hóa đơn tự động điền từ đơn hàng
    # BR-S03

    @web @smoke @p2
    Scenario: Form hóa đơn hiển thị dữ liệu từ đơn hàng
      Given Minh có đơn hàng #ORD-001 đã hoàn thành
      When Minh mở form xuất hóa đơn cho đơn hàng
      Then form hiển thị sản phẩm và số tiền từ đơn hàng
      And thông tin người mua được tự động điền từ đơn hàng

  Rule: Merchant nhận thông báo khi hóa đơn được duyệt
    # BR-S02

    @api @future @p3
    Scenario: Merchant nhận SMS khi hóa đơn được cơ quan thuế duyệt
      Given Minh có số điện thoại đã đăng ký
      When cơ quan thuế duyệt hóa đơn #INV-001
      Then Minh nhận SMS thông báo với mã duyệt
```

---

## Reference Files

- `references/mapping-guide.md` — Chi tiết mapping PRD section → Gherkin construct với ví dụ
- `references/domain-glossary.md` — Template thuật ngữ domain cho step language nhất quán

Đọc mapping-guide.md khi PRD format khác lạ hoặc không chắc cách extract Rules từ requirements.
