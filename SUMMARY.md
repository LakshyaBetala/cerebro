# ChaosMonkey — Project Summary

## What Is This?

ChaosMonkey is an **AI-powered security and architecture analysis platform** built for people who use AI tools (Cursor, Cline, Copilot) to build apps but don't fully understand what they've built.

The core idea: **Upload your GitHub repo → Understand your architecture → Fix your security flaws** — all without needing to be a security expert.

---

## The Problem

"Vibe coders" use AI assistants to generate entire applications. They ship fast, but they often:
- Don't understand the architecture they've built
- Ship hardcoded secrets, SQL injection risks, and insecure auth flows
- Can't explain their own codebase to investors or collaborators

---

## The Solution

ChaosMonkey acts as an **AI Brain** that:

1. **Ingests your repo** — Clones, chunks, and embeds every file using local HuggingFace models
2. **Indexes into a vector database** — ChromaDB stores the codebase vectors locally (free, no API costs)
3. **Queries an LLM** — Google Gemini 2.5 Pro analyzes the relevant code chunks and returns:
   - A **Health Score** (0-100)
   - A **Plain English explanation** of the architecture (for non-technical founders)
   - An **Architecture Diagram** (nodes & edges showing Frontend → API → DB)
   - A **Vulnerability List** with file paths, line numbers, and severity
4. **Generates Auto-Fix PRs** — Uses Gemini to write the exact code patches and opens a Pull Request on your repo

---

## Architecture Overview

```
┌─────────────────────────────────────────────────┐
│                    FRONTEND                      │
│              Next.js 16 + Tailwind               │
│                                                   │
│  Landing ─→ OAuth ─→ Dashboard ─→ Analyze Page   │
│                                                   │
│  [Bento Grid UI]  [AI Sidekick Chat]  [Auto-Fix] │
└─────────────────────┬───────────────────────────┘
                      │ REST API (fetch + polling)
                      ▼
┌─────────────────────────────────────────────────┐
│                    BACKEND                       │
│              FastAPI + LangChain                 │
│                                                   │
│  POST /api/repos/analyze  ─→  Background Task    │
│  GET  /api/repos/jobs/:id ─→  Poll Results       │
│  POST /api/repos/autofix  ─→  Generate PR        │
│                                                   │
│  ┌──────────────────────────────────────────┐    │
│  │           RAG PIPELINE                    │    │
│  │  Clone → Chunk → Embed → Store → Query   │    │
│  │                                           │    │
│  │  HuggingFace ──→ ChromaDB ──→ Gemini     │    │
│  │  (Local, Free)   (Local)     (Free API)   │    │
│  └──────────────────────────────────────────┘    │
└─────────────────────────────────────────────────┘
```

---

## Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| Local embeddings (HuggingFace) | $0 cost. No per-token API charges for large repos. |
| Local vector DB (ChromaDB) | No cloud dependency. Works offline. |
| Google Gemini (Free Tier) | Generous free quota. Supports `application/json` output. |
| Background Tasks + Polling | RAG takes 10-30 seconds. Polling prevents HTTP timeouts. |
| Demo/Simulation Mode | Auto-fallback when API keys are missing. App never crashes. |
| Glassmorphism Bento Grid UI | Premium, cinematic aesthetic that differentiates from "AI slop". |

---

## Current Status

- ✅ GitHub OAuth login flow
- ✅ Repository listing, search, filter
- ✅ Clone + Import repos to backend
- ✅ Full RAG pipeline (chunk → embed → query → structured JSON)
- ✅ Bento Grid results dashboard with GSAP animations
- ✅ Plain English "What You Built" explanation
- ✅ Auto-Fix PR generation (simulation mode)
- ✅ Severity color-coded vulnerability badges
- ✅ Error handling with retry buttons
- ✅ Production `.env` configuration

---

## What's Next (Future Roadmap)

- [ ] Real GitHub PR creation via GitHub Tree API
- [ ] Mermaid.js diagram rendering for architecture
- [ ] Chat-based Q&A with the AI Sidekick (RAG-powered)
- [ ] Webhook-based analysis triggers (auto-scan on push)
- [ ] Multi-repo comparison dashboard
- [ ] Export reports as PDF
