# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This is a Flask personal expense tracker, branded **Spendly** in the UI (GitHub repo name: `spendify`), aimed at an Indian audience (amounts shown in ₹). It is deliberately structured as a **teaching scaffold, not a finished app** — most backend functionality is intentionally left as numbered steps for a student to implement.

## Commands

Run all commands from the repo root (`/Users/satyam/Desktop/expense-tracker`), using the venv directly rather than relying on shell activation persisting across tool calls:

```bash
venv/bin/pip install -r requirements.txt
venv/bin/python app.py          # runs on http://localhost:5001, debug=True
venv/bin/pytest                 # test suite (no test files exist yet)
```

There is no build step or linter configured.

## Architecture

- **`app.py`** — single-file Flask app with all routes. Currently only `/`, `/register`, `/login`, `/terms`, `/privacy` render templates directly (no form handling, auth, or DB access yet). Routes for `/logout`, `/profile`, `/expenses/add`, `/expenses/<id>/edit`, `/expenses/<id>/delete` are placeholders returning plain strings labeled with their intended implementation step (e.g. "Step 3", "Step 7") — implement these in place rather than restructuring the file.
- **`database/db.py`** — currently just a spec comment, not implemented. It is expected to define `get_db()` (SQLite connection with `row_factory` and foreign keys enabled), `init_db()` (creates tables with `CREATE TABLE IF NOT EXISTS`), and `seed_db()` (sample data for dev). No ORM is used — raw SQLite via `sqlite3`.
- **`templates/`** — Jinja2 templates. `base.html` is the shared shell (nav, footer, `{% block title/head/content/scripts %}`); page templates `{% extends "base.html" %}`. Footer links (Terms, Privacy) use `url_for()` to named routes. Match `base.html`'s structure when adding new pages rather than duplicating the shell.
- **`static/css/style.css`** — shared/base styles (nav, footer, general layout). **`static/css/landing.css`** — landing-page-specific styles, loaded only via `landing.html`'s `{% block head %}`. Follow this pattern for future page-specific styling: shared rules in `style.css`, page-specific rules in their own file loaded per-template.
- **`static/js/main.js`** — single shared JS file for all pages; **vanilla JS only, no frameworks or libraries** (established convention for this project — e.g. the "how it works" modal is plain JS with open/close/outside-click/stop-video-on-close behavior, no dependencies).
- **`database/expense_tracker.db`** (when created) and `__pycache__/`, `venv/`, `.DS_Store` are gitignored.

## Working conventions

- When asked to add a page or feature, scope changes tightly to what's requested — these prompts have historically been narrow ("do not modify anything else on the pages"). Don't refactor unrelated templates/routes while adding something new.
- New routes follow the existing pattern in `app.py`: simple `@app.route` + `render_template`, grouped near the other real (non-placeholder) routes.
