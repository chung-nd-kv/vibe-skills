# Domain Glossary

Consistent terminology for Gherkin step language. Fill in or replace sections with your product's domain terms to ensure step definitions are reusable across feature files.

---

## How to Use This Glossary

This file has two purposes:
1. **Standardize language** — pick one term per concept and use it everywhere so step definitions are reusable
2. **Define personas** — named test personas make scenarios readable and step definitions composable

Customize the sections below to match your product's domain.

---

## Domain Terms Template

### [Your Product Area 1] — e.g., Billing / Invoicing
| Concept | Use This Term | Avoid |
|---------|--------------|-------|
| [domain concept] | [preferred term] | [alternative terms to avoid] |
| [domain concept] | [preferred term] | [alternative terms to avoid] |

### [Your Product Area 2] — e.g., Authentication / Access
| Concept | Use This Term | Avoid |
|---------|--------------|-------|
| [domain concept] | [preferred term] | [alternative terms to avoid] |

---

## Persona Library

Named personas make scenarios readable and step definitions reusable. Define 4–6 personas covering your main roles and edge cases.

### Persona Template

| Persona | Role | Subscription / Permission | Use For |
|---------|------|--------------------------|---------|
| [Name] | [Role] | [Plan/Level] | Happy path flows |
| [Name] | [Role] | [Plan/Level] | Permission error flows |
| [Name] | [Role] | [Plan/Level] | Admin / management flows |
| [Name] | [Role] | [Plan/Level] | Integration / external flows |

### Example Persona Set (SaaS B2B context)

| Persona | Role | Subscription / Permission | Use For |
|---------|------|--------------------------|---------|
| Minh | Business owner | Active paid plan | Happy path — primary user flows |
| Lan | Business owner | Free or expired plan | Permission denied, upgrade prompt flows |
| Hung | Accountant / back-office | Standard access | Accounting, reporting flows |
| An | External agent / partner | Limited access | Third-party integration flows |
| Hoa | Admin / Manager | Full access | Multi-tenant, admin configuration flows |

> Tip: Use real-sounding names (not "User1", "TestUser") — scenarios become more readable and stakeholders engage more naturally.

---

## Step Phrase Library

Reusable step patterns for common scenarios. Adapt to your domain.

### Authentication / Authorization
```gherkin
Given Minh is logged in as [role]
Given Lan does not have permission to access [feature]
Given Minh has an active [plan name] subscription
Given Minh's subscription has expired
```

### Data State Setup
```gherkin
Given Minh has [N] [items] in [system area]
Given [record] #[ID] is in [status] status
Given the [period/context] for [timeframe] is open
Given Minh's [identifier] is registered with [system]
```

### Business Actions
```gherkin
When Minh [action verb] [object] for [recipient/context]
When Minh submits [form/document] to [destination]
When [external system] processes the [object]
When the [timeout/scheduled event] occurs
```

### Observable Outcomes
```gherkin
Then the [object] status changes to "[status]"
Then Minh sees [message/screen/result]
Then Minh receives notification "[message text]"
Then the [object] appears in Minh's [list/view]
Then [downstream system] reflects the [change]
```

---

## File Naming Conventions

Use kebab-case with domain prefix if your product has multiple areas:

```
[domain]-[feature-area]-[sub-feature].feature

Examples (generic):
  billing-invoice-submission.feature
  billing-invoice-cancellation.feature
  auth-subscription-management.feature
  reporting-export-data.feature
  integrations-webhook-delivery.feature
```

---

## Tag Reference

Every scenario carries tags from **4 dimensions**. Apply one from each relevant dimension.

### Dimension 1 — Test Layer (required)
| Tag | Layer | Runner |
|-----|-------|--------|
| `@web` | E2E / Browser | Playwright + Cucumber |
| `@api` | API / Service | REST client + Cucumber |
| `@integration` | Cross-service | Multi-service environment |

### Dimension 2 — Execution Scope (required)
| Tag | Meaning |
|-----|---------|
| `@smoke` | Happy path — runs on every push |
| `@regression` | Full suite — runs on PR / nightly |
| `@wip` | Work in progress — excluded from CI |
| `@future` | Not yet automated (Could Have) |

### Dimension 3 — Priority (matches PRD MoSCoW)
| Tag | PRD Priority |
|-----|-------------|
| *(none)* | Must Have — always runs |
| `@p2` | Should Have |
| `@p3 @future` | Could Have |

### Dimension 4 — Domain (on Feature, customize per product)
| Tag | Domain / Area |
|-----|--------------|
| `@[domain1]` | [description] |
| `@[domain2]` | [description] |
