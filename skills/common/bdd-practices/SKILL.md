---
name: bdd-practices
description: Cucumber/Gherkin BDD best practices guidance and Gherkin review. Use when user writes Gherkin scenarios, asks to review .feature files, needs help with Discovery Workshops, Example Mapping, or asks about BDD best practices", "how to write good Gherkin", "declarative vs imperative", scenario design", "Given-When-Then", or converts traditional tests to BDD. Do NOT use for Playwright-specific test execution analysis or flaky test detection (use playwright-bdd-analyzer instead).
---

# Cucumber BDD Best Practices

Guide teams through high-quality BDD practices across Discovery, Formulation, and Automation phases.

## Three Core BDD Practices

### 1. Discovery — "What it could do"

Structured conversations to build shared understanding before coding.

- **Example Mapping**: Four-color index cards to map rules and examples
- **OOPSI Mapping**: Outcomes, Outputs, Processes, Scenarios, Inputs
- **Feature Mapping**: Identify actors, decompose tasks, map examples

Workshop setup: Three Amigos (PO + Dev + Tester), 25-30 min per User Story, as late as possible before development.

For detailed guidance, see [references/discovery-workshop.md](references/discovery-workshop.md).

### 2. Formulation — "What it should do"

Structure examples into executable documentation using Gherkin.

**Core principle**: Treat readers the way you want to be treated — make it understandable to someone who doesn't know the feature.

**Cardinal rule**: One scenario, one behavior!

```gherkin
# ✅ DECLARATIVE — describes "what to do"
Scenario: Free subscribers can only see free articles
  Given Free Frieda has a free subscription
  When Free Frieda logs in with valid credentials
  Then she sees a free article

# ❌ IMPERATIVE — describes "how to do it"
Scenario: Free subscribers can only see free articles
  Given the user is on the login page
  When I enter "free@example.com" in the Email field
  And I click the "Submit" button
  Then I see "FreeArticle1" on the homepage
```

**Verification method**: Ask "If the implementation changes, would this wording need to change?" If yes → rewrite.

For full Gherkin rules, step guidelines, and style standards, see [references/gherkin-golden-rules.md](references/gherkin-golden-rules.md).

### 3. Automation — "What it actually does"

Use automated examples to guide development: take one example → connect as test (fails) → implement (passes) → repeat.

## Common Anti-Patterns

| Anti-Pattern | Signal | Fix |
|---|---|---|
| Procedure-driven tests | Multiple When-Then pairs | Split into separate scenarios |
| Overly imperative | UI details (click, field, button) | Describe behavior, not steps |
| Misused Scenario Outline | Equivalent class repetition | Only vary meaningful differences |
| Hardcoded test data | Brittle assertions on changing data | Pattern validation, hide data in step defs |

For detailed examples and corrections, see [references/anti-patterns.md](references/anti-patterns.md).

## Interactive Guidance Flow

When reviewing or helping write Gherkin, follow this process:

### For Discovery assistance

Ask these questions:
1. Who is the main user of this User Story?
2. What does the user want to achieve?
3. What rules or constraints exist?
4. Can you give a specific example?
5. Are there edge cases or exceptions?

### For Formulation review

Apply this checklist:
- Does the scenario describe behavior, not implementation?
- Is it declarative rather than imperative?
- Does each scenario cover only one behavior?
- Do steps follow Given-When-Then order?
- Is third-person present tense used?
- Is the title clear and concise?
- Are steps between 3-5 (max single digits)?

### For providing corrections

Use this template:
```
### 🔴 Issue Found: [problem name]
❌ Original: [original Gherkin]
💡 Analysis: [why this is a problem]
✅ Suggested: [corrected Gherkin]
📝 Explanation: [why this is better]
```

## Scenario Outline Checklist

Before using Scenario Outline, verify:
1. Each row represents a **different equivalence class** (not just different data)
2. Combination explosion is managed (N fields × M inputs = M^N — consider each input appearing once)
3. No columns represent **different behaviors** (split if so)
4. Reader needs to see data explicitly (hide some in step definitions if not)

For more details, see [references/scenario-outline-checklist.md](references/scenario-outline-checklist.md).

## Key Reminders

- Gherkin is a **communication tool**, not just testing syntax
- Audience includes **non-technical people** — prioritize readability
- Maintain **behavior-driven thinking** — avoid procedure-driven
- Continuously **refactor** scenarios like you refactor code
- Use **third person, present tense** consistently