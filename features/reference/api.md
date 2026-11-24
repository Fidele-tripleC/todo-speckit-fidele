# API Reference

**Status:** Feature 1 authentication, Feature 2 private lists, and Feature 3 private todos.

Base path: `/todo/`.

Authenticated routes require `Authorization: Bearer <token>`.

## Endpoints

| Method | Path | Auth | Purpose |
|--------|------|------|---------|
| `POST` | `/todo/register` | No | Create an account and session |
| `POST` | `/todo/login` | No | Sign in with username and password |
| `POST` | `/todo/logout` | Yes | Invalidate the current session token |
| `GET` | `/todo/lists` | Yes | Return the signed-in user's lists, ordered by name |
| `POST` | `/todo/lists` | Yes | Create a list owned by the signed-in user |
| `PUT` | `/todo/lists/:listId` | Yes | Rename a list owned by the signed-in user |
| `DELETE` | `/todo/lists/:listId` | Yes | Delete a list owned by the signed-in user, and its todos |
| `GET` | `/todo/lists/:listId/todos` | Yes | Return todos in an owned list |
| `POST` | `/todo/lists/:listId/todos` | Yes | Add a todo to an owned list |
| `PUT` | `/todo/todos/:id` | Yes | Update an owned todo's title and/or `completed` |
| `DELETE` | `/todo/todos/:id` | Yes | Delete an owned todo |

`POST /todo/lists` accepts `{ "name": "Groceries" }`. A `userId` in the body is ignored. The saved owner is `req.user.id`.

List success body:

```json
{
  "id": 1,
  "name": "Groceries",
  "userId": 42,
  "createdAt": "2026-07-02T12:00:00.000Z",
  "updatedAt": "2026-07-02T12:00:00.000Z"
}
```

`GET /todo/lists` returns an array of those objects. Another user's list is omitted. Rename or delete of a list the caller does not own returns `404` with `{ "message": "List with id=<id> not found." }`. Deleting a list also deletes its todos.

`POST /todo/lists/:listId/todos` accepts `{ "title": "Buy milk" }`. A `userId` in the body is ignored. The parent list must belong to the caller, or the response is `404` with `{ "message": "List with id=<id> not found." }`. The saved todo uses `userId` from the session and `listId` from the owned list. New todos start with `completed: false`.

Todo success body (`201` on create, `200` on update):

```json
{
  "id": 10,
  "listId": 1,
  "title": "Buy milk",
  "completed": false,
  "userId": 42,
  "createdAt": "2026-07-02T12:05:00.000Z",
  "updatedAt": "2026-07-02T12:05:00.000Z"
}
```

`GET /todo/lists/:listId/todos` returns that user's todos for the list, incomplete first, then oldest `createdAt` first. `PUT /todo/todos/:id` accepts `title` and/or `completed`. A missing or unowned todo returns `404` with `{ "message": "Todo with id=<id> not found." }`. Delete returns `200` with `{ "message": "Todo deleted successfully." }`.

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
| Empty list name | `400` | `List name is required.` |
| List name longer than 100 characters | `400` | `List name must be 100 characters or fewer.` |
| Invalid list id | `400` | `Invalid list id.` |
| List missing or owned by someone else | `404` | `List with id=<id> not found.` |
| List deleted | `200` | `List deleted successfully.` |
| Empty todo title | `400` | `Todo title is required.` |
| Todo title longer than 255 characters | `400` | `Todo title must be 255 characters or fewer.` |
| Invalid todo id | `400` | `Invalid todo id.` |
| Todo missing or owned by someone else | `404` | `Todo with id=<id> not found.` |
| Todo deleted | `200` | `Todo deleted successfully.` |

## Feature provenance

| Area | Introduced |
|------|------------|
| Register, login, logout, session token | Feature 1 |
| List CRUD, owner-scoped `GET /todo/lists` | Feature 2 |
| Todo CRUD under an owned list | Feature 3 |
