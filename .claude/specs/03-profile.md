Spec Document

1. Overview
Implement the profile page: replace the placeholder GET /profile route in app.py
with a real view that shows the signed-in user's account details and a summary
of their expenses. This is the first route to depend on the session established
by login (Step 3). Editing profile fields, changing password, and logout are
separate, later steps and are not part of this spec.

2. Depends on
- Step 1 — Database setup (database/db.py: get_db, users + expenses tables)
- Login/session handling already implemented in app.py's /login route, which
  sets session["user_id"] on successful sign-in. Must be complete.

3. Routes
Modify existing route in app.py:
- GET /profile → if no authenticated session, redirect (302) to /login
                → else fetch the user and their expense summary, render
                  profile.html

No new routes are added. No changes to /login, /register, /logout, or any
other route. Do not implement /logout in this spec — the "Sign out" link
in the template may point to url_for("logout") even though that route is
still a Step 3 placeholder.

4. Data Needed (from existing tables, no schema changes)
From `users` (looked up by session["user_id"]):
- name
- email
- created_at ("Member since")

From `expenses` (looked up by user_id = session["user_id"]):
- Total spent: SUM(amount)
- Total number of expenses: COUNT(*)
- Per-category totals: SUM(amount) GROUP BY category, for a simple breakdown
  (categories are the fixed list already used by database/db.py's seed data:
  Food, Transport, Bills, Health, Entertainment, Shopping, Other)

5. Auth Guard
- Check `session.get("user_id")` at the top of the profile() view.
- If missing, redirect (302) to url_for("login") — do not render an error
  page or expose any profile data.
- If present but the user_id no longer matches a row in `users` (edge case),
  clear the session and redirect to url_for("login") the same way.

6. Functions to Implement (app.py)
Modify the existing profile() view:
- Guard clause as described in section 5.
- db = get_db(); fetch the user row by id.
- Fetch expense summary with parameterized queries (SUM/COUNT/GROUP BY),
  scoped to the current user_id only.
- Pass user, total_spent, expense_count, and category_totals into
  render_template("profile.html", ...).
- Close the db connection before returning.
- Handle the "no expenses yet" case (new user, empty expenses table for
  them): total_spent should show ₹0, expense_count 0, category_totals empty
  — no division-by-zero or None-formatting errors.

7. Files to Create
- templates/profile.html — extends base.html, following the structure used
  by register.html/login.html (a card-based section under {% block content %}).
  Sections:
  - Header: user's name, email, "Member since <date>"
  - Summary stats: total spent (₹), number of expenses logged
  - Category breakdown: simple list or bar per category with its total
    (reuse the visual language of the landing page's mock category bars,
    but as page-specific markup/CSS — do not touch landing.html or
    landing.css)
  - A "Sign out" link pointing to url_for("logout")
- static/css/profile.css — page-specific styles for the above, loaded only
  via profile.html's {% block head %}, following the landing.css pattern
  described in CLAUDE.md. Shared nav/footer/card basics stay in style.css;
  only profile-specific layout (stat row, category bars) goes here.

8. Files to Change
- app.py → replace the placeholder profile() view with the real
  implementation described in sections 5–6.

9. Dependencies
No new pip packages. Use:
- flask.session (already imported/used by the login route)
- database.db.get_db (already implemented)
- Currency formatting: format amounts as ₹ with two decimals in the
  template (Jinja `"%.2f"|format(value)`), matching the ₹ convention used
  elsewhere in the UI. No new formatting library.

10. Rules for Implementation
- @app.route("/profile") stays GET-only — no form submission on this page
- Use parameterized queries only — never string-format SQL
- Scope every expense query to the logged-in user's id — never return
  another user's data
- Do not modify base.html, landing.html, landing.css, register.html,
  login.html, or style.css beyond what's already there
- Do not implement logout, password change, or profile editing here
- Do not add flash messaging or new dependencies

11. Expected Behavior
- Visiting /profile while logged in (valid session) shows name, email,
  member-since date, total spent, expense count, and a per-category
  breakdown for that user only
- Visiting /profile while logged out redirects to /login
- A logged-in user with zero expenses sees ₹0 / 0 expenses and no category
  breakdown rows, without errors
- The "Sign out" link is present and points at /logout (link only — the
  route's actual behavior is out of scope until Step 3 is implemented)

12. Error Handling Expectations
- No session / invalid session → redirect, never a 500 or stack trace
- Empty expense history → handled gracefully (₹0, 0 count), not an
  exception from SUM()/GROUP BY returning NULL/empty

13. Definition of Done
[ ] GET /profile redirects to /login when there is no valid session
[ ] GET /profile renders profile.html with the correct user's name, email,
    and member-since date when logged in
[ ] Total spent and expense count are correct and scoped to that user only
[ ] Category breakdown reflects that user's expenses only, grouped and
    summed correctly
[ ] New users with no expenses see zero-state values, no errors
[ ] templates/profile.html and static/css/profile.css created, following
    existing template/CSS conventions
[ ] No other routes, templates, or shared CSS/JS files modified
[ ] All queries use parameterized SQL
