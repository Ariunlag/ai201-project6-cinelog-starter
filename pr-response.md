# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` to follow the project's `verb_to_noun` naming convention. I also updated the import and function call in `routes/watchlist/watchlist.py`.

**How I verified:** Ran a project-wide search for both function names. The only `add_to_watchlist` references are the service definition, route import, and route call, and there are no remaining `save_to_watchlist` references.

## Comment 2 — Deduplication
**What I did:** Added an `AlreadyInWatchlistError` and updated `add_to_watchlist()` to query for an existing entry with the same `user_id` and `film_id`. If one exists, the function raises the new exception instead of inserting a duplicate.
**How I verified:** Compared the implementation with the established deduplication pattern in `add_to_collection()` and confirmed the duplicate check runs before `db.session.add()` and `db.session.commit()`.

## Comment 3 — Missing test
**What I did:** Added `tests/test_watchlist.py` with `test_add_to_watchlist_nonexistent_film_raises`, following the same fixtures and `pytest.raises(FilmNotFoundError)` structure as the equivalent collection service test.
**How I verified:** Ran `pytest tests/test_watchlist.py -v`; the test passed (`1 passed`).

## Comment 4 — Default visibility
**My position:**
**Reasoning:**
**Tradeoff acknowledged:**

## Comment 5 — Sort order
**My position:**
**Reasoning:**
**Engagement with reviewer's point:**

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->
