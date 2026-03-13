---
name: prd-to-gherkin
description: Convert PRD / feature spec documents into Gherkin .feature files. Use when the user has a PRD, feature spec, user story, or requirements document (in any format — Anthropic feature-spec, Confluence, Notion, plain text) and wants to generate Gherkin scenarios. Also use when user says "write feature file from PRD", "generate Gherkin from requirements", "convert spec to BDD", "tạo feature file từ PRD", "viết Gherkin từ spec", or pastes a requirements/spec document and asks for test scenarios. Groups scenarios by business Rule using the Gherkin Rule: keyword, following BDD best practices from bdd-practices skill.
---

# PRD to Gherkin Feature File Writer

Convert product requirements (PRDs, feature specs, user stories) into well-structured Gherkin `.feature` files grouped by **business Rule**, following BDD best practices.

## Core Mental Model: PRD → Example Mapping → Feature File

This skill bridges two worlds: **product thinking** (PRD) and **BDD thinking** (Gherkin). The translation follows the Example Mapping structure:

```
PRD Section            →  Gherkin Construct
─────────────────────────────────────────────
Feature Name           →  Feature: <name>
Problem Statement      →  Feature description (narrative)
User Stories (As a...) →  Persona context in Background or step names
MoSCoW Requirement     →  Rule: <business rule>
  Must Have            →  Rule: (generate scenarios first)
  Should Have          →  Rule: (generate after Must Have)
  Could Have           →  Rule: (mark @future tag)
User Story Example     →  Scenario: <happy path>
Constraint / Edge Case →  Scenario: <alternative/error path>
Open Questions         →  # TODO: comment in feature file
```

**The Rule: keyword is the key** — it groups related scenarios under the business rule they validate, making the feature file self-documenting and traceable back to the PRD requirement.

---

## Step 1 — Parse and Understand the PRD

When given a PRD or feature spec, extract:

1. **Feature name** — what is being built (1 Feature per PRD section / epic)
2. **Primary persona(s)** — who the user is (from User Stories or Problem Statement)
3. **Business Rules** — each Must Have / Should Have requirement maps to one Rule
4. **Examples** — specific behaviors or edge cases (from examples, constraints, or implicit logic)
5. **Open questions** — unresolved decisions → add as `# TODO:` comments

If the PRD is missing critical information to write complete scenarios, **ask one targeted question** before proceeding. Don't ask multiple questions at once.

---

## Step 2 — Structure the Feature File

### File naming
Use kebab-case matching the feature name:
```
payment-onboarding.feature
einvoice-submission.feature
kat-journal-entry.feature
```

### Feature file structure (ALWAYS follow this order)

```gherkin
@[domain-tag]
Feature: [Feature Name from PRD]
  As a [primary persona]
  I want [goal from problem statement]
  So that [business value / outcome]

  Background: (optional — only if ALL scenarios share the same setup)
    Given [shared precondition]

  Rule: [Business Rule — extracted from MoSCoW requirement]

    Scenario: [Happy path — what success looks like]
      Given [initial state — who and what context]
      When [action the persona takes]
      Then [observable outcome]

    Scenario: [Alternative / Edge case]
      Given [different initial state]
      When [same or related action]
      Then [different observable outcome]

  Rule: [Next Business Rule]

    Scenario: [...]
      ...
```

---

## Step 3 — Apply BDD Quality Rules

For every scenario generated, enforce these rules (from bdd-practices):

### Cardinal Rules
- **One scenario = one behavior**. If you find yourself writing `When... Then... When... Then...` → split into two scenarios.
- **Declarative over imperative**. Never describe UI actions (click, enter, navigate). Describe business intent.
- **Third person, present tense**. `the user logs in` not `I logged in` or `user will log in`.
- **3–5 steps per scenario** (max 9). If longer, abstract repeated setup to `Background:` or persona steps.

### Rule quality check
Each `Rule:` should be:
- A **business rule statement**, not a technical statement
  - ✅ `Rule: Only active merchants can submit invoices`
  - ❌ `Rule: Check merchant status in database before API call`
- Traceable to a specific MoSCoW requirement in the PRD
- Testable — you can write at least 2 distinct scenarios for it (happy + edge)

### Scenario quality check
Each `Scenario:` title should:
- Complete the sentence: "This system works correctly when..."
  - ✅ `Scenario: Active merchant submits invoice with valid data`
  - ❌ `Scenario: Test submit button`
- Not repeat the Rule name verbatim

---

## Step 4 — Apply Tag Taxonomy

Tags serve **three purposes**: documentation, CI/CD pipeline slicing, and **test layer routing** (which runner executes this scenario).

Every scenario should have exactly **one tag from each dimension**:

```
Scenario = [test layer] + [execution scope] + [priority] + (optional domain)

Example: @api @smoke @regression
         @web @regression @p2
         @integration @smoke
```

---

### Dimension 1 — Test Layer (REQUIRED on every Scenario)

This is the most important tag — it tells the CI runner **which test harness** to use.

| Tag | Layer | When to use | Runner |
|---|---|---|---|
| `@web` | E2E / Browser | User-facing flows that require a browser — login screens, form submissions, UI state changes | Playwright + Cucumber |
| `@api` | API | Business logic verified via HTTP calls — no browser needed; the behavior is the API contract | REST client + Cucumber |
| `@integration` | Integration | Cross-service / cross-system flows — service A triggers something observable in service B | Multi-service test environment |

**Decision guide — which layer?**

Ask: *"Where does the core behavior live?"*

```
Business rule is enforced in the backend (validation, calculation, state machine)
  AND the scenario doesn't require seeing a UI element
  → @api

The user must interact through a browser (UI state, navigation, form rendering)
  OR the acceptance criterion is about what the user SEES on screen
  → @web

The scenario spans two separate services / systems
  (e.g., POS emits an event → accounting system receives it)
  → @integration
```

**Rule: never mix layers in one scenario.** If a scenario needs both browser interaction AND a service call side-effect, split into:
- `@web` scenario: verifies UI behavior
- `@integration` scenario: verifies downstream effect

---

### Dimension 2 — Execution Scope (REQUIRED on every Scenario)

| Tag | When to use |
|---|---|
| `@smoke` | Happy path only — 1–2 per Rule max. Runs on every push. |
| `@regression` | All other scenarios. Runs on PR / nightly. |
| `@wip` | Under active development, expected to fail. Excluded from CI. |
| `@future` | Not yet automated (Could Have). Serves as living documentation. |

---

### Dimension 3 — Priority (maps to PRD MoSCoW)

| PRD Priority | Tag | CI behavior |
|---|---|---|
| Must Have | *(no tag — default)* | Always runs |
| Should Have | `@p2` | Runs in full suite |
| Could Have | `@p3 @future` | Excluded from automation |

---

### Dimension 4 — Domain (on Feature block, optional per team)

Customize these to your product areas. Examples:

| Tag | Domain / Area |
|---|---|
| `@[domain1]` | e.g., `@billing` |
| `@[domain2]` | e.g., `@auth` |
| `@[domain3]` | e.g., `@reporting` |

---

### Full example with layer tags

```gherkin
@billing @einvoice
Feature: Invoice Submission

  Rule: Only active subscribers can issue invoices

    # Business rule lives in API — no browser needed
    @api @smoke
    Scenario: Active subscriber issues an invoice successfully
      Given Minh has an active paid subscription
      And Minh has a completed order
      When Minh calls the invoice submission API
      Then the API returns 201 Created
      And the invoice status is "Submitted"

    @api @regression
    Scenario: Free-tier user is rejected by the API
      Given Lan has a free subscription
      When Lan calls the invoice submission API
      Then the API returns 403 Forbidden
      And the error message is "Subscription upgrade required"

  Rule: Invoice form pre-fills with order data

    # Requires seeing the rendered form — browser needed
    @web @smoke
    Scenario: Invoice form opens pre-filled with order data
      Given Minh has completed order #ORD-001
      When Minh opens the invoice form for the order
      Then the invoice form displays the order's products and amounts
      And the buyer information is pre-filled from the order

  Rule: Issued invoices sync to the accounting system

    # Cross-service: POS issues invoice → accounting receives journal entry
    @integration @smoke
    Scenario: Issued invoice creates a journal entry in accounting
      Given Minh has issued invoice #INV-001
      When the sync process runs
      Then a journal entry appears in the accounting system for #INV-001

    @integration @regression
    Scenario: Failed sync triggers an alert after 3 retries
      Given invoice #INV-001 has been issued
      When the accounting sync fails 3 consecutive times
      Then the system sends an alert to the accountant
      And the invoice is marked "Sync pending"

  Rule: Merchants can preview invoice before issuing

    @web @regression @p2
    Scenario: Merchant edits buyer info in the preview
      Given Minh is viewing the invoice preview for order #ORD-001
      When Minh updates the buyer's tax code in the preview
      Then the updated tax code appears in the preview form

  Rule: Merchant receives SMS on invoice approval

    @api @future @p3
    Scenario: Merchant receives SMS when invoice is approved
      Given Minh has a mobile number registered
      When the tax authority approves invoice #INV-001
      Then Minh receives an SMS notification with the approval reference
```

---

### Tag combinations cheat sheet

| Scenario type | Tags to use |
|---|---|
| Core business logic, backend only | `@api @smoke` or `@api @regression` |
| UI flow, user-visible behavior | `@web @smoke` or `@web @regression` |
| Service A → Service B side effect | `@integration @smoke` or `@integration @regression` |
| Should Have API scenario | `@api @regression @p2` |
| Could Have, not yet automated | `@web @future @p3` (or `@api @future @p3`) |
| Active development, may fail | `@web @wip` |

---

## Step 5 — Handle Scenario Outline (Parametrization)

Use `Scenario Outline` ONLY when:
- Multiple **equivalence classes** exist (not just different data values)
- The behavior changes meaningfully per row
- The table has ≤ 5 rows (more → extract to step definitions or test data files)

```gherkin
# ✅ Good use — different subscription tiers = different behavior
Rule: Access level matches subscription tier

  Scenario Outline: Member accesses content matching their subscription
    Given <user> has a <tier> subscription
    When <user> requests <content_type>
    Then the system <grants_or_denies> access

    Examples:
      | user  | tier    | content_type    | grants_or_denies |
      | Minh  | free    | premium article | denies           |
      | Lan   | basic   | premium article | grants           |
      | Hung  | premium | premium article | grants           |

# ❌ Bad use — same equivalence class, different strings
Scenario Outline: Search returns results
  When the user searches for "<term>"
  Then results related to "<term>" are displayed

  Examples:
    | term     |
    | invoice  |
    | receipt  |   # Same behavior, wastes CI time
    | payment  |   # Same behavior, wastes CI time
```

---

## Output Format

Always output:
1. **One `.feature` file per PRD epic/feature** (not one file per requirement)
2. **Summary table** after the file showing traceability:
3. **Vietnamese language** for all steps, unless the PRD is in English and explicitly states to keep it in English and keep Gherkin syntax in English.

```
## Traceability Matrix

| Rule | PRD Requirement | Scenarios | Tags |
|------|----------------|-----------|------|
| Only active merchants can submit | Must Have #1 | 2 | @smoke @regression |
| Invoice total must match line items | Must Have #2 | 2 | @regression |
| Draft invoices can be edited | Should Have #1 | 3 | @regression @p2 |
```

3. **Open questions block** if PRD has unresolved items:

```
## ⚠️ Open Questions (from PRD)

These items need clarification before scenarios can be written:

- [ ] What happens when merchant submits duplicate invoice number?
      → Needed for: Rule "Invoice number must be unique"
- [ ] Is there a timeout for draft invoice expiry?
      → Needed for: Rule "Draft invoices can be saved"
```

---

## Reference Files

- `references/mapping-guide.md` — Detailed PRD section → Gherkin construct mapping with examples
- `references/domain-glossary.md` — Domain terms for consistent step language across feature files

Read mapping-guide.md when the PRD format is unusual or when you're unsure how to extract Rules from the requirements structure.
