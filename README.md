
# Django + Celery Budget Management System

This project is a backend system for managing advertising budgets and scheduling across brands and campaigns. It supports daily/monthly budget enforcement, dayparting, automatic resets, and background task processing — built with Django, Celery, and SQLite.

---

## Features

- Track daily and monthly spend per brand
- Auto-pause campaigns on budget limit breaches
- Enforce campaign dayparting schedules
- Reset budgets daily and monthly
- Periodic automation using Celery + Beat
- Admin panel for brand/campaign management
- Static typing with `mypy`
- Simulated spend management command

---

## Tech Stack

- Django (ORM, admin, commands)
- Celery (task queue)
- Redis (Celery broker)
- SQLite (development DB)
- Python typing (`mypy`)

---

## Setup Instructions

> All logic and boilerplate are preconfigured. Just install, migrate, and run.

### 1. Clone the repo & setup virtual environment

```bash
git clone https://github.com/rafaelsalmoncs/Campaign-Budget-Tracker.git
cd ad_budget_project
python -m venv venv
source venv/bin/activate
```

### 2. Install dependencies

```
bash
pip install -r requirements.txt
```

### 3. Apply database migrations

```bash
python manage.py makemigrations
python manage.py migrate
```

### 4. Create admin user (optional)

```bash
python manage.py createsuperuser
```

### 5. Run Django server

```bash
python manage.py runserver
```

### 6. Start Celery workers and beat scheduler (in two separate terminals)

```bash
celery -A ad_budget_project worker --loglevel=info
celery -A ad_budget_project beat --loglevel=info
```

---

##  Simulate Spend Command

```bash
python manage.py simulate_spend <campaign_id> <amount>
```

This increases campaign and brand spend and auto-disables campaigns if budget thresholds are breached.

E.g.

```bash
python manage.py simulate_spend 1 75.00
```

---

## Admin Panel

Visit [http://localhost:8000/admin](http://localhost:8000/admin)

Manage:
- Brands
- Campaigns
- Dayparting schedules

Filters, search, and list views are preconfigured.

---

## Data Model Overview

### Brand
- `daily_budget`, `monthly_budget`
- `current_daily_spend`, `current_monthly_spend`
- `is_active`

### Campaign
- Belongs to a `Brand`
- Has a `Schedule` for allowed hours
- Tracks `daily_spend`
- Auto-activates/deactivates

### Schedule
- Defines active hours via `start_hour` / `end_hour`
- Defines dayparting window

---

## System Workflow

- Celery tasks run periodically to:
  - Enforce budget limits (`every 5 min`)
  - Enforce dayparting (`hourly`)
  - Reset daily budgets (`midnight`)
  - Reset monthly budgets (`1st of month`)
- Spend is simulated via management command
- Campaigns deactivate if budgets are exceeded or time is out of range

---

## Celery Tasks

| Task | Frequency |
|------|-----------|
| Budget enforcement | Every 5 minutes |
| Dayparting | Hourly |
| Reset daily spend | Midnight |
| Reset monthly spend | 1st of each month |

Configured via `CELERY_BEAT_SCHEDULE` in `settings.py`. Task schedule frequencies can be adjusted in settings.py under CELERY_BEAT_SCHEDULE.

---

## Static Typing

- Type hints used in:
  - Models
  - Celery tasks
  - Command handlers
- All logic is statically typed
- `mypy.ini` included for strict checks
- Run: `mypy .`

---

## Clarifications and version notes

- The project tries to get an environment variable and uses a placeholder SECRET_KEY if it is not found for simplicity. In production, this should be set via an environment variable.
- No project setup steps (e.g. file creation, config insertion) are required. All required files and folder structures are included in this repo.
- Admin and command logic is implemented and ready to run out-of-the-box. A single management command (`simulate_spend`) accurately demonstrates core logic in a testable way.
- Admin panel uses Django’s built-in admin system with `list_display`, `search_fields`, and `list_filter` for effective backend management — no custom views were needed.
- DEBUG is set to True for development/testing purposes only.
- Requires Redis running on localhost:6379 (default port).

---

## Project/Repo Structure

```
ad_budget_project/
├── ads/
│   ├── models.py
│   ├── admin.py
│   ├── tasks.py
│   └── management/
│       └── commands/
│           └── simulate_spend.py
├── ad_budget_project/
│   ├── __init__.py
│   ├── settings.py
│   ├── celery.py
├── db.sqlite3  # (created after first run)
├── manage.py
├── requirements.txt
├── mypy.ini
└── README.md
```

---

## Assumptions & Simplifications

This implementation intentionally balances completeness with clarity. The following assumptions and simplifications were made:

### Model Design

* **One schedule per campaign**
  Each `Campaign` is linked to a single `Schedule` model (one-to-one or nullable foreign key). No support for multiple or recurring time windows per day.

* **Flat budgets per brand**
  Brands have a single daily and monthly budget. There's no support for campaign-specific budgets or budget tiers.

* **Spend is externally simulated**
  Ad impressions or costs are not tracked automatically. Spend is manually simulated via a management command (`simulate_spend`) to test budget logic.

### Celery Tasks

* **Tasks are periodic, not event-driven**
  Budget enforcement and dayparting checks run on a schedule (via Celery Beat). Real-time enforcement (e.g., per spend event) is not implemented for simplicity.

* **Idempotent logic**
  Tasks are written to be safely repeatable without double-counting or duplicating state changes.

### Data and Time Assumptions

* **Time zones are treated uniformly (UTC)**
  All datetime logic assumes UTC. No localization or timezone-aware schedules per campaign or user.

* **Current hour logic for dayparting**
  Dayparting compares only the current hour (`0–23`) without accounting for minutes or partial overlaps.

### Storage

* **SQLite is used for simplicity**
  While the system is compatible with PostgreSQL, SQLite is used for development ease and portability.

---

## Status

Fully functioning and testable Django + Celery backend system.