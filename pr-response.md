# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->
I use Claude Code to mostly testing the project findings and make sure there is no errors. Additionally, I test the function to see no potential bugs in this program, and in Comment 4, 5 I used it to debate the ways it is functioning in the code and system design tradeoffs, accounting for external and internal regulations (EU about privacy) and engagement/sharing-by-default cases.

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
**My position:** I'm keeping `public=True` as the default. CineLog's core value is social — seeing what other people are watching and queuing — and a watchlist that's private until a user finds and flips a setting is a watchlist almost nobody will ever make public, because opt-in settings have very low discovery. If the goal is a Letterboxd/Goodreads-style feed where activity is visible by default, the default has to carry that weight, not a toggle buried in account settings.

**User behavior I'm optimizing for:** the median user who never visits settings at all. Defaulting to public means every new watchlist entry is immediately useful to the social/discovery features (recommendations, shared profiles, "friends are watching") without requiring an extra action. I'm explicitly trading a more cautious default for a higher-participation network — this is a bet that most users don't consider their watchlist sensitive (it's "movies I want to see," not viewing history or personal data) and would rather it "just work" as shareable content.

**Tradeoff acknowledged:** the real cost is silent oversharing — a user who *does* care about privacy (e.g., doesn't want a partner, employer, or follower seeing a specific title) has no signal that they need to act, and won't discover the setting until after something is already visible. That's a worse failure mode than the inverse (a user missing out on social features because they forgot to opt in), since it's privacy-negative rather than just growth-negative. I'm accepting that risk for this PR because entries are movie titles, not sensitive data, and mitigating it properly (e.g., a first-run visibility prompt, or a global "make new entries private" preference) is a real feature, not a one-line change — it's out of scope here but worth a follow-up ticket if reviewers push back.

## Comment 5 — Sort order
**My position:** `get_watchlist()` currently sorts alphabetically (`Film.title.asc()`, [services/watchlist_service.py:64](services/watchlist_service.py#L64)), while `get_collection()` sorts newest-added-first ([services/collection_service.py:102](services/collection_service.py#L102)). I'm changing the watchlist to sort newest-first too, so both endpoints follow one convention instead of two.

**User behavior I'm optimizing for:** someone opening their watchlist to decide what to queue next. A "want to watch" list is read chronologically by intent — the film you just added is the one fresh in your mind, and it's the one you're most likely to act on next. Alphabetical order buries that signal: on a list of any real size, a title you added five minutes ago could be sorted anywhere from A to Z, so the user has to scan the whole list instead of just checking the top.

**Counterargument a careful reviewer would raise / tradeoff I wasn't acknowledging:** alphabetical order is actually *better* for one real use case — relocating a specific, already-known title. If a user has 80 films on their watchlist and wants to check whether "Inception" is already on it, alphabetical is a direct lookup; newest-first means the position of every title shifts every time something new is added, so a title you found last week may not be where you left it. That's a genuine, not hypothetical, cost of switching, and my position glossed over it — I'm accepting it because "browse what to watch next" is the more common watchlist interaction than "look up one known title," but a reviewer could reasonably push back and ask for search/filter instead of relying on sort order to solve findability.

The second, more concrete issue: the existing "sort order" test doesn't cover this at all. `test_get_watchlist_returns_newest_first` in `tests/test_watchlist.py` builds `CollectionEntry` rows and asserts against `get_collection()` — it's a leftover copy-paste from `test_collection.py` that never got adapted to the watchlist service, so it passes regardless of how `get_watchlist()` sorts. I'm rewriting it to build `WatchlistEntry` rows and call `get_watchlist()` directly, so it actually fails if the ordering regresses.
**How I verified:** Ran `pytest tests/test_watchlist.py -v` after fixing both the `order_by` clause and the test to confirm newest-first ordering is real, not just asserted by a mislabeled test.

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->