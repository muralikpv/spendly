# Spec: Registration

## Overview
This step implements account creation for Spendly. The `register.html` template and
the `GET /register` route already exist and render the sign-up form, but submitting
it does nothing yet. This step wires up the `POST /register` handler: validating
input, hashing the password, inserting the new user into the `users` table, and
starting a session so the user is signed in immediately after registering. This is
the first step that introduces authenticated state, which later steps (login,
logout, profile) will build on.

## Depends on
- Step 01 — Database Setup (`users` table, `get_db()`, `init_db()` must exist and work)

## Routes
- `POST /register` — validate and create a new account, log the user in, redirect to `/profile` — public
- `GET /register` — unchanged, already implemented

## Database changes
No schema changes. The existing `users` table (`id`, `name`, `email`, `password_hash`,
`created_at`) already has everything registration needs.

New query functions needed in `database/db.py` (no schema change, just data access):
- `get_user_by_email(email)` — returns a user row or `None`, used to check for duplicate emails
- `create_user(name, email, password_hash)` — inserts a new user, returns the new user's id

## Templates
- **Create:** none
- **Modify:** none — `register.html` already posts to `/register` and already renders
  `{% if error %}{{ error }}{% endif %}`, which is all the new route needs.

## Files to change
- `app.py` — add `app.secret_key`, add `import` for the new `flask.session` and
  `werkzeug.security.generate_password_hash` usage, replace the `GET`-only
  `/register` route with one that accepts `POST` and handles validation, user
  creation, and session login.
- `database/db.py` — add `get_user_by_email(email)` and `create_user(name, email, password_hash)`.

## Files to create
None.

## New dependencies
No new dependencies.

## Rules for implementation
- No SQLAlchemy or ORMs
- Parameterised queries only
- Passwords hashed with werkzeug (`generate_password_hash` / `check_password_hash`)
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- Validation errors re-render `register.html` with the existing `error` variable —
  do not introduce a new error-passing convention
- Keep all raw SQL in `database/db.py`; `app.py` route handlers call functions, they
  don't execute SQL directly
- Validate on the server even though the form has `required`/`type="email"` HTML
  attributes — those are not trustworthy on their own
- Minimum password length is 8 characters, matching the form's placeholder text
  ("Min. 8 characters")
- Duplicate email must be rejected with a clear error message, not a raw SQLite
  `IntegrityError`
- Do not implement `/login` POST handling, `/logout`, or `/profile` in this step —
  those remain out of scope (later steps)

## Definition of done
- [ ] Submitting the register form with a new name/email/password creates a row in `users` with a hashed password (not plaintext)
- [ ] After successful registration, the browser is redirected to `/profile` and a session cookie is set
- [ ] Submitting with an email that already exists in `users` re-renders `register.html` with an error message and does not insert a duplicate row
- [ ] Submitting with a password under 8 characters re-renders `register.html` with an error message and does not insert a row
- [ ] Submitting with a missing name, email, or password re-renders `register.html` with an error message and does not insert a row
- [ ] `GET /register` still renders the form normally with no error
- [ ] App starts without errors and existing routes (`/`, `/login`, `/terms`, `/privacy`) are unaffected
