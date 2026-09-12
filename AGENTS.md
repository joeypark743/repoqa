# AGENTS.md — AI Engineer Learning Project

## Purpose

This is a **guided learning project**. The real deliverable is not the codebase — it's
Joey understanding, concretely, what the job of an **AI engineer** is: the decisions,
the tradeoffs, the daily craft.

We build one real system together. Codex coaches; Joey drives. Shipping working code is
the vehicle, not the point.

> This file is a living document. It is v0. We revise it as the project teaches us things.

---

## What an "AI engineer" does (the map we're studying)

Not a research scientist (trains new models) and not a classic ML engineer (pipelines,
feature stores). An **AI engineer builds products on top of existing foundation models**:

- **Prompt & context engineering** — getting reliable behavior out of a model
- **Retrieval (RAG)** — feeding the model the right information at answer time
- **Evaluation** — measuring quality so changes are improvements, not vibes
- **Iteration loop** — change one thing, re-measure, keep what wins
- **Orchestration** — chaining calls, tools, agent loops
- **Production concerns** — latency, cost, observability, guardrails, failure modes

This project touches every item on that list at least once.

---

## The project

**Working name:** `repoqa` (rename freely — that's Joey's call)

A **codebase Q&A assistant**: point it at a code repository, ask it questions in chat
("how does auth work here?", "where do we handle rate limiting?", "what changed in the
retry logic recently?"), get answers **with citations to specific files and lines**.

Why this project:
- It's genuinely useful in **developer tools**, the domain Joey picked.
- Code is *hard* to retrieve well — it forces us to confront retrieval quality head-on
  instead of getting lucky.
- It has a clean evaluation story (did it cite the right files? is the answer correct?).
- It grows naturally from a 100-line script to a system with real production concerns.

**Target repo to point it at:** `../MiroFish` (github.com/666ghj/MiroFish) — a multi-agent
AI prediction engine. ~35 Python files (backend), ~16 Vue files (frontend), Docker,
bilingual EN/ZH docs. Good fit: mid-sized, real, and itself an AI system, so questions
about it are substantive.

*Known tradeoff:* Joey is still new to this repo, so ground truth for the eval set
(Phase 3) gets built by reading the code as we go — which is also the honest use case for
`repoqa`: understanding an unfamiliar codebase. The bilingual docs are a real retrieval
wrinkle we'll hit in Phase 4.

---

## How we work together

The collaboration is **directional — both directions.**

**Joey owns:**
- What we build and why; scope cuts; when a phase is "good enough" to move on
- Which target repo; UX and taste calls
- The final call on every fork in the road

**Codex owns / drives:**
- Teaching the concept *before* we build it
- Laying out 2–3 options with tradeoffs **and a recommendation** at every fork
- Flagging when Joey is about to over-build, or skip a fundamental worth understanding
- Keeping us honest against each phase's verify criteria

**Decision protocol** — at every significant fork, Codex posts:
1. **Options** (2–3, concrete)
2. **Recommendation** + why
3. **Why this choice matters for an AI engineer** (the learning angle)
4. **What I need from you**

Joey decides. We log it in `DECISIONS.md` (one line: date · decision · why).

**Teaching cadence — "concept before code":**
Before each new subsystem, Codex gives a short explainer — what it is, why it exists,
the main alternatives, what an AI engineer is expected to know about it. *Then* we build
the smallest version that works. *Then* we measure it.

**Joey can always say:** "why this and not X", "show me the alternative", "slow down and
explain that", "let me try this part myself first, then review it".

**Session ritual:**
- **Start:** restate the current phase and the next *verifiable* goal.
- **End:** update `LEARNING_LOG.md` (what we built, what Joey learned, open questions) and
  `DECISIONS.md`.

(`DECISIONS.md` and `LEARNING_LOG.md` get created in Phase 0.)

---

## Roadmap

Pace is flexible / no deadline — these are **phases, not weeks**. Each has a verify bar;
we don't advance until it's met.

### Phase 0 — Orientation & setup
- Talk through the AI engineer role map above until it's real, not abstract
- Confirm target repo (`../MiroFish`); choose LLM provider + embedding model (joint decision)
- Repo scaffold, dependencies, API keys, `.env`
- **Verify:** a script calls the model and prints a response · Joey writes one paragraph,
  in his own words, on what an AI engineer does

### Phase 1 — Naive RAG baseline (thin end-to-end slice)
- Ingest: walk the repo, chunk files (naive fixed-size), embed, store in a local vector DB
- Retrieve: top-k by similarity
- Generate: put chunks in the prompt, answer with file citations
- Interface: `repoqa ask "question"` on the CLI
- **Verify:** answers 5 hand-written questions with plausibly-relevant citations · Joey can
  trace one query through every stage and explain each

### Phase 2 — Make it a conversation
- Multi-turn history; query rewriting (turn a vague follow-up into a standalone search query)
- **Verify:** a 4-turn chat where turn 3 is a pronoun-heavy follow-up still retrieves right

### Phase 3 — Evaluation harness  ← *the core AI-engineer skill*
- Build a labeled eval set (~20–40 questions → expected files / expected answer points)
- Metrics: retrieval recall@k, answer correctness (LLM-as-judge), citation accuracy
- Run it, record a **baseline score**, commit the numbers
- **Verify:** `repoqa eval` prints a scorecard · Joey can name the weakest metric and a
  hypothesis for why

### Phase 4 — Improve retrieval (iterate against the evals)
- Symbol/AST-aware chunking · metadata filters · hybrid search (keyword + vector) · reranking
- **Change one thing at a time, re-run evals, keep a results table**
- **Verify:** a measurable gain over the Phase 3 baseline, with the experiment log showing
  what moved the number

### Phase 5 — Productionize
- Observability: log every query — retrieved chunks, tokens, latency, cost
- Guardrails: graceful "I don't know", prompt injection hiding in retrieved code,
  context-window overflow
- A minimal web UI (or genuinely good CLI UX); deploy it or make it one-command runnable
- **Verify:** an inspectable log/dashboard after a session · a real cost-per-query number ·
  a deployed URL or `make run`

### Phase 6 — Reflection
- Short write-up: architecture, what you'd do differently, what the role means to you now
- **Verify:** the doc exists · Joey can give the 5-minute verbal version

---

## Tech stack

**Fixed:**
- **Python** — the default language for this work
- **No RAG framework early** (no LangChain/LlamaIndex). We write the retrieve→prompt→generate
  loop by hand so the moving parts are visible. We can adopt a framework later, on purpose,
  once Joey knows what it's hiding.

**Decided:**
- **Generation LLM:** Anthropic Codex — `Codex-sonnet-5` workhorse, `Codex-haiku-4-5`
  for bulk ops, `Codex-opus-5` for occasional quality comparison only

**Still joint (Phase 0):**
- Embedding model · vector store (start trivial — sqlite-vec / Chroma / FAISS) · CLI framework

**Cost guardrails:**
- Anthropic Console spend limit set (~$50 hard cap, ~$25 alert); prefer prepaid credits
- Token + running-cost counter added in Phase 1 (not Phase 5) — it's cheap and it teaches
- Every batch or looped API call prints an estimated cost and has a hard iteration cap
  *before* it runs; test on a tiny slice first

---

## Anti-goals

We are deliberately **not**:
- training or fine-tuning a model (that's a different job)
- chasing state-of-the-art anything
- building multi-user / multi-tenant infrastructure
- adding config, abstraction, or "flexibility" nobody asked for
- moving to the next phase before its verify bar is green

---

## Definition of done (whole project)

`repoqa` answers real questions about a real repo with accurate citations; there is an eval
suite with a tracked score history showing deliberate improvement; it logs cost and latency;
and Joey can explain — out loud, from the architecture up — every design decision and what
each one has to do with being an AI engineer.
