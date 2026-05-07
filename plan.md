# Greenlit — CTO in a Box for AI-Built Apps

_Last updated: Session 6 — Strategic Overhaul + CTO Decisions_

---

## What Greenlit Actually Is

**"CTO in a Box"** — the technical co-founder you don't have.

You built your app with Lovable/Bolt/Cursor. You have 500 users. You don't know what CORS is.
Greenlit is the CTO who:
- Looks at your code and tells you "here's what you actually built" (in plain English)
- Finds the holes a hacker would find (and proves it with real evidence)
- Writes the fix for you (paste it into Cursor, done)
- Watches your app 24/7 and tells you if something breaks

> **Tagline**: "You built it with AI. We protect it with AI."

---

## Competitive Landscape (Honest Assessment)

| Tool | What They Do | Pricing | Greenlit's Edge | Greenlit's Gap |
|---|---|---|---|---|
| **GitHub Advanced Security** | CodeQL SAST + secret scanning + push protection | $30-60/seat/mo (Enterprise only) | 100x cheaper, plain English, no enterprise gate | Not in developer workflow yet |
| **CodeRabbit** | AI PR reviews + inline suggestions + 1-click commit | $24-48/user/mo | We do runtime probing, not just code review | No PR-level integration |
| **Snyk** | SCA + SAST + Container + IaC — full lifecycle | Enterprise pricing | AI-readable output, non-tech audience | No SCA, no container scanning |
| **Aikido** | All-in-one AppSec with noise reduction | Mid-market SaaS | Vibe-coder specific checks | No reachability analysis |
| **StackHawk** | DAST + API discovery in CI/CD (ZAP-based) | Usage-based | Plain-English output + fix prompts | Not in CI/CD pipeline |
| **VAS / SafeVibe** | URL-only scan for vibe-coded apps | Free/freemium | SAST + DAST + Auto-Fix PRs + persistence | One-shot only, no persistence |

### Why People Will Pay for Greenlit (Not the Others)

**The user**: A non-technical founder who built a SaaS with Lovable/Bolt/Cursor. They have 500 users. They don't know what CORS is. They don't have a security engineer. They need someone to tell them "your app has a hole, here's the fix, paste it into Cursor."

**Why not free tools?** GitHub's free scanning requires understanding CodeQL queries. VAS is one-shot with no follow-up. Snyk's output assumes you know what a CVE is.

**Why not enterprise tools?** $30-60/seat/mo for GHAS. $24/user/mo for CodeRabbit. Overkill for a solo founder with 1 repo.

**Why Greenlit?** ₹499/mo gets you: a CTO that explains what you built, finds the holes, writes the fix, and watches your app 24/7. The output is designed to feed back into the AI tool that built your app. Like having a senior engineer on retainer for the price of a pizza.

---

## Honest Rating

| Milestone | Rating | What changes |
|---|---|---|
| Today (SAST + DAST + dark UI) | **6.5 / 10** | Good demo, unfinished product |
| + Real Auto-Fix PRs + AI Fix Prompts + Architecture Education | **7.5 / 10** | Actual product, not a demo |
| + Public reports + Badge + GitHub Action | **8.5 / 10** | Distribution + virality |
| + ₹10K MRR + 10 paying users | **9.0 / 10** | Proof of business |
| + Platform integration (Lovable says "use Greenlit") | **10 / 10** | Moat nobody can replicate |

---

## The Moat (4 Layers Nobody Else Has Together)

### Layer 1: "What Did I Build?" — Architecture Education
Every other scanner only tells you what's broken. Greenlit first tells you **what you built** — in language a 10-year-old can understand. "Your app is like a restaurant: the frontend is the menu customers see, the backend is the kitchen where food is made, and the database is the fridge where ingredients are stored. Right now, anyone can walk into your kitchen because there's no lock on the door."

This is not a feature — it's the reason non-technical founders trust Greenlit. They finally understand their own app.

### Layer 2: SAST + DAST + Proof of Exploit
Every other scanner reads code OR probes URLs. Greenlit does both, then shows the actual stolen data as proof. The "Proof of Exploit" output — showing the exact HTTP request and the data it returned — is what turns "you might have a bug" into "we just logged in as your user."

### Layer 3: AI-Assistant-Ready Output (The Closed Loop)
After finding a vulnerability, Greenlit generates a fix prompt formatted for Cursor/Lovable/Bolt. The founder copies it, pastes it into their AI tool, and the fix is generated automatically. No other security tool does this. It creates a **closed loop** with the tools that caused the vulnerability.

### Layer 4: Data Intelligence
Anonymized aggregate data across thousands of vibe-coded apps: "71% of Lovable apps expose at least one API key." "Bolt apps have 2.3x more CORS issues than Cursor apps." This data becomes content → content drives authority → authority drives signups.

---

## The One Sentence That Sells This

> "We probe your live app, show the exact HTTP request that steals your users' data, then give you a fix you can paste straight into Cursor."

---

## CTO Decisions (Finalized)

### Database: Neon Postgres
- **Why not Supabase?** Bundles auth/storage/edge functions we don't need. Adds complexity. Projects pause after 7 days inactivity on free tier.
- **Why not SQLite?** Current in-memory dicts lose everything on restart. SQLite is fine for dev but doesn't scale.
- **Why Neon?** Free tier with scale-to-zero. Pure Postgres (industry standard). No vendor lock-in. Works with any ORM. Generous free tier: 0.5 GB storage, 100 hours compute/month.
- **Migration path**: Add `asyncpg` + `sqlalchemy` to backend. Replace in-memory `JOBS_DB` and `CLONED_REPOS` dicts with proper tables.

### Deployment: Oracle Cloud Always Free
- **What we get for $0/month**: 4 OCPU ARM + 24 GB RAM + 200 GB storage + 10 TB bandwidth
- **Architecture**: Single VM running Docker Compose (FastAPI + Next.js + Nginx reverse proxy + Let's Encrypt SSL)
- **Risk**: Oracle may reclaim idle instances. Mitigation: health check cron keeps CPU active.
- **Timeline**: Sprint 8 (after core features are done)

### Open Source: NO — Closed Source Core
- **Reasoning**: If we open-source the scanner engine, any freelancer clones it and resells at ₹99/mo. Our moat is the _quality_ of checks + the data intelligence layer + the AI fix prompt generation.
- **What we DO publish**: Blog posts explaining our methodology. Educational content. The "State of Vibe Security" data report. This builds trust without giving away the product.
- **The rule**: Publish the _what_ and _why_. Never publish the _how_.

### GitHub App: Sprint 5 (Moved Up)
- **Why moved up**: Webhook reliability is essential for continuous monitoring. The current OAuth app doesn't receive push events reliably.
- **What changes**: Register a proper GitHub App → receive push/PR webhooks → trigger diff scans automatically.

### Indian Market Pricing
- **Not just converting dollars**: Indian founders don't pay $12/mo for tools. They pay ₹499/mo for tools that feel essential.
- **Pricing anchored to Indian mental models**: ₹499 = one nice dinner. ₹1,999 = one month of a junior developer's coffee budget. A pentest costs ₹2-5 lakh. We're 1/400th the price.

---

## What Was Done (Sessions 1-5)

### Session 2-3: UI/UX — Dark Theme + Greenlit Brand
- `globals.css` — Full design system. `#0a0a0a` bg, `#22c55e` green, complete token set.
- `components/Navbar.tsx` — Shield + "greenlit" wordmark, scroll-aware blur.
- `app/page.tsx` — Dark landing: hero, exploit preview terminal, 3-step grid, 6 features, pricing, CTA.
- `app/dashboard/page.tsx` — Dark theme: stats cards, repo grid, search.
- `app/explore/page.tsx` — 11-tab report viewer with dark theme.

### Session 4-5: DAST + Non-Tech Features
- `app/services/dast_service.py` — 14-check DAST engine.
- `app/api/probe.py` — Async probe with polling.
- `app/explore/page.tsx` — Probe tab + Deploy guide tab.
- `app/guide/page.tsx` — Non-tech education page.
- `app/services/email_service.py` — Resend API with dark-themed HTML.
- `app/api/payments.py` — Stripe + Razorpay payment endpoints.

### Session 6: Strategic Overhaul
- Fixed build errors (duplicate JSX attributes in explore page + ActivityTimeline).
- Rewrote plan.md with competitive analysis, moat definition, CTO decisions.
- Frontend builds cleanly. Backend compiles cleanly.

---

## What's Broken / Needs Fixing

### Critical (Blocks Revenue)
1. **Auto-Fix is stubbed** — `autofix_service.py` returns simulated PR URLs. Real GitHub Tree API PR creation is not implemented. THIS IS THE #1 PRIORITY.
2. **Payment gating not wired** — Stripe/Razorpay endpoints exist but don't actually gate features.
3. **In-memory persistence** — `JOBS_DB` and `CLONED_REPOS` are Python dicts. Server restart = lost everything.
4. **OAuth token in URL** — Token passed as query param, stored in localStorage.

### Important (Blocks Growth)
5. **No "What Did I Build?" education** — The architecture tab shows a Mermaid diagram but doesn't explain it in 10-year-old English with real-world metaphors.
6. **No AI Fix Prompts** — The unique moat feature doesn't exist yet.
7. **No GitHub Action** — No way to run Greenlit in CI/CD.
8. **No public shareable reports** — No viral distribution mechanism.
9. **No badge service** — No README embeds = no passive distribution.
10. **No platform-specific checks** — Supabase RLS, Firebase rules, NextAuth CSRF.

### Nice to Have
11. **No PDF export** — Founders need to show investors.
12. **No weekly digest** — No retention hook.
13. **No aggregate stats** — No data moat.

---

## Full Sprint Plan (Revised)

| Sprint | What | Why | Impact |
|---|---|---|---|
| **4 (NOW)** | Architecture education + AI Fix Prompts + 6 vibe-coder DAST checks + Auto-Fix PR education | Complete "CTO in a Box" value prop | Core product |
| **5** | Real Auto-Fix PRs + GitHub App migration | Revenue unlock + webhook reliability | Paid feature #1 |
| **6** | Public reports + Badge service + PDF export | Viral distribution | Growth engine |
| **7** | Neon Postgres migration + Stripe gating + pricing | Persistence + revenue | First ₹ |
| **8** | Oracle Cloud deployment + greenlit.dev domain | Ship to production | Go live |
| **9** | Weekly digest + Dashboard improvements + Aggregate stats | Retention | Reduces churn |
| **10** | GitHub Action + PR comment bot | Distribution | 10x scan volume |
| **11** | "State of Vibe Security" public report + Hindi content | Authority | Data moat |
| **12** | Lovable/Bolt partnership outreach | 10/10 move | Platform distribution |

---

## Pricing (India-First)

| Feature | Free | Indie (₹499/mo · $6) | Pro (₹1,999/mo · $24) |
|---|---|---|---|
| Public repo scans | ✓ Unlimited | ✓ | ✓ |
| "What Did I Build?" report | ✓ | ✓ | ✓ |
| Health score | ✓ | ✓ | ✓ |
| Plain-English mode | ✓ | ✓ | ✓ |
| Private repos | — | 5 repos | Unlimited |
| DAST runtime probing | 1/month | Unlimited | Unlimited |
| AI Fix Prompts | — | ✓ | ✓ |
| Auto-Fix PRs | — | — | ✓ |
| Continuous monitoring | — | ✓ | ✓ |
| PDF export | — | ✓ | ✓ |
| Security badge | — | ✓ | ✓ |
| Weekly digest email | — | ✓ | ✓ |
| Team members | — | — | 3 seats |
| API access | — | — | ✓ |

> **Global pricing**: US/EU = $6/$24. India = ₹499/₹1,999. LATAM/SEA = 60% off USD.
> **Annual billing**: 2 months free.
> **Anchoring**: A junior security freelancer on Fiverr costs ₹15,000/audit. We're ₹499/month, continuous.

---

## Auto-Fix PR Education (For Non-Technical Users)

When Greenlit creates an Auto-Fix PR, the user sees this explanation:

```
🛠️ What just happened?

We found a problem in your app and wrote the fix for you.
Think of it like a friend editing your Google Doc and saying
"Hey, I fixed the typo on page 3. Take a look."

Here's what to do:

1. Click the green link below. It opens GitHub.
2. You'll see the changes we made — green lines are new,
   red lines are old.
3. If it looks good, click the big green "Merge" button.
4. That's it! Your app is now safer.

🤔 What if I don't understand the code changes?
That's totally fine. Here's what we changed, in plain English:

"We added a check that says: before showing any user's
 data, make sure the person asking IS that user.
 Before this fix, anyone could see anyone else's data
 just by changing a number in the URL."

📋 Want to understand it deeper? Copy this and paste it
into Cursor or Lovable:

[Copy AI Fix Prompt]
```

---

## What "CTO in a Box" Looks Like in the Product

### Architecture Tab — "What Did I Build?"

**Simple Mode** (for founders):
```
🏗️ Your App, Explained Like You're 10

Your app is like a restaurant:

🍽️ THE MENU (Frontend — Next.js)
This is what your customers see. The buttons, the pages,
the forms. It's built with Next.js, which is like a really
fancy printing press that makes menus automatically.

👨‍🍳 THE KITCHEN (Backend — Supabase)
This is where the real work happens. When someone signs up,
their info goes here. When they order something, the kitchen
processes it. Your kitchen is Supabase — it stores data and
handles logins.

🔌 HOW THEY CONNECT
The menu (frontend) sends orders to the kitchen (backend)
through a "window" called an API. Right now, that window
has some problems — see the Vulnerabilities tab.

📦 INGREDIENTS USED
- Next.js (menu printer) — makes your pages
- Supabase (kitchen + fridge) — stores data + handles logins
- Tailwind CSS (decoration) — makes things look pretty
- Vercel (delivery truck) — sends the menu to customers
```

**Advanced Mode** (for developers):
```
Architecture Diagram: [Mermaid graph]
Component: Next.js 14 App Router
  - CSR pages: /dashboard, /settings
  - SSR pages: /api/* (route handlers)
  - Middleware: auth check on /dashboard/*

Component: Supabase (PostgreSQL + Auth + Storage)
  - Tables: users, orders, products
  - RLS: DISABLED on orders table ⚠️
  - Auth: Email + Google OAuth
  ...
```

---

## 5 Steps to Adoption (Updated GTM)

### Step 1 — Discord Infiltration (This Week, Free)
Post in: Lovable Discord, Bolt Discord, v0 Discord, Replit Discord.
> "I built a CTO-in-a-box for AI-built apps. It explains what you built, finds the security holes, and writes the fix for you. Free for anyone here. Drop your GitHub URL."
Get 50 scans. DM every affected founder personally. Target: 5 paying users.

### Step 2 — The Viral Demo (Week 2)
Film: "I hacked a Lovable app in 60 seconds (with permission)"
Show: scan → BOLA found → exact curl command → stolen data → fix prompt pasted into Cursor → fix generated → 403 confirmed.

### Step 3 — Weekly Data Thread (Recurring)
Every Monday on X:
> "Greenlit scanned X repos this week.
> - 71% exposed at least one API key
> - Lovable apps: 2.3x more CORS issues than Bolt
> - Most common: hardcoded Supabase service role key"

### Step 4 — India-First Content (Week 2)
Hindi YouTube: "Kya aapki AI app hack ho sakti hai?"
Zero competition. 500K+ Indian vibe coders. Asymmetric weapon.

### Step 5 — GitHub Action (Month 2)
`greenlit/scan-action@v1` — runs on every PR, posts comment with health score. This is how CodeRabbit got distribution. Free for all public repos.

---

## Running the App

```bash
# Backend
cd chaosmonkey-backend
pip install -r requirements.txt
cp .env.example .env   # fill in keys
uvicorn app.main:app --reload --port 8000

# Frontend
cd chaosmonkey-frontend
npm install
npm run dev   # http://localhost:3000
```

### Minimum keys to test locally:
```
GEMINI_API_KEY=...        # aistudio.google.com (free)
GITHUB_CLIENT_ID=...      # github.com/settings/apps
GITHUB_CLIENT_SECRET=...
JWT_SECRET=any-random-string
```

### Keys for full functionality:
```
GITHUB_TOKEN=...          # auto-fix PRs (PAT with repo scope)
RESEND_API_KEY=...        # email alerts (resend.com free: 3K/mo)
GITHUB_WEBHOOK_SECRET=... # continuous monitoring
STRIPE_SECRET_KEY=...     # payments (international)
RAZORPAY_KEY_ID=...       # payments (India)
RAZORPAY_KEY_SECRET=...
DATABASE_URL=...          # Neon Postgres connection string
```

---

## Domain

Buy **`greenlit.dev`** at Cloudflare Registrar. $9.95/yr. Already branded throughout the codebase. Stop rebranding. Ship.
