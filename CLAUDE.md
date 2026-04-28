# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Layout

This is a **two-repo workspace** for the ChaosMonkey project. The two app folders each contain their own `.git` directory and are tracked as nested checkouts (the parent `git status` shows them with the `m` prefix; recent commits reference them as "submodules"). Treat them as independent repos:

- `chaosmonkey-backend/` — FastAPI + LangChain RAG backend (Python)
- `chaosmonkey-frontend/` — Next.js 16 dashboard (TypeScript / React 19)

Always cd into the relevant subdirectory before running install/build/lint commands. Don't run `git add` from the repo root expecting changes inside these subfolders to stage — commit inside each repo.

## Common Commands

### Backend (`chaosmonkey-backend/`)
```bash
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000      # dev server (also runnable via `python -m app.main`)
```
There is no test suite or linter wired up. The server reload excludes `storage/*` and `__pycache__/*` so cloned repos under `.storage/repos/` won't trigger reloads.

### Frontend (`chaosmonkey-frontend/`)
```bash
npm install
npm run dev      # Next dev server on :3000
npm run build    # next build
npm run start    # production server
npm run lint     # eslint (uses eslint-config-next)
```

### Local end-to-end run
Start backend on `:8000` and frontend on `:3000` in separate terminals. The backend's CORS allowlist is hard-coded to `http://localhost:3000` and `http://127.0.0.1:3000` (`app/main.py`) — running the frontend on any other origin will fail CORS preflight. The GitHub OAuth callback is also hard-coded to `http://localhost:8000/auth/github/callback` in `app/auth/github.py`.

## Architecture

### Demo-mode fallback is load-bearing
Every external-dependency code path (`rag_service.py`, `autofix_service.py`, `chat_service.py`) checks for the relevant API key (`GEMINI_API_KEY`, `GITHUB_TOKEN`) and **silently falls back to a hardcoded mock** (`DEMO_REPORT`, simulated PR URLs, canned chat reply) when the key is missing or when no code files are found in a cloned repo. This is intentional — the app must never crash from a missing key. When changing these services, preserve the demo fallback or add an explicit env-driven switch.

### Background-task + polling pattern
Long-running RAG analysis runs through FastAPI `BackgroundTasks`, not synchronously:

1. `POST /api/repos/analyze-url` (the primary v2 endpoint) generates a `job_id`, stores `JOBS_DB[job_id] = {"status": "processing", ...}`, kicks off `process_full_analysis`, and returns immediately.
2. `process_full_analysis` runs `clone_repo` → `generate_analysis_report` → `cleanup_repo`, all wrapped in `asyncio.to_thread(...)` because they are blocking (subprocess git, torch, ChromaDB).
3. Frontend polls `GET /api/repos/jobs/{job_id}` until `status` becomes `complete` or `error`.

Two important pieces of state in `app/api/repos.py` to know about:
- `JOBS_DB` and `CLONED_REPOS` are **in-memory dicts** — they don't survive restarts. Don't write features that assume persistence; if you add one, swap to Redis (already on the roadmap in `SUMMARY.md`).
- `ACTIVE_ANALYSES` deduplicates concurrent analyses of the same URL so a user double-clicking "Analyze" reuses the existing job_id. `MAX_JOB_AGE_SECONDS = 300` auto-expires stuck jobs.

Cloned repos under `REPO_STORAGE_PATH` (`<backend>/.storage/repos/<uuid8>`) are deleted in the `finally` block of `process_full_analysis` to keep disk from filling up. Don't disable that cleanup without an alternative.

### RAG pipeline (`app/services/rag_service.py`)
Heavy ML imports (`torch`, `sentence-transformers`, LangChain loaders, ChromaDB) are **lazy-loaded inside `generate_analysis_report`** so server startup stays fast. Keep them lazy when editing — moving them to module top-level adds ~10s to every `--reload`.

The pipeline:
1. `DirectoryLoader` over a fixed list of `code_patterns` (Python, JS/TS, configs, etc.), skipping `node_modules`, `venv`, `.next`, `dist`, lockfiles.
2. `RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)`.
3. `HuggingFaceEmbeddings("all-MiniLM-L6-v2")` — local, free, no API cost.
4. ChromaDB in-memory vectorstore, retrieved with `k=8` over four canned queries (security, architecture, data flow, broken imports). Results are deduplicated and concatenated.
5. **Single Gemini call** (`gemini-2.5-pro`, `response_mime_type="application/json"`) returns the whole 10-key report (`health_score`, `tech_stack`, `architecture`, `connections`, `data_flow`, `simple_explanation`, `advanced_explanation`, `vulnerabilities`, `broken_links`, `improvements`). The frontend renders sections off this exact shape — when changing the prompt, keep the keys stable or update the consumers in `chaosmonkey-frontend/app/analyze/page.tsx`.
6. After parsing, missing keys are filled from a `defaults` dict so partial responses don't crash the UI.

### Auto-Fix is mostly stubbed
`autofix_service.generate_and_apply_fix` returns simulated PR URLs (`/pull/simulate-123`, `/pull/real-fix-456`). Even the "real" path only asks Gemini for a diff and returns a fake URL — actual GitHub Tree API PR creation is not implemented (see comments at `autofix_service.py:44-51`). When asked to "make the auto-fix work for real," that is a sizable feature, not a bug fix.

### Auth flow has a known security smell
`app/auth/github.py` redirects the access token back to the frontend as a **query parameter** (`?access_token=...`), and the frontend stores it in `localStorage` (`services/api.ts`). This is intentionally listed as a "high" severity vulnerability inside the demo report itself. Don't be surprised by it — but also don't replicate the pattern for new tokens.

### Frontend
- `app/page.tsx` (landing) → `app/dashboard/page.tsx` (lists user repos via `fetchRepos()`) → `app/analyze/page.tsx` (Bento Grid results).
- `services/api.ts` is the single API client; `BASE_URL` defaults to `http://127.0.0.1:8000` and is overridable via `NEXT_PUBLIC_API_URL`.
- Mermaid is rendered via `components/MermaidDiagram.tsx` from the `architecture.mermaid_code` field of the report.
- Tailwind CSS 4 (PostCSS plugin form) and GSAP for animations.

## Environment Variables

Backend `.env` (loaded by `app/config.py` via `python-dotenv`):
- `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET` — required for OAuth.
- `JWT_SECRET` — read but currently unused for signing (placeholder).
- `GEMINI_API_KEY` — when missing, RAG and chat fall back to demo/canned responses.
- `GITHUB_TOKEN` — optional; only consumed by the (stubbed) auto-fix path.

Frontend `.env.local`:
- `NEXT_PUBLIC_API_URL` — defaults to `http://127.0.0.1:8000`.
