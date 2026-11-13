# Табел на бригадата

Single-file static web app (`index.html`, vanilla JS, no build) for a construction firm: daily attendance per worker (цял ден / половин ден / почивка), month calendar, weekly wages, monthly salaries with a payments history. Bulgarian UI, EUR.

## Hosting
- GitHub Pages from `main` branch root of repo `iampashov/tabel` → https://iampashov.github.io/tabel/
- Deploy = commit + `git push origin main` (ask the user before pushing).

## Backend (Supabase, shared project `comet-blog` nwbvlyempbzzbjdjvjuq)
- Tables with prefix `tabel_`: `tabel_workers(id,name,rate_full,rate_half,active,sort)`, `tabel_days(worker_id,day,status)` PK (worker_id,day), `tabel_payments(id,worker_id,month,amount,paid_on,note)`, `tabel_config(key,value)` (holds the PIN, no API access).
- Access: RLS on every table, policy `tabel_ok()` = request header `x-tabel-key` equals `tabel_config.pin`. The page stores the PIN in localStorage after `rpc/tabel_check_pin` returns true. Publishable key is embedded in the page (safe).
- Change the PIN: `update tabel_config set value='NEW' where key='pin'` via the Management API (`POST /v1/projects/nwbvlyempbzzbjdjvjuq/database/query`, PAT in memory dir `.systematic_hub_supabase_pat`; the `.comet_supabase_pat` one is INVALID).
- Schema SQL: `schema.sql` in this folder (idempotent).
- Sync: REST fetch, optimistic UI, refetch after every write, poll every 30 s + on tab focus. No realtime websocket.

## Gotchas
- python3 urllib fails with SSL cert error on this Mac → use curl for the Management API.
- Deleting a worker cascades to their days and payments (confirmed in-page).
- Rates are per worker (default 65 / 33). Changing a rate recalculates past months too (no rate history).
- `tabel_notes(day pk, note)`: one shared note per day (обект/бележка). Day view input, recent-notes chips, calendar pins, month/week lists.
- LIVE DATA since 13.11.2026 ~17:30 (real workers Георги, Тилчо, …). When testing against the live DB: create a row named "Тест …", delete it by its exact id/name, NEVER range deletes (`day=gte.…`) or table-wide cleanups.
