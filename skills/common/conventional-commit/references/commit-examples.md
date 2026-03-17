# Commit Message Examples

## Good Examples

```
feat(user): add profile avatar upload

Supports PNG/JPG up to 5MB. Images are resized to 256x256
and stored in the user's S3 bucket.

Refs: #234
```

```
fix(cart): prevent negative quantity on item update

The quantity input accepted negative values, causing incorrect
total calculations. Added min=1 validation on both client and server.
```

```
refactor(order): extract payment processing to dedicated service

Moves payment logic from OrderService to PaymentService to
improve separation of concerns and enable reuse in SubscriptionService.
```

```
test(auth): add integration tests for token refresh

Covers expired token, invalid refresh token, and concurrent
refresh scenarios.
```

```
chore(deps): update Angular to v18.2.0

Also updates related packages: @angular/cdk, @angular/material.
No breaking changes in this minor version.
```

```
feat(api)!: change pagination from offset to cursor-based

BREAKING CHANGE: The `page` parameter is replaced by `cursor`.
Existing clients must update their pagination logic.
```

## Bad Examples (with fixes)

```
# Bad: vague description
Updated stuff
# Good: specific type + scope + description
refactor(dashboard): simplify chart data transformation
```

```
# Bad: not imperative mood
feat(user): added login feature
# Good: imperative mood
feat(user): add login feature
```

```
# Bad: too long, includes implementation details
fix(api): fixed the bug where the API was returning 500 errors when the database connection pool was exhausted by adding a retry mechanism with exponential backoff
# Good: concise description, details in body
fix(api): add retry mechanism for database connection failures
```

```
# Bad: period at end
feat(auth): add OAuth2 support.
# Good: no period
feat(auth): add OAuth2 support
```

```
# Bad: uppercase description
feat(Auth): Add OAuth2 Support
# Good: lowercase
feat(auth): add OAuth2 support
```
