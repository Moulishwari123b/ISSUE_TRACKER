# Django Project Structure - Issue Tracker

## Project Overview
This is a Django-based Issue Tracker application with a React frontend. The project uses Django REST Framework for API endpoints and SQLite for the database.

---

## Directory Structure

```
issue_tracker/
│
├── db.sqlite3                          # SQLite database file
├── manage.py                           # Django management script
├── package.json                        # Node.js dependencies (for frontend tooling)
├── README_SQL_TABLES.md                # SQL tables documentation
│
├── issue_tracker/                      # Main Django project configuration
│   ├── __init__.py                     # Python package marker
│   ├── asgi.py                         # ASGI configuration (async server gateway interface)
│   ├── settings.py                     # Django settings and configuration
│   ├── urls.py                         # Main URL router
│   └── wsgi.py                         # WSGI configuration (web server gateway interface)
│
├── core/                               # Main Django application
│   ├── __init__.py                     # Python package marker
│   ├── admin.py                        # Django admin configuration
│   ├── api_urls.py                     # API endpoint routes
│   ├── api_views.py                    # API view handlers
│   ├── apps.py                         # App configuration
│   ├── forms.py                        # Django forms
│   ├── models.py                       # Database models (Issue, Comment, User, etc.)
│   ├── role_rules.py                   # Role-based access control rules
│   ├── tests.py                        # Unit tests
│   ├── urls.py                         # App-level URL router
│   ├── views.py                        # Standard view handlers
│   │
│   └── migrations/                     # Database migration files
│       ├── __init__.py                 # Python package marker
│       ├── 0001_initial.py             # Initial schema migration
│       ├── 0002_comment_attachment_issue_rejection_reason_and_more.py
│       └── 0003_issue_solution_alter_issue_status.py
│
├── Frontend/                           # React frontend application
│   ├── eslint.config.js                # ESLint configuration
│   ├── index.html                      # HTML entry point
│   ├── package.json                    # Frontend dependencies
│   ├── README.md                       # Frontend documentation
│   ├── vite.config.js                  # Vite bundler configuration
│   │
│   ├── public/                         # Static public assets
│   │
│   └── src/                            # Frontend source code
│       ├── api.js                      # API client/axios configuration
│       ├── App.jsx                     # Main React component
│       ├── App.css                     # App styles
│       ├── index.css                   # Global styles
│       ├── main.jsx                    # React entry point
│       │
│       └── assets/                     # Images, fonts, and other assets
│
└── media/                              # User-uploaded files
    ├── attachments/                    # Issue attachments
    └── comment_attachments/            # Comment attachments
```

---

## File Descriptions

### Django Project Configuration (`issue_tracker/`)
- **settings.py** - Contains all Django settings including database configuration, installed apps, middleware, templates, static files, and secrets
- **urls.py** - Main URL dispatcher that routes requests to the core app
- **wsgi.py** - Entry point for production WSGI servers (Gunicorn, uWSGI)
- **asgi.py** - Entry point for asynchronous servers

### Core Application (`core/`)
- **models.py** - Django ORM models defining the database schema
- **views.py** - Traditional Django view functions (if used for templates)
- **api_views.py** - Django REST Framework view classes for API endpoints
- **urls.py** - URL patterns for template-based views
- **api_urls.py** - URL patterns for REST API endpoints
- **admin.py** - Django admin site configuration for managing models
- **forms.py** - Django forms for form validation and rendering
- **role_rules.py** - Custom permission and role-based access control logic
- **tests.py** - Unit tests for models, views, and API endpoints
- **migrations/** - Database schema change history

### Frontend (`Frontend/`)
- **src/api.js** - Axios client for communicating with Django REST API
- **src/App.jsx** - Main React component and application logic
- **vite.config.js** - Vite build tool configuration
- **index.html** - HTML template for the React app

### Media Files (`media/`)
- **attachments/** - Files uploaded to issues
- **comment_attachments/** - Files uploaded to issue comments

---

## Key Files at Root Level
- **manage.py** - Django command-line utility (run migrations, create superuser, start dev server)
- **db.sqlite3** - SQLite database (local development only)
- **package.json** - Node dependencies for frontend tooling

---

## Quick Reference

### Running the Django Server
```bash
python manage.py runserver
```

### Running Migrations
```bash
python manage.py migrate
```

### Creating a Superuser
```bash
python manage.py createsuperuser
```

### Running Tests
```bash
python manage.py test
```

### Frontend Development
```bash
cd Frontend
npm install
npm run dev
```

---

## Architecture Notes

- **Backend**: Django + Django REST Framework (REST API)
- **Frontend**: React + Vite
- **Database**: SQLite (development), can be configured for PostgreSQL/MySQL (production)
- **Authentication**: Django auth system with custom role-based access control
- **File Uploads**: User uploads stored in `media/` directory with separate folders for different attachment types
