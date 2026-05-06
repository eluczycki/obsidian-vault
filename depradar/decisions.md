# Depradar — Decisions

## Risk Score Algorithm ✅

**File:** `src/lib/github.ts` (`calculateRiskScore`)

**Output:** Score 1-100 (higher = more risk), plus a level label.

### Color Bands

| Score | Level | Color | Meaning |
|-------|-------|-------|---------|
| 1-15  | Low | 🟢 | Active, no known issues |
| 16-30 | Medium | 🟡 | Some concerns, monitor |
| 31-70 | High | 🟠 | Significant issues, evaluate carefully |
| 71-100 | Critical | 🔴 | Serious problems, avoid or replace |

### Algorithm (Current)

**Base score:** 10

**Activity (30d commits):**
| Commits | Points |
|---------|--------|
| 0 | +25 |
| 1-2 | +15 |
| 3-9 | +5 |
| 10+ | +0 |

**CVE Scoring (weighted by severity):**
| Severity | Points each |
|----------|-------------|
| Critical | +15 |
| High | +8 |
| Medium | +3 |
| Low | +1 |

**Staleness (>6 months since last commit):** +15 (only if not already flagged for zero activity)

**Cap:** 100

### Inspiration
- Socket.dev's exponential decay with severity caps
- Snyk's likelihood × impact model
- GitHub Advisory's EPSS + CVSS approach

### Data Architecture ✅

**Build-time static JSON** (not DB):
- `scripts/fetchPackages.ts` runs as `prebuild` step before `next build`
- Fetches GitHub commits (date-range pagination) + NVD CVE breakdown + npm version for all 25 packages
- Writes `public/data/packages.json` — gitignored, regenerated on each deploy
- API routes (`/api/packages`, `/api/packages/[name]`) read from JSON — zero runtime API calls
- Data refreshes on every redeploy; no rate limits at runtime

**Why not DB:** Fast iteration, zero infra complexity for MVP. Supabase wiring is planned post-MVP for user accounts + repo scanning + live refresh.


### NIST URL Fix ✅
- Previously used broad `keywordSearch` → returned ~350K unrelated CVEs for common names
- Fixed to use CPE keyword: `keywords=CPE%3A2.3%3Aa%3Apackagename%3Apackagename%3A*...`
- Filters to actual npm package CVEs matching `node.js` + vendor CPE pattern

**Output:** Score 1-100 (higher = more risk), plus a level label.

### Color Bands

| Score | Level | Color | Meaning |
|-------|-------|-------|---------|
| 1-15  | Low | 🟢 | Active, no known issues |
| 16-30 | Medium | 🟡 | Some concerns, monitor |
| 31-70 | High | 🟠 | Significant issues, evaluate carefully |
| 71-100 | Critical | 🔴 | Serious problems, avoid or replace |

### Signal Categories

**1. Security Risk (primary)**
- Critical CVEs: +20 each
- High CVEs: +10 each
- Medium CVEs: +5 each
- Low CVEs: +2 each

**2. Abandonment Risk (secondary)**
- Zero commits in 90 days: +25
- Zero commits in 30 days: +15 (additive on 90-day rule)
- Very few commits (1-5 in 30 days): +8
- Last commit >12 months ago: +15
- Last commit >6 months ago: +8

**3. Maintenance Health (tertiary)**
- Open issues >100: +5
- Open issues >500: +10
- Single contributor: +3

### Base Score
Starts at 10 — nothing is ever perfectly zero-risk.

### Design Philosophy
- CVEs dominate the score: 1 critical CVE pushes toward red zone immediately.
- Abandonment amplifies security risk: abandoned + CVEs is worse than active + same CVEs.
- Popularity (stars/watchers) does NOT factor in — orthogonal to risk.
- Package age alone doesn't matter — old and actively maintained is fine.

### Inspiration
- Socket.dev's exponential decay with severity caps
- Snyk's likelihood × impact model
- GitHub Advisory's EPSS + CVSS approach

### Level Thresholds
```ts
level =
  score >= 71 → "critical"
  score >= 31 → "high"
  score >= 16 → "medium"
  "low"
```