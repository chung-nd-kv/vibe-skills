# Requirements Patterns — Good vs Bad Examples

Dùng reference này để calibrate chất lượng của Must Have / Should Have requirements khi viết spec.

---

## The Core Test

Một requirement tốt phải pass được câu hỏi này:

> **"Tôi có thể ngồi viết một test case pass/fail cho requirement này không, mà không cần hỏi thêm bất kỳ câu hỏi nào?"**

Nếu không → requirement chưa đủ rõ.

---

## Pattern 1 — Access Control

**Template:** "Only [who] can [action] when [condition]"

```
✅ Only users with an active paid subscription can create invoices
✅ Only the invoice owner or an admin can edit a submitted invoice
✅ Guest users can view the product catalog but cannot add items to cart

❌ Access should be properly controlled
❌ The feature should have good security
❌ Only authorized users can use this
```

**BDD signal:** This pattern maps directly to a `Rule:` + persona-based Scenario.

---

## Pattern 2 — Validation

**Template:** "The system must [accept/reject] [input] when [condition], and [response]"

```
✅ The system must reject invoice submission if the total amount is 0 or negative, and display "Invalid amount"
✅ Tax code must be exactly 10 or 13 digits; submission with any other length must be blocked with a format error message
✅ Email field must follow RFC 5321 format; invalid emails must be rejected before form submission

❌ Input should be validated
❌ The form should have proper validation
❌ Bad data shouldn't be accepted
```

**BDD signal:** Each validation case = one Scenario (or Scenario Outline row).

---

## Pattern 3 — State Transition

**Template:** "When [event], the [entity] must transition from [state A] to [state B] and [side effect]"

```
✅ When a merchant submits an invoice, the invoice status must change from "Draft" to "Submitted" and a submission timestamp must be recorded
✅ When payment fails 3 consecutive times, the subscription status must change to "Suspended" and the account owner must receive an email notification
✅ Once an invoice is "Approved", its content must be immutable — edit operations must be rejected with a 403 error

❌ Invoice status should update correctly
❌ The system should handle state changes
❌ Workflow should work properly
```

---

## Pattern 4 — Performance / SLA

**Template:** "The system must [action] within [time limit] under [load condition]"

```
✅ Invoice list must load within 2 seconds for accounts with up to 10,000 invoices
✅ The sync process must complete within 5 minutes for batches of up to 1,000 records
✅ PDF export must be available for download within 30 seconds; if not ready, the system must show a progress indicator

❌ The system should be fast
❌ Performance should be acceptable
❌ Loading time should be reasonable
```

**Note:** If you can't define the number, it's an open question, not a requirement.

---

## Pattern 5 — Notification / Side Effect

**Template:** "When [trigger event], the system must notify [recipient] via [channel] with [content] within [timeframe]"

```
✅ When an invoice is approved by the tax authority, the merchant must receive an in-app notification within 5 minutes
✅ When a sync job fails, the system must send an alert email to the registered admin email within 1 minute of failure
✅ When a user's subscription expires, the system must send a reminder email 7 days, 3 days, and 1 day before expiry

❌ Users should be notified about important events
❌ Email should be sent when things happen
❌ Alerts should work correctly
```

---

## Pattern 6 — Data Integrity

**Template:** "The system must ensure [invariant] — [specific constraint with measurable boundary]"

```
✅ Journal entry debits must equal credits to the cent; any imbalance must block the entry from being posted
✅ Each invoice must have a unique invoice number within a merchant's account; duplicate numbers must be rejected at creation time
✅ Deleted products must be preserved in historical order line items — deletion must only remove the product from the active catalog

❌ Data should be consistent
❌ The system should maintain data integrity
❌ Records should be accurate
```

---

## Pattern 7 — Integration Contract

**Template:** "When [trigger], the system must [call/receive] [external system] with [payload/format] and handle [success/failure] by [behavior]"

```
✅ When an invoice is issued, the system must call the tax authority API within 30 seconds with the invoice data in TT78/2021 XML format; on success, store the tax authority reference code; on failure, mark the invoice "Pending sync" and retry up to 3 times with exponential backoff
✅ The system must accept webhook events from the payment gateway in JSON format; duplicate events (same event ID) must be idempotently ignored
✅ When the accounting sync fails after 3 retries, the system must write the failed event to a dead-letter queue and send an alert to the ops team

❌ The system should integrate with the tax authority
❌ API calls should be handled properly
❌ External systems should work with ours
```

---

## Domain-Specific Patterns

### Accounting / Finance

```
✅ Chart of accounts must follow VAS TT200/2014 account numbering — account codes outside the defined range must be rejected
✅ Multi-currency transactions must record both the original currency amount and the VND equivalent at the exchange rate on the transaction date
✅ Fiscal year closing must prevent new journal entries in closed periods; backdated entries require admin override with an audit log entry
```

### E-Invoice / Tax

```
✅ E-invoices must include the 10-digit seller tax code; invoices with missing or invalid tax codes must be rejected before submission
✅ Invoice cancellation must be reported to the tax authority within 24 hours of cancellation
✅ The system must maintain an immutable audit log of all invoice status changes, accessible to authorized users for 5 years per Circular 78/2021
```

### POS / Retail

```
✅ A sale cannot be finalized if any item's stock quantity would go below 0 unless the product is marked "Allow negative stock"
✅ Discount percentage must be between 0 and 100 inclusive; any other value must be rejected
✅ A voided receipt must be marked "VOID" in all reports — voided transactions must not be included in revenue calculations
```

### Subscription / SaaS

```
✅ Free-tier accounts are limited to 50 invoices per month; attempts to create the 51st invoice must be blocked with an upgrade prompt
✅ Subscription downgrades must take effect at the end of the current billing cycle — no mid-cycle downgrades
✅ When a subscription is cancelled, the account must remain fully functional until the end of the paid period
```

---

## MoSCoW Classification Guide

If unsure which bucket a requirement falls into, use this:

| Question | Answer → classification |
|---|---|
| Without this, is the feature useless for the primary user? | Yes → **Must Have** |
| Without this, does the feature still deliver its core value? | Yes (barely) → **Should Have** |
| Would this delight users but they'd still use it without? | Yes → **Could Have** |
| Is this for a future version, or explicitly deferred? | Yes → **Won't Have** |

**Calibration check:** If more than 50% of requirements are "Must Have", the scope is too large. Push to move some to "Should Have" or split into phases.
