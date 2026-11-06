# Users use cases

### UC-USR-01 Customer registers
- **Actor:** Guest
- **Preconditions:** Not signed in.
- **Main flow:** Guest fills full name, username, email, phone, password → AJAX `POST /api/auth/register` → server validates and checks duplicates → user saved with role `customer` → session started → redirected to the dashboard.
- **Failure flows:** Invalid fields → 422 with per-field errors shown under each input; duplicate username/email/phone → 409 field error. Duplicates are detected regardless of letter case or phone formatting (`Ali` = `ali`, `+973 3300-0001` = `+97333000001`).
- **Alternate flow:** Opened from a locked stadium page (`/register?next=/stadiums/{code}`) → returns to that stadium after registering.
- **Result:** Active customer account, signed in.

### UC-USR-02 User signs in
- **Actor:** Any user
- **Preconditions:** Active account.
- **Main flow:** Username or email + password → `POST /api/auth/login` → password verified → session id regenerated → `last_login_at` set → redirect to dashboard.
- **Failure flows:** Wrong credentials or inactive account → 401 generic message; 5 failures in 15 minutes → 429.
- **Result:** Signed-in session with role.

### UC-USR-03 User signs out
- **Main flow:** `POST /api/auth/logout` (CSRF-protected) → session destroyed → redirected to sign-in.

### UC-USR-04 Edit profile
- **Actor:** Any signed-in user
- **Main flow:** Update full name, email, phone, gender, birth date → `PUT /api/profile` → duplicates checked excluding self → saved.
- **Failure flows:** 422 validation, 409 duplicate.
- **Result:** Profile updated; role and username cannot be changed here.

### UC-USR-05 Change password
- **Main flow:** Current password + new password + confirmation → `PUT /api/profile/password` → current verified → new hash stored.
- **Failure flows:** Wrong current password → 422 on `currentPassword`; weak password → 422.

### UC-USR-06 Admin creates a user
- **Actor:** Admin
- **Main flow:** Fill user form with role (admin/manager/customer) and initial password → `POST /api/users` → saved, `created_by` = admin.
- **Result:** New user can sign in; managers become selectable on stadiums.

### UC-USR-07 Admin edits or deactivates a user
- **Actor:** Admin
- **Main flow:** `PUT /api/users/{code}` or `POST /api/users/{code}/toggle`.
- **Failure flows:** Admin targets themself for deactivation or role change → 409 `SELF_ACTION_FORBIDDEN`; demoting a manager who still manages stadiums → 409 `MANAGER_HAS_STADIUMS`.
- **Result:** Deactivated users cannot sign in; existing sessions are rejected on the next request.
