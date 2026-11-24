# API Reference

**Status:** Feature 1 authentication is implemented. Lists and todos are not stored yet.

Base path: `/todo/`.

Authenticated routes require `Authorization: Bearer <token>`.

## Endpoints

| Method | Path | Auth | Purpose |
|--------|------|------|---------|
| `POST` | `/todo/register` | No | Create an account and session |
| `POST` | `/todo/login` | No | Sign in with username and password |
| `POST` | `/todo/logout` | Yes | Invalidate the current session token |
| `GET` | `/todo/lists` | Yes | Return the signed-in user's lists |

`GET /todo/lists` returns `[]` until Feature 2 stores lists. It does not return another user's data.

## Auth success payload

`POST /todo/register` returns `201`. `POST /todo/login` returns `200`.

```json
{
  "userId": 1,
  "username": "jdoe",
  "email": "jdoe@example.com",
  "fName": "Jane",
  "lName": "Doe",
  "role": "worker",
  "token": "<jwt>"
}
```

The password hash is never included.

## Errors

`{ "message": "Human-readable explanation." }`

| Situation | Status | Message |
|-----------|--------|---------|
| Missing required registration or login field | `400` | Field-specific required message |
| Password shorter than 8 characters | `400` | `Password must be at least 8 characters.` |
| Duplicate username | `400` | `Username is already taken.` |
| Duplicate email | `400` | `Email is already registered.` |
| Wrong username or password | `401` | `Invalid username or password.` |
| Missing bearer token | `401` | `Unauthorized! No token provided.` |
| Expired or revoked token | `401` | `Unauthorized! Invalid or expired token.` |
| Sign out | `200` | `Signed out successfully.` |

## Feature provenance

| Area | Introduced |
|------|------------|
| Register, login, logout, session token | Feature 1 |
| `GET /todo/lists` auth gate (empty until lists exist) | Feature 1 |
