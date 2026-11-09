# FastAPI Blog

A small blog application built with FastAPI, async SQLAlchemy, and Jinja2 templates. It exposes a JSON REST API with JWT authentication and also serves server-rendered HTML pages for browsing posts, users, and accounts.

## Features

- User registration and login with hashed passwords
- JWT bearer-token authentication (OAuth2 password flow)
- Create, read, update, and delete blog posts
- User profiles with post listings
- Server-rendered pages with Jinja2 templates
- Async SQLite database via `aiosqlite`
- Automatic database table creation on startup

## Tech Stack

| Layer | Technology |
| --- | --- |
| Framework | FastAPI |
| ORM | SQLAlchemy 2.0 (async) |
| Database | SQLite (`aiosqlite`) |
| Templates | Jinja2 |
| Validation | Pydantic v2 / pydantic-settings |
| Auth | PyJWT, pwdlib (Argon2) |
| Server | Uvicorn |

## Project Structure

```
web-app/
├── main.py            # FastAPI app, page routes, exception handlers
├── config.py          # Settings loaded from .env
├── database.py        # Async engine, session factory, Base
├── models.py          # SQLAlchemy models (User, Post)
├── schemas.py         # Pydantic request/response schemas
├── auth.py            # Password hashing, JWT, current-user dependency
├── routers/
│   ├── users.py       # User API endpoints
│   └── posts.py       # Post API endpoints
├── templates/         # Jinja2 HTML templates
├── static/            # CSS, JS, icons, default profile image
├── media/             # Uploaded profile pictures
└── blog.db            # SQLite database (created at runtime)
```

## Getting Started

### Prerequisites

- Python 3.10+
- `pip`

### Installation

```bash
# Clone the repository and enter the project directory
cd web-app

# Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate        # macOS / Linux
# .venv\Scripts\activate         # Windows

# Install dependencies
pip install fastapi "uvicorn[standard]" sqlalchemy aiosqlite jinja2 \
    pydantic pydantic-settings "pydantic[email]" pyjwt pwdlib python-multipart
```

### Configuration

Create a `.env` file in the project root:

```env
SECRET_KEY=replace-this-with-a-long-random-string
```

Optional settings (defaults shown):

```env
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

> `SECRET_KEY` is required. Generate a strong value, for example:
> `python -c "import secrets; print(secrets.token_hex(32))"`

### Run the App

```bash
uvicorn main:app --reload
```

- Web app: <http://127.0.0.1:8000>
- Interactive API docs (Swagger UI): <http://127.0.0.1:8000/docs>
- ReDoc: <http://127.0.0.1:8000/redoc>

The SQLite database and tables are created automatically on first startup.

## Web Pages

| Route | Description |
| --- | --- |
| `/` or `/posts` | Home page listing all posts |
| `/posts/{post_id}` | Single post page |
| `/users/{user_id}/posts` | Posts by a specific user |
| `/login` | Login page |
| `/register` | Registration page |
| `/account` | Account page |

## API Endpoints

All API routes are prefixed with `/api`.

### Users

| Method | Endpoint | Auth | Description |
| --- | --- | --- | --- |
| `POST` | `/api/users` | No | Register a new user |
| `POST` | `/api/users/token` | No | Log in and receive a JWT |
| `GET` | `/api/users/me` | Yes | Get the current user |
| `GET` | `/api/users/{user_id}` | No | Get a user by ID |
| `GET` | `/api/users/{user_id}/posts` | No | List posts by a user |
| `PATCH` | `/api/users/{user_id}` | Yes | Update your own profile |
| `DELETE` | `/api/users/{user_id}` | Yes | Delete your own account |

### Posts

| Method | Endpoint | Auth | Description |
| --- | --- | --- | --- |
| `GET` | `/api/posts` | No | List all posts |
| `POST` | `/api/posts` | Yes | Create a post |
| `GET` | `/api/posts/{post_id}` | No | Get a post by ID |
| `PUT` | `/api/posts/{post_id}` | Yes | Replace a post |
| `PATCH` | `/api/posts/{post_id}` | Yes | Partially update a post |
| `DELETE` | `/api/posts/{post_id}` | Yes | Delete a post |

## Authentication

Login uses the OAuth2 password flow. Although the form field is named `username`, it accepts the user's **email address**.

```bash
# 1. Request a token
curl -X POST http://127.0.0.1:8000/api/users/token \
  -d "username=you@example.com" \
  -d "password=yourpassword"

# Response
# {"access_token": "<jwt>", "token_type": "bearer"}

# 2. Use the token on protected endpoints
curl http://127.0.0.1:8000/api/users/me \
  -H "Authorization: Bearer <jwt>"
```

Tokens expire after 30 minutes by default (`ACCESS_TOKEN_EXPIRE_MINUTES`).

## Example Requests

```bash
# Register
curl -X POST http://127.0.0.1:8000/api/users \
  -H "Content-Type: application/json" \
  -d '{"username": "jane", "email": "jane@example.com", "password": "supersecret"}'

# Create a post
curl -X POST http://127.0.0.1:8000/api/posts \
  -H "Authorization: Bearer <jwt>" \
  -H "Content-Type: application/json" \
  -d '{"title": "Hello World", "content": "My first post!"}'

# List posts
curl http://127.0.0.1:8000/api/posts
```

## Data Model

**User**

- `id`, `username` (unique), `email` (unique), `password` (hashed), `image_file`
- One-to-many relationship with `Post`

**Post**

- `id`, `title`, `content`, `user_id`, `date_posted`
- Belongs to a `User` via `author`

## Notes

- `.env` is git-ignored; never commit your secret key.
- Passwords are hashed with `pwdlib`'s recommended algorithm before storage.
- The app mounts `/static` and `/media` for serving assets and profile pictures.
