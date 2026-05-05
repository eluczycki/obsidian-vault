# Depradar Plan

## Vision
Track dependency health through GitHub activity + CVE data. Help developers spot abandoned or risky deps before they become a problem.

## MVP — Public Dependency Dashboard

**What:** A public page showing 25 popular npm packages with health signals.

**Why:** Proves the concept, draws people in, zero auth overhead.

---

### The List

- Hardcode 25 popular npm packages for now
- Sources: well-known, widely-used packages (lodash, express, react, etc.)
- Ranking/sourcing is future work — just need 25 known-good starting points

---

### Per-Package Widget

For each package, display:

- **Name** — package name + version
- **Velocity graph** — commits over 30/60/90 days
- **Risk score** — number 1-100, color-coded:
  - 🟢 Green: 1-15
  - 🟡 Yellow: 16-30
  - 🟠 Orange: 31-70
  - 🔴 Red: 71-100
- **CVE count** — number of known vulnerabilities (link to NIST)
- **Last commit** — relative date
- **Click → detail page**

### Detail Page (per package)

- Larger velocity graph
- Links to GitHub repo
- Links to NIST CVE page
- Version history

---

### Data Sourcing

| Signal | Source | Notes |
|--------|--------|-------|
| Commit history | GitHub API | velocity graph |
| CVE data | NIST NVD API | free, no auth |
| Package metadata | NPM registry | name, version, repo URL |

---

### Out of Scope (post-MVP)

- User accounts / GitHub OAuth
- User repo scanning
- Alerts system
- History logging
- CI/CD webhook integration
- Subscription tiers

---

## Tech Stack

- **Frontend:** Next.js (same pattern as gitradar)
- **Backend:** Next.js API routes
- **Data:** GitHub API + NPM API + NIST NVD
- **Deploy:** Coolify
- **Auth:** None for MVP

---

## Build Order

1. **Static list** — 25 npm packages, hardcoded
2. **GitHub API integration** — fetch commit history for each
3. **Velocity graph** — render commits as a simple bar/line chart
4. **NIST CVE lookup** — fetch CVE count per package
5. **Risk score calculation** — formula TBD (commits + CVEs + age)
6. **Dashboard page** — 25 widgets, grid layout
7. **Detail page** — per-package page with larger graph + links
8. **Polish** — risk score colors, responsive layout

---

## Decisions (to fill in later)

- [ ] Risk score formula
- [ ] Data refresh cadence (cron? on-demand?)
- [ ] How to store/cache package data
- [ ] Whether to use a DB or just query APIs live
