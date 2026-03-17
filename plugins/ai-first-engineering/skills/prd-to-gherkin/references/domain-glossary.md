# Domain Glossary Template

Thuật ngữ nhất quán cho Gherkin step language. Mỗi dự án customize file này trong living docs repo.

---

## Cách dùng

File này có 2 mục đích:
1. **Chuẩn hóa ngôn ngữ** — chọn 1 term cho mỗi concept, dùng everywhere để step definitions tái sử dụng
2. **Định nghĩa personas** — tên personas giúp scenarios dễ đọc và step definitions composable

---

## Domain Terms Template

### [Tên domain 1] — VD: Thanh toán / Hóa đơn
| Concept | Dùng term này | Tránh dùng |
|---------|--------------|-----------|
| [khái niệm] | [term ưu tiên] | [các term khác không dùng] |

### [Tên domain 2] — VD: Xác thực / Phân quyền
| Concept | Dùng term này | Tránh dùng |
|---------|--------------|-----------|
| [khái niệm] | [term ưu tiên] | [các term khác không dùng] |

---

## Persona Library

Personas mặc định cho SaaS B2B context. Customize cho dự án cụ thể.

| Persona | Role | Gói / Quyền | Dùng cho |
|---------|------|-------------|---------|
| Minh | Chủ cửa hàng | Gói trả phí active | Happy path — luồng chính user |
| Lan | Chủ cửa hàng | Gói miễn phí / hết hạn | Lỗi permission, upgrade prompt |
| Hung | Kế toán | Quyền standard | Luồng accounting, báo cáo |
| An | Đại lý / đối tác | Quyền hạn chế | Integration, third-party flows |
| Hoa | Admin / Quản lý | Full access | Multi-tenant, admin config |

> Tip: Dùng tên thực (không phải "User1", "TestUser") — scenarios dễ đọc hơn và stakeholders engage tự nhiên hơn.

---

## Step Phrase Library

Các pattern step tái sử dụng. Customize theo domain dự án.

### Xác thực / Phân quyền
```gherkin
Given Minh đã đăng nhập với vai trò [role]
Given Lan không có quyền truy cập [feature]
Given Minh có gói [tên gói] đang active
Given gói dịch vụ của Minh đã hết hạn
```

### Thiết lập dữ liệu
```gherkin
Given Minh có [N] [items] trong [khu vực hệ thống]
Given [record] #[ID] đang ở trạng thái [status]
Given kỳ [period/context] cho [timeframe] đang mở
Given mã số thuế của Minh đã đăng ký với [hệ thống]
```

### Hành động nghiệp vụ
```gherkin
When Minh [action verb] [object] cho [recipient/context]
When Minh gửi [form/document] đến [destination]
When [hệ thống ngoài] xử lý [object]
When [timeout/scheduled event] xảy ra
```

### Kết quả observable
```gherkin
Then trạng thái [object] chuyển sang "[status]"
Then Minh thấy [message/screen/result]
Then Minh nhận thông báo "[nội dung thông báo]"
Then [object] xuất hiện trong [danh sách/view] của Minh
Then [hệ thống downstream] phản ánh [thay đổi]
```

---

## File Naming Conventions

Dùng kebab-case, tổ chức theo module:

```
features/
├── [module]/
│   ├── [module]-[feature-area].feature
│   └── [module]-[feature-area-2].feature

Ví dụ:
features/
├── billing/
│   ├── invoice-submission.feature
│   ├── invoice-cancellation.feature
│   └── payment-reconciliation.feature
├── inventory/
│   ├── stock-transfer.feature
│   └── stock-count.feature
├── auth/
│   └── subscription-management.feature
└── reporting/
    └── revenue-report.feature
```

---

## Tag Reference

Mỗi scenario mang tags từ **3-4 dimensions**. 1 tag từ mỗi dimension.

### Dimension 1 — Test Layer (bắt buộc)
| Tag | Layer | Runner |
|-----|-------|--------|
| `@web` | E2E / Browser | Playwright + Cucumber |
| `@api` | API / Service | REST client + Cucumber |
| `@integration` | Cross-service | Multi-service environment |

### Dimension 2 — Execution Scope (bắt buộc)
| Tag | Ý nghĩa |
|-----|---------|
| `@smoke` | Happy path — chạy mỗi push |
| `@regression` | Full suite — chạy trên PR / nightly |
| `@wip` | Đang phát triển — excluded khỏi CI |
| `@future` | Chưa automate (Could Have / Won't Have) |

### Dimension 3 — Priority (map từ PRD MoSCoW)
| Tag | PRD Priority |
|-----|-------------|
| *(không tag)* | Must Have — luôn chạy |
| `@p2` | Should Have |
| `@p3 @future` | Could Have / Won't Have |

### Dimension 4 — Domain (trên Feature, customize theo dự án)
| Tag | Domain |
|-----|--------|
| `@[domain]` | [mô tả] |
