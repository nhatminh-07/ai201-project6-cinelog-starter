# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->
I use Claude Code to mostly testing the project findings and make sure there is no errors. Additionally, I test the function to see no potential bugs in this program.

## Comment 1 — Rename
**What I did:** I found all instances of `save_to_watchlist()` by using the Search all Instances of `save_to_watchlist()` function, powered by Visual Studio Code.
**How I verified:**
I run integration tests and test main software to make sure it doesn't get any problems.

## Comment 2 — Deduplication
**What I did:** I saw the `add_to_watchlist()` has no deduplication checks. I used GenAI to test the code itself, find out that in the collection_services, there are the duplication checks before testing the result (data request in testing)
**How I verified:** I verified the function using the full pytest suite, manual test and return proper watchlist data.

## Comment 3 — Missing test
**What I did:** Added `tests/test_watchlist.py`, following the same `app`/`sample_user` fixture pattern already established in `tests/test_collection.py`. Added `test_add_to_watchlist_nonexistent_film_raises`, the watchlist equivalent of `test_add_to_collection_nonexistent_film_raises`: it asserts that `add_to_watchlist()` raises `FilmNotFoundError` (not a raw DB integrity error) when given a `film_id` that doesn't exist.
**How I verified:** Ran `pytest tests/test_watchlist.py -v` — the new test passes, along with the file's existing coverage for entry creation, deduplication, and watchlist retrieval.

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