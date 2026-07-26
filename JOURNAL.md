# Module 3 Journal

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/163

**Issue title:** Review creation does not verify profile ownership

**Tier:** [ ] Tier 1  [x] Tier 2  [ ] Tier 3

**Problem summary:**
The `POST /reviews` endpoint passes the authenticated user's ID down to
`create_review()` in `core/services/review_service.py`, but that function never
actually uses it — it builds a `Review` straight from whatever `profile_id` the
request body supplied. That makes this a broken object-level authorization bug:
anyone who learns another user's profile UUID can create reviews attached to a
profile they don't own, because nothing ties the submitted `profile_id` back to
the caller. The inconsistency is easy to see in the same file, where
`get_review()` and `list_reviews()` both join `Profile` and filter on
`Profile.user_id == user_id` before returning anything. A successful fix makes
`create_review()` enforce that same ownership check — looking up the profile,
confirming it belongs to the authenticated user, and rejecting the request
otherwise — so the write path is scoped exactly like the read paths already are.

**Branch name:** `fix/163-review-profile-ownership`

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [ ] Issue added to cohort ledger

### "Is this right for me?" — selection notes

- **Scope is bounded and legible.** The issue names the exact file and function,
  and the correct behavior already exists in the same module as a working
  reference. I am changing one function plus its tests, not designing anything
  new.
- **I can state the acceptance criteria myself.** Creating a review against a
  profile I don't own must fail with an authorization error; creating one
  against my own profile must keep working unchanged.
- **It is Tier 2 rather than Tier 1 because it crosses modules** — the API
  route, the service layer, and the `Profile`/`Review` models all have to agree
  on where the check lives and what error surfaces to the client. I chose it
  anyway because the cross-module reach is shallow and traceable, and the
  security framing makes it a more substantial thing to reason about and write
  up than a one-line attribute fix.
- **Risk I am watching:** picking the wrong layer for the check. Putting it in
  the route would leave the service insecure for any future caller, so the fix
  belongs in `create_review()` where the existing read-path checks already live.
- **Testing is straightforward.** The repo already has service-level tests, so I
  can add a case for the cross-user rejection alongside the existing happy path.
