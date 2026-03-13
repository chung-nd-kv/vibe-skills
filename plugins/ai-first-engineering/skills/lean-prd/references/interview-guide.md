# Hướng dẫn phỏng vấn theo loại Feature

Dùng file này khi cần hỏi sâu hơn cho feature phức tạp. Chọn loại feature phù hợp rồi hỏi theo bộ câu hỏi tương ứng.

---

## Loại 1: CRUD (Tạo / Đọc / Sửa / Xóa)

Feature cho phép user quản lý data — tạo record, xem danh sách, sửa, xóa.

### Câu hỏi chính
| # | Câu hỏi | Mục đích |
|---|---------|---------|
| 1 | Entity chính là gì? Có những thuộc tính nào bắt buộc? | Domain Context |
| 2 | Ai được tạo/sửa/xóa? Có phân quyền theo role không? | Business Rules (permission) |
| 3 | Có validation nào khi tạo/sửa? VD: format, unique, range? | Business Rules (validation) |
| 4 | Xóa là soft delete hay hard delete? Có thể khôi phục không? | Workflow (state machine) |
| 5 | Có giới hạn số lượng record không? VD: max 100 sản phẩm/gói free | Business Rules (limit) |

### Business Rules thường gặp
- Permission rule: "Chỉ [role] mới được [action]"
- Validation rule: "[field] phải thỏa [condition]"
- Uniqueness rule: "[field] không được trùng trong [scope]"
- Cascade rule: "Khi xóa [parent], [child] phải [behavior]"
- Default rule: "Khi tạo mới, [field] mặc định là [value]"

---

## Loại 2: Workflow / State Machine

Feature có luồng xử lý nhiều bước — entity chuyển qua nhiều trạng thái.

### Câu hỏi chính
| # | Câu hỏi | Mục đích |
|---|---------|---------|
| 1 | Entity chính đi qua những trạng thái nào? Vẽ sơ bộ được không? | Workflow |
| 2 | Ai/điều kiện gì trigger chuyển trạng thái? | Business Rules (guards) |
| 3 | Có chuyển ngược (rollback) được không? VD: Submitted → Draft? | Workflow (reverse transitions) |
| 4 | Khi nào thì "kẹt" ở 1 state? Có timeout không? | Business Rules (timeout/escalation) |
| 5 | Có parallel paths không? VD: vừa chờ duyệt vừa chờ thanh toán? | Workflow (fork/join) |

### Business Rules thường gặp
- Guard rule: "Chỉ chuyển từ [state A] sang [state B] khi [condition]"
- Timeout rule: "Nếu ở [state] quá [duration], tự động [action]"
- Approval rule: "[role] phải duyệt trước khi chuyển sang [state]"
- Cancellation rule: "Chỉ hủy được khi đang ở [states], không hủy được khi [states]"
- Notification rule: "Khi chuyển sang [state], thông báo cho [role]"

---

## Loại 3: Integration (Tích hợp hệ thống ngoài)

Feature kết nối với service/API bên ngoài — đồng bộ data, gửi/nhận.

### Câu hỏi chính
| # | Câu hỏi | Mục đích |
|---|---------|---------|
| 1 | Tích hợp với hệ thống nào? Data flow chiều nào (push/pull/bi-directional)? | Domain Context |
| 2 | Khi hệ thống ngoài không phản hồi (timeout, error), xử lý thế nào? | Business Rules (error handling) |
| 3 | Có retry không? Bao nhiêu lần? Khoảng cách bao lâu? | Business Rules (retry policy) |
| 4 | Data mapping như thế nào? Field nào bên mình map sang field nào bên họ? | Domain Context (mapping) |
| 5 | Có conflict resolution không? VD: cả 2 bên cùng sửa 1 record? | Business Rules (conflict) |

### Business Rules thường gặp
- Sync rule: "Data [entity] phải đồng bộ sang [system] trong [timeframe]"
- Retry rule: "Khi [system] trả lỗi, retry [N] lần, cách nhau [duration]"
- Fallback rule: "Khi [system] không khả dụng, [alternative behavior]"
- Mapping rule: "[field A] bên mình = [field B] bên họ"
- Idempotency rule: "Gửi lại request trùng không tạo duplicate"

---

## Loại 4: Notification / Communication

Feature gửi thông báo — email, SMS, push, in-app.

### Câu hỏi chính
| # | Câu hỏi | Mục đích |
|---|---------|---------|
| 1 | Kênh gửi nào? (email, SMS, push, in-app) Có fallback kênh không? | Domain Context |
| 2 | Trigger gửi là gì? Event nào kích hoạt? | Business Rules (trigger) |
| 3 | User có tắt được thông báo không? Granularity tới đâu? | Business Rules (preference) |
| 4 | Nội dung dynamic hay template? Có localization không? | Screen States |
| 5 | Có rate limiting không? VD: max 5 SMS/ngày? | Business Rules (limit) |

### Business Rules thường gặp
- Trigger rule: "Khi [event], gửi [notification type] cho [recipient]"
- Preference rule: "User có thể tắt [notification type] trong settings"
- Rate limit rule: "Không gửi quá [N] [type] cho 1 user trong [period]"
- Delivery rule: "Nếu [channel] fail, fallback sang [channel]"
- Content rule: "Notification phải chứa [required fields]"

---

## Loại 5: Calculation / Report

Feature tính toán, tổng hợp, xuất báo cáo.

### Câu hỏi chính
| # | Câu hỏi | Mục đích |
|---|---------|---------|
| 1 | Công thức tính cụ thể? Input, output, các bước trung gian? | Business Rules (formula) |
| 2 | Làm tròn thế nào? Bao nhiêu chữ số thập phân? | Business Rules (precision) |
| 3 | Kỳ tính (period) là gì? Ngày/tuần/tháng/năm? Timezone? | Domain Context |
| 4 | Khi data thiếu hoặc bất thường, tính thế nào? | Business Rules (edge cases) |
| 5 | Ai xem được báo cáo nào? Có filter/export không? | Business Rules (permission) |

### Business Rules thường gặp
- Formula rule: "[Output] = [formula] từ [inputs]"
- Rounding rule: "[Value] làm tròn [method] đến [precision]"
- Period rule: "Báo cáo tính theo [period], cutoff lúc [time]"
- Missing data rule: "Khi [field] trống, dùng [default/skip/error]"
- Permission rule: "[Role] chỉ xem được data của [scope]"

---

## Mẹo chung

1. **Luôn hỏi edge case:** "Trường hợp nào hệ thống phải từ chối?" — câu này khai thác exception rules hiệu quả nhất
2. **Hỏi về concurrency:** "Nếu 2 người cùng làm action này cùng lúc?" — nhiều feature quên case này
3. **Hỏi về data lớn:** "Nếu có 10,000 records thì sao?" — phát hiện pagination/performance rules
4. **Đừng hỏi về UI:** "Nút ở đâu, màu gì" không phải việc của PRD — để designer/mockup skill lo
