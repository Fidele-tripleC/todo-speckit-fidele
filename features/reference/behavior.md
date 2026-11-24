# Behavior & Rules Reference

**Status:** Feature 1 authentication, Feature 2 list rules, and Feature 3 todo rules.

| Rule | Enforcement | Provenance |
|------|-------------|------------|
| Sign-in uses username and password, not email | `POST /todo/login` | Feature 1, FR-001 |
| Registration requires first name, last name, email, username, and password | Client rules and `POST /todo/register` | Feature 1, FR-002 |
| Password is bcrypt-hashed with 10 salt rounds and is never returned | `User` default scope and register/login payload | Feature 1, FR-003 |
| Session is a JWT stored in `sessions`; client sends `Authorization: Bearer <token>` | `authenticate`, axios request interceptor | Feature 1, FR-004 |
| Session lasts 24 hours | JWT `expiresIn: 86400` and `expirationDate` | Feature 1, FR-005 |
| Login reuses a non-expired session for the same user | `createOrReuseSession` | Feature 1, FR-006 |
| New users receive role `worker` | `users.role` default | Feature 1, FR-007 |
| A valid token resolves to one `req.user.id` | `authenticate` | Feature 1, FR-008 |
| Email must be present and match `/^[^\s@]+@[^\s@]+\.[^\s@]+$/`; invalid message is "Enter a valid email address." | `emailRules` on the register form | Feature 1, FR-009 |
| Username is stored and compared in lowercase | User hook and login/register normalization | Feature 1 |
| Duplicate username or email returns `400` | Register controller | Feature 1 |
| Wrong username or password returns the same `401` message | Login controller | Feature 1 |
| Missing, expired, or revoked token returns `401`; the client clears `localStorage` key `user` and redirects to login | `authenticate` and axios response interceptor | Feature 1 |
| Signed-in users who open login or register are sent to home | `router.beforeEach` | Feature 1 |
| Users without a session who open a protected route are sent to login | `router.beforeEach` | Feature 1 |
| Session is stored in `localStorage` under `user` | Login and register views | Feature 1 |
| Sign out clears the server token, removes `user`, and opens login | `POST /todo/logout`, `logoutUser`, and `MenuBar` | Feature 1, Feature 2 |
| The dashboard is one lists view titled **My Lists**, with **+ New List**, edit, and delete dialogs | `Dashboard.vue` | Feature 2 |
| `MenuBar` shows the signed-in user's name and **Sign out**, and is hidden on login and register | `App.vue`, `MenuBar.vue` | Feature 2 |
| List names are trimmed; empty names are rejected; names longer than 100 characters are rejected | Create and rename validation | Feature 2, FR-005 |
| Lists are returned in alphabetical order by name | `GET /todo/lists` `order` | Feature 2, FR-006 |
| A list belongs to one user. `userId` comes from the session, never the request body | `list.controller.create` | Feature 2, FR-002, FR-004 |
| Reads, renames, and deletes include `userId: req.user.id`. Another user's list returns `404` | `getAccessibleListOrNull` | Feature 2, FR-003 |
| An empty list collection shows "No lists yet. Create your first list." | `Dashboard.vue` | Feature 2 |
| Each list row has an **Items** action that opens a dialog for that list only | `Dashboard.vue` | Feature 3, FR-010 |
| **+ Add Item** is inside the items dialog. An empty list shows "No todos in this list yet." | `Dashboard.vue` | Feature 3 |
| Todo titles are trimmed; empty titles are rejected; titles longer than 255 characters are rejected | Create and edit validation | Feature 3, FR-006 |
| A new todo belongs to the opened list and the signed-in user, and starts incomplete | `todo.controller.create` | Feature 3, FR-002, FR-005, FR-007 |
| Todos are returned incomplete first, then by `createdAt` ascending | `GET /todo/lists/:listId/todos` | Feature 3, FR-009 |
| Todo reads, updates, and deletes match `userId: req.user.id`. Another user's list or todo returns `404` | `getAccessibleListOrNull`, `getAccessibleTodoOrNull` | Feature 3, FR-003, FR-004 |
| Deleting a list deletes its todos | `lists` `hasMany` `todos` `onDelete: CASCADE` | Feature 3, FR-008 |
| A completed todo title is struck through | `Dashboard.vue` | Feature 3 |

## Feature provenance

| Area | Introduced |
|------|------------|
| Authentication, session, route guards | Feature 1 |
| Private todo lists and dashboard | Feature 2 |
| Todo items inside an owned list | Feature 3 |
