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

- [x] Risk score formula
- [x] Data refresh cadence (on every redeploy via prebuild)
- [x] How to store/cache package data (static JSON at build time)
- [x] Package repo reconnect strategy (multi-source fallback chain)
- [ ] Supabase wiring (user accounts, repo scanning, subscriptions)
- [ ] Responsive design review

## Post-MVP

### Supabase Wiring
- Add `DATABASE_URL` to Coolify env vars (internal PostgreSQL: `postgresql://10.21.0.X:5432/supabase_db`)
- Add Prisma or Drizzle ORM
- Schema: `users` (GitHub OAuth), `user_repos` (linked repos), `package_lists` (curated lists), `scan_history`
- Build cron/edge function to refresh GitHub data periodically

### Responsive Design
- Review dashboard grid on mobile (cards may need narrower min-width)
- Detail page stats row may need scroll/breakdown on small screens
- Chart toggle buttons may need touch-friendly sizing

### Repo Scanning
- Allow users to link GitHub repos (OAuth)
- Scan all `package.json` deps in user repos
- Score the full dependency graph with actual used versions

### Package Manager Expansions
- Add pip/PyPI, Cargo/Rust, Go modules support
- Each new registry needs its own API integration and CVE source

### Alerting & History
- Email/Slack alerts when a tracked package crosses a risk threshold
- Historical tracking — show score over time, not just current snapshot

Package Manager Additions:

Language-Specific Package Managers

- npm (Node Package Manager): The default for JavaScript/Node.js, hosting the world's largest software registry.
- pip: The standard tool for Python, used to install packages from the Python Package Index (PyPI).
- Maven: A widely used build and dependency manager for Java and other JVM-based projects.
- Cargo: The built-in manager for Rust, handling both builds and dependency resolution.
- Composer: The primary tool for PHP, managing project-level libraries.
- NuGet: The central package manager for the .NET ecosystem (C#, F#, etc.).
- CocoaPods: A popular dependency manager for Swift and Objective-C Cocoa projects.
- RubyGems: The standard manager for the Ruby language. [[4](https://missing.csail.mit.edu/2019/package-management/), [5](https://en.wikipedia.org/wiki/List_of_software_package_management_systems), [6](https://builtin.com/software-engineering-perspectives/package-managers), [9](https://www.youtube.com/watch?v=QWkHQ6ifKm0), [10](https://www.devopsschool.com/blog/top-10-package-managers-features-pros-cons-comparison/), [11](https://www.activestate.com/resources/quick-reads/dependency-management-with-pip-pythons-package-manager/#:~:text=Pip%20is%20Python's%20package%20manager%2C%20providing%20essential,one.%20Pip%20will%20not%20flag%20dependency%20conflicts.)]

System-Level Package Managers

These tools manage software and libraries for entire operating systems, often handling core system updates and shared libraries.

- APT (Advanced Package Tool): Used by Debian and Ubuntu-based Linux distributions.
- DNF (Dandified YUM): The modern replacement for YUM, used in Fedora, RHEL, and CentOS.
- Homebrew: A popular "missing" package manager for macOS (and Linux), favored by developers for installing CLI tools.
- Pacman: The package manager for Arch Linux, known for its speed and simplicity.
- Chocolatey: A popular community-driven package manager for Windows automation. [[6](https://builtin.com/software-engineering-perspectives/package-managers), [10](https://www.devopsschool.com/blog/top-10-package-managers-features-pros-cons-comparison/), [12](https://nareshit.com/blogs/linux-package-management-apt-yum-dnf-guide), [15](https://www.linux.org/threads/package-managers.44959/), [16](https://patchbay.tech/your-promotion-to-package-manager-manager/#:~:text=Homebrew%20is%20far%20from%20the%20only%20command,included%20in%20Linux%20operating%20systems%20by%20default.)]

Specialized & Modern Alternatives

  

Newer tools often focus on better performance, deterministic installs (lock files), or specific data science needs.

- Yarn / pnpm: Popular alternatives to npm for JavaScript that prioritize speed and disk efficiency.
- Conda: A language-agnostic manager (often used for Python/R) that handles complex environment and binary dependency issues in data science.
- Poetry / uv: Modern Python tools that combine dependency resolution with virtual environment management. [[5](https://en.wikipedia.org/wiki/List_of_software_package_management_systems), [10](https://www.devopsschool.com/blog/top-10-package-managers-features-pros-cons-comparison/), [17](https://thenewstack.io/how-to-choose-the-best-python-package-management-tool/), [18](https://www.fe.engineer/handbook/package-managers), [21](https://dev.to/adamghill/python-package-manager-comparison-1g98)]

  

