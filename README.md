# Meal Resource Management System (MRMS)

A Django-based web platform that helps reduce food waste by connecting food donors with food collectors for pickup and redistribution.

## Table of Contents
- [Overview](#overview)
- [Key Features](#key-features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Data Model](#data-model)
- [User Flows](#user-flows)
- [Getting Started](#getting-started)
- [Area Seed Data](#area-seed-data)
- [Admin Panel](#admin-panel)
- [Important Notes](#important-notes)
- [Troubleshooting](#troubleshooting)
- [Future Enhancements](#future-enhancements)

## Overview
MRMS provides a workflow for:
- registering users (food donors),
- creating food collection requests,
- assigning requests implicitly by area to food collectors,
- allowing collectors to mark requests as completed,
- collecting feedback from both user and collector sides.

The system includes:
- a public-facing landing/about site,
- authentication and account creation,
- role-based dashboards (User and Food Collector),
- an admin interface (with Jazzmin theme) for data management.

## Key Features
- **Role-based accounts** using a custom auth user model (`USER`, `FOOD_COLLECTOR`, `ADMIN`).
- **Food collection requests** with details, address, area, food type, and optional image upload.
- **Area-based routing** of requests to collectors via shared area mapping.
- **Role dashboards** showing pending/completed counts and area metrics.
- **Profile management** for both users and collectors.
- **Feedback submission** for platform improvements from both roles.
- **Django Admin + Jazzmin** for managing users, areas, and requests.

## Tech Stack
- **Backend:** Django
- **Database:** SQLite (`db.sqlite3`)
- **Frontend:** Django templates + Bootstrap/Argon-based static assets
- **Admin UI:** `django-jazzmin`
- **Media handling:** Pillow (`ImageField` uploads for profile/food photos)
- **HTTP integration:** Requests library (used in login gate check)

## Project Structure
```text
Meal-Resource-Management-System/
|- MRMS/                     # Project config, URLs, core views
|- accounts/                 # Custom user model, profiles, auth-related data
|- FCRs/                     # Areas, food requests, feedback models
|- data_transfer/            # Data import command(s)
|- templates/                # Public pages and role-specific dashboards/views
|- static/                   # CSS/JS/assets (landing + dashboard theme)
|- media/                    # Uploaded images (runtime)
|- manage.py
|- requirements.txt
|- copy.json                 # Seed-like source used by add_area command
`- db.sqlite3                # Local SQLite database
```

## Data Model
Core entities:
- `CustomUser` (`accounts`): Email-based authentication, role, phone, gender.
- `UserProfile` (`accounts`): User address, area, profile image.
- `FoodCollectorProfile` (`accounts`): Collector area, salary, profile image.
- `area_master` (`FCRs`): Master list of service areas.
- `FoodCollectionRequests` (`FCRs`): Donation request with status (`Pending`/`Completed`), area, food type, image.
- `feedbacks_master_byUsers` / `feedbacks_master_byFC` (`FCRs`): Feedback records from both role types.

Automatic profile creation is handled using Django `post_save` signals when a `CustomUser` is created.

## User Flows
### Public
- Visit landing page (`/`)
- Read mission/about page (`/about/`)
- Register (`/register/`) or log in (`/login/`)

### User (Donor)
- Log in and open user dashboard (`/index/`)
- Create a food collection request (`/Create_food_collection_request/<email>`)
- View all submitted requests and status (`/User_view_fcrs/<email>`)
- View/edit profile (`/User_profile/<email>`)
- Submit feedback (`/feedbacks_master_byUsers/<email>`)

### Food Collector
- Log in and open collector dashboard (`/index/`)
- View area-specific requests (`/FC_view_fcrs/<email>`)
- Open request details and mark as completed (`/FC_view_fcr/<email>/<id>`)
- View/edit profile (`/FC_profile/<email>`)
- Submit feedback (`/feedbacks_byFC/<email>`)

## Getting Started
### 1) Clone and enter the project
```bash
git clone <your-repository-url>
cd Meal-Resource-Management-System
```

### 2) Create virtual environment
On Windows (PowerShell):
```powershell
python -m venv env
.\env\Scripts\Activate.ps1
```

### 3) Install dependencies
```bash
pip install -r requirements.txt
```

### 4) Run migrations
```bash
python manage.py makemigrations
python manage.py migrate
```

### 5) (Optional) Load area data
If `copy.json` is available in the project root:
```bash
python manage.py add_area
```

### 6) Create an admin user
```bash
python manage.py createsuperuser
```

### 7) Start the development server
```bash
python manage.py runserver
```

Open: [http://127.0.0.1:8000/](http://127.0.0.1:8000/)

## Area Seed Data
The command `python manage.py add_area` reads `copy.json`, extracts each `station_name`, and creates corresponding entries in `area_master` if they do not already exist.

This helps quickly bootstrap the area list used by:
- user registration,
- request creation,
- collector-side filtering.

## Admin Panel
Admin URL: [http://127.0.0.1:8000/admin/](http://127.0.0.1:8000/admin/)

Configured apps in admin include:
- custom users and profiles,
- area master,
- food collection requests,
- user and collector feedback.

The admin uses Jazzmin configuration from `MRMS/settings.py` for improved visual layout.

## Important Notes
- This repository currently includes a local SQLite database file (`db.sqlite3`) and an `env` directory. For clean source control, exclude these in `.gitignore`.
- The login flow calls an external Airtable API as a gate check before authentication. For production, move secrets/tokens to environment variables.
- `DEBUG = True` in settings is suitable only for local development.

## Troubleshooting
- **`ModuleNotFoundError` or missing packages:** Ensure the virtual environment is active, then reinstall with `pip install -r requirements.txt`.
- **Static files not loading:** Confirm `STATIC_URL` and `STATICFILES_DIRS` paths are unchanged and run server from project root.
- **Image upload issues:** Verify the `media/` folder exists and `MEDIA_URL`/`MEDIA_ROOT` are configured.
- **Login blocked unexpectedly:** Check connectivity or response behavior for the external Airtable check in `MRMS/views.py`.

## Future Enhancements
- Replace email-in-URL pattern with ID/session-safe routing patterns.
- Add form/model validation hardening and stronger permission checks.
- Add automated tests for auth, request lifecycle, and role access.
- Move secrets and runtime config to `.env`-based settings.
- Add deployment setup (Gunicorn/Uvicorn + Nginx + PostgreSQL).
