# Review use cases

### UC-REV-01 Customer rates a stadium they played at
- **Actor:** Customer
- **Preconditions:** Signed in; a completed booking (or a confirmed booking that has ended) at this stadium; has never rated it.
- **Main flow:** Stadium page → Reviews → choose 1–5 stars, write a comment → `POST /api/stadiums/{code}/reviews` → saved, stadium average and count recomputed, manager notified → page reloads at `#reviews` showing the review; the form is replaced by "You have already rated this stadium."
- **Failure flows:** Never played → the form is not shown; calling the API anyway → 403 `REVIEW_NOT_ALLOWED`. Second attempt → 409 `REVIEW_ALREADY_EXISTS`. Bad rating / short comment → 422.

### UC-REV-02 Read reviews
- **Actors:** Signed-in users; guests
- **Main flow:** Signed-in users see the newest 5 and "Show more reviews". Guests see only the average and count, with the comments locked behind "Sign in to read reviews"; `GET /api/stadiums/{code}/reviews` returns 401 for guests.

### UC-REV-03 Moderate a review
- **Actor:** Manager of the stadium (`reviews.moderate`), admin
- **Main flow:** Remove button → confirm → soft delete → rating recomputed.
- **Rule:** The author cannot rate the stadium again after a removal (one rating per customer per stadium).
- **Failure flows:** Another manager's stadium → 404.
