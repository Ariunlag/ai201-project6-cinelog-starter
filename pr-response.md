# PR Response Doc — CineLog Watchlist Feature

## AI Usage
I used AI as a devil's advocate for my drafts on default visibility and sort order. I asked what a careful reviewer would challenge and which tradeoffs I had missed. The critique identified two real gaps: changing the visibility default alone does not provide privacy while CineLog's read endpoint ignores the flag, and newest-first sorting can bury older films indefinitely. I revised my responses to make privacy enforcement part of the recommendation and to acknowledge recency bias and the longer-term value of user-controlled sorting or priority.

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
**My position:** Watchlist entries should default to `public=False`, with users explicitly choosing to make them public.
**Reasoning:** CineLog is a community film-tracking app, but a watchlist is also personal planning data: users may save guilty pleasures, unfinished research, or films they simply are not ready to share. I am optimizing for users to save freely without first evaluating the social meaning of every addition. Privacy is difficult to restore after an entry has been exposed, while a private entry can safely be published later. However, changing the database default is not sufficient by itself. CineLog currently lets a watchlist be requested by user ID, and `get_watchlist()` does not filter on `public` or verify ownership. I would pair the private default with owner-aware authorization, public-only filtering for other viewers, and an explicit way to publish an entry; otherwise `public=False` would create a misleading promise of privacy.
**Tradeoff acknowledged:** Keeping `public=True` better supports CineLog's community side: profiles become useful for film discovery immediately, and users do not have to publish every saved film manually. A private default will reduce the amount of shared content because some users will overlook or decline the opt-in, and it requires additional API and UI work before sharing is convenient. I still favor explicit consent because saving a film is not necessarily the same action as recommending it publicly, but the publishing control must be easy to find so privacy does not make the social feature feel empty.

## Comment 5 — Sort order
**My position:** I would change the watchlist to sort by `WatchlistEntry.date_added.desc()`, so the most recently saved films appear first.
**Reasoning:** A CineLog watchlist behaves more like an inbox of film discoveries than a reference catalog. Users are likely to return soon after hearing about a film, so newest-first preserves that recent context and puts the latest intent at the top. It also matches the existing collection endpoint's ordering, making the two lists behave consistently. Alphabetical order is predictable when a user already knows a title, but search or an explicit alphabetical option would serve that task better than making it the only ordering.
**Engagement with reviewer's point:** I agree with the maintainer that date-added order better reflects how users build a watchlist: a newly saved film should not disappear into an unrelated position determined by its title. A real downside is recency bias—frequent additions can push older films down indefinitely, even though those older entries may be the ones the user most wants to stop postponing. Date added also is not true priority; only the user knows what they intend to watch next. I would still use newest-first as the initial default because it reflects observable activity without inventing priority, then add selectable alphabetical/oldest-first sorting or manual priority if CineLog develops the watchlist into a planning tool.

## Comment 6 — Rebase
**What conflicted:** The watchlist branch was based on the pre-refactor schema, where film IDs were integers, while `main` had migrated `Film.id` and related foreign keys to UUID strings. The rebase also produced an add/add conflict in `.gitignore` because both branches introduced generated-file rules.
**How I resolved it:** I restored `WatchlistEntry` on top of the current `main` model and changed its `film_id` column to `db.String(36)` with a foreign key to `film.id`. I updated the watchlist service and route documentation to describe `film_id` as a UUID, retained the relationships used when serializing watchlist films, and combined the `.gitignore` rules so main's environment/database exclusions and the Python cache exclusions are preserved.
**How I verified no conflict remains:** I searched the watchlist code for remaining integer film-ID references, confirmed both collection and watchlist `film_id` columns use `db.String(36)`, and ran `pytest tests/ -v` with all 5 tests passing. The rebase completed with a clean working tree, and `git log --merges --oneline origin/main..HEAD` returned no commits, confirming the feature branch adds no merge commits.

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->
