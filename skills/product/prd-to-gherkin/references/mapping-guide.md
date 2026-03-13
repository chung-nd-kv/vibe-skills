# PRD → Gherkin Mapping Guide

Detailed translation patterns for converting each section of a feature spec (Anthropic feature-spec format or equivalent) into Gherkin constructs.

---

## 1. Feature: — From Problem Statement

The Feature block captures WHO, WHAT, and WHY from the PRD narrative.

### Mapping
```
PRD: Problem Statement + Primary User
→ Feature: [name]
     As a [who]
     I want [what]
     So that [why / business outcome]
```

### Example
```
PRD Problem Statement:
  "Small business owners currently have no way to submit e-invoices
   directly from the POS system without switching to a separate application.
   This forces them to manually re-enter data, causing errors and delays."

→ Feature file:
@pos @einvoice
Feature: E-Invoice Submission from POS
  As a small business owner using the POS terminal
  I want to submit e-invoices directly from the POS
  So that I can avoid manual data re-entry and reduce submission errors
```

---

## 2. Rule: — From MoSCoW Requirements

Each distinct business rule in the requirements → one `Rule:` block.

### Must Have → Core Rules (no extra tag)
```
PRD Must Have:
  "Merchant must be registered with tax authority before submitting invoices"

→ Rule: Merchants must be registered with the tax authority to submit invoices
```

### Should Have → Secondary Rules (add @p2)
```
PRD Should Have:
  "Merchants should be able to save invoice drafts and return to them later"

→ Rule: Merchants can save invoice drafts for later completion
  # Tag scenarios with @p2
```

### Could Have → Future Rules (add @future @p3)
```
PRD Could Have:
  "Merchants could receive SMS notification when invoice is approved"

→ Rule: Merchants receive notifications on invoice status changes
  # Tag scenarios with @future @p3
  # These can be stubs — Scenario title only, body = pending
```

### When a requirement has multiple behaviors → split into multiple Rules
```
PRD Must Have:
  "The system must validate invoice data including: 
   tax code format, total amount match, and required fields"

→ Rule: Invoice tax code must follow standard format
→ Rule: Invoice total must equal sum of line items  
→ Rule: All mandatory invoice fields must be present
```

---

## 3. Scenario: — From User Stories and Examples

### Happy path — from "As a user, I want..."
```
PRD User Story:
  "As a merchant, I want to submit a completed invoice 
   so that it is sent to the tax authority"

→ @smoke
   Scenario: Merchant successfully submits a completed invoice
     Given Minh is a registered merchant with an active tax code
     And Minh has completed invoice #INV-2024-001 with all required fields
     When Minh submits the invoice
     Then the invoice is sent to the tax authority
     And Minh receives a submission confirmation with reference number
```

### Alternative/Error path — from constraints, edge cases, or implicit logic
```
From the same PRD constraint: "Merchant must be registered"

→ @regression
   Scenario: Unregistered merchant cannot submit invoices
     Given Lan is a merchant without tax authority registration
     When Lan attempts to submit an invoice
     Then the system rejects the submission with message "Tax registration required"
     And the invoice remains in draft status
```

### Edge cases to always consider
For each Rule, ask yourself:
- What if the precondition is NOT met?
- What if the data is invalid/incomplete?
- What if the user has insufficient permissions?
- What if a required external system is unavailable?
- What if the action has already been performed?

---

## 4. Background: — Shared Preconditions

Use Background ONLY when every scenario in the feature (or within a Rule scope) shares the same setup. Overusing Background creates hidden dependencies.

### When to use
```
✅ ALL scenarios need the same merchant context:
Background:
  Given Minh is a registered merchant with an active tax code
  And Minh is logged into the POS system

✅ All scenarios in a Rule share setup:
Rule: Invoice amounts must match

  Background: (note: Gherkin Background is Feature-level, use wisely)
    Given Minh has created invoice #INV-001 with 3 line items
```

### When NOT to use
```
❌ Don't use Background when only some scenarios need it:
Background:
  Given the user is logged in          # Only login scenarios need this
  And the invoice form is open         # Only invoice scenarios need this
  And the merchant database is seeded  # Only specific scenarios need this
```

---

## 5. Persona Naming Convention

Use **named personas** instead of generic "the user" for clarity and consistency.

### Domain Personas
| Persona | Role | Use for |
|---------|------|---------|
| Minh | Active merchant, owner | Happy path business owner flows |
| Lan | New/inactive merchant | Onboarding, permission error flows |
| Hung | Accountant | KAT accounting flows |
| An | Tax agent | TCT/TCTN integration flows |
| Hoa | Admin/Manager | Multi-tenant, admin flows |

### Usage
```gherkin
# ✅ Named persona — clear, reusable step definitions
Given Minh is an active merchant with a valid tax code

# ❌ Generic — harder to understand context
Given the user is logged in and has appropriate permissions
```

---

## 6. Step Language Patterns

### Given — State, not action
```gherkin
# ✅ State (describes a situation that exists)
Given Minh has a completed invoice with 3 line items
Given the system is connected to the tax authority API
Given this is Minh's first invoice submission this month

# ❌ Action (should be When)
Given Minh logs into the system
Given Minh creates an invoice
```

### When — Single user action or system event
```gherkin
# ✅ Single clear action
When Minh submits invoice #INV-2024-001
When the tax authority rejects the invoice
When the submission timeout expires after 30 seconds

# ❌ Multiple actions (split the scenario)
When Minh fills in the invoice form and clicks submit and confirms
```

### Then — Observable outcome (not implementation)
```gherkin
# ✅ Observable outcome from user perspective
Then the invoice status changes to "Submitted"
Then Minh sees confirmation message with reference number
Then the invoice appears in Minh's submitted invoices list

# ❌ Implementation detail
Then the database record is updated with status = 'SUBMITTED'
Then the API returns HTTP 200 with body {"status": "ok"}
```

---

## 7. Handling Complex PRDs

### PRD with multiple epics → multiple feature files
```
PRD: "Payment Onboarding"
  Epic 1: Merchant Registration
  Epic 2: Bank Account Verification  
  Epic 3: First Payment Processing

→ merchant-registration.feature
→ bank-account-verification.feature
→ first-payment-processing.feature
```

### PRD with vague requirements → ask or make explicit
```
PRD: "The system should handle errors gracefully"

→ This is too vague. Either:
   1. Ask: "What specific error scenarios should I cover? e.g., network timeout, invalid data, permission denied?"
   2. Or generate common cases and note them as assumptions:
   
   # Assumption: "handle errors gracefully" covers these scenarios.
   # Verify with PO before automation.
   
   Scenario: Network timeout during invoice submission
   Scenario: Tax authority API returns invalid response
   Scenario: Merchant loses connectivity mid-submission
```

### PRD with technical requirements → translate to behavior
```
PRD Must Have: "The system must use HTTPS for all API calls to tax authority"

→ This is a technical requirement, NOT a user-visible behavior.
   Do NOT write a scenario for this — it belongs in technical specs.
   
   Instead, if there's a related user behavior:
   Scenario: Merchant's invoice data is transmitted securely
     Given Minh submits an invoice
     Then the transmission to the tax authority is encrypted
     # Note: Specific encryption method verified in integration tests
```
