# ReviewService — `services/Reviews/ReviewService.php`

### `list(Stadium $stadium, int $page): PageResult`
Visible reviews, newest first, 5 per page (`ReviewRules::PAGE_SIZE`). Only reachable by signed-in users (route `auth`).

### `eligibility(User $user, Stadium $stadium): ReviewEligibility`
| Order | Check | Result |
| --- | --- | --- |
| 1 | Role lacks `reviews.write` | `not_a_customer` |
| 2 | A review by this customer exists for the stadium, **including a removed one** | `already_reviewed` |
| 3 | Customer has a `completed` booking, or a `confirmed` one whose `end_at` passed | `can_review` |
| 4 | Otherwise | `not_played` |

The stadium page renders the form only for `can_review`.

### `create(User $user, Stadium $stadium, ReviewFormDto $dto): Review`
**Logic:** 1) re-check eligibility on the server (a hidden form opened by hand still fails); 2) transaction: insert + recompute `stadiums.rating_average/rating_count`; 3) notify staff (`review.created`).

| Case | Result |
| --- | --- |
| Played, first review | 201 |
| Second attempt (also after removal) | 409 `REVIEW_ALREADY_EXISTS` |
| Never played here | 403 `REVIEW_NOT_ALLOWED` |
| Staff role | 403 `FORBIDDEN` from `can:reviews.write` (or `REVIEW_NOT_ALLOWED` if granted without being a customer) |
| Two requests at once | The `UNIQUE(stadium_id, customer_id)` violation is mapped to 409 `REVIEW_ALREADY_EXISTS` |
| Rating outside 1–5, comment < 3 chars | 422 |

### `delete(User $user, string $code): void`
Moderation only: `reviews.moderate` **and** manages the stadium (admins: any). Soft-deletes and recomputes the rating. Authors cannot delete their own review (they could never rate again). Out of scope → 404.
