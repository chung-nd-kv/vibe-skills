---
name: code-reviewer
description: Systematic code review for quality, correctness, and maintainability. Use when reviewing pull requests, code changes, diffs, or when asked to review/critique code. Also use when user says review this, check my code, any issues with this, PR review, code review, or provides code and asks for feedback. Covers functionality, architecture, performance, security, testing, and documentation with structured feedback using priority prefixes BLOCKING, SUGGESTION, QUESTION, NIT. Do NOT use when user wants to write new code from scratch or needs help debugging runtime errors.
---

# Code Reviewer

Systematic approach to reviewing code changes with consistent, actionable feedback.

## Review Process

1. **Understand context** — Read PR description, linked issues, related files
2. **Identify scope** — What changed? What's the intent? What could break?
3. **Review by area** — Apply relevant checklists from [references/checklists.md](references/checklists.md)
4. **Check architecture** — Does it follow project patterns? See [references/dotnet-patterns.md](references/dotnet-patterns.md) for .NET/Clean Architecture
5. **Provide feedback** — Use comment format below, prioritize blocking issues first

## Comment Format

Use prefixes to indicate priority:

| Prefix | Meaning | Action |
|--------|---------|--------|
| `[BLOCKING]` | Must fix before merge | Required |
| `[SUGGESTION]` | Improvement opportunity | Optional |
| `[QUESTION]` | Need clarification | Response needed |
| `[NIT]` | Minor style issue | Optional |

Structure each comment as:

```
[PREFIX] Brief issue description

Why: Explanation of the problem or risk
Fix: Suggested solution or alternative
```

**Example:**
```
[BLOCKING] SQL injection vulnerability in user search

Why: User input concatenated directly into query string
Fix: Use parameterized query

// Before
var sql = $"SELECT * FROM Users WHERE Name = '{input}'";

// After  
var sql = "SELECT * FROM Users WHERE Name = @name";
cmd.Parameters.AddWithValue("@name", input);
```

## Review Priority Order

Review in this order to catch critical issues first:

1. **Security** — Auth, injection, data exposure, multi-tenant isolation
2. **Correctness** — Logic errors, edge cases, data integrity
3. **Performance** — N+1 queries, missing indexes, resource leaks
4. **Architecture** — SOLID violations, wrong layer, circular dependencies
5. **Code quality** — Naming, duplication, readability
6. **Testing** — Missing tests, flaky patterns
7. **Documentation** — Missing or outdated docs

## Feedback Principles

- Point to exact lines with specific alternatives
- Explain *why* something is problematic, not just *what*
- Focus on code, not the author
- Acknowledge good patterns when found
- Batch related issues into a single comment
- For large PRs: focus on architecture and security first, nits later

## References

- **Review checklists**: See [references/checklists.md](references/checklists.md)
- **.NET/Clean Architecture patterns**: See [references/dotnet-patterns.md](references/dotnet-patterns.md)
- **Angular patterns**: See [references/angular-patterns.md](references/angular-patterns.md)
- **Flutter / Dart**: See [references/flutter-dart-patterns.md](references/flutter-dart-patterns.md)