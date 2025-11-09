# Football Academy — Backend (Django) + Frontend (Next.js)

A full‑stack football academy management system for coaches and admins to manage groups, players, evaluations, attendance, media uploads, and export bilingual PDF reports.

## Overview

- Backend: Django + Django REST Framework with JWT auth (SimpleJWT)
- Frontend: Next.js (App Router) + React + TypeScript
- Features: groups, players, coach profiles, skill evaluations (1–5), monthly attendance, photo uploads, bilingual PDF exports (English + Arabic labels), role‑based permissions.

## Tech Stack

- Backend: `Django`, `djangorestframework`, `djangorestframework-simplejwt`, `django-cors-headers`, `django-filter`, `python-dotenv`, `reportlab`, `Pillow`
- Optional (Arabic shaping): `arabic-reshaper`, `python-bidi`
- Optional (Postgres): `psycopg2-binary`
- Frontend: `Next.js 14`, `React 18`, `TypeScript`, `Tailwind CSS`, `@tanstack/react-query`

## Project Structure

- `academy/` — Django project and app (`core/`)
- `frontend/` — Next.js app (App Router) with pages like `/login`, `/signup`, `/groups`, `/players`
- `media/` — User-uploaded images (served by Django in debug)

## Prerequisites

- Python 3.11+ (3.12 recommended on Windows)
- Node.js 18+
- Git (optional)

## Backend Setup (Django)

1) Create and activate a virtual environment

- Windows PowerShell:
  - `python -m venv venv`
  - `venv\Scripts\Activate.ps1`

2) Install dependencies

- Minimal set:
  - `pip install django djangorestframework djangorestframework-simplejwt django-cors-headers django-filter python-dotenv Pillow reportlab`
- Optional Arabic support (better PDF labels):
  - `pip install arabic-reshaper python-bidi`
- Optional Postgres (instead of SQLite):
  - `pip install psycopg2-binary`

3) Configure environment

Create `academy/.env` with at least:

```
DJANGO_SECRET_KEY=change-this
DJANGO_DEBUG=true
DJANGO_ALLOWED_HOSTS=*
# Uncomment and set to use Postgres (otherwise SQLite is used)
# DB_NAME=your_db
# DB_USER=your_user
# DB_PASSWORD=your_password
# DB_HOST=localhost
# DB_PORT=5432
```

Notes:
- When `DJANGO_DEBUG=true`, CORS allows all origins. In production, edit `academy/academy/settings.py` CORS settings or set `DEBUG=false` and update allowed origins.

4) Apply migrations and create an admin

- `cd academy`
- `python manage.py migrate`
- `python manage.py createsuperuser`

5) Run the development server

- `python manage.py runserver`
- Default dev API base: `http://127.0.0.1:8000`

## Frontend Setup (Next.js)

1) Install dependencies

- `cd frontend`
- `npm install`

2) Configure environment

Create `frontend/.env.local` (or copy from `.env.local.example`):

```
NEXT_PUBLIC_API_URL=http://127.0.0.1:8000
```

3) Run the development server

- `npm run dev`
- Default dev URL: `http://localhost:3000`

## Authentication & Roles

- JWT auth via SimpleJWT
  - `POST /api/auth/token/` with `{ username, password }` → `{ access, refresh }`
  - `POST /api/auth/token/refresh/`
- Signup creates a user + coach profile
  - `POST /api/auth/signup/` → coach resource
- Me endpoint
  - `GET /api/auth/me/` → `{ user, coach|null, is_staff }`
  - `PATCH /api/auth/me/` → update basic user fields and coach profile (including photo)
- Change password
  - `POST /api/auth/change-password/` → `{ detail }`
- Roles & permissions
  - Admin (`is_staff=True`) has full access
  - Coaches can read everything; they can write only within their own groups/players/evaluations

## Core API Endpoints

Base path: `http://127.0.0.1:8000/api/`

- Coaches (`/coaches/`) [admin]
  - `GET /coaches/` list coaches
  - `POST /coaches/create-with-user/` create both `User` and `Coach`
  - `DELETE /coaches/{id}/` (blocked if coach still owns groups)
- Groups (`/groups/`)
  - `GET /groups/` list (admin: all; coach: own)
  - `POST /groups/` create (admin must provide `coach_id`)
  - `GET /groups/{id}/` get details (includes `players`)
  - `DELETE /groups/{id}/` delete
  - Actions:
    - `GET /groups/{id}/report-pdf/` download group PDF
    - `POST /groups/{id}/reset-evaluations/` null all player evaluations in this group
  - Attendance context: `GET /groups/{id}/?month=YYYY-MM` → player `attendance_days` reflects monthly record if present
- Players (`/players/`)
  - `GET /players/?group={group_id}` filter by group
  - `POST /players/` create (coach can only add to own group)
  - `GET /players/{id}/` get
  - `PATCH /players/{id}/` update (supports `multipart/form-data` for `photo`)
  - `DELETE /players/{id}/` delete
  - Action:
    - `GET /players/{id}/report-pdf/` download player PDF
- Evaluations (`/evaluations/`)
  - `GET /evaluations/?player={id}` list (one per player)
  - `POST /evaluations/` create (coach is auto‑set and must match player’s group coach)
  - `PATCH /evaluations/{id}/` update
  - `GET|PUT|PATCH /evaluations/{id}/attendance?month=YYYY-MM` set/get monthly attendance days for the evaluation’s player

## PDF Reports

- Group report: compact table of players with photo, phone, and average rating
- Player report: single‑page report with sections (Technical, Physical, Understanding, Psychological, Overall)
- Bilingual labels: English + Arabic (when `arabic-reshaper` and `python-bidi` installed; Windows fonts auto‑detected)

## Media & Uploads

- `MEDIA_URL=/media/` and `MEDIA_ROOT=academy/media`
- `ImageField` for `coach.photo` and `player.photo` requires `Pillow`
- In debug, Django serves media via `urlpatterns += static(...)`

## Frontend Routes (dev)

- `/login` — obtain JWT and redirect
- `/signup` — create account (coach)
- `/groups` — list groups; admin can create/delete and assign coach
- `/groups/{id}` — group detail; add existing players; reset evaluations; download PDF
- `/players` — per‑group player management with photo upload
- `/players/{id}` — player profile with detailed evaluation editing and PDF download
- `/profile` — update user/coach profile (navbar link)

## Running Tests (backend)

- `cd academy`
- `python manage.py test`
  - Includes PDF download tests in `core/tests/test_pdf_reports.py`

## Deployment Notes

- Set `DJANGO_DEBUG=false` and `DJANGO_ALLOWED_HOSTS` appropriately
- Configure CORS for your frontend origin (update `settings.py` or add `django-cors-headers` config)
- Use Postgres by setting `DB_*` variables and installing `psycopg2-binary`
- Collect static files: `python manage.py collectstatic`

## Troubleshooting

- “Invalid token” or navbar disappears → token expired; login again
- PDF Arabic labels show blocks/garbled → install `arabic-reshaper` + `python-bidi`; ensure a TrueType font is available (Arial/Tahoma/etc.)
- Image uploads fail → verify `Pillow` installed and `MEDIA_ROOT` exists
- CORS errors → ensure `NEXT_PUBLIC_API_URL` points to the backend and CORS is allowed in Django

## License

Proprietary project; no license file included. Ask the project owner before redistributing.