# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:** I used my editor's find-all-references for save_to_watchlist and renamed them to add_to_watchlist.
**How I verified:** Ensuring the editor's find-all-references for save_to_watchlist had no more results.

## Comment 2 — Deduplication
**What I did:** I used the pattern of add_to_collection() in services/collection_service.py to write the deduplication logic to add_to_watchlist() in services/watchlist_service.py. I replaced the collection values with watchlist and added a new error AlreadyInWatchlistError to raise for the check.
**How I verified:** I created a test case similar to the deduplication test in test_collection for add_to_watchlist() and confirmed it passed.

## Comment 3 — Missing test
**What I did:** Created the test case for add_to_watchlist() using test_add_to_collection_nonexistent_film_raises and changing the logic for watchlist.
**How I verified:** I ran the test case added to make sure it worked.

## Comment 4 — Default visibility
**My position:** I think default visibility should be set to public=False.
**Reasoning:** Looking at other sites with a watchlist feature, they have it set to private view by default. I think this is a good way of providing privacy for our users as well and it can be their decision to make it public.
**Tradeoff acknowledged:** Defaulting to private reduces discoverability — users who'd happily share won't, until they find the toggle, so social/sharing features see less engagement out of the box. I'm accepting that cost because accidental over-sharing (a privacy leak) is effectively irreversible, while under-sharing is one click away. The asymmetry favors private.

## Comment 5 — Sort order
**My position:** I agree with watchlists to default to "date added" order.
**Reasoning:** Ordering with date makes the most sense as a watchlist is built out over time and directly relates to the desirability users have with which movies they'd want to see in their watchlist. Newly added movies are the most updated movies they'd like to watch.
**Engagement with reviewer's point:** I agree with Jamila's point that users want to see what they've added recently as it makes sense for users to want to watch films still fresh in their minds. 

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->
