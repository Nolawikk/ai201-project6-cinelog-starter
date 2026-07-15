# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end -->

## Comment 1 — Rename
**What I did:**
Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` to match the project's `verb_to_noun` naming convention already used by `add_to_collection()`, `remove_from_collection()`, and `get_collection()` in `collection_service.py`.

**How I verified:**
Ran `git grep save_to_watchlist` across the whole repo to confirm no references to the old name remained (it returned nothing after updating both the function definition and its one call site in `routes/watchlist/watchlist.py`). Also ran `pytest tests/ -v` to confirm the full suite still passed.

## Comment 2 — Deduplication
**What I did:**
Added a duplicate check to `add_to_watchlist()`, mirroring the pattern in `add_to_collection()`: after confirming the film exists, I query for an existing `WatchlistEntry` with the same `user_id` and `film_id`. If one exists, I raise a new `AlreadyInWatchlistError` instead of silently creating a duplicate row.

**How I verified:**
Ran `pytest tests/ -v` — all existing tests still pass. (I'll add a dedicated test for this in Comment 3/the stretch goal.)

## Comment 3 — Missing test
**What I did:**
Created `tests/test_watchlist.py` and wrote `test_add_to_watchlist_nonexistent_film_raises`, modeled directly on `test_add_to_collection_nonexistent_film_raises` in `test_collection.py` — same fixture structure (`app`, `sample_user`, `sample_film`), same use of `pytest.raises(FilmNotFoundError)` with a fake UUID-style film_id.

**How I verified:**
Ran `pytest tests/test_watchlist.py -v` in isolation (passed), then ran the full suite with `pytest tests/ -v` to confirm nothing else broke.

## Comment 4 — Default visibility
**My position:**
`WatchlistEntry.public` should default to `True`, with users able to opt out and make individual entries (or their account) private whenever they choose.

**Reasoning:**
Most users aren't thinking about privacy in the moment when they casually add a movie to their watchlist — they're just marking something they want to watch. Defaulting to public keeps the social side of the feature alive: friends can see what you're planning to watch without either person having to dig through settings first. If the default were private, the feature would likely go unused socially, since most users never revisit a default they didn't consciously set.

**Tradeoff acknowledged:**
The real cost of a public default is that some users will end up sharing their watchlist without having consciously decided to. I think this is an acceptable tradeoff here specifically because a watchlist reveals intent, not confirmed behavior — it's a lower-stakes signal than, say, a viewing history or rating. Combined with the fact that an opt-out is always available, I don't think the risk outweighs the benefit to the social feature.