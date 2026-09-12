# Decisions Log

One line per decision: `YYYY-MM-DD · decision · why`.

- 2026-09-09 · Project = `repoqa`, a codebase Q&A assistant (RAG + chat, cites files/lines) · widest coverage of the AI engineer role; dev-tools domain; retrieval is hard for code so it forces real learning
- 2026-09-09 · Target repo = `../MiroFish` · real, mid-sized, itself an AI system; Joey was already exploring it
- 2026-09-09 · Generation LLM = Anthropic Claude — Sonnet 5 (`claude-sonnet-5`) workhorse, Haiku 4.5 (`claude-haiku-4-5`) for bulk ops, Opus 5 (`claude-opus-5`) for occasional comparison only · strong enough not to confound retrieval work, low enough cost to iterate on evals freely
- 2026-09-09 · Cost guardrails · set Anthropic Console spend limit (~$50 hard cap, alert ~$25); prefer prepaid credits; every batch/loop call prints an estimated cost and caps iterations before running
