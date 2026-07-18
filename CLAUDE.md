# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Spendly is a Flask-based expense tracker, built incrementally as a learning project. The codebase is intentionally scaffolded in stages — many files contain `# Students will implement this in Step N` comments and placeholder route handlers (e.g. `/logout`, `/profile`, `/expenses/add`) that return plain strings like `"Logout — coming in Step 3"` instead of real logic. When working here, check whether a file/route is still a placeholder before assuming it's broken, and implement features in a style consistent with the step-by-step scaffolding rather than jumping ahead to unrelated future steps.

`database/db.py` is the clearest example: it currently only contains a comment describing the three functions it must eventually expose:
- `get_db()` — SQLite connection with `row_factory` and foreign keys enabled
- `init_db()` — creates all tables using `CREATE TABLE IF NOT EXISTS`
- `seed_db()` — inserts sample data for development

## Commands

Run from the project root with the venv active.

```
# Activate virtualenv (Windows)
venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run the dev server (http://127.0.0.1:5001)
python app.py

# Run tests
pytest
```

There is no separate lint/format/build tooling configured (no `pyproject.toml`, `setup.cfg`, or linter config) — don't invent lint commands.

## Architecture

- **`app.py`** — single Flask application module; all routes are defined directly on the module-level `app` object (no blueprints). This is the entry point (`python app.py`, debug mode, port 5001).
- **`database/`** — SQLite access layer, imported by `app.py`. `db.py` is meant to hold all raw SQL/connection logic; keep query code out of `app.py` route handlers.
- **`templates/`** — Jinja2 templates, all extending `templates/base.html`, which defines the shared nav/footer and the `title` / `head` / `content` / `scripts` blocks. Add new pages by extending `base.html` and filling in `content`.
- **`static/css/style.css`** — shared site-wide styles (nav, footer, forms, auth pages). **`static/css/landing.css`** — landing-page-specific styles, loaded only by `landing.html` via its own `head` block.
- **`static/js/main.js`** — single shared JS file included on every page via `base.html`; currently empty, features are added here as they're built.
- Branding: the product name is "Spendly", tagline "Track every rupee. Own your finances." (INR-oriented expense tracking).

## Conventions observed in existing code

- Routes are grouped in `app.py` under `# --- Routes ---` / `# --- Placeholder routes` comment banners — keep new routes similarly organized (implemented vs. not-yet-implemented).
- Auth forms (`login.html`, `register.html`) POST to plain paths (`/login`, not `/api/login`) and expect an `error` template variable for validation feedback (`{% if error %}`).
- No JS framework/bundler is used — templates are server-rendered Jinja2, and any client-side behavior goes directly in `static/js/main.js` with a `<script>` tag, no build step.
