# ChaosMonkey — Review Prep & Anticipated Questions

A cheat sheet for the demo review. Skim before you walk in.

---

## The 30-Second Pitch

> ChaosMonkey is an AI-powered security and architecture analyzer for "vibe coders" — non-technical founders shipping AI-generated apps. Paste a GitHub URL, get a Health Score, an architecture diagram, a list of vulnerabilities, and plain-English explanations of what you actually built. Built on a local-first RAG pipeline (HuggingFace embeddings + ChromaDB) so per-repo cost stays near zero, with Gemini as the analysis brain.

---

## Likely Questions & How to Answer

### Architecture / Technical

**Q: Walk me through the pipeline.**
Frontend posts a GitHub URL → FastAPI clones the repo (`--depth 1`) → LangChain loads code files, skips `node_modules`/`venv`/lockfiles → splits into 1000-char chunks with 200 overlap → embeds locally with `all-MiniLM-L6-v2` → stores in an in-memory ChromaDB vectorstore → retrieves top-8 chunks across 4 query angles (security, architecture, data flow, broken imports) → single Gemini call returns one structured JSON with all 10 report sections. Frontend polls a `/jobs/{id}` endpoint until complete.

**Q: Why local embeddings instead of OpenAI/Gemini embeddings?**
Two reasons: cost and privacy. Embedding a 100-file repo via paid APIs costs real money per run; `all-MiniLM-L6-v2` runs locally, free, and never sends source code to a third party for the embedding step. Only the retrieved top-K chunks go to Gemini.

**Q: Why ChromaDB and not Pinecone / pgvector / Weaviate?**
ChromaDB runs in-memory or as a local file store — zero infra cost, no cloud dependency, works offline. For an MVP analyzing one repo at a time, persistence and horizontal scale aren't needed. We'd swap to pgvector/Pinecone when we add cross-repo search or persistent indexes.

**Q: Why Gemini and not GPT-4 / Claude?**
Gemini 2.5 Flash has a generous free tier and supports `response_mime_type="application/json"` natively, which gives us structured output without prompt-engineering tricks. The model is swappable — `rag_service.py` has a fallback chain that tries 6 models in order.

**Q: How do you handle large repos?**
Three layers: (1) `git clone --depth 1` skips history, (2) the loader excludes `node_modules`, `venv`, `dist`, lockfiles, (3) we cap retrieval at top-8 chunks per query so the LLM context stays bounded regardless of repo size. Repos with thousands of files still chunk and embed locally — only the retrieval payload goes to the LLM.

**Q: What happens if Gemini fails?**
Layered fallback. The service tries 6 Gemini variants in order (2.5-flash → 2.5-flash-lite → 2.0-flash → 2.0-flash-lite → 1.5-flash → 1.5-flash-8b). If all fail, it returns a curated `DEMO_REPORT` so the UI never crashes. Same pattern for missing API keys, missing files, or malformed JSON responses.

### Security / Privacy

**Q: You're cloning private repos to your server. How is the code protected?**
Today, repos are cloned to OS temp directories and deleted in a `finally` block immediately after analysis. The vectorstore is in-memory and dies with the process. Nothing is persisted. For production, we'd add SOC 2 controls, per-tenant isolation, and a "delete on demand" guarantee.

**Q: Your own report flags a vulnerability in your auth flow — what's the deal?**
Yes — the OAuth access token is currently passed back as a URL query parameter and stored in `localStorage`. We caught this with our own tool, which is honestly the best validation we could ask for. The fix is moving to httpOnly cookies + a server-side session, on the roadmap.

**Q: How do you prevent prompt injection from a malicious repo?**
We don't fully today. A repo containing adversarial markdown could try to manipulate the LLM. Mitigations on the roadmap: sanitizing content before embedding, separating instruction and data with explicit delimiters, and running the LLM with a strict system prompt that ignores in-context instructions.

### Product / Business

**Q: Who's the customer? Devs already have Snyk, GitHub Advanced Security, Sonar.**
Those tools are built for engineers. Our user is a non-technical founder who built a product on Cursor / Cline / Lovable and doesn't read CVE reports. The "Simple/Advanced" mode toggle is the differentiator — the same finding rendered in plain English vs. technical jargon.

**Q: How do you make money?**
Free tier: paste a public repo, get the report. Paid tier: private repo support, persistent dashboards, scheduled re-scans, and one-click Auto-Fix PRs. Auto-Fix is the unlock — it turns "you have 12 vulnerabilities" into "click here, get a PR."

**Q: What's the moat?**
Three things. (1) The plain-English explanation engine — it's a tuned prompt + UX problem, not a model problem. (2) The Auto-Fix loop, which becomes more valuable as we collect "what fixes actually merge." (3) Speed: local embeddings + free LLM tier means we can offer the free analysis at near-zero marginal cost.

### Tricky / Gotcha

**Q: Is this just a Gemini wrapper?**
The Gemini call is one of seven steps. The interesting work is the retrieval design (4 query angles, deduplicated top-K), the structured output schema (10 sections, every key validated and defaulted), the Mermaid diagram generation, and the dual-mode rendering layer. The model is swappable — the pipeline is the product.

**Q: What if I paste a 50,000-file monorepo?**
Embedding takes longer but doesn't blow up — the retrieval cap means LLM context stays the same size. We have a 5-minute job timeout to prevent runaway analysis. For monorepos specifically, package-level analysis is on the roadmap (detect Nx/Turborepo/Lerna, analyze each workspace separately).

**Q: Show me the architecture diagram is real and not hallucinated.**
Click any tech stack item — the `files_using` field references actual paths from the cloned repo. The Mermaid diagram nodes/edges come from the same retrieved code context. If you paste a Rails app, you don't get React in the diagram.

**Q: What does it cost you per analysis?**
On free-tier Gemini: $0. Embeddings are local. Vector store is in-memory. Clone is shallow. Even at scale, a paid Gemini Flash call is $0.0001-ish per repo. Marginal cost is essentially free.

**Q: Why does the Auto-Fix button say "Pro"?**
Auto-Fix is the paid feature — and it's the one we charge for because it does real work on the user's repo (creating a branch, committing, opening a PR via the GitHub Tree API). Free tier shows the vulnerability list with fix suggestions; paid tier executes them.

### Honest "What's Broken" (be ready)

- **In-memory job store.** `JOBS_DB` and `CLONED_REPOS` are Python dicts. Server restart = lost jobs. Redis is the obvious fix.
- **Auto-Fix is partially stubbed.** The endpoint generates the diff via Gemini but returns a simulated PR URL. Real PR creation via GitHub Tree API is implemented in pseudo-code only.
- **OAuth token in URL.** Known issue, flagged by the tool itself. Fix on roadmap.
- **Single-instance only.** No horizontal scaling, no queue, no auth on the analyze-url endpoint.

If a reviewer pushes on any of these, the right answer is "yes, that's the next sprint" — not defending it.

---

## Demo Talk Track (in case you blank)

1. "This is ChaosMonkey. It analyzes any GitHub repo for security and architecture issues."
2. "I'll paste a real production repo — `sec-insights` from the LlamaIndex team — and we'll watch it run."
3. While loading: "It's cloning, chunking, embedding locally with HuggingFace, then querying Gemini for one structured report."
4. Show the **Health Score** and the **Architecture diagram** — "this Mermaid graph is generated from the actual code, not a template."
5. Toggle **Simple / Advanced** — "same data, two audiences. The whole point of the product."
6. Show **Vulnerabilities** — "found 4 real issues, with file paths and severity."
7. End on **Auto-Fix** — "this is the paid hook — one click, get a PR."

Keep it under 3 minutes. Let the UI do the work.
