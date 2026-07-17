# PR Response Doc — CineLog Watchlist Feature

## AI Usage
I used Claude (Claude Code) in a few specific ways during this project:

- **Finding the UUID conflict git didn't flag.** After my rebase reported no conflicts, I asked Claude why the UUID migration hadn't shown up. It explained that a clean rebase only means no *textual* conflicts, and that my `WatchlistEntry.film_id` was still `db.Integer` while `Film.id` is a UUID string — a semantic conflict git can't detect. I then changed it to `db.String(36)`.
- **Understanding interactive rebase.** I used it to understand why `git rebase -i` wasn't showing all my commits (it only lists commits above the base) and how to recover a paused rebase with `git rebase --abort`.

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
**My position:** I think default visibility should be set to public=True.
**Reasoning:** CineLog's purpose of being a community film tracking app should have public watchlists in order to share and see each other's watchlists and ratings to build on an individuals collection.
**Tradeoff acknowledged:** A user may not realize their list is public, and once something has been seen (or indexed/cached), that exposure can't be fully undone. I'm accepting that cost because CineLog's core value is social sharing, and a private-by-default setting would leave most lists empty-to-others and undercut the feature's purpose.

## Comment 5 — Sort order
**My position:** I agree with watchlists to default to "date added" order.
**Reasoning:** Ordering with date makes the most sense as a watchlist is built out over time and directly correlates with the timeline a user has watched their films.
**Engagement with reviewer's point:** I agree with Jamila's point that users want to see what they've added recently as it makes sense for users to keep up with what they've most recently watched for real-life senarios as to talk about. 

## Comment 6 — Rebase
**What conflicted:** The .gitignore was conflicting but git didn't recognize the UUID conflict.
**How I resolved it:** I accepted the imcoming changes for .gitignore and asked Claude to help identify what UUID conflicts there were and changed all the old integer IDs to UUID.
**How I verified no conflict remains:** I was able to continue with the rebase and ran the pytest tests to ensure nothing was broken afterwards.

![git log --oneline showing linear history](commits.png)

## PR Description

### What this feature does
This PR adds a **watchlist** to CineLog — a list of films a user wants to watch later, kept separate from their collection (films already watched). It adds:

- A `WatchlistEntry` model linking a user to a film, with a `date_added` timestamp and a `public` visibility flag.
- `POST /watchlist/<user_id>/add` — add a film to a user's watchlist. Body: `{ "film_id": "<uuid>" }`.
- `GET /watchlist/<user_id>` — view a user's watchlist.
- Service logic in `services/watchlist_service.py` that validates the film exists (`FilmNotFoundError`) and prevents duplicate entries (`AlreadyInWatchlistError`), following the same pattern as the collection service.

### Design decisions
1. **Visibility default — `public=True`.** New watchlist entries are public by default. CineLog is a community film-tracking app, so being able to see and share each other's watchlists is core to its value; defaulting to public maximizes discovery. The accepted tradeoff is reduced privacy (a user may not realize their list is visible) — mitigated by the per-entry `public` flag, which a user can turn off for any film. (See Comment 4.)
2. **Sort order — most recently added first.** `get_watchlist()` returns entries ordered by `date_added` descending, mirroring the collection feature, so users see what they most recently added at the top. (See Comment 5.)

### How to manually test
Prereqs:
```bash
pip install -r requirements.txt
python app.py        # serves http://localhost:5000, SQLite cinelog.db
```

There's no signup or film-creation endpoint yet, so first seed one user and one film in a Python shell:
```bash
python
>>> from app import create_app, db
>>> from models import User, Film
>>> app = create_app()
>>> with app.app_context():
...     u = User(username="amy", email="amy@example.com")
...     f = Film(title="Paddington 2", year=2017, genre="Comedy")
...     db.session.add_all([u, f]); db.session.commit()
...     print("USER", u.id); print("FILM", f.id)
```
Copy the printed USER and FILM UUIDs, then (PowerShell users: use `curl.exe`):

1. **Add a film** — expect `201` and the entry JSON:
   ```bash
   curl.exe -X POST http://localhost:5000/watchlist/<USER_ID>/add -H "Content-Type: application/json" -d "{\"film_id\": \"<FILM_ID>\"}"
   ```
2. **View the watchlist** — expect the film, with `date_added` and `public: true`:
   ```bash
   curl.exe http://localhost:5000/watchlist/<USER_ID>
   ```
3. **Duplicate guard** — run step 1 again with the same film. Expect `409` and an "already in this user's watchlist" error, and only one entry in the list.
4. **Nonexistent film** — POST a made-up UUID. Expect `404` and a "no film found" error:
   ```bash
   curl.exe -X POST http://localhost:5000/watchlist/<USER_ID>/add -H "Content-Type: application/json" -d "{\"film_id\": \"00000000-0000-0000-0000-000000000000\"}"
   ```
5. **Sort order** — add a second film, GET the watchlist again, and confirm the most recently added film appears first.

Automated tests: `pytest tests/test_watchlist.py` (deduplication and nonexistent-film cases).
