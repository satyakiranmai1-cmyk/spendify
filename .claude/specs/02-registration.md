Spec Document

1. Overview
Implement account creation: handle POST submissions from the existing register.html
form in app.py, validate input, store a new user in the database, and route the
user onward. This is the first authentication step; login/session handling
(Step 3) builds on top of it.

2. Depends on
Step 1 — Database setup (database/db.py: get_db, users table). Must be complete.

3. Routes
Modify existing route in app.py:
- GET /register  → renders register.html (unchanged behavior)
- POST /register → validates form data, creates user, redirects or re-renders
                    register.html with an error

No new routes are added. No changes to /login, /logout, or any other route.

4. Form Fields (templates/register.html)
- name             (text, required)
- email            (email, required)
- password         (password, required, min. 8 characters per placeholder text)
- confirm_password (password, required — must match `password`)

5. Validation Rules
- All four fields must be non-empty (after stripping whitespace)
- Password must be at least 8 characters
- password and confirm_password must match
- Email must not already exist in the users table
- On any validation failure:
  - Re-render register.html with status 400
  - Pass the specific failure message via the existing `error` template variable
  - Do not leak whether a raw SQL/database error occurred — show a generic
    "That email is already registered" for duplicate email

6. Success Behavior
- Hash the password with werkzeug.security.generate_password_hash
- Insert the new user via get_db() using a parameterized INSERT
- On success, redirect (302) to the login route (url_for("login"))
- Do not log the user in automatically — Step 3 (login/session) owns
  establishing a session

7. Functions to Implement (app.py)
Modify the existing register() view to branch on request.method:
- GET  → return render_template("register.html")
- POST → run validation → on failure: render_template("register.html", error=...)
       → on success: insert user, redirect to url_for("login")

Use Flask's `request` object (import from flask alongside existing imports).
No new helper modules required — keep logic inside the register() view, following
the single-file app.py convention already established.

8. Files to Change
- app.py → update register() to accept GET and POST, add form handling logic

9. Files to Create
None — register.html and the users table already exist.

10. Dependencies
No new pip packages. Use:
- flask.request (already available via existing flask import)
- werkzeug.security.generate_password_hash (already used in database/db.py)
- database.db.get_db (already implemented)

11. Rules for Implementation
- @app.route("/register", methods=["GET", "POST"]) — do not create a second route
- Use parameterized queries only — never string-format SQL
- Never store or log plaintext passwords
- register.html has a confirm_password field alongside password — do not otherwise
  modify templates/register.html, base.html, or any CSS
- Do not implement login/session logic here — only account creation
- Do not add flash messaging or new dependencies (e.g. Flask-WTF) — keep the
  existing `error` variable pattern used by the template

12. Expected Behavior
- Submitting valid, unique details creates a row in `users` and redirects to
  /login
- Submitting a duplicate email re-renders the register form with an error
  and does not create a row
- Submitting a short password, mismatched confirm_password, or blank field
  re-renders the register form with an error and does not create a row
- Refreshing after a successful registration does not resubmit the form
  (guaranteed by the redirect)

13. Error Handling Expectations
- Duplicate email (UNIQUE constraint violation) → caught, shown as a friendly
  error, not a raw 500/traceback
- Missing/blank fields → caught before hitting the database
- Password under 8 characters → caught before hitting the database

14. Definition of Done
[ ] POST /register creates a user with a hashed password
[ ] Duplicate email is rejected with a clear error, no server crash
[ ] Blank fields are rejected with a clear error
[ ] Password under 8 characters is rejected with a clear error
[ ] Mismatched password/confirm_password is rejected with a clear error
[ ] Successful registration redirects to /login
[ ] register.html's existing `error` block is reused, with a confirm_password
    field added alongside password
[ ] No new routes, templates, or dependencies introduced
[ ] All queries use parameterized SQL
