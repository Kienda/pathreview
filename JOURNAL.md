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

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** [9d6a50b](https://github.com/Kienda/pathreview/commit/9d6a50b76cbbf3ce4798b3de0842ee648938475f)

**Reproduction summary:**
I configured the review service test as if an ownership-scoped profile lookup found no
profile for the current user, then called `create_review()` with another user's profile
ID. The test fails because the service still creates and returns a pending review
instead of returning `None` without writing to the database.

**PLAN.md link:** [PLAN.md](https://github.com/Kienda/pathreview/blob/fix/163-review-profile-ownership/PLAN.md)

**Walkthrough video (recommended):** Not recorded.

**Blockers or open questions:**
The current test suite has no endpoint integration-test harness, so I still need to
decide whether Week 9 route-level coverage belongs in `tests/security/` or should use a
smaller mocked route test alongside the service regression.

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
I implemented the ownership-scoped profile lookup in `create_review()`, added the route's
404 response for missing or unowned profiles, and prevented background processing from
being scheduled after rejection. The focused service and route tests pass.

**Next steps:**
Open the pull request, request peer or mentor feedback, address any applicable review
comments, and complete Check-in 2 with the final PR link and validation results.

**Blockers:**
The repository-wide suite currently has unrelated pre-existing failures, and GNU Make
is not installed in the local PowerShell environment. I ran the underlying pytest
command directly and confirmed the seven review-creation tests pass.

---

### Check-in 2 (end of week)

To be completed when the pull request is finalized.
