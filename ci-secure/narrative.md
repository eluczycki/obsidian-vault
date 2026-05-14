# CI Secure — Product Narrative

## What is CI Secure?

CI Secure helps DevOps and security teams answer one question before a dependency enters a CI/CD pipeline: **is this package safe to use right now?**

It works by combining live CVE data (from OSV.dev / GitHub Advisories), GitHub activity signals (commit frequency, issue age, fork health), and npm metadata (deprecation flags, release cadence) into a single risk score and status label per package.

---

## Why "Where Pipeline Meets Policy"

Modern pipelines pull in hundreds of packages. Most teams don't audit them — they just run `npm install` and move on. The ones that do care about security usually rely on Snyk or Socket after the fact, when a vulnerability is already public.

CI Secure is about **pre-commit gates** and **visibility**: before you promote a build, you should know the risk profile of every dependency. If `lodash` has 10 CVEs and hasn't had a commit in 3 years, that's policy-relevant — especially in regulated industries.

The tagline reflects that intersection: the pipeline is where code flows, policy is what determines what code is allowed to flow.

---

## What changed from Depradar

Depradar was a public dashboard — a curiosity tool for devs to check if their deps were abandoned. CI Secure is the same underlying data, but positioned as a DevOps tool:

| | Depradar | CI Secure |
|--|--|--|
| Audience | Individual developers | DevOps, platform, security teams |
| Access | Public, unauthenticated | Teams, org-level views |
| Tone | Curious, "is this dead?" | Security-conscious, "is this safe?" |
| Primary action | Browse/search | Integrate into CI workflow |
| Brand | Radar / abandoned packages | Shield / CI pipeline security |

---

## Core product bets

1. **CVE data as primary signal** — Socket and Snyk do this well, but we can be more transparent about freshness and commit-signal context
2. **npm-aware abandonment** — Package has npm releases but no GitHub commits? Not abandoned. Commit frequency ≠ maintenance.
3. **Multi-ecosystem** — npm first, then pip/cargo/go. Same risk model, different registry APIs.
4. **CI integration** — Not scoped for v0.5, but the roadmap goes toward webhook gates and policy configuration