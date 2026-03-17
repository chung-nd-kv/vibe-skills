---
name: conventional-commit
description: Conventional Commits 1.0.0 format for consistent, parseable git commit messages. Use when committing changes, writing commit messages, or when the user mentions commit, save, git commit, or wants to persist work. Covers type selection, scope rules, description format, breaking changes, and multi-line commit workflow.
---

# Conventional Commits

Structured commit messages following the [Conventional Commits 1.0.0](https://www.conventionalcommits.org/) specification.

## Format

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

## Types

| Type       | When                                    | Example                                    |
| ---------- | --------------------------------------- | ------------------------------------------ |
| `feat`     | New feature                             | `feat(auth): add OAuth2 login`             |
| `fix`      | Bug fix                                 | `fix(cart): correct total calculation`      |
| `refactor` | Code restructuring (no behavior change) | `refactor(api): extract error handler`     |
| `docs`     | Documentation only                      | `docs(readme): update setup instructions`  |
| `test`     | Adding or updating tests                | `test(order): add unit tests for checkout` |
| `chore`    | Build, config, tooling                  | `chore(deps): update Angular to v18`       |
| `style`    | Formatting, semicolons (no logic)       | `style(lint): apply prettier formatting`   |
| `perf`     | Performance improvement                 | `perf(query): add database index`          |
| `build`    | Build system changes                    | `build(ci): update Node.js to v20`         |
| `ci`       | CI/CD configuration                     | `ci(github): add deploy workflow`          |
| `revert`   | Revert a previous commit                | `revert: undo feat(auth) OAuth changes`    |

## Rules

- **Description:** lowercase, imperative mood, no period at end, max 72 characters
- **Scope:** optional but recommended, lowercase (module, component, or area)
- **Breaking changes:** Add `!` after type/scope: `feat(api)!: remove deprecated endpoint`
- **Body:** Separated by blank line, explains "what" and "why" (not "how")
- **Footer:** `BREAKING CHANGE: <description>` or `Refs: #123`

## Workflow

1. **Check status** — `git status` to see what changed
2. **Stage changes** — Prefer `git add <file1> <file2>` over `git add -A`
3. **Analyze changes** — Determine type, scope, and description
4. **Compose message** — Follow the format above
5. **Commit** — Use HEREDOC for multi-line messages:
   ```bash
   git commit -m "$(cat <<'EOF'
   feat(auth): add OAuth2 login flow

   Implements OAuth2 authorization code flow with PKCE.
   Adds login, callback, and token refresh endpoints.

   Refs: #142
   EOF
   )"
   ```
6. **Verify** — `git log --oneline -1` to confirm

For examples of good and bad commit messages, see [references/commit-examples.md](references/commit-examples.md).
