# Depradar — Status

## Current: v0.3.0 (staging)

**Shipped:**
- 250 packages (seeded, API serving 115 due to list gap — see below)
- Supabase DB wired (staging)
- Trend arrows on dashboard cards
- Top X% percentile on detail page
- Search + random 16 dashboard
- Live lookup `GET /api/packages/lookup`
- On-demand refresh `POST /api/packages/refresh/[name]`
- Uncapped scores, displayed as `100+`

**Known issues:**
- PACKAGES list has 117 entries, not 250 — needs expansion to reach 250

## v0.4 Plan (next sprint)

See `v0.4-plan.md` for full details.

**Goal:** System isolation — app writes its own status, OpenClaw monitors from outside.

**Key components:**
1. `GET /api/status` — single JSON endpoint for system + app + script status
2. Server scripts write to `data/script-status.json` (no AI, no OpenClaw)
3. OpenClaw cron job (`*/5 * * * *`) polls `/api/status`, posts alerts to Discord `#1501238142265593946`
4. Alert format: 🔴 CRITICAL / 🟡 WARNING / ✅ Recovered

**Implementation order:**
1. Status endpoint (`GET /api/status`)
2. Script logging (scripts write to local JSON)
3. OpenClaw monitor cron (replace existing LLM-based job)
4. Alert refinement (dedup, recovery notices)

## v0.4 Scope
- Discord alerts only (no email/Slack)
- Status page / historical data — out of scope
- No auto-remediation