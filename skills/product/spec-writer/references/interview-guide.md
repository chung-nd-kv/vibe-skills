# Interview Guide — Additional Question Bank

Use this reference when the standard opening question doesn't surface enough detail. Pick questions relevant to the feature type being specced.

---

## By Feature Type

### CRUD / Data Management Features
*(creating, editing, viewing, deleting records)*

**Data model questions:**
- Dữ liệu này có lifecycle không? (draft → active → archived?)
- Ai có quyền tạo / sửa / xóa? Có permission model không?
- Có bulk operations không? (import CSV, delete nhiều records cùng lúc)
- Data này có relationship với entity nào khác? Khi xóa thì cascade hay restrict?
- Có audit trail / history không? Ai cần xem lịch sử thay đổi?

**Validation questions:**
- Required fields là gì? Validation rule cho từng field?
- Có unique constraint không? Duplicate handling như thế nào?
- Có field nào conditional (hiện khi field khác có giá trị nào đó) không?

---

### Workflow / Process Features
*(approval flows, multi-step processes, state machines)*

**State questions:**
- Các trạng thái của entity này là gì? (e.g., Draft → Submitted → Approved → Rejected)
- Ai có quyền trigger mỗi transition?
- Có timeout / expiry không? Nếu không có action sau X ngày thì sao?
- Có thể revert về state trước không? Trong hoàn cảnh nào?

**Notification questions:**
- Ai cần được notify ở mỗi state change?
- Channel nào? (email, in-app, SMS, webhook)
- Có thể configure notification không, hay hardcoded?

**Parallel flow questions:**
- Có concurrent approvers không? (all must approve vs any one can approve)
- Điều gì xảy ra nếu một approver reject trong multi-approver flow?

---

### Integration / Sync Features
*(connecting to external systems, importing/exporting data)*

**Data flow questions:**
- Direction: push (mình gửi ra), pull (mình lấy về), hay bidirectional?
- Trigger: real-time (event-driven), scheduled (cron), hay manual?
- Volume: bao nhiêu records mỗi lần? Có pagination không?

**Error handling questions:**
- Điều gì xảy ra khi external system unavailable?
- Retry strategy như thế nào? (immediate, exponential backoff, dead-letter queue)
- Partial failure: nếu 50/100 records fail, xử lý 50 success và 50 fail như thế nào?
- Idempotency: có thể chạy lại cùng một sync mà không tạo duplicate không?

**Data mapping questions:**
- Field mapping giữa hai systems như thế nào? Có mismatch/transformation không?
- Conflict resolution: nếu record đã được sửa ở cả hai phía, ai thắng?

---

### Reporting / Analytics Features
*(dashboards, exports, summaries)*

**Scope questions:**
- Ai xem được report này? Có data isolation theo role / tenant không?
- Time range: real-time vs historical? Retention period?
- Aggregation level: individual records hay summary metrics?

**Export questions:**
- Format: PDF, Excel, CSV, API response?
- Có schedule export / email delivery không?
- Large data: nếu report có 100k rows, xử lý như thế nào? (async job, pagination)

**Accuracy questions:**
- Có SLA cho data freshness không? (real-time vs T-1 day)
- Rounding / currency precision rules là gì?

---

### Notification / Alert Features
*(emails, push notifications, in-app alerts)*

**Trigger questions:**
- Event nào trigger notification?
- Có debouncing không? (nhiều events liên tiếp → 1 notification hay nhiều?)
- Điều kiện để suppress notification là gì?

**Content questions:**
- Template có static hay dynamic content?
- Có personalization không? (dùng data từ event để customize nội dung)
- Localization / language support?

**Delivery questions:**
- Priority: transactional (luôn gửi) vs promotional (có thể opt-out)?
- Fallback: nếu email fail thì push, nếu push fail thì SMS?
- Delivery confirmation / read tracking cần không?

---

### Payment / Billing Features
*(checkout, subscription, invoicing)*

**Transaction questions:**
- Flow: one-time hay recurring?
- Có trial period / discount / promo code không?
- Partial payment, installment, hay credit note support cần không?

**Failure handling:**
- Nếu payment fail, retry strategy?
- Grace period trước khi downgrade / lock account?
- Dunning flow: sequence of actions khi recurring payment fail?

**Compliance questions:**
- Region nào? Tax calculation rules (VAT, GST)?
- Invoice legal requirements (e-invoice, digital signature)?
- Refund policy và implementation?

---

## Universal Probing Questions

Dùng khi câu trả lời của user còn chung chung:

| Nếu user nói... | Hỏi thêm... |
|---|---|
| "Users should be able to..." | "Tất cả users hay một role cụ thể?" |
| "The system should handle errors" | "Error nào? Handling behavior cụ thể là gì?" |
| "It should be fast" | "Fast nghĩa là gì? < bao nhiêu ms? Dưới load bao nhiêu?" |
| "We need to notify users" | "Notify khi nào? Qua channel nào? Nội dung là gì?" |
| "It should be secure" | "Threat model là gì? Authentication? Authorization? Encryption?" |
| "Similar to [competitor]" | "Phần nào của [competitor]? Có gì khác với implementation của họ không?" |
| "We'll figure it out later" | "Điều này có ảnh hưởng đến architecture của must-have không?" |

---

## Red Flags — Spec Smells

Nếu gặp những dấu hiệu này, push back trước khi viết:

- **Giải pháp được mô tả như requirement** → "The button should call /api/v2/submit" — hỏi lại behavior expected là gì
- **"All users"** mà không có definition — hỏi rõ role, subscription, permission
- **Success metric chỉ là feature delivery** → "We'll count the feature as successful when it's released" — push for outcome metric
- **Open questions về Must Have requirements** — nếu câu trả lời cho open question thay đổi Must Have, spec chưa ready
- **Scope creep trong "Could Have"** — Could Have quá nhiều = Should Have bị xuống hạng để giữ deadline
