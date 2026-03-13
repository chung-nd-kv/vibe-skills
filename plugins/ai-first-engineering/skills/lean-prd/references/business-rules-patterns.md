# Business Rules Patterns

Thư viện pattern cho business rules phổ biến. Dùng file này khi cần viết rules cho domain SaaS B2B hoặc tương tự.

---

## Cách đọc pattern

Mỗi pattern có:
- **Tên pattern** — nhận diện nhanh loại rule
- **Template** — câu rule dạng fill-in-the-blank
- **Ví dụ tốt** — rule đúng chuẩn, testable, đủ 5 cột
- **Ví dụ xấu** — rule thường gặp nhưng sai, kèm lý do

---

## Pattern 1: Permission (Ai được làm gì)

### Template
> Chỉ [role/condition] mới được [action] [entity]

### Ví dụ tốt
| ID | Rule | Pre-condition | Trigger | Expected | Exception |
|----|------|--------------|---------|----------|-----------|
| BR-M01 | Chỉ merchant có gói trả phí active mới được xuất hóa đơn | merchant.subscription = active_paid | Merchant gửi yêu cầu xuất hóa đơn | Hóa đơn được tạo status "Draft" | Từ chối + "Cần nâng cấp gói dịch vụ" |

### Ví dụ xấu
```
❌ "User phải có quyền phù hợp"
   → "phù hợp" là gì? Không testable.

❌ "Phân quyền theo hệ thống RBAC"
   → Implementation detail, không phải business rule.

❌ "Admin quản lý được tất cả"
   → "tất cả" quá mơ hồ. Liệt kê cụ thể.
```

---

## Pattern 2: Validation (Data phải đúng gì)

### Template
> [Field/Entity] phải thỏa [condition] trước khi [action]

### Ví dụ tốt
| ID | Rule | Pre-condition | Trigger | Expected | Exception |
|----|------|--------------|---------|----------|-----------|
| BR-M02 | Tổng tiền hóa đơn phải bằng tổng các dòng chi tiết | Hóa đơn có ít nhất 1 dòng chi tiết | Hệ thống tính tổng hóa đơn | total = sum(line_items.amount) | Lỗi validation "Tổng tiền không khớp" |
| BR-M03 | Mã số thuế phải đúng format 10 hoặc 13 chữ số | Merchant nhập mã số thuế | Merchant lưu thông tin thuế | MST được lưu thành công | Lỗi "Mã số thuế không đúng định dạng" |

### Ví dụ xấu
```
❌ "Data phải hợp lệ"
   → Hợp lệ nghĩa là gì cụ thể?

❌ "Validate bằng regex pattern ^[0-9]{10,13}$"
   → Regex là implementation. Rule là "10 hoặc 13 chữ số".

❌ "Form phải check required fields"
   → Liệt kê field nào required cụ thể.
```

---

## Pattern 3: State Transition (Chuyển trạng thái)

### Template
> [Entity] chỉ chuyển từ [state A] sang [state B] khi [condition]

### Ví dụ tốt
| ID | Rule | Pre-condition | Trigger | Expected | Exception |
|----|------|--------------|---------|----------|-----------|
| BR-M04 | Hóa đơn chỉ gửi được khi đang ở trạng thái Draft và đã qua validation | invoice.status = Draft AND validation passed | Merchant bấm gửi hóa đơn | invoice.status → Submitted | Giữ nguyên Draft + hiển thị lỗi validation |

### Ví dụ xấu
```
❌ "Hóa đơn phải đi qua đúng flow"
   → Flow nào? Liệt kê transitions cụ thể.

❌ "Update status trong database"
   → Implementation. Rule là điều kiện chuyển state.
```

---

## Pattern 4: Calculation (Công thức tính)

### Template
> [Output] = [formula] từ [inputs], áp dụng [rounding/condition]

### Ví dụ tốt
| ID | Rule | Pre-condition | Trigger | Expected | Exception |
|----|------|--------------|---------|----------|-----------|
| BR-M05 | Thuế VAT = giá trước thuế × thuế suất, làm tròn đến đồng | Dòng chi tiết có giá và thuế suất | Hệ thống tính thuế cho dòng chi tiết | vat_amount = round(price × tax_rate, 0) | Nếu tax_rate = 0 hoặc null → vat_amount = 0 |

### Ví dụ xấu
```
❌ "Tính thuế theo quy định"
   → Quy định nào? Công thức cụ thể?

❌ "Dùng BigDecimal để tránh floating point"
   → Implementation. Rule là cách tính + làm tròn.
```

---

## Pattern 5: Limit / Quota (Giới hạn)

### Template
> [Entity/Action] giới hạn tối đa [N] [unit] trong [scope/period]

### Ví dụ tốt
| ID | Rule | Pre-condition | Trigger | Expected | Exception |
|----|------|--------------|---------|----------|-----------|
| BR-S01 | Merchant gói free tối đa 50 hóa đơn/tháng | merchant.subscription = free | Merchant tạo hóa đơn mới | Hóa đơn được tạo, counter tăng | Khi đạt 50: từ chối + "Đã đạt giới hạn, nâng cấp gói" |

### Ví dụ xấu
```
❌ "Có giới hạn theo gói"
   → Giới hạn gì? Bao nhiêu? Gói nào?

❌ "Rate limit API 100 req/s"
   → Đó là infrastructure rule, không phải business rule.
```

---

## Pattern 6: Notification / Side-effect (Khi X xảy ra, Y phải xảy ra)

### Template
> Khi [event], hệ thống phải [side-effect] cho [recipient]

### Ví dụ tốt
| ID | Rule | Pre-condition | Trigger | Expected | Exception |
|----|------|--------------|---------|----------|-----------|
| BR-S02 | Khi hóa đơn được cơ quan thuế duyệt, gửi email thông báo cho merchant | invoice.status chuyển sang Approved | Cơ quan thuế phản hồi approval | Email gửi đến merchant.email với mã duyệt | Nếu email fail → retry 3 lần, sau đó log lỗi |

### Ví dụ xấu
```
❌ "Thông báo cho user khi có thay đổi"
   → Thay đổi gì? Thông báo kiểu gì? Kênh nào?

❌ "Dùng Firebase Push Notification"
   → Implementation. Rule là khi nào gửi, nội dung gì.
```

---

## Pattern 7: Uniqueness / Conflict (Không được trùng)

### Template
> [Field] phải unique trong [scope]. Khi trùng → [behavior]

### Ví dụ tốt
| ID | Rule | Pre-condition | Trigger | Expected | Exception |
|----|------|--------------|---------|----------|-----------|
| BR-M06 | Số hóa đơn phải unique trong 1 merchant | Merchant tạo hoặc import hóa đơn | Hệ thống check số hóa đơn | Hóa đơn được tạo thành công | Từ chối + "Số hóa đơn [X] đã tồn tại" |

### Ví dụ xấu
```
❌ "Không cho phép duplicate"
   → Duplicate gì? Trong scope nào?

❌ "Thêm unique constraint vào DB"
   → Implementation.
```

---

## Pattern 8: Default / Auto-fill (Tự động điền)

### Template
> Khi [condition], hệ thống tự động điền [field] = [value/source]

### Ví dụ tốt
| ID | Rule | Pre-condition | Trigger | Expected | Exception |
|----|------|--------------|---------|----------|-----------|
| BR-S03 | Khi tạo hóa đơn từ đơn hàng, tự động điền thông tin người mua từ đơn hàng | Đơn hàng có thông tin buyer | Merchant chọn "Xuất hóa đơn" từ đơn hàng | buyer_name, buyer_tax_code, buyer_address được pre-fill | Nếu đơn hàng thiếu info → để trống, merchant tự nhập |

### Ví dụ xấu
```
❌ "Form phải thông minh, tự điền được"
   → "Thông minh" không testable. Cụ thể field nào, nguồn nào.
```

---

## Pattern 9: Time-bound (Thời hạn / Hết hạn)

### Template
> [Entity/Action] có hiệu lực trong [duration]. Khi hết hạn → [behavior]

### Ví dụ tốt
| ID | Rule | Pre-condition | Trigger | Expected | Exception |
|----|------|--------------|---------|----------|-----------|
| BR-S04 | Hóa đơn Draft tự động hủy sau 30 ngày không hoạt động | invoice.status = Draft | 30 ngày kể từ last_modified không có action | invoice.status → Expired | Nếu merchant edit trước 30 ngày → reset timer |

---

## Pattern 10: Cascade / Dependency (Khi thay đổi X, ảnh hưởng Y)

### Template
> Khi [entity A] thay đổi [attribute/state], [entity B] phải [behavior]

### Ví dụ tốt
| ID | Rule | Pre-condition | Trigger | Expected | Exception |
|----|------|--------------|---------|----------|-----------|
| BR-M07 | Khi merchant bị disable, tất cả hóa đơn Draft của merchant đó bị hủy | Merchant có hóa đơn Draft | Admin disable merchant | Tất cả invoice.status Draft → Cancelled | Hóa đơn đã Submitted không bị ảnh hưởng |

---

## Checklist: Rule viết xong rồi, check lại

- [ ] Rule là câu tiếng Việt hoàn chỉnh, không phải keyword
- [ ] Pre-condition mô tả trạng thái ban đầu, không phải action
- [ ] Trigger là 1 hành động cụ thể (verb + object)
- [ ] Expected Outcome là kết quả observable, không phải internal state
- [ ] Exception mô tả cả message user nhìn thấy
- [ ] Không chứa từ mơ hồ: "phù hợp", "hợp lệ", "nhanh", "đúng cách"
- [ ] Không chứa implementation: tên framework, database, API endpoint
- [ ] Có thể viết được ít nhất 2 scenarios từ rule này (happy + error)
