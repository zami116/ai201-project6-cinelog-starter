# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename

**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` so it matches the `add_to_collection()` naming convention used in `services/collection_service.py`. Updated the one caller, `routes/watchlist/watchlist.py`, in both the import line and the call site.
**How I verified:** Ran a project-wide search for `save_to_watchlist` before the change, which found three hits: the definition, the import and the call. Re-ran the same search after the change and got zero matches, so no docs, tests or other modules still use the old name. Then ran `pytest tests/ -v` and all 4 existing tests passed.

## Comment 2 — Deduplication

**What I did:** Added a duplicate check to `add_to_watchlist()` in `services/watchlist_service.py`, following the pattern in `add_to_collection()` in `services/collection_service.py`. That function's check queries `CollectionEntry` with `filter_by(user_id=..., film_id=...).first()`, so it looks for an existing row for the same user and the same film. If one is found, it raises `AlreadyInCollectionError` and does not create or return anything. Otherwise it creates the entry. My version runs the same query against `WatchlistEntry`, placed after the film-exists check and before the new entry is created. When a duplicate is found it raises `AlreadyInCollectionError` and writes nothing. I also updated the POST route in `routes/watchlist/watchlist.py` to catch that error and return HTTP 409 with the message, which is how `routes/collection.py` handles it.
**How I verified:** Ran a script against an in-memory database. It added the same film to the same user's watchlist twice. The second call raised the error and `WatchlistEntry` still had exactly one row. `pytest tests/ -v` still passes (4 tests). `WatchlistEntry` has no unique constraint like `CollectionEntry`'s `unique_user_film_collection`, so this service-level check is the only guard against duplicates.

## Comment 3 — Missing test

**What I did:** Created `tests/test_watchlist.py` with `test_add_to_watchlist_nonexistent_film_raises`. I modelled it on `test_add_to_collection_nonexistent_film_raises` in `tests/test_collection.py`, and copied the same `app` and `sample_user` fixtures, the same `app_context()` wrapper, and the same `pytest.raises(FilmNotFoundError)` assertion. The only change is that it calls `add_to_watchlist()` instead of `add_to_collection()`. It uses the integer 999999 rather than the UUID string in the model test. Film.id is an integer on this branch, and a large number can't collide with a real film if a sample_film is added later.
**How I verified:** Ran `pytest tests/test_watchlist.py -v` and the new test passed. Then ran `pytest tests/ -v` and all 5 tests passed (4 collection, 1 watchlist), so the new file did not break anything.

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