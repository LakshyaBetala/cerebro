# ChaosMonkey — AI Security & Architecture Platform

> **Upload your repo. Understand what you built. Fix security flaws with one click.**

ChaosMonkey is an AI-powered platform designed for **vibe coders** and non-technical founders. It uses Retrieval-Augmented Generation (RAG) to deeply analyze your GitHub repository — mapping out the architecture, finding security vulnerabilities, and explaining everything in plain English.

---

## Features

- **GitHub OAuth Login** — Connect your GitHub account and import repositories with one click.
- **RAG-Powered Analysis** — Uses local HuggingFace embeddings + ChromaDB to index your codebase, then queries Google Gemini to generate a full security & architecture report.
- **Plain English Explanations** — The "Vibe Coder" mode explains your tech stack and architecture in simple, non-technical language.
- **Auto-Fix Engine** — Generates code patches and opens Pull Requests to fix detected vulnerabilities automatically.
- **Cinematic Dark Glass UI** — A premium Bento Grid dashboard with GSAP animations and glassmorphism design.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 16, Tailwind CSS 4, GSAP, Lucide Icons |
| Backend | Python FastAPI, LangChain, ChromaDB |
| LLM | Google Gemini 2.5 Pro (Free Tier) |
| Embeddings | HuggingFace `all-MiniLM-L6-v2` (Local, Free) |
| Auth | GitHub OAuth 2.0 |

---

## Project Structure

```
agent_ops/
├── chaosmonkey-backend/          # FastAPI Backend
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
├── chaosmonkey-frontend/         # Next.js Frontend
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
| Variable | Required | Description |
|----------|----------|-------------|
| `GITHUB_CLIENT_ID` | ✅ | From your GitHub OAuth App |
| `GITHUB_CLIENT_SECRET` | ✅ | From your GitHub OAuth App |
| `JWT_SECRET` | ✅ | Any random string for session signing |
| `GEMINI_API_KEY` | ✅ | Free key from [aistudio.google.com](https://aistudio.google.com/apikey) |
| `GITHUB_TOKEN` | ❌ | Optional — for Auto-Fix PR creation |

### Frontend (`chaosmonkey-frontend/.env.local`)
| Variable | Required | Description |
|----------|----------|-------------|
| `NEXT_PUBLIC_API_URL` | ✅ | Backend URL (default: `http://localhost:8000`) |

> **How to get GitHub OAuth keys:** Go to [github.com/settings/developers](https://github.com/settings/developers), create a new OAuth App with callback URL `http://localhost:8000/auth/github/callback`.

---

## User Flow

1. **Landing Page** → Click "Get Started Free"
2. **GitHub OAuth** → Authorize the app
3. **Dashboard** → See all your repositories, search/filter, import repos
4. **Import** → Click "Import" to clone a repo to the backend
5. **Analyze** → Click the Play button to start RAG analysis
6. **Results** → View health score, architecture diagram, vulnerabilities
7. **Auto-Fix** → Click "Generate Fixes" to create a PR with patches

---

## License

MIT
