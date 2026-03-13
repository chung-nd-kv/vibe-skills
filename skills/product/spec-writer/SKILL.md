---
name: spec-writer
description: Write product feature specs and PRDs through structured interview then generate a complete document. Use when user wants to write a PRD, feature spec, product requirements document, or says "write a spec for", "tạo PRD cho", "viết feature spec", "help me spec out", "I need to document this feature", "draft requirements for", or describes a feature idea and needs it structured. Conducts a focused interview first, then produces a BDD-ready spec following the Anthropic feature-spec format with Problem Statement, User Stories, MoSCoW Requirements, and Success Metrics.
---

# Spec Writer

Write high-quality feature specs through structured conversation, then produce a complete PRD document ready for engineering handoff and BDD scenario generation.

## Core Philosophy

A good spec answers three questions before anything else:
- **Who** has the problem? (not the persona using the software — the human experiencing pain)
- **What** pain do they have right now, without this feature?
- **Why** does solving it matter to the business?

Everything else — requirements, metrics, scope — flows from clear answers to these three questions. If the "who/what/why" is fuzzy, the spec will be fuzzy too.

The output format is deliberately structured for **two audiences**:
1. Engineering — to know what to build and what's out of scope
2. QA / BDD — to extract business rules and generate Gherkin scenarios (via `prd-to-gherkin` skill)

---

## Workflow: Interview → Draft → Refine

### Phase 1 — Scoping Interview (always start here)

Never jump straight to writing. Conduct a brief interview first.

Start with one opening question, then follow up based on answers. Do not dump all questions at once — feel like a conversation, not a form.

**Opening question:**
> "Trước khi viết spec, mình cần hiểu rõ vấn đề. Bạn có thể mô tả: người dùng nào đang gặp vấn đề gì, và hiện tại họ đang xử lý như thế nào?"

*(Adapt language to match user — Vietnamese or English based on how they write)*

**Follow-up questions** (ask only what's still missing after the opening answer):

| If missing | Ask |
|---|---|
| Who exactly | "Đây là loại user nào? Role, subscription level, use case cụ thể?" |
| Current pain | "Hiện tại họ giải quyết vấn đề này bằng cách nào? Pain point cụ thể là gì?" |
| Business value | "Tại sao feature này quan trọng với business? KPI nào bị ảnh hưởng?" |
| Scope clarity | "Có gì chắc chắn **không** nằm trong scope lần này không?" |
| Success definition | "Làm sao biết feature này thành công? Metric nào thay đổi?" |

**Stop asking when** you have: a clear user, a concrete pain point, a measurable outcome, and a rough sense of the main requirements. Usually 3–5 exchanges.

---

### Phase 2 — Draft the Spec

After the interview, generate the full spec document using the template below.

**Filling in requirements — use this mental model:**

```
Everything the user said they need urgently     → Must Have
Everything that would make it significantly better → Should Have
Nice polish, future ideas mentioned in passing  → Could Have
Things explicitly out of scope or deferred      → Won't Have
```

Each Must Have requirement should be:
- A **business rule statement** (what the system must enforce or enable)
- Testable — you can write a yes/no acceptance test for it
- Not a UI/implementation detail ("the button must be blue" → wrong level)

**Good vs bad requirements:**
```
✅ "Only users with an active paid subscription can access this feature"
✅ "The system must notify the user if processing takes longer than 30 seconds"
✅ "Submitted data must be validated before being sent to the external API"

❌ "The form should look nice"           → not testable
❌ "Use REST API with JSON payload"      → implementation detail
❌ "Handle all edge cases properly"      → too vague
```

---

### Phase 3 — Refine

After presenting the draft, ask:
> "Bạn thấy Requirements section có miss gì không? Đặc biệt phần Must Have — có rule nào quan trọng mà mình chưa capture không?"

Then ask about open questions:
> "Có câu hỏi nào còn chưa có câu trả lời mà cần clarify trước khi dev bắt đầu không?"

---

## Output Template

Always use this exact structure. Every section is required — write "N/A" if truly not applicable, never skip silently.

```markdown
# Feature Spec: [Feature Name]

## Metadata
| Field | Value |
|---|---|
| Status | Draft / In Review / Approved |
| Author | [name] |
| Date | [YYYY-MM-DD] |
| Version | 1.0 |
| Stakeholders | [PM, Tech Lead, QA Lead] |

---

## Problem Statement

[2–4 sentences. Describe the pain in concrete terms: who, what they're experiencing right now, and why the current situation is unacceptable. No solution language here.]

**Evidence:** [data, quotes, support tickets, or research that validates the problem — even if anecdotal]

---

## Goals

### What success looks like
- [Measurable outcome 1]
- [Measurable outcome 2]

### Non-Goals (explicitly out of scope)
- [Thing 1 we are NOT doing in this version]
- [Thing 2 — note why if not obvious]

---

## Users

### Primary User
**Who:** [role, context, relevant attributes]
**Current behavior:** [how they handle this today without the feature]
**Key frustration:** [the specific moment of pain]

### Secondary Users (if any)
**Who:** [role]
**How they're affected:** [downstream impact]

---

## User Stories

As a [primary user],
I want [capability],
So that [outcome / value].

[Add 1–3 stories. Each story = one distinct user goal. Avoid cramming multiple goals into one story.]

---

## Requirements

### Must Have
*Without these, the feature fails its core purpose.*

- **[REQ-M1]** [Business rule statement — what the system must enforce or enable]
- **[REQ-M2]** [Business rule statement]
- **[REQ-M3]** [Business rule statement]

### Should Have
*Important, but the feature ships without them if needed.*

- **[REQ-S1]** [Business rule statement]
- **[REQ-S2]** [Business rule statement]

### Could Have
*Desirable polish — only if capacity allows.*

- **[REQ-C1]** [Business rule statement]

### Won't Have (this version)
*Explicitly deferred — set expectations.*

- [Deferred capability] — *reason: [brief rationale]*

---

## Success Metrics

| Metric | Baseline | Target | Measurement Method |
|---|---|---|---|
| [Primary KPI] | [current value] | [goal value] | [how to measure] |
| [Secondary KPI] | [current value] | [goal value] | [how to measure] |

**Leading indicators** (signal we're on track before launch):
- [metric to watch during development / beta]

---

## Open Questions

Questions that must be answered before development starts:

- [ ] **[Q1]** [Question] → *Owner: [who should answer]* → *Needed by: [date/milestone]*
- [ ] **[Q2]** [Question] → *Owner: [who should answer]*

Questions that can be resolved during development:

- [ ] **[Q3]** [Question]

---

## Assumptions

Things we're taking as true for this spec. If any assumption turns out to be wrong, the spec needs revisiting:

- [Assumption 1]
- [Assumption 2]

---

## Dependencies

- [System / team / external service this feature depends on]
- [API or data source that must be available]
```

---

## Quality Checklist

Before finalizing, verify:

- [ ] Problem Statement has no solution language — it describes pain, not features
- [ ] Every Must Have is a business rule, not a UI or implementation detail
- [ ] Every Must Have can be answered with "pass / fail" (is it testable?)
- [ ] Non-Goals explicitly exclude the most tempting scope creep items
- [ ] Success Metrics have a baseline — without baseline, "target" is meaningless
- [ ] Open Questions have an owner and urgency level
- [ ] The spec could be handed to prd-to-gherkin to generate BDD scenarios

---

## BDD Handoff Note

After approval, this spec is the input for `prd-to-gherkin` skill.

Each **Must Have / Should Have requirement** becomes a `Rule:` block in the feature file.
Each **User Story** becomes the persona context for scenario steps.
Each **Open Question** becomes a `# TODO:` comment in the feature file.

For clean BDD generation, requirements should be written as **observable system behaviors**, not internal states or implementation choices.

---

## Reference Files

- `references/interview-guide.md` — Deeper question bank by feature type (CRUD, integration, workflow, notification)
- `references/requirements-patterns.md` — Common requirement patterns with good/bad examples by domain
