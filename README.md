# FastAPI Blog

A full-stack blog application built with FastAPI, async SQLAlchemy, PostgreSQL, server-rendered Jinja pages, JWT authentication, and AWS S3 profile image uploads.

![FastAPI Blog home feed](docs/screenshots/home-feed.png)

## What It Does

FastAPI Blog is a polished blog platform with both HTML pages and JSON APIs. It supports user registration, login, authenticated post management, profile pictures, password reset flows, pagination, and a Bootstrap-powered light/dark interface.

## Screenshots

| Home Feed | Post Detail |
| --- | --- |
| ![Home feed](docs/screenshots/home-feed.png) | ![Post detail](docs/screenshots/post-detail.png) |

| Login | Register |
| --- | --- |
| ![Login page](docs/screenshots/login.png) | ![Register page](docs/screenshots/register.png) |

| New Post | User Posts |
| --- | --- |
| ![New post modal](docs/screenshots/new-post-modal.png) | ![User posts page](docs/screenshots/user-posts.png) |

| Account Settings | More Feed Content |
| --- | --- |
| ![Account settings](docs/screenshots/account-settings.png) | ![Scrolled home feed](docs/screenshots/home-feed-scrolled.png) |

## Features

- Server-rendered blog pages with Jinja2 templates.
- JSON API routes for users and posts.
- User registration and email-based login.
- JWT bearer-token authentication with protected routes.
- Create, read, update, and delete blog posts.
- Account page with profile details, profile picture upload, password change, logout, and account deletion.
- Password reset flow with expiring reset tokens and email delivery.
- PostgreSQL persistence through SQLAlchemy 2.0 async ORM.
- Alembic migrations for database schema management.
- AWS S3 profile image storage under `profile_pics/`.
- Pillow image processing that crops profile pictures to square JPEGs before upload.
- Paginated post feeds with "Load More" behavior.
- Bootstrap 5 UI with light, dark, and auto theme modes.
- OpenAPI and Swagger docs from FastAPI.
- Async tests using pytest, HTTPX, PostgreSQL, and Moto for mocked AWS S3.

## Tech Stack

| Layer | Tools |
| --- | --- |
| Backend | FastAPI, Starlette, Uvicorn |
| Templates/UI | Jinja2, Bootstrap 5, custom CSS, browser JavaScript |
| Database | PostgreSQL, SQLAlchemy async ORM, psycopg |
| Migrations | Alembic |
| Validation/Settings | Pydantic, pydantic-settings |
| Auth/Security | PyJWT, pwdlib, OAuth2 bearer tokens |
| Images/Storage | Pillow, boto3, AWS S3 |
| Email | aiosmtplib, HTML email templates |
| Testing | pytest, HTTPX, Moto |

## Project Structure

```text
.
|-- alembic/              # Database migrations
|-- docs/screenshots/     # README screenshots
|-- media/                # Local development media leftovers
|-- populate_images/      # Images used by the seed script
|-- routers/              # API routers for users and posts
|-- static/               # CSS, JavaScript, icons, default profile image
|-- templates/            # Jinja2 pages and email templates
|-- tests/                # Async API tests
|-- auth.py               # Password hashing, JWT creation, current-user dependency
|-- check_s3.py           # S3 upload/delete smoke test
|-- config.py             # Environment-driven settings
|-- database.py           # Async SQLAlchemy engine/session
|-- image_utils.py        # Image processing and S3 helpers
|-- main.py               # FastAPI app, frontend routes, exception handlers
|-- models.py             # SQLAlchemy models
|-- populate_db.py        # Optional database seeding script
`-- schemas.py            # Pydantic request/response models
```

## Getting Started

These steps assume Python 3.11+ and PostgreSQL are installed.

### 1. Create a Virtual Environment

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
```

On macOS/Linux:

```bash
python -m venv venv
source venv/bin/activate
python -m pip install --upgrade pip
```

### 2. Install Dependencies

This repo currently does not include a tracked dependency manifest, so install the packages used by the application:

```powershell
pip install fastapi[standard] uvicorn sqlalchemy alembic "psycopg[binary]" pydantic-settings PyJWT pwdlib pillow boto3 aiosmtplib python-multipart email-validator jinja2 pytest httpx moto
```

### 3. Create Environment Variables

Create a `.env` file in the project root:

```env
DATABASE_URL=postgresql+psycopg://bloguser:blogpass@localhost/blog
SECRET_KEY=replace-with-a-long-random-secret
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

S3_BUCKET_NAME=your-s3-bucket-name
S3_REGION=ap-south-1
S3_ACCESS_KEY_ID=your-access-key-id
S3_SECRET_ACCESS_KEY=your-secret-access-key

MAX_UPLOAD_SIZE_BYTES=5242880
POST_PER_PAGE=10
RESET_TOKEN_EXPIRE_MINUTES=60

MAIL_SERVER=smtp.example.com
MAIL_PORT=587
MAIL_USERNAME=your-smtp-username
MAIL_PASSWORD=your-smtp-password
MAIL_FROM=noreply@example.com
MAIL_USE_TLS=true

FRONTEND_URL=http://localhost:8000
```

Keep `.env` private. It is already ignored by Git.

### 4. Create the PostgreSQL Database

Create a local database that matches your `DATABASE_URL`. For example:

```sql
CREATE DATABASE blog;
CREATE USER bloguser WITH PASSWORD 'blogpass';
GRANT ALL PRIVILEGES ON DATABASE blog TO bloguser;
```

### 5. Run Migrations

```powershell
alembic upgrade head
```

### 6. Optional: Seed Demo Data

```powershell
python populate_db.py
```

The seed script creates demo users, posts, and sample profile images. Do not reuse seed credentials in production.

### 7. Verify S3 Access

```powershell
python check_s3.py
```

This uploads and deletes a small test object from `profile_pics/` in your configured S3 bucket.

### 8. Run the App

```powershell
fastapi dev main.py
```

Or:

```powershell
uvicorn main:app --reload
```

Then open:

- App: `http://localhost:8000`
- Swagger UI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`

## API Overview

### Users

| Method | Path | Purpose |
| --- | --- | --- |
| `POST` | `/api/users` | Register a user |
| `POST` | `/api/users/token` | Log in and receive a JWT |
| `GET` | `/api/users/me` | Get the authenticated user |
| `POST` | `/api/users/forgot-password` | Request password reset email |
| `POST` | `/api/users/reset-password` | Reset password with token |
| `PATCH` | `/api/users/me/password` | Change password while logged in |
| `GET` | `/api/users/{user_id}` | Get a public user profile |
| `GET` | `/api/users/{user_id}/posts` | Get paginated posts for a user |
| `DELETE` | `/api/users/{user_id}` | Delete the authenticated user's account |
| `PATCH` | `/api/users/{user_id}/picture` | Upload profile picture to S3 |
| `DELETE` | `/api/users/{user_id}/picture` | Delete profile picture from S3 |

### Posts

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/api/posts` | Get paginated posts |
| `POST` | `/api/posts` | Create a post |
| `GET` | `/api/posts/{post_id}` | Get a single post |
| `PUT` | `/api/posts/{post_id}` | Replace a post |
| `PATCH` | `/api/posts/{post_id}` | Partially update a post |
| `DELETE` | `/api/posts/{post_id}` | Delete a post |

Authenticated routes require:

```http
Authorization: Bearer <access_token>
```

## Running Tests

The test suite uses a PostgreSQL test database configured in `tests/conftest.py`:

```text
postgresql+psycopg://bloguser:blogpass@localhost/test_blog
```

Create the `test_blog` database before running tests:

```sql
CREATE DATABASE test_blog;
GRANT ALL PRIVILEGES ON DATABASE test_blog TO bloguser;
```

Then run:

```powershell
pytest
```

Tests cover user creation, duplicate email validation, login helpers, profile picture upload through mocked S3, password reset email dispatch, post CRUD authorization, and pagination.

## Notes

- Profile images are normalized to `300x300` JPEG files before upload.
- If a user has no uploaded profile picture, the app serves `static/profile_pics/default.jpg`.
- Deleting a user also deletes their posts through ORM cascade behavior.
- The frontend stores the JWT access token in `localStorage`.
- Alembic reads `DATABASE_URL` through `config.py`, so migrations use the same `.env` configuration as the app.
