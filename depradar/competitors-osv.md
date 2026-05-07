# osv.dev vs dprdr.co — Competitive Analysis

**Date:** 2026-05-07
**Author:** Daryl (subagent)

---

## What osv.dev Actually Is

**Product:** A distributed, open-source vulnerability database + API infrastructure run by Google/OpenSSF.

**Core offering:**
- Aggregates vulnerability data from GitHub Security Advisories, PyPA, RustSec, GSD, and others
- All data uses the [OpenSSF OSV format](https://ossf.github.io/osv-schema/) — a standardized schema that maps vulns to package versions or commit hashes
- Serves as the canonical backing data source for many security tools (including Depradar)

**What it provides (free, no limits):**
- API at `api.osv.dev/v1` — query by commit hash, package+version, or batch query
- Web UI at osv.dev — searchable list of all vulnerabilities
- OSV-Scanner — first-party CLI tool to scan lockfiles, SBOMs, directories, container images
- GitHub Actions workflows for CI/CD integration
- No rate limits. No API key required.
- Response size limit of 32MiB (HTTP/1.1), none for HTTP/2

**Supported ecosystems:** npm, PyPI, Go, RubyGems, crates.io, Maven, NuGet, Packagist, Pub, Swift Package Manager, Debian, Alpine, Wolfi, and more (including Julia, Android, OSS-Fuzz)

**Target user:** Developers and security engineers who need raw vulnerability data. It's infrastructure — not a consumer product. You query it programmatically or use OSV-Scanner locally.

**Paid features:** None. It's a Google-run open-source infrastructure project.

**Coverage:** Hundreds of thousands of vulnerability records. Very up-to-date (CVEs from today are visible on the list page).

---

## What dprdr.co (Depradar) Actually Is

**Product:** A dependency health dashboard that scores npm packages on a 1–100 risk scale.

**Data source:** Uses OSV as its CVE backend, then layers on GitHub activity signals.

**Risk score formula (from the data):**
- Base score (baseline)
- CVE factor — count + severity (critical/high/medium/low)
- Activity factor — recent commit velocity (30/60/90 day windows)
- Maintenance factor — open issue count

**Features:**
- 25-package MVP (static JSON at build time, no API)
- Per-package detail pages with full CVE list (with OSV links), commit history chart, CVE breakdown by severity, risk breakdown factor analysis
- Velocity labels: dormant, sluggish, snappy, sonic, breakneck, warp-speed
- Status labels: active, abandoned
- GitHub stars/forks/open issues embedded
- Links out to npm, GitHub, OSV list page for the package

**Target user:** Developers evaluating npm packages who want a quick "should I use this?" signal before adding a dependency. It's a decision-support tool, not a security scanner.

**Paid features:** None yet. MVP.

---

## Honest Comparison: Where dprdr.co Adds Value vs osv.dev

### Novel signals osv.dev doesn't have

**1. Commit velocity scoring (unique to Depradar)**
osv.dev tells you what vulnerabilities exist. Depradar tells you how alive the project is. A package with CVEs that has 73 commits/month in the last 30 days (vite) is meaningfully different from one with 0 commits/month (moment). osv.dev has no concept of activity level.

**2. Abandonment detection (unique to Depradar)**
`moment` is tagged `"status": "abandoned"` with `"lastCommit": "2024-02-18"`. osv.dev won't tell you a package is abandoned — it will still surface its CVEs, but there's no signal that the maintainer walked away. This is genuinely useful for long-term dependency decisions.

**3. Composite risk score (unique to Depradar)**
osv.dev returns raw CVE records. Depradar distills that into a 0–100 risk score with labeled severity (critical/high/moderate/low) and a factor breakdown. For quick decision-making, a single number is faster than scanning a list of vulnerability objects.

**4. Package context / comparison view**
osv.dev is a database you query. Depradar is a dashboard that shows 25 popular packages side-by-side. You can see at a glance that `lodash` (critical, 10 CVEs, sluggish velocity) and `vite` (critical, 19 CVEs, sonic velocity) are both "critical" risk but for opposite reasons. That's a different use experience.

---

### Where Depradar is just a prettier widget on OSV data

**1. CVE data is 100% sourced from osv.dev**
Every CVE entry in Depradar has an `osvUrl` linking back to osv.dev. The CVE summaries, CVSS vectors, severity labels — all from OSV. Depradar is not curating its own vulnerability data. If osv.dev goes away, so does Depradar's security data.

**2. No API, no integration story**
osv.dev has a well-documented REST API with no rate limits. Depradar is a static 25-package snapshot. For any automated workflow (CI, dependency bot, etc.), osv.dev wins trivially. Depradar's value is in human-in-the-loop decision making, not automation.

**3. No SBOM/lockfile scanning**
OSV-Scanner can scan your `package-lock.json` or a container image and report only vulnerabilities that actually affect your pinned versions. Depradar shows vulnerabilities for the latest version of a package, not what's in your lockfile. A package might have 10 CVEs total but only 2 affecting the version you're using — Depradar won't tell you that.

**4. No ecosystem breadth**
Depradar covers only npm packages (25 of them). osv.dev covers dozens of ecosystems. If you're in the Go or Rust ecosystem, Depradar is useless.

---

## What Would osv.dev Have to Build to Replace Depradar?

**Short answer: not much, and it's plausible.**

osv.dev already has the CVE data. Adding:
1. **GitHub API integration** — commit frequency, stars, open issues — is trivial. They already have the infrastructure.
2. **A risk scoring layer** — composite scores per package — is a straightforward computation on top of existing data.
3. **A dashboard/UI** — osv.dev is notably minimal. Adding a "package health" tab alongside the vulnerability list page is a product decision, not a hard engineering problem.

The main thing osv.dev would need is **willingness to move up the stack** from pure infrastructure to end-user product. They're a database/API company, not a dashboard company. But Google has built consumer-facing products on top of their infrastructure before (e.g., Cronet). It's not outside their capability.

**Would they?** Unlikely in the near term — osv.dev is maintained by a small team focused on data quality and scanner tooling. A consumer dashboard for npm package evaluation is a different product/market. But "unlikely" ≠ "never."

---

## Summary Verdict

| Dimension | osv.dev | Depradar |
|---|---|---|
| CVE data depth | ✅ Full raw data | ✅ From OSV (same data) |
| CVE coverage | ✅ All ecosystems, all packages | ❌ npm only, 25 packages |
| Activity signals | ❌ None | ✅ Commit velocity, abandonment |
| Risk scoring | ❌ Raw data only | ✅ Composite 0–100 score |
| API | ✅ Full REST API, no limits | ❌ None (static build) |
| Lockfile/SBOM scan | ✅ OSV-Scanner | ❌ None |
| Freshness | ✅ Real-time | ⚠️ Build-time snapshot |
| Target user | Security engineers, tool builders | Developers evaluating packages |
| Paid tier | Free | Free (MVP) |

**Depradar's genuine value:** Package health signals (velocity, abandonment) layered on top of OSV CVE data, with a human-readable risk score and dashboard. It's useful for the specific use case of "should I add this npm package?" where osv.dev is too technical and too raw.

**Depradar's honest limitations:** It is a thin layer on top of OSV data for the CVE portion. It has no API, no automation story, no lockfile awareness, and tiny coverage. The activity/abandonment signals are genuinely novel and useful, but CVE data is commoditized. If OSV built a dashboard, Depradar's window closes unless it moves up the stack (community curation, broader ecosystems, lockfile integration, CI plugin).

**Bottom line:** Depradar is a clever MVP that solves a real pain point (package evaluation for developers) that osv.dev ignores. But it's riding on OSV's data infrastructure and would need to expand significantly — both in coverage and in non-CVE signals — to be defensible long-term.