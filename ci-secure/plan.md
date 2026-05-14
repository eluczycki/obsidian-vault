# CI Secure — v0.5 Plan

## Rebrand: Depradar → CI Secure

**Product shift:** From "abandoned package radar" (developer curiosity) to "CI pipeline security" (DevOps tool).

**Tagline:** "Where Pipeline Meets Policy"

**What changes:**
- Name: `depradar` (project) → `CI Secure` (product)
- Tone: technical, compliance-adjacent, security-conscious
- Audience: DevOps engineers, security teams, platform teams — not just individual devs
- Repo can stay `depradar` internally; product surfaces as "CI Secure"

---

## v0.5 — UI Rebrand + Mobile

### 1. Color System Overhaul

**Goal:** Replace pure black with a tinted greyscale palette. Cards stay dark, content areas shift lighter.

**Proposed palette (dark theme):**
| Role | Hex | Usage |
|------|-----|-------|
| Background | `#0F1419` | Page background (was `#000000`) |
| Surface | `#1A2332` | Cards, panels |
| Surface-alt | `#212B3C` | Hover states, nested panels |
| Border | `#2E3A4D` | Dividers, card borders |
| Text primary | `#E8EDF5` | Main text |
| Text muted | `#7A8BA3` | Secondary text, labels |
| Accent | `#3B82F6` | Links, active states (CI blue) |

**Why:** Pure black (`#000`) is harsh on eyes and feels dated. `#0F1419` is close to black but with a cool blue-grey tint that pairs with the CI Secure logo. Cards keep `#000` or very dark to maintain card hierarchy.

**Scope:** CSS variables in `globals.css` or a `theme.css`. No component logic changes.

---

### 2. New Logo + Branding

**Assets:**
- Logo: `public/logo.png` (new CI Secure logo — dark blue circular broken "C" + wordmark)
- Favicon: `app/favicon.ico` (update to match)
- OG image: `public/og-image.png` (update for social sharing)

**Changes:**
- Header logo swap: replace depradar logo → CI Secure logo
- Update `manifest.json` / page `<title>` / meta tags
- Remove any "depradar" references from UI strings
- Footer tagline: add "Where Pipeline Meets Policy" or brand one-liner

**Files to update:**
- `src/components/Header.tsx` or equivalent
- `src/app/layout.tsx` (title, description, OG tags)
- `public/` assets

---

### 3. Mobile Responsiveness

#### Dashboard
- **Target:** 3 cards per row on mobile (≤768px), 4 on tablet, responsive grid up
- **Card size:** Reduce padding, font sizes, graph height on mobile
- **Min-width:** Set `min-width: 280px` on cards, let CSS grid handle the rest
- **Scroll:** Dashboard should be scannable without infinite scrolling being the only option

#### Detail Page
- **Scale down:** Reduce font sizes and container widths
- **Font sizes:** Body from 16px → 14px on mobile; title from 24px → 20px
- **Stats row:** Stack vertically on mobile instead of horizontal
- **Graph:** Reduce height; ensure it doesn't overflow container
- **Priority:** Not pixel-perfect — mobile is secondary access, but functional

---

### 4. Team/Enterprise Dashboard Consideration

Not fully scoped for v0.5, but keep UI patterns in mind:
- **Reusable card/panel components** — team dashboard will need similar widgets
- **API-first thinking** — all team data should flow through API routes, not hardcoded
- **User context** — if team features need auth, layout should accommodate a user avatar/login state

Flag this in decisions rather than implementing in v0.5.

---

## Out of Scope for v0.5

- Auth / user accounts
- Team/org features
- Pipeline integration (webhooks, CI plugin)
- Additional ecosystems (pip, cargo, etc.)
- Alerting

---

## Tech Notes

**Color implementation:**
```css
:root {
  --bg: #0F1419;
  --surface: #1A2332;
  --surface-alt: #212B3C;
  --border: #2E3A4D;
  --text: #E8EDF5;
  --text-muted: #7A8BA3;
  --accent: #3B82F6;
}
```

**Mobile breakpoints to test:**
- `< 640px` — mobile (3 cards, reduced fonts)
- `640px - 1024px` — tablet (4 cards)
- `> 1024px` — desktop (full layout)

**Files likely to change:**
- `src/app/globals.css` — color variables
- `src/components/Header.tsx` — logo swap
- `src/app/layout.tsx` — title, meta, OG
- `src/components/PackageWidget.tsx` — mobile card sizing
- `src/app/package/[name]/page.tsx` — mobile detail page
- `src/components/DashboardContent.tsx` — grid layout on mobile

---

## Decisions

- [x] Color palette: `#0F1419` bg / `#1A2332` surface / `#3B82F6` accent
- [ ] Logo asset: `public/logo.png` — attach freshmaker's logo
- [ ] Card background on mobile: keep `#000` or shift to `--surface`? (recommend `--surface`)
- [ ] Detail page mobile priority: functional vs pretty (functional for v0.5)