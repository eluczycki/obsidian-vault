# Depradar — Decisions

## Risk Score Algorithm ✅

**File:** `src/lib/risk.ts`

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