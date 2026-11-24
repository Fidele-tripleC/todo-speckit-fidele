# Data Model Reference

**Status:** Feature 1 users and sessions. Feature 2 lists. Feature 3 todos. Feature 4 edits the existing `users` row. Feature 5 adds `todos.dueDate`.

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
| `role` | STRING(20) | Required; default `worker`; not changed by profile update |

### `sessions`

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `token` | STRING | Required; cleared to an empty string on sign-out |
| `email` | STRING | Required |
| `expirationDate` | DATE | Required; 24 hours after creation |
| `userId` | INTEGER FK | Required, references `users.id` |

### `lists`

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `name` | STRING(100) | Required; trimmed; max 100 characters |
| `userId` | INTEGER FK | Required; references `users.id`; set from the signed-in user on create |
| `createdAt` | DATE | Sequelize timestamp |
| `updatedAt` | DATE | Sequelize timestamp |

### `todos`

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `listId` | INTEGER FK | Required; references `lists.id`; list delete cascades |
| `title` | STRING(255) | Required; trimmed; max 255 characters |
| `completed` | BOOLEAN | Required; default `false` |
| `dueDate` | DATEONLY | Nullable; optional calendar date `YYYY-MM-DD` |
| `userId` | INTEGER FK | Required; references `users.id`; set from the signed-in user on create |
| `createdAt` | DATE | Sequelize timestamp |
| `updatedAt` | DATE | Sequelize timestamp |

## Associations

- `users` has many `sessions` (`userId`, delete cascades).
- `sessions` belongs to `users`.
- `users` has many `lists` (`userId`, delete cascades).
- `lists` belongs to `users`.
- `lists` has many `todos` (`listId`, delete cascades).
- `todos` belongs to `lists`.
- `users` has many `todos` (`userId`, delete cascades).
- `todos` belongs to `users`.

## Feature provenance

| Area | Introduced |
|------|------------|
| `users`, `sessions` | Feature 1 |
| `lists` | Feature 2 |
| `todos` | Feature 3 |
| `todos.dueDate` | Feature 5 |
| Profile edits on `users` (no new table) | Feature 4 |
