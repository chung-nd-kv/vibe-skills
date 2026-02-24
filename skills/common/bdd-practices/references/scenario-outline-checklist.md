# Scenario Outline Checklist

When using Scenario Outline, verify these four criteria:

## 1. Equivalence Class Check

- Does each row represent a **different equivalence class**?
- Searching "elephant" in addition to "panda" does NOT add test value if they're the same class
- Each row should test a meaningfully different behavior or outcome

## 2. Combination Necessity

- Do you need to cover all input combinations?
- N fields with M inputs each = M^N combinations — this explodes quickly
- Consider: each input appearing only once, without considering all combinations
- Use pairwise testing if combinations matter

## 3. Behavior Separation

- Are there columns representing **different behaviors**?
- Signal: columns are never referenced together in the same step
- Fix: split Scenario Outline by column into separate scenarios

## 4. Data Transparency

- Does the reader **need** to see all data explicitly?
- Some data can be hidden in step definitions
- Some data can be derived from other data
- Only expose what helps understanding