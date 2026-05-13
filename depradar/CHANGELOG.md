# Changelog

All notable changes to Depradar are documented here.

## v0.4.0 — Backfill & Automation

**Released:** 2026-05-13

### What's new

**Automated polling** — `scripts/hourly-polling.ts` runs every hour via cron, updates all packages with fresh npm metadata, GitHub activity, CVE data, and risk scores. Flagged packages (npm 404, GitHub gone) are marked for review automatically.

**5-year backfill** — `scripts/chunked-backfill.ts` sweeps full commit history from GitHub in 6-month chunks per package. All 256 packages now have complete commit history going back to 2021-05-13. Previously-truncated packages (e.g., `express`) are fixed.

**Review flagging system** — Dead/abandoned repos, npm packages no longer in registry, and broken GitHub links are flagged with a `needs_review` flag and a JSON reason object (`needs_review_reason`). Review queue can be managed via API.

**Full 365-day commit history** — Stored per package in `commit_history` as an array of `{date, commits}` objects, not just last 30 days. Chart windows (60d/6m/1yr/3yr/5yr) are now accurate.

**Backfill history API** — `GET /api/packages/backfill-history` shows per-package backfill status: `needsBackfill`, `backfillComplete`, `flaggedForReview`.

### Bug fixes

- `jsonb_build_object` with parameterized values failed with "could not determine data type of parameter $1" — fixed by casting all values with `::text`
- Chart zero-padding only applied to future days (graph started from last data point instead of full window) — fixed
- Stamp badge positioning shifted below graph on all-packages view — fixed
- Card redesign: all-3-status stamps centered in graph top 20%, auto-window caps at available data, disabled toggles for unavailable time windows

### Infrastructure

- Cron wrapper at `/tmp/depradar-cron-wrapper.sh` — re-fetches container by name pattern on each exec (survives redeploys), 1-hour lock prevents overlapping runs
- Container now includes `pg`, `dotenv`, and `tsx` for script execution
- Backfill status tracked per-package (`is_backfill_complete`, `backfill_started_at`)

### What's NOT in this release (see v0.5)
- User accounts
- Admin interface
- Rebrand

---

## v0.1.0 — MVP/PoC

### What's working
- **Risk score** (1–100) with age-weighted CVEs + activity + maintenance signals
- **5-band risk tiers**: Low (1–15) / Moderate (16–35) / Medium (36–55) / High (56–80) / Critical (81–100)
- **CVE entries** sorted oldest-first with shield icons and age-adjusted point values + tooltips
- **Commit activity chart** (60d/6m/1yr/3yr/5yr time windows)
- **Dashboard card grid** with risk pills and CVE counts
- **Detail page** with full risk breakdown and CVE sub-entries
- **Fixed-size uniform score badges** (52px) across all views — filled and bordered
- **OSV.dev** as CVE data source (GitHub advisory database)
- **25-package hardcoded dataset** — placeholder until DB wiring

### Tech stack
- Next.js + TypeScript
- Recharts (commit activity)
- OSV.dev API (vulnerability data)
- GitHub REST API (commit/issue signals)
- Staged at: stg.dprdr.co

---

### Next up (v0.2.0)
- [ ] DB wiring — PostgreSQL via Coolify
- [ ] Dynamic package list from DB instead of hardcoded
- [ ] Filter/sort dashboard by risk tier, velocity, CVE count, etc.
- [ ] Add new package form or URL input