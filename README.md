# Greenlit — Build Safe with AI

> **Paste your GitHub repo. Get a plain-English audit, security check, and 1-click fix in 60 seconds. The always-on safety net for AI-built apps.**

Greenlit is the trust layer for vibe coders and non-technical founders shipping with Lovable, Bolt, v0, Antigravity, Cursor, and Replit Agent. It uses RAG (HuggingFace embeddings + ChromaDB + Gemini) to map your architecture, find vulnerabilities, triage what actually matters, and fix the criticals with one click.

> **Note**: The repository folders are still named `chaosmonkey-backend/` and `chaosmonkey-frontend/` from the project's prior identity. The folder rename is a separate git-submodule migration step (see Sprint 1 checklist).

---

## Features

- **Plain-English audit** — Toggle between Simple mode (10-year-old-friendly, consequence-driven) and Technical mode.
- **RAG-powered analysis** — Local HuggingFace embeddings + ChromaDB index your codebase, then Gemini 2.5 Pro generates a 10-key report.
- **Triage that respects your time** — We surface what's broken AND what we ignored (and why), so you don't drown in noise.
- **Auto-Fix PRs** — One click opens a real GitHub PR fixing the criticals (real PR creation lands in Sprint 5).
- **Continuous monitoring** *(Sprint 4+)* — GitHub App webhook re-scans on every push.
- **Greenlit badge** *(Sprint 8)* — Public score + README badge for social proof.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 16, Tailwind CSS 4, GSAP, Lucide Icons |
| Backend | Python FastAPI, LangChain, ChromaDB |
| LLM (audit) | Google Gemini 2.5 Pro |
| LLM (monitoring re-scans, Sprint 6+) | Anthropic Claude Haiku 4.5 |
| Embeddings | HuggingFace `all-MiniLM-L6-v2` (local, free) |
| Auth | GitHub OAuth 2.0 → GitHub App (Sprint 4) |
| DB (Sprint 2+) | Supabase Postgres |
| Queue (Sprint 2+) | Upstash Redis |
| Hosting | Oracle Cloud Free Tier (backend) + Vercel (frontend) |
| Payments (Sprint 7+) | Stripe + Razorpay (India) |

---

## Project Structure

```
agent_ops/
├── chaosmonkey-backend/          # FastAPI Backend (folder rename pending)
│   ├── app/
│   │   ├── api/repos.py          # REST endpoints (clone, analyze, autofix, jobs)
│   │   ├── auth/github.py        # GitHub OAuth login/callback
│   │   ├── services/
│   │   │   ├── rag_service.py    # RAG pipeline (chunk → embed → query LLM)
│   │   │   ├── autofix_service.py # Generate patches + PR creation
│   │   │   ├── repo_clone.py     # Git clone handler
│   │   │   └── github_service.py # GitHub API wrapper
│   │   ├── config.py             # Environment variable loader
│   │   └── main.py               # FastAPI app entry point
│   ├── .env.example              # Required environment variables
│   └── requirements.txt          # Python dependencies
│
├── chaosmonkey-frontend/         # Next.js Frontend (folder rename pending)
│   ├── app/
│   │   ├── page.tsx              # Landing page
│   │   ├── dashboard/page.tsx    # Repository dashboard
│   │   ├── analyze/page.tsx      # Bento Grid analysis results
│   │   └── auth/callback/        # GitHub OAuth callback handler
│   ├── components/               # Reusable UI components
│   ├── services/api.ts           # API client (fetch wrappers)
│   ├── types/index.ts            # TypeScript interfaces
│   ├── .env.example              # Frontend env variables
│   └── package.json
│
├── README.md                     # This file
└── SUMMARY.md                    # Project overview & architecture
```

---

## Setup & Installation

### Prerequisites
- Python 3.10+
- Node.js 18+
- Git

### 1. Clone the Repository
```bash
git clone https://github.com/LakshyaBetala/cerebro.git
cd cerebro
```

### 2. Backend Setup
```bash
cd chaosmonkey-backend

# Create environment file
copy .env.example .env
# Fill in your keys (see Environment Variables below)

# Install dependencies
pip install -r requirements.txt

# Start the server
uvicorn app.main:app --reload --port 8000
```

### 3. Frontend Setup
```bash
cd chaosmonkey-frontend

# Create environment file
copy .env.example .env.local

# Install dependencies
npm install

# Start the dev server
npm run dev
```

### 4. Open the App
Visit **http://localhost:3000** and click **"Get Started Free"** to begin.

---

## Environment Variables

### Backend (`chaosmonkey-backend/.env`)

**Sprint 1 (current — what you need today):**

| Variable | Required | Description |
|----------|----------|-------------|
| `GITHUB_CLIENT_ID` | ✅ | From your GitHub OAuth App |
| `GITHUB_CLIENT_SECRET` | ✅ | From your GitHub OAuth App |
| `JWT_SECRET` | ✅ | Any random string for session signing |
| `GEMINI_API_KEY` | ✅ | Free key from [aistudio.google.com](https://aistudio.google.com/apikey) |
| `GITHUB_TOKEN` | ❌ | Optional — for Auto-Fix PR creation |

**Future sprints (don't fill yet, scaffolded in `.env.example`):**

| Variable | Sprint | Description |
|----------|--------|-------------|
| `SUPABASE_URL`, `SUPABASE_KEY` | 2 | Postgres for users/repos/audits |
| `UPSTASH_REDIS_URL` | 2 | Job queue + dedup cache |
| `ANTHROPIC_API_KEY` | 6 | Claude Haiku 4.5 for monitoring scans |
| `RESEND_API_KEY` | 10 | Email digests + alerts |
| `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET` | 7 | Payments (US/EU) |
| `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET` | 7 | Payments (India) |
| `R2_ACCOUNT_ID`, `R2_ACCESS_KEY`, `R2_SECRET_KEY` | 8 | Cloudflare R2 for report blobs |
| `GITHUB_APP_ID`, `GITHUB_APP_PRIVATE_KEY` | 4 | GitHub App for webhooks |

### Frontend (`chaosmonkey-frontend/.env.local`)
| Variable | Required | Description |
|----------|----------|-------------|
| `NEXT_PUBLIC_API_URL` | ✅ | Backend URL (default: `http://localhost:8000`) |

> **How to get GitHub OAuth keys:** Go to [github.com/settings/developers](https://github.com/settings/developers), create a new OAuth App with callback URL `http://localhost:8000/auth/github/callback`.

---

## User Flow

1. **Landing Page** → Click "Get Started Free"
2. **Paste GitHub URL** → No signup needed for first audit
3. **Live progress** → "Cloning... Reading... Checking... Writing report..."
4. **Report** → Health score, plain-English explanation, vulnerabilities, ignored issues, fix suggestions
5. **Connect GitHub** → For 1-click auto-fix (Sprint 5)
6. **Enable monitoring** *(Sprint 4)* → Every push triggers a re-scan + email alert

---

## License

MIT
