# Depradar — Decisions

## Risk Score Algorithm v2

**File:** `src/lib/risk.ts`

**Output:** Score 1-100 (higher = more risk), plus a level label.

### Color Bands

| Score | Level | Meaning |
|-------|-------|---------|
| 1-15  | 🟢 Low | Active, no known issues |
| 16-30 | 🟡 Medium | Some concerns, monitor |
| 31-70 | 🟠 High | Significant issues, evaluate carefully |
| 71-100 | 🔴 Critical | Serious problems, avoid or replace |

### Signal Categories

**1. Security Risk (+CVE multiplier)**
- Critical CVEs: +20 each
- High CVEs: +10 each
- Medium CVEs: +5 each
- Low CVEs: +2 each
- **Abandoned multiplier: 1.5x** — no maintainer to push a fix

**2. Activity Risk**
- Zero commits in 30 days: +20
- Very low (1-5 commits/30d): +8
- Low (6-9 commits/30d): +4

**3. Staleness**
- Last commit >1 year ago: +15
- Last commit >6 months ago: +8
- (only if not already captured by activity penalty)

**4. Maintenance Health**
- Open issues >500: +10
- Open issues >100: +5

### Abandoned Flag Triggers
- Zero commits in 90 days
- OR last commit >1 year ago

### Velocity Labels (30d commits per month)
| Commits/mo | Label |
|------------|-------|
| 100+ | warp-speed |
| 50-99 | sonic |
| 20-49 | breakneck |
| 10-19 | brisk |
| 5-9 | snappy |
| 1-4 | sluggish |
| 0 | dormant |

### Base Score: 5

### Design Philosophy
- CVEs dominate the score.
- Abandonment amplifies security risk.
- Contributor count NOT baked into abandoned trigger — no commits is no commits regardless.
- Stars/watchers do NOT affect risk — orthogonal to health.
- Open issues affects score (maintenance health signal) but NOT the abandoned label.

### Inspiration
- Socket.dev's exponential decay with severity caps
- Snyk's likelihood × impact model
- GitHub Advisory's EPSS + CVSS approach

---

## CVE Source

**Switched from NIST NVD to Google OSV.dev** (2026-05-07)
- NVD: rate-limited at 6 req/min unauthenticated, all packages were failing
- OSV.dev: no API key, reliable, same data from GitHub Advisory Database
- Endpoint: `POST https://api.osv.dev/v1/query` with `{ package: { name, ecosystem: "npm" } }`

---

## Data Architecture

**Build-time static JSON** (not DB):
- `scripts/fetchPackages.ts` runs as `prebuild` step before `next build`
- Fetches GitHub commits (page-based, up to 10 pages) + OSV CVEs + GitHub metadata + npm version
- Writes `public/data/packages.json` — gitignored, regenerated on each deploy
- API routes (`/api/packages`, `/api/packages/[name]`) read from JSON — zero runtime API calls
- Data refreshes on every redeploy; no rate limits at runtime

**Why not DB:** Fast iteration, zero infra complexity for MVP. Supabase wiring is planned post-MVP.

---

## GitHub API Rate Limits

| Auth | Limit |
|------|-------|
| Unauthenticated | 60 req/hr |
| Personal token | 5,000 req/hr |

### Per-package API cost at build time
- `/repos/:owner/:repo` → 1 call (stars, forks, open issues)
- `/repos/:owner/:repo/commits` → 1-10 calls (pagination, 100 per page)
- **Total per package: ~1-11 calls**

### Scaling to 3K-15K packages (post-DB)
Not viable with raw API at build time. Plan:
1. **Aggressive caching** in Supabase — refresh metadata weekly, commits daily
2. **Incremental fetching** using `since` param (commits only newer than last fetch)
3. **Background cron** spread across hourly jobs — not one-shot build
4. **Contributor count** — requires separate API call (`/repos/contributors`) — skip for MVP, add later with DB caching

---

## NIST URL Fix

- Previously used broad `keywordSearch` → returned ~350K unrelated CVEs for common names
- Fixed to use CPE keyword: `keywords=CPE%3A2.3%3Aa%3Apackagename%3Apackagename%3A*...`
- Filters to actual npm package CVEs matching `node.js` + vendor CPE pattern