# Task-Management-REST-API_week-9
This project teaches API design principles and backend development skills.
# Task Management REST API

A RESTful task management API built with Flask, SQLAlchemy and JWT authentication. It covers user accounts, task CRUD, filtering, search, sorting, pagination, rate limiting, consistent error handling, and API documentation.

## Features

- JWT authentication: register, login, refresh, logout (token revocation)
- Task CRUD with per-user data isolation
- Filtering (`status`, `priority`, `category`), search (`q`), sorting, pagination
- Input validation and a consistent JSON response/error format
- Rate limiting (default 100/hour, 10/min on auth routes) and CORS
- Versioned API (`/api/v1`), OpenAPI spec, Postman collection
- 16 automated tests (pytest)

## Tech stack

Flask 3 · Flask-SQLAlchemy · Flask-JWT-Extended · Flask-Limiter · Flask-CORS · SQLite (swappable via `DATABASE_URL`)

## Setup

```bash
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env          # set real secrets for production
python run.py                 # http://localhost:5000/api/v1
```

Tables are created automatically on first start.

## Quick start

```bash
# Register (returns tokens)
curl -X POST localhost:5000/api/v1/auth/register -H "Content-Type: application/json" \
  -d '{"username":"john","email":"john@example.com","password":"SecurePass123"}'

# Create a task
curl -X POST localhost:5000/api/v1/tasks \
  -H "Authorization: Bearer <access_token>" -H "Content-Type: application/json" \
  -d '{"title":"Complete API project","priority":"high","due_date":"2030-02-15"}'

# List: filter + sort + paginate
curl "localhost:5000/api/v1/tasks?status=pending&sort_by=due_date&sort_order=asc&page=1" \
  -H "Authorization: Bearer <access_token>"
```

## Endpoints

| Method | Path | Auth | Description |
|---|---|---|---|
| POST | `/api/v1/auth/register` | – | Create account |
| POST | `/api/v1/auth/login` | – | Obtain tokens |
| POST | `/api/v1/auth/refresh` | refresh token | New access token |
| POST | `/api/v1/auth/logout` | access token | Revoke token |
| GET | `/api/v1/tasks` | ✔ | List (filter/search/sort/paginate) |
| POST | `/api/v1/tasks` | ✔ | Create |
| GET | `/api/v1/tasks/{id}` | ✔ | Retrieve |
| PUT / PATCH | `/api/v1/tasks/{id}` | ✔ | Full / partial update |
| DELETE | `/api/v1/tasks/{id}` | ✔ | Delete |

Full reference: [`docs/api.md`](docs/api.md) · OpenAPI: [`docs/swagger.yaml`](docs/swagger.yaml) (paste into https://editor.swagger.io) · Postman: [`docs/postman_collection.json`](docs/postman_collection.json)

## Project structure

```
app/
├── __init__.py        # application factory
├── extensions.py      # db, jwt, limiter instances
├── models.py          # User, Task, TokenBlocklist
├── errors.py          # error handlers + JWT error callbacks
├── auth/routes.py     # authentication endpoints
├── tasks/routes.py    # task endpoints
└── utils/             # validators, decorators, responses
tests/                 # pytest suite (in-memory SQLite)
docs/                  # api.md, swagger.yaml, postman_collection.json
config.py · run.py
```

## Technical details

- **Architecture:** application factory + blueprints; extensions are created once and bound in `create_app`, which keeps testing simple.
- **Auth:** short-lived access tokens (15 min) and refresh tokens (7 days). Passwords are hashed with Werkzeug (PBKDF2). Logout stores the token's `jti` in a blocklist checked on every request.
- **Data isolation:** every task query filters by the authenticated user's id; another user's task returns 404, so its existence is not revealed.
- **Validation:** centralized in `utils/validators.py`; errors are returned per field. `sort_by` is whitelisted to prevent arbitrary column access, and `per_page` is capped at 100.
- **Error format:** all errors return `{"status": "error", "message": ..., "errors": {...}}`, including 404/405/429/500 and JWT failures. Unhandled exceptions are logged.

## Testing

```bash
pytest -v
```

The suite covers registration/login/refresh/logout, token revocation, protected routes, task CRUD, validation errors, per-user isolation, filtering, search, sorting, pagination and JSON error responses.

```
16 passed
```

## Screenshots

Add screenshots here for submission (Postman runs, `pytest` output, Swagger editor view):

- `docs/screenshots/postman.png`
- `docs/screenshots/tests.png`
- `docs/screenshots/swagger.png`

## Possible extensions

Password reset by email, task assignment/sharing, comments, Alembic migrations, Redis-backed rate limiting for multi-instance deployments.
