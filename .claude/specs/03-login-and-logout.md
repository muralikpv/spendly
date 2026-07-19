# Spec: Login and Logout

## Overview
This step implements signing in and signing out for Spendly. The `login.html`
template and the `GET /login` route already exist and render the sign-in form,
but submitting it does nothing yet, and `/logout` is still a placeholder string.
This step wires up the `POST /login` handler — looking up the user by email,
verifying the password hash, and starting a session — and implements `/logout`
to clear that session. It also makes the nav aware of session state so a
signed-in user has a visible way to sign out. This builds directly on the
session-based auth introduced in registration and is a prerequisite for any
route that will eventually require a logged-in user (profile, expenses).

## Depends on
- Step 01 — Database Setup (`users` table, `get_db()`, `init_db()`)
- Step 02 — Registration (`get_user_by_email()`, `session["user_id"]` convention,
  password hashing with werkzeug)

## Routes
- `POST /login` — validate email/password against `users`, start session, redirect to `/` — public
- `GET /login` — unchanged, already implemented
- `GET /logout` — clear the session, redirect to `/` — logged-in (safe to hit while logged out too; it just clears an empty session)

## Database changes
No schema changes. No new query functions needed — `get_user_by_email(email)`
from Step 02 is sufficient to look up the row for password verification.

## Templates
- **Create:** none
- **Modify:**
  - `templates/login.html` — none needed; already posts to `/login` and already
    renders `{% if error %}{{ error }}{% endif %}`.
  - `templates/base.html` — nav currently always shows "Sign in" / "Get started"
    regardless of session state. Wrap the nav links in
    `{% if session.get('user_id') %}...{% else %}...{% endif %}` so a signed-in
    user sees a "Sign out" link (pointing at `{{ url_for('logout') }}`) instead
    of "Sign in" / "Get started". This is the only way to reach `/logout` from
    the UI, so it's in scope here.

## Files to change
- `app.py` — replace the `GET`-only `/login` route with one that accepts
  `POST` and handles validation, password verification, and session login;
  replace the placeholder `/logout` route with one that clears the session
  and redirects; add `werkzeug.security.check_password_hash` import.
- `templates/base.html` — conditional nav based on `session.get('user_id')`.

## Files to create
None.

## New dependencies
No new dependencies.

## Rules for implementation
- No SQLAlchemy or ORMs
- Parameterised queries only
- Passwords hashed with werkzeug (`check_password_hash` against the stored
  `password_hash` — never compare plaintext passwords)
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- Validation errors re-render `login.html` with the existing `error` variable —
  do not introduce a new error-passing convention
- Keep all raw SQL in `database/db.py`; `app.py` route handlers call functions,
  they don't execute SQL directly
- Use one generic error message for both "no such user" and "wrong password"
  (e.g. "Invalid email or password") — never reveal which one was wrong
- `/logout` must work even if no session exists (don't error on an already-
  logged-out visitor)
- Do not add a login-required decorator or protect `/profile` or
  `/expenses/*` routes in this step — those remain placeholders and stay out
  of scope until their own steps
- Do not implement real `/profile` content in this step

## Definition of done
- [ ] Submitting the login form with a registered user's correct email/password redirects to `/` and sets a session cookie
- [ ] Submitting with a correct email but wrong password re-renders `login.html` with a generic "Invalid email or password" error and does not set a session
- [ ] Submitting with an email that doesn't exist re-renders `login.html` with the same generic error message (no hint the account doesn't exist)
- [ ] Submitting with a missing email or password re-renders `login.html` with an error message
- [ ] Visiting `/logout` while logged in clears the session and redirects to `/`
- [ ] Visiting `/logout` while already logged out does not error and redirects to `/`
- [ ] After logging in, the nav shows a "Sign out" link instead of "Sign in" / "Get started"
- [ ] After logging out, the nav reverts to showing "Sign in" / "Get started"
- [ ] `GET /login` still renders the form normally with no error
- [ ] App starts without errors and existing routes (`/`, `/register`, `/terms`, `/privacy`) are unaffected
