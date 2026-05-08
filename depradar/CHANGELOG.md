# Changelog

All notable changes to Depradar are documented here.

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