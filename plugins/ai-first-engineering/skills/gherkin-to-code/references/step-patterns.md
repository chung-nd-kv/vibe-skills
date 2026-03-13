# Step Definition Patterns

Common patterns cho step definitions theo test layer. Customize theo stack cụ thể trong CLAUDE.md.

---

## Nguyên tắc chung

1. **Reuse trước, tạo mới sau** — scan existing steps, chỉ tạo mới khi chưa có
2. **1 step = 1 việc** — không gộp nhiều actions vào 1 step
3. **Parameterize** — dùng regex/expression capture cho data values
4. **Stateless giữa scenarios** — mỗi scenario bắt đầu từ clean state
5. **Comment Rule ID** — mỗi step group ghi chú Rule liên quan

---

## Pattern theo Test Layer

### @api — API Layer Steps

**Given — Setup test data:**
```
Pattern: Tạo entity với trạng thái cụ thể

Given Minh có gói trả phí đang active
→ Create merchant "Minh" with subscription status = active

Given Minh có đơn hàng đã hoàn thành #ORD-001
→ Create order for merchant "Minh", status = completed, id = ORD-001

Given Lan có gói miễn phí
→ Create merchant "Lan" with subscription status = free
```

**When — API call:**
```
Pattern: HTTP request tới endpoint

When Minh yêu cầu xuất hóa đơn cho đơn hàng
→ POST /api/invoices { merchantId: minh.id, orderId: order.id }
   with auth token of "Minh"

When hệ thống tính tổng hóa đơn
→ GET /api/invoices/{id}/total
   or call service method directly
```

**Then — Assert response + state:**
```
Pattern: Check HTTP response + DB state

Then hóa đơn được tạo với trạng thái "Nháp"
→ Assert response.status = 201
→ Assert DB: invoice.status = "Draft"

Then hệ thống từ chối với thông báo "Cần nâng cấp gói dịch vụ"
→ Assert response.status = 403
→ Assert response.body.message contains "Cần nâng cấp gói dịch vụ"
→ Assert DB: no new invoice created

Then tổng tiền hóa đơn là 450.000đ
→ Assert response.body.total = 450000
```

---

### @web — Browser Layer Steps

**Given — Setup + navigate:**
```
Pattern: Setup data + mở trang

Given Minh đã đăng nhập vào POS
→ Create merchant "Minh" (nếu chưa có)
→ Login as "Minh"
→ Navigate to POS dashboard

Given Minh có đơn hàng #ORD-001 đã hoàn thành
→ Create order in DB (same as @api)
→ Ensure order visible in UI
```

**When — Browser interaction:**
```
Pattern: User action trên UI (declarative, không imperative)

When Minh mở form xuất hóa đơn cho đơn hàng
→ Navigate to invoice form for order ORD-001
   (NOT: click button X, fill field Y — đó là implementation)

When Minh cập nhật mã số thuế người mua trong preview
→ Find tax code field → update value
```

**Then — Assert UI state:**
```
Pattern: Check visible content

Then form hiển thị sản phẩm và số tiền từ đơn hàng
→ Assert: product list visible with correct items
→ Assert: amounts match order data

Then thông tin người mua được tự động điền từ đơn hàng
→ Assert: buyer name field = order.buyerName
→ Assert: buyer tax code field = order.buyerTaxCode
```

---

### @integration — Cross-Service Steps

**Given — Setup across services:**
```
Pattern: Setup state trong nhiều service

Given Minh đã xuất hóa đơn #INV-001
→ Create invoice in billing service, status = "Issued"

Given hệ thống accounting đang hoạt động
→ Health check accounting service
→ Clear test data in accounting
```

**When — Trigger cross-service:**
```
Pattern: Action ở service A, observe ở service B

When quá trình đồng bộ chạy
→ Trigger sync job/event
→ Wait for processing (with timeout)

When accounting sync thất bại 3 lần liên tiếp
→ Simulate 3 sync failures (mock external dependency)
```

**Then — Assert downstream:**
```
Pattern: Check state ở service khác

Then bút toán xuất hiện trong hệ thống kế toán cho #INV-001
→ Query accounting service for invoice INV-001
→ Assert: journal entry exists with correct amounts

Then hệ thống gửi cảnh báo cho kế toán
→ Assert: notification sent to accountant role
→ Assert: notification content references INV-001
```

---

## Shared Patterns (tái sử dụng across layers)

### Persona Setup
```
Given [persona] là merchant [trạng thái] có mã số thuế hợp lệ
→ Parameterized: create merchant by persona name + status
→ Personas map: Minh=active_paid, Lan=free, Hung=accountant, An=agent, Hoa=admin
```

### Error Assertion
```
Then hệ thống từ chối với thông báo "[message]"
→ @api: assert HTTP 4xx + error message
→ @web: assert error toast/banner visible with message
→ Common step, branch by test layer context
```

### Status Check
```
Then trạng thái [entity] chuyển sang "[status]"
→ @api: assert DB state
→ @web: assert UI status badge/label
→ Parameterized: entity type + expected status
```

---

## Anti-patterns

### Imperative steps (SAI)
```
❌ When Minh click vào nút "Xuất hóa đơn"
❌ When Minh nhập mã số thuế vào ô input
❌ When Minh scroll xuống cuối trang rồi bấm Submit

✅ When Minh yêu cầu xuất hóa đơn
✅ When Minh cập nhật mã số thuế
✅ When Minh gửi hóa đơn
```

### Quá nhiều assertions trong 1 Then (SAI)
```
❌ Then hóa đơn được tạo và trạng thái là Nháp và Minh nhận mã và email được gửi

✅ Then hóa đơn được tạo với trạng thái "Nháp"
   And Minh nhận được mã hóa đơn
   And email xác nhận được gửi cho Minh
```

### Step quá specific (SAI)
```
❌ Given merchant ID 12345 có subscription plan_id 67 status active

✅ Given Minh có gói trả phí đang active
   (step definition lo việc tạo đúng data)
```
