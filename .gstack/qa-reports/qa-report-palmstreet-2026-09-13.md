# QA Report — PalmStreet (localhost:8091, Standard tier)
Date: 2026-09-13 · Branch: master · Mode: Full (no diff, root-level app)

## Summary
- Issues found: **0**
- Fixes applied: 0 (verified: 0, best-effort: 0, reverted: 0)
- Deferred: 0
- Health score: **10/10 → 10/10** (no regressions, nothing to fix)

> "QA found 0 issues, health score 10/10 → 10/10."

## Setup notes
- Working tree was clean; `checkpoint_mode` set to `continuous` per user choice.
- `npm install` run (no `node_modules` was committed — expected, gitignored).
- App served via `node server.js` on port 8091 (the harness's own preview tool
  kept auto-selecting a static-only server on a stale cwd, so the real Express
  server was run directly in the background instead — see note below).
- `data/scores.json` and `data/visits.json` created during testing were deleted
  after the run (they're gitignored, but no reason to leave test rows behind).
- `.claude/launch.json` was added (kept, per your choice) so `/qa` or `/run` can
  start this server directly next time.

## What was tested
1. **Landing screen** — title, START/TOP 10 buttons, instructions text, footer credit. No console errors on load.
2. **TOP 10 (empty state)** — "Aucun score enregistré pour l'instant. Sois le premier !" renders correctly; closes cleanly.
3. **Gameplay** — START begins the run, character animates, score increments; Space triggers Ollie; wipeout correctly ends the run and shows the final score.
4. **Score submission** — filled pseudo field, "Enregistrer mon score" → `POST /api/scores` → 200, "Score enregistré !" confirmation shown.
5. **TOP 10 (populated)** — submitted score appears correctly ranked (#1, name, score).
6. **Responsive/mobile** — 375×812 viewport renders the full layout correctly, `BEST` score persists via localStorage across reload.
7. **Audio toggle** — mute/unmute button switches icon state correctly, no errors.
8. **API validation** (direct `curl`, beyond what the UI can trigger):
   - `POST /api/scores` with negative score → `400 {"error":"invalid score"}` ✅
   - `POST /api/scores` with score `99999999` (over `MAX_SCORE_VALUE`) → `400` ✅
   - `GET /api/visits` with no key → `401` ✅
   - `GET /api/visits` with wrong key → `401` ✅
   - `GET /api/visits` with correct key → returns visit log ✅
9. **Console/network** — no JS errors, no failed requests, no 4xx/5xx from legitimate UI flows across the whole session.

## Deferred
None.

## Notes for next session
- The harness's built-in dev-server preview (`preview_start` by name) kept
  serving a generic static file server instead of running `server.js`
  (couldn't POST — 501s on `/api/visits`), seemingly due to a stale cached
  server keyed to the parent directory. Running `node server.js` directly in
  the background and pointing the browser at its port was the reliable path.
  If this recurs, it's worth checking there first rather than assuming an app bug.
