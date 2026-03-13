# PRD → Gherkin Mapping Guide

Chi tiết cách chuyển mỗi section của Lean PRD thành Gherkin constructs.

---

## 1. Feature: — Từ Bối cảnh + User Stories

Feature block lấy AI, CÁI GÌ, và TẠI SAO từ PRD.

### Mapping
```
PRD: Bối cảnh + User Stories
→ Feature: [tên]
     As a [ai]
     I want [cái gì]
     So that [tại sao / giá trị kinh doanh]
```

### Ví dụ
```
Bối cảnh PRD:
  "Chủ cửa hàng nhỏ hiện không có cách xuất hóa đơn điện tử
   trực tiếp từ POS mà phải chuyển sang ứng dụng khác.
   Điều này buộc họ nhập lại dữ liệu, gây sai sót và chậm trễ."

→ Feature file:
@pos @einvoice
Feature: Xuất hóa đơn điện tử từ POS
  As a chủ cửa hàng sử dụng POS
  I want xuất hóa đơn trực tiếp từ POS
  So that tránh nhập lại dữ liệu thủ công và giảm sai sót
```

---

## 2. Rule: — Từ Business Rules (MoSCoW)

Mỗi business rule riêng biệt trong PRD → 1 `Rule:` block.

**Quy tắc tuyệt đối: Mọi Scenario PHẢI nằm trong Rule:**

### Must Have → Core Rules (không cần tag thêm)
```
PRD Must Have:
  BR-M01: Chỉ merchant có gói trả phí active mới được xuất hóa đơn

→ Rule: Chỉ merchant có gói trả phí active mới được xuất hóa đơn
  # BR-M01
```

### Should Have → Secondary Rules (tag @p2)
```
PRD Should Have:
  BR-S01: Merchant có thể lưu hóa đơn nháp và quay lại sau

→ Rule: Merchant có thể lưu hóa đơn nháp để hoàn thành sau
  # BR-S01 — tag scenarios với @p2
```

### Could Have / Won't Have → Future Rules (tag @future @p3)
```
PRD Won't Have:
  Merchant nhận SMS khi hóa đơn được duyệt

→ Rule: Merchant nhận thông báo khi hóa đơn được duyệt
  # Tag scenarios với @future @p3
  # Có thể chỉ ghi title, body = pending
```

### Khi requirement có nhiều behaviors → tách thành nhiều Rules
```
PRD Must Have:
  BR-M02: Hệ thống phải validate dữ liệu hóa đơn gồm:
   format mã số thuế, tổng tiền khớp, và required fields

→ Rule: Mã số thuế phải đúng format 10 hoặc 13 chữ số
→ Rule: Tổng tiền hóa đơn phải bằng tổng các dòng chi tiết
→ Rule: Tất cả trường bắt buộc của hóa đơn phải có giá trị
```

---

## 3. Scenario: — Từ Pre-condition / Trigger / Expected / Exception

### Happy path — từ Expected Outcome
```
PRD Business Rule:
  BR-M01 | Pre: merchant.subscription = active_paid
         | Trigger: merchant gửi yêu cầu xuất hóa đơn
         | Expected: hóa đơn được tạo, status = "Nháp"

→ @smoke
  Scenario: Merchant gói trả phí xuất hóa đơn thành công
    Given Minh có gói trả phí đang active
    And Minh có đơn hàng đã hoàn thành
    When Minh yêu cầu xuất hóa đơn cho đơn hàng
    Then hóa đơn được tạo với trạng thái "Nháp"
    And Minh nhận được mã hóa đơn
```

### Error path — từ Exception
```
PRD Business Rule:
  BR-M01 | Exception: từ chối + "Cần nâng cấp gói dịch vụ"

→ @regression
  Scenario: Merchant gói miễn phí bị từ chối xuất hóa đơn
    Given Lan có gói miễn phí
    When Lan yêu cầu xuất hóa đơn
    Then hệ thống từ chối với thông báo "Cần nâng cấp gói dịch vụ"
    And hóa đơn không được tạo
```

### Edge cases cần xem xét cho mỗi Rule
- Precondition KHÔNG thỏa mãn?
- Data invalid / thiếu?
- User không đủ quyền?
- Hệ thống ngoài không available?
- Action đã thực hiện rồi (duplicate)?
- Concurrent access (2 người cùng làm)?

---

## 4. Background: — Precondition chung

Dùng Background CHỈ KHI mọi scenario trong Feature (hoặc trong scope Rule) có cùng setup.

```gherkin
# ✅ TẤT CẢ scenarios cần cùng merchant context:
Background:
  Given Minh là merchant đã đăng ký với cơ quan thuế
  And Minh đã đăng nhập vào POS

# ❌ Chỉ một số scenarios cần — KHÔNG dùng Background:
Background:
  Given user đã đăng nhập            # Chỉ login scenarios cần
  And form hóa đơn đã mở             # Chỉ invoice scenarios cần
```

---

## 5. Persona Naming Convention

Dùng **tên personas** thay vì generic "user" cho rõ ràng và nhất quán.

| Persona | Role | Dùng cho |
|---------|------|---------|
| Minh | Chủ cửa hàng active | Happy path — luồng chính |
| Lan | Merchant mới / inactive | Lỗi permission, upgrade prompt |
| Hung | Kế toán | Luồng accounting, báo cáo |
| An | Đại lý / đối tác ngoài | Integration, third-party flows |
| Hoa | Admin / Quản lý | Multi-tenant, admin flows |

> Đây là personas mặc định. Mỗi dự án có thể customize trong domain glossary riêng.

```gherkin
# ✅ Persona có tên — rõ ràng, step definitions tái sử dụng
Given Minh là merchant active có mã số thuế hợp lệ

# ❌ Generic — khó hiểu context
Given user đã đăng nhập và có quyền phù hợp
```

---

## 6. Step Language Patterns

### Given — Trạng thái, không phải hành động
```gherkin
# ✅ Trạng thái (mô tả tình huống đã tồn tại)
Given Minh có hóa đơn hoàn thành với 3 dòng chi tiết
Given hệ thống đã kết nối với API cơ quan thuế
Given đây là lần xuất hóa đơn đầu tiên trong tháng của Minh

# ❌ Hành động (nên là When)
Given Minh đăng nhập vào hệ thống
Given Minh tạo một hóa đơn
```

### When — 1 hành động duy nhất của user hoặc system event
```gherkin
# ✅ Hành động rõ ràng
When Minh gửi hóa đơn #INV-2024-001
When cơ quan thuế từ chối hóa đơn
When thời gian timeout 30 giây hết hạn

# ❌ Nhiều hành động (tách scenario)
When Minh điền form hóa đơn rồi bấm gửi rồi xác nhận
```

### Then — Kết quả observable (không phải implementation)
```gherkin
# ✅ Kết quả observable từ góc nhìn user
Then trạng thái hóa đơn chuyển sang "Đã gửi"
Then Minh thấy thông báo xác nhận với mã tham chiếu
Then hóa đơn xuất hiện trong danh sách hóa đơn đã gửi

# ❌ Implementation detail
Then database record cập nhật status = 'SUBMITTED'
Then API trả về HTTP 200 với body {"status": "ok"}
```

---

## 7. Handling PRD phức tạp

### PRD nhiều module → nhiều feature files
```
PRD: "Quản lý thanh toán"
  Module: Đăng ký merchant
  Module: Xác minh tài khoản ngân hàng
  Module: Xử lý thanh toán đầu tiên

→ features/payment/merchant-registration.feature
→ features/payment/bank-account-verification.feature
→ features/payment/first-payment-processing.feature
```

### PRD requirement mơ hồ → hỏi hoặc ghi rõ assumption
```
PRD: "Hệ thống phải xử lý lỗi một cách hợp lý"

→ Quá mơ hồ. Hỏi: "Cụ thể scenario lỗi nào cần cover?
   VD: timeout mạng, data không hợp lệ, không có quyền?"

   Hoặc generate common cases và ghi assumption:

   # Giả định: "xử lý lỗi hợp lý" bao gồm các scenario sau.
   # Verify với PM trước khi automate.
```

### PRD requirement kỹ thuật → translate sang behavior
```
PRD: "Hệ thống phải dùng HTTPS cho tất cả API calls"

→ Đây là technical requirement, KHÔNG phải behavior.
  Không viết scenario cho nó.
  Nếu có related user behavior thì viết cho behavior đó.
```
