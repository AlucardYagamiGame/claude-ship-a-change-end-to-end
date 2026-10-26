# NOTES — `PUT /users/:id`

## Plan

The plan touched exactly two source files: add an `updateUser(id, { name, email })`
helper to `db/store.js` (returns the updated user, or `undefined` for an unknown
id), then add a `PUT /users/:id` route in `routes/users.js` that uses it. The
route checks existence first (404, matching `GET /users/:id`), then validates
the body (400), then updates and returns 200. Before approving I tightened the
validation beyond the tests' "missing field" case: `name` must be a non-blank
string and `email` must look like an email address, so wrong types or empty
strings are rejected too, not just absent fields.

## Model

Claude Opus 5.5. The change is small, but it has to fit existing conventions
and get the edge cases (not-found vs. invalid input, non-numeric ids) right on
the first pass, and a stronger model costs little on a change this size.

## Commits

1. `Add updateUser helper to the in-memory store` — data layer only.
2. `Add PUT /users/:id endpoint to update a user` — the route and validation.
3. `Add NOTES.md` — this write-up.

Split by layer so each commit is self-contained and reviewable on its own: the
store helper is useful and correct without the route, and the route commit
shows only HTTP behaviour.

## Review

The review confirmed the not-found path is safe: a non-numeric id becomes
`NaN`, matches no user, and returns 404 instead of crashing; `req.body` is
guarded so a request without a JSON body yields 400, not a `TypeError`. It
flagged one redundant step — trimming `email` after a regex that already
forbids whitespace — which I removed. Known limitation, left as-is on purpose:
duplicate emails are not rejected, since `POST /users` doesn't reject them
either and changing that is outside this feature's scope.
