# Data Model Reference

**Status:** Feature 1 users and sessions.

## Tables

### `users`

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `fName` | STRING | Required |
| `lName` | STRING | Required |
| `email` | STRING | Required, unique |
| `username` | STRING(100) | Required, unique; stored lowercase |
| `password` | STRING(255) | Required; bcrypt hash only; excluded from the default scope |
| `role` | STRING(20) | Required; default `worker` |

### `sessions`

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `token` | STRING | Required; cleared to an empty string on sign-out |
| `email` | STRING | Required |
| `expirationDate` | DATE | Required; 24 hours after creation |
| `userId` | INTEGER FK | Required, references `users.id` |

## Associations

- `users` has many `sessions` (`userId`, delete cascades).
- `sessions` belongs to `users`.

## Feature provenance

| Area | Introduced |
|------|------------|
| `users`, `sessions` | Feature 1 |
